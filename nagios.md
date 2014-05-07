# Nagios

Nagios® Core™ je Open Source aplikacija za monitoring sistema i mreže. Nadgleda hostove i servise koje odredite, obavještavajući vas kada nešto ne bude u redu ili kada nešto bude ponovo ok. Nagios Core je prvenstveno dizajniran da radi pod Linux-om, ali bi trebao da radi i pod drugim sistemima.

Neka od mnogobrojnih svojstava Nagios Core-a su:

* Monitoring mrežnih servisa (SMTP, POP3, HTTP, NNTP, PING...)
* Monitoring host resursa (processor load, disk usage...)
* Prost plugin dizajn koji omogućava korisnicima lak razvoj svoj check fajlova za servise
* Paralelne provjere servisa
* Mogućnost definisanja host hijerarhije pomoću 'parent' hostova, omogućavajući detekciju i praveći razliku između hostova koji ne rade i koji nisu dostupni.
* Notifikacije kada se pojave ili riješi problemi sa servisom ili hostom (putem mejla, SMS-a)
* Automatska log file rotacija
* Web interface za pregled mrežnog status, notifikacije i istorijat problema, log fajlove itd.

Biće nam potrebna dva servera. Na jednom će se nalaziti aplikacija a na drugom, koji će nadgledati aplikaciju, Nagios.

## Organizacija foldera
|                            |                                                                                                  |
| ---------------------------| -------------------------------------------------------------------------------------------------|  
|/usr/local/nagios| #osnovni direktorijum
|/usr/local/nagios/libexec| #instalirani pluginovi (iz ovog foldera se pokreću provjere servisa)
|/usr/local/nagios/bin| #sadrži osnovne fajlove za pokretanje
|/usr/local/nagios/sbin| #drugi nagios exec. fajlovi
|/usr/local/nagios/etc| #configuracioni fajlove
|/usr/local/nagios/share| #nagios web sajt fajlovi
|/usr/local/nagios/var| #logovi i drugi slični podaci

## Osonovni nagios koncepti

* **Host**: Server, radna stanica, mrežni uređaj... koji se nadgleda.

**Primjer definicije hosta:**

	define host{
	host_name			bogus-router
	alias				Bogus Router #1
	address				192.168.1.254
	parents				server-backbone
	check_command			check-host-alive
	check_interval			5
	retry_interval			1
	max_check_attempts		5
	check_period			24x7
	process_perf_data		0
	retain_nonstatus_information	0
	contact_groups			router-admins
	notification_interval		30
	notification_period		24x7
	notification_options		d,u,r
	}

* **Host group**: grupa sličnih hostova. Npr. mogu se grupisati svi web serveri, fajl serveri itd.

**Primjer definicije grupe hostova:**

	define hostgroup{
		hostgroup_name		novell-servers
		alias			Novell Servers
		members			netware1,netware2,netware3,netware4
		}

* **Service**: servis koji se nadgleda na hostu, kao HTTP, DNS, NFS itd.

**Primjer definicije servisa:**

	define service{
		host_name		linux-server
		service_description	check-disk-sda1
		check_command		check-disk!/dev/sda1
		max_check_attempts	5
		check_interval	5
		retry_interval	3
		check_period		24x7
		notification_interval	30
		notification_period	24x7
		notification_options	w,c,r
		contact_groups		linux-admins
		}

* **Servise Group**: omogućava grupisanje više servisa. Ovo je korisno za grupisanje više HTTP npr.

**Primjer definicije grupe servisa:**

	define servicegroup{
		servicegroup_name	dbservices
		alias			Database Services
		members			ms1,SQL Server,ms1,SQL Server Agent,ms1,SQL DTC
		}

* **Contact**: osoba kojoj se šalju notifikacije kada se desi ili riješi neki problem.

**Primjer definicije kontakta:**

	define contact{
		contact_name                    jdoe
		alias                           John Doe
		host_notifications_enabled		1
		service_notifications_enabled	1
		service_notification_period     24x7
		host_notification_period        24x7
		service_notification_options    w,u,c,r
		host_notification_options       d,u,r
		service_notification_commands   notify-by-email
		host_notification_commands      host-notify-by-email
		email			jdoe@localhost.localdomain
		pager			555-5555@pagergateway.localhost.localdomain
		address1			xxxxx.xyyy@icq.com
		address2			555-555-5555
		can_submit_commands	1
		}

## Instalacija

Instalirajmo Nagios Core 4 na lokalnom računaru.

Instalacija potrebnih paketa:

    $ sudo apt-get update
    $ sudo apt-get install build-essential apache2 php5-gd wget libgd2-xpm libgd2-xpm-dev libapache2-mod-php5 sendmail daemon

 Kreirajmo sada nagios korisnika i nagcmd grupu (u koju dodajemo nagios korisnika):

     $ sudo useradd nagios
     $ sudo groupadd nagcmd
     $ sudo usermod -a -G nagcmd nagios

Download Nagios Core 4.2 tar fajla:

    $ sudo wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-4.0.2.tar.gz
    $ sudo tar -xvzf nagios-4.0.2.tar.gz

Pokrenimo sada configure sa dodatnim opcijama:

    $ cd nagios-4.0.2/
    $ sudo ./configure --with-nagios-group=nagios --with-command-group=nagcmd --with-mail=/usr/bin/sendmail

    $ sudo make all
    $ sudo make install
    $ sudo make install-init
    $ sudo make install-config
    $ sudo make install-commandmode
    $ sudo make install-webconf

Kopiramo eventhandlere i dodijelimo privilegije:

    $ sudo cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
    $ sudo chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers

Kreirajmo Nagios korisnika za pristup internetu:

	$ sudo htpasswd -cm /usr/local/nagios/etc/htpasswd.users nagiosadmin

Ako sa pokušamo da pokrenemo Nagios, dobićemo sljedeću grešku:

	$ sudo /etc/init.d/nagios start
	/etc/init.d/nagios: 20: .: Can't open /etc/rc.d/init.d/functions

Postoji problem sa Nagios init skriptom na Ubuntuu. Moramo izmijeniti init skriptu i promijeniti par linija. 

Kopirajte sljedeće u terminal:

	sudo sed -i "s/^\.\ \/etc\/rc.d\/init.d\/functions$/\.\ \/lib\/lsb\/init-functions/g" /etc/init.d/nagios
	sudo sed -i "s/status\ /status_of_proc\ /g" /etc/init.d/nagios
	sudo sed -i "s/daemon\ --user=\$user\ \$exec\ -ud\ \$config/daemon\ --user=\$user\ --\ \$exec\ -d\ \$config/g" /etc/init.d/nagios
	sudo sed -i "s/\/var\/lock\/subsys\/\$prog/\/var\/lock\/\$prog/g" /etc/init.d/nagios
	sudo sed -i "s/\/sbin\/service\ nagios\ configtest/\/usr\/sbin\/service\ nagios\ configtest/g" /etc/init.d/nagios
	sudo sed -i "s/\"\ \=\=\ \"/\"\ \=\ \"/g" /etc/init.d/nagios
	sudo sed -i "s/\#\#killproc\ \-p\ \${pidfile\}\ \-d\ 10/killproc\ \-p \${pidfile\}/g" /etc/init.d/nagios
	sudo sed -i "s/runuser/su/g" /etc/init.d/nagios
	sudo sed -i "s/use_precached_objects=\"false\"/&\ndaemonpid=\$(pidof daemon)/" /etc/init.d/nagios
	sudo sed -i "s/killproc\ -p\ \${pidfile}\ -d\ 10\ \$exec/\/sbin\/start-stop-daemon\ --user=\$user\ \$exec\ --stop/g" /etc/init.d/nagios
	sudo sed -i "s/\/sbin\/start-stop-daemon\ --user=\$user\ \$exec\ --stop/&\n\tkill -9 \$daemonpid/" /etc/init.d/nagios

sada pokušajte ponovo da pokrenete Nagios.

Instalirajmo sada Nagios pluginove:

    $ sudo wget http://www.nagios-plugins.org/download/nagios-plugins-1.5.tar.gz
    $ sudo tar -xvzf nagios-plugins-1.5.tar.gz
    $ cd nagios-plugins-1.5/
    $ sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagios
    $ sudo make
    $ sudo make install

Kopirajmo i linkujmo Nagios Apache konfiguraciju:

	$ sudo cp /etc/apache2/conf.d/nagios.conf /etc/apache2/sites-available/nagios
	$ sudo ln -s /etc/apache2/sites-available/nagios /etc/apache2/sites-enabled/nagios

Sada moramo promijeniti privilegije za /usr/local/nagios/var/rw/

	$ sudo chown -R nagios:www-data /usr/local/nagios/var/rw/

Provjerimo sada da li ima grešaka u Nagios konfiguracijama:

	$ sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Podesimo da se Nagios i Apache pokreću pri startovanju sistema:

	$ sudo ln -s /etc/init.d/nagios /etc/rcS.d/S98nagios
	$ sudo ln -s /etc/init.d/apache2 /etc/rcS.d/S99apache2

Sada možemo da pokrenemo Nagios:

	$ sudo service nagios start

kao i Apache servis:

	$ sudo service apache2 start

Sada možemo otvoriti nagios u browseru sa **http://IPAdresa/nagios** i unesite **username** i **password** koji se kreirali.

**NRPE**

Nagios Remote Plug-in Executor (NRPE) je Nagios dodatak koji omogućava 'lokalne' provjere na udaljenom hostu (tj. provjere svih servisa će biti vršene na serveru gdje se nalazi Nagios). NRPE moramo instalirati na oba servera.

Instalacija nrpe plugin-a lokalno (ili na serveru gdje se nalazi Nagios):

	$ mkdir -p /usr/local/src/nrpe
	$ cd /usr/local/src/nrpe
	$ wget http://kent.dl.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz
	$ tar -xf nrpe-2.15.tar.gz
	$ cd nrpe-2.15
	$ ./configure --with-ssl=/usr/bin/openssl--with-ssl-lib=/usr/lib/x86_64-linux-gnu

Ako naiđete na grešku **configure: error: Cannot find ssl libraries** instalirajte sljedeće:

	$ sudo apt-get install libssl-dev
	$ dpkg -L libssl-dev
	$ ln -s /usr/lib/x86_64-linux-gnu/libssl.so /usr/lib/libssl.so

Sada pokušajte <code>configure</code> komandu.

Ako nije uspjelo, pokušajte sa drugim rješenjem sa [ovog linka](http://ubuntuforums.org/showthread.php?t=1975917).

NRPE pokrećemo uz pomoć xinetd:
	
	$ sudo apt-get install xinetd

	$ sudo make all

Ako ne postoji <code>make</code>:

	$ sudo apt-get install make 

	$ sudo make install
	$ sudo make install-daemon
	$ sudo make install-daemon-config
	$ sudo make install-xinetd

	$ sudo chown nagios.nagios /usr/local/nagios
	$ sudo chown -R nagios.nagios /usr/local/nagios/libexec

	/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d

	Izmijenimo /etc/xinetd.d/nrpe fajl. U only_from dodajemo IP adresu nagios servera. (samo razmak izmedju IP adresa)

Sada možemo provjeriti da li NRPE radi na našem računaru:

	$ netstat -at | grep 5666
	$ /usr/local/nagios/libexec/check_nrpe -H 127.0.0.1

NRPE daemon se vezuje za port 5666. Izmijenimo iptables filter da prihvati konekciju na naš nagios server.

	$ sudo iptables -A INPUT -s 188.226.192.11 -p tcp --dport 5666 -j ACCEPT

Instalirajmo sada NRPE na serveru.
	
	$ sudo apt-get update
	$ sudo apt-get install nagios-nrpe-server
	$ sudo apt-get install nagios-nrpe-plugin

Izmijenimo <code>allowed_hosts</code> u nrpe.cfg i dodajmo IP adresu servera i našeg računara.

*Your IP*: http://www.whatismyip.com/

Restartujmo nrpe servis:

	$ sudo service nagios-nrpe-server restart

i provjerimo da li nrpe radi sa:

	$ sudo /usr/lib/nagios/plugins/check_nrpe -H 127.0.0.1
	$ sudo /usr/lib/nagios/plugins/check_nrpe -H 107.170.116.145

Vratimo se na lokalna podešavanja. Otvorimo <code>/usr/local/nagios/etc/objects</code>. Ovjde se nalaze podešavanja za komande, hostove, kontakte i sl. Kopirajmo localhost.cfg fajl u server.cfg:

	$ sudo cp localhost.cfg server.cfg

Izmijenimo <code>server.cfg</code> fajl tako da ima sljedeće:

	define host{
	use linux-server
	host_name server
	alias server
	address 107.170.116.145
	}

	define hostgroup{
	hostgroup_name servers
	alias Servers
	members server
	}

i definišimo jedan servis za početak:

	define service{
	use generic-service
	host_name server
	service_description Current Users
	check_command check_remote_users
	}

S obzirom da komanda <code>check_remote_users</code> ne postoji, moraćemo je sami kreirati. Izmijenimo <code>commands.cfg</code> fajl iz objects direktorijuma dodajući na dnu fajla:

	define command{
	command_name check_remote_users
	command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_users
	}

U nagios.cfg fajl uključujemo novi server:

	cfg_file=/usr/local/nagios/etc/objects/server.cfg


Restartujmo Nagios:

	$ sudo service nagios restart

i otvorimo nagios u browseru:

	127.0.0.1/nagios

Ako dodajemo novi plugin, kopirajmo plugin-fajl u <code>libexec</code> folder i promijenimo dozovle:

	chown nagios:nagios plugin-name
	chmod 751 plugin-name

### Postgres

server.cfg

	define service{
	        use generic-service
	        host_name server
	        service_description Postgresql
	        check_command check_postgresql
	}

commands.cfg

	define command{
	        command_name check_postgresql
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_pgsql
	}

On server: (in /etc/nagios/nrpe.cfg)

	command[check_pgsql]=/usr/lib/nagios/plugins/check_pgsql -H localhost -d expman_production -l expman -p secret

### Elasticsearch cluser health

Download:
http://exchange.nagios.org/directory/Addons/Active-Checks/Elasticsearch-cluster-status/details

On server:

Install elasticsearch server.
Install elasticsearch gem.

nrpe.cfg:

	command[check_elasticsearch_cluser]=/usr/lib/nagios/plugins/check_elasticsearch_cluser.rb --ip 107.170.106.199 --port 9200

	$ sudo service nagios-nrpe-server restart

	define service{
	  use generic-service
	  host_name server
	  service_description ElasticSearch Cluster Health
	  check_command check_elasticsearch_cluser_health
	}

	define command{
    command_name check_elasticsearch_cluster_health
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_elasticsearch_cluster
	}

	$ sudo service nagios restart

### Unicorn

Download:

* [link1](https://github.com/scalp42/check_unicorn/blob/master/check_unicorn)
* [link2](https://github.com/scalp42/check_unicorn/blob/master/check_unicorn_processes)

nrpe.conf
	
	command[check_unicorn]=/usr/lib/nagios/plugins/check_unicorn 2
	command[check_unicorn_processes]=/usr/lib/nagios/plugins/check_unicorn_processes 2

	define service{
      use generic-service
      host_name server
      service_description Unicorn
      check_command check_unicorn
	}

	define service{
      use generic-service
      host_name server
      service_description Unicorn processes
      check_command check_unicorn_processes
	}

	define command{
    command_name check_unicorn
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_unicorn
	}

	define command{
    command_name check_unicorn_processes
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_unicorn_processes
	}

### Nginx

Download:
https://github.com/cloved/check_nginx_status/blob/master/check_nginx_status.sh

Enable the nginx_status page:

	$ nginx -V 2>&1 | xargs -n1 | grep status

should return **--with-http_stub_status_module**

Add:

	location /nginx_status {
    stub_status on;
    access_log   off;
    allow MY_IP;
    deny all;
    }

in the app file in sites-enabled (nginx).

Restart nginx and visit IP/nginx_status.

nginx.conf:

    command[check_unicorn]=sudo /usr/lib/nagios/plugins/check_nginx_status IP 80 nginx_status

change /etc/sudoers:

	%nagios ALL=(ALL) NOPASSWD:ALL

nagios side:

		define service{
	      use generic-service
	      host_name server
	      service_description Nginx status
	      check_command check_nginx_status
		}

		define command{
	    command_name check_nginx_status
	    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_nginx_status
		}

### Disk

nginx.conf:

    command[check_disk]=/usr/lib/nagios/plugins/check_disk -w 10% -c 5% -p /


### Advanced Postgres checks

Download:
http://bucardo.org/wiki/Check_postgres

**Connection**

	command[check_pg_connection]=/usr/lib/nagios/plugins/check_postgres.pl --action=connection --dbuser=expman --db=expman_production --dbpass=secret --host=localhost

**Bloat**
Checks the amount of bloat in tables and indexes. (Bloat is generally the amount of dead unused space taken up in a table or index. This space is usually reclaimed by use of the VACUUM command.)

	command[check_bloat]=/usr/lib/nagios/plugins/check_postgres.pl --action=bloat --dbuser=expman --db=expman_production --dbpass=secret --host=localhost --warning='100 M' --critical='200 M'

**Backends**

Checks the current number of connections for one or more databases, and optionally compares it to the maximum allowed, which is determined by the Postgres configuration variable max_connections.

	command[check_backends]=/usr/lib/nagios/plugins/check_postgres.pl --action=backends --dbuser=expman --db=expman_production --dbpass=secret --host=localhost --warning='150' --critical='150'

**database_size**
Checks the size of all databases and complains when they are too big. There is no need to run this command more than once per database cluster.

	command[check_db_size]=/usr/lib/nagios/plugins/check_postgres.pl --action=database_size --dbuser=expman --db=expman_production --dbpass=secret --host=localhost --warning='450 GB' --critical='500 GB'

**disk_space**
Checks on the available physical disk space used by Postgres. This action requires that you have the executable "/bin/df" available to report on disk sizes, and it also needs to be run as a superuser, so it can examine the data_directory setting inside of Postgres.

*Password for superuser postgres?*

**query_time**
Checks the length of running queries on one or more databases.

	command[check_query_time]=/usr/lib/nagios/plugins/check_postgres.pl --action=query_time --dbuser=expman --db=expman_production --dbpass=secret --host=localhost --warning=20s --critical=40s --port=5432

###Default plugins

https://nagios-plugins.org/doc/man/index.html

**Check SSH**
Try to connect to an SSH server at specified server and port.

**Check ping**
Use ping to check connection statistics for a remote host.

**Check HTTP**
This plugin tests the HTTP service on the specified host. It can test normal (http) and secure (https) servers, follow redirects, search for strings and regular expressions, check connection times, and report on certificate expiration times.

commands.cfg:

	define command{
	        command_name check_ssh_nrpe
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_ssh_nrpe
	}

	define command{
	        command_name check_http_nrpe
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_http_nrpe
	}

	define command{
	        command_name check_ping_nrpe
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_ping_nrpe
	}

	define command{
	        command_name check_procs_nrpe
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_procs_nrpe
	}

	define command{
	        command_name check_load_nrpe
	        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_load_nrpe
	}

server.cfg:

	define service{
	        use                             generic-service
	        host_name                       server
	        service_description             SSH
	        check_command                   check_ssh_nrpe
	        }

	define service{
	        use                     generic-service
	        host_name               server
	        service_description     HTTP
	        check_command           check_http_nrpe
	}

	define service{
	        use                     generic-service
	        host_name               server
	        service_description     PING
	        check_command           check_ping_nrpe
	}


	define service{
	        use                     generic-service
	        host_name               server
	        service_description     Current load
	        check_command           check_load_nrpe
	}

	define service{
	        use                     generic-service
	        host_name               server
	        service_description     Local procs
	        check_command           check_procs_nrpe
	}

nrpe.cfg:

	command[check_load_nrpe]=/usr/lib/nagios/plugins/check_load -w 5.0,4.0,3.0 -c 10.0,6.0,4.0
	command[check_procs_nrpe]=/usr/lib/nagios/plugins/check_procs -w 10 -c 20 --metric CPU
	command[check_http_nrpe]=/usr/lib/nagios/plugins/check_http -H IP
	command[check_ssh_nrpe]=/usr/lib/nagios/plugins/check_ssh IP
	command[check_ping_nrpe]=/usr/lib/nagios/plugins/check_ping -H IP -w 100.0,20% -c 500.0,60%

## Notifikacije

Definisanje generic-contact:

**templates.cfg**

	define contact{
        name                            generic-contact
        service_notification_period     24x7
        host_notification_period        24x7
        service_notification_options    w,u,c,r,f,s
        host_notification_options       d,u,r,f,s
        service_notification_commands   notify-service-by-email
        host_notification_commands      notify-host-by-email
        register                        0
	}

**contacts.cfg**

	define contact{
	        contact_name                    sgupta
	        use                             generic-contact
	        alias                           Sanjay Gupta (Developer)
	        email                           sgupta@thegeekstuff.com
	        pager                           333-333@pager.thegeekstuff.com
	        }
	define contact{
	        contact_name                    jbourne
	        use                             generic-contact
	        alias                           Jason Bourne (Sysadmin)
	        email                           jbourne@thegeekstuff.com
	        }

	define contactgroup{
	contactgroup_name          db-admins
	alias                      Database Administrators
	members                    jsmith, jdoe, mraj
	}

	define contactgroup{
	contactgroup_name          unix-admins
	alias                      Linux System Administrator
	members                    jbourne, dpatel, mshankar
	}

**localhost.cfg**

	define host{
	use                     linux-server
	host_name               email-server
	alias                   Corporate Email Server
	address                 192.168.1.14
	contact_groups          unix-admins
	}

	define service{
	use                             generic-service
	host_name                       prod-db
	service_description             CPU Load
	contact_groups                  unix-admins
	check_command                   check_nrpe!check_load
	}

http://linuxg.net/how-to-send-emails-via-terminal/

	$ sudo apt-get install msmtp
	$ sudo apt-get install heirloom-mailx

Kreirajte fajl <code>~/.msmtprc</code>.

host smtp.gmail.com

port 587

protocol smtp

auth on

from Demic Redzep

user demicredzep1990@gmail.com

password yourpassword

tls on

tls_nocertcheck

	$ chmod 600 .msmtprc

test:

	echo "Hello" | mail contact@linuxg.net