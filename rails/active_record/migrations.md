## Active Record Migrations

Migracije predstavljaju jednostavan nacin za generisanje i manipulisanjem semom baze podataka na konzistentan nacin. Uz koriscenje Ruby DSL i izbjegavanjem pisanja SQL (cija sintaksa cesto zavisi od baze podataka) migracije omogucavaju odvojenost aplikacionog koda od konkretnog sistema za implementaciju baze podataka.

ActiveRecord koristi migracije da bi manipulisao db/schema.rb fajlom koji predstavlja semu baze podataka. Svaka migracija moze se posmatrati kao git commit koji semu baze podataka ili git repo prevodi u novo stanje.

Na bazama koje podrzavaju transakcije, migracije ce biti izvrsene unutar transakcije. Na bazama koje ne podrzavaju transakcije, moze doci do nekonzistentnog stanja i u ovakvim slucajevima sam korisnik mora intervenisati.

Primjer migracije:

    class CreateProducts < ActiveRecord::Migration
      def change
        create_table :products do |t|
          t.string :name
          t.text :description
     
          t.timestamps
        end
      end
    end

Primjer migracije gdje se eksplicitno navodi ponasanje u slucaju primjene ove migracije i njenog povlacenja (`rollback`).

      
    class ChangeProductsPrice < ActiveRecord::Migration
      def change
        reversible do |dir|
          change_table :products do |t|
            dir.up   { t.change :price, :string }
            dir.down { t.change :price, :integer }
          end
        end
      end
    end

Alternativa: 

    class ChangeProductsPrice < ActiveRecord::Migration
      def up
        change_table :products do |t|
          t.change :price, :string
        end
      end
     
      def down
        change_table :products do |t|
          t.change :price, :integer
        end
      end
    end

### Kreiranje migracije

Primjer:

    rails generate migration AddPartNumberToProducts

Rezultat:

    class AddPartNumberToProducts < ActiveRecord::Migration
      def change
      end
    end

Rails konvencije:

1. "AddXXXToYYY"
2. "RemoveXXXFromYYY"
3. "CreateYYY"
4. "CreateJoinTableXxxYyy"


1. 

    class AddPartNumberToProducts < ActiveRecord::Migration
      def change
        add_column :products, :part_number, :string
      end
    end

2.

    class RemovePartNumberFromProducts < ActiveRecord::Migration
      def change
        remove_column :products, :part_number, :string
      end
    end

3.

    class CreateProducts < ActiveRecord::Migration
      def change
        create_table :products do |t|
          t.string :name
          t.string :part_number
        end
      end
    end

4. 

    class CreateJoinTableCustomerProduct < ActiveRecord::Migration
      def change
        create_join_table :customers, :products do |t|
          # t.index [:customer_id, :product_id]
          # t.index [:product_id, :customer_id]
        end
      end
    end

Ostale mogucnosti generatora:

1. kreiranje indeksa
2. kreiranje vise kolona odjednom
3. kreiranje references polja

1.
Komanda

    rails generate migration AddPartNumberToProducts part_number:string:index

Rezultat

    class AddPartNumberToProducts < ActiveRecord::Migration
      def change
        add_column :products, :part_number, :string
        add_index :products, :part_number
      end
    end

2. 
Komanda

    rails generate migration AddDetailsToProducts part_number:string price:decimal

Rezultat

    class AddDetailsToProducts < ActiveRecord::Migration
      def change
        add_column :products, :part_number, :string
        add_column :products, :price, :decimal
      end
    end

3. 
Komanda 

    rails generate migration AddUserRefToProducts user:references

Rezultat

    class AddUserRefToProducts < ActiveRecord::Migration
      def change
        add_reference :products, :user, index: true
      end
    end

### 2.2 Generatori modela

Podrzani modifikatori tipova:

- limit (maksimalna velicina stringa, teksta, integer polja)
- precision (ukupan broj cifara u broju)
- scale (broj cifara nakon decimalne tacke)
- polymorphic (definise i dodaje type kolonu kod belong_to asocijacija)
- null (definise da li kolona moze imati NULL vrijednosti)


Komanda

    rails generate migration AddDetailsToProducts price:decimal{5,2} supplier:references{polymorphic}

Rezultat

    class AddDetailsToProducts < ActiveRecord::Migration
      def change
        add_column :products, :price, :decimal, precision: 5, scale: 2
        add_reference :products, :supplier, polymorphic: true, index: true
      end
    end

### 3 Pisanje migracija

metod ``create_table``

    create_table :products do |t|
      t.string :name
    end

metod ``create_join_table`` (indeksi nisu kreirani po default-u)

    create_join_table :products, :categories do |t|
      t.index :product_id
      t.index :category_id
    end

metod ``change_table``

    change_table :products do |t|
      t.remove :description, :name
      t.string :part_number
      t.index :part_number
      t.rename :upccode, :upc_code
    end

Ostale metode podrzane u change_table bloku:

- add_column
- add_index
- add_reference
- add_timestamps
- create_table
- create_join_table
- drop_table (must supply a block)
- drop_join_table (must supply a block)
- remove_timestamps
- rename_column
- rename_index
- remove_reference
- rename_table

#### 3.6 Koriscenjem reversible

Primjer

    class ExampleMigration < ActiveRecord::Migration
      def change
        create_table :products do |t|
          t.references :category
        end
     
        reversible do |dir|
          dir.up do
            #add a foreign key
            execute <<-SQL
              ALTER TABLE products
                ADD CONSTRAINT fk_products_categories
                FOREIGN KEY (category_id)
                REFERENCES categories(id)
            SQL
          end
          dir.down do
            execute <<-SQL
              ALTER TABLE products
                DROP FOREIGN KEY fk_products_categories
            SQL
          end
        end
     
        add_column :users, :home_page_url, :string
        rename_column :users, :email, :email_address
      end 

  #### 3.7 Koriscenjem up/down metoda

      class ExampleMigration < ActiveRecord::Migration
        def up
          create_table :products do |t|
            t.references :category
          end
       
          # add a foreign key
          execute <<-SQL
            ALTER TABLE products
              ADD CONSTRAINT fk_products_categories
              FOREIGN KEY (category_id)
              REFERENCES categories(id)
          SQL
       
          add_column :users, :home_page_url, :string
          rename_column :users, :email, :email_address
        end
       
        def down
          rename_column :users, :email_address, :email
          remove_column :users, :home_page_url
       
          execute <<-SQL
            ALTER TABLE products
              DROP FOREIGN KEY fk_products_categories
          SQL
       
          drop_table :products
        end
      end

### 4 Izvrsavanje migracija

Izvrsava change ili up metode iz svih migracija koje dosad nisu bile izvrsene (primijenjene)

    rake db:migrate

db:migrate task u pozadini poziva db:schema:dump task koji modifikuje fajl db/schema.rb

izvrsavanje migracija do i ukljucujuci specificnu verziju

    rake db:migrate VERSION=20080906120000

#### 4.1 Ponistavanje migracije

Ponistavanje samo poslednje migracije

    rake db:rollback

Ponistavanje odgovarajuceg broja posljednih migracija

    rake db:rollback STEP=3

Redo za ponistavanje pa ponovno primjenjivanje migracije
  
    rake db:migrate:redo [STEP=3]

#### 4.2 Resetovanje baze podataka

Brisanje baze, njenovo ponovno kreiranje i ucitavanje seme u bazu se postize sledecom komandom:

    rake db:reset

#### 4.3 Pokretanje specificne migracije

    rake db:migrate:up VERSION=20080906120000
    rake db:migrate:down VERSION=20080906120000

#### 4.4 Pokretanje migracije u odredjenoj sredini

    rake db:migrate RAILS_ENV=test

### 5 Modifikovanje postojecih migracija

- Ako je migracija vec izvrsena sama promjena migracije nema efekta. Neophodno je prethodno izvrsiti ponistavanje migracije.

- Ako je migracija javno objavljena u repositorijumu nikakve direktne promjene na migraciji se ne preporucuju. Pravilna procedura ukljucuje kreiranje nove migracije koja ponistava efekte prethodne u slucaju da je to neophodno.


### 7 Sema baze podataka

Preporucen nacin za aktiviranje nove aplikacije nije izvrsavanje svih migracija od prve do poslednje. Preporucen nacin (koji je ujedno i brzi i jednostavniji) je ucitavanje trenutne seme u bazu podataka. Ovaj pristup se primjenjuje u pozadini pri kreiranju testne baze podataka na osnovu seme definisane u db/schema.rb ili db/structure.sql 

Za kreiranje baze i ucitavanje seme na serveru u produkciji moze se koristiti sledeca komanda:

    RAILS_ENV=production rake db:create db:schema:load
    
U Rails aplikaciji moguce je podesiti tip seme baze podataka

U config/application.rb izvrsiti editovanje sljedeceg podesavanja:

    config.active_record.schema_format = :sql

ili

    config.active_record.schema_format = :ruby


### 8 Active Record i Referencijalni Integritet

Pristup koji ActiveRecord zagovara je da sva logika povezana sa modelima treba da bude zajedno sa modelima a ne u bazi. Samim tim mehanizmi kao sto su trigger ili restrikcije na stranim kljucevima koje smjestaju tu logiku u bazu podataka nisu cesto korisceni.

Validacije i :dependent opcije su mehanizmi pomocu kojih modeli omogucavaju integritet podataka. Medjutim kako ovi mehanizmi funkcionisu na aplikacionom nivou oni ne mogu u potpunosti garantovati referencijalni integritet i zato u odredjenim slucajevima je neophodno generisati dodatna pravila i restrikcije u bazi (npr. restrikcije stranog kljuca).
