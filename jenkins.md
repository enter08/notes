# Jenkins

Jenkins je open source [continous integration](https://en.wikipedia.org/wiki/Continuous_integration) Java alat. Jenkins pruža CI servise za razvoj softvera.

## Instalacija

Dokumentacija:

* [Link 1](http://blog.loadimpact.com/2013/11/29/bootstrap-your-ci-with-jenkins-and-github)
* [Link 2](http://dean.io/setting-up-jenkins-ci-for-ruby-on-rails-testing/)
* [Link 3](http://blogs.burnsidedigital.com/2013/01/setting-jenkins-ci-server-for-rails-project-on-a-vagrant-box/)


Dodavanje ključa repozitorijuma:

    $ wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -

Rješenje za mogući problem ***gpg: no valid opengpg data found occurs***, potražite [ovdje](http://stackoverflow.com/questions/18967942/gpg-import-fails-with-no-valid-openpgp-data-found) ili [ovdje](http://stackoverflow.com/questions/21338721/gpg-no-valid-openpgp-data-found).

Sada treba dodati repozitorijum, source listi:

    $ /etc/apt/sources.list
    $ deb http://pkg.jenkins-ci.org/debian binary/

Ako na serveru, ova dva koraka ne rade, onda:

    $ sudo -i
    echo 'deb http://pkg.jenkins-ci.org/debian binary/' >> /etc/apt/sources.list
    exit

ili:

    $ echo 'deb http://pkg.jenkins-ci.org/debian binary/' | sudo tee -a /etc/apt/sources.list

Sada treba update-ovati listu paketa:

    $ sudo apt-get update

i instalirati Jenkins:

    $ sudo apt-get install jenkins

Sada bi Jenkins trebao biti dostupan na localhost:8080 ili IP:8080 na nekom serveru.

Na serveru će nam biti potrebno da Jenkins bude dostupan na portu 80 kako bismo na kraju ispravno povezali GitHub i Jenkins.

Instalirajmo najprije nginx:

    $ sudo aptitude -y install nginx

Izbišrimo default podešavanja u sites-enables:

    $ cd /etc/nginx/sites-available
    $ sudo rm default ../sites-enabled/default

U sites-available kreiramo jenkins file:

	upstream app_server {
	    server 127.0.0.1:8080 fail_timeout=0;
	}

	server {
	    listen 80;
	    listen [::]:80 default ipv6only=on;
	    server_name ci.yourcompany.com; //(instead the server_name, add your IP - remove this comment!)

	    location / {
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_set_header Host $http_host;
	        proxy_redirect off;

	        if (!-f $request_filename) {
	            proxy_pass http://app_server;
	            break;
	        }
	    }
	}

Sada linkujemo ovaj konfiguracioni fajl iz site-available u sites-enable:

	  $ sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/
	  $ sudo service nginx restart

Sada bi jenkins trebao biti dostupan na IP:80 (ili samo IP) a da se omogući pristup jenkinsu sa IP/jenkins umjesto location / u jenkins configuracionom fajlu, pišemo: location ^~ /jenkins.

Ako naiđete na problem ***It appears that your reverse proxy setup is broken***, probajte sljedeće:

* [link1](http://www.phase2technology.com/blog/running-jenkins-behind-nginx/)
* [link2](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+says+my+reverse+proxy+setup+is+broken)

ili u jenkins conf fajlu dodajte:

    proxy_pass          http://localhost:8080;
    proxy_read_timeout  90;
    proxy_redirect      http://localhost:8080 $scheme://(IP or domain);

    $ service nginx restart
    $ service jenkins restart

## Instalacija plugin-ova

Pluginovim se može pristupiti unutar Manage Jenkins u side baru, zatim Manage plugins pod karticom Available plugins.

Instalirajte sljedeće pluginove:

* Git
* GitHub
* Rake
* RVM
* Ruby
* Ruby metrics
* rbenv
* Green ball
* ANSI color

Kliknite na **install without restart** i čekirajte ***restart after installation*** ispod instalacije pluginova.

## Sigurnost

* U Jenkins > Manage Jenkins > Configure Global Security čekirajte ***Enable security***.
	* Pod **Security Realm**, odaberite ***Jenkins's own user database*** i čekirajte ***Allow users to sign up***
	* Pod **Authorization**, odaberite ***Project-based Matrix Authorization Strategy***. S obzirom da korisničko ime za administratorski pristup mora biti **admin**, dodajte prvog korisnika sa ***admin*** i čekirajte sve dozvole. Dodajte drugog korisnika sa ***github*** i čekirajte samo **Read** čekboks u Overall koloni.

**Napomena**: Prethodni koraci ne kreiraju admin i github korisnike tako da ih moramo kreirati ručno. Klikom na Save, pojaviće se prozor za login.

## Dodavanje SSH ključa

Ponovite korake za dodavanje SSH ključa sa [GitHub help stranice](https://help.github.com/articles/generating-ssh-keys) ali promijenite ime fajla da ne bi izbrisali stari ključ (npr. id_rsa2).
Dodajte ključ na GitHubu na ImeProjekta > Settings > Deploy key

Isti ključ dodajemo i na jenkinsu:

Jenkins > Credentials > Global Credentials > Add Credentials
**Kind**: SSH Username with private key

Sada možemo kreirati prvi projekat. Na glavnoj stranici biramo opciju New Item. Dodajte neko ime i odaberite **Build a free software project**. 

Na sljedećoj stranici dodajte link GitHub projekta. Npr:

    https://github.com/enter08/Expense-Manager/

Pod Source Code Management odaberite Git i dodajte link repozitorijuma. Npr:

    https://github.com/enter08/Expense-Manager.git

Iz drop down liste Credentials, odaberite nedavno kreirani ključ.

Da bi trigger **Build when a change is pushed to GitHub** funkcionisao, pročitajte sljedeće (ne može lokalno):
* [Trigger jenkins builds](http://fourkitchens.com/blog/2011/09/20/trigger-jenkins-builds-pushing-github)
* [How to configure git post commit hook](http://stackoverflow.com/questions/12794568/how-to-configure-git-post-commit-hook)

Da testirate do sada urađeno, napravite neku izmjenu na projektu i push-ujte to na GitHub. Sačuvajte sve u konfiguracijama jenkinsa i kliknite na **Build now**.

U Build history ćete vidjeti broj job-a i datum. Klikom na link dobićemo rezultate build-a. Na trenutnoj stranici bi trebalo da piše koja promjena je napravljena na projektu. Inače, u Build history svaki uspješni build je označen zelenom loptom, neuspješni crvenom.

Vratimo se podešavanjima. Kliknite na **Configure**.

Pod **Build Environment** čekirajte **rbenv build wrapper**.
Unesite i svoju ruby verziju. Npr. 2.0.0-p353.

U database.yml.example dodajte informacije testne baze podataka (kopirajte iz database.yml).

Pod Build > Execute Shell, dodajte sljedeće linije:

	cp config/database.yml.example config/database.yml
	bundle install
	rake db:create
	rake db:schema:load
	rake db:test:prepare
	rake ci:setup:rspec spec

Da bi ci:setup:rspec (continous integration) radio, potreban nam je ci_reporter gem.
Dodajte gem '[ci_reporter](https://github.com/nicksieger/ci_reporter)', '1.8.0' unutar test grupe. 
(**Napomena**: Navedite tačnu verziju, jer posljednja verzija 1.8.3, ne radi ispravno).

U Rakefile-u dodajte:

	require 'ci/reporter/rake/rspec' 
	require 'ci/reporter/rake/cucumber'

Trebaće nam i simple_cov gem koji će prikazivati Rcov coverage, tj. koliko je koda pokriveno testovima.

Dodajte gemove:

	gem 'simplecov-rcov'
	gem 'simplecov'

U spec/spec_helper.rb, prije Rspec:configure bloka, treba dodati:

	require 'simplecov'
	require 'simplecov-rcov'
	SimpleCov.formatter = SimpleCov::Formatter::RcovFormatter
	SimpleCov.start 'rails'

Push-ovati ovo na GitHub i sačuvati jenkins podešavanja. Sa Build now pokrećemo ***job***. Otvorimo job koji se izvršava u Build History. Pod Console Output možemo pratiti izvršavanje.

Korisni linkovi:

 * http://gistflow.com/posts/492-jenkins-ci-setup-for-rails-application-from-scratch
 * http://blogs.burnsidedigital.com/2013/01/setting-jenkins-ci-server-for-rails-project-on-a-vagrant-box/
 * http://www.webascender.com/Blog/ID/522/Setting-up-Jenkins-for-GitHub-Rails-Rspec#.U0Y-i99qNPZ
 * http://artsy.github.io/blog/2012/05/27/using-jenkins-for-ruby-and-ruby-on-rails-teams/
 * http://danmcclain.net/blog/2011/11/22/using-jenkins-with-rails/