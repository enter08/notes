#1 Zivotni ciklus objekta
>U okviru normalnog rada Rails aplikacije, objeki se mogu kreirati(create), promijeniti(update) i unistiti(destroy). Callback metode omogucavaju pozivanje odredjene logike prije ili poslije izmjene stanja objekta.

#2 Callback pregled
>Callback metode se pozivaju u odredjenim trenucima zivotnog ciklusa objekta. Pomocu callback metoda omoguceno je pisanje koda koji ce se pozvati kad god je neki Active Record objekat kreiran, sacuvan, izmijenjen, izbrisan, ucitan iz baze podataka.

### 2.1 Callback registracija

	class User < ActiveRecord::Base
	  validates :login, :email, presence: true
	 
	  before_validation :ensure_login_has_a_value
	 
	  protected
	  def ensure_login_has_a_value
	    if login.nil?
	      self.login = email unless email.blank?
	    end
	  end
	end


>Metodama je moguce proslijediti blok. Pozeljno je koristiti ovakav pristup ako je   
kod unutar bloka dovoljno kratak da moze stati u jednoj liniji.


	class User < ActiveRecord::Base
	  validates :login, :email, presence: true

	  before_create do |user|
	    user.name = user.login.capitalize if user.name.blank?
	  end
	end

>Callback metode mogu biti registeovane da se pozivaju samo na odredjene dogadjaje  
zivotnog ciklusa.

	class User < ActiveRecord::Base
	  before_validation :normalize_name, on: :create
	 
	  # :on takes an array as well
	  after_validation :set_location, on: [ :create, :update ]
	 
	  protected
	  def normalize_name
	    self.name = self.name.downcase.titleize
	  end
	 
	  def set_location
	    self.location = LocationService.query(self)
	  end
	end

>Smatra se kao *good practice* deklarisati callback metode kao private ili protected.  
Ako se ostave kao private, moguce ih je pozivati izvan modela sto narusava princip enkapsulacije
objekata.

#3 Dostupne Callback metode

###3.1 Create objekta
* before_validation
* after_validation
* before_save
* around_save
* before_create
* around_create
* after_create
* after_save

###3.2 Update 
* before_validation
* after_validation
* before_save
* around_save
* before_update
* around_update
* after_update
* after_save

###3.3 Destroy 
* before_destroy
* around_destroy
* after_destroy

###3.4 after_initialize i after_find

> after_initialize callback metoda se poziva svaki put kada je neki Active Record objekat  
instanciran, bilo da je direktno koriscenjem *new* ili ucitavanjem iz baze.

>after_find callback metoda se poziva svaki put kada Active Record ucitava zapis iz baze  
podataka. after_find se poziva prije after_initialize ako su oba metoda definisana.

	class User < ActiveRecord::Base
	  after_initialize do |user|
	    puts "You have initialized an object!"
	  end
	 
	  after_find do |user|
	    puts "You have found an object!"
	  end
	end
	 
	>> User.new
	You have initialized an object!
	=> #<User id: nil>
	 
	>> User.first
	You have found an object!
	You have initialized an object!
	=> #<User id: 1>

#4 Pozivanje Callback metoda

* create
* create!
* decrement!
* destroy
* destroy!
* destroy_all
* increment!
* save
* save!
* save(validate: false)
* toggle!
* update_attirbute
* update
* update!
* valid?

> after_find callback se aktivira sljedecim finder metodama.

+ all
+ first
+ find
+ find_by
+ find_by_*
+ find_by_*!
+ find_by_sql
+ last

> after_initialize callback se aktivira svaki put kada se inicijalizuje novi objekat klase.

#5 Zaobilazenje Callback metoda

>Kao i u slucaju validcije, moguce je zaoubici callback-ove. Ove metode treba koristiti  
pazljivo, zato sto je moguce da odredjena pravila i logika stoje iza odredjenih callback  
metoda.

* decrement
* decrement_counter
* delete
* delete_all
* increment
* increment_counter
* toggle
* touch
* update_column
* update_columns
* update_all
* update_counters

#6 Halting execution

>Red(queue) za izvrsavanje. Ovaj red ukljucuje sve validacije modela , registrovane  
callback metode, i operacije nad bazom podataka.

>Cijeli callback lanac je uokviren transakcijom. Ako neki *before* callback metod vrati  
**false** ili podigne *exception*, lanac izvrsavanja se zaustavlja(HALT) i vrsi se **ROLLBACK**. *after* callback metodi ovo postizu samo podizanjem exception-a.

#7 Callback-ovi relacija

Callback-ovi fukncionisui preko veza modela i mogu biti na taj nacin definisani.  

		class User < ActiveRecord::Base
		  has_many :posts, dependent: :destroy
		end
		 
		class Post < ActiveRecord::Base
		  after_destroy :log_destroy_action
		 
		  def log_destroy_action
		    puts 'Post destroyed'
		  end
		end
		 
		>> user = User.first
		=> #<User id: 1>
		>> user.posts.create!
		=> #<Post id: 1, user_id: 1>
		>> user.destroy
		Post destroyed
		=> #<User id: 1>

#8 Uslovne Callback metode

>Kao i kod validacija, poziv callback metoda je moguce napraviti tako da zavisi od  
zadovoljavanja odredjenog uslova. Ovo postizemo koriscenjem :if i :unless opcija. 

###8.1 Upotreba :if i :unless sa simbolom

>Opcije :if i :unless mozemo povezati sa simbolom koji odgovara imenu odredjene funckije  
koja se poziva prije callback-a.

	class Order < ActiveRecord::Base
	  before_save :normalize_card_number, if: :paid_with_card?
	end

###8.2 Upotreba :if i :unless sa stringom
	
    class Order < ActiveRecord::Base
      before_save :normalize_card_number, if: "paid_with_card?"
    end

###8.3 Upotreba :if i :unless sa Proc objektom

    class Order < ActiveRecord::Base
      before_save :normalize_card_number,
        if: Proc.new { |order| order.paid_with_card? }
    end	

###8.4 Visestruki uslovi u callback-u

    class Comment < ActiveRecord::Base
      after_create :send_email_to_author, if: :author_wants_emails?,
        unless: Proc.new { |comment| comment.post.ignore_comments? }
    end

#9 Callback klase

>Active Record omogucava kreiranje klasa koje enkapsuliraju callback metode, pa je njihovo  
ponovno koriscenje olaksano.

	 class PictureFileCallbacks
	   def after_destroy( picture_file )
	     if File.exists?( picture_file.filepath )
	       File.delete( picture_file.filepath )
	     end
	   end
	 end



    class PictureFile < ActiveRecord::Base
      after_destroy PictureFileCallbacks.new
    end

>Callback kao metod klase :

    class PictureFileCallbacks
      def self.after_destroy(picture_file)
        if File.exists?(picture_file.filepath)
          File.delete(picture_file.filepath)
        end
      end
    end



    class PictureFile < ActiveRecord::Base
      after_destroy PictureFileCallbacks
    end

#10 Callback transakcija

>Postoje dodatna dva callback metoda koja se pozivaju izvrsenjem transakcije na bazi podataka  
after_commit i after_rollback.

>after_commit i after_rollback callback metode ce biti pozvane za sve create, update i destroy
akcije u okviru bloka transakcije.

    PictureFile.transaction do
      picture_file_1.destroy
      picture_file_2.save!
    end


    class PictureFile < ActiveRecord::Base
      after_commit :delete_picture_file_from_disk, :on => [:destroy]
 
      def delete_picture_file_from_disk
        if File.exist?(filepath)
          File.delete(filepath)
        end
      end
    end    