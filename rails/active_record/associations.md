# Active Record Associations


- Asocijacija predstavlja vezu između dva modela. 
- Rails podržava šest vrsta asocijacija:
    1. belongs_to (1:1)

![belongs_to](http://guides.rubyonrails.org/images/belongs_to.png "belongs_to")
		
    2. has_one (1:1)

![has_one](http://guides.rubyonrails.org/images/has_one.png "has_one")

    3. has_many (1:više)

![has_many](http://guides.rubyonrails.org/images/has_many.png "has_many")

Fajlovi migracije za has_one i has_many izgledaju potpuno isto.

    4. has_many :through (više:više)

![has_many_through](http://guides.rubyonrails.org/images/has_many_through.png "has_many_through")

Fajl migracije je specifican:

```
class CreateAppointments < ActiveRecord::Migration
  def change
    create_table :physicians do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :patients do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :appointments do |t|
      t.belongs_to :physician
      t.belongs_to :patient
      t.datetime :appointment_date
      t.timestamps
    end
  end
end
```
Interesantna je ovakva upotreba has_many_through veze:
```
class Document < ActiveRecord::Base
  has_many :sections
  has_many :paragraphs, through: :sections
end
 
class Section < ActiveRecord::Base
  belongs_to :document
  has_many :paragraphs
end
 
class Paragraph < ActiveRecord::Base
  belongs_to :section
end
```
Koriste se ugniježdene has_many veze. Tada će Rails dovhatiti @document.paragraphs

    5. has_one :through (1:1)

![has_one_through](http://guides.rubyonrails.org/images/has_one_through.png "has_one_through")

    6. has_and_belongs_to_many (više:više) - neposredno

![has_and_belongs_to_many](http://guides.rubyonrails.org/images/habtm.png "has_and_belongs_to_many")

Migracija:
```
class CreateAssembliesAndParts < ActiveRecord::Migration
  def change
    create_table :assemblies do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end
 
    create_table :assemblies_parts do |t|
      t.belongs_to :assembly
      t.belongs_to :part
    end
  end
end
```
### has_one ? belongs_to
Da bi se kreirala 1:1 veza izemđu dva modela, treba jednom dodati has_one, a drugom belongs_to. Kome dodajemo šta? Zavisi od toga gdje 
želimo da se kreira spoljni ključ. 

```
class Supplier < ActiveRecord::Base
  has_one :account
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier
end
```
U ovom slučaju bi model Account imao spoljni ključ na Supplier-a. 

### has_many_through ? has_and_belongs_to_many
Ovo su dva načina kreiranja veze više:više, s tim što je prvi posredan, a drugi neposredan. Koji koristiti? 
Ako postoji potreba za validacijom ili posebnim atributima modela posrednika, onda koristiti has_many_through, inače nema potrebe. 

### Polimorfne asocijacije

![polymorphic](http://guides.rubyonrails.org/images/polymorphic.png "polymorphic")

Migracija: 

```
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string  :name
      t.integer :imageable_id
      t.string  :imageable_type
      t.timestamps
    end
  end
end
```
Pojednostavljeno:
```
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string :name
      t.references :imageable, polymorphic: true
      t.timestamps
    end
  end
end
```
###Self Join
```
class Employee < ActiveRecord::Base
  has_many :subordinates, class_name: "Employee",
                          foreign_key: "manager_id"
 
  belongs_to :manager, class_name: "Employee"
end
```

### Kontrola keširanja
```
customer.orders                 # vraća order-e iz baze 
customer.orders.size            # koristi tu keširanu kopiju 
customer.orders(true).empty?    # otarasi se keširane kopije i koristi stvarne podatke  
```

### Kreiranje join tabele kod has_and_belongs_to_many
```
class CreateAssembliesPartsJoinTable < ActiveRecord::Migration
  def change
    create_table :assemblies_parts, id: false do |t|
      t.integer :assembly_id
      t.integer :part_id
    end
  end
end
```
### Kontrola asocijacija
```
module MyApplication
  module Business
    class Supplier < ActiveRecord::Base
       has_one :account,
        class_name: "MyApplication::Billing::Account"
    end
  end
 
  module Billing
    class Account < ActiveRecord::Base
       belongs_to :supplier,
        class_name: "MyApplication::Business::Supplier"
    end
  end
end
```
Mora se navesti čitava putanja, ako su modeli u različitim namespace-ovima.

### Dvosmjerne asocijacije
```
class Customer < ActiveRecord::Base
  has_many :orders, inverse_of: :customer
end
 
class Order < ActiveRecord::Base
  belongs_to :customer, inverse_of: :orders
end
```
```
c = Customer.first
o = c.orders.first
c.first_name == o.customer.first_name # => true
c.first_name = 'Manny'
c.first_name == o.customer.first_name # => true, inače false
```    
inverse_of ne radi sa through, as i polymorfic.                       
### Metodi koji belongs_to dodaje

```
class Order < ActiveRecord::Base
  belongs_to :customer
end
```
- customer         
 @customer = @order.customer
- customer=      
@order.customer = @customer    
- build_customer
@customer = @order.build_customer(customer_number: 123,
                                  customer_name: "John Doe")
- create_customer
@customer = @order.create_customer(customer_number: 123,
                                   customer_name: "John Doe")


### Opcije koje belongs_to podržava

|                            |                                                                                                  |
| ---------------------------| -------------------------------------------------------------------------------------------------|  
| :autosave                  | &nbsp;Rails čuva/uništava sve izmjene roditeljskog objekta, kad god se nad njim pozove save.     |
| :class_name                | &nbsp;Ako ime  modela ne može biti izvedeno iz imena asocijacije, navodi se o kom modelu se radi.|
| :counter_cache             | &nbsp;Dodaje se radi izbjegavanja count(*) upita nad bazom. Umjesto toga se čuva keširan brojač.                        |
| dependent: :destroy        | &nbsp;Poziva metod destroy nad svim povezanim objektima i briše ih.                              |
| dependent: :delete         | &nbsp;Briše sve povezane objekte, ali bez pozivanja metoda destroy nad njima.                    |
| dependent: :restrict       | &nbsp;Pozivanje metoda destroy nad tim objektom javlja grešku ako postoje povezani objekti.      |
| :foreign_key               | &nbsp;Omogućava setovanje spoljnog ključa na ime po želji, ali se tako mora zvati  u migracijama.|
| :primary_key               | &nbsp;Omogućava setovanje primarnog ključa, tako da to ne mora da bude id.                       |
| :through                   | &nbsp;Označava metod preko kog se izvršava upit.
                             |
| :touch                     | &nbsp;Ako se setuje na true, updated_at ili updated_on povezanog objekta će biti izmijenjeni           . |
| :validate                  | &nbsp;Ako se setuje na false, povezani objekti neće biti validirani kad god se taj objekat čuva. |


### Opseg vazenja belongs_to
Mogu se koristiti svi [standardni upiti](http://guides.rubyonrails.org/active_record_querying.html), ali sledeći su najčešći:

	- where
	- include
	- readonly
	- select

```
class LineItem < ActiveRecord::Base
  belongs_to :order, -> { includes :customer }
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class Customer < ActiveRecord::Base
  has_many :orders
end
```
Kod je malo efikasniji ako često koristite @line_item.order.customer.

Dodjeljivanje objekta belongs_to vezi ga ne čuva automatski, a ni povezani objekat.

### Opcije koje has_one podržava

|                            |                                                                                                  |
| ---------------------------| -------------------------------------------------------------------------------------------------|                        |
| :as                   | &nbsp;Označava da se radi o polimorfnoj asocijaciji.  |
| :source                    | &nbsp;Izvor aspocijacije kod has_one :through, ako se ne može zaključiti po imenu asocijacije.  |
| :source_type               | &nbsp;Izvor asocijacije kod has_one :through koja se dobija preko polimorfne asocijacije.      |


has_one podržava i opcije kao i belongs_to. Postoji razlika kod dependent:

    :destroy - svi povezani objekti se uništavaju
    :delete_all - svi povezani objekti se brišu direktno iz baze, pa se call back-ovi neće izvršiti
    :nullify - Spoljni ključevi se setuju na NULL. Call back-ovi se ne izvršavaju
    :restrict_with_exception - ako ima vezanih objekata, izuzetak se hvata
    :restrict_with_error - dodaje se greška vlasniku ako uopšte ima povezanih objekata

### Opseg vazenja has_one
Mogu se koristiti svi [standardni upiti](http://guides.rubyonrails.org/active_record_querying.html), ali sledeće ćemo razmotriti:

	- where
	- include
	- readonly
	- select

```
class Supplier < ActiveRecord::Base
  has_one :account, -> { where "confirmed = 1" }
end
```

Provjera da li je povezani objekat nil?
```
if @supplier.account.nil?
  @msg = "No account found for this supplier"
end
```
Kada se objekti čuvaju?
Pored već navedenog način, čuvanje povezanih objekata se može izbjeći korišćenjem association.build metoda.
Dodjeljivanje objekta has_one vezi ga čuva automatski, a i povezani objekat.
## Metode koje has_many dodaje
```
class Customer < ActiveRecord::Base
  has_many :orders
end
```

|                            |                                                                                                  |
| ---------------------------| -------------------------------------------------------------------------------------------------|  
| collection(force_reload = false) | &nbsp;Vraća niz povezanih objekata. Ako ih nema, vraća prazan niz.                         |
| collection<<(object, ...)        | &nbsp;Dodaje jedan ili više objekata kolekciji. FK = PK kolekcije koja poziva.             |
| collection.delete(object, ...)   | &nbsp;Briše jedan ili više objekata i postavljajući im a FK na NULL.                       |
| collection.destroy(object, ...)  | &nbsp;Briše jedan ili više objekata tako što poziva destroy nad svakim.                    |
| collection=objects               | &nbsp;Kolekcija će sadržati samo te objekte.                                               |
| collection_singular_ids          | &nbsp; Vraća niz id-jeva objekata iz kolekcije.                                            |
| collection_singular_ids=ids      | &nbsp;Kolekcija će sadržati samo te objekte sa tim id-jevima.                              | 
| collection.clear                 | &nbsp;Briše sve objekte iz kolekcije.                                                      |
| collection.empty?                | &nbsp;Provjerava da li je kolekcija prazna.                                                |
| collection.size                  | &nbsp;Vraća broj objekata u kolekciji.                                                     |
| collection.find(...)             | &nbsp;Traži objekte čiji su id-jevi  argumenti.                                            |
| collection.where(...)            | &nbsp; Traži objekte u kolekciji na osnovi where uslova.                                   |
| collection.exists?(...)          | &nbsp;Provjerava da li takav objekat postoji u kolekciji.                                  |
| collection.build(attributes = {}, ...) | &nbsp;Kreira jedan ili više objekata vezanog tipa, ali ih još ne čuva.               |
| collection.create(attributes = {})     | &nbsp;Radi isto, samo što ih i sačuva.                                               |


### Opseg vazenja has_many
Mogu se koristiti svi [standardni upiti](http://guides.rubyonrails.org/active_record_querying.html).
```
class Customer < ActiveRecord::Base
  has_many :recent_orders,
    -> { order('order_date desc').limit(100) },
    { offset(11) }
    class_name: "Order",
end
```
Nismo ograničeni samo onim što Rails nudi, već možemo sami da definišemo nove metode, a, ako te metode dijeli
više objekata, onda se definiše čitav modul:

```
module FindRecentExtension
  def find_recent
    where("created_at > ?", 5.days.ago)
  end
end
 
class Customer < ActiveRecord::Base
  has_many :orders, -> { extending FindRecentExtension }
end
 
class Supplier < ActiveRecord::Base
  has_many :deliveries, -> { extending FindRecentExtension }
end
```
Dodavanje indexa:
```
add_index :person_posts, :post, :unique => true
```



## has_and_belongs_to_many
```
class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
```
Kreirani su sledeći metodi:

	- assemblies(force_reload = false)
	- assemblies<<(object, ...)
	- assemblies.delete(object, ...)
	- assemblies.destroy(object, ...)
	- assemblies=objects
	- assembly_ids
	- assembly_ids=ids
	- assemblies.clear
	- assemblies.empty?
	- assemblies.size
	- assemblies.find(...)
	- assemblies.where(...)
	- assemblies.exists?(...)
	- assemblies.build(attributes = {}, ...)
	- assemblies.create(attributes = {})

Ako join tabela ima dodatnih kolona (što se uopšte ne preporučuje!!!), dodaju se kao atributi rekordima dobijenim preko te asocijacije. Ti rekordi su uvijek read-only, jer Rails ne može da ih sačuva. 


:join_table

Ako ima koje je po uobičajeno nije ono što odgovara, može se promijeniti ovom opcijom. 