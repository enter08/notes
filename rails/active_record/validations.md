# 1 Pregled

<pre><code>class Person < ActiveRecord::Base
	validates :name, presence: true
end 
 
Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false </code></pre>

Validacija nam govori da Person neće biti validna bez atributa name.

## Zašto koristimo validaciju?

Validaciju koristimo da osiguramo da samo validne podatke čuvamo u bazi podataka.
**Model-level** validacija je najbolji način da osiguramo da će samo validni podaci biti sačuvani. Ali, postoje i drugi načini za validaciju:

* Ograničenja u bazi podataka čine validaciju zavisnom od baze podataka. Primjer: uniqueness na ključu u bazi.
* Client-side validacija - nepouzdana ako se koristi sama.
* Controller-level validacija - ne preporučuje se

## Kada se izvršava validacija?

Postoje dvije vrste *Active Record* objekata, oni koji odgovaraju nekom redu unutar baze podataka i oni koji se još uvijek ne nalaze u bazi. Kada kreiramo novi objekat, on još uvijek nije u bazi sve dok ga na sačuvamo. Active Record koristi **new_record?** metodu da odredi da li je neki objekat već u bazi. Metoda vraća true ako se objekat **ne** nalazi u bazi podataka.

Validacija se izvršava **prije** čuvanja objekta u bazi.

Postoji vise načina da se promijeni stanje objekta u bazi. Neke metode će zahtijevati validaciju, a neke ne. Ovo znači da je moguće sačuvati objekat u bazi koji nije validan.

* Metode koje čuvaju objekte u bazi, samo ako su validni:

	* create
	* create!
	* save
	* save!
	* update
	* update!

*Napomena:* verzije metoda sa uzvičnikom (pr. save! (*enlg.* bang methods)) prave izuzetak ako metoda nije validna, non-bang verzije save i update vraćaju false, dok create samo vrati objekat.

* Metode koje preskaču validaciju i koje čuvaju objekat iako nije validan.

	* decrement!
	* decrement_counter
	* increment!
	* increment_counter
	* toggle!
	* touch
	* update_all
	* update_attribute
	* update_column
	* update_columns
	* update_counters

*Napomena:* **save** takođe može da preskoči validaciju sa:

<code> save(validate: false) </code>

## valid? i invalid?

Rails koristi valid? metodu da provjeri da li je neka metoda validna. Primjer je kod na početku. 
Nakon što Active Record validira objekat, greškama, u slučaju da postoje, se može pristupiti pomoću *errors*.
Izuzetak su objekti kreirani pomoću *new* gdje error neće vratiti grešku iako objekat nije validan.

<pre><code>$ rails c
&gt;&gt; p = Person.new
&gt;&gt; p. errors
=> {}
&gt;&gt; p.valid?
=> false</code></pre>

## errors[]

Spisku grešaka određenog atributa objekta se moze pristupiti pomoću errors[:attribute]. Metoda vraća niz grešaka za navedeni atribut ili prazan niz ako je atribut validan. Ovaj metod je koristan samo **nakon** validacije, jer metoda samo štampa greške iz već izvršene validacije, a ne izvršava samu validaciju.

# 2 Helperi za validaciju

Active Record nudi određen broj validacionih helpera koji se mogu koristiti unutar definicije klase.
Svaki helper prihvata proizvoljan broj imena atributa i na svakom su dostupne opcije **:on** i **:message** koje definišu kada validacija treba da se pokrene i koja poruka treba da se prikaže ako validacija ne uspije. *:on* opcija uzima vrijednost *:create* ili *:update*.

Slijedi spisak dostupnih helpera.

## acceptance

Ova metoda validira da je checkbox čekiran kada je forma submitovana. Ovo se obično koristi kada se od korisnika traži da prihvati ToS (Terms of Service) aplikacije. Ova validacije je česta kod web aplikacija i ovo 'prihvatanje' ne mora da se čuva u bazi podataka.

<pre><code>class Person < ActiveRecord::Base
	validates :terms_of_service, acceptance: true
end</code></pre>

Default poruka za grešku je "must be accepted".

Može imati *:accept* opciju, koja određuje vrijednost koja se razmatra za prihvatanje.

<pre><code>class Person < ActiveRecord::Base
	validates :terms_of_service, acceptance: { accept: 'yes' }
end</code></pre>

## validates_associated

Ovaj helper se koristi kada model ima vezu sa drugim modelima gdje takođe vršimo validaciju. Kada pokušamo da sačuvamo objekat, *valid?* će biti pozvan nad svakim objektom u modelu i povezanim modelima (associated models).

<pre><code>class Library < ActiveRecord::Base
	has_many :books
	validates_associated :books
end </code></pre>

*Napomena:* Ne koristite *validates_associated* u svim povezanim modelima (samo u jednom).

Default poruka za grešku je "is invalid".

## confirmation

Ovaj helper se koristi kada postoje dva tekstualna polja u kojim se očekuje isti sadržaj. Primjer su e-mail i password. Ovaj helper kreira virtuelni atribut cije ime se proširuje za "_confirmation".

<pre><code>class Person < ActiveRecord::Base
	validates :email, confirmation: true
end</code></pre>

U view template se koristi nešto poput:

<pre><code><%= text_field :person, :email %>
<%= text_field :person, :email_confirmation %></code></pre>

Kada se vrši potvrda, potrebno je potvrditi i prisustvo confirmation polja:

<code> validates :email_confirmation, presence: true </code>

Default poruka za grešku je "doesn't match confirmation".

## exclusion

Ovaj helper validira da vrijednosti atributa nisu uključene u datom skupu.

<pre><code> class Account < ActiveRecord::Base
	validates :subdomain, exclusion: { in: %w(www us ca jp),
    message: "Subdomain %{value} is reserved." }
end </code></pre>

Sinonim za :in je :within.

## format

Ovaj helper provjerava da li vrijednosti atributa odgovaraju datom regularnom izrazu, koji se definiše sa opcijom :with.

<pre><code> class Product < ActiveRecord::Base
	validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/,
    message: "Only letters allowed" }
end</code></pre>

Default poruka za grešku je "is invalid".

## inclusion

Ovaj helper validira da su vrijednosti atributa uključene u datom skupu.

<pre><code> class Coffee < ActiveRecord::Base
	validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }
 end </code></pre>

Default poruka za grešku je "is not included in the list".

## length

Ovaj helper validira dužinu vrijednosti atributa.

<pre><code>class Person < ActiveRecord::Base
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end</code></pre>

Poruke koje se štampaju, u slučaju da validacija ne uspije, se mogu definisati sa :wrong_length, :too_long, i :too_short opcijama, uz %{count} koji vraća broj koji odgovara dužini korišćenog ograničenja. Opcija **:message** je takodje dostupna.

<pre><code>class Person < ActiveRecord::Base
	validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end</code></pre>

## numericality

Ovaj helper validira da atribut ima samo broj;ane vrijednosti. Po default-u, numericality vraća **true** jednog opcionog znaka nakon kojeg slijedi cijeli ili realan broj. Ako nam je potreban samo cijeli broj, koristimo opciju :only_integer.

<pre><code>class Player < ActiveRecord::Base
	validates :points, numericality: true
	validates :games_played, numericality: { only_integer: true }
end</code></pre>

Osim :only_integer, dostupne su i sljedeće opcije:

* :greater_than
* :greater_than_or_equal_to
* :equal_to
* :less_than
* :less_than_or_equal_to
* :odd
* :even

Default poruka za grešku je "is not a number".

## presence

Ovaj helper validira da određeni atribut nije prazan. Koristi blank? metodu da provjeri da li je vrijednost atributa nil ili prazan string.

<pre><code>class Person < ActiveRecord::Base
	validates :name, :login, :email, presence: true
end</code></pre>

Ako provjeravamo da li je asocijacija (povezani objekat) prisutan, potrebno je provjeriti prisustvo tog objekta a ne spoljnog ključa koji se koristi da se povežu objekti.

<pre><code>class LineItem < ActiveRecord::Base
	belongs_to :order
	validates :order, presence: true
end</code></pre>

Ako želimo da validiramo zapise čije se prisustvo zahtijeva, koristimo opciju :inverse_of

<pre><code>class Order < ActiveRecord::Base
	has_many :line_items, inverse_of: :order
end</code></pre>

Ako validiramo prisustvo boolean polja, koristimo:

<code>validates :field_name, inclusion: { **in:** [true, false] }.</code>

Default poruka za grešku je "can't be blank".

## absence

Ovaj helper validira da je određeni atribut odsutan. Koristi present? metodu da provjeri da vrijednost atributa nije nil ili prazan string.

<pre><code>class Person < ActiveRecord::Base
	validates :name, :login, :email, absence: true
end</code></pre>

Primjeri za asocijacije su analogni primjerima iz presence helpera.

## uniqueness

Ovaj helper validira da je vrijednost atributa jedinstvena prije nego što se objekat sačuva. Ne kreira uniqueness ograničenje u bazi tako da se može desiti da dvije različite konekcije na bazu kreiraju dva zapisa sa istom vrijednošću za kolonu koja treba da je jedinstvena. Da bi se ovo izbjeglo, potrebno je napraviti unique indeks u bazi.

<pre></code>class Account < ActiveRecord::Base
	validates :email, uniqueness: true
end</code></pre>

Postoji **:scope** opcija koja se koristi da se navedu drugi atributi koji se koriste da se limitira provjera jedinstvenosti:

<pre><code>class Holiday < ActiveRecord::Base
	validates :name, uniqueness: { scope: :year,
    message: "should happen once per year" }
end</code></pre>

Takođe, postoji i :case_sensitive opcija.

## validates_with

Ovaj helper predaje zapis drugoj klasi za validaciju.

<pre><code>class Person < ActiveRecord::Base
  validates_with GoodnessValidator
end
 
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if record.first_name == "Evil"
      record.errors[:base] << "This person is evil"
    end
  end
end</code></pre>

Ovaj helper uzima klasu ili više klasa koje se koriste za validaciju. Poruka za grešku se mora ručno definisati jer ne postoji default vrijednost. Da bi se validate metoda implemetirala mora postojati record parametar koji se validira.

<pre><code>class Person < ActiveRecord::Base
	validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
 
class GoodnessValidator < ActiveModel::Validator
def validate(record)
    if options[:fields].any?{|field| record.send(field) == "Evil" }
      record.errors[:base] << "This person is evil"
    end
  end
end</code></pre>

Validator će biti inicijalizovan samo jednom, a ne svaki put kada se pokrene validacija. 

Ako želimo instance varijable unutar validatora, to radimo na sljedeći način:

<pre><code>class Person < ActiveRecord::Base
  validate do |person|
    GoodnessValidator.new(person).validate
  end
end
 
class GoodnessValidator
  def initialize(person)
    @person = person
  end
 
  def validate
    if some_complex_condition_involving_ivars_and_private_methods?
      @person.errors[:base] << "This person is evil"
    end
  end
 
  # …
end</code></pre>

## validates_each

Ovaj helper validira atribut unutar bloka.

Ako, na primjer, ne želimo da imena i prezimena počinju sa malim slovom:

<pre><code>class Person < ActiveRecord::Base
    validates_each :name, :surname do |record, attr, value|
      record.errors.add(attr, 'must start with upper case') if value =~ /\A[a-z]/
    end
end</code></pre>

Bloku se predaje zapis, ime atributa i vrijednost atributa. Možemo testirati bilo šta unutar bloka. Poruka za grešku ne postoji po defaultu (mora se definisati).

# 3 Uobičajene opcije za validaciju

## :allow_nil

Ova opcija preskače validaciju kada je validirana vrijednost nil.

<pre><code>class Coffee < ActiveRecord::Base
    validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }, allow_nil: true
end</code></pre>

## :allow_blank

Ova opcija dozvoljava prazan string ali i nil, jer koristi metodu blank?.

## :message

Definiše poruku koja se prikazuje u slučaju neuspješne validacije.

## :on

:on opcija određuje kada će se validacija izvršiti. Po defaultu, validacija se pokreće nakon čuvanja i update-ovanja. Da se promijeni default, koristi se <code>on: :create</code> ili <code>on: :update</code>

<pre><code>class Person < ActiveRecord::Base
  # možemo update-ovati e-mail da bude kao neki već postojeći e-mail (tj. postojaće dva ista e-mail u bazi)
  validates :email, uniqueness: true, on: :create
 
  # biće moguće kreirati zapis gdje godina ne mora biti broj
  validates :age, numericality: true, on: :update
 
  # default
  validates :name, presence: true
end</code></pre>

# 4 Striktna validacija

Validaciju možemo izvršiti sa opcijom strict koja će vratiti izuzetak (exeption) iz ActiveModel::StrictValidationFailed kada objekat nije validan.

<pre><code>class Person < ActiveRecord::Base
    validates :name, presence: { strict: true }
end
 
Person.new.valid?
#=> ActiveModel::StrictValidationFailed: Name can't be blank</code></pre>

Postoji mogućnost da odredimo i koji izuzetak će biti vraćen sa:

<code>strict: TokenGenerationException</code>

# 5 Kondicionalna validacija

Često je korisno da validiramo objekat samo kada je zadovoljen određen uslov. To možemo uraditi koristeći **:if** i **:unless** opcije, koje mogu raditi sa simbolima, stringovima, Proc-om ili nizom. *:if* opcija se koristi kada određujemo kada bi validacija **trebala** da se izvrši. Ako određujemo kada validacija **ne bi trebala** da se izvrši, korisimo *:unless* opciju.

## Korićenje simbola sa :if i :unless

:if i :unless opcije koriste simbole koji odgovaraju imenu metode koja se poziva prije nego se validacija izvrši. Ovo je najčešća primjena simbola sa ovim opcijama.

<pre><code>class Order < ActiveRecord::Base
    validates :card_number, presence: true, if: :paid_with_card?
 
    def paid_with_card?
        payment_type == "card"
    end
end</code></pre>

## Korićenje stringova sa :if i :unless

Stringovi koji se koriste u validaciji se izvšavaju pomoću eval metode i moraju sadržati validan Ruby kod. Ova opcije bi se trebala koristiti kada stringovi predstavljaju vrlo kratke uslove.

<pre><code>class Person < ActiveRecord::Base
    validates :surname, presence: true, if: "name.nil?"
end</code></pre>

## Korišćenje Proc objekata sa :if i :unless

Korišćenjem Proc objekta možemo pisati uslov unutar bloka. Najpogodniji su za jednolinijske uslove.

<pre><code>class Account < ActiveRecord::Base
    validates :password, confirmation: true,
    unless: Proc.new { |a| a.password.blank? }
end</code></pre>

## Grupisanje kondicionalnih validacija

Nekada nam je potrebno da više validacija ima jedan uslov, što postižemo korišćenjem **with_options**

<pre><code>class User < ActiveRecord::Base
    with_options if: :is_admin? do |admin|
      admin.validates :password, length: { minimum: 10 }
      admin.validates :email, presence: true
    end
end</code></pre>

Sve validacije unutar bloka će imati uslov <code>if: :is_admin?</code>

## Kombinovanje uslova validacije

Kada više uslova definišu da li će se, ili ne, validacija izvšiti, može se koristit niz. Čak se :if i :unless mogu zajedno koristiti.

<pre><code>class Computer < ActiveRecord::Base
    validates :mouse, presence: true,
        if: ["market.retail?", :desktop?]
        unless: Proc.new { |c| c.trackpad.present? }
end</code></pre>

Validacija se izvršava samo ako su svi uslovi iz :if i nijedan iz :unless true.

# 6 Definisanje validacija

## Definisanje validatora

Validatori su klase koji nasleđuju ActiveModel::Validator. Ove klase moraju implementirati validate metodu koja uzima zapis kao argument kojeg validira. Validator se poziva pomoću validates_with metode. 

<pre><code>class MyValidator < ActiveModel::Validator
    def validate(record)
      unless record.name.starts_with? 'X'
        record.errors[:name] << 'Need a name starting with X please!'
      end
    end
end
 
class Person
  include ActiveModel::Validations
  validates_with MyValidator
end</code></pre>

Najlakši način da se doda validator za validaciju individualnih atributa je pomoću ActiveModel::EachValidator, a mora biti implementirana validate_each? metoda koja ima tri argumenta: record, attribute, value.

<pre><code>class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
end
 
class Person < ActiveRecord::Base
  validates :email, presence: true, email: true
end</code></pre>

## Definisanje metoda

Osim validatora, postoji mogućnost kreiranja metoda koji verifikuju stanje modela i dodaju poruku errors kolekciji ako nisu validni. Nakon toga se moraju pozvati korišćenjem validate metode, prosljeđujući simbol imena metode za validaciju.

<pre><code>class Invoice < ActiveRecord::Base
    validate :expiration_date_cannot_be_in_the_past,
             :discount_cannot_be_greater_than_total_value
 
    def expiration_date_cannot_be_in_the_past
        if expiration_date.present? && expiration_date < Date.today
         errors.add(:expiration_date, "can't be in the past")
        end
    end
 
    def discount_cannot_be_greater_than_total_value
        if discount > total_value
          errors.add(:discount, "can't be greater than total value")
        end
    end
end</code></pre>

Po defaultu, validacija će se izvršiti svaki put kada se pozove valid?, ali i to je moguće kontrolisati, dodajući opciju :on sa :create ili :update.

<pre><code>class Invoice < ActiveRecord::Base
validate :active_customer, on: :create
 
    def active_customer
        errors.add(:customer_id, "is not active") unless customer.active?
    end
end</code></pre>

# 7 Rad sa greškama validacije

Slijedi spisak najkorišćenijih metoda za rad sa errors kolekcijom. 

*Napomena:* Spisak svih dostupnih metoda možete pronaći u **ActiveModel::Errors**.

## errors

Vraća instancu klase ActiveModel::Errors koja sadrži sve greške. Svaki ključ je ime atributa a vrijednost je niz stringova sa svim greškama.

<pre><code>class Person < ActiveRecord::Base
    validates :name, presence: true, length: { minimum: 3 }
end
 
person = Person.new
person.valid? # => false
person.errors
# => {:name=>["can't be blank", "is too short (minimum is 3 characters)"]}
 
person = Person.new(name: "John Doe")
person.valid? # => true
person.errors # => []</code></pre>

## errors.add

Add metoda omogućava dodavanje poruka za određene atribute. Koriste se errors.full_messages i errors.to_a metode da se vide poruke u formi u kojoj će biti prikazane korisniku. 

<pre><code>class Person < ActiveRecord::Base
    def a_method_used_for_validation_purposes
        errors.add(:name, "cannot contain the characters !@#%*()_-+=")
    end
end
 
person = Person.create(name: "!@#")
 
person.errors[:name]
 # => ["cannot contain the characters !@#%*()_-+="]
 
person.errors.full_messages
 # => ["Name cannot contain the characters !@#%*()_-+="]</code></pre>

 drugačiji zapis je:

 <code>errors[**:name**] = "cannot contain the characters !@#%*()_-+="</code>

## errors[:base]

 Mogu se dodati greške koje se odnose na čitav objekat, bez obzira na vrijednosti atributa. Ovaj metod se koristi kada neki objekat želimo da proglasimo nevalidnim. S obzirom da je errors[:base] niz, pripisujemo nizu string koji će se koristiti kao poruka za grešku.  

 <pre><code>class Person < ActiveRecord::Base
    def a_method_used_for_validation_purposes
        errors[:base] << "This person is invalid because ..."
    end
end</code></pre>

## errors.clear

Ova metoda se koristi kada želite da izbrišete sve poruke iz errors kolekcije. Naravno, pozivanjem errors.clear nad nevalidnim objektom neće ga učiniti validnim, već će errors kolekcija biti prazna. Kada sljedeći put validacija ne uspije, errors kolekcija će ponovo biti puna.

<pre><code>class Person < ActiveRecord::Base
    validates :name, presence: true, length: { minimum: 3 }
end
 
person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
 
person.errors.clear
person.errors.empty? # => true
 
p.save # => false
 
p.errors[:name]
# => ["can't be blank", "is too short (minimum is 3 characters)"]
</code></pre>

## errors.size

Vraća broj poruka koje se štampaju u slučaju neuspjele validacije.

<pre><code>class Person < ActiveRecord::Base
  validates :name, presence: true, length: { minimum: 3 }
end
 
person = Person.new
person.valid? # => false
person.errors.size # => 2
 
person = Person.new(name: "Andrea", email: "andrea@example.com")
person.valid? # => true
person.errors.size # => 0</code></pre>

# 8 Prikaz validacionih grešaka u browseru (Views)

Rails ne uključuje nijedan view helper koji bi mogao pomoći u generisanju poruka direktno. Međutim, zbog velikog broja metoda za interakciju sa validacijom uopšte, prikaz grešaka ne predstavlja problem.

    <% if @post.errors.any? %>
       <div id="error_explanation">
       <h2><%= pluralize(@post.errors.count, "error") %> prohibited this post from being saved:</h2>
 
    <ul>
    <% @post.errors.full_messages.each do |msg| %><br/>
        <li><%= msg %></li><br/>
    <% end %><br/>
    </ul>
    </div>
    <% end %>


**Izvor:**
[Rails Guides - Active Record Validations](http://guides.rubyonrails.org/active_record_validations.html)


