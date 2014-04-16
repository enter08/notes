## Active Record Query Interface

### 11. Mehanizmi prevencije nastanka nekonzistentnog stanja

#### 11.1 Optimisticko zakljucavanje

Osnova ovakvog pristupa je pretpostavka da su konflikti (do kojih dolazi kad vise korisnika istovremeno mijenja isti red u bazi) rijetki. Odlika ovog pristupa je da omogucava vecem broju korisnika da istovremeno pristupa istom redu u bazi podataka. To se postize time sto se provjera da li je doslo do promjena u vremenskom intervalu nakon sto je trenutni korisnik pristupio odgovarajucem redu u bazi.

Implementacija ovog mehanizma zasniva se na ``lock_version`` koloni ciji je tip integer. Svaki uspjesna promjena inkrementira vrijednost ovog polja odgovarajuceg reda u bazi podataka. Do konflikta dolazi kada pri pokusaju promjene novi set vrijednosti sadrzi manju vrijednost ``lock_version`` polja u odnosu na trenutnu. Ovakvo stanje rezultira bacanjem izuzetka ``ActiveRecord::StaleObjectError`` cije obrada i procesiranje je odgovornost korisnickog aplikacionog koda.

- ActiveRecord::Base.lock_optimistically = false (iskljucuje optimisticko zakljucavanje)
-  self.locking_column = :lock_client_column (ovaj izraz unutar definicije modela modifikuje predodredjeno ime za kolonu ``lock_version``)

#### 11.2 Pesimisticko zakljucavanje

Ovaj pristup se zasniva na mehanizmima koje pruzaju same baze podataka. Ovim mehanizmom jedan korisnik dobija eskluzivni pristup odredjenom redu u bazi podataka. Odredjene baze omogucavaju da se ovaj uslov ublazi. Da bi se preventivno sprijecio nastanak ``deadlock`` situacije koriste se transakcije. 

    Item.transaction do
      i = Item.lock.first
      i.name = 'Jones'
      i.save
    end

### 12 Spajanje tabela

ActiveRecord definise klasni metod ``joins`` za kreiranje ``JOIN`` SQL deklaracija. Kao i u slucajevima ostalih ActiveRecord metoda postoji vise nacina na koji se ``joins`` metod moze pozvati:

- SQL string kao input
- Niz ili Hash na imenovanim asocijacijama (podrzava samo INNER JOIN, vise informacija o INNER vs OUTER JOIN na [link1](http://www.programmerinterview.com/index.php/database-sql/inner-vs-outer-joins), [link2](http://stackoverflow.com/questions/38549/difference-between-inner-and-outer-join)), [link3](http://i.imgur.com/1m55Wqo.jpg)
- Specificiranje uslova na tabelama nastalim kao rezultat joins deklaracija

Postavka:

    class Category < ActiveRecord::Base
      has_many :posts
    end
     
    class Post < ActiveRecord::Base
      belongs_to :category
      has_many :comments
      has_many :tags
    end
     
    class Comment < ActiveRecord::Base
      belongs_to :post
      has_one :guest
    end

Primjer 1:

    Category.joins(:posts)

Rezultat 1:

    SELECT categories.* FROM categories
    INNER JOIN posts ON posts.category_id = categories.id

Primjer 2:

    Post.joins(:category, :comments)

Rezultat 2:

    SELECT posts.* FROM posts
    INNER JOIN categories ON posts.category_id = categories.id
    INNER JOIN comments ON comments.post_id = posts.id


#### 12.3 Definisanje uslova na spojenim tabelama pomocu niza, stringa i hasha

Alternativa 1:

    time_range = (Time.now.midnight - 1.day)..Time.now.midnight
    Client.joins(:orders).where('orders.created_at' => time_range)

Alternativa 2:

    time_range = (Time.now.midnight - 1.day)..Time.now.midnight
    Client.joins(:orders).where(orders: {created_at: time_range})


### 13 Pohlepno ucitavanje asocijacija

Pohlepno ucitavanje asocijacija pruza resenje za problem ``N+1 upita``. Sledeci primjer generise 11 upita

    clients = Client.limit(10)
     
    clients.each do |client|
      puts client.address.postcode
    end

ActiveRecord nudi resenje za navedeni problem u obliku ``includes`` metoda. U prisustvu ovog metoda ActiveRecord ucitava odgovarajuce asocijacije u minimalnom broju SQL upita.

Ovaj kod:

    clients = Client.includes(:address).limit(10)
     
    clients.each do |client|
      puts client.address.postcode
    end

Rezultira sledecim SQL upitom: 

    SELECT * FROM clients LIMIT 10
    SELECT addresses.* FROM addresses
    WHERE (addresses.client_id IN (1,2,3,4,5,6,7,8,9,10))

ActiveRecord omogucava pohlepno ucitavanje vise asocijacija istovremeno:

Na osnovu niza kao inputa:

    Post.includes(:category, :comments)

Na osnovu hash-a kao inputa:

    Category.includes(posts: [{comments: :guest}, :tags]).find(1)

#### 13.2 Definisanje uslova na pohlepno ucitanim asocijacijama

Iako ActiveRecord omogucava definisanje uslova na pohlepno ucitanim asocijacijama to se ne preporucuje. Bolja alternativa da se uslovi definisu na ``joins`` tabelama.

Definisanje uslova na pohlepno ucitanim asocijacijama generise SQL upite sa ``LEFT OUTER JOIN``


### 14 Opsezi (Scopes)

`Opsezi` sluze za definisanje cesto koriscenih upita i zatim njihovo koriscenje kao metoda na modelima i njihovim asocijacijama. Ostali metodi (kao sto su where, joins i includes) takodje su primjenjivi na ``scopes``. Rezultat svih `opseg` metoda su objekti klase ``ActiveRecord::Relation``. Ovo omogucava nadovezivanje drugih metoda, pa cak i `opsega`. 

Definicija opsega:
    
    class Post < ActiveRecord::Base
      scope :published, -> { where(published: true) }
    end

Identicno kao:

    class Post < ActiveRecord::Base
      def self.published
        where(published: true)
      end
    end

Pozivanje opsega je moguce na klasi Post:

    Post.published # => [published posts]

Kao i na asocijaciji koja se sastoji od vise Post objekata:

    category = Category.first
    category.posts.published # => [published posts belonging to this category]

#### 14.1 Opsezi mogu imati argumente

Definicija:

    class Post < ActiveRecord::Base
      scope :created_before, ->(time) { where("created_at < ?", time) }
    end

Poziv:

    Post.created_before(Time.zone.now)

Za ovu namjenu ipak se preporucuje koriscenje alternativne sintakse i definisanje klasnih metoda koje prihvataju iste argumente (vidi prethodnu sekciju).

#### 14.2 Spajanje opsega

Opsesi se mogu nadovezivati kada dolazi do spajanja njihovih uslova sa ``AND``. 

Ako zelimo da naglasimo da u slucaju uslova koji adresiraju iste atribute zelimo poslednji da bude jedini primjenjivi mozemo koristiti ``merge`` metod.

U slucaju ``default_scope`` moramo znati da ostali opsezi kao i where uslovi koji adresiraju isti atribut ce prepisati iste uslove iz  ``default_scope``.


    class User < ActiveRecord::Base
      default_scope  { where state: 'pending' }
      scope :active, -> { where state: 'active' }
      scope :inactive, -> { where state: 'inactive' }
    end
     
    User.all
    # => SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'
     
    User.active
    # => SELECT "users".* FROM "users" WHERE "users"."state" = 'active'
     
    User.where(state: 'inactive')
    # => SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'

#### 14.3 Default opseg (default_scope)

``default_scope`` omogucava definisanje upita zajednickog za sve upite odredjenog modela.


#### 14.4 Uklanjanje opsega

Za uklanjanje bilo kakvog opsega, ukljucujuci i default opseg mozemo koristiti ``unscoped`` metod.

    Client.unscoped.all


### 15. Dynamic finders

Ovaj mehanizam je predvidjen za uklanjanje iz Rails-a u verziji 4.1. Kao alternativa za ovu funkcionalnost preporucuje se koriscenje ``opsega``.

Dynamic finders predstavljaju varijante find_by funkcije baziranih na imenima atributa odredjenog modela. Varijante sa ! na kraju bacaju izuzetke tipa ActiveRecord::RecordNotFound. Mogu se takodje nadovezivati.

### 16. Nadji ili napravi novi objekat

#### 16.1 find_or_create_by

    Client.find_or_create_by(first_name: 'Andy')

find_or_create_by vraca ili vec postojeci rekord ili novi tek kreirani rekord. Novi rekord moze biti sacuvan u bazi ili ne. To zavisi od procesa validacije. find_or_create_by! varijanta baca izuzetak ActiveRecord::RecordInvalid u slucaju greske u procesu validacije.

Inicijalizacija novog rekorda kreiranog u find_or_create_by metodi je moguca pomocu bloka koji se izvrsava jedino kad se novi rekord kreira.

    Client.find_or_create_by(first_name: 'Andy') do |c|
      c.locked = false
    end

#### 16.3 find_or_initialize_by

Ovaj metod se razlikuje od prethodnog jedino u tome sto poziva `new` umjesto `create` sto kao efekat ima to da novo kreirani objekat nece biti sacuvan u bazi.


### 17 Pronalazenje rekorda uz direktno koriscenje SQL

metod ``find_by_sql`` vraca niz sa jednim ili vise rekorda kao rezultat

    Client.find_by_sql("SELECT * FROM clients
    INNER JOIN orders ON clients.id = orders.client_id
    ORDER clients.created_at desc")

metod ``connection#select_all`` dohvata rekorde iz baze ali ne kreira ActiveRecord objekte vec niz hash objekata koji predstavljaju odgovarajuce rekorde u bazi.
  
    Client.connection.select_all("SELECT * FROM clients WHERE id = '1'")

metod ``pluck`` prihvata listu record polja kao argument a vraca niz sa odgovarajucim vrijednostima za prethodno navedena polja. pluck metod konvertuje rezultate direktno u niz, bez prethodnog kreiranja ActiveRecord objekata.

    Client.pluck(:id, :name)
    # SELECT clients.id, clients.name FROM clients
    # => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]

``pluck`` izvrsava upit momentalno pa samim tim ne podrzava nadovezivanje na njega ``opsege`` ili dodatnih uslova. Medjutim pluck metod moze da se nadoveze na prethodne uslove ili ``opsege``.

``ids`` metod moze da se koristi kao precica za ``pluck(:id)`` metod
  
    Person.ids
    # SELECT id FROM people

### 18 Postojanje objekata

find metod u pozadini, samo vraca true ili false

      Client.exists?(1) 

vraca true ako bilo koji od recorda za navedene id-ijeve postoji

      Client.exists?(1,2,3)
      # or
      Client.exists?([1,2,3])

    
razlicite alternative koje omogucavaju utvrdjivanje da li postoje u odredjenoj kolekciji rekordi.
      
      # via a model
      Client.exists?
      Client.any?
      Client.many?

### 19 Izracunavanja

Metode iz ove grupe:

- Count
- Average (npr. ``Client.average("orders_count")``)
- Minimum (npr. ``Client.minimum("age")``)
- Maximum (npr. ``Client.maximum("age")``)
- Sum (npr. ``Client.average("orders_count")``)

se pozivaju direktno na modelu (npr. ``Client.count``) 

ili na relaciji (npr. ``Client.where(first_name: 'Ryan').count``).


### 20 Izvrsavanje EXPLAIN metoda ili debug-ovanje generisanih SQL upita


    User.where(id: 1).joins(:posts).explain


[Vise informacija o koriscenju PostgreSQL EXPLAIN](http://www.postgresql.org/docs/current/static/using-explain.html)
