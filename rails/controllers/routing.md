# 1 Namjena Rails Router-a

Rails router prepoznaje URL adrese i otprema ih do neke akcije kontrolera. Može i da generiše putanje i URL adrese, izbjegavajući time pisanje stringova u view fajlovima.

[ActionDispatch::Routing](http://api.rubyonrails.org/classes/ActionDispatch/Routing.html)

## 1.1 Povezivanje URL-a sa kodom

Kada Rails aplikacija primi zahtjev za:

<pre><code>GET /patients/17</code></pre>

pita rutera da isto spoji sa nekom kontroler akcijom. Ako je prva spojena ruta (*engl.* matching route):

<pre><code>get '/patients/:id', to: 'patients#show'</code></pre>

zahtjev se šalje kontroler show akciji patient-a sa { id: '17' } u parametrima.

## 1.2. Generisanje putanja i URL adresa iz koda

Ako izmijenimo prethodnu rutu da bude:

<pre><code>get '/patients/:id' to: 'patients#show', as: 'patient'</code></pre>

a aplikacija u kontroleru sadrži sljedeći kod:

<pre><code>@patient = Patient.find(17)</code></pre>

i ovo je odgovarajući view kod:

    <%= link_to 'Patient Record', patient_path(@patient) %>

ruter će generisati putanju <code>/patients/17</code>.

Ovo će dovesti do znatno preglednijeg koda.

# 2 Resurs rutiranje: Rails Default

Resurs rutiranje (*engl.* resource routing) omogućava brzo deklarisanje svih ruta za dati kontroler. Umjesto posebnog deklarisanja ruta za <code>index</code>, <code>show</code>, <code>new</code>, <code>edit</code>, <code>create</code>, <code>update</code> i <code>destroy</code> akcije, *resourceful* ruta ih deklariše u jednoj liniji koda.

## 2 Resursi na mreži

Browser-i zahtijevaju strane od Railsa praveći zahtjev za URL adresama koristeći određeni HTTP metod, kao <code>GET</code>, <code>POST</code>, <code>PATCH</code>, <code>PUT</code> i <code>DELETE</code>. Svaki metod je zahtjev za izvođenje operacije na resursu. Resurs ruta mapira određen broj zahtjeva sa akcijama u jednom kontroleru.

Kada Rails aplikacija primi zahtjeva za:

<pre><code>DELETE /photos/17</code></pre>

pita rutera da isto spoji sa nekom kontroler akcijom. Ako je prva spojena ruta:

<pre><code>resources :photos</code></pre>

Rails bi poslao ovaj zahtjev destroy metodi, photos kontroleru sa { id: '17' } u parametrima.

## 2.2 CRUD, metode i akcije

U Railsu, *resourceful* ruta obezbjeđuje spajanje HTTP metoda i URL adresa sa kontroler akcijom. Po konvenciji, svaka akcija takođe mapira određenu CRUD operaciju u bazi. 

Jedan unos u fajlu za rutiranje:

<pre><code>resources :photos</code></pre>

kreira sedam različitih ruta u aplikaciji. Sve su povezane sa Photos kontrolerom:


|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Primjena**|
|GET|/photos|index|prikazuje sve slike|
|GET|/photos/new|new|vraća HTML formu za kreiranje nove slike|
|POST|/photos|create|kreira novu sliku|
|GET|/photos/:id|show|prikazuje određenu sliku|
|GET|/photos/:id/edit|edit|vraća HTML formu za izmjenu slike|
|PATCH/PUT|/photos/:id|update|update-uje određenu sliku|
|DELETE|/photos/:id|destroy|briše određenu sliku|

## 2.3 Putanje i URL helperi

Kreiranje *resourceful* rute će takođe izložiti određen broj helpera kontrolerima u aplikaciji. U slučaju <code>resouces :photos</code>:

* <code>photos_path</code> će vratiti <code>/photos</code>
* <code>new_photo_path</code> će vratiti <code>/photos/new</code>
* <code>edit_photo_path(:id)</code> će vratiti <code>/photos/:id/edit</code>
* <code>photo_path(:id)</code> će vratiti <code>/photos/:id</code>

Svaki od ovih helpera ima i odgovarajući <code>_url</code> helper (kao <code>photos_url</code>) koji vraća istu putanju sa prefiksom trenutnog hosta, porta, kao i prefiksom putanje.

## 2.4 Definisanje više resursa odjednom

<pre><code>resources :photos, :books, :vides</code></pre>

Identično je sa:

<pre><code>resources :photos
resources :books
resources :videos</code></pre>

## 2.5 Pojedinačni resursi

Često imamo resurs koji pretražujemo bez navođenja ID-ja. Npr. u slučaju da želimo da <code>/profile</code> uvijek prikazuje trenutno logovanog korisnika. U ovom slučaju može se koristiti pojedinačni resurs da se spoji <code>/profile</code> (umjesto <code>/profile/:id</code>) sa show akcijom:

<pre><code>get 'profile', to: 'users#show'</code></pre>

Ako predajemo string, <code>to</code> će očekivati controller#action format, dok će simbol očekivati akciju:

<pre><code>get 'profile', to: :show</code></pre>

Ova resourceful ruta: 

<pre><code>resource :geocoder</code></pre>

će kreirati šest različitih ruta u aplikaciji. Sve su povezane sa Geocoder kontrolerom:


|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Primjena**|
|GET|/geocoder/new|new|vraća HTML formu za kreiranje geokodera|
|POST|/geocoder|create|kreira novi geokoder|
|GET|/geocoder/:id|show|prikazuje taj jedan jedini geokoder resurs|
|GET|/geocoder/:id/edit|edit|vraća HTML formu za izmjenu geokodera|
|PATCH/PUT|/geocoder/:id|update|update-uje taj jedan određeni geokoder resurs|
|DELETE|/geocoder/:id|destroy|briše geokoder resurs|

Helperi:

* <code>new_geocoder_path</code> će vratiti <code>/geocoder/new</code>
* <code>edit_geocoder_path</code> će vratiti <code>/geocoder/edit</code>
* <code>geocoder_path</code> će vratiti <code>/geocoder</code>

## 2.6 Kontroler namespace-ovi i rutiranje

Namespace-ove najčešće koristimo kada želimo da napravimo grupu kontrolera kao npr. administrativne kontrolere pod Admin:: namespace. Ovi kontroleri bi se nalazili u <code>app/controller/admin</code> direktorijumu, a mogu se grupisati u ruteru:

<pre><code>namespace :admin do
    resources :posts, :comments
end</code></pre>

Ovo će kreirati određen broj ruta za svaki post i comment kontroler. Za Admin::PostsController, Rails će kreirati:

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Primjena**|
|GET|/admin/posts|index|admin_posts_path|
|GET|/admin/posts/new|new|new_admin_post_path|
|POST|/admin/posts|create|admin_post_path|
|GET|/admin/posts/:id|show|admin_post_path(:id)|
|GET|/admin/posts/:id/edit|edit|edit_admin_post_path(:id)|
|PATCH/PUT|/admin/posts/:id|update|admin_post_path(:id)|
|DELETE|/admin/posts/:id|destroy|admin_post_path(:id)|

Ako želimo da rutiramo <code>/posts</code> (bez prefiksa <code>/admin</code>), za Admin::PostsController koristimo:

<pre><code>scope module: 'admin' do
    resources :posts, :comments
end</code></pre>

ili, za jedan slučaj:

<pre><code>resources :posts, module: 'admin'</code></pre>

Ako želimo da rutiramo <code>/admin/posts</code> za PostsController (bez Admin:: module prefiksa), korstimo:


<pre><code>scope '/admin' do
    resources :posts, :comments
end</code></pre>

ili, za jedan slučaj:

<pre><code>resources :posts, '/admin/posts'</code></pre>

U oba slučaja, imenovane rute ostaju iste kao i kada nismo koristili scope.

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Helper**|
|GET|/admin/posts|index|posts_path|
|GET|/admin/posts/new|new|new_post_path|
|POST|/admin/posts|create|post_path|
|GET|/admin/posts/:id|show|post_path(:id)|
|GET|/admin/posts/:id/edit|edit|edit_post_path(:id)|
|PATCH/PUT|/admin/posts/:id|update|post_path(:id)|
|DELETE|/admin/posts/:id|destroy|post_path(:id)|

## 2.7 Ugnježdeni resursi

Čest je slučaj korišćenja resursa koji su logički *children* drugih resursa.

<pre><code>class Magazine < ActiveRecord::Base
  has_many :ads
end
 
class Ad < ActiveRecord::Base
  belongs_to :magazine
end</code></pre>

Ugnježdene (*engl.* nested) rute omogućavaju preslikavanje ovih veza u rutiranju. U ovom slučaju, koristili bismo:

<pre><code>resources :magazines do
    resouces :ads
end</code></pre>

U dodatku rutama za magazine, ova deklaracija će takođe rutirati reklame (ads) na AdsController. URL adrese reklame u zahtjevima za magazin:

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Primjena**|
|GET|/magazines/:magazine_id/ads|index|lista spisak svih reklama za određeni magazin|
|GET|/magazines/:magazine_id/ads/new|new|vraća HTML formu za kreiranje nove reklama koja pripada određenom magazinu|
|POST|/magazines/:magazine_id/ads|create|kreiranje nove reklame koja pripada određenom magazinu|
|GET|/magazines/:magazine_id/ads/:id|show|prikaz određene reklame koja pripada određenom magazinu|
|GET|/magazines/:magazine_id/ads/:id/edit|edit|vraća HTML formu za izmjenu reklama koja pripada određenom magazinu|
|PATCH/PUT|/magazines/:magazine_id/ads/:id|update|update-ovanje određene reklame koja pripada određenom magazinu|
|DELETE|/magazines/:magazine_id/ads/:id|destroy|brisanje reklame koja pripada određenom magazinu|

Ovo će takođe kreirati routing helpere kao <code>magazine_ads_url</code> i <code>edit_magazine_ad_path</code>. Ovi helperi uzimaju instancu od Magazine kao prvi parametar (<code>magazine_ads_url(@magazine)</code>).

### 2.7.1 Deep Nesting

Možemo praviti ugnježdene resurse unutar drugih ugnježdenih resursa. Na primjer:

<pre><code>resources :publishers do
  resources :magazines do
    resources :photos
  end
end</code></pre>

U ovom slučaju aplikacija bi prepoznala putanju: <code>/publisher/1/magazines/2/photos/3</code>

Odgovarajući route helper bi bio <code>publisher_magazine_photo_url</code>, zahtijevajući određivanje objekata na sva tri nivoa. Ovo se ne preporučuje. Jamis Buck u svom popularnom [članku](http://weblog.jamisbuck.org/2007/2/5/nesting-resources) kaže:

> Resources should never be nested more than **1 level** deep.

### 2.7.2 Shallow Nesting

Jedan od načina za izbjegavanje deep nestinga je generisanje akcija kolekcija pod roditeljskim opsegom. Tako će se razumjeti hijerarhija ali se neće ugnježdavati akcije članova. Dakle, kreiramo rute sa minimalnom količinom informacija, da bi se jedinstveno odredio resurs, kao:

<pre><code>resources :posts do
  resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]</code></pre>

Ova ideja pravi balans između deskriptivnih ruta i deep nestinga. Za ovo postoji i skraćena sintaksa. Koristeći <code>:shallow</code> opciju, prethodni kod možemo zapisati kao:

<pre><code>resources :posts do
  resources :comments, shallow: true
end</code></pre>

Primjer: [What's New in Edge Rails: Shallow Routes](http://archives.ryandaigle.com/articles/2008/9/7/what-s-new-in-edge-rails-shallow-routes)

Ovu opciju možemo definisati i u roditeljskom resursu:

<pre><code>resources :posts, shallow: true do
  resources :comments
  resources :quotes
  resources :drafts
end</code></pre>

<code>shallow</code> metod DSL-a kreira scope unutar koga je svako ugnježdavanje shallow. Sljedeće će da generiše iste rute kao prethodni primjer:

<pre><code>shallow do
  resources :posts do
    resources :comments
    resources :quotes
    resources :drafts
  end
end</code></pre>

Postoje dvije opcije za scope da izmijeni shallow rute. <code>:shallow_path</code> stavlja prefikse putanjama članova (member paths) sa određenim parametrom:

<pre><code>scope shallow_path: "secret" do
  resources :posts do
    resources :comments, shallow: true
  end
end</code></pre>

Sljedeće rute će biti generisane:

|  |  | |
| -----------------------| -------------------| --------------- |
|**HTTP metod**|**Putanja**|**Helper**|
|GET|/posts/:post_id/comments(.:format)|post_comments|
|POST|/posts/:post_id/comments(.:format)|post_comments|
|GET|/posts/:post_id/comments/new(.:format)|new_post_comment|
|GET|/secret/comments/:id(.:format)|comment|
|GET|/secret/comments/:id/edit(.:format)|edit_comment|
|PATCH/PUT|/secret/comments/:id(.:format)|comment|
|DELETE|/secret/comments/:id(.:format)|comment|

Opcija <code>:shallow_prefix</code> će dodati određeni parametar imenima helpera:

<pre><code>scope shallow_prefix: "secret" do
  resources :posts do
    resources :comments, shallow: true
  end
end</code></pre>

|  |  | |
| -----------------------| -------------------| --------------- |
|**HTTP metod**|**Putanja**|**Helper**|
|GET|/posts/:post_id/comments(.:format)|post_comments|
|POST|/posts/:post_id/comments(.:format)|post_comments|
|GET|/posts/:post_id/comments/new(.:format)|new_post_comment|
|GET|/comments/:id(.:format)|secret_comment|
|GET|/comments/:id/edit(.:format)|edit_secret_comment|
|PATCH/PUT|/comments/:id(.:format)|secret_comment|
|DELETE|/comments/:id(.:format)|secret_comment|

## 2.8 Routing concern-i

Routing concerns omogućavaju deklarisanje često korišćenih ruta koje se mogu ponovo koristiti unutar drugih resursa ili ruta.

**Definisanje concerna**

<pre><code>concern :commentable do
  resources :comments
end
 
concern :image_attachable do
  resources :images, only: :index
end</code></pre>

Mogu se koristiti u resursima da se izbjegne duplikacija koda: 

<pre><code>resources :messages, concerns: :commentable
resources :posts, concerns: [:commentable, :image_attachable]</code></pre>

Napisano je ekvivalentno sa:

<pre><code>resources :messages do
  resources :comments
end
 
resources :posts do
  resources :comments
  resources :images, only: :index
end</code></pre>

Takođe se mogu koristiti na bilo kom mjestu u kodu:

<pre><code>namespace :posts do
  concerns :commentable
end</code></pre>

## 2.9 Kreiranje putanja i URL adresa iz objekata

Pored helpera, Rails može da kreira putanje i URL adrese iz niza parametara. Neka imamo sljedeći skup ruta:

<pre><code>resources :magazines do
  resources :ads
end</code></pre>

Kada koristimo <code>magazine_ad_path</code>, tada možemo predati instancu od Magazine i Ad umjesto numeričkog ID-ja:

    <%= link_to 'Ad details', magazine_ad_path(@magazine, @ad) %>

Možemo koristiti <code>url_for</code> sa skupom objekata i Rails će automatski odrediti koju rutu želite:

    <%= link_to 'Ad details', url_for([@magazine, @ad]) %>

U ovom slučaju Rails će vidjeti da je <code>@magazines</code> instanca od Magazine i <code>@ad</code> instanca od Ad i zbog toga koristiti <code>magazine_ad_path</code> helper. U helperima kao <code>link_to</code>, možemo samo navesti objekat umjesto poziva <code>url_for</code>:

    <%= link_to 'Ad details', [@magazine, @ad] %>

Za druge akcije, potrebno je dodati ime akcije kao prvi element niza:

    <%= link_to 'Edit Ad', [:edit, @magazine, @ad] %>

Ovo omogućava tretiranje instaci modela kao URL adresa i ključna je prednost korićenja ovakvog stila.

## 2.10 Dodavanja više RESTful akcija

Sedam ruta koje RESTful rutiranje kreira kao podrazumijevane, nisu ograničenje. Moguće je dodati dodatne rute koje se primjenjuju na kolekciju ili individualne članove kolekcije.

### 2.10.1 Dodavanje memeber ruta

Member rute se dodaju unutar **member** bloka unutar resource bloka:

<pre><code>resources :photos do
  member do
    get 'preview'
  end
end</code></pre>

Ovo će prepoznati <code>/photos/1/preview</code> sa GET i rutirati na *preview* akciju PhotosController-a sa resource vrijednošću id-ja iz params[:id]. Kreiraće i <code>preview_photo_url</code> i <code>preview_photo_path</code> helpere.

Unutar bloka member ruta, svako ime rute određuje HTTP metod koji će da prepozna. Može se koristiti <code>get</code>, <code>patch</code>, <code>post</code> ili <code>delete</code>. Ako nema više member ruta, može se koristiti <code>:on</code> opcija, bez bloka:

<pre><code>resources :photos do
  get 'preview', on: :member
end</code></pre>

<code>:on</code> opcija se može i izostaviti i ovo će kreirati istu member rutu, osim što će vrijednost resource id-ja biti dostupna u params[:photo_id] umjesto params[:id].

### 2.10.2 Dodavanje collection ruta

<pre><code>resources :photos do
  collection do
    get 'search'
  end
end</code></pre>

Ovo će omogućiti Railsu da prepozna putanje poput <code>/photos/search</code> sa GET i rutirati na search akciju PhotosController-a. Kreiraće i <code>search_photos_url</code> i <code>search_photos_path</code> route helpere.

I ovdje je moguće koristiti <code>:on</code> opciju.

**Napomena:**
Da dodamo alternativnu <code>new</code> akciju:
<pre><code>resources :comments do
  get 'preview', on: :new
end</pre></code>

Prepoznaje putanje kao <code>/comments/new/preview</code> sa GET.


# 3 Non-Resourceful rute

Pored resource rutiranja, Rails nudi odličnu podršku za rutiranje proizvoljnih URL adresa na akcije. Ovdje se svaka ruta u aplikaciji posebno podešava.

Iako se preporučuje resourceful rutiranje, postoji mnogo mjesta gdje je prikladnije koristiti obično rutiranje.


## 3.1 Bound parametri

Kada se obična putanja podešava, predaje se niz simbola koje Rails mapira na djelova dolaznog HTTP zahtjeva. Dva od ovih simbola su posebna: <code>:controller</code> mapira ime kontrolera u aplikaciji i <code>:action</code> mapira ime akcije unutar kontrolera. Na primjer:

<pre><code>get ':controller(/:action(/:id))'</code></pre>

Ako je dolazni zahtjev <code>/photos/show/1</code> procesiran od ove rute, onda će se show akcija dodati u PhotosController a parametar '1' će biti dostupan kao params[:id]. Ova ruta će takođe rutirati dolazni zahtjev <code>/photos</code> na PhotosController#index, jer su <code>:action</code> i <code>:id</code> opcioni parametri.

## 3.2 Dinamički segmenti

Broj dinamičkih segmenata unutar obične rute nije ograničen. Sve osim :controller i :action će biti dostupno akciji kao dio parametara.

<pre><code>get ':controller/:action/:id/:user_id'</code></pre>

Dolazna putanja <code>/photos/show/1/2</code> će biti otpremljena show akciji PhotosController-a. Params[:id] će biti '1' i params[:user_id] će biti '2'.

**Napomena:** :namespace-ovi i :module-i se ne mogu koristiti sa <code>:controller</code> segmentom putanje. Ako je to ipak neophodno, onda se namespace može navesti pomoću ograničenja: 

<code>get ':controller(/:action(/:id))', controller: /admin\/[^\/]+/</code>

## 3.3 Statički segmenti

<pre><code>get ':controller/:action/:id/with_user/:user_id'</code></pre>

Ova ruta će odgovarati na putanje kao <code>/photos/show/1/with_user/2</code>.

## 3.4 String upit

Params će prihvatiti i bilo koji parametar iz stringa upita. Na primjer, za ovu rutu:

<pre><code>get ':controller/:action/:id'</code></pre>

dolazna putanja <code>/photos/show/1?user_id=2</code> će biti otpremljena na show akciju Photos kontrolera. Params će biti <code>{ controller: 'photos', action: 'show', id: '1', user_id: '2' }</code>.

## 3.5 Definisanje default-a

Unutuar rute, nije potrebno čak ni koristiti <code>:controller</code> i <code>:action</code> simbole. Predaju se po defaultu:

<pre><code>get 'photos/:id', to: 'photos#show'</code></pre>

Sa ovom rutom, Rails će spojiti dolaznu putanju <code>/photos/12</code> sa show akcijom PhotosController.

I drugi defaulti se mogu definisati u ruti u vidu hasha za <code>:defaults</code> opciju. Ovo se odnosi i na parametre koji nisu određeni kao dinamički segmenti. Na primjer:

<pre><code>get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }</code></pre>

Rails bi <code>photos/12</code> poslao show akciji PhotosController-a i postavio params[:format] na 'jpg'.

## 3.6 Imenovanje ruta

Imenovanje ruta se vrši <code>:as</code> opcijom:

<pre><code>get 'exit', to: 'sessions#destroy', as: :logout</code></pre>

Ovo će kreirati <code>logout_path</code> i <code>logout_url</code> kao helpere aplikacije. Pozivanje <code>logout_path</code> će vratiti <code>/exit</code>.

Ovo se može koristi i za preklapanje metoda rutiranja definisanih od resursa, kao:

<pre><code>get ':username', to: 'users#show', as: :user</code></pre>

Ovo će definisati <code>user_path</code> metodu koja će biti dostupna u kontrolerima, helerima i view fajlovima koje će ići na rutu poput <code>/bob</code>.

## 3.7 Ograničenje HTTP metoda

Metode <code>get</code>, <code>post</code>, <code>put</code> i <code>delete</code> se koriste za ograničavanje ruta na određen metod. Može se koristiti match metod sa <code>:via</code> opcijom da se navede više metoda odjednom:

<pre><code>match 'photos', to: 'photos#show', via: [:get, :post]</code></pre>

Ili sve odjednom:

<pre><code>match 'photos', to: 'photos#show', via: :all</code></pre>

## 3.8 Ograničenja segmenata

Opciju <code>:constraints</code> koristimo da odredimo format za dinački segment:

<pre><code>get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ }</code></pre>

Rails će spojiti putanju kao <code>/photos/A01234</code> ali ne i <code>/photos/56789</code>

Istu rutu je moguće kraće zapisati kao: <pre><code>get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/</code></pre>

<code>:constraints</code> uzima regularni izraz uz napomenu da se [anchor simboli](http://msdn.microsoft.com/en-us/library/h5181w5w(v=vs.110).aspx) ne mogu koristiti.

Ali oni nisu ni potrebni.

## 3.9 Ograničenja zasnovana na request objektima

Moguće je ograničiti i rutu zasnovanu na bilo kojoj metodi na **Request objektu** (pogledati: *10.1 - Action Controller Overview*) koja vraća String.

Request-based ograničenje se određuje isto kao i ono na segmentu:

<pre><code>get 'photos', constraints: {subdomain: 'admin'}</code></pre>

Ograničenja je moguće odrediti unutar bloka:

<pre><code>namespace :admin do
  constraints subdomain: 'admin' do
    resources :photos
  end
end</code></pre>

## 3.11 Route Globbing i Wildcard segmenti

**Route globbing** je način da se određeni parametar spoji sa svim (pre)ostalim djelovima rute. Na primjer:

<pre><code>get 'photos/*other', to: 'photos#unknown'</code></pre>

Ova ruta će spojiti <code>photos/12</code> i <code>photos/long/path/to/12</code> podešavajući params[:other] na <code>12</code> ili <code>long/path/to/12</code>. Fragmenti koji su prefiks zvijezdi se zovu 'wildcard segmenti'. Wildcard segmenti se mogu pojaviti bilo gdje u ruti.

<pre><code>get 'books/*section/:title', to: 'books#show'</code></pre>

A možemo imati i više od jednog wildcard segmenta:

<pre><code>get '*a/foo/*b', to: 'test#index'</code></pre>

a spojilo bi <code>zoo/woo/foo/bar/baz</code> gdje su params[:a] 'zoo/woo' i params[:b] 'bar/baz'.

Opcijom <code>:format</code> (true ili false) određujemo da li je format segment obavezan.

## 3.12 Redirekcija

Pomoću redirect helpera moguće je redirektovati bilo koju putanju:

<pre><code>get '/stories', to: redirect('/posts')</code></pre>

Mogu se ponovo koristiti i dinamički segmenti:

<pre><code>get '/stories/:name', to: redirect('/posts/%{name}')</code></pre>

A mogu se koristiti i blokovi, koji će primiti parametre i request objekat:

<pre><code>get '/stories/:name', to: redirect {|params, req| "/posts/#{params[:name].pluralize}" }
get '/stories', to: redirect {|p, req| "/posts/#{req.subdomain}" }</code></pre>

## 3.14 Korišćenje root

Sa root metodom možemo odrediti šta je Rails da rutira sa '/':

<pre><code>root to: 'pages#main'
root 'pages#main' # kraći zapis prethodnog</code></pre>

Root ruta bi se trebala nalaziti na vrhu fajla. Root metoda se može koristiti unutar namespace-ova i scope-ova.

## 3.15 Rutiranje Unicode karaktera

Rute sa Unicode karakterima se mogu definisati direktno:

<pre><code>get 'こんにちは', to: 'welcome#index'</code></pre>

# 4 Prilagođavanje Resourceful ruta

## 4.1 Određivanje kontrolera za korišćenje

<code>:controller</code> opcija omogućava određivanja kontrolera koji će se koristiti za određeni resurs. Na primjer:

<pre><code>resources :photos, controller: 'images'</code></pre>

će prepoznati dolazne putanje koje počinju sa <code>/photos</code> ali rutirati na Images kontroler:

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Helper**|
|GET|/photos|index|photos_path|
|GET|/photos/new|new|new_photo_path|
|POST|/photos|create|photo_path|
|GET|/photos/:id|show|photo_path(:id)|
|GET|/photos/:id/edit|edit|edit_photo_path(:id)|
|PATCH/PUT|/photos/:id|update|photo_path(:id)|
|DELETE|/photos/:id|destroy|photo_path(:id)|

Za kontrolere unutar namespace, koristi se notacija sa direktorijumima:

<pre><code>resources :user_permissions, controller: 'admin/user_permissions'</code></pre>

## 4.2 Određivanje ograničenja

Možemo koristiti opciju <code>:constraints</code> da zahtijevamo određeni format za id. Na primjer:

<pre><code>resources :photos, constraints: {id: /[A-Z][A-Z][0-9]+/}</code></pre>

Ako želimo da jedno ograničenja primijenimo na više ruta, koristimo blok:

<pre><code>constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end</code></pre>

## 4.3 Preklapanje imena helpera

Opciju <code>:as</code> možemo koristiti ako želimo da predefinišemo normalno imenovanje helpera. Na primjer:

<pre><code>resources :photos, as: 'images'</code></pre>

će prepoznati dolaznu putanju koja počinje sa <code>/photos</code> i rutirati zahtjev na PhotosController, ali će koristiti vrijednost <code>:as</code> opcije za imenovanje helpera:

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Helper**|
|GET|/photos|index|images_path|
|GET|/photos/new|new|new_image_path|
|POST|/photos|create|image_path|
|GET|/photos/:id|show|image_path(:id)|
|GET|/photos/:id/edit|edit|edit_image_path(:id)|
|PATCH/PUT|/photos/:id|update|image_path(:id)|
|DELETE|/photos/:id|destroy|image_path(:id)|

## 4.4 Preklapanje new i edit segmenata

<code>:path_names</code> opcija omogućava preklapanje automatski generisanih "new" i "edit" segmenata u putanji:

<pre><code>resources :photos, path_names: { new: 'make', edit: 'change' }</code></pre>

Ovo će prouzrokovati to da rutiranje prepozna putanje kao:

<pre><code>/photos/make
/photos/1/change</code></pre>

## 4.5 Prefiksi imena helpera

Opciju <code>:as</code> možemo koristi i da dodamo prefiks imenima helpera koje Rails generiše. Ovu opciju treba koristiti da se izbjegne eventualno preklapanje između ruta korišćenjem path scope-a. Na primjer:

<pre><code>scope 'admin' do
  resources :photos, as: 'admin_photos'
end
 
resources :photos</code></pre>

Dobićemo helpere <code>admin_photos_path</code>, <code>new_admin_photo_path</code> itd.

Da dodamo prefiks **grupi** route helpera, koristimo <code>:as</code> sa scope-om:

<pre><code>scope 'admin', as: 'admin' do
  resources :photos, :accounts
end
 
resources :photos, :accounts</code></pre>

Ovo će generisati rute poput <code>admin_photos_path</code> i <code>admin_account_path</code> koje mapiraju <code>/admin/photos</code> i <code>/admin/accounts</code>.

Prefiks se može dodati i rutama sa imenovanima parametrom:

<pre><code>scope ':username' do
  resources :posts
end</code></pre>

što će rezultovati URL adresama kao <code>/bob/posts/1</code> i dozvoliti referenciranje username dijela putanje kao params[:username] u kontrolerima, helperima i view fajlovima.

## 4.6 Restrikcija kreiranih ruta

Po defaultu, Rails kreira rute za sedam default akcija (<code>index</code>, <code>show</code>, <code>new</code>, <code>create</code>, <code>edit</code>, <code>update</code> i <code>destroy</code>) za svaki rutu u aplikaciji. Možemo koristiti <code>:only</code> i <code>:except</code> opcije da unaprijedimo prethodno rečeno. Opcija <code>:only</code> govori Railsu koje putanje jedino da kreira:

<pre><code>resources :photos, only: [:index, :show]</code></pre>

Sada, GET zahtjev za <code>/photos</code> bi uspio, ali POST zahtjev za <code>/photos</code> (koji bi inače bio rutiran na <code>create</code> akciju) ne bi uspio.

Opcija <code>:except</code> određuje koje rute se (jedino) neće kreirati:

<pre><code>resources :photos, except: :destroy</code></pre>

## 4.7 Prevedene putanje

Korišćenjem scope-a, možemo promijeniti imena putanja generisana od resursa:

<pre><code>scope(path_names: { new: 'nova', edit: 'izmjena' }) do
  resources :categories, path: 'kategorija'
end</code></pre>

Rails sada kreira rute za CategoriesController:

|  |  | | |
| -----------------------| -------------------| --------------- |--------------- |
|**HTTP metod**|**Putanja**|**Akcija**|**Helper**|
|GET|/kategorija|index|categories_path|
|GET|/kategorija/nova|new|new_category_path|
|POST|/kategorija|create|category_path|
|GET|/kategorija/:id|show|category_path(:id)|
|GET|/kategorija/:id/izmjena|edit|edit_category_path(:id)|
|PATCH/PUT|/kategorija/:id|update|category_path(:id)|
|DELETE|/kategorija/:id|destroy|category_path(:id)|

## 4.8 Preklapanje forme u jednini

Ako želimo da definišemo resurs u jednini, dodajemo dodatna pravila na Inflector:

<pre><code>ActiveSupport::Inflector.inflections do |inflect|
  inflect.irregular 'tooth', 'teeth'
end</code></pre>

## 4.9 Korišćenje :as u Nested resursima

Opcija <code>:as</code> preklapa automatski generisano ime resursa u ugnježdenim rutama helpera.

<pre><code>resources :magazines do
  resources :ads, as: 'periodical_ads'
end</code></pre>

Ovo će kreirati helpere kao <code>magazine_periodical_ads_url</code> i <code>edit_magazine_periodical_ad_path</code>.

# 5 Pregled i testiranje ruta

## 5.1 Spisak postojećih ruta

Za dobijanje liste svih dostupnih ruta u aplikaciji, posjetiti <code>http://localhost:3000/rails/info/routes</code> u browseru dok server radi u **development** okruženju. Isti rezultate možemo dobiti komandom <code>rake routes</code> u terminalu.

Takođe je moguće ograničiti štampanje spiska svih postojećih ruta na određen kontroler podešavanjem <code>CONTROLLER</code> promjenljive okruženja:

<pre><code>$ CONTROLLER=users rake routes</code></pre>

## 5.1 Testiranje ruta

Rails nudi tri ugrađene metode potvrde (assertions) koje olakšavaju testiranje ruta:

* <code>assert_generates</code>
* <code>assert_recognizes</code>
* <code>assert_routing</code>

### 5.1.1 assert_generates

<code>assert_generates</code> potvrđuje da određen skup opcija generiše određenu putanju i da se može koristiti sa default rutom ili nekom predefinisanom rutom:

<pre><code>assert_generates '/photos/1', { controller: 'photos', action: 'show', id: '1' }
assert_generates '/about', controller: 'pages', action: 'about'</code></pre>

### 5.1.2 assert_recognizes

<code>assert_recognizes</code> je suprotna <code>assert_generates</code>. Potvrđuje da je određena putanja prepoznata i da rutira na određeno mjesto u aplikaciji. 

<pre><code>assert_recognizes({ controller: 'photos', action: 'show', id: '1' }, '/photos/1')</code></pre>

Argument <code>:method</code> se može koristiti da se odredi HTTP metod:

<pre><code>assert_recognizes({ controller: 'photos', action: 'create' }, { path: 'photos', method: :post })</code></pre>

### 5.1.3 assert_routing

<code>assert_routing</code> provjerava rutu u oba pravca: testira da li putanja generiše opcije i da li opcije generišu putanju.

<pre><code>assert_routing({ path: 'photos', method: :post }, { controller: 'photos', action: 'create' })</code></pre>

**Izvor:**
[Rails Guides - Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html)
