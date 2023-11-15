Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 4 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit täydentää ratkaisusi tehtävien palautusjärjestelmän kautta näkyvän esimerkivastauksen avulla.

## Avoimien rajapintojen hyödyntäminen: baarien haku

Suuri osa internetin palveluista hyödyntää nykyään joitain avoimia rajapintoja, joiden tarjoaman datan avulla sovellukset voivat rikastaa omaa toiminnallisuuttaan.

Käyttöömme valikoituu Beermapping API <https://beermapping.com/api/>, joka tarjoaa mahdollisuuden oluita tarjoilevien ravintoloiden tietojen etsintään.

Beermapingin API:a käyttävät sovellukset tarvitsevat yksilöllisen API-avaimen. Saat avaimen sivulta https://beermapping.com/api/ kirjauduttuasi ensin sivulle (kirjautumisen jälkeen vaihda selaimen osoiteriviltä osoite takaisin muotoon https://beermapping.com/api/). Vastaava käytäntö on olemassa hyvin suuressa osassa nykyään tarjolla olevissa avoimissa rajapinnoissa.

API:n tarjoamat palvelut on listattu sivulla https://beermapping.com/api/reference/

Saamme esim. selville tietyn paikkakunnan olutravintolat tekemällä HTTP-get-pyynnön osoitteeseen <code>https://beermapping.com/webservice/loccity/[apikey]/[city]<location></code>

Paikkakunta siis välitetään osana URL:ia.

Kyselyjen tekemistä voi kokeilla selaimella tai komentoriviltä curl-komennolla. Saamme esimerkiksi Espoon olutravintolat selville seuraavasti:

```ruby
mluukkai@melkki$ curl https://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/espoo
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>12411</id><name>Gallows Bird</name><status>Brewery</status><reviewlink>https://beermapping.com/location/12411</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=12411&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=12411&amp;d=1&amp;type=norm</blogmap><street>Merituulentie 30</street><city>Espoo</city><state></state><zip>02200</zip><country>Finland</country><phone>+358 9 412 3253</phone><overall>91.66665</overall><imagecount>0</imagecount></location><location><id>21108</id><name>Captain Corvus</name><status>Beer Bar</status><reviewlink>https://beermapping.com/location/21108</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21108&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21108&amp;d=1&amp;type=norm</blogmap><street>Suomenlahdentie 1</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02230</zip><country>Finland</country><phone>+358 50 4441272</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21496</id><name>Olarin panimo</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21496</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21496&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21496&amp;d=1&amp;type=norm</blogmap><street>Pitkäniityntie 1</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02810</zip><country>Finland</country><phone>045 6407920</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21516</id><name>Fat Lizard Brewing</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21516</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21516&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21516&amp;d=1&amp;type=norm</blogmap><street>Lämpömiehenkuja 3</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02150</zip><country>Finland</country><phone>09 23165432</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21545</id><name>Simapaja</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21545</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21545&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21545&amp;d=1&amp;type=norm</blogmap><street>Kipparinkuja 2</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02320</zip><country>Finland</country><phone></phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
```

Kuten huomaamme, vastaus tulee XML-muodossa. Käytänne on hieman vanhahtava, sillä tällä hetkellä ylivoimaisesti suosituin web-palveluiden välillä käytettävä tiedonvaihdon formaatti on json.

Selaimella näemme palautetun XML:n hieman ihmisluettavammassa muodossa:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-1.png)

**HUOM1: älä käytä tässä näytettyä API-avainta vaan rekisteröi itsellesi oma avain.**

**HUOM2: syksyllä 2022 API ei löydä Espoosta yhtään baaria, kokeile joitan muuta kaupunkia! Suomen osalta API:n paikkatuntemus on heikko**

Tehdään nyt sovellukseemme olutravintoloita etsivä toiminnallisuus.

Luodaan tätä varten sivu osoitteeseen places, eli määritellään route.rb:hen

    get 'places', to: 'places#index'

ja luodaan kontrolleri:

```ruby
class PlacesController < ApplicationController
  def index
  end
end
```

ja näkymä app/views/places/index.html.erb, joka aluksi ainoastaan näyttää hakuun tarvittavan lomakkeen:

```erb
<h1>Beer places search</h1>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>
```

Lomake siis lähettää HTTP POST -kutsun places_path:iin. Määritellään tälle oma reitti routes.rb:hen

    post 'places', to: 'places#search'

Päätimme siis että metodin nimi on <code>search</code>. Laajennetaan kontrolleria seuraavasti:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    render :index
  end
end
```

Ideana on se, että <code>search</code>-metodi hakee panimoiden listan beermapping API:sta, jonka jälkeen panimot listataan index.html:ssä eli tämän takia metodin <code>search</code> lopussa renderöidään näkymätemplate <code>index</code>.

Kontrollerista metodissa <code>search</code> on siis tehtävä HTTP-kysely beermappin API:n sivulle. Paras tapa HTTP-kutsujen tekemiseen Rubyllä on HTTParty-gemin käyttö ks. https://github.com/jnunemaker/httparty. Lisätään seuraava Gemfileen:

    gem 'httparty'

Otetaan uusi gem käyttöön suorittamalla komentoriviltä tuttu komento <code>bundle install</code>

Kokeillaan nyt etsiä konsolista käsin Helsingin ravintoloita (muista uudelleenkäynnistää konsoli):

```ruby
> api_key = "731955affc547174161dbd6f97b46538"
> url = "http://beermapping.com/webservice/loccity/#{api_key}/"
> response = HTTParty.get "#{url}helsinki"
```

Kutsu palauttaa luokan <code>HTTParty::Response</code>-olion. [Dokumentaatiosta](https://www.rubydoc.info/github/jnunemaker/httparty/HTTParty/Response) selviää, että oliolta voidaan kysyä esim. HTTP-pyynnön vastaukseen liittyvät _headerit_ seuraavasti

```ruby
> response = HTTParty.get "#{url}helsinki"
> response.headers
=> {"date"=>["Mon, 17 Sep 2018 20:43:11 GMT"],
 "server"=>["Apache"],
 "upgrade"=>["h2,h2c"],
 "connection"=>["Upgrade, close"],
 "set-cookie"=>["easylogin_session=eff28ad09a8f62046917a8c424e4b0b3; path=/"],
 "expires"=>["Mon, 26 Jul 1997 05:00:00 GMT"],
 "cache-control"=>
  ["no-store, no-cache, must-revalidate", "post-check=0, pre-check=0"],
 "pragma"=>["no-cache"],
 "last-modified"=>["Mon, 17 Sep 2018 20:43:11 GMT"],
 "vary"=>["Accept-Encoding"],
 "content-length"=>["972"],
 "content-type"=>["text/xml;charset=UTF-8"]}
>
```

Headerit sisältävät HTTP-pyynnön vastaukseen liittyvää metadataa, esim. vastauksen muodon kertoo _content-type_

```
"content-type"=>["text/xml;charset=UTF-8"]
```

Eli vastaukseen liittyvä data on UTF-8-muodossa enkoodattua XML:ää.

HTTP-pyynnön statuskoodi selviää seuraavasti:

```ruby
> response.code
 => 200
```

Statuskoodi ks. https://www.rfc-editor.org/rfc/rfc9110.html#name-successful-2xx on tällä kertaa 200 eli ok, kutsu on siis onnistunut.

Vastausolion metodi <code>parsed_response</code> palauttaa metodin palauttaman datan Rubyn hashina:

```ruby
> response.parsed_response
=> {"bmp_locations"=>
  {"location"=>
    [{"id"=>"6742",
      "name"=>"Pullman Bar",
      "status"=>"Beer Bar",
      "reviewlink"=>"https://beermapping.com/location/6742",
      "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5",
      "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm",
      "street"=>"Kaivokatu 1",
      "city"=>"Helsinki",
      "state"=>nil,
      "zip"=>"00100",
      "country"=>"Finland",
      "phone"=>"+358 9 0307 22",
      "overall"=>"72.500025",
      "imagecount"=>"0"},
     {"id"=>"6743",
      "name"=>"Belge",
      "status"=>"Beer Bar",
      "reviewlink"=>"https://beermapping.com/location/6743",
      "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6743&d=5",
      "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6743&d=1&type=norm",
      "street"=>"Kluuvikatu 5",
      "city"=>"Helsinki",
      "state"=>nil,
      "zip"=>"00100",
      "country"=>"Finland",
      "phone"=>"+358 10 766 35",
      "overall"=>"67.499925",
      "imagecount"=>"1"},
...
```

Vaikka palvelin siis palauttaa vastauksensa XML-muodossa, parsii HTTParty-gem vastauksen ja mahdollistaa sen käsittelyn suoraan miellyttävämmässä muodossa Rubyn hashinä.

Kutsun palauttamat ravintolat sisältävä taulukko saadaan seuraavasti:

```ruby
> places = response.parsed_response['bmp_locations']['location']
> places.size => 12
```

Helsingistä tunnetaan siis 12 paikkaa. Tutkitaan ensimmäistä:

```ruby
> places.first
=> {"id"=>"6742",
 "name"=>"Pullman Bar",
 "status"=>"Beer Bar",
 "reviewlink"=>"https://beermapping.com/location/6742",
 "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5",
 "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm",
 "street"=>"Kaivokatu 1",
 "city"=>"Helsinki",
 "state"=>nil,
 "zip"=>"00100",
 "country"=>"Finland",
 "phone"=>"+358 9 0307 22",
 "overall"=>"72.500025",
 "imagecount"=>"0"}
```


Luodaan panimoiden esittämiseen oma olio, kutsuttakoon sitä nimellä <code>Place</code>. Sijoitetaan luokka models-hakemistoon.

```ruby
class Place < OpenStruct
end
```

Koska luomme luokan olutravintolaa esittävän hashin perusteella, teemme luokan siten että perimme siihen Rubyn valmiin [OpenStruct](https://ruby-doc.org/stdlib-3.1.2/libdoc/ostruct/rdoc/OpenStruct.html)-luokan toiminnallisuuden.

OpenStructin avulla hash on helppo "kääriä" olioksi, joka mahdollistaa hashin kenttiin viittaamisen pistenotaatiolla.

Esim. jos meillä on normaali hash, joka on määritelty seuraavasti

```ruby
baari_hash = {
 "name"=>"Pullman Bar",
 "status"=>"Beer Bar",
  "city"=>"Helsinki"
}
```

joudumme viittaamaan sen kenttiin hakasulkeilla:

```ruby
> baari_hash['name']
=> "Pullman Bar"
> baari_hash['city']
=> "Helsinki"
```

Jos "käärimme" hashin OpenStruct-olioksi:

```ruby
> baari = OpenStruct.new baari_hash
```

pääsemme kaikkiin kenttiin käsiksi pistenotaatiolla:

```ruby
baari.name
=> "Pullman Bar"
baari.city
=> "Helsinki"
```

ja näin saamme aikaan olion joka muistuttaa käyttötavaltaan normaaleja Railsin modeleja, kuten Beer, Brewery ym.

Emme kuitenkaan halua käyttää ohjelmassamme suoraan OpenStructeja ja siksi luomme olutpaikoille oman luokan _Places_ joka perii OpenStructin:

```ruby
class Place < OpenStruct
end
```

Oman luokan määritteleminen tekee koodista selkeämmän ja mahdollistaa tarvittaessa metodien liittämisen luokan olioille.

Luokkaamme siis käytetään siten, että annetaan sille konstruktoriparametriksi olutpaikkaa vastaava hash:

```ruby
irb(main):011:0> baari = Place.new places.first
=> #<Place id="6742", name="Pullman Bar", status="Beer Bar", reviewlink="https://beermapping.com/location/6742", proxylink="http://beerma...
irb(main):012:0> baari.name
=> "Pullman Bar"
irb(main):013:0> baari.zip
=> "00100"
irb(main):014:0>
```

Kirjoitetaan sitten kontrolleriin alustava koodi. Kovakoodataan etsinnän tapahtuvan aluksi Helsingistä ja luodaan ainoastaan ensimmäisestä löydetystä paikasta Place-olio:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"

    response = HTTParty.get "#{url}helsinki"
    places_from_api = response.parsed_response["bmp_locations"]["location"]
    @places = [ Place.new(places_from_api.first) ]

    render :index, status: 418
  end
end
```

Lisätään myös renderiin status-koodi 418, jotta Railsin käyttämä [Turbo](https://github.com/hotwired/turbo-rails) osaa renderöidä saman sivun uudestaan post-pyynnön jälkeen. Tämän statuskoodin avulla myös testit toimivat, sillä jos statuskoodiksi asettaisi esimerkiksi 303 [testit hajoaisivat](https://stackoverflow.com/a/30555199). Tämä hack on esimerkki huonosta koodista, mutta navigoidaksemme Turbon ja testien kanssa se on vaadittu.

Muokataan app/views/places/index.html.erb:tä siten, että se näyttää löydetyt ravintolat

```erb
<h1>Beer places search</h1>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>

<% if @places %>
  <ul>
    <% @places.each do |place| %>
      <li><%=place.name %></li>
    <% end %>
  </ul>
<% end %>
```

Koodi vaikuttaa toimivalta (huom. joudut uudelleenkäynnistämään Rails serverin jotta HTTParty-gem tulee ohjelman käyttöön).

Laajennetaan sitten koodi näyttämään kaikki panimot ja käyttämään lomakkeelta tulevaa parametria haettavana paikkakuntana:

```ruby
  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"

    @places = response.parsed_response["bmp_locations"]["location"].map do | place |
      Place.new(place)
    end

    render :index, status: 418
  end
```

Sovellus toimii muuten, mutta jos haetulla paikkakunnalla ei ole ravintoloita, tapahtuu virhe.

Käyttämällä debuggeria huomaamme, että näissä tapauksissa API:n palauttama paikkojen lista näyttää seuraavalta:

```ruby
{"id"=>nil, "name"=>nil, "status"=>nil, "reviewlink"=>nil, "proxylink"=>nil, "blogmap"=>nil, "street"=>nil, "city"=>nil, "state"=>nil, "zip"=>nil, "country"=>nil, "phone"=>nil, "overall"=>nil, "imagecount"=>nil}
```

Eli paluuarvona on hash. Jos taas haku löytää oluita paluuarvo on taulukko, jonka sisällä on hashejä. Virittelemme koodia ottamaan tämän huomioon. Koodi huomioi myös mahdollisuuden, jossa API palauttaa hashin, joka ei kuitenkaan vastaa olemassaolematonta paikkaa. Näin käy jos haetulla paikkakunnalla on vain yksi ravintola.

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"
    places_from_api = response.parsed_response["bmp_locations"]["location"]

    if places_from_api.is_a?(Hash) && places_from_api['id'].nil?
      redirect_to places_path, notice: "No places in #{params[:city]}"
    else
      places_from_api = [places_from_api] if places_from_api.is_a?(Hash)
      @places = places_from_api.map do | location |
        Place.new(location)
      end
      render :index, status: 418
    end
  end

end
```

Koodi on tällä hetkellä rumaa, mutta parantelemme sitä hetken kuluttua. Näytetään baareista enemmän tietoja sivulla. Määritellään näytettävät kentät Place-luokan staattisena metodina:

```ruby
class Place < OpenStruct
  def self.rendered_fields
    [:id, :name, :status, :street, :city, :zip, :country, :overall ]
  end
end
```

index.html.erb:n paranneltu koodi seuraavassa:

```erb
<h1>Beer places search</h1>

<p id="notice"><%= notice %></p>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>

<% if @places %>
  <table>
    <thead>
      <% Place.rendered_fields.each do |field| %>
        <th><%= field %></th>
      <% end %>
    </thead>
    <% @places.each do |place| %>
      <tr>
        <% Place.rendered_fields.each do |field| %>
          <td><%= place.send(field) %></td>
        <% end %>
      </tr>
    <% end %>
  </table>
<% end %>
```

Ravintolat näytetään nyt [HTML-taulukkona](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics).

## Olion metodien kutsuminen _send_-metodin avulla

Taulukon rivit muodostava koodi on muodossa

```erb
<tr>
  <% Place.rendered_fields.each do |field| %>
    <td><%= place.send(field) %></td>
  <% end %>
</tr>
```

Mistä tässä oikeastaan on kyse?

Ennen muutosta näkymä muodostettiin seuraavasti:

```erb
<% @places.each do |place| %>
  <li><%= place.name %></li>
<% end %>
```

eli jokaisesta baarista näytettiin sen nimi eli <code>place.name</code>

Nykyinen koodimme saa aikaan saman kuin seuraava, helpommin ymmärrettävissä muodossa oleva koodi:

```erb
<tr>
  <td><%= place.id %></td>
  <td><%= place.name %></td>
  <td><%= place.status %></td>
  <td><%= place.street %></td>
  <td><%= place.city %></td>
  <td><%= place.zip %></td>
  <td><%= place.country %></td>
  <td><%= place.overall %></td>
</tr>
```

Rubyssä olioiden metodeja voidaan kutsua myös "epäsuoraan" käyttämällä metodia <code>send</code>. Eli sen sijaan että sanomme <code>place.name</code> voimme tehdä metdoikutsun syntaksilla <code>place.send(:name)</code>. Olutpaikan rivin muodostaminen voidaan siis muuttaa muotoon:

```erb
<tr>
  <td><%= place.send(:id) %></td>
  <td><%= place.send(:name) %></td>
  <td><%= place.send(:status) %></td>
  <td><%= place.send(:street) %></td>
  <td><%= place.send(:city) %></td>
  <td><%= place.send(:zip) %></td>
  <td><%= place.send(:country) %></td>
  <td><%= place.send(:overall) %></td>
</tr>
```

Ja koska määrittelimme metodin <code>Place.rendered_fields</code> palauttamaan listan <code>[ :id, :name, :status, :street, :city, :zip, :country, :overall ]</code>, voimme generoida td-tagit <code>each</code>-loopin avulla:

```erb
<tr>
  <% Place.rendered_fields.each do |field| %>
    <td><%= place.send(field) %></td>
  <% end %>
</tr>
```

Kannattaako näin tehdä? Kyse on osittain makuasiasta. Määrittelemällä näytettävien kenttien listan saimme nyt tehtyä myös taulukon otsakerivin looppaamalla:

```erb
<thead>
  <% Place.rendered_fields.each do |field| %>
    <td><%= field %></td>
  <% end %>
</thead>
```

Jos nyt päättäisimme lisätä tai poistaa jotain näytettäviä kenttiä, riittää kun muutamme luokan <code>Places</code> määrittelemää listaa ja näkymään ei tarvitse erikseen koskea:

```ruby
class Place < OpenStruct
  def self.rendered_fields
    [ :id, :name, :status, :street, :city, :zip, :country, :overall ]
  end
end
```

## Erikoismerkkejä sisältävät nimet

Sovelluksessamme on vielä pieni ongelma Jos yritämme etsiä New Yorkin olutravintoloita on seurauksena virhe. Välilyönnit on korvattava URL:ssä koodilla %20. Korvaamista ei kannata tehdä itse 'käsin', välilyönti ei nimittäin ole ainoa merkki joka on koodattava URL:iin. Kuten arvata saattaa, on Railsissa tarjolla tarkoitusta varten valmis metodi <code>ERB::Util.url_encode</code>. Kokeillaan metodia konsolista:

```ruby
> ERB::Util.url_encode("St John's")
 => "St%20John%27s"
>
```

Tehdään nyt muutos koodiin korvaamalla HTTP GET -pyynnön tekevä rivi seuraavalla:

```ruby
response = HTTParty.get "#{url}#{ERB::Util.url_encode(params[:city])}"
```

> ## Tehtävä 1
>
> Tee edelläoleva koodi ohjelmaasi. Lisää myös navigointipalkkiin linkki olutpaikkojen hakusivulle

## Places-kontrollerin refaktorointi

Railsissa kontrollereiden ei tulisi sisältää sovelluslogiikkaa. Ulkopuoleisen API:n käyttö onkin syytä eristää omaksi luokakseen. Luontevin paikka luokan koodille on hakemisto _lib_. Sijoitetaan siis seuraava tiedostoon _lib/beermapping_api.rb_:

```ruby
class BeermappingApi
  def self.places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.map do | place |
      Place.new(place)
    end
  end

  def self.key
    "731955affc547174161dbd6f97b46538"
  end
end
```

Luokka siis määrittelee stattisen metodin, joka palauttaa taulukon parametrina määritellystä kaupungista löydetyistä olutpaikoista. Jos paikkoja ei löydy, on taulukko tyhjä. API:n eristävä luokka ei ole vielä viimeiseen asti hiotussa muodossa, sillä emme vielä täysin tiedä mitä muita metodeja tarvitsemme.

Jotta _lib_-hakemistoon sijoitettu koodi toimisi (sekä omalla koneella, että Fly.io:ssa ja Herokussa), tulee tiedostoon _config/application.rb_ lisätä seuraavat kaksi riviä

```ruby
config.autoload_paths << Rails.root.join("lib")
config.eager_load_paths << Rails.root.join("lib")
```

lisäys tulee tehdä _Application_-luokan määrittelyn sisälle

```ruby
module Ratebeer
  class Application < Rails::Application
    # ...

    # lisäys tänne
  end
end
```

Jotta muutokset tulevat voimaan täytyy sovellus käynnistää uudestaan.

Kontrollerista tulee nyt siisti:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    @places = BeermappingApi.places_in(params[:city])
    if @places.empty?
      redirect_to places_path, notice: "No locations in #{params[:city]}"
    else
      render :index, status: 418
    end
  end
end
```

## Olutpaikkojen etsimistoiminnon testaaminen

Tehdään seuraavaksi Rspec-testejä toteuttamallemme toiminnallisuudelle. Uusi toiminnallisuutemme käyttää siis hyväkseen ulkoista palvelua. Testit on kuitenkin syytä kirjoittaa siten, ettei ulkoista palvelua käytetä. Onneksi ulkoisen rajapinnan korvaaminen stub-komponentilla on Railsissa helppoa.

Päätämme jakaa testit kahteen osaan. Korvaamme ensin ulkoisen rajapinnan kapseloivan luokan <code>BeermappingApi</code> toiminnallisuuden stubien avulla kovakoodatulla toiminnallisuudella. Testi siis testaa, toimiiko places-sivu oikein olettaen, että <code>BeermappingApi</code>-komponentti toimii.

Testaamme sitten erikseen Rspecillä kirjoitettavilla yksikkötesteillä <code>BeermappingApi</code>-komponentin toiminnan.

Aloitetaan siis web-sivun places-toiminnallisuuden testaamisesta. Tehdään testiä varten tiedosto /spec/features/places_spec.rb

```ruby
require 'rails_helper'

describe "Places" do
  it "if one is returned by the API, it is shown at the page" do
    allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
      [ Place.new( name: "Oljenkorsi", id: 1 ) ]
    )

    visit places_path
    fill_in('city', with: 'kumpula')
    click_button "Search"

    expect(page).to have_content "Oljenkorsi"
  end
end
```

Testi alkaa heti mielenkiintoisella komennolla:

```ruby
allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
  [ Place.new( name: "Oljenkorsi", id: 1 ) ]
)
```

Komento "kovakoodaa" luokan <code>BeermappingApi</code> metodin <code>places_in</code> vastaukseksi määritellyn yhden Place-olion sisältävän taulukon, jos metodia kutsutaan parametrilla "kumpula".

Kun nyt testissä tehdään HTTP-pyyntö places-kontrollerille, ja kontrolleri kutsuu API:n metodia <code>places_in</code>, metodin todellisen koodin suorittamisen sijaan places-kontrollerille palautetaankin kovakoodattu vastaus.

> ## Tehtävä 2
>
> Laajenna testiä kattamaan seuraavat tapaukset:
>
> - jos API palauttaa useita olutpaikkoja, kaikki näistä näytetään sivulla
> - jos API ei löydä paikkakunnalta yhtään olutpaikkaa (eli paluuarvo on tyhjä taulukko), sivulla näytetään ilmoitus "No locations in _etsitty paikka_"
>
> Viikon 3 luku [kirjautumisen hienosäätöä](https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/web/viikko3.md#kirjautumisen-hienos%C3%A4%C3%A4t%C3%B6%C3%A4) antaa vihjeitä toista kohtaa varten.

Siirrytään sitten luokan <code>BeermappingApi</code> testaamiseen. Luokka siis tekee HTTP GET -pyynnön HTTParty-kirjaston avulla Beermapping-palveluun. Voisimme edellisen esimerkin tapaan stubata HTTPartyn get-metodin. Tämän on kuitenkin hieman ikävää, sillä metodi palauttaa <code>HTTPartyResponse</code>-olion ja sellaisen muodostaminen stubauksen yhteydessä käsin ei välttämättä ole kovin mukavaa.

Parempi vaihtoehto onkin käyttää gemiä _webmock_ https://github.com/bblimke/webmock/ sillä se mahdollistaa stubauksen HTTPartyn käyttämän kirjaston tasolla.

Otetaan gem käyttöön lisäämällä Gemfilen **test-scopeen** rivi <code>gem 'webmock'</code>;

```ruby
group :test do
  # ...
  gem 'webmock'
end
```

**HUOM: webmock on määriteltävä _ainoastaan_ test-scopeen, muuten se estää kaikki sovelluksen tekemät HTTP-pyynnöt!**

Suoritetaan <code>bundle install</code>.

Tiedostoon `spec/rails_helper.rb` pitää vielä lisätä rivi:

```ruby
require 'webmock/rspec'
```

Webmock-kirjaston käyttö on melko helppoa. Esim. seuraava komento stubaa _jokaiseen_ URLiin (määritelty regexpillä <code>/.\*/</code>) tulevan GET-pyynnön palauttamaan 'Lapin kullan' tiedot XML-muodossa:

```ruby
stub_request(:get, /.*/).to_return(body: "<beer><name>Lapin kulta</name><brewery>Hartwall</brewery></beer>", headers:{ 'Content-Type' => "text/xml" })
```

Eli jos kutsuisimme komennon tehtyämme esim. <code>HTTParty.get("http://www.google.com")</code> olisi vastauksena

```xml
<beer>
  <name>Lapin kulta</name>
  <brewery>Hartwall</brewery>
</beer>
```

Tarvitsemme siis testiämme varten sopivan "kovakoodatun" datan, joka kuvaa Beermapping-palvelun HTTP GET -pyynnön palauttamaa XML:ää.

Eräs tapa testisyötteen generointiin on kysyä se rajapinnalta itseltään, eli tehdään komentoriviltä <code>curl</code>-komennolla HTTP GET -pyyntö:

```ruby
mluukkai@melkki$ curl http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/turku
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
```

Nyt voimme copypastata HTTP-pyynnön palauttaman XML-muodossa olevan tiedon testiimme. Jotta saamme XML:n varmasti oikein sijoitetuksi merkkijonoon, käytämme hieman erikoista syntaksia
ks. http://blog.jayfields.com/2006/12/ruby-multiline-strings-here-doc-or.html jossa merkkijono sijoitetaan merkkien <code><<-END_OF_STRING</code> ja <code>END_OF_STRING</code> väliin.

Seuraavassa tiedostoon spec/lib/beermapping_api_spec.rb sijoitettava testikoodi (päätimme sijoittaa koodin alihakemistoon lib koska testin kohde on lib-hakemistossa oleva apuluokka):

```ruby
require 'rails_helper'

describe "BeermappingApi" do
  it "When HTTP GET returns one entry, it is parsed and returned" do

    canned_answer = <<-END_OF_STRING
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
    END_OF_STRING

    stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

    places = BeermappingApi.places_in("turku")

    expect(places.size).to eq(1)
    place = places.first
    expect(place.name).to eq("Panimoravintola Koulu")
    expect(place.street).to eq("Eerikinkatu 18")
  end

end
```

Testi siis ensin määrittelee, että URL:iin joka loppuu merkkijonoon "turku" (määritelty regexpillä <code>/.\*turku/</code>) kohdistuvan HTTP GET -kutsun palauttamaan kovakoodatun XML:n, HTTP-kutsun palauttamaan headeriin määritellään, että palautettu tieto on XML-muodossa. Ilman tätä määritystä HTTParty-kirjasto ei osaa parsia HTTP-pyynnön palauttamaa dataa oikein.

Itse testi tapahtuu suoraviivaisesti tarkastelemalla BeermappingApi:n metodin <code>places_in</code> palauttamaa taulukkoa.

_Huom:_ stubasimme testissä ainoastaan merkkijonoon "turku" loppuviin URL:eihin (<code>/.\*turku</code>) kohdistuvat HTTP GET -kutsut. Jos testin suoritus aiheuttaa jonkin muunlaisen HTTP-kutsun, huomauttaa testi tästä:

```ruby
) BeermappingApi When HTTP GET returns no entries, an empty array is returned
     Failure/Error: places = BeermappingApi.places_in("kumpula")
     WebMock::NetConnectNotAllowedError:
       Real HTTP connections are disabled. Unregistered request: GET http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/kumpula

       You can stub this request with the following snippet:

       stub_request(:get, "http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/kumpula").
         to_return(:status => 200, :body => "", :headers => {})
```

Kuten virheilmoitus antaa ymmärtää, voidaan komennon <code>stub_request</code> avulla stubata myös merkkijonona määriteltyyn yksittäiseen URL:iin kohdistuva HTTP-kutsu. Sama testi voi myös sisältää useita <code>stub_request</code>-kutsuja, jotka kaikki määrittelevät eri URLeihin kohdistuvien pyyntöjen vastaukset.

> ## Tehtävä 3
>
> Laajenna testejä kattamaan seuraavat tapaukset
>
> - HTTP GET ei palauta yhtään paikkaa, eli tällöin metodin <code>places_in</code> tulee palauttaa tyhjä taulukko
> - HTTP GET palauttaa useita paikkoja, eli tällöin metodin <code>places_in</code> tulee palauttaa kaikki HTTP-kutsun XML-muodossa palauttamat ravintolat taulukollisena Place-olioita
>
> Stubatut vastaukset kannattaa jälleen muodostaa curl-komennon avulla API:n tehdyillä kyselyillä
>
> Muista käyttää debuggeria apuna testatessa

Erilaisten vale- ja lavastekomponenttien tekeminen eli metodien ja kokonaisten olioiden stubaus sekä mockaus on hyvin laaja aihe. Voit lukea aiheesta Rspeciin liittyen seuraavasta http://rubydoc.info/gems/rspec-mocks/

Nimityksiä stub- ja mock-olio tai "stubaaminen ja mockaaminen" käytetään usein varsin huolettomasti. Onneksi Rails-yhteisö käyttää termejä oikein. Lyhyesti ilmaistuna stubit ovat olioita, joihin on kovakoodattu valmiiksi metodien vastauksia. Mockit taas toimivat myös stubien tapaan kovakoodattujen vastausten antajana, mutta sen lisäksi mockien avulla voidaan määritellä odotuksia siitä miten niiden metodeja kutsutaan. Jos testattavana olevat oliot eivät kutsu odotetulla tavalla mockien metodeja, aiheutuu tästä testivirhe.

Mockeista ja stubeista lisää esim. seuraavassa: http://martinfowler.com/articles/mocksArentStubs.html

## Suorituskyvyn optimointi

Tällä hetkellä sovelluksemme toimii siten, että se tekee kyselyn beermappingin palveluun aina kun jonkin kaupungin ravintoloita haetaan. Voisimme tehostaa sovellusta muistamalla viime aikoina suoritettuja hakuja.

Rails tarjoaa avain-arvopari-periaatteella toimivan hyvin helppokäyttöisen cachen eli välimuistin sovelluksen käyttöön.

Välimuisti on oletusarvoisesti poissa päältä. Saat sen päälle suorittamalla komentoriviltä komennon <code>rails dev:cache</code>

Muuta myös tiedostosta _config/environments/development.rb_ rivit

```ruby
config.cache_store = :memory_store
config.cache_store = :null_store
```

muotoon

```ruby
config.cache_store = :file_store, 'tmp/cache_store'
```

sekä uudelleenkäynnistä konsoli ja sovellus.

Cacheen päästään käsiksi muuttujaan <code>Rails.cache</code> talletetun olion kautta. Kokeillaan konsolista:

```ruby
> Rails.cache.write "avain", "arvo"
 => true
> Rails.cache.read "avain"
 => "arvo"
> Rails.cache.read "kumpula"
 => nil
> Rails.cache.write "kumpula", Place.new(name: "Oljenkorsi")
 => true
> Rails.cache.read "kumpula"
 => #<Place:0x00000104628608 @name="Oljenkorsi">
```

Cacheen voi tallettaa melkein mitä vaan. Ja rajapinta on todella yksinkertainen, ks. http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html

Ensimmäinen metodikutsu siis aiheuttaa tietokantahaun ja tallettaa olion välimuistiin. Seuraava kutsu saa avainta vastaavan olion suoraan välimuistista.

Oletusarvoisesti Railsin cache tallettaa avain-arvo-parit tiedostojärjestelmään. Cachen käyttämä talletustapa on kuitenkin konfiguroitavissa, ks. http://guides.rubyonrails.org/caching_with_rails.html#cache-stores

Tuotantokäytössä välimuistin datan tallettaminen tiedostojärjestelmään ei ole suorituskyvyn kannalta optimaalista. Parempi ratkaisu onkin esim. [Memcached](http://memcached.org/), ks. tarkemmin esim. https://devcenter.heroku.com/articles/building-a-rails-3-application-with-memcache

**Huom:** koska testimme alkavat pian testaamaan Rails.cachea hyväksikäyttävää koodia, kannattaa cache konfiguroida käyttämään testien aikana talletuspaikkanaan tiedostojärjestelmän sijaan **keskusmuistia**. Tämä tapahtuu lisäämällä tiedostoon _config/environments/test.rb_ rivi

```ruby
config.cache_store = :memory_store
```

Viritellään luokkaa <code>BeermappingApi</code> siten, että se tallettaa tehtyjen kyselyjen tulokset välimuistiin. Jos kysely kohdistuu jo välimuistissa olevaan kaupunkiin, palautetaan tulos välimuistista.

```ruby
class BeermappingApi
  def self.places_in(city)
    city = city.downcase

    places = Rails.cache.read(city)
    return places if places

    places = get_places_in(city)
    Rails.cache.write(city, places)
    places
  end

  def self.get_places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.map do | place |
      Place.new(place)
    end
  end

  def self.key
    "731955affc547174161dbd6f97b46538"
  end
end
```

Avaimena käytetään pienillä kirjaimilla kirjoitettua kaupungin nimeä. Koodi on melko suoraviivainen, jos avainta vastaavat olutpaikat löytyvät cachesta (eli arvo ei ole nil), palautetaan ne. Jos taas cachessa ei ole kaupungin olutpaikkoja, haetaan ne metodilla <code>get_places_in(city)</code> talletetaan cacheen ja palautetaan metodin kutsujalle.

Jos teemme nyt haun kaksi kertaa peräkkäin esim. New Yorkin oluista, huomaamme, että toisella kerralla vastaus tulee huomattavasti nopeammin.

Pääsemme sovelluksen välimuistiin tallettamaan dataan käsiksi myös konsolista:

```ruby
> Rails.cache.read("helsinki").map(&:name)
 => ["Pullman Bar", "Belge", "Suomenlinnan Panimo", "St. Urho's Pub", "Kaisla", "Pikkulintu", "Bryggeri Helsinki", "Stadin Panimo", "Panimoravintola Bruuveri"]
>
```

Konsolista käsin on myös mahdollista tarvittaessa poistaa tietylle avaimelle talletettu data:

```ruby
> Rails.cache.delete("helsinki")
 => true
> Rails.cache.read("helsinki")
 => nil
>
```

Voisimme yksinkertaistaa koodia hieman käyttämällä Rails.cachen metodia [fetch](https://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html#method-i-fetch)

```ruby
class BeermappingApi
  def self.places_in(city)
    city = city.downcase
    Rails.cache.fetch(city) { get_places_in(city) }
  end

  def get_places_in(city)
    # ...
  end
end
```

Fetch toimii siten, että jos välimuiststa löytyy dataa sen parametrina olevalla avaimella, palauttaa metodi välimuistissa olevan datan. Jos välimuistissa ei ole avainta vastaavaa dataa, suoritetaan komennon mukana oleva koodilohko ja talletetaan koodilohkon paluuarvo välimuistiin. Myös itse komento _fetch_ palauttaa lohkon saaman arvon.

## Vanhentunut data

Välimuistin käytön ongelmana on mahdollinen tiedon epäajantasaisuus. Eli jos joku lisää ravintoloita beermappingin sivuille, välimuistissamme säilyy edelleen vanha data. Jollain tavalla tulisi siis huolehtia, että välimuistiin ei pääse jäämään liian vanhaa dataa.

Yksi ratkaisu olisi aika ajoin nollata välimuistissa oleva data komennolla:

    Rails.cache.clear

Tilanteeseemme paremmin sopiva ratkaisu on määritellä välimuistiin talletettavalle datalle enimmäiselinikä.

> ## Tehtävä 4
>
> ### tämä ei ole viikon tärkein tehtävä, joten älä jää jumittamaan tähän jos kohtaat ongelmia
>
> Määrittele välimuistiin talletettaville ravintolatiedoille enimmäiselinikä, esim. 1 viikko. Testatessasi tehtävän toimintaa, kannattaa kuitenkin käyttää pienempää elinikää, esim. yhtä minuuttia.
>
> Tehtävän tekeminen ei edellytä kovin suuria muutoksia koodiisi, oikeastaan muutoksia tarvitaan vain _yhdelle_ riville. Tarvittavat vihjeet löydät sivulta http://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-store Ajan käsittelyssä auttaa http://guides.rubyonrails.org/active_support_core_extensions.html#time
>
> **Huom:** kuten aina, nytkin kannattaa testailla enimmäiseliniän asettamisen toimivuutta konsolista käsin!
>
> **Huom2:** jos saat välimuistin sekaisin, muista <code>Rails.cache.clear</code> ja <code>Rails.cache.delete avain</code>

## Testit ja cache

Tehtävässä 3 teimme Webmock-gemin avulla testejä luokalle <code>BeermappingApi</code>. On syytä huomioida, että välimuisti vaikuttaa myös testaamiseen, ja olisikin kenties parasta testata erikseen tilanne, jossa data ei löydy välimuistista (cache miss) sekä tilanne, jossa data on jo välimuistissa (cache hit):

```ruby
require 'rails_helper'

describe "BeermappingApi" do
  describe "in case of cache miss" do

    before :each do
      Rails.cache.clear
    end

    it "When HTTP GET returns one entry, it is parsed and returned" do
      canned_answer = <<-END_OF_STRING
  <?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
  END_OF_STRING

      stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

      places = BeermappingApi.places_in("turku")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("Panimoravintola Koulu")
      expect(place.street).to eq("Eerikinkatu 18")
    end

  end

  describe "in case of cache hit" do
    before :each do
      Rails.cache.clear
    end

    it "When one entry in cache, it is returned" do
      canned_answer = <<-END_OF_STRING
  <?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
  END_OF_STRING

      stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

      BeermappingApi.places_in("turku")
      places = BeermappingApi.places_in("turku")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("Panimoravintola Koulu")
      expect(place.street).to eq("Eerikinkatu 18")
    end
  end
end
```

Ensimmäisessä <code>describe</code>-lohkossa oleva <code>before :each</code>-lohko tyhjentää välimuistin ennen testien suorittamista, eli kun itse testi tekee metodikutsun <code>BeermappingApi.places_in</code>, haetaan olutpaikkojen tiedot HTTP-pyynnöllä. Toisessa describe-lohkossa taas testeissä kutsutaan metodia <code>BeermappingApi.places_in</code> kaksi kertaa. Ensimmäinen kutsu varmistaa, että haettavan paikan tiedot talletetaan välimuistiin. Toisen kutsun tulos tulee välimuistista ja tulosta testataan testikoodissa.

Testi sisältää nyt paljon toisteisuutta ja kaipaisi refaktorointia, mutta menemme kuitenkin eteenpäin.

**Vielä uusi huomautus asiasta:** koska testaamme Rails.cachea hyväksikäyttävää koodia, kannattaa cache konfiguroida käyttämään testien aikana talletuspaikkanaan tiedostojärjestelmän sijaan **keskusmuistia**. Tämä tapahtuu lisäämällä tiedostoon _config/environments/test.rb_ rivi

```ruby
config.cache_store = :memory_store
```

## Sovelluskohtaisen datan tallentaminen

Koodissamme API-key on nyt kirjoitettu sovelluksen koodiin. Tämä ei tietenkään ole ollenkaan järkevää. Railsissa on useita mahdollisuuksia konfiguraatiotiedon tallentamiseen, ks. https://guides.rubyonrails.org/configuring.html

Ehkä paras vaihtoehto suhteellisen yksinkertaisen sovelluskohtaisen datan tallettamiseen ovat ympäristömuuttujat. Esimerkki seuraavassa:

Asetetaan ensin komentoriviltä ympäristömuuttujalle <code>BEERMAPPING_APIKEY</code>

```ruby
mluukkai@melkki$ export BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538"
```

Rails-sovellus pääsee ympäristömuuttujiin käsiksi hash-tyyppisen muuttujan <code>ENV</code> kautta:

```ruby
> ENV['BEERMAPPING_APIKEY']
 => "731955affc547174161dbd6f97b46538"
>
```

Poistetaan kovakoodattu apiavain ja luetaan se ympäristömuuttujasta:

```ruby
class BeermappingApi
  # ...

  def self.key
    return nil if Rails.env.test? # testatessa ei apia tarvita, palautetaan nil
    raise 'BEERMAPPING_APIKEY env variable not defined' if ENV['BEERMAPPING_APIKEY'].nil?
    ENV.fetch('BEERMAPPING_APIKEY')
  end
end
```

Koodiin on myös lisätty suoritettavaksi poikkeus tilanteessa, jossa apiavainta ei ole määritelty.

Ympäristömuuttujan arvon tulee siis olla määritelty jos käytät olutravintoloiden hakutoimintoa. Saat määriteltyä ympäristömuuttujan käynnistämällä sovelluksen seuraavasti:

```ruby
mluukkai@melkki$ export BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538"
mluukkai@melkki$ rails s
```

tai määrittelemällä ympäristömuuttujan käynnistyskomennon yhteydessä:

```ruby
mluukkai@melkki$ BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538" rails s
```

Voit myös määritellä ympäristömuuttujan arvon (export-komennolla) komentotulkin käynnistyksen yhteydessä suoritettavassa tiedostossa (.zshrc, .bashrc tai .profile komentotulkista riippuen).

Ympäristömuuttujille on helppo asettaa arvo myös Fly.io:ssa ks. https://fly.io/docs/reference/secrets/#setting-secrets ja Herokussa, ks. https://devcenter.heroku.com/articles/config-vars

**HUOM** Jos haluat pitää Github Actionsin toimintakunnossa, joudut määrittelemään ympäristömuuttujan workflown-konfiguraatioon ks.
https://docs.github.com/en/actions/learn-github-actions/environment-variables

## Lisäselvennys kontrollerin toiminnasta

Tarkastellaan hieman tarkemmin kontrollerien <code>show</code>-metodien toimintaperiaatetta. Seuraavaakin tehtävää silmälläpitäen kerrataan asiaa.

Tarkastellaan panimon kontrolleria. Yksittäisen panimon näyttämisestä vastaava kontrollerimetodi ei sisällä mitään koodia:

```ruby
def show
end
```

oletusarvoisesti renderöityvä näkymätemplate app/views/breweries/show.html.erb kuitenkin viittaa muuttujaan <code>@brewery</code>:

```ruby
<h2><%= @brewery.name %></h2>

<p>
  <em>Established year:</em>
  <%= @brewery.year %>
</p>
```

eli miten muuttuja saa arvonsa? Arvo asetetaan kontrollerissa _esifiltteriksi_ määritellyssä metodissa <code>set_brewery</code>.

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]
  #...

  def set_brewery
    @brewery = Brewery.find(params[:id])
  end
end
```

kontrolleri siis määrittelee, että aina ennen metodin <code>show</code> suorittamista suoritetaan koodi

```ruby
@brewery = Brewery.find(params[:id])
```

joka lataa panimo-olion muistista ja tallettaa sen näkymää varten muuttujaan.

Kuten koodista on pääteltävissä, kontrolleri pääsee käsiksi panimon id:hen <code>params</code>-hashin kautta. Mihin tämä perustuu?

Kun katsomme sovelluksen routeja joko komennolla <code>rails routes</code> tai selaimesta (menemällä mihin tahansa epävalidiin osoitteeseen kuten localhost:3000/foobar), huomaamme, että yksittäiseen panimoon liittyvä routetieto on seuraava

```ruby
brewery_path	 GET	 /breweries/:id(.:format)	 breweries#show
```

eli yksittäisen panimon URL on muotoa _breweries/42_ missä lopussa oleva luku on panimon id. Kuten polkumäärittely vihjaa, sijoitetaan panimon id <code>params</code>-hashin avaimen <code>:id</code> arvoksi.

Voisimme määritellä 'parametrillisen' polun myös käsin. Jos lisäisimme routes.rb:hen seuraavan

```ruby
get 'panimo/:id', to: 'breweries#show'
```

pääsisi yksittäisen panimon sivulle osoitteesta http://localhost:3000/panimo/42. Osoitteen käsittelisi edelleen kontrollerin metodi <code>show</code>, joka pääsisi käsiksi id:hen tuttuun tapaan <code>params</code>-hashin kautta.

Jos taas päättäisimme käyttää jotain muuta kontrollerimetodia, ja määrittelisimme reitin seuraavasti

```ruby
get 'panimo/:panimo_id', to: 'breweries#nayta'
```

kontrollerimetodi voisi olla esim. seuraava:

```ruby
def nayta
  @brewery = Brewery.find(params[:panimo_id])
  render :index
end
```

eli tällä kertaa routeissa määriteltiin, että panimon id:hen viitataan <code>params</code>-hashin avaimella <code>:panimo_id</code>.

## Ravintolan sivu

> ## Tehtävät 5-6 (vastaa kahta tehtävää)
>
> Tee sovellukselle ominaisuus, jossa ravintolan nimeä klikkaamalla avautuu oma sivu, jossa on näkyvillä ravintolan yhteystiedot.
>
> - ravintolan urliksi kannattaa vailta Rails-konvention mukainen places/:id, routes.rb voi näyttää esim. seuraavalta:
>
> ```ruby
> resources :places, only: [:index, :show]
> # mikä generoi samat polut kuin seuraavat kaksi
> # get 'places', to: 'places#index'
> # get 'places/:id', to: 'places#show'
>
> post 'places', to: ' places#search'
> ```
>
> - HUOM: ravintolan tiedot löytyvät hieman epäsuorasti cachesta siinä vaiheessa kun ravintolan sivulle ollaan menossa. Jotta pääset tietoihin käsiksi on ravintolan id:n lisäksi "muistettava" kaupunki, josta ravintolaa etsittiin, tai edelliseksi tehdyn search-operaation tulos. Yksi tapa muistamiseen on käyttää sessiota, ks. https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/web/viikko3.md#k%C3%A4ytt%C3%A4j%C3%A4-ja-sessio
>
> Toinen tapa toiminnallisuuden toteuttamiseen on sivulla http://beermapping.com/api/reference/ oleva "Locquery Service"
>
> _HUOM1_ Koska _Place_ ei ole ActiveRecord-luokka, ei seuraava toimi
>
> `link_to place.name, place`
>
> linkin kohdeosoite on määriteltävä pidemmässä muodossa
>
> `link_to place.name, place_path(place.id)`
>
> _HUOM2_ jos sinulla on vaikeuksia tehdä ravinotalan nimestä klikattava linkki, voit muuttaa taulukon _send_-metodia käyttävästä versiosta seuraavaan "karvalakkimalliin":
>
> ```erb
> <table>
>  <thead>
>    <th>id</th>
>    <th>name</th>
>    <th>status</th>
>    <th>street</th>
>    <th>city</th>
>    <th>zip</th>
>    <th>country</th>
>    <th>overall</th>
>  </thead>
>  <% @places.each do |place| %>
>    <tr>
>      <td><%= place.id %></td>
>      <td><%= place.name %></td>
>      <td><%= place.status %></td>
>      <td><%= place.street %></td>
>      <td><%= place.city %></td>
>      <td><%= place.zip %></td>
>      <td><%= place.country %></td>
>      <td><%= place.overall %></td>
>    </tr>
>  <% end %>
> </table>
> ```
>
> Kokeile hajottaako ravintoloiden sivun lisääminen mitään olemassaolevaa testiä. Jos, niin voit yrittää korjata testit. Välttämätöntä se ei kuitenkaan tässä vaiheessa ole.

Tehtävän jälkeen sovelluksesi voi näyttää esim. seuraavalta:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-2.png)

## Oluen reittaus suoraan oluen sivulta

Tällä hetkellä reittaukset luodaan erilliseltä sivulta, jolta reitattava olut valitaan erillisestä valikosta. Olisi luontevampaa, jos reittauksen voisi tehdä myös suoraan kunkin oluen sivulta.

Vaihtoehtoisia toteutustapoja on useita. Tutkitaan seuraavassa ehkä helpointa ratkaisua. Käytetään <code>form_for</code>-helperiä, eli luodaan lomake pohjalla olevaa olia hyödyntäen. **BeersControllerin** metodiin show tarvitaan pieni muutos:

```ruby
def show
  @rating = Rating.new
  @rating.beer = @beer
end
```

Eli siltä varalta, että oluelle tehdään reittaus, luodaan näykymätemplatea varten reittausolio, joka on jo liitetty tarkasteltavaan olioon. Reittausolio on luotu new:llä eli sitä ei siis ole talletettu kantaan, huomaa, että ennen metodin <code>show</code> suorittamista on suoritettu esifiltterin avulla määritelty komento, joka hakee kannasta tarkasteltavan oluen: <code>@beer = Beer.find(params[:id])</code>

Näkymätemplatea /views/beers/show.html.erb muutetaan seuraavasti:

```erb
<p style="color: green"><%= notice %></p>

<%= render @beer %>

<% if current_user %>
  <h4>give a rating:</h4>

  <%= form_with(model: @rating) do |form| %>
    <%= form.hidden_field :beer_id %>
    score: <%= form.number_field :score %>
    <%= form.submit "Create rating" %>
  <% end %>

  <div>
    <%= link_to "Edit this beer", edit_beer_path(@beer) %>
    <%= button_to "Destroy this beer", @beer, method: :delete %>
  </div>
<% end %>
```

Jotta lomake lähettäisi oluen id:n, tulee <code>beer_id</code>-kenttä lisätä lomakkeeseen. Emme kuitenkaan halua käyttäjän pystyvän manipuloimaan kenttää, joten kenttä on määritelty lomakkeelle <code>hidden_field</code>:iksi.

Koska lomake on luotu <code>form_with</code>-helperillä, tapahtuu sen lähettäminen automaattisesti HTTP POST -pyynnöllä <code>ratings_path</code>:iin eli reittauskontrollerin <code>create</code>-metodi käsittelee lomakkeen lähetyksen. Kontrolleri toimii ilman muutoksia!

Ratkaisussa on pieni ongelma. Jos reittauksessa yritetään antaa epävalidi pistemäärä:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-4.png)

renderöi kontrolleri (eli reittauskontrollerin metodi <code>create</code>) oluen näkymän sijaan uuden reittauksen luomislomakkeen:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-3.png)

Ongelman voisi kiertää katsomalla mistä osoitteesta create-metodiin on tultu ja renderöidä sitten oikea sivu riippuen tulo-osoitteesta. Emme kuitenkaan tee nyt tätä muutosta.

Korjaamme ensin erään vielä vakavamman ongelman. Edellistä kahta kuvaa tarkastelemalla huomaamme että jos reittauksen (joka yritetään antaa oluelle _Weihenstephaner Hefeweizen_) validointi epäonnistuu, ei tehty oluen valinta ole enää tallessa (valittuna on _Iso 3_).

Ongelman syynä on se, että pudotusvalikon vaihtoehdot generoivalle metodille <code>options_from_collection_for_select</code> ei ole kerrottu mikä vaihtoehdoista tulisi valita oletusarvoisesti, ja tälläisessä tilanteessa valituksi tulee kokoelman ensimmäinen olio. Oletusarvoinen valinta kerrotaan antamalla metodille neljäs parametri:

```erb
options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id)
```

Eli muutetaan näkymätemplate app/views/ratings/new.html.erb seuraavaan muotoon:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@rating.errors.count, "error") %> prohibited rating from being saved:</h2>

      <ul>
      <% @rating.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>
<% end %>
```

Sama ongelma itse asiassa vaivaa muutamia sovelluksemme lomakkeita, kokeile esim. mitä tapahtuu kun yrität luoda oluen jolle et anna nimeä. Korjaa lomake jos haluat.

> ## Tehtävä 7
>
> Tee myös olutkerhoihin liittyminen mahdolliseksi suoraan olutkerhon sivulta.
>
> Kannattaa noudattaa samaa toteutusperiaatetta kuin oluen sivulta tapahtuvassa reittaamisessa, eli lisää olutseuran sivulle lomake, jonka avulla voidaan luoda uusi <code>Membership</code>-olio, joka liittyy olutseuraan ja kirjautuneena olevaan käyttäjään. Lomakkeen hidden_field kenttiin voi asettaa arvot käyttämällä <code>value</code>-parametriä:
>
> ```erb
> <%= form_with(model: @membership) do |form| %>
>   <%= form.hidden_field :beer_club_id, value: @beer_club.id %>
>   <%= form.hidden_field :user_id, value: current_user.id %>
>   <%= form.submit "Join the beer club" %>
> <% end %>
> ```

Hienosäädetään olutseuraan liittymistä

> ## Tehtävä 8
>
> Tee ratkaisustasi sellainen, jossa liittymisnappia ei näytetä jos kukaan ei ole kirjautunut järjestelmään tai jos kirjautunut käyttäjä on jo seuran jäsen.
>
> Muokkaa koodiasi siten (membership-kontrollerin sopivaa metodia), että olutseuraan liittymisen jälkeen selain ohjautuu olutseuran sivulle ja sivu näyttää allaolevan kuvan mukaisen ilmoituksen uuden käyttäjän liittymisestä.

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5.png)

> ## Tehtävä 9
>
> Laajennetaan toiminnallisuutta vielä siten, että jäsenten on mahdollisuus erota olutseurasta.
>
> Lisää olutseuran sivulle nappi, joka mahdollistaa seurasta eroamisen. Napin tulee olla näkyvillä vain jos kirjautunut käyttäjä menee sellaisen seuran sivulle, jossa hän on jäsenenä. Eroamisnappia painamalla jäsenyys tuhoutuu ja käyttäjä ohjautuu omalle sivulleen jolla tulee näyttää ilmoitus eroamisesta, allaolevat kuvat selventävät haluttua toiminnallisuutta.
>
> Vihje: eroamistoiminnallisuuden voi toteuttaa liittymistoiminnalisuuden tapaan olutseuran sivulle sijoitettavalla lomakkeella. Lomakkeen käyttämäksi HTTP-metodiksi tulee määritellä delete:
>
> ```erb
> <%= form_with(..., method: :delete) do |form| %>
>   ...
>   <%= form.hidden_field :beer_club_id, value: @beer_club.id %>
>   <%= form.hidden_field :user_id, value: current_user.id %>
>   <%= form.submit "End the membership" %>
> <% end %>
> ```
>
> Tehtävän toteuttamiseen on monta keinoa, yksi keino on saada selville käyttäjän <code>membership</code> olion id, jonka avulla voi käsin asettaa oikean id:n polkuun.
>
> Lomaketta käytettäessä on siis kontrollerissa asetettava muuttujan <code>@membership</code> arvoksi käyttäjän seuraan liittävä olio. Jos toteutat tehtävän käyttämällä <code>membership</code> olion id:tä, on se myös päästettävä kontrollerissa läpi <code>membership_params</code> metodissa.

Jos käyttäjä on seuran jäsen, näytetään seuran sivulla eroamisen mahdollistava painike:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5a.png)

Erottaessa seurasta tehdään uudelleenohjaus käyttäjän sivulle ja näytetään asianmukainen ilmoitus:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5b.png)

## Migraatioista

Olemme käyttäneet Railsin migraatioita jo ensimmäisestä viikosta alkaen. On aika syventyä aihepiiriin hieman tarkemmin.

> ## Tehtävä 10
>
> Lue ajatuksella http://guides.rubyonrails.org/migrations.html

## Oluttyyli

> ## Tehtävät 11-13 (vastaa kolmea tehtävää)
>
> Laajenna sovellustasi siten, että oluttyyli ei ole enää merkkijono, vaan tyylit on talletettu tietokantaan. Jokaiseen oluttyyliin liittyy myös tekstuaalinen kuvaus. Tyylin kuvauksen tyypiksi kannattaa määritellä <code>text</code>, tyypin <code>string</code> avulla määritellyn sarakkeen oletuskoko on nimittäin vain 255 merkkiä.
>
> Muutoksen jälkeen oluen ja tyylin suhteen tulee olla seuraava

![kuva](http://yuml.me/c5a711bb.png)

> Huomaa, oluella nyt oleva attribuutti <code>style</code> tulee poistaa, jotta ei synnyt ristiriitaa assosiaation ansiosta generoitavan aksessorin ja vanhan kentän välille.
>
> Saattaa olla hieman haasteellista suorittaa muutos siten, että oluet linkitetään automaattisesti oikeisiin tyylitietokannan tauluihin.
> Tämäkin onnistuu, jos teet muutoksen useassa askeleessa, esim:
>
> - luo tietokantataulu tyyleille
> - tee tauluun rivi jokaista _beers_-taulusta löytyvää erinimistä tyyliä kohti (tämä onnistuu konsolista käsin)
> - uudelleennimeä _beers_-taulun sarake style esim. _old_style_:ksi (tämä siis migraation avulla)
> - tee _beers_-taulun viiteavain tyylejä varten (tämäkin migraation avulla, voit tehdä tämän ja edellisen askeleen samassa migraatiossa)
> - liitä konsolista käsin oluet _style_-olioihin käyttäen hyväksi oluilla vielä olevaa old_style-saraketta
>   - tyylikkäämpää on tehdä myös tämä askel migraatiossa
> - tuhoa oluiden taulusta migraation avulla _old_style_
>
> **Huomaa, että Fly.io/Heroku-instanssin ajantasaistaminen kannattaa tehdä samalla!**
>
> Vihje: voit harjoitella datamigraation tekemistä siten, että kopioit ennen migraation aloittamista tietokannan eli tiedoston _db/development.sqlite3_ ja jos migraatiossa menee jokin pieleen, voit palauttaa tilanteen ennalleen kopion avulla. Myös debuggeri (binding.break) saattaa osoittautua hyödylliseksi migraation kehittelemisessä.
>
> Voit myös suorittaa siirtymisen uusiin tietokannassa oleviin tyyleihin suoraviivaisemmin eli poistamalla oluilta _style_-sarakkeen ja asettamalla oluiden tyylit esim. konsolista.
>
> Muutoksen jälkeen uutta olutta luotaessa oluen tyyli valitaan panimoiden tapaan valmiilta listalta. Lisää myös tyylien sivulle vievä linkki navigaatiopalkkiin.
>
> Tyylien sivulle kannattaa lisätä lista kaikista tyylin oluista.
>
> **HUOM1** Jos lisäät luokalle _Beer_ määreen <code>belongs_to :style</code> et enää pääse käsiksi \_style*-nimiseen merkkijonomuotoiseen attribuuttiin pistenotaatiolla _beer.style_, vaan joudut käyttämään muotoa _beer['style']_
>
> **HUOM2** varmista, että _uusien oluiden luominen toimii_ vielä laajennuksen jälkeen! Joudut muuttamaan muutamaakin kohtaa, näistä vaikein huomata lienee olutkontrollerin apumetodi <code>beer_params</code>.

Tehtävän jälkeen oluttyylin sivu voi näyttää esim. seuraavalta

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-6.png)

Hyvä lista oluttyyleistä kuvauksineen löytyy osoitteesta http://beeradvocate.com/beer/style/

> ## Tehtävä 14
>
> Tyylien tallettaminen tietokantaan hajottaa suuren osan testeistä. Ajantasaista testit. Huomaa, että myös FactoryBotin tehtaisiin on tehtävä muutoksia.
>
> Vaikka hajonneita testejä on suuri määrä, älä mene paniikkiin. Selvitä ongelmat testi testiltä, yksittäinen ongelma kertautuu monteen paikkaan ja testien ajantasaistaminen ei ole loppujenlopuksi kovin vaikeaa.
>
> _HUOM_ voit poistaa railsin automaattisesti generoimat testit, esim. testin _spec/views/styles/index.html.erb_spec.rb_

## Olutsää

> ## Tehtävä 15
>
> Lisää olutpaikat näyttävälle sivulle paikan tämänhetkinen säätiedoitus. Säätiedoituksen tarjoavia palveluita on kymmeniä. Itse käytin [https://weatherstack.com/](https://weatherstack.com/):ta. Muista jälleen käsitellä koodissa apiavainta järkevästi!

Tehtävän jälkeen olutpaikkojen sivu voi näyttää esim. seuraavalta

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-7.png)

> ## Tehtävä 16
>
> Tehtävä 15 hajottaa osan testeitä. Korjaa testit. Voit merkata tämän tehtävän tehdyksi ainoastaan jos teet edellisen tehtävän.

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi GitHubiin. Deployaa myös uusin versio Fly.io:n tai Herokuun. Muista myös testata Rubocopilla, että koodisi noudattaa edelleen määriteltyjä tyylisääntöjä.

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/stats/courses/rails2023/
    
[Viikko 6](https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/web/viikko6.md)
