# Backup

## Pregled

[Dokumentacija](http://meskyanichi.github.io/backup/v3/)

Backup (gem) pruža uredan DSL za kreiranje backup skripti za arhiviranje fajlova i baza podataka kroz skladišta podataka (Storages) kao što su:

* Amazon S3
* Rackspace Cloud Files
* Ninefold
* Dropbox
* FTP
* SFTP
* SCP
* RSync
* Lokalno

Sva navedena skladišta, osim RSync-a, podržavaju:

* **Cycling** (da odredimo broj backup fajlova koji će se čuvati. Kada se dostigne limit, najstariji fajl se briše (kružno čuvanje)).
* **Splitter** (da velike pakete rastavimo na manje djelove)

Backup job-ove kreiramo kao nove modele koristeći Ruby DSL:

    Backup::Model.new(:my_backup, 'Description for my_backup') do
    # ... Model Components ...
    end

**:my_backup** je simbol koji se koristi kao trigger i koristi se da se izvrši ***job***.

    $ backup perform --trigger my_backup

### Arhive i baze podataka

Arhive kreiraju osnovne **tar** arhive. Baze kreiraju backup-ove za jednu od sljedećih baza podataka koje su podržane:

* MySQL
* PostgreSQL
* MongoDB
* Redis
* Riak

Unutar backup modela se može definisati bilo koji broj arhiva i baza podataka.

### Kompresori i Enkriptori

Dovanjem kompresora sve arhive i baze podataka će biti kompresovana unutar jedne arhive. Backup uključuje [Gzip](http://meskyanichi.github.io/backup/v3/compressor-gzip/), [Bzip2](http://meskyanichi.github.io/backup/v3/compressor-bzip2/) i [Custom](http://meskyanichi.github.io/backup/v3/compressor-custom/) kompresore.

### Notifikacije

Notifiers se koriste da šalju notifikacije nakon uspješnog ili neuspješnog backup modela. Podržani servisi notifikacije su:

* Email (SMTP, Sendmail, Exim i File delivery)
* Twitter
* Campfire
* Prowl
* Hipchat
* Pushover
* Nagios
* HTTP POST

Više detalja je dostupno na [ovom](http://meskyanichi.github.io/backup/v3/notifiers/) linku.

## Instalacija

U Gemfile-u dodajte gem 'backup'.

    $ bundle install

Generisanje backup modela:

    $ backup generate:model --trigger ideus_backup \
    --archives --storages='local' --compressors='gzip' --notifiers='mail'

Komande za generisanje modela:

    $ backup help generate:model

[Primjeri](http://meskyanichi.github.io/backup/v3/generator/)

Ovo će generisati novi backup model fajl u ~/Backup/models/ideus_backup.rb koji izgleda ovako:

    Model.new(:ideus_backup, 'Description for ideus_backup') do
  
	    split_into_chunks_of 250

	    archive :my_archive do |archive|
	      archive.add "/path/to/a/file.rb"
	      archive.add "/path/to/a/folder/"
	      archive.exclude "/path/to/a/excluded_file.rb"
	      archive.exclude "/path/to/a/excluded_folder/"
	    end

	    store_with Local do |local|
	      local.path       = "~/backups/"
	      local.keep       = 5
	    end

	    compress_with Gzip

	    notify_by Mail do |mail|
	      mail.on_success           = true
	      mail.on_warning           = true
	      mail.on_failure           = true

	      mail.from                 = "sender@email.com"
	      mail.to                   = "receiver@email.com"
		    mail.address              = "smtp.gmail.com"
		    mail.port                 = 587
		    mail.domain               = "your.host.name"
		    mail.user_name            = "sender@email.com"
		    mail.password             = "my_password"
		    mail.authentication       = "plain"
		    mail.encryption           = :starttls
	    end

    end

Takođe se kreira konfiguracioni fajl config.rb. Kada se pokrene backup, prvo se učita ovaj fajl gdje se mogu vršiti opšta podešavanja.

Za sada preskačemo notifikacije.

Prilagodićemo ideus_backup.rb za Expense Manager app.

    Backup::Model.new(:expense_manager, 'Backup for expenses') do

      database PostgreSQL do |db|
		    db.name               = "expman_development"
		    db.username           = "expman"
		    db.password           = "secret"
		    db.host               = "localhost"
		    db.port               = 5432
		  end

      split_into_chunks_of 250

		  archive :config_archive do |archive|
		  # :config_archive će biti ime tar fajla
		    archive.add "config"
		    archive.exclude "config/deploy"
		    # osim foldera, mogu se dodavati i isključivati fajlovi.
		  end

		  store_with Local do |local|
		    local.path       = "~/backups/"
		    local.keep       = 5
      end

		  compress_with Gzip

		end

Da provjerimo da li postoji neka sintaksna greška:

    $ backup check

### Izvršavanje backup-a

    $ backup perform --triger expense_manager

### Rake task za povratak baze

	namespace :db do
	  task import: :environment do
	    import_path = "~/backups"
	    sql_file = "PostgreSQL.sql"
	    # za sada, sami raspakujte ovaj fajl
	    database_config = Rails.configuration.database_configuration[Rails.env]

	    # Unpack
	    # system "tar xf ides_backup.tar -C #{import_path}"
	    # system "gzip -df #{import_path}/#{sql_file}.gz"

	    # Import
	    system "psql --username=#{database_config['username']} -h localhost #{database_config['database']} < #{import_path}/#{sql_file}"
	  end
	end

## Whenever gem

Whenever je Ruby gem koji omogućava pisanje i izvršavanja cron job-ova (koji se koriste za automatsko izvšavanje taskova u određenim vremenskim periodima).

### Installacija

[Dokumentacija](https://github.com/javan/whenever)

    $ gem install whenever

ili sa Bundlerom u Gemfile-u:

    gem 'whenever', :require => false

Unutar projekta:

    $ cd /apps/my-project
    $ wheneverize .

Ovo će kreirati config/schedule.rb fajl.

Primjeri:

	every 3.hours do
	  runner "MyModel.some_process"
	  rake "my:rake:task"
	  command "/usr/bin/my_great_command"
	end

	every 1.day, :at => '4:30 am' do
	  runner "MyModel.task_to_run_at_four_thirty_in_the_morning"
	end

	every :hour do # Dostupno: :hour, :day, :month, :year, :reboot
	  runner "SomeModel.ladeeda"
	end

	every :sunday, :at => '12pm' do # Ili bilo koji dan u sedmici ili :weekend, :weekday
	  runner "Task.do_something_great"
	end

	every '0 0 27-31 * *' do
	  command "echo 'you can use raw cron syntax too'"
	end

	# pokreni ovaj taks samo na serverima sa :app role u Capistrano-u
	# pogledaj Capistrano roles sekciju u dokmentaciji 
	every :day, :at => '12:20am', :roles => [:app] do
	  rake "app_server:task"
	end

Lista svih opcija se može vidjeti pomoću:

    $ whenever --help

### Backup + Whenever

Nakon instalacije whenever gem, kreiraj folder config.

	$ gem install whenever
	$ cd ~/Backup
	$ mkdir config
	$ wheneverize .

U config/schedule.rb:

	every 1.minute do
		command "cd ~/rails_projects/expman && backup perform --trigger expense_manager"
	end

Na kraju, pokrećemo cron job da radi svaki minut:

    $ whenever
    $ whenever --update-crontab