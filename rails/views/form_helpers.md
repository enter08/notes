## Form Helpers

Ovo uputstvo ce vam omoguciti da naucite kako da:

- generisete forme za pretragu kao i druge slicne forme koje nisu povezane sa modelima u bazi podataka
- generisete forme pomocu kojih se moze manipulisati modelima koji se cuvaju u bazi podataka
- generisete select kontrole
- koristite ugradjene rails mehanizme za rad sa datumima i vremenom
- koristite mehanizme za slanje fajlova na udaljeni server
- generisete forme koje interaguju sa externim servisima i resursima 
- generisete kompleksne forme

Vise informacija vezano za sintaksu metoda za generisanje formi i elemenata forme mozete naci na sledecem [linku](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html)

### 1 Rad sa jednostavnim formama

Osnovni tag je **form_tag** koji generise sledeci HTML kod u slucaju kada je trenutna strana /home/index

Primjer:

    <%= form_tag do %>
      Form contents
    <% end %>

Generisani HTML:

    <form accept-charset="UTF-8" action="/home/index" method="post">
      <div style="margin:0;padding:0">
        <input name="utf8" type="hidden" value="&#x2713;" />
        <input name="authenticity_token" type="hidden" value="f755bb0ed134b7..." />
      </div>
      Form contents
    </form>

Generisani HTML kod sadrzi i skriveni div koji ima dva inputa polja: **utf8** i **authenticity\_token**. Prvi sluzi da definise format za kodiranje forme dok drugi predstavlja zastitni mehanizam od cross-site request forgery tipa napada. **Utf8** polje se generise za forme i sa GET i POST akcijom dok **authenticity_token** polje se koristi samo u formama sa POST akcijom.

#### 1.1 Genericka forma za pretragu

Najcesce upotrebljavani vid forme je forma za pretragu. Ona najcesce sadrzi:

- HTML form elemenat sa GET metodom,
- Labelu za input elemenat,
- Tekst input elemenat, i
- Elemenat za slanje forme

Ovaj kod:

<pre><code><%= form_tag("/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
</code></pre>

Generise sledecu HTML formu:

    <form accept-charset="UTF-8" action="/search" method="get">
        <label for="q">Search for:</label>
        <input id="q" name="q" type="text" />
        <input name="commit" type="submit" value="Search" />
    </form>

Za rad sa formama za pretragu preporucuje se koriscenje GET metoda. Na ovaj nacin korisnici mogu uz koriscenje bookmark mehanizma sacuvati link za odredjene pretrage radi kasnijeg ponovnog koriscenja.

#### 1.2 Sintaksa metoda za generisanje formi

**form_tag** metod prihvata dva argumenta: *putanju do odredjene akcije* i *hes strukturu koja sadrzi ostale opcije*. Opcije ukljucuju HTTP metod pomocu kojeg ce forma biti upucena iz brovzera ka serveru, kao i druge HTML opcije koje uticu na sadrzaj i izgled elemenata koji su dio HTML forme. 

Kao i u slucaju *link_to* pomocnog metoda prvi argument *form_tag* metoda ne mora da bude putanja u string formatu, to moze da bude i hes sa URL parametrima koje Rails zna kako da konvertuje u putanju 

**Napomena**: voditi racuna o zagradama u slucaju kada se i putanja i opcije prosledjuju kao hes.

    form_tag(controller: "people", action: "search", method: "get", class: "nifty_form")
    # => '<form accept-charset="UTF-8" action="/people/search?method=get&class=nifty_form" method="post">'

    form_tag({controller: "people", action: "search"}, method: "get", class: "nifty_form")
    # => '<form accept-charset="UTF-8" action="/people/search" method="get" class="nifty_form">'

#### 1.3 Ostali metodi za generisanje elemenata HTML formi

Rails posjeduje niz metoda koji omogucavaju generisanje elemenata HTML formi kao sto su: checkbox kontrole, tekst polja, radio dugmad. Imena ovih metoda su izvedena iz imena samog elementa kojeg generisu, sa sufiksom **\_tag** (npr. *text_field_tag* i *check_box_tag*). 

Prvi argument ovih metoda je ime koje ce biti povezano sa podacima iz odgovarajuceg elementa (npr. **text_field_tag(:query)**). Na serveru se tim podacima moze pristupiti na sledeci nacin **params[:query]**.

Primjer koriscenja radio dugmadi:

    <%= radio_button_tag(:age, "child") %>
    <%= label_tag(:age_child, "I am younger than 21") %>
    <%= radio_button_tag(:age, "adult") %>
    <%= label_tag(:age_adult, "I'm over 21") %>

Rezultat:

    <input id="age_child" name="age" type="radio" value="child" />
    <label for="age_child">I am younger than 21</label>
    <input id="age_adult" name="age" type="radio" value="adult" />
    <label for="age_adult">I'm over 21</label>

Napomena: Uvijek koristite labele za checkbox-ove i radio dugmad. Labele povezuju tekst sa odgovarajucim kontrolama i prosiriju region na koji korisnik moze da klikne kako bi interagovao sa odgovarajucom HTML kontrolom.


Primjer poziva nekih od cesce upotrebljavanih HTML kontrola:

    <%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
    <%= password_field_tag(:password) %>
    <%= hidden_field_tag(:parent_id, "5") %>
    <%= search_field(:user, :name) %>
    <%= telephone_field(:user, :phone) %>
    <%= date_field(:user, :born_on) %>
    <%= datetime_field(:user, :meeting_time) %>
    <%= datetime_local_field(:user, :graduation_day) %>
    <%= month_field(:user, :birthday_month) %>
    <%= week_field(:user, :birthday_week) %>
    <%= url_field(:user, :homepage) %>
    <%= email_field(:user, :address) %>
    <%= color_field(:user, :favorite_color) %>
    <%= time_field(:task, :started_at) %>

Odgovarajuci HTML elementi koji prethodni metodi generisu:

    <textarea id="message" name="message" cols="24" rows="6">Hi, nice site</textarea>
    <input id="password" name="password" type="password" />
    <input id="parent_id" name="parent_id" type="hidden" value="5" />
    <input id="user_name" name="user[name]" type="search" />
    <input id="user_phone" name="user[phone]" type="tel" />
    <input id="user_born_on" name="user[born_on]" type="date" />
    <input id="user_meeting_time" name="user[meeting_time]" type="datetime" />
    <input id="user_graduation_day" name="user[graduation_day]" type="datetime-local" />
    <input id="user_birthday_month" name="user[birthday_month]" type="month" />
    <input id="user_birthday_week" name="user[birthday_week]" type="week" />
    <input id="user_homepage" name="user[homepage]" type="url" />
    <input id="user_address" name="user[address]" type="email" />
    <input id="user_favorite_color" name="user[favorite_color]" type="color" value="#000000" />
    <input id="task_started_at" name="task[started_at]" type="time" />

[Spisak svih metoda (29)](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html):

- button_tag, check_box_tag, color_field_tag, date_field_tag,
- datetime_field_tag, datetime_local_field_tag, email_field_tag,
- field_set_tag, file_field_tag, form_tag, hidden_field_tag,
- image_submit_tag, label_tag, month_field_tag, number_field_tag,
- password_field_tag, phone_field_tag, radio_button_tag,
- range_field_tag, search_field_tag, select_tag, submit_tag,
- telephone_field_tag, text_area_tag, text_field_tag,
- time_field_tag, url_field_tag, utf8_enforcer_tag, week_field_tag

HTML5 metodi iz ove grupe su: search, telephone, date, time, color, datetime, datetime-local, month, week, URL, and email. Njihova upotreba zahtijeva upotrebu dodatnih mehanizama kako bi pravilno funkcionisali na browserima koji nemaju odgovarajucu podrsku za ove elemente.

### 2 Metode za generisanje formi baziranih na modelima

#### 2.1 Pomocni metodi za modele

Imena ovih metoda su izvedena iz odgovarajucih HTML elemenata forme i razlikuju se od prethodno predstavljenih metoda po tome sto nemaju sufiks \_tag. Npr. tekst polje za objekat :person i njegov attribut :name se generise pomocu text_field(:person, :name). Ovaj poziv generise sledeci HTML kod:

    <input id="person_name" name="person[name]" type="text" value="Henry"/>

Pretpostavka je da kontroler generise @person objekat a da rezultat poziva metoda @person.name je Henry. 

Napomena: prvi argument ovih metoda mora biti ime objekta (npr. :person ili 'person') a ne sami objekat.

#### 2.2 Povezivanje forme sa odgovarajucim modelom ([form_for](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html))

Povezivanje forme sa odgovarajucim modelom omogucava izbjegavanje navodjenja imena objekta pri pozivu metoda iz prethodne sekcije. Ovo se postize uz koriscenje metoda **form_for**.

Ovaj primjer:

    <%= form_for @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
      <%= f.text_field :title %>
      <%= f.text_area :body, size: "60x12" %>
      <%= f.submit "Create" %>
    <% end %>

Generise sledeci HTML kod:

    <form accept-charset="UTF-8" action="/articles/create" method="post" class="nifty_form">
      <input id="article_title" name="article[title]" type="text" />
      <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
      <input name="commit" type="submit" value="Create" />
    </form>

Za primijetiti:

- @article predstavlja objekat nad kojim se vrsi manipulacija
- ime objekta za koji se forma vezuje koristi se za generisanje imena odgovarajucih HTML kontrola
- [form_for metod](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for) prihvata jedan hash koji sadrzi vise opcija (*:url* se koristi za url opcije, *:html* za html opcije, a postoji i *:namespace* opcija koja sluzi za diferenciranje imena polja forme u slucaju da vise slicnih postoji na istoj strani)
- form_for metod generise kao yield argument objekat koji je graditelj forme (*f* varijabla)
- metodi za generisanje odgovarajucih kontrola se pozivaju na objektu koji je graditelj forme
- sintaksa metoda za generisanje odgovarajucih kontrola je identicna kao kod metoda koji se pozivaju direktno na modelima, jedina razlika je u tome sto nije potrebno navoditi prvi argument koji je ime modela


Metod **fields_for** omogucava generisanje HTML kontrola bez form elemenata. Ovo je korisno u slucajevima kada je neophodno generisati HTML kontrole za povezane modele.

Primjer:
  
    <%= form_for @person, url: {action: "create"} do |person_form| %>
      <%= person_form.text_field :name %>
      <%= fields_for @person.contact_detail do |contact_details_form| %>
        <%= contact_details_form.text_field :phone_number %>
      <% end %>
    <% end %>

Rezultira:

    <form accept-charset="UTF-8" action="/people/create" class="new_person" id="new_person" method="post">
      <input id="person_name" name="person[name]" type="text" />
      <input id="contact_detail_phone_number" name="contact_detail[phone_number]" type="text" />
    </form>

#### 2.3 Dostupne precice uz koriscenje identifikacije rekorda


    ## Creating a new article
    # long-style:
    form_for(@article, url: articles_path)
    # same thing, short-style (record identification gets used):
    form_for(@article)
     
    ## Editing an existing article
    # long-style:
    form_for(@article, url: article_path(@article), html: {method: "patch"})
    # short-style:
    form_for(@article)


Rails konvencije:

- form_for(@article) se moze koristiti bez obzira na to da li se radi o kreiranju novog ili mijenjaju postojeceg modela. Razlog je to sto form_for metod ima ugradjenu funkcionalnost gdje uz koriscenje metoda new_record? moze da utvrdi o kojem se slucaju radi i da generise odgovarajucu varijantu forme.
 
- Rails ce da dodijeli odgovarajuce vrijednosti za class i id atribute: forma koja kreira novu instancu nekog modela ce da ima id i klasu new_model_name. U slucaju da forma sluzi za mijenjanje vec postojece instance nekog modela onda ce klasa biti edit_model_name a id edit_model_name_id

- U slucaju koriscenja ruta i mehanizma prostora imena za rute neophodno je definisati i ime prostora imena pri pozivu form_for metode (form_for [:admin, @article]). Slanje forme generisane prethodnim pozivom form_for metode ce rezultirati izvrsavanjem metoda na ArticlesController-u unutar admin prostora imena (uz koriscenje admin_article_path(@article) rute u slucaju mijenjanja vec postojece instance modela Article).

#### 2.4 Kako rade forme sa PATCH, PUT i DELETE metodima?

Vecina brovzera ne podrzava metode koji nisu GET ili POST. Kako Rails preporucuje i podrzava REST principe izrade web aplikacija to ukljucuje i koriscenje drugih HTTP metoda (HTTP verbs) kao sto su PATCH, PUT i DELETE.

Rails resenje za ovaj problem je u tunelovanju svih metoda kroz POST HTTP zahtjeve. To se postize time sto se odgovarajuci metod proslijedi kao vrijednost sakrivenog polja u samoj formi. Na serveru se nakon parsiranja detektuje ovo polje i njegova vrijednost. Rails tretira ostale podatke na nacin predvidjen za metod koji je prosledjen u datom polju a ne kao POST zahtjev.

Primjer: 

    form_tag(search_path, method: "patch")

Generise:

    <form accept-charset="UTF-8" action="/search" method="post">
      <div style="margin:0;padding:0">
        <input name="_method" type="hidden" value="patch" />
        <input name="utf8" type="hidden" value="&#x2713;" />
        <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c..." />
      </div>
      ... 
    </form>

### 3 Generisanje Select kontrola

HTML Select elemenat zahtijeva dodatnu podrsku kako bi mogao biti generisan sa slicnom lakocom kao i ostali elementi forme.

Rails nudi sledece metode:

Metodi koji nisu vezani za model:

- select_tag(field_name, html_string_of_options)
- options_for_select(array_of_pairs_name_and_id)
- options_from_collection_for_select(collection, value_method, text_method)

Metodi koji su vezani za model:

- select                    # select_tag + options_for_select
- collection_select         # select_tag + options_from_collection_for_select


Primjeri koriscenja:

    <%= select_tag(:city_id, '<option value="1">Lisbon</option>...') %>

    <%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...], 2) %>
     
    output:
     
    <option value="1">Lisbon</option>
    <option value="2" selected="selected">Madrid</option>
    ...

    <%= select_tag(:city_id, options_for_select(...)) %>

Ako je neophodno definisati dodatne atribute na samim elementima opcija moze se koristiti sledeca sintaksa:

    <%= options_for_select([['Lisbon', 1, {'data-size' => '2.8 million'}], ['Madrid', 2, {'data-size' => '3.2 million'}]], 2) %>
     
    output:
     
    <option value="1" data-size="2.8 million">Lisbon</option>
    <option value="2" selected="selected" data-size="3.2 million">Madrid</option>

Za generisanje opcija iz kolekcije predvidjen je sledeci metod:

    <%= options_from_collection_for_select(City.all, :id, :name) %> 


Primjeri koriscenja metoda koji su **vezani za modele**:

    # controller:
    @person = Person.new(city_id: 2)

Select koji kombinuje pozive metoda *select_tag* i *options_for_select*

    <%= select(:person, :city_id, [['Lisbon', 1], ['Madrid', 2], ...]) %>
    

Collection_select koji kombinuje pozive metoda *select_tag* i *options_from_collection_for_select*

    <%= collection_select(:person, :city_id, City.all, :id, :name) %>



### 4 Metodi za generisanje Date and Time HTML kontrola

Metodi iz ove grupe su specificni iz dva razloga:

- Njihova reprezentacija u HTML kodu sadrzi vise polja za unos (npr. godina, mjesec, dan) tako da u params hash-u postoji vise varijabli povezanih sa odredjenim datumom ili vremenom
- Ostali metodi imaju varijante sa _tag sufiksom koji predstavljaju varijante metoda koje ne rade sa modelima dok one bez tog sufiksa rade sa modelima. Metode iz ove grupe koje ne rade sa modelima imaju nazive sa **select_** prefix-om (npr. select_date, select_time) dok one koje rade sa modelima imaju nazive sa **_select** sufiksom (npr. date_select, time_select).

Primjeri koriscenja:

    <%= select_date Date.today, prefix: :start_date %>
    # default prefix: :date
    
    # output

    <select id="start_date_year" name="start_date[year]"> ... </select>
    <select id="start_date_month" name="start_date[month]"> ... </select>
    <select id="start_date_day" name="start_date[day]"> ... </select>

    # using in controller
    Date.civil(params[:start_date][:year].to_i, params[:start_date][:month].to_i, params[:start_date][:day].to_i)

Primjeri koriscenja metoda vezanih za modele:

    <%= date_select :person, :birth_date %>

    # this generates following HTML output

    <select id="person_birth_date_1i" name="person[birth_date(1i)]"> ... </select>
    <select id="person_birth_date_2i" name="person[birth_date(2i)]"> ... </select>
    <select id="person_birth_date_3i" name="person[birth_date(3i)]"> ... </select>

    # this generates following params hash
    
    {:person => {'birth_date(1i)' => '2008', 'birth_date(2i)' => '11', 'birth_date(3i)' => '22'}}

Vise informacija na sledecem [linku](http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html)

### 5 Slanje fajlova

Najvaznija stvar kod slanja fajlova je da encoding atribut forme mora biti *multipart/form-data*. form_for metoda radi ovo automatski u pozadini za nas. U slucaju koriscenja metoda form_tag mi moramo da vodimo racuna o ovome.
   
    <%= form_tag({action: :upload}, multipart: true) do %>
      <%= file_field_tag 'picture' %>
    <% end %>
     
    <%= form_for @person do |f| %>
      <%= f.file_field :picture %>
    <% end %>

    # access to data in params with
    params[:person][:picture]

Velicina fajla utice na to kako ce se procesirati u pozadini na serveru. Poslati fajl moze biti predstavljen kao instanca klase StringIO ili File. U oba slucaja moguce je pristupiti originalnom imenu fajla kao i njegovom tipu kroz metode **original_filename** i **content_type**.

    def upload
      uploaded_io = params[:person][:picture]
      File.open(Rails.root.join('public', 'uploads', uploaded_io.original_filename), 'wb') do |file|
        file.write(uploaded_io.read)
      end
    end

Postoje dva poznata gem-a [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) i [Paperclip](https://github.com/thoughtbot/paperclip) koja omogucavaju napredne operacije sa upload-ovanim fajlovima.

### 6 Modifikovanje Form Builder-a

Ovo kod:

    <%= form_for @person do |f| %>
      <%= text_field_with_label f, :first_name %>
    <% end %>

Moze se generisati na sledeci nacin:

    <%= form_for @person, builder: LabellingFormBuilder do |f| %>
      <%= f.text_field :first_name %>
    <% end %>

    class LabellingFormBuilder < ActionView::Helpers::FormBuilder
      def text_field(attribute, options={})
        label(attribute) + super
      end
    end

### 7 Konvencije za imenovanje parametara

#### 7.1 Osnovne strukture

Primjer 1:

    <input id="person_name" name="person[name]" type="text" value="Henry"/>
  
    # generated hash
    {'person' => {'name' => 'Henry'}}

    # access through params hash
    params[:person][:name]

Primjer 2:

    <input id="person_address_city" name="person[address][city]" type="text" value="New York"/>

    {'person' => {'address' => {'city' => 'New York'}}}

U slucaju kada vise parametara ima isto ime Rails ih ignorise. Medjutim u slucaju kada ime parametra sadrzi [] Rails ce akumilirati vrijednosti u niz. 

    <input name="person[phone_number][]" type="text"/>
    <input name="person[phone_number][]" type="text"/>
    <input name="person[phone_number][]" type="text"/>

    # access to data through params hash
    params[:person][:phone_number]       # this will return an array

Moguce je kombinovati hash-ove i nizove.

Sledeci primjer generise sta?

    <input name="addresses[][line1]" type="text"/>
    <input name="addresses[][line2]" type="text"/>
    <input name="addresses[][city]" type="text"/>


Uz koriscenje pomocnih metoda za generisanje formi:

    <%= form_for @person do |person_form| %>
      <%= person_form.text_field :name %>
      <% @person.addresses.each do |address| %>
        <%= person_form.fields_for address, index: address do |address_form|%>
          <%= address_form.text_field :city %>
        <% end %>
      <% end %>
    <% end %>

Generisemo sledecu formu:

    <form accept-charset="UTF-8" action="/people/1" class="edit_person" id="edit_person_1" method="post">
      <input id="person_name" name="person[name]" type="text" />
      <input id="person_address_23_city" name="person[address][23][city]" type="text" />
      <input id="person_address_45_city" name="person[address][45][city]" type="text" />
    </form>
    
A kada popunimo formu i posaljemo je na server ona ce rezultirati sledecim hashom:

    {'person' => {'name' => 'Bob', 'address' => {'23' => {'city' => 'Paris'}, '45' => {'city' => 'London'}}}}


### 8 Forme ka eksternim resursima

    <%= form_tag 'http://farfar.away/form', authenticity_token: false) do %>
      Form contents
    <% end %>

ili

    <%= form_for @invoice, url: external_url, authenticity_token: 'external_token' do |f| %>
      Form contents
    <% end %>

### 9 Kreiranje kompleksnih formi

Pod terminom kompleksna forma misli se na forme koje osim sto omogucavaju mijenjanje modela na kojem su baziranje takodje omogucavaju i mijenjanje povezanih modela. Primjer: kada kreiramo objekat klase Person zelimo da na istoj formi omogucimo definisanje vise adresa (kuca, posao, i sl.). Kasnije kad vrsimo promjene nad tim objektom klase Person trebalo bi da budemo u mogucnosti da dodamo, izbrisemo ili promijenimo bilo koju od adresa.


Postavka:

    class Person < ActiveRecord::Base
      has_many :addresses
      accepts_nested_attributes_for :addresses
    end
     
    class Address < ActiveRecord::Base
      belongs_to :person
    end

Forma:

    <%= form_for @person do |f| %>
      Addresses:
      <ul>
        <%= f.fields_for :addresses do |addresses_form| %>
          <li>
            <%= addresses_form.label :kind %>
            <%= addresses_form.text_field :kind %>
     
            <%= addresses_form.label :street %>
            <%= addresses_form.text_field :street %>
            ...
          </li>
        <% end %>
      </ul>
    <% end %>

fields_for ce generisati ugnjezdeni blok onoliko puta koliko ima adresa. U slucaju da ne postoji nijedna on nece prikaziti nijedno polje za adresu. Da bi korisnik mogao da vidi polja i da unese odgovarajuce vrijednosti cesto se kreira novi prazan rekord.

    def new
      @person = Person.new
      3.times { @person.addresses.build}
    end

fields_for generise imena kontrola tako da odgovaraju onom formatu koji **accepts_nested_attributes_for** ocekuje.

    {
        :person => {
            :name => 'John Doe',
            :addresses_attributes => {
                '0' => {
                    :kind  => 'Home',
                    :street => '221b Baker Street',
                },
                '1' => {
                    :kind => 'Office',
                    :street => '31 Spooner Street'
                }
            }
        }
    }

Kljucevi u addresses_attributes nemaju nikakav znacaj. Bitno je da su razliciti za svaku od adresa.


    def create
      @person = Person.new(person_params)
      # ...
    end
     
    private
    def person_params
      params.require(:person).permit(:name, addresses_attributes: [:id, :kind, :street])
    end

Brisanje objekata:

    class Person < ActiveRecord::Base
      has_many :addresses
      accepts_nested_attributes_for :addresses, allow_destroy: true
    end

    <%= form_for @person do |f| %>
      Addresses:
      <ul>
        <%= f.fields_for :addresses do |addresses_form| %>
          <li>
            <%= check_box :_destroy%>
            <%= addresses_form.label :kind %>
            <%= addresses_form.text_field :kind %>
            ...
          </li>
        <% end %>
      </ul>
    <% end %>

    def person_params
      params.require(:person).
        permit(:name, addresses_attributes: [:id, :kind, :street, :_destroy])
    end

Sprecavanje praznih rekorda:

    class Person < ActiveRecord::Base
      has_many :addresses
      accepts_nested_attributes_for :addresses, reject_if: lambda {|attributes| attributes['kind'].blank?}
    end