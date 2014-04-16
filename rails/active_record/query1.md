    class Client < ActiveRecord::Base
      has_one :address
      has_many :orders
      has_and_belongs_to_many :roles
    end

    class Address < ActiveRecord::Base
      belongs_to :client
    end

    class Order < ActiveRecord::Base
      belongs_to :client, counter_cache: true
    end

    class Role < ActiveRecord::Base
      has_and_belongs_to_many :clients
    end

#1 Dohvatanje objekata iz baze podataka


+ bind
+ create_with
+ eager_load
+ extending
+ from
+ group
+ having
+ includes
+ joins
+ limit
+ lock
+ none
+ offset
+ order
+ preload
+ readonly
+ references
+ reorder
+ reverse_order
+ select
+ distinct
+ uniq
+ where

###1.1 Dohvatanje jendog objekta

###1.1.1 Upotrebom primarnog kljuca

>Upotrebom Model.find(primary_key) vrsi se dohvatanje objekta koji odgovara zadatom  
primarnom kljucu

    client = Client.find(10)

    SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1

>Model.find(primary_key) ce podici exeption ActiveRecord::RecordNotFound 

###1.1.2 take

    client = Client.take

    SELECT * FROM clients LIMIT 1

###1.1.3 first	

    client = Client.first

    SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1

###1.1.4 last

    client = Client.last

    SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1

###1.1.5 find_by

    Client.find_by first_name: 'Lifo'
    # => #<Client id: 1, first_name: "Lifo">
 
    Client.find_by first_name: 'Jon'
    # => nil

    Client.where(first_name: 'Lifo').take

###1.1.6 take!

###1.1.7 first!

###1.1.8 last!

###1.1.9 find_by!

###1.2.1 Niz primarnih kljuceva

    # Find the clients with primary keys 1 and 10.
    client = Client.find([1, 10]) # Or even Client.find(1, 10)
    # => [#<Client id: 1, first_name: "Lifo">, #<Client id: 10, first_name: "Ryan">]

    SELECT * FROM clients WHERE (clients.id IN (1,10))

>Model.find(arary) ce podici ActiveRecord::RecordNotFound ako ne nadje sve objekte za zadate primarne kljuceve

###1.2.2 take

>Model.take(limit) dohvata zapise bez redosleda ogranicen limit-om

    Client.take(2)
    # => [#<Client id: 1, first_name: "Lifo">,
          #<Client id: 2, first_name: "Raf">]

    SELECT * FROM clients LIMIT 2

###1.2.3 first

>Model.first(limit) 

    Client.first(2)
    # => [#<Client id: 1, first_name: "Lifo">,
          #<Client id: 2, first_name: "Raf">]

    SELECT * FROM clients ORDER BY id ASC LIMIT 2

###1.2.4 last

    Client.last(2)
    # => [#<Client id: 10, first_name: "Ryan">,
          #<Client id: 9, first_name: "John">]

    SELECT * FROM clients ORDER BY id DESC LIMIT 2

#1.3 Dohvatanje objekata u gomili(batches)

>Cesta je potreba za iteracijom nad velikim brojem zapisa, npr. slanje newsletter-a velikom  
broju korisnika

    # This is very inefficient when the users table has thousands of rows.
    User.all.each do |user|
      NewsLetter.weekly_deliver(user)
    end

>Ovaj pristup je cesto neefikasan povecavanjem velicine tabele, s obzirom da User.all.each  
govori Active Record-u da dohvati cijelu tabelu, praveci objekat modela za svaku vrstu i onda cuva cijeli niz objekata u memoriji. Dakle nastaje problem za veliki br zapisa.

>Rails doaje dvije metode koje omogucavaju podjelu zapisa u odredjene gomile(batches).Prva metoda find_each dohvata **gomilu** zapisa i prosledjuje svaki zapis bloku zasebno kao model.  
Druga metoda find_in_batches, dohvata **gomilu** i prosledjuje cijelu gomilu (batch) bloku kao niz modela.

###1.3.1 find_each

    User.find_each do |user|
      NewsLetter.weekly_deliver(user)
    end

>Dostupne su sve opcije kao za regularne find metode. Postoje jos dvije dodatne opcije:  
:batch_size

    User.find_each(batch_size: 5000) do |user|
      NewsLetter.weekly_deliver(user)
    end

:start

    User.find_each(start: 2000, batch_size: 5000) do |user|
      NewsLetter.weekly_deliver(user)
    end

> Ova :start opcija definise primarni kljuc od kojeg treba poceti dohvatanje. Korisno kod  
prekinutih procesa sa pretpostavkom da se sacuvao poslednji primarni kljuc kao checkpoint.

###1.3.2 find_in_batches

    # Give add_invoices an array of 1000 invoices at a time
    Invoice.find_in_batches do |invoices|
      export.add_invoices(invoices)
    end

>Iste opcije kao i za find_each, pored standardnih find opcija, i dodatne :start i :batch_size.

> Opcije :order i :limit su rezervisane za internu upotrebu kako find_each tako i find_  in_bathces metoda

#2 Uslovi 

>Uslovi mogu biti definisani putem stringa, hash-a ili niza.

###2.1 Obicni string uslov

>Client.where("orders_count = '2'").

###2.2 Upotreba niza

>Prethodni primjer , gdje je broje promjenjiv, kao argument 

    Client.where("orders_count = ?", params[:orders])

>Active Record prolazi kroz prvi argument uslova,  svi dodatni elementi mijenjaju znak pitanja (?).Slijed primer sa vise uslova.

    Client.where("orders_count = ? AND locked = ?", params[:orders], false)

####2.2.1
>Slicno kao u prethodnom primjeru samo su definisani key/value hash vrijednosti, radi bolje preglednosti kada je dugacak uslov.

    Client.where("created_at >= :start_date AND created_at <= :end_date",
      {start_date: params[:start_date], end_date: params[:end_date]})

###2.3 Hash uslovi 

>Dozvoljeno je definisanje uslova pomocu hash-eva, gdje je kljuc polje koje zelite da postavite sa odredjenom vrijednoscu.

    Client.where(locked: true)

>U slucaju belongs_tu veze, asocijtivni kljuc se moze koristiti da bi se definisao model,  
ako je Active record objekat koriscen kao vrijednost.

    Post.where(author: author)
    Author.joins(:posts).where(posts: {author: author})


###2.3.2 Uslovi sa opsegom

    Client.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)

    SELECT * FROM clients WHERE (clients.created_at BETWEEN '2008-12-21 00:00:00' AND '2008-12-22 00:00:00')

###2.3.3 Uslov sa podskupom

> Ako zelimo da dohvatamo zapise pomocu **IN** izraza mozemo proslijediti nizu u hash uslovu

    Client.where(orders_count: [1,3,5])

    SELECT * FROM clients WHERE (clients.orders_count IN (1,3,5))

###2.4 NOT uslov

    Post.where.not(author: author)

#3 Uredjenje (Ordering)

>Order metoda za dohvatanje zapisa u odredjenom redosljedu

    Client.order(:created_at)
    # OR
    Client.order("created_at")

    Client.order(created_at: :desc)
    # OR
    Client.order(created_at: :asc)
    # OR
    Client.order("created_at DESC")
    # OR
    Client.order("created_at ASC")

> Ili uredjenje po vise polja 

    Client.order(orders_count: :asc, created_at: :desc)
    # OR
    Client.order(:orders_count, created_at: :desc)
    # OR
    Client.order("orders_count ASC, created_at DESC")
    # OR
    Client.order("orders_count ASC", "created_at DESC")


#4 Selekcija odredjenih polja

    Client.select("viewable_by, locked")


    SELECT viewable_by, locked FROM clients

>Ovo zahtijeva odredjenu paznju jer inicijalizujemo objekat modela sa samo 2 polja  
koja smo oznacili. Ako pokusamo pristup poljima kojih nema u inicijalizovanom objektu  
dobicemo exception ActiveModel::MissingAttributeError: missing attribute: <attribute>

>Ako zelimo da dohvatimo jedan zapis po nekoj jedinstvanoj vrijednosti odredjenog polja imamo distinct  

    Client.select(:name).distinct

    SELECT DISTINCT name FROM clients

  	query = Client.select(:name).distinct
    # => Returns unique names
 
    query.distinct(false)
    # => Returns all names, even if there are duplicates

#5 Limit i ofset


    Client.limit(5)

    SELECT * FROM clients LIMIT 5

>Vratice 5 klijenata i to prvih 5 jer nije definisan ofset

    Client.limit(5).offset(30)

    SELECT * FROM clients LIMIT 5 OFFSET 30

>Vratice 5 klijenata pocev od 31-og, tj ofsetom definisemo koliko zapisa preskacemo.


#6 Group

>Da bi dodali GROUP BY , specifikujemo group metodu na find.

>Pr.ako zelimo da dohvatimo skup datuma kada su porudzbine kreirane.

    Order.select("date(created_at) as ordered_date, sum(price) as  
     total_price").group("date(created_at)")

    SELECT date(created_at) as ordered_date, sum(price) as total_price
    FROM orders
    GROUP BY date(created_at)


#7 Having

>SQL koristi **HAVING**  da bi definisao uslov na GROUP BY polja . Ovo mozemo realizovati  
dodavanjem :having opcije na find.

    Order.select("date(created_at) as ordered_date, sum(price) as total_price").
      group("date(created_at)").having("sum(price) > ?", 100)

    SELECT date(created_at) as ordered_date, sum(price) as total_price
    FROM orders
    GROUP BY date(created_at)
    HAVING sum(price) > 100

>Dobijamo po jedan objekat za svaki datum , ali samo one cija ce sumirana cijena iznositi vise od 100$.


#8 Preklapanje uslova

###8.1 except

>Mogu se definisati odredjeni uslovi koji ce biti izuzeti koriscenjem **except** metode.

    Post.where('id > 10').limit(20).order('id asc').except(:order)

    SELECT * FROM posts WHERE id > 10 LIMIT 20

###8.2 unscope

>except metoda ne radi kada je u pitanju merged relacija 

    Post.comments.except(:order)

>Imacemo :order ako je order definisan u nekom default scope-u na Comment-u. S namjerom  
da se odstrane sva uredjenja cak i merge-ovane relacije koristi se unscope na sljedeci nacin:  

    Post.order('id DESC').limit(20).unscope(:order) = Post.limit(20)
    Post.order('id DESC').limit(20).unscope(:order, :limit) = Post.all

>Dodatno mozemo unscope-ovati where klauzule.

    Post.where(:id => 10).limit(1).unscope(where: :id, :limit).order('id DESC') = Post.order('id DESC')

###8.2 only

> Mozemo vrsiti preklapanje uslova upotrebom only metode 

    Post.where('id > 10').limit(20).order('id desc').only(:order, :where)

    SELECT * FROM posts WHERE id > 10 ORDER BY id DESC


###8.4 reorder

>reorder metoda preklapa default scope order. Npr:

    class Post < ActiveRecord::Base
      ..
      ..
      has_many :comments, order: 'posted_at DESC'
    end
 
    Post.find(10).comments.reorder('name')

    SELECT * FROM posts WHERE id = 10 ORDER BY name


> U slucaju da nismo koristili reorder SQL koji bi se izvrsavao 

    SELECT * FROM posts WHERE id = 10 ORDER BY posted_at DESC

###8.5 reverse_order

    Client.where("orders_count > 10").order(:name).reverse_order

    SELECT * FROM clients WHERE orders_count > 10 ORDER BY name DESC

> Ako nije definisana order klauzula reverse_order uredjuje po primarnom kljucu   
u obrnutom redosledu

    Client.where("orders_count > 10").reverse_order

    SELECT * FROM clients WHERE orders_count > 10 ORDER BY clients.id DESC

>Ova metoda nema argumenata

#9 Null relacija

>**none** metoda vraca praznu relaciju bez zapisa. Svako dodatno uslovljavanje  
vracenu relaciju ce nastaviti sa generisanjem praznih relacija. Ova metoda moze biti korisna  
u situacijama gdje je potreban "chainable" odgovor na metod koji bi mogao da vrati 0 rezultata.

    Post.none # returns an empty Relation and fires no queries.

    # The visible_posts method below is expected to return a Relation.
    @posts = current_user.visible_posts.where(name: params[:name])
 
    def visible_posts
      case role
      when 'Country Manager'
        Post.where(country: country)
      when 'Reviewer'
        Post.published
      when 'Bad User'
        Post.none # => returning [] or nil breaks the caller code in this case
      end
    end

#10 Readonly objekti

 >Active Record pruza readonly metod na relaciju da bi eksplicitno onemogucio  
 modifikacije na povratne objekte. Pokusaj da se promijeni readonly zapis ce podici  
 ActiveRecord::ReadOnlyRecord exception.

    client = Client.readonly.first
    client.visits += 1
    client.save

>kako je client postavljen kao readonly objekat ovaj kod ce podici exception na poziv  
client.save sa promijenjenom vrijednoscu **visits**