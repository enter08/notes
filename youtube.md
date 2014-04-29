# YouTube on Rails

YouTube je treći najposjećeniji sajt na svijetu. U ovom tutorijalu, naučićemo kako da napravimo aplikaciju koja omogućava korisnicima dodavanje i gledanje video klipova sa YouTube-a. Informacije o video klipovima će biti dostupne preko YouTube API-ja. Video klipovi će biti prikazani preko standardnom YouTube playera uz pomoć YouTube IFrame API-ja.

* Izvorni kod na GitHub-u: [SitePoint](https://github.com/bodrovis/SitePoint-YtVideos)
* Demo verzija na Heroku-u: [SitePoint Heroku App](http://sitepoint-yt-videos.herokuapp.com/)

Za početak, kreirajmno novu Rails aplikaciju:

	$ rails new yt_videos -T

<code>-T</code> će kreirati aplikaciju bez default test fajlova.

U Gemfile-u dodajemo bootstrap i youtube_it gem koji radi sa YouTube API.

	gem 'bootstrap-sass', '~> 3.1.1'
	gem 'youtube_it', '~> 2.4.0'

	$ bundle install

Dodajemo:

	@import "bootstrap";

u <code>custom.css.scss</code> fajl u stylesheet folderu. 

**Video** model treba da sadrži sljedeće atribute:

* id – integer, primarni ključ, indeksiran, unique
* link – string
* uid – string. Dodaćemo i ovdje index i unique.
* title – string. Naslov YouTube videa sa youtube sajta.
* author – string. Ime autora videa sa youtube sajta.
* duration – string. Dužina trajanja videa u formatu: “00:00:00″
* likes – integer. Broj lajkova.
* dislikes – integer. Broj dislajkova.

Kreirajmo model:

	$ rails g model Video link:string title:string author:string duration:string likes:integer dislikes:integer
	$ rails g migration add_uid_to_videos uid:string:uniq:index

routes.rb:

	resources :videos, only: [:index, :new, :create]
	root to: 'videos#index'

application.html.erb:

	<div class="container">
	  <% flash.each do |key, value| %>
	    <div class="alert alert-<%= key %>">
	      <button type="button" class="close" data-dismiss="alert">&times;</button>
	      <%= value %>
	    </div>
	  <% end %>
	</div>

index.html.erb

	<div class="jumbotron">
	  <div class="container">
	    <h1>Share your videos with the world!</h1>
	    <p>Click the button below to share your video from YouTube.</p>
	    <p>
	      <%= link_to 'Add video now!', new_video_path, class: 'btn btn-primary btn-lg' %>
	    </p>
	  </div>
	</div>

videos_controller.rb

	class VideosController < ApplicationController

		def new
		  @video = Video.new
		end
		 
		def create
		  @video = Video.create(video_params)
		  if @video.save
		    flash[:success] = 'Video added!'
		    redirect_to root_url
		  else
		    render 'new'
		  end
		end
	end

	private
		
	def video_params
		params.require(:video).permit(:uid, :link, :title, :author, :duration, :likes, :dislikes)
	end

new.html.erb

	<div class="container">
	  <h1>New video</h1>
	 
	  <%= form_for @video do |f| %>
	    <%= render 'shared/errors', object: @video %>
	 
	    <div class="form-group">
	      <%= f.label :link %>
	      <%= f.text_field :link, class: 'form-control', required: true %>
	      <span class="help-block">A link to the video on YouTube.</span>
	    </div>
	 
	    <%= f.submit class: 'btn btn-default' %>
	  <% end %>
	</div>


shared/_errors.html.erb

		<% if object.errors.any? %>
		  <div class="panel panel-danger">
		    <div class="panel-heading">
		      <h3 class="panel-title">The following errors were found while submitting the form:</h3>
		    </div>
		 
		    <div class="panel-body">
		      <ul>
		        <% object.errors.full_messages.each do |msg| %>
		          <li><%= msg %></li>
		        <% end %>
		      </ul>
		    </div>
		  </div>
		<% end %>

Da dohvatimo informacije iz YouTube API-ja, biće nam potreban jedinstveni id videa. Taj id se već nalazi u URL-u videa. Ali URL može imati više formata. Ovo su validni YT URL formati:

http://www.youtube.com/watch?v=0zM3nApSvMg&feature=feedrecgrecindex
http://www.youtube.com/user/IngridMichaelsonVEVO#p/a/u/1/QdK8U-VIH_o
http://www.youtube.com/v/0zM3nApSvMg?fs=1&hl=en_US&rel=0
http://www.youtube.com/watch?v=0zM3nApSvMg#t=0m10s
http://www.youtube.com/embed/0zM3nApSvMg?rel=0
http://www.youtube.com/watch?v=0zM3nApSvMg
http://youtu.be/0zM3nApSvMg

Da dohvatimo ovaj ID, koristićemo sljedeći regularni izraz:

video.rb

	YT_LINK_FORMAT = /\A.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|\&v=)([^#\&\?]*).*/i
	 
	validates :link, presence: true, format: YT_LINK_FORMAT

taj ID treba da dohvatimo prije dodavanje videa:

video.rb

	before_create -> do
	  uid = link.match(YT_LINK_FORMAT)
	  self.uid = uid[2] if uid && uid[2]
	 
	  if self.uid.to_s.length != 11
	    self.errors.add(:link, 'is invalid.')
	    false
	  elsif Video.where(uid: self.uid).any?
	    self.errors.add(:link, 'is not unique.')
	    false
	  else
	    get_additional_info
	  end
	end

Sada se moraju dohvatiti informacije o video klipu:

video.rb

private
 
	def get_additional_info
	  begin
	    client = YouTubeIt::OAuth2Client.new(dev_key: 'Your_YT_developer_key')
	    video = client.video_by(uid)
	    self.title = video.title
	    self.duration = parse_duration(video.duration)
	    self.author = video.author.name
	    self.likes = video.rating.likes
	    self.dislikes = video.rating.dislikes
	  rescue
	    self.title = '' ; self.duration = '00:00:00' ; self.author = '' ; self.likes = 0 ; self.dislikes = 0
	  end
	end
	 
	def parse_duration(d)
	  hr = (d / 3600).floor
	  min = ((d - (hr * 3600)) / 60).floor
	  sec = (d - (hr * 3600) - (min * 60)).floor
	 
	  hr = '0' + hr.to_s if hr.to_i < 10
	  min = '0' + min.to_s if min.to_i < 10
	  sec = '0' + sec.to_s if sec.to_i < 10
	 
	  hr.to_s + ':' + min.to_s + ':' + sec.to_s
	end

Ovdje se uključuje youtube_it. Prvo se kreira client promjenljiva za upite. Ovdje će nam biti potreban YouTube developer ključ za upite sa javnim YouTube podacima. Da dobijete svoj developer ključ, registrujte novu aplikaciju na https://code.google.com/apis/console. Otvorite podešavanja i otvorite ''APIs and auth'' pa zatim Credentials. Kreirajte novi javni ključ za browser aplikaciju koji će biti vaš developer ključ. 
Nakon inicijalizacije klijenta, videu se pristupa sa video_by gdje će biti dostupne sve informacije. Trajanje video klipa će biti predstavljeno u sekundama tako da će nam biti potrebna parse_duration metoda zbog formatiranja.

Do sada, aplikacija omogućava dodavanje novih video klipova, dohvata informacije o tom klipu a dodali smo i neke validacije.

## Prikaz

U videos_controller.rb dodajemo:

	def index
	  @videos = Video.order('created_at DESC')
	end

I u view-u:

	<% if @videos.any? %>
	  <div class="container">
	    <h1>Latest videos</h1>
	 
	    <div id="player-wrapper"></div>
	 
	    <% @videos.in_groups_of(3) do |group| %>
	      <div class="row">
	        <% group.each do |video| %>
	          <% if video %>
	            <div class="col-md-4">
	              <div class="yt_video thumbnail">
	                <%= image_tag "https://img.youtube.com/vi/#{video.uid}/mqdefault.jpg", alt: video.title,
	                              class: 'yt_preview img-rounded', :"data-uid" => video.uid %>
	 
	                <div class="caption">
	                  <h5><%= video.title %></h5>
	                  <p>Author: <b><%= video.author %></b></p>
	                  <p>Duration: <b><%= video.duration %></b></p>
	                  <p>
	                    <span class="glyphicon glyphicon glyphicon-thumbs-up"></span> <%= video.likes %>
	                    <span class="glyphicon glyphicon glyphicon-thumbs-down"></span> <%= video.dislikes %>
	                  </p>
	                </div>
	              </div>
	            </div>
	          <% end %>
	        <% end %>
	      </div>
	    <% end %>
	  </div>
	<% end %>

**#player-wrapper** je prazan blok gdje će se nalaziti YT player. in_groups_of govori koliko će se video klipova nalaziti u jednom redu. Preview slika može imati tri formata:

* https://img.youtube.com/vi/mqdefault.jpg – 320×180 slika bez crnih traka ispod i iznad slike.
* https://img.youtube.com/vi/hqdefault.jpg – 480×360 slika sa crnim trakama ispod i iznad slike.
* https://img.youtube.com/vi/<1,2,3>.jpg – 120×90 slika sa više scena iz klipa sa crnim trakama ispod i iznad slika.

Da prikažemo klipove na sajtu, koristime Youtube IFrame API. 

U application.html.erb dodajemo:

		<script src="https://www.google.com/jsapi"></script>
		<script src="https://www.youtube.com/iframe_api"></script>
	</head>


U javascripts/yt_player.coffee dodajemo:

	jQuery ->
	  $('.yt_preview').click -> makeVideoPlayer $(this).data('uid')

	  # Initially the player is not loaded
	  window.ytPlayerLoaded = false

	  _run = ->
	    # Runs as soon as Google API is loaded
	    $('.yt_preview').first().click()
	    return

	  $(window).bindWithDelay('resize', ->
	    player = $('#ytPlayer')
	    player.height(player.width() / 1.777777777) if player.size() > 0
	    return
	  , 500)

	  makeVideoPlayer = (video) ->
	    if !window.ytPlayerLoaded
	      player_wrapper = $('#player-wrapper')
	      player_wrapper.append('<div id="ytPlayer"><p>Loading player...</p></div>')

	      window.ytplayer = new YT.Player('ytPlayer', {
	        width: '100%'
	        height: player_wrapper.width() / 1.777777777
	        videoId: video
	        playerVars: {
	          wmode: 'opaque'
	          autoplay: 0
	          modestbranding: 1
	        }
	        events: {
	          'onReady': -> window.ytPlayerLoaded = true
	          'onError': (errorCode) -> alert("We are sorry, but the following error occured: " + errorCode)
	        }
	      })
	    else
	      window.ytplayer.loadVideoById(video)
	      window.ytplayer.pauseVideo()
	    return

	  google.setOnLoadCallback _run

	  return


1.7(7) je 16:9 rezolucija.

Koristi se bindWithDelay funkcija koju moramo definisati:

javascript/jquery.bind_with_delay.js

	/*
	 bindWithDelay jQuery plugin
	 Author: Brian Grinstead
	 MIT license: http://www.opensource.org/licenses/mit-license.php

	 http://github.com/bgrins/bindWithDelay
	 http://briangrinstead.com/files/bindWithDelay

	 Usage:
	 See http://api.jquery.com/bind/
	 .bindWithDelay( eventType, [ eventData ], handler(eventObject), timeout, throttle )

	 Examples:
	 $("#foo").bindWithDelay("click", function(e) { }, 100);
	 $(window).bindWithDelay("resize", { optional: "eventData" }, callback, 1000);
	 $(window).bindWithDelay("resize", callback, 1000, true);
	 */

	(function($) {

	    $.fn.bindWithDelay = function( type, data, fn, timeout, throttle ) {

	        if ( $.isFunction( data ) ) {
	            throttle = timeout;
	            timeout = fn;
	            fn = data;
	            data = undefined;
	        }

	        // Allow delayed function to be removed with fn in unbind function
	        fn.guid = fn.guid || ($.guid && $.guid++);

	        // Bind each separately so that each element has its own delay
	        return this.each(function() {

	            var wait = null;

	            function cb() {
	                var e = $.extend(true, { }, arguments[0]);
	                var ctx = this;
	                var throttler = function() {
	                    wait = null;
	                    fn.apply(ctx, [e]);
	                };

	                if (!throttle) { clearTimeout(wait); wait = null; }
	                if (!wait) { wait = setTimeout(throttler, timeout); }
	            }

	            cb.guid = fn.guid;

	            $(this).bind(type, data, cb);
	        });
	    };

	})(jQuery);