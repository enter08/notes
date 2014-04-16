# Upravljanje logovima

Cilj je prepoznati sva mjesta gdje naša aplikacija kreira podatke ili gdje neki korisnik kreira podatke u aplikaciji. Zatim su nam potrebne određene komponente koje će spriječiti da ta mjesta beskonačno rastu. Tip tih komponenti, podrazumijeva se, nije isti. Dakle, potrebna nam je strategija za upravljanje izvora brzog rasta podataka. Nekada, ostaviti ih da rastu je ispravna stvar, kao na primjer, tabela zemalja svijeta. Ali, reći da ta tabela nikada neće biti prazna je, na primjer, dobar korak. 
Od velikog značaja su logovi aplikacije i web servera. Ako se njima dobro ne upravlja, mogu zauzeti veliki prostor disku. 

## Apache logovi

Stock instalacija Apache-a koristi ***CustomLog*** direktivu da kontroliše logove HTTP zahtjeva. Lokaciju na filesystem-u gdje se logovi pojavljuju od apache2.conf možemo vidjeti sa:

    CustomLog /var/log/apache2/other_vhosts_access.log vhost_combined

Tu se nalaze log unosi. Ali da li tu ostaju zauvijek? Zahvaljujući **log rotaciji**, ne. Log rotacija radi na sljedeći način: Cron task provjerava direktorijum konfiguracionog fajla da vidi koji fajlovi moraju da se rotiraju i onda vrši određene operacije na tim fajlovima. Na primjer, log fajl koji se zvao **secure.log** je preimenovan u **secure.log.1** a novi **secure.log** je otvoren. U sljedećoj rotaciji **secure.log.1** će biti preimenovan u **secure.log.2** i **secure.log** u **secure.log.1** a novi **secure.log** se otvara itd.

Ovo je podrazumijevano Apache log rotacioni konfiguracioni fajl:

**maintenance/default_apache2_logrotate.d**

	/var/log/apache2/*.log {
		weekly
		missingok
		rotate 52
		compress
		delaycompress
		notifempty
		create 640 root adm
		sharedscripts
		postrotate
		if [ -f "`. /etc/apache2/envvars ; \
		echo ${APACHE_PID_FILE:-/var/run/apache2.pid}`" ]; then
		/etc/init.d/apache2 reload > /dev/null
		fi
		endscript
	}

* Izraz za match-ovanje fajla **/var/log/apache2/*.log** govori logrotate-u koji fajlovi trebaju da se rotiraju. U ovom slučaju, svi fajlovi koji završavaju sa .log u /var/log/httpd/ direktorijumu će se rotirati.
* **weekly** direktiva govori logrotate-u da rotira ove fajlove nedjeljno.
* **rotate52** direktiva kaže da će logovi biti rotirani 52 puta prije brisanja.
* **compress** kompresuje logove pomoću ***gzip***.
* **delaycompress** odlaže kompresiju dok se fajl drugi put ne rotira.
* **notifempty** kaže da ne treba rotirati ako je log fajl prazan.
* **create 640 root adm** dodaje dozvole novokreiranim log fajlovima.
* **sharedscripts** uzrokuje da se postrotirajuća skripta pokrene samo jednom iako postoji još mnogo fajlova za rotiranje.
* **postrotate** sadrži blok shell koda koji se izvršava nakon rotiranja log fajlova. Kod uzrokuje restartovanja Apache-a.

**reload** direktiva u **postrotate** predstavlja problem. Reload će restartovati Apache ali i Passenger (?) procese (za app u primjeru u knjizi). Ovaj problem se rješava korišćenjem Apache opcije **piped log rotation**. Apache treba da rotira glavni log bez restartovanja. Za ovo je potrebno dodati nekoliko direktiva našem Apache konfiguracionom fajlu. 
Kopirajmo ***apache2.conf*** u modules/apache/files i izmijenimo CustomLog direktivu tako da je svaki log entry pisan kroz Apache **rotatelogs** i dodajemo interval u sekundama (jednom dnevno, tj. 86,400 sekundi), kada želimo da se log rotira.

    CustomLog "|/usr/sbin/rotatelogs \
    /var/log/apache2/other_vhosts_access.log 86400" vhost_combined

U jednom Puppet file-u podesimo apache2.conf fajl i izbrišimo logrotate.d skriptu, s obrizom da pokrećemo svoju log rotaciju.

**maintenance/modules/apache2/manifests/init.pp**

	class apache2 {
		# other resources
		file {
			"/etc/apache2/apache2.conf": mode => "0644",
			owner => "root",
			group => "root",
			source => "puppet:///modules/apache/apache2.conf",
			notify => Service["apache2"],
			require => Package["apache2"];
			"/etc/logrotate.d/apache2":
			ensure => absent;
		}
		# other resources
	}

Pokretanjem Puppeta primijenićemo izmjeno. Sada se u /var/log/apache2 nalazi novi log fajl sa timestamp-om sa imenovan kao other_vhosts_access.log.1304985600, na primjer. 

Piped log rotation možemo koristiti i kada logovi dostignu određenu veličinu i da promijenimo format imena log fajla. Detaljnije na Apache [vebsajtu](http://httpd.apache.org/docs/2.2/programs/rotatelogs.html
).

## Logovi Rails aplikacije

Rails aplikacije mogu kreirati veću količinu log podataka, tako da će i ovdje biti potrebno naći način za upravljanje log fajlovima. Rails logrotate skripta ne dolazi ni sa jednim standardnim Linux paketima, tako da ćemo pokretati našu skriptu i staviti je na odgovarajuće mjesto, koristeći Puppet.

Rotaciona skripta naše aplikacije će biti slična podrazumijevanoj skripti Apache-a, sa sljedećim razlikama:

* Čuvaćemo samo 10 umjesto 52 posljedna log fajla.
* Dodaćemo kod za restartovanje aplikacije da kompletiramo log rotaciju.

**maintenance/modules/apache2/templates/logrotate_rails.conf.erb**

	<%= current_path %>/log/production.log {
		missingok
		rotate 10
		compress
		delaycompress
		postrotate
			touch <%= current_path %>/tmp/restart.txt
		endscript
	}

U slučaju da promijenimo lokaciju aplikacije, koristićemo Puppet template u modules/massiveapp/templates/massiveapp.logrotate.conf.erb sa sljedećim sadržajem:

**maintenance/modules/apache2/templates/logrotate_rails.conf.erb**

	<%= current_path %>/log/production.log {
		missingok
		rotate 10
		compress
		delaycompress
		postrotate
			touch <%= current_path %>/tmp/restart.txt
	endscript
	}

Sada dodajemo init.pp fajl modula aplikacije, koji će jednostavno postaviti rotacionu log skriptu na pravo mjesto.

**maintenance/only_template/modules/massiveapp/manifests/init.pp**

	class massiveapp {

		$current_path = "/var/massiveapp/current"

		file {
			"/etc/logrotate.d/massiveapp.conf":
			owner => root,
			group => root,
			mode => 755,
			content => template("massiveapp/massiveapp.logrotate.conf.erb")
		}
	}

Najkraći ugrađeni vremenski interval koji logrotate omogućava je ***daily***, tako da se logrotate skripta automatski može pokretati jednom dnevno. Ali pomoću cron joba možemo podesiti da se skripta pokreće i češće (na primjer dva puta dnevno). 

**maintenance/modules/massiveapp/manifests/init.pp**

	class massiveapp {
		$current_path = "/var/massiveapp/current"
		file {
			"/etc/logrotate.d/massiveapp.conf":
			owner => root,
			group => root,
			mode => 755,
			content => template("massiveapp/massiveapp.logrotate.conf.erb")
		}
		cron {
			"massiveapp_logrotate":
			command => "/usr/bin/logrotate -f /etc/logrotate.d/massiveapp.conf",
			user => "vagrant",
			hour => [0,12],
			minute => "0"
		}
	}

