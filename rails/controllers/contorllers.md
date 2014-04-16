#1 Sta radi controller

Action Controller predstavlja C u MVC. Nakon sto rutiranje odredi koji kontroler ce koristiti za  
request, nas kontroler je odgovoran da izgradi smisao iz request-a i da ogovarajuci output.

U vecini RESTful aplikacija, kontroler ce da prihvati request (ovo je nevidljivo za nas), dohvati  
ili sacuva podatke iz modela i korisiti view da kreira HTML output.

Tako da o kontroleru mozemo razmisljati kao o posredniku izmedju modela i view-a. On omogucava da  podaci iz modela budu vidljivi view-u tako da moze prikazati te podatke korisniku, cuva ili update-uje podatke korisnika u model. 

#2 Konvencija imenovanja kontrolera

Po konvenciji dodjele imena Rails kontrolerima favorizuje se pluralizacija zadnje rijeci od imena  
kontrolera, iako se ne zahtijeva izricito (ApplicationController). Npr. preferira se  
ClientsController nad ClientController, SiteAdminsController nad SiteAdminController ili  
SitesAdminsController itd.  

#3 Metode i akcije

Kontroler je Ruby klasa koja nasledjuje ApplicationController i ima metode kao svaka druga klasa.  
Kada nasa aplikacija dobije request, rutiranje odredjuje koji kontroler i koja akcija se pozivaju,  
tada Rails kreira instancu tog kotrolera i pokrece metodu istog imena kao i akcija.

    class ClientsController < ApplicationController
      def new
      end
    end

Npr, ako korisnik otvori `/clients/new` nase aplikacije da bi dodao novog klijenta, Rails pravi  
instancu ClientsController i pokrece **new** metodu. Metoda new moze da view-u ucini dostupnu  
promjenljivu @client kreiranjem novog Client-a.

    def new
      @client = Client.new
    end

ApplicationController nasledjuje ActonController::Base, koji definise niz korisnih metoda.  
Samo *public* metode se mogu pozivati kao akcije. Smatra se kao *good practice* smanjiti  
vidljivost metoda cija namjera nije da budu akcije, kao helper metode i sl.

#4 Parametri

Postoje dvije vrste parametara u web aplikacijama. Prvi su parametri koji se salju kao  
dio URL-a, zovu se **query string** parametri. Query string je sve posle **?** u URL-u  
Drugi tip parametara su POST podaci. Ova informacija obicno se salje sa HTML forme  
koja je popunjena od strane korisinika. Zove se POST podaci jer mogu biti poslati samo kao  
dio HTTP POST zahtjeva. Rails ne pravi razliku izmedju query string parametara i POST  
parametara, i oba tipa parametara su dostupni u params hash-u naseg kontrolera.

    class ClientsController < ApplicationController
      # This action uses query string parameters because it gets run
      # by an HTTP GET request, but this does not make any difference
      # to the way in which the parameters are accessed. The URL for
      # this action would look like this in order to list activated
      # clients: /clients?status=activated
      def index
        if params[:status] == "activated"
          @clients = Client.activated
        else
          @clients = Client.inactivated
        end
      end
 
      # This action uses POST parameters. They are most likely coming
      # from an HTML form which the user has submitted. The URL for
      # this RESTful request will be "/clients", and the data will be
      # sent as part of the request body.
      def create
        @client = Client.new(params[:client])
        if @client.save
          redirect_to @client
        else
          # This line overrides the default rendering behavior, which
          # would have been to render the "create" view.
          render "new"
        end
      end
    end

###4.1 Hash i Array parametri

params hash nije ogranicen na jednodimenzionalne key value vrijednosti. Moze da sadrzi nizove kao  i ugnjezdeni hash. Da bi poslali niz vrijednosti, mora se dodati par zagrada *[]* imenu kljuca: 

    GET /clients?ids[]=1&ids[]=2&ids[]=3

Vrijednost **params[:ids]** ce sada biti **["1", "2", "3"]**. Treba primijetiti da su vrijednosti  
uvijek stringovi.

Da bi poslali hash potrebno je ukljuciti ime kljuca u zagradama:

    <form accept-charset="UTF-8" action="/clients" method="post">
      <input type="text" name="client[name]" value="Acme" />
      <input type="text" name="client[phone]" value="12345" />
      <input type="text" name="client[address][postcode]" value="12345" />
      <input type="text" name="client[address][city]" value="Carrot City" />
    </form>

Kada se izvrsi submit forme, vrijednost params[:clients] ce biti { "name" => "Acme",  
"phone" => "12345", "adress" => { "postcode" => "12345", "city => "Carrot City"}}.
Pimijetimo ugnjezdeni hash u params[:client][:address].

Params hash je zapravo instanca ActiveSupport::HashWithIndifferentAccess, koji funkcionishe  
kao hash ali dopsuta da se koriste simboli i stringovi.

###4.2 JSON parametri

Parametri se mogu zadati u JSON formatu. Rails ce po automatizmu izvrsiti konverziju u params  
hash.

    { "company": { "name": "acme", "address": "123 Carrot Street" } }

Imacemo params[:company] kao { "name" => "acme", "adress" => "123 Carrot Street" }

Takodje ako je ukljucen **config.wrap_parameters** ili pozivom **wrap_parameters** unutar kontrolera mozemo preskociti root element JSON parametra. Parametri ce biti klonirani i ukljuceni u kljuc srazmjerno imenu kontrolera. Tako da gorepomenuti parametar moze biti zapisan kao :

    { "name": "acme", "address": "123 Carrot Street" }

Pretpostavimo da se podaci salju CompaniesController-u, bice ukljuceni u :company na sljedeci  
nacin:

    { name: "acme", address: "123 Carrot Street", company: { name: "acme", address: "123 Carrot Street" } }


###4.3 Routing parametri

Params hash ce uijek sadrzati :controller i :action kljuceve, ali je pozeljno koristiti metode  
*controller_name* i *action_name* umjesto pristupanja ovim vrijednostima. Svaki drugi parametar  
definisan rutiranjem kao :id ce takodje biti dostupan. Za primjer uzimamo listu klijenata  
gdje se moze prikazati aktivni ili neaktivni clanovi. Mozemo dodati rutu koja dohvata :status  
parametar.

    get '/clients/:status' => 'clients#index', foo: 'bar'

U ovom sluaju kada korisnik otvori URL /clients/active, params[:status] ce biti postavljen  
na "active". Kada se ova ruta koristi, params[:foo] ce biti postavljena na "bar" kao da je  
proslijedjena kao query string. Na isti nacin params[:action] ce sadrzati "index".

###4.4 default_url_options

Moguce je postaviti globalne default parametre za URL generisanje, definisanjem metode pod  
imenom `default_url_options` u kontroleru. Ovakva funkcija mora vratiti hash sa zeljenim  
default vrijednostima ciji kljucevi moraju biti simboli:

    class ApplicationController < ActionController::Base
      def default_url_options
        { locale: I18n.locale }
      end
    end

Ove opcije predstavljaju pocetnu taku kod generisanja URL-ova, tako da ih je moguce prepisati  
opcijama proslijedjenim url_for pozivima.

Ako definisemo default_url_options u ApplicationController-u kao u navedenom primjeru, koristice  
se za svako URL generisanje. Ovaj metod moze biti definisan u nekom odredjenom kontroleru gdje  
ce uticati samo na URL-ove koje se tu generisu.

###4.5 Jaki parametri

U slucaju jakih parametara, ActionController parametri su zabranjeni za upotrebu u Active Model  
masovnim dodjeljivanjem(mass assignements) sve dok ne dobiju dozvolu za to(whitelist). Ovo znaci da se mora donijeti svjesna odluka koje atribute omoguciti za update-ovanje i time sprijeciti ispoljavanje i otkrivanje onoga sto ne bi trebalo.

    class PeopleController < ActionController::Base
      # This will raise an ActiveModel::ForbiddenAttributes exception
      # because it's using mass assignment without an explicit permit
      # step.
      def create
        Person.create(params[:person])
      end
 
      # This will pass with flying colors as long as there's a person key
      # in the parameters, otherwise it'll raise a
      # ActionController::ParameterMissing exception, which will get
      # caught by ActionController::Base and turned into that 400 Bad
      # Request reply.
      def update
        person = current_account.people.find(params[:id])
        person.update_attributes!(person_params)
        redirect_to person
      end
 
      private
        # Using a private method to encapsulate the permissible parameters
        # is just a good pattern since you'll be able to reuse the same
        # permit list between create and update. Also, you can specialize
        # this method with per-user checking of permissible attributes.
        def person_params
          params.require(:person).permit(:name, :age)
        end
    end  

####4.5.1 Permitted Scalar values

Za 

    params.permit(:id)

kljuc :id ce dobiti dozvolu(whitelisting) ako se pojavi u params i ako ima pridruzenu permitted  
scalar value. U suprotnom ce kljuc biti izfiltriran, tako da nizovi hash-evi i ostali objekti  
ne mogu biti umetnuti.

Permitterd scalar tipovi su String, Symbol, NilClass, Numeric, TrueClass, FalseClass, Date, Time,  
DateTime, StringIO, IO, ActionDiscpatch::Http::UploadedFile i Rack::Test::UploadedFile.

Da bi deklarisali da vrijednost u params mora biti niz permitted scalar vrijednosti pridruzimo  
kljuc praznom stringu: 

    params.permit(id: [])

Za dodjelu dozvole(whitelist) cijelom hash-u parametara, koristi se metoda permit!.

    params.require(:log_entry).permit!

Ovo ce markirati :log_entry hash i svaki njegov pod-hash kao *permitted*. 

####4.5.2 Ugnjezdeni parametri

Permit se moze koristiti i na ugnjezdene parametre:

    params.permit(:name, { emails: [] },
              friends: [ :name,
                         { family: [ :name ], hobbies: [] }])


Ova deklaracija daje dozvolu za atribute name, emails, friends. 

####4.5.3 Dodatni primjeri

Kod upotrebe permitted atributa u new akciji nastaje problem, sto ne mozemo koristiti require  
na root kljucu jer u tom trenutku pozivanja new ne postoji.

    # using `fetch` you can supply a default and use
    # the Strong Parameters API from there.
    params.fetch(:blog, {}).permit(:title, :author)

accepts_nested_attributes_for:

    class Author < ActiveRecord::Base
      has_many :books
      accepts_nested_attributes_for :books, allow_destroy: true
    end

accepts_nested_attributes_for dozvoljava da radimo update i destroy povezanih zapisa. Zasnovano  
na id i _destroy parametrima:

    # permit :id and :_destroy
    params.require(:author).permit(:name, books_attributes: [:title, :id, :_destroy])

Hash-evi sa integer kljucevima se tretiraju drugacije i moguce je deklarisati atribute kao  
direktne potomke. Ovakav tip parametara se dobija upotrebom accepts_nested_attributes_for  
u kombinaciji sa has_many vezom.

    # To whitelist the following data:
    # {"book" => {"title" => "Some Book",
    #             "chapters_attributes" => { "1" => {"title" => "First Chapter"},
    #                                        "2" => {"title" => "Second Chapter"}}}}
 
    params.require(:book).permit(:title, chapters_attributes: [:title])


#5 Sesije

Aplikacija ima sesiju za svakog korisnika gdje je moguce skladistenje manje kolicine podataka koji 
ce biti ocuvani izmedju zathjeva. Sesija je dostupna samo u kontroleru i view-u i moze koristiti  
jedan od nekoliko mehanizama skladistenja:

+ ActionDispatch::Session::CookieStore - Vrsi cuvanje na klijentu
+ ActionDispatch::Session::CacheStore - Cuva podatke u Rails cache-u
+ ActionDispatch::Session::ActiveRecordStore -Cuva podatke u bazi podataka koristeci ActiveRecord  
+ ActionDispatch::Session::MemCacheStore - Cuva podatke u memcached cluster

Sve sesije koriste cookie za skladistenje jedinstvenog ID-a za svaku sesiju(mora se koristiti cookie, Rails nece dozvoliti da se proslijed ID putem URL-a zbog sigurnosti)

Za vecinu mehanizama skladistenja ovaj ID se koristi za pretragu podataka sesije na serveru, tj  
tabeli baze podataka. Postoji izuzetak a to je default i preporuceni mehanizam skladistenja -  
CookieStore, koji cuva sve podatke sesije u samom cookie-u (ID je i dalje dostupan). Ovaj nacin  
ima prednosti po pitanju tezine i ne-zahtijevanju dodatnog podesavanja u slucaju nove aplikacije i upotrebe sesije. Podaci cookie-ja su kriptografski potpisani da bi bili tamper-proof, ali nije enkriptovana, tako da svako sa pristupom moze citati sadrzaj ali ga ne moze mijenjati(Rails nece prihvatiti u slucaju da je doslo do izmjene)

U cookie-u se moze skladistiti oko 4kB podataka. Skladistenje velike kolicine podataka nije  
preporucljivo nezavisno od primjene. Posebno treba izbjeci skladistenje kompleksnih objekata  
(sve osim osnvnih Ruby objekata) u sesiji.

Ako nase sesije ne slkadiste podatke posebne vaznosti ili se ne zahtijeva trajno postojanje  
ili duze vrijeme, za razmotriti je upotreba ActionDispatch::Session::CacheStore. Ovim nacinom  
se skladiste sesije upotrebom cahce implementacije konfigurisane za nasu aplikaciju. Mana ovakvog  
pristupa je kratkotrajnost sesije.

Ako nam je potreban drugaciji mehanizam skladistenja sesija, to mozemo promijeniti config/initializers/session_store.rb fajlu:

    # Use the database for sessions instead of the cookie-based default,
    # which shouldn't be used to store highly confidential information
    # (create the session table with "rails g active_record:session_migration")
    # YourApp::Application.config.session_store :active_record_store

Rails postavlja kjuc sesije (ime cookie-a) kada se vrsi potpis podataka sesije. Ovo se  
takodje moze mijenjati u config/initializers/session_store.rb:

    # Be sure to restart your server when you modify this file.
    YourApp::Application.config.session_store :cookie_store, key: '_your_app_session'

Moze se proslijediti :domain kljuc da bi se specificirao ime domena za cookie:

    # Be sure to restart your server when you modify this file.
    YourApp::Application.config.session_store :cookie_store, key: '_your_app_session', domain: ".example.com"

Rails postavlja (za CookieStore) tajni kljuc za potpis podataka sesije. Ovo se moze mijenjati  
u config/initializer/secret_token.rb

    # Be sure to restart your server when you modify this file.
 
    # Your secret key for verifying the integrity of signed cookies.
    # If you change this key, all old signed cookies will become invalid!
    # Make sure the secret is at least 30 characters and all random,
    # no regular words or you'll be exposed to dictionary attacks.
    YourApp::Application.config.secret_key_base = '49d3f3de9ed86c74b94ad6bd0...'


###5.1 Pristupanje sesiji

U kontroleru sesijama se moze pristupati preko **session** instance metode.

Vrijednosti sesije se cuvaju koriscenjem kljuc/vrijednost parova kao hash:

    class ApplicationController < ActionController::Base
 
      private
 
      # Finds the User with the ID stored in the session with the key
      # :current_user_id This is a common way to handle user login in
      # a Rails application; logging in sets the session value and
      # logging out removes it.
      def current_user
        @_current_user ||= session[:current_user_id] &&
          User.find_by(id: session[:current_user_id])
      end
    end

Da bi nesto smjestili u sesiji, izvrsimo dodjelu kljucu kao hash:

    class LoginsController < ApplicationController
      # "Create" a login, aka "log the user in"
      def create
        if user = User.authenticate(params[:username], params[:password])
          # Save the user ID in the session so it can be used in
          # subsequent requests
          session[:current_user_id] = user.id
          redirect_to root_url
        end
      end
    end

Da bi izbrisali nesto iz sesije, dodijelimo tom kljucu nil:

    class LoginsController < ApplicationController
      # "Delete" a login, aka "log the user out"
      def destroy
        # Remove the user id from the session
        @_current_user = session[:current_user_id] = nil
        redirect_to root_url
      end
    end

Za resetovanje cijele sesije, koristi se `reset_session`.

###5.2 Flash

Flash je posebni dio sesije koji se "cisti" svakim zahtjevom. Ovo znaci da vrijednosti koje su  
tu sacuvane ce biti dostupneu sljedecem zahtjevu, sto je korisno za prosledjivanje error poruka.  

Pristupa se na slican nacin kao i sesiji, kao hash.

Primjer log-out. Kontroler moze poslati poruku koja ce biti prikazana korisniku na sljedecem  
zahtjevu:

    class LoginsController < ApplicationController
      def destroy
        session[:current_user_id] = nil
        flash[:notice] = "You have successfully logged out."
        redirect_to root_url
      end
    end

Flash poruke je moguce dodijeliti kao dio redirekcije. Moguce je dodijeliti :notice, :alert,  
ili generalno :flash :

    redirect_to root_url, notice: "You have successfully logged out."
    redirect_to root_url, alert: "You're stuck here!"
    redirect_to root_url, flash: { referral_code: 1234 }

Destroy akcija vrsi redirekciju na root_url, gdje ce poruka biti prikazana. U potpunosti je  
na sljedecoj akciji da odluci , sta ce (ako uopste), da radi sa tim sto je prethodna akcija  
stavila u flash. Uobiacajeno je da se prikazuju sve greske od flash-a u layout-u aplikacije:

    <html>
      <!-- <head/> -->
      <body>
        <% flash.each do |name, msg| -%>
          <%= content_tag :div, msg, class: name %>
        <% end -%>
 
      <!-- more content -->
      </body>
    </html> 

Na ovaj nacin, ako neka akcija postavi poruku upozorenja(notice,alert) i sl, layout ce to prikazati.

Mozemo proslijediti sve sto se moze skladistiti u sesiji, nismo ograniceni na poruke upozorenja:

    <% if flash[:just_signed_up] %>
      <p class="welcome">Welcome to our site!</p>
    <% end %>

Ako zelimo da neka flash vrijednost bude prenesena na drugi zahtjev, koristimo `keep` metodu:

    class MainController < ApplicationController
      # Let's say this action corresponds to root_url, but you want
      # all requests here to be redirected to UsersController#index.
      # If an action sets the flash and redirects here, the values
      # would normally be lost when another redirect happens, but you
      # can use 'keep' to make it persist for another request.
      def index
        # Will persist all flash values.
        flash.keep
 
        # You can also use a key to keep only some kind of value.
        # flash.keep(:notice)
        redirect_to users_url
      end
    end

####5.2.1 flash.now

Po default-u dodavanje vrijednosti u flash ce ih uciniti dostupnim u sljedecem zahtjevu, ali  
ponekad zelimo da im pristupimo u istom zahtjevu. Npr. ako create akcija neuspjesno sacuva  
resurs i vrsi se direktno renderovanje *new* templejta, to nece rezultirati novim zahtjevom, ali dalje hocemo da prikazemo poruku uz pomoc flash-a. Ovo se moze postici upotrebom `flash.now`.

    class ClientsController < ApplicationController
      def create
        @client = Client.new(params[:client])
        if @client.save
          # ...
        else
          flash.now[:error] = "Could not save client"
          render action: "new"
        end
      end
    end

#6 Cookies 

Aplikacija moze skladistiti male kolicine podataka na klientu pod imenom **cookies**  
koja ce bit ocuvana kroz zahtjeve cak i sesije. Rails omogucava lak pristup cookie-ima  
preko metode **cookies** koja slicno sesijama radi kao hash.

    class CommentsController < ApplicationController
      def new
        # Auto-fill the commenter's name if it has been stored in a cookie
        @comment = Comment.new(author: cookies[:commenter_name])
      end
 
      def create
        @comment = Comment.new(params[:comment])
        if @comment.save
          flash[:notice] = "Thanks for your comment!"
          if params[:remember_name]
            # Remember the commenter's name.
            cookies[:commenter_name] = @comment.author
          else
            # Delete cookie for the commenter's name cookie, if any.
            cookies.delete(:commenter_name)
          end
          redirect_to @comment.article
        else
          render action: "new"
        end
      end
    end

Dok se kod sesija postavlja vrijednost kljuca na nil, da bi izbrisali cookie vrijednost koristimo  
cookies.delete(:key)

#7 Renderovanje xml i json podataka

ActionController cini lakim renderovanje xml ili json podatak. Ako smo generisali kontroler  
koriscenjem scaffold, izgledace kao:

    class UsersController < ApplicationController
      def index
        @users = User.all
        respond_to do |format|
          format.html # index.html.erb
          format.xml  { render xml: @users}
          format.json { render json: @users}
        end
      end
    end

Da se primijetiti da smo koristili render xml:@users, a ne render xml:@users.to_xml.  
Ako objekat nije string, tada Rails po automatizmu poziva to_xml.

#8 Filteri

Filteri su metode koje se izvrsavaju prije, posle ili "oko" akcije kontrolera.

Filteri se nasledjuju, tako da kada postavimo filter na ApplicationController-u, on ce biti  
pozvan na svakom kontroleru nase aplikacije.

"Before" filteri mogu da zadrze ciklus zahtjeva. Uobicajen before filter je onaj koji zahtijeva  
da je korisnik ulogovan da bi akcija bila izvrsena:

    class ApplicationController < ActionController::Base
      before_action :require_login
 
      private
 
      def require_login
        unless logged_in?
          flash[:error] = "You must be logged in to access this section"
          redirect_to new_login_url # halts request cycle
        end
      end
    end

Metoda cuva error poruku u flash-u i vrsi redirekciju na login formu ako korsinik nije  
ulogovan. Ako neki "before" filter vrsi rendering ili redirekciju, akcija se nece izvrsiti.  
Ako postoje dodatni filteri isplanirani da se izvrsavaju posle tog filtera, njihovo izvrsavanje  
je takodje obstavljeno.

U navedenom primjeru filter je dodat ApplicationController-u pa ga zbog toga svi kontroleri  
aplikacije nasledjuju. Ovo ce uciniti da sve u aplikaciji zahtijeva korisnika da bude ulogovan.
Iz jasnih razloga ne bi trebalo da svi kontroleri zahtijevaju ovakva ogranicenja. Mozemo  
sprijeciti pozivanje filtera za odredjene akcije upotrbom skip_before_action:

    class LoginsController < ApplicationController
      skip_before_action :require_login, only: [:new, :create]
    end

Sada new i create akcije LoginsController-a ce raditi kao ranije bez zahtijevanja korisnika da  
bude ulogovan. :only opcija se koristi da se oznaci koje akcije preskociti odredjenim filterom.  
Postoji :except opcija koja radi u drugom smjeru.

###8.1 After i Around filteri

Mozemo pozivati filtere koji ce se pozivati posle izvrsenja neke akcije ili cak u oba slucaja  
prije i posle.

After filteri su slicni before filterima, ali zbog toga sto je akcija vec ivrsena imaju pristup podacima (odgovor) koje treba poslati klijentu. Ocigledno, after filteri ne mogu da zaustave  
izvrsavanje akcije.

Around filteri se oslanjaju na yield u smislu njihovih odgovarajucih akcija.  

Primjer website-a gdje promjene imaju proceduru odobrenja, administrator bi bio u  
mogucnosti da ih pregleda sa lakocom, samo ih treba uokviriti transakcijom:

    class ChangesController < ApplicationController
      around_action :wrap_in_transaction, only: :show
 
      private
 
      def wrap_in_transaction
        ActiveRecord::Base.transaction do
          begin
            yield
          ensure
            raise ActiveRecord::Rollback
          end
        end
      end
    end

Around filter takodje obuhvata renderovanje. U ovom primjeru sami view cita iz baze podataka  
(via scope), tako ce ciniti unutar transakcije i pritom prikazivati podatke.

###8.2 Ostali nacini za upotrebu filtera

Dok je uobicajen nacin za koriscenje filtera kreiranje **private** metoda i upotreba *_action  
za njihovo dodavanje, postoje jos dva nacina za istu stvar.

Prvi nacin je koriscenje bloka direktno sa *_action metodom. Blok prihvata kontroler kao argument  
a require_login (pomenut u ranijim primjerima) se moze prepraviti da radi sa blokom:

    class ApplicationController < ActionController::Base
      before_action do |controller|
        redirect_to new_login_url unless controller.send(:logged_in?)
      end
    end

Primjetno je da filter u ovom slucaju koristi send jer logged_in? metoda je privatni flter  
se ne izvrsava u opsegu kontrolera. Ovo nije preporucen nacin da se implementira konkretno  
ovaj filter, ali u nekim jednostavnijim slucajevima moze biti od koristi.

Drugi nacin je koriscenjem klase za rukovanje filterom. Ovaj nacin je koristan u slozenijim  
slucajevima koje nije moguce implementirati na jednostavan i *reusable* nacin koriscenjem  
druge dvije metode. Kao primjer navodi se prepravka login filtera za rad sa klasom:

    class ApplicationController < ActionController::Base
      before_action LoginFilter
    end
 
    class LoginFilter
      def self.filter(controller)
        unless controller.send(:logged_in?)
          controller.flash[:error] = "You must be logged in"
          controller.redirect_to controller.new_login_url
        end
      end
    end

Opet, ni ovo nije idealan primjer za ovaj filter, zato sto se ne izvrsava u opsegu kontrolera  
vec dobija kontroler proslijedjen kao argument. Filter klasa ima klasni metod filter koji se izvrsava  prije ili poslije akcije zavsino od njegove definicije (prije akcije u konkretnom primjeru). Klase koje koriste around filter mogu da koriste isti filter metod koji ce se izvrsavati na isti nacin. Mora se pozvati *yield* da bi se akcija izvrsila. Alternativa je imati oba i before i after metod koji se izvrsavaju prije i posle akcije.  

#9 Zastita od laznih zahtjeva

Cross-site request forgery je tip napada kod kog sajt prevari korisnika i nagoni u pravljenje  
zahtjeva na drugi sajt, moguce dodavati, mijenjati ili brisati podatke bez znanja ili dozvole  
korisnika.

Prvi korak u izbjegavanju ovih napada je osigurati da se svim "destruktivnim" akcijama (create, update, destroy) moze pristupiti iskljucivo sa non-GET zahtjevima. Ako pratite RESTful konvencije  
vec to praktikujete. Ipak maliciozni sajt dalje moze poslati non-GET zahtjev vasem sajtu vrlo  
lako, i to je mjesto gdje se ukljuuje zastita od laznih zahtjeva.

Nacin na koji se ovo radi, je dodavanje tokena koji nije lako pogoditi i koji je poznat samo  
vasem serveru za svaki zahtjev. Na ovaj nacin zahtjevima bez odgovarajuceg tokena nece biti  
moguc pristup.

Ako generisemo formu :

    <%= form_for @user do |f| %>
      <%= f.text_field :username %>
      <%= f.text_field :password %>
    <% end %>

token se dodaje u "hidden" polje:

    <form accept-charset="UTF-8" action="/users/1" method="post">
    <input type="hidden"
           value="67250ab105eb5ad10851c00a5621854a23af5489"
           name="authenticity_token"/>
    <!-- fields -->
    </form>

Rails dodaje ovaj token na svaku formu koja je generisana uz pomoc form helpera, tako da vecinu  
vremena ne moramo da brinemo o tome. Medjutim ako formu rucno kreiramo ili nam je potrebno da  
dodamo token iz drugog razloga, dostaupan je pomocu metoda form_authenticity_token:

form_authenticity_token generise validan token. Korisno u situacijama gdje Rails ga ne generise  
automatski.

#10 Request Response objekti

U svakom kontroleru postoje dva pristupna metoda koji pokazuju na request i response objekte  
povezanih sa ciklusom zahtjeva koji je trenutno u izvrsavanju. Request metoda sadrzi instancu  
AbstractRequest i response metoda vraca response objekat koji reprezentuje ono sto ce biti  
poslato nazad ka klijentu.

###10.1 Request objekat

Request objekat sadrzi korisne informacije o zahtjevu koji dolazi od klijenta. U svojstva   
kojima je moguce pristupiti spadaju :

| svojstvo zahtjeva | Svrha   		               |  
| ---- | ----------------------------------------- | 
| host | ime hosta koje se koristi za ovaj zahtjev |                 
| domain(n=2) | Prvih n segmenata hostname-a pocevsi sa desne strane(TLD) |                 
| format |               Tip sadrzaja koji zahtijeva klijent |                  
| method | HTTP metod koji se koristi za ovaj zahtjev |
| get?,post?, patch?,put?,delete,head?| Vraca true ako je HTTP metod GET/POST/PATCH/PUT/DEL/HEAD |
| headers | Vraca hash koji sadrzi hedere povezane sa zahtjevom |
| port | port number (integer) koji se koristi za zahtjev |
| protocol | Vraca string koji sadrzi korisceni protokol plus "://" |
| query_string | Vraca query_string dio URL-a, tj. sve posle "?" |
| remote_ip | IP adresa klijenta |
| url | Cijeli URL koriscen za zahtjev |

####10.1.1 path_parameters, query_parameters i request_parameters

Rails skuplja sve parametre poslate uz zahtjev u params hash-u, bilo da su poslati kao  
query_string ili kao "tijelo" post-a. Request objekat ima tri accessor-a koji omogucavaju pristup  
ovim parametrima zavsino od toga odakle dolaze. Query_parameters hash sadrzi parametre poslate  
kao query_string  dok request_parameters hash sadrzi parametre poslate kao dio post "tijela".  
Path_parameters hash sadrzi parametre koje prepoznaje rutiranje kao dio putanje koja vodi do  
konkretnog kontrolera i akcije.

####10.2 Response objekat

Response objekat se obicno ne korsiti direktno, ali je izgradjen tokom izvrsavanja akcije  
renderovanja podataka koji su poslati nazad korisniku, dok ponekad kao u after filteru moze biti  
korisno pristupati mu direktno. Neke od tih accessor metoda imaju setter-e koji im dozvoljavaju da  promijene njihovu vrijednost.

| svojstvo odgovora |    Svrha                    |
| ------------------|---------------------------- |
| body | String podataka koji se salju nazad klijentu. Najcesce HTML |
| status | HTTP statusni kod odgovora, kao npr 200 za uspjesan zahtjev ili 404 *file not found* |
| location | URL na koji je klijent redirektovan, ako postoji |
| content_type | Tip sadrzaja u odgovoru |
| charset | Character set koji se koristi u odgovoru. Default je "utf-8" |
| headers | Hederi koji se koriste za odgovor |

####10.2.1 Setovanje posebnih hedera

Ako zelimo da postavimo posebne hedere za odgovor tada koristimo response.headers . Headers   
atribut je hash koji mapira imena hedera na njihove vrijednosti, i Rails ce postaviti neke od   
njih po automatizmu. Ako zelimo da dodamo ili promijenimo neke od hedera, dodijelimo u  
response.headers na sljedeci nacin: 

    response.headers["Content-Type"] = "application/pdf"

U ovom slucaju vise smisla ima koristiti content_type sete direktno.

#11 HTTP autentikacija

Rail dolazi sa dva ugradjena mehanizma za HTTP autentikaciju:

+ Basic Authentication
+ Digest Authentication

###11.1 HTTP Basic authentication

HTTP Basic autentikacija je sema autentikacije podrzana od strane veceg broja browser-a.  Za  
primjer uzimamo administrativnu sekciju koja je dostupna samo kad se unese username i password  
u osnovni HTTP dialog window browser-a. Koriscenje ugradjene autentikacije se svodi na koriscenje  jedne metode, http_basic_authenticate_with.

    class AdminsController < ApplicationController
      http_basic_authenticate_with name: "humbaba", password: "5baa61e4"
    end

Sa ovom funkcijom, sada mozemo kreirati kontrolere koji nasledjuju AdminController. Filter  
ce biti pozvan za sve akcije u tim kontrolerima, pri tom ce ih stititi HTTP basic autentikacijom.

###11.2 HTTP Digest authentication

HTTP digest autentikacija je superiorna u odnosu na basic autentikaciju jer ne zahtijeva od  
klijenta da salje ne-enkriptovan pasvord kroz mrezu. Koriscenje digest autentikacije se svodi  
na upootrebu jedne metode i to authenticate_or_request_with_http_digest.

    class AdminsController < ApplicationController
      USERS = { "lifo" => "world" }
 
      before_action :authenticate
 
      private
 
      def authenticate
        authenticate_or_request_with_http_digest do |username|
          USERS[username]
        end
     end
    end

U prethodnom primjeru authenticate_or_request_with_http_digest blok uzima samo jedan argument  
username i blok vrace password. U slucaju da blok vrati nil ili false rezultat je neuspjesna 
autentikacija.

#12 Streaming i File Downloads

Ponekad zelimo da posaljemo fajl korisniku umjesto renderovanja HTML stranice. Svi kontroleri u  
Rails-u imaju send_data i send_file metode, koje vrse stream-uju podatke ka klijentu. send_file  
je pogodnija jer omogucava da se navede ime fajla sa diska ciji ce sadrzaj strem-ovati.

Da bi vrsili stream podataka ka klijentu , koristie send_data:

    require "prawn"
    class ClientsController < ApplicationController
      # Generates a PDF document with information on the client and
      # returns it. The user will get the PDF as a file download.
      def download_pdf
        client = Client.find(params[:id])
        send_data generate_pdf(client),
              filename: "#{client.name}.pdf",
              type: "application/pdf"
      end
 
      private
 
      def generate_pdf(client)
        Prawn::Document.new do
          text client.name, align: :center
          text "Address: #{client.address}"
          text "Email: #{client.email}"
        end.render
      end
     end

Download_pdf akcija u navedenom primjeru ce pozvati privatnu metodu koja generise PDF dokument  
i vraca ga kao string. Ovaj string ce zatim biti stream-ovan do klijenta kao download fajla i  
ime ce biti predlozeno korisniku. Ponekad prilikom stream-ovanja fajlova ka korisniku, mozda ne  
zelite da download-ujete fajl. Npr. slike koje mogu biti ugradjene u HTML stranice. Da bi rekli  
brauzeru da fajl nije namijenjen download-u, mozemo postaviti :disposition opciju kao *inline*.  
Suprotna i default vrijednost za ovu opciju je "attachment".

###12.1 Sending files

Ako zelite da posaljete fajl koji vec postoji na disku, koristite send_file metod

    class ClientsController < ApplicationController
      # Stream a file that has already been generated and stored on disk.
      def download_pdf
        client = Client.find(params[:id])
        send_file("#{Rails.root}/files/clients/#{client.id}.pdf",
              filename: "#{client.name}.pdf",
              type: "application/pdf")
      end
    end

Ovo ce citati i vrsiti stream fajla, izbjegavajuci ucitavanje cijelog fajla u memoriju.  
Mozete iskljuciti strem-ovanje opcijom :stream ili prilagodti velicinu bloka sa :buffer_size  
opcijom.

Ako :type nije specificiran, bice pretpostavljen na osnovu ekstenzije specificirane u :filename.  
Ako tip sadrzaja nije registrovan za tu ekstenziju, koristi se application/octet-stream .

###12.2 RESTful downloads

Iako send_data radi kako treba, ako pravite RESTful aplikaciju sa odvojenim akcijama za download  
obicno nije potrebno. U REST terminologiji, PDF fajl iz pomenutog pimjera se moze smatrati  
kao jos jedna reprezentacija klijentskog resursa. Slijedi primjer koda koji ilustruje laksi nacin  
"RESTful downloada" kao dio show akcije, bez stream-ovanja:

    class ClientsController < ApplicationController
      # The user can request to receive this resource as HTML or PDF.
      def show
        @client = Client.find(params[:id])
 
        respond_to do |format|
          format.html
          format.pdf { render pdf: generate_pdf(@client) }
        end
      end
    end

Sa ciljem da prethodni kod proradi potrebno je dodati PDF MIME tip Rails-u u fajlu  
config/initializers/mime_types.rb:

    Mime::Type.register "application/pdf", :pdf

Sada korisnik moze dobiti PDF verziju klijenta dodavanjem ".pdf" na URL.

    GET /clients/1.pdf


#13 Log filtering

Rails cuva log fajl za svako okruzenje u log folderu. Od izuzetne su koristi i znacaja  
kada debug-ujete sta se zapravo desava sa vasom aplikacijom, dok u stvarnim situacijama  
necete htjeti da bas svaka infomacija bude sacuvana u log fajlu.

####13.1 Parameters filtering

Moguce je filtrirati odredjene parametre iz vasih log fajlova dodavanjem istih u  
config.filter_parameters u konfiguraciju aplikacije. Ovi parametri ce biti oznaceni kao  
[FILTERED] u log-u.

    config.filter_parameters << :password


####13.2 Redirects filtering

Ponekad je pozeljno filtrirati iz vasih log fajlova odredjene osjetljive lokacije na koje  
se vrsi redirekcija u vasoj aplikaciji. To se moze ostvariti sa config.filter_redirect opcijom: 

    config.filter_redirect << 's3.amazonaws.com'

Mozete postaviti kao String,Regexp ili niz od oba:

    config.filter_redirect.concat ['s3.amazonaws.com', /private_path/]

URL koje se budu poklapale ce biti oznacene kao [FILTERED].

#14 Rescue

Vjerovatno ce vasa aplikacija sadrzati bug-ove ili ce u suprotnom bacati exception koji se mora  
handle-ovati. Npr. Ako korisnik posjeti link za resurs koji vise ne postoji u bazi podataka,  
ActiveRecord ce baciti ActiveRecord::RecordNotFound exception.

Rails default obrada izuzetaka prikazuje "500 Server Error" poruku za sve exception-e. Ako je zahtjev kreiran lokalno, Uz pomoc traceback-a dodatne informacije ce biti prikazane , pa je  
lakse saznati sto nije u redu i raditi na tome. Ako je zahtjev remote Rails ce prikazati  
"500 Server Error"  poruku korisniku, ili "404 Not Found" ako je u postojala greska u rutiranju  
ili zapis nije bilo moguce pronaci. Ponekad zelite da prilagodite kako se ove greske hvataju i  
kako su prikazane korisniku. Postoji vise nivoa obrade zuzetaka u rails-u:

###14.1 Deafault 500 and 404 templates

Po default-u aplikacija ce renderovati ili 404 ili 500 error poruke. Ove poruke su sadrzane u  
staticnim HTML fajlovvima u public folderu, 404.html i 500.html. Mozete prilagoditi ove fajlove  
da dodaju neke dodatne informacije. 

###14.2 rescue_from

Da bi vrsili obradu izuzetaka odredjenog tipa koristi se rescue_from, koji vrsi obradu izuzetaka  
u cijelom kontroleru i njegovim podklasama.

Kada se izuzetak  uhvati sa rescue_from direktivom, exception objekat je proslijedjen handler-u.  
Handler moze biti ili metod ili Proc objekat proslijedjen :with opciji. Mozemo koristiti blok  
direktno umjesto eksplicitne upotrebe Proc objekta

Slijedi primjer upotrebe rescue_from da se presretnu sve ActiveRecord::RecordNotFound greske.

    class ApplicationController < ActionController::Base
      rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
 
      private
 
      def record_not_found
        render text: "404 Not Found", status: 404
      end
    end

Npr. moguce je kreirati exception klase koje ce se podizati kada korisnik nema pristup odredjenoj  
sekciji vase aplikacije:

    class ApplicationController < ActionController::Base
      rescue_from User::NotAuthorized, with: :user_not_authorized
 
      private
 
      def user_not_authorized
        flash[:error] = "You don't have access to this section."
       redirect_to :back
     end
    end
 
    class ClientsController < ApplicationController
      # Check that the user has the right authorization to access clients.
      before_action :check_authorization
 
      # Note how the actions don't have to worry about all the auth stuff.
      def edit
        @client = Client.find(params[:id])
      end
 
      private
 
      # If the user is not authorized, just throw the exception.
      def check_authorization
        raise User::NotAuthorized unless current_user.admin?
      end
    end

#15 Force HTTP protocol

Ponakad cete imati zelju da primorate odredjeni kontroler da bude dostupan i kroz HTTPS protokol
iz sigurnosnih razloga. Mozete koristiti force_ssl metod u vasem kontroleru.

    class DinnerController
      force_ssl
    end

Kao i kod filtera moguce je proslijediti :only i :except da bi se osigurala veza samo sa odredjenim
akcijama:

    class DinnerController
      force_ssl only: :cheeseburger
      # or
      force_ssl except: :cheeseburger
    end