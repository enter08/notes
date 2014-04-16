# Obezbjeđivanje sshd

Novi host koji je dostupan putem Interneta može biti laka meta za hakere. Jedna od prvih stvari koje moramo uraditi, da bi se sačuvali od ***password-guessing*** ili ***dictionary*** napada, kada pokrenemo server, jeste onemogućavanja ssh password autentikacije i zahtijevanje autentikacije javnog ključa.

Na netu postoji mnogo opisa generisanja para ključeva i uplodovanje javnog ključa. **Hint**: ***Google***. Kada javni ključ na serveru bude na mjestu, gasimo autentikaciju passworda izmjenom /etc/ssh/sshd_config, gdje mijenjamo:

    #PasswordAuthentication yes

u
	
	PasswordAuthentication no

I javljamo sshd-u da ponovo učita konfiguracioni fajlu sa:

    $ sudo /sbin/service reload sshd

Da bismo bili sigurni, otvorićemo novi prozor u terminalu i uspostaviti novu ssh konekciju na server. Na taj način, znamo da će stvari raditi prije logouta.

Uz pomoć Puppet manifesta konfigurišemo package-file-service vezu:

**topics/modules/sshd/manifests/init.pp**

	class ssh {
		package {
			"ssh":
				ensure => present,
				before => File["/etc/ssh/sshd_config"]
			}
			file {
				"/etc/ssh/sshd_config":
				owner => root,
			group => root,
			mode => 644,
			source => "puppet:///modules/ssh/sshd_config"
			}
			service {
			"ssh":	ensure => true,
			enable => true,
			subscribe => File["/etc/ssh/sshd_config"]
		}
	}

## Rješenje #2

[Izvor](http://serverfault.com/questions/312500/how-do-i-configure-sshd-on-debian-to-use-public-key-authentication)

 Izmijenimo /etc/ssh/sshd_config fajl da budeo kao:

	# Both of these are probably already there, but commented
	PubkeyAuthentication yes
	# The next line makes sure that sshd will look in 
	# $HOME/.ssh/authorized_keys for public keys
	AuthorizedKeysFile      %h/.ssh/authorized_keys
	# Again, this rule is already there, but usually defaults to 'yes'
	PasswordAuthentication no

Kreiranje para ključeva: [How to setup ssh keys](https://www.digitalocean.com/community/articles/how-to-set-up-ssh-keys--2)

Nakon toga, **restart ssh** sa:

    /etc/init.d/sshd restart

Prije svega navedenog morao je biti kreiran .ssh direktorijum sa odgovarajućom dozvolom. (chmod 0700 za ~/.ssh a i chmod 0600 za authorized_keys)

Onemogućavanje SSH password autentikacije za određene korisnike ili Linux grupe:

[Xmodulo članak](http://xmodulo.com/2013/03/how-to-force-ssh-login-via-public-key-authentication.html)