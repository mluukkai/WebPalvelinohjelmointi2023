## Kisko afterparty pe 16.12. klo 16-18

Suomen johtava Rails-talo [Kisko](https://www.kiskolabs.com/) järjestää kurssilaisille illanvieton pe 16.12. klo 16-18

<img src="https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/kisko.png" width="600">

Jos haluat mukaan, kysy ilmoittautumislinkkiä matti.luukkainen@helsinki.fi tai Discordissa kurssikanavalla tai @mluukkai

Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 3 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit täydentää ratkaisusi tehtävien palautusjärjestelmän kautta näkyvän esimerkivastauksen avulla.

## Muutama huomio

### Rubocop

Muista testata rubocopilla, että kaikki tulevaisuudessa tekemäsi koodisi noudattaa määriteltyjä tyylisääntöjä.

Jos käytät Visual studio codea, kannattaa asentaa [rubocop-laajennus](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko3.md#rubocop)

### Ongelmia lomakkeiden kanssa

Viikolla 2 muutimme oluiden luomislomaketta siten, että uuden oluen tyyli ja panimo valitaan pudotusvalikoista. Lomake siis muutettiin käyttämään tekstikentän sijaan _select_:iä:

```ruby
<div>
  <%= form.label :style, style: "display: block" %>
  <%= form.select :style, options_for_select(@styles) %>
</div>

<div>
  <%= form.label :brewery_id, style: "display: block" %>
  <%= form.select :brewery_id, options_from_collection_for_select(@breweries, :id, :name) %>
</div>
```

eli pudotusvalikkojen valintavaihtoehdot välitetään lomakkeelle muuttujissa <code>@styles</code> ja <code>@breweries</code>, joille kontrollerin metodi <code>new</code> asettaa arvot:

```ruby
def new
  @beer = Beer.new
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

Näiden muutosten jälkeen oluen tietojen editointi ei yllättäen enää toimi. Seurauksena on virheilmoitus <code>undefined method `map' for nil:NilClass</code>, johon olet kenties jo kurssin aikana törmännyt:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w4-0.png)

Syynä tälle on se, että uuden oluen luominen ja oluen tietojen editointi käyttävät molemmat samaa lomakkeen generoivaa näkymätemplatea (app/views/beers/\_form.html.erb) ja muutosten jälkeen näkymän toiminta edellyttää, että muuttuja <code>@breweries</code> sisältää panimoiden listan ja muuttuja <code>@styles</code> sisältää oluiden tyylit. Oluen tietojen muutossivulle mennään kontrollerimetodin <code>edit</code> suorituksen jälkeen, ja joudummekin muuttamaan kontrolleria seuraavasti korjataksemme virheen:

```ruby
def edit
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

Täsmälleen samaan ongelmaan törmätään jos yritetään luoda olut, joka ei ole validi. Tällöin nimittäin kontrollerin metodi <code>create</code> yrittää renderöidä uudelleen lomakkeen generoivan näkymätemplaten. Metodissa on siis ennen renderöintiä asetettava arvo templaten tarvitsemille muuttujille <code>@styles</code> ja <code>@breweries</code>:

```ruby
def create
  @beer = Beer.new(beer_params)

  respond_to do |format|
    if @beer.save
      format.html { redirect_to beers_path, notice: 'Beer was successfully created.' }
      format.json { render action: 'show', status: :created, location: @beer }
    else
      @breweries = Brewery.all
      @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter"]

      format.html { render action: 'new' }
      format.json { render json: @beer.errors, status: :unprocessable_entity }
    end
  end
end
```

Onkin hyvin tyypillistä, että kontrollerimetodit <code>new</code>, <code>create</code> ja <code>edit</code> sisältävät paljon samaa, näkymätemplaten tarvitsemien muuttujien alustukseen käytettyä koodia. Onkin järkevää ekstraktoida yhteinen koodi omaan metodiin:

```ruby
def set_breweries_and_styles_for_template
  @breweries = Brewery.all
  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
end
```

Metodia voidaan kutsua kontrollerin metodeista <code>new</code>, <code>create</code> ja <code>edit</code>:

```ruby
def new
  @beer = Beer.new
  set_breweries_and_styles_for_template
end
```

tai ehkä vielä tyylikkäämpää on hoitaa asia <code>before_action</code> määreellä:

```ruby
class BeersController < ApplicationController
  # ...
  before_action :set_breweries_and_styles_for_template, only: [:new, :edit, :create]

  # ...
```

tällöin muuttujien <code>@styles</code> ja <code>@breweries</code> arvot asettava metodi siis suoritetaan automaattisesti aina ennen metodien
<code>new</code>, <code>create</code> ja <code>edit</code> suoritusta. Metodissa <code>create</code> muuttujien arvot asetetaan ehkä turhaan sillä niitä tarvitaan ainoastaan validoinnin epäonnistuessa. Kenties olisikin parempi käyttää eksplisiittistä kutsua createssa.

### Ongelmia Herokun tai Fly.io:n kanssa

Moni kurssin osallistujista on törmännyt siihen, että paikallisesti loistavasti toimiva sovellus on aiheuttanut Herokussa pahaenteisen virheilmoituksen _We're sorry, but something went wrong_.

Heti ensimmäisenä kannattaa tarkistaa, että paikalliselta koneelta kaikki koodi on lisätty versionhallintaan, eli <code>git status</code>

Epätriviaalit ongelmat selviävät aina Herokun/Fly.io:n lokin avulla. Herokussa lokia päästään tutkimaan komentoriviltä komennolla <code>heroku logs</code> ja Fly.io:ta käytettäessä komennolla <code>fly logs</code>

Seuraavassa Herokulle tyypillisen ongelmatilanteen loki:

```ruby
mbp-18:ratebeer-public mluukkai$ heroku logs
2022-08-28T18:53:05.867973+00:00 app[web.1]:                   ON a.attrelid = d.adrelid AND a.attnum = d.adnum
2022-08-28T18:53:05.867973+00:00 app[web.1]:
2022-08-28T18:53:05.867973+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.867973+00:00 app[web.1]:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.874380+00:00 app[web.1]: Completed 500 Internal Server Error in 10ms
2022-08-28T18:53:05.878587+00:00 app[web.1]: :               SELECT a.attname, format_type(a.atttypid, a.atttypmod),
2022-08-28T18:53:05.878587+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.878587+00:00 app[web.1]:
2022-08-28T18:53:05.868310+00:00 app[web.1]:
2022-08-28T18:53:05.867973+00:00 app[web.1]:                      pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
2022-08-28T18:53:05.867973+00:00 app[web.1]:                  AND a.attnum > 0 AND NOT a.attisdropped
2022-08-28T18:53:05.868310+00:00 app[web.1]:                ORDER BY a.attnum
2022-08-28T18:53:05.878587+00:00 app[web.1]:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.867973+00:00 app[web.1]:                 FROM pg_attribute a LEFT JOIN pg_attrdef d
2022-08-28T18:53:05.882824+00:00 app[web.1]: LINE 5:                WHERE a.attrelid = '"users"'::regclass
2022-08-28T18:53:05.882824+00:00 app[web.1]:                                           ^
2022-08-28T18:53:05.878587+00:00 app[web.1]:                      pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod
2022-08-28T18:53:05.878587+00:00 app[web.1]:                   ON a.attrelid = d.adrelid AND a.attnum = d.adnum
2022-08-28T18:53:05.874380+00:00 app[web.1]: Completed 500 Internal Server Error in 10ms
2022-08-28T18:53:05.878587+00:00 app[web.1]: ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "users" does not exist
```

lokia tarkasti lukemalla selviää että syynä on seuraava

```ruby
ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "users" does not exist
```

eli migraatiot ovat jääneet suorittamatta. Korjaus on helppo:

    heroku run rails db:migrate

Fly.io suorittaa migraatiot automaattisesti tuotantoonviennin yhteydessä, joten todennäköisesti tämä virhe ei siellä ole vaivana.

Seuraavassa loki eräästä toisesta, myös Fly.io:n kanssa hyvin tyypillisestä virhetilanteesta:

```ruby
2022-08-28T19:32:31.609344+00:00 app[web.1]:     6:   <% @ratings.each do |rating| %>
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]:   app/views/ratings/index.html.erb:6:in `_app_views_ratings_index_html_erb___254869282653960432_70194062879340'
2022-08-28T19:32:31.609530+00:00 app[web.1]:
2022-08-28T19:32:31.609530+00:00 app[web.1]: ActionView::Template::Error (undefined method `username' for nil:NilClass):
2022-08-28T19:32:31.609344+00:00 app[web.1]:   app/views/ratings/index.html.erb:7:in `block in _app_views_ratings_index_html_erb___254869282653960432_70194062879340'
2022-08-28T19:32:31.609530+00:00 app[web.1]:     7:       <li> <%= rating %> <%= link_to rating.user.username, rating.user %> </li>
2022-08-28T19:32:31.609530+00:00 app[web.1]:     4:
2022-08-28T19:32:31.609530+00:00 app[web.1]:     6:   <% @ratings.each do |rating| %>
2022-08-28T19:32:31.609530+00:00 app[web.1]:     5: <ul>
2022-08-28T19:32:31.609715+00:00 app[web.1]:    10:
```

Tarkka silmä huomaa lokin seasta että ongelma on _ActionView::Template::Error (undefined method `username' for nil:NilClass)_ ja virhe syntyi tiedoston _app/views/ratings/index.html.erb_ riviä 7 suoritettaessa. Virheen aiheuttanut rivi on

```ruby
<li> <%= rating %> <%= link_to rating.user.username, rating.user %> </li>
```

vaikuttaa siis siltä, että tietokannassa on <code>rating</code>-olio, johon liittyvä <code>user</code> on <code>nil</code>. Kyseessä on siis jo [viikolta 2 tuttu](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#ongelmia-herokussa) ongelma.

Ongelman perimmäinen syy on joko se, että jonkin ratingin <code>user_id</code>-kentän arvo on <code>nil</code>, tai että jonkin rating-olion <code>user_id</code>:n arvona on virheellinen id. Tilanteesta selvitään esim. tuhoamalla 'huonot' rating-oliot konsolista käsin. Herokussa konsoli avautuu komennolla <code>heroku run console</code>. Fly.io:n konsoliin pääset antamalla ensin komennon <code>fly ssh console</code> ja sen jälkeen komennon <code>/app/bin/rails c</code>

```ruby
> bad_ratings = Rating.all.select{ |r| r.user.nil? or r.beer.nil? }
=> [#<Rating id: 1, score: 10, beer_id: 2, created_at: "2022-08-28 19:04:43", updated_at: "2022-08-28 19:04:43", user_id: nil>]
> bad_ratings.each{ |bad| bad.destroy }
=> [#<Rating id: 1, score: 10, beer_id: 2, created_at: "2022-08-28 19:04:43", updated_at: "2022-08-28 19:04:43", user_id: nil>]
> Rating.all.select{ |r| r.user.nil? or r.beer.nil? }
=> []
>
```

Ylläoleva hakee varalta kannasta myös ratingit, joihin ei liity mitään olemassaolevaa olutta.

Eli jos ja kun joudut Fly.io:n tai Herokun kanssa ongelmiin, selvitä analyyttisesti mistä on kyse, loki ja konsoli auttavat aina hädässä!

### Migraation peruminen

Silloin tällöin (esim. jos luodaan vahingossa huono scaffold, ks. seuraava kohta) syntyy tilanteita, joissa edelliseksi suoritetettu migraatio on syytä perua. Tämä onnistuu komennolla

    rails db:rollback

### Huono scaffold

Jos haluat poistaa scaffold-generaattorin luomat tiedostot, onnistuu tämä komennolla

    rails destroy scaffold resurssin_nimi

missä _resurssin_nimi_ on scaffoldilla luomasi resurssin nimi. **HUOM:** jos suoritit jo huonoon scaffoldiin liittyvän migraation, tee ehdottomasti ennen scaffoldin tuhoamista <code>rails db:rollback</code>

Muuten kaikki allaoleva koodi ei toimi ilman muutoksia.

## Testaaminen

Toistaiseksi olemme tehneet koodia, jonka toimintaa olemme testanneet ainoastaan selaimesta. Tämä on suuri virhe. Jokaisen eliniältään laajemmaksi tarkoitetun ohjelman on syytä sisältää riittävän kattavat automaattiset testit, muuten ajan mittaan käy niin että ohjelman laajentaminen tulee liian riskialttiiksi.

Käytämme testaukseen Rspec:iä ks. http://rspec.info/, https://github.com/rspec/rspec-rails ja http://betterspecs.org/

Otetaan käyttöön rspec-rails gem lisäämällä Gemfileen seuraava:

```ruby
group :test do
  # ...
  gem 'rspec-rails', '~> 6.0.0.rc1'
end
```

> Materiaaleja kirjottaessa ainoa tarjolla oleva versio rspec-rails 6:sta on .rc1 päätteinen. Rspec-projektin repositorio ohjeistaa käyttämään 6.0.0 versiota, joka saattaa toimia jälleen kurssin aikana.

Uusi gem otetaan käyttöön tutulla tavalla, eli antamalla komentoriviltä komento <code>bundle install</code>

rspec saadaan initialisoitua sovelluksen käyttöön antamalla komentoriviltä komento

    rails generate rspec:install

Initialisointi luo sovellukselle hakemiston /spec jonka alihakemistoihin testit eli "spekit" sijoitetaan.

Railsin oletusarvoinen, mutta nykyään vähemmän käytetty testausframework sijoittaa testit hakemistoon /test. Ko. hakemisto on tarpeeton rspecin käyttöönoton jälkeen ja se voidaan poistaa.

Testejä (oikeastaan rspecin yhteydessä ei pitäisi puhua testeistä vaan speceistä tai spesifikaatioista, käytämme kuitenkin jatkossa sanaa testi) voidaan kirjoittaa usealla tasolla: yksikkötestejä modeleille tai kontrollereille, näkymätestejä, integraatiotestejä kontrollereille. Näiden lisäksi sovellusta voidaan testata käyttäen simuloitua selainta capybara-gemin https://github.com/jnicklas/capybara avulla.

Kirjoitamme jatkossa lähinnä yksikkötestejä modeleille sekä capybaran avulla simuloituja selaintason testejä.

## Yksikkötestit

Tehdään kokeeksi muutama yksikkötesti luokalle <code>User</code>. Voimme luoda testipohjan käsin tai komentoriviltä rspec-generaattorilla

    rails generate rspec:model user

Hakemistoon /spec/models tulee tiedosto user_spec.rb

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

Kokeillaan ajaa testit komentoriviltä komennolla <code>rspec spec</code> (huom: saattaa olla, että joudut tässä vaiheessa käynnistämään terminaalin uudelleen!).

Testien suoritus etenee seuraavasti:

```ruby
$ rspec spec
*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) User add some examples to (or delete) /Users/mluukkai/opetus/ratebeer/spec/models/user_spec.rb
     # Not yet implemented
     # ./spec/models/user_spec.rb:4


Finished in 0.00932 seconds (files took 3.31 seconds to load)
1 example, 0 failures, 1 pending
```

Komento <code>rspec spec</code> määrittelee, että suoritetaan kaikki testit, jotka löytyvät hakemiston spec alihakemistoista. Jos testejä on paljon, on myös mahdollista ajaa suppeampi joukko testejä:

    rspec spec/models                # suoritetaan hakemiston model sisältävät testit
    rspec spec/models/user_spec.rb   # suoritetaan user_spec.rb:n määrittelemät testi

Testien suorituksen voi myös automatisoida aina kun testi tai sitä koskeva koodi muuttuu. [guard](https://github.com/guard/guard) on tähän käytetty kirjasto ja siihen löytyy monia laajennoksia.

Aloitetaan testien tekeminen. Kirjoitetaan (tiedostoon _user_spec.rb_) aluksi testi joka testaa, että konstruktori asettaa käyttäjätunnuksen oikein:

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it "has the username set correctly" do
    user = User.new username: "Pekka"

    expect(user.username).to eq("Pekka")
  end
end
```

Testi kirjoitetaan <code>it</code>-nimiselle metodille annettavan koodilohkon sisälle. Metodin ensimmäisenä parametrina on merkkijono, joka toimii testin nimenä. Muuten testi kirjoitetaan samaan tapan kuin esim. jUnitilla, eli ensin luodaan testattava data, sitten suoritetaan testattava toimenpide ja lopuksi varmistetaan että vastaus on odotettu.

Suoritetaan testi ja havaitaan sen menevän läpi:

```ruby
$ rspec spec

Finished in 0.00553 seconds (files took 2.11 seconds to load)
1 example, 0 failures
```

Toisin kuin jUnit-testauskehyksessä, Rspecin yhteydessä ei käytetä assert-komentoja testin odotetun tuloksen määrittelemiseen. Käytössä on hieman erikoisemman näköinen syntaksi, kuten testin viimeisellä rivillä oleva:

    expect(user.username).to eq("Pekka")

Äskeisessä testissä käytettiin komentoa <code>new</code>, joten olioa ei talletettu tietokantaan. Kokeillaan nyt olion tallettamista. Olemme määritelleet, että User-olioilla tulee olla salasana, jonka pituus on vähintään 4 ja että salasana sisältää sekä numeron että ison kirjaimen. Eli jos salasanaa ei aseteta, ei oliota tulisi tallettaa tietokantaan. Voimme kysyä oliolta metodilla _valid?_ onko sille suoritettu validointi onnistuneesti eli käytännössä, onko olio talletettu tietokantaan.

Testataan että näin tapahtuu:

```ruby
RSpec.describe User, type: :model do

  # aiemmin määritellyn testin koodi ...

  it "is not saved without a password" do
    user = User.create username: "Pekka"

    expect(user.valid?).to be(false)
    expect(User.count).to eq(0)
  end
end
```

Testi menee läpi.

Testin ensimmäinen tarkistus

```ruby
expect(user.valid?).to be(false)
```

on kyllä ymmärrettävä, mutta kiitos rspec-magian, voimme ilmaista sen myös seuraavasti

```ruby
expect(user).not_to be_valid
```

Tämän muodon toiminta perustuu sille, että oliolla <code>user</code> on totuusarvoinen metodi <code>valid?</code>.

Huomaamme, että käytämme testeissä kahta samuuden tarkastustapaa <code>be(false)</code> ja <code>eq(0)</code>, mikä näillä on erona? Matcherin eli 'tarkastimen' <code>be</code> avulla voidaan varmistaa, että kyse on kahdesta samasta oliosta. Totuusarvojen vertailussa <code>be</code> onkin toimiva tarkistin. Esim. merkkijonojen vertailuun se ei toimi, kokeile muuttaa ensimmäisen testin vertailu muotoon:

```ruby
expect(user.username).to be("Pekka")
```

nyt testi ei mene läpi:

```ruby
1) User has the username set correctly
    Failure/Error: expect(user.username).to be("Pekka")

      expected #<String:70322613325340> => "Pekka"
          got #<String:70322613325560> => "Pekka"

      Compared using equal?, which compares object identity,
      but expected and actual are not the same object. Use
      `expect(actual).to eq(expected)` if you don't care about
      object identity in this example.
```

Kun riittää että vertailtavat oliot ovat sisällöltään samat, tuleekin käyttää tarkistinta <code>eq</code>, käytännössä useimmissa tilanteissa näin on kaikkien muiden paitsi totuusarvojen kanssa. Tosin totuusarvojenkin <code>eq</code> toimisi eli voisimme kirjoittaa myös

```ruby
expect(user.valid?).to eq(false)
```

Tehdään sitten testi kunnollisella salasanalla:

```ruby
it "is saved with a proper password" do
  user = User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1"

  expect(user.valid?).to be(true)
  expect(User.count).to eq(1)
end
```

Testin ensimmäinen "ekspektaatio" varmistaa, että luodun olion validointi onnistuu, eli että metodi <code>valid?</code> palauttaa true. Toinen ekspektaatio taas varmistaa, että tietokannassa olevien olioiden määrä on yksi.

Olisimme jälleen voineet käyttää käyttäjän validiteetin tarkastamiseen hieman luettavampaa muotoa

```ruby
expect(user).to be_valid
```

On huomattavaa, että rspec **nollaa tietokannan aina ennen jokaisen testin ajamista**, eli jos teemme uuden testin, jossa tarvitaan Pekkaa, on se luotava uudelleen:

```ruby
it "with a proper password and two ratings, has the correct average rating" do
  user = User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1"
  brewery = Brewery.new name: "test", year: 2000
  beer = Beer.new name: "testbeer", style: "teststyle", brewery: brewery
  rating = Rating.new score: 10, beer: beer
  rating2 = Rating.new score: 20, beer: beer

  user.ratings << rating
  user.ratings << rating2

  expect(user.ratings.count).to eq(2)
  expect(user.average_rating).to eq(15.0)
end
```

Kuten arvata saattaa, ei testin alustuksen (eli testattavan olion luomisen) toistaminen ole järkevää, ja yhteinen osa voidaan helposti eristää. Tämä tapahtuu esim. tekemällä samanlaisen alustuksen omaavalle osalle testeistä oma <code>describe</code>-lohko, jonka alkuun määritellään ennen jokaista testiä suoritettava <code>let</code>-komento, joka alustaa user-muuttujan uudelleen jokaista testiä ennen:

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it "has the username set correctly" do
    user = User.new username: "Pekka"

    expect(user.username).to eq("Pekka")
  end

  it "is not saved without a password" do
    user = User.create username: "Pekka"

    expect(user).not_to be_valid
    expect(User.count).to eq(0)
  end

  describe "with a proper password" do
    let(:user){ User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1" }
    let(:test_brewery) { Brewery.new name: "test", year: 2000 }
    let(:test_beer) { Beer.create name: "testbeer", style: "teststyle", brewery: test_brewery }

    it "is saved" do
      expect(user).to be_valid
      expect(User.count).to eq(1)
    end

    it "and with two ratings, has the correct average rating" do
      rating = Rating.new score: 10, beer: test_beer
      rating2 = Rating.new score: 20, beer: test_beer

      user.ratings << rating
      user.ratings << rating2

      expect(user.ratings.count).to eq(2)
      expect(user.average_rating).to eq(15.0)
    end
  end
end
```

Muuttujien alustus tapahtuu hieman erikoisen <code>let</code>-metodin avulla, esim.

```ruby
let(:user){ User.create username: "Pekka", password: "Secret1", password_confirmation: "Secret1" }
```

saa aikaan sen, että määrittelyn jälkeen muuttuja _user_ viittaa <code>let</code>-metodin koodilohkossa luotuun User-olioon.

Siitä huolimatta, että muuttujan alustus on nyt vain yhdessä paikassa koodia, suoritetaan alustus uudelleen ennen jokaista metodia. Huom: metodi <code>let</code> suorittaa olion alustuksen vasta kun olioa tarvitaan oikeasti, tästä saattaa joissain tilanteissa olla yllättäviä seurauksia!

Erityisesti vanhemmissa Rspec-testeissä näkee tyyliä, jossa testeille yhteinen alustus tapahtuu <code>before :each</code> -lohkon avulla. Tällöin testien yhteiset muuttujat on määriteltävä instanssimuuttujiksi, eli tyyliin <code>@user</code>.

Testien ja describe-lohkojen nimien valinta ei ole ollut sattumanvaraista. Määrittelemällä testauksen tulos formaattiin "documentation" (parametri -fd), saadaan testin tulos ruudulle mukavassa muodossa:

```ruby
$ rspec -fd spec

User
  has the username set correctly
  is not saved without a password
  with a proper password
    is saved
    and with two ratings, has the correct average rating

Finished in 0.12949 seconds (files took 1.95 seconds to load)
4 examples, 0 failures
```

Pyrkimyksenä onkin kirjoittaa testien nimet siten, että testit suorittamalla saadaan ohjelmasta mahdollisimman ihmisluettava "spesifikaatio".

Voit myös lisätä rivin `-fd` tiedostoon `.rspec`, jolloin projektin rspec-testit näytetään aina documentation formaatissa.

> ## Tehtävä 1
>
> Lisää luokalle User testit, jotka varmistavat, että liian lyhyen tai pelkästään pienistä kirjaimista muodostetun salasanan omaavan käyttäjän luominen create-metodilla ei tallenna oliota tietokantaan, ja että luodun olion validointi ei ole onnistunut

Muista aina nimetä testisi niin että ajamalla Rspec dokumentointiformaatissa, saat kieliopillisesti järkevältä kuulostavan "speksin".

> ## Tehtävä 2
>
> Luo Rspecin generaattorilla (tai käsin) testipohja luokalle <code>Beer</code> ja tee testit, jotka varmistavat, että
>
> - oluen luonti onnistuu ja olut tallettuu kantaan jos oluella on nimi, tyyli ja panimo asetettuna
> - oluen luonti ei onnistu (eli creatella ei synny validia oliota), jos sille ei anneta nimeä
> - oluen luonti ei onnistu, jos sille ei määritellä tyyliä
>
> Jos jälkimmäinen testi ei mene läpi, laajenna koodiasi siten, että se läpäisee testin. Vinkki: oluelle täytyy asettaa panimon id, mutta entä jos panimoa ei ole olemassa?
>
> Jos teet testitiedoston käsin, muista sijoittaa se hakemistoon spec/models

## Testiympäristöt eli fixturet

Edellä käyttämämme tapa, jossa testien tarvitsemia oliorakenteita luodaan testeissä käsin, ei ole välttämättä kaikissa tapauksissa järkevä. Parempi tapa voi olla koota testiympäristön rakentaminen, eli testien alustamiseen tarvittava data omaan paikkaansa, "testifixtureen". Käytämme testien alustamiseen Railsin oletusarvoisen fixture-mekanismin sijaan FactoryBot-nimistä gemiä, kts.
https://github.com/thoughtbot/factory_bot ja https://github.com/thoughtbot/factory_bot_rails

Lisätään Gemfileen seuraava

```ruby
group :test do
  # ...
  gem 'factory_bot_rails'
end
```

ja päivitetään gemit komennolla <code>bundle install</code>

Tehdään fixtureja varten tiedosto spec/factories.rb ja kirjoitetaan sinne seuraava:

```ruby
FactoryBot.define do
  factory :user do
    username { "Pekka" }
    password { "Foobar1" }
    password_confirmation { "Foobar1" }
  end
end
```

Tiedostossa määritellään "oliotehdas" luokan <code>User</code> olioiden luomiseen. Tehtaaseen ei tarvinnut määritellä erikseen tehtaan luomien olioiden luokkaa, sillä FactoryBot päättelee sen suoraan käytettävän fixtuurin nimestä <code>user</code>.

Määriteltyjä tehtaita voidaan pyytää luomaan olioita seuraavasti:

```ruby
user = FactoryBot.create(:user)
```

FactoryBotin tehdasmetodin _create_ kutsuminen luo olion automaattisesti testausympäristön tietokantaan.

Muutetaan nyt testimme käyttämään _user_-olioiden luomiseen FactoryBotiä:

```ruby
describe "with a proper password" do
  let(:user) { FactoryBot.create(:user) } # tämä rivi muuttui
  let(:test_brewery) { Brewery.new name: "test", year: 2000 }
  let(:test_beer) { Beer.create name: "testbeer", style: "teststyle", brewery: test_brewery }

  it "is saved" do
    expect(user).to be_valid
    expect(User.count).to eq(1)
  end

  it "and with two ratings, has the correct average rating" do
    rating = Rating.new score: 10, beer: test_beer
    rating2 = Rating.new score: 20, beer: test_beer

    user.ratings << rating
    user.ratings << rating2

    expect(user.ratings.count).to eq(2)
    expect(user.average_rating).to eq(15.0)
  end
end
```

Muutos aiempaan on vielä melko pieni. Laajennetaan fixtureita vielä siten, että voimme luoda niiden avulla myös testien käyttämät _rating_-oliot. Muutetaan tiedostoa spec/factories.rb seuraavasti

```ruby
FactoryBot.define do
  factory :user do
    username { "Pekka" }
    password { "Foobar1" }
    password_confirmation { "Foobar1" }
  end

  factory :brewery do
    name { "anonymous" }
    year { 1900 }
  end

  factory :beer do
    name { "anonymous" }
    style { "Lager" }
    brewery # olueeseen liittyvä panimo luodaan brewery-tehtaalla
  end

  factory :rating do
    score { 10 }
    beer # reittaukseen liittyvä olut luodaan beer-tehtaalla
    user # reittaukseen liittyvä user luodaan user-tehtaalla
  end
end
```

Reittausten luovan oliotehtaan _:rating_ lisäksi tiedostossa määritellään panimoita ja oluita luovat fixturet.

Tehdas <code>FactoryBot.create(:brewery)</code> luo panimon, jonka nimi on 'anonymous' ja perustamisvuosi 1900.

Tehdas <code>FactoryBot.create(:beer)</code> luo oluen, jonka tyyli on 'Lager' ja nimi 'anonymous' ja oluelle luodaan panimo, johon olut liittyy. Vastaavasti tehdas <code>FactoryBot.create(:rating)</code> luo reittauksen, johon liittyvät tehtaan luomat olut ja käyttäjä. Lisäksi reittauksen arvoksi eli kenttään _score_ asetetaan 10.

Testi voidaan muuttaa seuraavaan muotoon

```ruby
describe "with a proper password" do
  let(:user) { FactoryBot.create(:user) }

  it "is saved" do
    expect(user).to be_valid
    expect(User.count).to eq(1)
  end

  it "and with two ratings, has the correct average rating" do
    FactoryBot.create(:rating, score: 10, user: user)
    FactoryBot.create(:rating, score: 20, user: user)

    expect(user.ratings.count).to eq(2)
    expect(user.average_rating).to eq(15.0)
  end
end
```

Testi siis luo kaksi reittausta, toisen pistemäärä on 10 ja toisen 20, jotka liitetään _let_-komennossa tehtaan avulla luodulle käyttäjälle:

```ruby
FactoryBot.create(:rating, score: 10, user: user)
FactoryBot.create(:rating, score: 20, user: user)
```

Saman tehtaan avulla on siis mahdollista luoda useita olioita. Esimerkiksi seuraava

```ruby
FactoryBot.create(:brewery)
FactoryBot.create(:brewery)
FactoryBot.create(:brewery)
```

loisi kolme _eri_ panimo-olioa, jotka ovat kaikki samansisältöistä.

Tehtaalla luotavien olioiden sisältöä voidaan muokata parametrien avulla, esim.

```ruby
FactoryBot.create(:brewery)
FactoryBot.create(:brewery, name: 'crapbrew')
FactoryBot.create(:brewery, name: 'homebrew', year: 2011)
```

loisi kolme panimoa, joista yksi saisi oletusarvoisen nimen _anonymous_ ja perustamisvuoden _1900_. Toinen panimo saisi oletusarvoisen perustamisvuoden mutta nimen _crapbrew_, kolmannen panimon nimi sekä perustusvuosi määrittyisi annettujen parametrien mukaan.

Myös tehtaalta <code>user</code> voitaisiin pyytää kahta eri olioa.

```ruby
FactoryBot.create(:user)
FactoryBot.create(:user)
```

Tämä kuitenkin aiheuttaisi poikkeuksen, sillä <code>User</code>-olioiden validointi edellyttää, että username on yksikäsitteinen ja tehdas luo oletusarvoisesti aina "Pekka"-nimisen käyttäjän.

Seuraava kuitenkin olisi ok, eli luotaisiin kaksi erinimistä käyttäjää, oletusarvoisen nimen saava _Pekka_ ja _Vilma_

```ruby
FactoryBot.create(:user)
FactoryBot.create(:user, username: 'Vilma')
```

Lisää ohjeita FactoryBotin käyttöön osoitteessa https://www.rubydoc.info/gems/factory_bot/file/GETTING_STARTED.md

## Käyttäjän lempiolut, -panimo ja -oluttyyli

Toteutetaan seuraavaksi test driven -tyylillä (tai behaviour driven niinkuin rspecin luojat sanoisivat) käyttäjälle metodit, joiden avulla saadaan selville käyttäjän lempiolut, lempipanimo ja lempioluttyyli käyttäjän tekemien reittausten perusteella.

Oikeaoppisessa TDD:ssä ei tehdä yhtään koodia ennen kuin minimaalinen testi sen pakottaa. Tehdäänkin ensin testi, jonka avulla vaaditaan että <code>User</code>-olioilla on metodi <code>favorite_beer</code>:

```ruby
it "has method for determining the favorite_beer" do
  user = FactoryBot.create(:user)
  expect(user).to respond_to(:favorite_beer)
end
```

Testi ei mene läpi, eli lisätään luokalle User metodin runko:

```ruby
class User < ApplicationRecord
  # ...

  def favorite_beer
  end
end
```

Testi menee nyt läpi. Lisätään seuraavaksi testi, joka varmistaa, että ilman reittauksia ei käyttäjllä ole mieliolutta, eli että metodi palauttaa nil:

```ruby
it "without ratings does not have a favorite beer" do
  user = FactoryBot.create(:user)
  expect(user.favorite_beer).to eq(nil)
end
```

Testi menee läpi sillä Rubyssa metodit palauttavat oletusarvoisesti nil.

Refaktoroidaan testiä hieman lisäämällä juuri kirjoitetulle kahdelle testille oma <code>describe</code>-lohko

```ruby
describe "favorite beer" do
  let(:user){ FactoryBot.create(:user) }

  it "has method for determining one" do
    expect(user).to respond_to(:favorite_beer)
  end

  it "without ratings does not have one" do
    expect(user.favorite_beer).to eq(nil)
  end
end
```

Lisätään sitten testi, joka varmistaa että jos reittauksia on vain yksi, osaa metodi palauttaa reitatun oluen.

```ruby
it "is the only rated if only one rating" do
  beer = FactoryBot.create(:beer)
  rating = FactoryBot.create(:rating, score: 20, beer: beer, user: user)

  # jatkuu...
end
```

Alussa siis luodaan olut, sen jälkeen reittaus. Reittauksen <code>create</code>-metodille annetaan parametreiksi pistemäärä sekä olut- ja käyttäjäoliot (joista molemmat on luotu FactoryBotillä), joihin reittaus liitetään.

Luotu reittaus siis liittyy käyttäjään ja on käyttäjän ainoa reittaus. Testi siis lopulta odottaa, että reittaukseen liittyvä olut on käyttäjän lempiolut:

```ruby
it "is the only rated if only one rating" do
  beer = FactoryBot.create(:beer)
  rating = FactoryBot.create(:rating, score: 20, beer: beer, user: user)

  expect(user.favorite_beer).to eq(beer)
end
```

Testi ei mene läpi, sillä metodimme ei vielä tee mitään ja sen paluuarvo on siis aina <code>nil</code>.

Tehdään [TDD:n hengen mukaan](https://stanislaw.github.io/2016/01/25/notes-on-test-driven-development-by-example-by-kent-beck.html) ensin "huijattu ratkaisu", eli ei vielä yritetäkään tehdä lopullista toimivaa versiota:

```ruby
class User < ApplicationRecord
  # ...

  def favorite_beer
    return nil if ratings.empty?   # palautetaan nil jos reittauksia ei ole

    ratings.first.beer             # palataan ensimmaiseen reittaukseen liittyvä olut
  end
end
```

Tehdään vielä testi, joka pakottaa meidät kunnollisen toteutuksen tekemiseen [(ks. triangulation)](https://stanislaw.github.io/2016/01/25/notes-on-test-driven-development-by-example-by-kent-beck.html):

```ruby
it "is the one with highest rating if several rated" do
  beer1 = FactoryBot.create(:beer)
  beer2 = FactoryBot.create(:beer)
  beer3 = FactoryBot.create(:beer)
  rating1 = FactoryBot.create(:rating, score: 20, beer: beer1, user: user)
  rating2 = FactoryBot.create(:rating, score: 25, beer: beer2, user: user)
  rating3 = FactoryBot.create(:rating, score: 9, beer: beer3, user: user)

  expect(user.favorite_beer).to eq(beer2)
end
```

Ensin luodaan kolme olutta ja sen jälkeen oluisiin sekä user-olioon liittyvät reittaukset.

Testi ei luonnollisesti mene vielä läpi, sillä metodin <code>favorite_beer</code> toteutus jätettiin aiemmin puutteelliseksi.

Muuta metodin toteutus nyt seuraavanlaiseksi:

```ruby
def favorite_beer
  return nil if ratings.empty?

  ratings.sort_by{ |r| r.score }.last.beer
end
```

eli ensin järjestetään reittaukset scoren perusteella, otetaan reittauksista viimeinen eli korkeimman scoren omaava ja palautetaan siihen liittyvä olut.

Koska järjestäminen perustui suoraan reittauksen attribuuttiin <code>score</code> oltaisiin metodin viimeinen rivi voitu kirjottaa myös hieman kompaktimmassa muodossa

```ruby
ratings.sort_by(&:score).last.beer
```

Miten metodi itseasiassa toimiikaan? Suoritetaan operaatio konsolista:

```ruby
> u = User.first
> u.ratings.sort_by(&:score).last.beer
  Rating Load (1.4ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
  Beer Load (0.4ms)  SELECT  "beers".* FROM "beers" WHERE "beers"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
```

Seurauksena on 2 SQL-kyselyä, joista ensimmäinen

```ruby
SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
```

hakee kaikki käyttäjään liittyvät reittaukset tietokannasta. Reittausten järjestäminen tapahtuu keskusmuistissa. Jos käyttäjään liittyvien reittausten määrä olisi erittäin suuri, kannattaisi operaatio optimoida siten, että se tehtäisiin suoraan tietokantatasolla.

Tutkimalla dokumentaatiota (http://guides.rubyonrails.org/active_record_querying.html#ordering ja http://guides.rubyonrails.org/active_record_querying.html#limit-and-offset) päädymme seuraavaan ratkaisuun:

```ruby
def favorite_beer
  return nil if ratings.empty?
  ratings.order(score: :desc).limit(1).first.beer
end
```

Voimme konsolista käsin tarkastaa operaation tuloksena olevan SQL-kyselyn (huomaa, että metodi <code>to_sql</code>):

```ruby
> u.ratings.order(score: :desc).limit(1).to_sql
=> "SELECT  \"ratings\".* FROM \"ratings\"  WHERE \"ratings\".\"user_id\" = ?  ORDER BY \"ratings\".\"score\" DESC LIMIT 1"
```

Suorituskyvyn optimoinnissa kannattaa kuitenkin pitää maltti mukana ja sovelluksen kehitysvaiheessa ei vielä välttämättä kannata jäädä optimoimaan jokaista operaatiota. Optimointia tehdessä kannattaa pitää mielessä: [Premature optimization is the root of all evil](http://wiki.c2.com/?PrematureOptimization)

## Testien apumetodit

Testissä tarvittavien oluiden rakentamisen tekevä koodi on hieman ikävä. Voisimme konfiguroida FactoryBotiin oluita, joihin liittyy reittauksia. Päätämme kuitenkin tehdä testitiedostoon reittauksellisen oluen luovan apumetodin <code>create_beer_with_rating</code>:

```ruby
def create_beer_with_rating(object, score)
  beer = FactoryBot.create(:beer)
  FactoryBot.create(:rating, beer: beer, score: score, user: object[:user] )
  beer
end
```

Apumetodia käyttämällä saamme siistityksi testiä

```ruby
it "is the one with highest rating if several rated" do
  create_beer_with_rating({ user: user }, 10 )
  create_beer_with_rating({ user: user }, 7 )
  best = create_beer_with_rating({ user: user }, 25 )

  expect(user.favorite_beer).to eq(best)
end
```

Reittauksen tehneen käyttäjän välittäminen apumetodille tapahtuu nyt hieman erikoisella tavalla, ruby-hashin avaimen arvona. Olisimme voineet määritellä, että käyttäjä välitetään normaalina parametrina, samoin kuin reittauksen pistemäärä:

```ruby
def create_beer_with_rating(user, score)
  beer = FactoryBot.create(:beer)
  FactoryBot.create(:rating, beer: beer, score: score, user: user )
  beer
end
```

Käyttämämme tapa on kuitenkin tässä tapauksessa joustavampi, sillä se mahdollistaa tehtävissä 3 ja 4 tarvittavan metodin <code>create_beer_with_rating</code> laajennuksen siten, että aiemmin tehty testikoodi ei hajoa.

Apumetodeja siis voi (ja kannattaa) määritellä rspec-tiedostoihin. Jos apumetodia tarvitaan ainoastaan yhdessä testitiedostossa, voi sen sijoittaa esim. tiedoston loppuun.

Parannetaan vielä edellistä hiukan määrittelemällä toinenkin metodi <code>create_beers_with_many_ratings</code>, jonka avulla on mahdollista luoda useita reitattuja oluita. Metodi saa reittaukset taulukon tapaan käyttäytyvän vaihtuvamittaisen parametrilistan (ks. http://www.ruby-doc.org/docs/ProgrammingRuby/html/tut_methods.html, kohta "Variable-Length Argument Lists") avulla:

```ruby
def create_beers_with_many_ratings(object, *scores)
  scores.each do |score|
    create_beer_with_rating(object, score)
  end
end
```

Kutsuttaessa metodia esim. seuraavasti

```ruby
create_beers_with_many_ratings( {user: user}, 10, 15, 9)
```

tulee parametrin <code>scores</code> arvoksi kokoelma, jossa ovat luvut 10, 15 ja 9. Metodi luo (metodin <code>create_beer_with_rating</code> avulla) kolme olutta, joihin kuhunkin parametrina annetulla käyttäjällä on reittaus ja reittauksien pistemääriksi tulevat parametrin <code>scores</code> luvut.

Seuraavassa vielä koko mielioluen testaukseen liittyvä koodi:

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do

  # ..

  describe "favorite beer" do
    let(:user){ FactoryBot.create(:user) }

    it "has method for determining the favorite beer" do
      expect(user).to respond_to(:favorite_beer)
    end

    it "without ratings does not have a favorite beer" do
      expect(user.favorite_beer).to eq(nil)
    end

    it "is the only rated if only one rating" do
      beer = FactoryBot.create(:beer)
      rating = FactoryBot.create(:rating, score: 20, beer: beer, user: user)

      expect(user.favorite_beer).to eq(beer)
    end

    it "is the one with highest rating if several rated" do
      create_beers_with_many_ratings({user: user}, 10, 20, 15, 7, 9)
      best = create_beer_with_rating({ user: user }, 25 )

      expect(user.favorite_beer).to eq(best)
    end
  end
end # describe User

def create_beer_with_rating(object, score)
  beer = FactoryBot.create(:beer)
  FactoryBot.create(:rating, beer: beer, score: score, user: object[:user] )
  beer
end

def create_beers_with_many_ratings(object, *scores)
  scores.each do |score|
    create_beer_with_rating(object, score)
  end
end
```

### FactoryBot-troubleshooting

Seuraavaan on koottu muutama aiempien vuosien aikana vastaantullut virhetilanne

#### vahingossa luotu oliotehdas

Kannattaa huomata, että jos määrittelet FactoryBot-gemin testiympäristön lisäksi kehitysympäristöön, eli

```ruby
group :development, :test do
   gem 'factory_bot_rails'
    # ...
end
```

jos luot Railsin generaattorilla uusia resursseja, esim:

    rails g scaffold bar name:string

syntyy nyt samalla myös oletusarvoinen oliotehdas:

```ruby
FactoryBot.define do
  factory :bar do
    name "MyString"
  end

  # ...
end
```

Tämä saattaa aiheuttaa yllättäviä tilanteita (mm. jos määrittelet itse saman nimisen tehtaan, käytetään sen sijaan oletusarvoista tehdasta!), eli kannattanee määritellä gemi ainoastaan testausympäristöön luvun https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko4.md#testiymp%C3%A4rist%C3%B6t-eli-fixturet ohjeen tapaan.

#### testitietokantaan jäävät oliot

Normaalisti rspec-tyhjentää tietokannan jokaisen testin suorituksen jälkeen. Tämä johtuu sitä, että oletusarvoisesti rspec suorittaa jokaisen testin transaktiossa, joka rollbackataan eli perutaan testin suorituksen jälkeen. Testit eivät siis todellisuudessa edes talleta mitään tietokantaan.

Joskus testeissä voi kuitenkin mennä kantaan pysyvästi olioita.

Oletetaan että testaisimme luokkaa <code>Beer</code> seuraavasti:

```ruby
describe "when one beer exists" do
  beer = FactoryBot.create(:beer)

  it "is valid" do
    expect(beer).to be_valid
  end

  it "has the default style" do
    expect(beer.style).to eq("Lager")
  end
end
```

testin luoma <code>Beer</code>-olio menisi nyt pysyvästi testitietokantaan, sillä komento <code>FactoryBot.create(:beer)</code> ei ole minkään testin sisällä, eikä sitä siis suoriteta peruttavan transaktion aikana!

Testien ulkopuolelle, ei siis tule sijoittaa olioita luovaa koodia (poislukien testeistä kutsuttavat metodit). Olioiden luomisen on tapahduttava testikontekstissa, eli joko metodin <code>it</code> sisällä:

```ruby
describe "when one beer exists" do
  it "is valid" do
    beer = FactoryBot.create(:beer)
    expect(beer).to be_valid
  end

  it "has the default style" do
    beer = FactoryBot.create(:beer)
    expect(beer.style).to eq("Lager")
  end
end
```

komennon <code>let</code> tai <code>let!</code> sisällä:

```ruby
describe "when one beer exists" do
  let(:beer){FactoryBot.create(:beer)}

  it "is valid" do
    expect(beer).to be_valid
  end

  it "has the default style" do
    expect(beer.style).to eq("Lager")
  end
end
```

tai hieman myöhemmin esiteltävissä <code>before</code>-lohkoissa.

Saat poistettua testikantaan vahingossa menneet oluet käynnistämällä konsolin testiympäristössä komennolla <code>rails c -e test</code>.

#### validointi

Validoinneissa määritellyt uniikkiusehdot saattavat joskus tuottaa yllätyksiä. Käyttäjän käyttäjätunnus on määritelty uniikisi, joten testi

```ruby
describe "the application" do
  it "does something with two users" do
    user1 = FactoryBot.create(:user)
    user2 = FactoryBot.create(:user)

  # ...
  end
end
```

aiheuttaisi virheilmoituksen

```
1) User the application does something with two users
    Failure/Error: user2 = FactoryBot.create(:user)

    ActiveRecord::RecordInvalid:
      Validation failed: Username has already been taken
    # ./spec/models/user_spec.rb:77:in `block (3 levels) in <main>'
```

sillä FactoryBot yrittää nyt luoda kaksi käyttäjäolioa määritelmän

```ruby
factory :user do
  username { "Pekka" }
  password { "Foobar1" }
  password_confirmation { "Foobar1" }
end
```

perusteella, eli molemmille tulisi usernameksi 'Pekka'. Ongelma ratkeaisi antamalla toiselle luotavista oliosta joku muu nimi:

```ruby
describe "the application" do
  it "does something with two users" do
    user1 = FactoryBot.create(:user)
    user2 = FactoryBot.create(:user, username: "Vilma")

  # ...
  end
end
```

Toinen vaihtoehto olisi määritellä FactoryBotin käyttämät usernamet ns. sekvenssien avulla, ks.
https://www.rubydoc.info/gems/factory_bot/file/GETTING_STARTED.md#sequences

Tehdas muuttuisi seuraavaan muotoon:

```ruby
FactoryBot.define do
  sequence :username do |n|
    "Pekka#{n}"
  end

  factory :user do
    username { generate :username }
    password { "Foobar1" }
    password_confirmation { "Foobar1" }
  end

  # ...
end
```

Nyt jokainen peräkkäisten tehtaan <code>FactoryBot.create(:user)</code> kutsujen luomien olioiden usernamet olisivat _Pekka1_, _Pekka2_, _Pekka3_ ...

**Älä kuitenkaan muuta** tehdasta tähän muotoon, muuten osa viikon testeistä ei toimi!

## Testit ja debuggeri

Toivottavasti olet jo tässä vaiheessa kurssia rutinoitunut [debuggerin](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#debuggeri) käyttäjä. Koska testitkin ovat normaalia Ruby-koodia, on myös _binding.pry_ käytettävissä sekä testikoodissa että testattavassa koodissa. Testausympäristön tietokannan tila saattaa joskus olla yllättävä, kuten edellä olevista esimerkeistä näimme. Ongelmatilanteissa kannattaa ehdottomasti pysäyttää testikoodi debuggerilla ja tutkia vastaako testattavien olioiden tila oletettua.

## Yksittäisten testien suorittaminen

Rspecillä voi suorittaa myös yksittäisiä testejä tai describe-lohkoja, esim. seuraava suorittaisi ainoastaan tiedoston user_spec.rb riviltä 108 alkavan testin

```ruby
rspec spec/models/user_spec.rb:108
```

Jos/kun törmäät testeissäsi ongelmatilanteita:

- älä suorita kaikkia testejä, vaan rajaa suoritus ongelmallisiin testeihin
- käytä debuggeria

> ## Tehtävä 3
>
> ### Tämä ja seuraava tehtävä voivat olla jossain määrin haastavia. Tehtävien teko ei ole viikon jatkamisen kannalta välttämätöntä eli älä juutu tähän kohtaan. Voit tehdä tehtävät myös viikon muiden tehtävien jälkeen.
>
> Tee seuraavaksi TDD-tyylillä <code>User</code>-olioille metodi <code>favorite_style</code>, joka palauttaa tyylin, jonka oluet ovat saaneet käyttäjältä keskimäärin korkeimman reittauksen.
>
> Lisää käyttäjän sivulle tieto käyttäjän mielityylistä.
>
> Älä tee kaikkea yhteen metodiin (ellet ratkaise tehtävää tietokantatasolla ActiveRecordilla tai päädy muuten eleganttiin kompaktiin ratkaisuun), vaan määrittele tarvittaessa sopivia apumetodeja. Jos huomaat metodisi sisältävän yli 6 riviä koodia, teet asioita todennäköisesti joko liikaa tai liian kankeasti, joten refaktoroi koodiasi. Rubyn kokoelmissa on paljon tehtävään hyödyllisiä apumetodeja, ks. http://ruby-doc.org/core-2.5.1/Enumerable.html
>
> Kannattaa ehdottomasti hyödyntää _rails konsolia_ kun teet tehtävää
>
> Tee tarvittaessa apumetodeja rspec-tiedostoon, jotta testisi pysyvät siisteinä. Jos apumetodeista tulee samantapaisia, ei kannata copypasteta vaan yleistää ne.

> ## Tehtävä 4
>
> Tee vielä TDD-tyylillä <code>User</code>-olioille metodi <code>favorite_brewery</code>, joka palauttaa panimon, jonka oluet ovat saaneet käyttäjältä keskimäärin korkeimman reittauksen.
>
> Lisää käyttäjän sivulle tieto käyttäjän mielipanimosta.

Metodien <code>favorite_brewery</code> ja <code>favorite_style</code> tarvitsema toiminnallisuus on hyvin samankaltainen ja metodit ovatkin todennäköisesti enemmän tai vähemmän copy-pastea. Viikolla 5 tulee olemaan esimerkki koodin siistimisestä.

## Capybara

Siirrymme seuraavaksi järjestelmätason testaukseen. Kirjoitamme siis automatisoituja testejä, jotka käyttävät sovellusta normaalin käyttäjän tapaan selaimen kautta. De facto -tapa Rails-sovellusten selaintason testaamiseen on Capybaran https://github.com/jnicklas/capybara käyttö. Itse testit kirjoitetaan edelleen Rspecillä, capybara tarjoaa siis rspec-testien käyttöön selaimen simuloinnin.

Capybara on oletusarvoisesti määriteltynä projektissa. Lisätään Gemfileen (test-scopeen) apukirjasto [launchy](https://github.com/copiousfreetime/launchy) eli test-scopen pitäisi näyttää suunilleen seuraavalta:

```ruby
group :test do
  gem 'rspec-rails', '~> 6.0.0.rc1'
  gem 'factory_bot_rails'
  gem "capybara"
  gem "selenium-webdriver"
  gem "webdrivers"
  gem 'launchy'
end
```

Jotta gem saadaan käyttöön, suoritetaan tuttu komento <code>bundle install</code>.

Nyt olemme valmiina ensimmäiseen selaintason testiin.

Selaintason testit on tapana sijoittaa hakemistoon _spec/features_. Yksikkötestit organisoidaan useimmiten siten, että kutakin luokkaa testaavat testit tulevat omaan tiedostoonsa. Ei ole aina itsestään selvää, miten selaimen kautta suoritettavat käyttäjätason testit kannattaisi organisoida. Yksi vaihtoehto on käyttää kontrollerikohtaisia tiedostoja, toinen taas jakaa testit eri tiedostoihin järjestelmän eri toiminnallisuuksien mukaan.

Aloitetaan testien määrittely panimoihin liittyvästä toiminnallisuudesta, luodaan tiedosto spec/features/breweries_page_spec.rb:

```ruby
require 'rails_helper'

describe "Breweries page" do
  it "should not have any before been created" do
    visit breweries_path
    expect(page).to have_content 'Listing breweries'
    expect(page).to have_content 'Number of breweries: 0'
  end
end
```

Testi aloittaa navigoimalla <code>visit</code>-metodia käyttäen panimoiden listalle. Kuten huomaamme, Railsin polkuapumetodit ovat myös Rspec-testien käytössä. Tämän jälkeen tarkastetaan sisältääkö renderöity sivu tekstin 'Listing breweries' ja tiedon siitä että panimoiden lukumäärä on 0 eli tekstin 'Number of breweries: 0'. Capybara asettaa sen sivun, jolla testi kulloinkin on muuttujaan <code>page</code>.

Testejä tehdessä tulee (erittäin) usein tilanteita, joissa olisi hyödyllistä nähdä <code>page</code>-muuttujan kuvaaman sivun html-muotoinen lähdekoodi. Tämä onnistuu lisäämällä testiin komento <code>puts page.html</code>

Toinen vaihoehto on lisätä testiin komento <code>save_and_open_page</code>, joka tallettaa ja avaa kyseisen sivun oletusselaimessa. Linuxissa joudut määrittelemään selaimen oletusselaimeksi <code>BROWSER</code>-ympäristömuuttujan avulla. Esim. osaston koneilla saat määriteltyä oletusselaimeksi chromiumin komennolla:

    export BROWSER='/usr/bin/chromium-browser'

Määrittely on voimassa vain siinä shellissä jossa teet sen. Jos haluat määrittelystä pysyvän, lisää se tiedostoon ~/.bashrc

Jotta sekä <code>puts page.html</code>, sekä <code>save_and_open_page</code> komennot toimivat on ne sijoitettava ennen testin viimeistä riviä. Molemmat komennot voikin tässä testissä sijoittaa vaikka heti ensimmäiselle riville.

Suorita nyt testi tuttuun tapaan komennolla <code>rspec spec</code>. Jos haluat ajaa ainoastaan nyt määritellyn testin, muista että voit rajata suoritettavat testit antamalla komennon esim. muodossa

    rspec spec/features/breweries_page_spec.rb

**Testi ei todennäköisesti mene läpi.** Selvitä mistä vika johtuu ja korjaa testi tai sivulla oleva teksti. Komennon <code>save_and_open_page</code> käyttö on suositeltavaa!

Lisätään testi, joka testaa tilannetta, jossa tietokannassa on 3 panimoa:

```ruby
it "lists the existing breweries and their total number" do
  breweries = ["Koff", "Karjala", "Schlenkerla"]
  breweries.each do |brewery_name|
    FactoryBot.create(:brewery, name: brewery_name)
  end

  visit breweries_path

  expect(page).to have_content "Number of breweries: #{breweries.count}"

  breweries.each do |brewery_name|
    expect(page).to have_content brewery_name
  end
end
```

Lisätään vielä testi, joka tarkastaa, että panimoiden sivulta pääsee linkkiä klikkaamalla yksittäisen panimon sivulle. Hyödynnämme tässä capybaran metodia <code>click_link</code>, jonka avulla on mahdollista klikata sivulla olevaa linkkiä:

```ruby
it "allows user to navigate to page of a Brewery" do
  breweries = ["Koff", "Karjala", "Schlenkerla"]
  year = 1896
  breweries.each do |brewery_name|
    FactoryBot.create(:brewery, name: brewery_name, year: year += 1)
  end

  visit breweries_path

  click_link "Koff"

  expect(page).to have_content "Koff"
  expect(page).to have_content "Established at 1897"
end
```

Testi menee läpi olettaen että sivulla käytetty kirjoitusasu on sama kuin testissä. Ongelmatilanteissa testiin kannattaa lisätä komento <code>save_and_open_page</code> ja varmistaa visuaalisesti testin avaaman sivun sisältö.

Kahdessa edellisessä testissä on sama alkuosa, eli aluksi luodaan kolme panimoa ja navigoidaan panimojen sivulle.

Seuraavassa vielä refaktoroitu lopputulos, jossa yhteisen alustuksen omaavat testit on siirretty omaan describe-lohkoon, jolle on määritelty <code>before :each</code> -lohko alustusta varten.

```ruby
require 'rails_helper'

describe "Breweries page" do
  it "should not have any before been created" do
    visit breweries_path
    expect(page).to have_content 'Listing breweries'
    expect(page).to have_content 'Number of breweries: 0'

  end

  describe "when breweries exists" do
    before :each do
      # jotta muuttuja näkyisi it-lohkoissa, tulee sen nimen alkaa @-merkillä
      @breweries = ["Koff", "Karjala", "Schlenkerla"]
      year = 1896
      @breweries.each do |brewery_name|
        FactoryBot.create(:brewery, name: brewery_name, year: year += 1)
      end

      visit breweries_path
    end

    it "lists the breweries and their total number" do
      expect(page).to have_content "Number of breweries: #{@breweries.count}"
      @breweries.each do |brewery_name|
        expect(page).to have_content brewery_name
      end
    end

    it "allows user to navigate to page of a Brewery" do
      click_link "Koff"

      expect(page).to have_content "Koff"
      expect(page).to have_content "Established at 1897"
    end

  end
end
```

Huomaa, että describe-lohkon sisällä oleva <code>before :each</code> suoritetaan kertaalleen ennen jokaista describen alla määriteltyä testiä ja **jokainen testi alkaa tilanteesta, missä tietokanta on tyhjä**.

Kannattaa myös huomata, että jos <code>before :each</code> -lohkossa määriteltyihin muuttujiin on viitattava yksittäisistä testeistä, eli _it_-lohkoista, tulee muuttujien nimen alkaa @-merkillä.

## Käyttäjän toiminnallisuuden testaaminen

Siirrytään käyttäjän toiminnallisuuteen, luodaan tätä varten tiedosto _spec/features/users_page_spec.rb_. Aloitetaan testillä, joka varmistaa, että käyttäjä pystyy kirjautumaan järjestelmään:

```ruby
require 'rails_helper'

describe "User" do
  before :each do
    FactoryBot.create :user
  end

  describe "who has signed up" do
    it "can signin with right credentials" do
      visit signin_path
      fill_in('username', with: 'Pekka')
      fill_in('password', with: 'Foobar1')
      click_button('Log in')

      expect(page).to have_content 'Welcome back!'
      expect(page).to have_content 'Pekka'
    end
  end
end
```

Testi demonstroi lomakkeen kanssa käytävää interaktiota, komento <code>fill\_in</code> etsii lomakkeesta id-kentän perusteella tekstikenttää, jolle se syöttää parametrina annetun arvon. <code>click\_button</code> toimii kuten arvata saattaa, eli painaa sivulta etsittävää painiketta.

Huomaa, että testissä on <code>before :each</code>-lohko, joka luo ennen jokaista testiä FactoryBotiä käyttäen User-olion. Ilman olion luomista kirjautuminen ei onnistuisi, sillä tietokanta on jokaiseen testin suoritukseen lähdettäessä tyhjä.

Capybaran dokumentaation kohdasta the DSL ks. https://github.com/jnicklas/capybara#the-dsl löytyy lisää esimerkkejä mm. sivulla olevien elementtien etsimiseen ja esim. lomakkeiden käyttämiseen.

Tehdään vielä muutama testi käyttäjälle. Virheellisen salasanan syöttämisen pitäisi ohjata takaisin kirjaantumissivulle:

```ruby
  describe "who has signed up" do
    # ...

    it "is redirected back to signin form if wrong credentials given" do
      visit signin_path
      fill_in('username', with: 'Pekka')
      fill_in('password', with: 'wrong')
      click_button('Log in')

      expect(current_path).to eq(signin_path)
      expect(page).to have_content 'Username and/or password mismatch'
    end
  end
```

Testi hyödyntää metodia <code>current_path</code>, joka palauttaa sen polun minne testin suoritus on metodin kutsuhetkellä päätynyt. Metodin avulla varmistetaan, että käyttäjä uudelleenohjataan takaisin kirjautumissivulle epäonnistuneen kirjautumisen jälkeen.

Ei ole aina täysin selvää missä määrin sovelluksen bisneslogiikkaa kannattaa testata selaintason testien kautta. Edellä tekemämme käyttäjä-olion suosikkioluen, panimon ja oluttyylin selvittävien logiikoiden testaaminen on ainakin viisainta tehdä yksikkötesteinä.

Käyttäjätason testein voidaan esim. varmistua, että sivuilla näkyy sama tilanne, joka tietokannassa on, eli esim. panimoiden sivun testissä tietokantaan generoitiin 3 panimoa ja sen jälkeen testattiin että ne kaikki renderöityvät panimoiden listalle.

Myös sivujen kautta tehtävät lisäykset ja poistot kannattaa testata. Esim. seuraavassa testataan, että uuden käyttäjän rekisteröityminen lisää järjestelmän käyttäjien lukumäärää yhdellä:

```ruby
it "when signed up with good credentials, is added to the system" do
  visit signup_path
  fill_in('user_username', with: 'Brian')
  fill_in('user_password', with: 'Secret55')
  fill_in('user_password_confirmation', with: 'Secret55')

  expect{
    click_button('Create User')
  }.to change{User.count}.by(1)
end
```

Huomaa, että lomakkeen kentät määriteltiin <code>fill\_in</code>-metodeissa hieman eri tavalla kuin kirjautumislomakkeessa. Kenttien id:t voi ja kannattaa aina tarkastaa katsomalla sivun lähdekoodia selaimen _view page source_ -toiminnolla.

Testi siis odottaa, että _Create user_ -painikkeen klikkaaminen muuttaa tietokantaan talletettujen käyttäjien määrää yhdellä. Syntaksi on hieno, mutta kestää hetki ennen kuin koko Rspecin ilmaisuvoimainen kieli alkaa tuntua tutulta.

Pienenä detaljina kannattaa huomioida, että metodille <code>expect</code> voi antaa parametrin kahdella eri tavalla.
Jos metodilla testaa jotain arvoa, annetaan testattava arvo suluissa esim <code>expect(current_path).to eq(signin_path)</code>. Jos sensijaan testataan jonkin operaation (esim. edellä <code>click_button('Create User')</code>) vaikutusta jonkun sovelluksen olion (<code>User.count</code>) arvoon, välitetään suoritettava operaatio koodilohkona <code>expect</code>ille.

Lue aiheesta lisää Rspecin dokumentaatiosta https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers

Edellinen testi siis testasi, että selaimen tasolla tehty operaatio luo olion tietokantaan. Onko vielä tehtävä erikseen testi, joka testaa että luodulla käyttäjätunnuksella voi kirjautua järjestelmään? Kenties, edellinen testihän ei ota kantaa siihen tallentuiko käyttäjäolio tietokantaan oikein.

Potentiaalisia testauksen kohteita on kuitenkin niin paljon, että kattava testaus on mahdotonta ja testejä tulee pyrkiä ensisijaisesti kirjoittamaan niille asioille, jotka ovat riskialttiita hajoamaan.

Tehdään vielä testi oluen reittaamiselle. Tehdään testiä varten oma tiedosto _spec/features/ratings_page_spec.rb_

```ruby
require 'rails_helper'

describe "Rating" do
  let!(:brewery) { FactoryBot.create :brewery, name: "Koff" }
  let!(:beer1) { FactoryBot.create :beer, name: "iso 3", brewery:brewery }
  let!(:beer2) { FactoryBot.create :beer, name: "Karhu", brewery:brewery }
  let!(:user) { FactoryBot.create :user }

  before :each do
    visit signin_path
    fill_in('username', with: 'Pekka')
    fill_in('password', with: 'Foobar1')
    click_button('Log in')
  end

  it "when given, is registered to the beer and user who is signed in" do
    visit new_rating_path
    select('iso 3', from: 'rating[beer_id]')
    fill_in('rating[score]', with: '15')

    expect{
      click_button "Create Rating"
    }.to change{Rating.count}.from(0).to(1)

    expect(user.ratings.count).to eq(1)
    expect(beer1.ratings.count).to eq(1)
    expect(beer1.average_rating).to eq(15.0)
  end
end
```

Testi rakentaa käyttämänsä panimon, kaksi olutta ja käyttäjän metodin <code>let!</code> aiemmin käyttämämme metodin <code>let</code> sijaan. Näin toimitaan siksi että huutomerkitön versio ei suorita operaatiota välittömästi vaan vasta siinä vaiheessa kun koodi viittaa olioon eksplisiittisesti. Olioon <code>beer1</code> viitataan koodissa vasta lopun tarkastuksissa, eli jos olisimme luoneet sen metodilla <code>let</code> olisi reittauksen luomisvaiheessa tullut virhe, sillä olut ei olisi vielä ollut kannassa, eikä vastaavaa select-elementtiä olisi löytynyt.

Testin <code>before</code>-lohkossa on koodi, jonka avulla käyttäjä kirjautuu järjestelmään. On todennäköistä, että samaa koodilohkoa tarvitaan useissa eri testitiedostoissa. Useassa eri paikassa tarvittava testikoodi kannattaa eristää omaksi apumetodikseen ja sijoittaa [moduuliin](https://relishapp.com/rspec/rspec-core/docs/helper-methods/define-helper-methods-in-a-module), jonka kaikki sitä tarvitsevat testitiedostot voivat sisällyttää itseensä. Luodaan moduli <code>Helpers</code>hakemistoon _spec_ sijoitettavaan tiedostoon _helpers.rb_ ja siirretään kirjautumisesta vastaava koodi sinne:

```ruby
module Helpers

  def sign_in(credentials)
    visit signin_path
    fill_in('username', with:credentials[:username])
    fill_in('password', with:credentials[:password])
    click_button('Log in')
  end
end
```

Metodi <code>sign_in</code> saa siis käyttäjätunnus/salasanaparin parametrikseen hashina.

Lisätään tiedostoon _rails_helper.rb_ heti muiden require-komentojen jälkeen rivi

    require 'helpers'

Voimme ottaa modulin määrittelemän metodi käyttöön testeissä komennolla <code>include Helper</code>:

```ruby
require 'rails_helper'

include Helpers

describe "Rating" do
  let!(:brewery) { FactoryBot.create :brewery, name: "Koff" }
  let!(:beer1) { FactoryBot.create :beer, name: "iso 3", brewery:brewery }
  let!(:beer2) { FactoryBot.create :beer, name: "Karhu", brewery:brewery }
  let!(:user) { FactoryBot.create :user }

  before :each do
    sign_in(username: "Pekka", password: "Foobar1")
  end
```

ja

```ruby
require 'rails_helper'

include Helpers

describe "User" do
  before :each do
    FactoryBot.create :user
  end

  describe "who has signed up" do
    it "can signin with right credentials" do
      sign_in(username: "Pekka", password: "Foobar1")

      expect(page).to have_content 'Welcome back!'
      expect(page).to have_content 'Pekka'
    end

    it "is redirected back to signin form if wrong credentials given" do
      sign_in(username: "Pekka", password: "wrong")

      expect(current_path).to eq(signin_path)
      expect(page).to have_content 'Username and/or password mismatch'
    end
  end

  it "when signed up with good credentials, is added to the system" do
    visit signup_path
    fill_in('user_username', with: 'Brian')
    fill_in('user_password', with: 'Secret55')
    fill_in('user_password_confirmation', with: 'Secret55')

    expect{
      click_button('Create User')
    }.to change{User.count}.by(1)
  end
end
```

Kirjautumisen toteutuksen siirtäminen apumetodiin siis kasvattaa myös testien luettavuutta, ja jos kirjautumissivun toiminnallisuus myöhemmin muuttuu, on testien ylläpito helppoa, koska muutoksia ei tarvita kuin yhteen kohtaan.

Saattaa olla järkevää siirtää myös aiemmin tiedostoon _user_spec.rb_ määrittelemämme apumetodit <code>create\_beer\_with\_rating</code> ja <code>create\_beers\_with\_many\_ratings</code> moduuliin _Helpers_, erityisesti jos jatkossa tulee tilanteita, joissa samaa toiminnallisuutta tarvitaan muissakin testeissä.

> ## Tehtävä 5
>
> Tee testi, joka varmistaa, että järjestelmään voidaan lisätä www-sivun kautta olut, jos oluen nimikenttä saa validin arvon (eli se on epätyhjä).
>
> Tee myös testi, joka varmistaa, että selain näyttää asiaan kuuluvan virheilmoituksen jos oluen nimi ei ole validi, ja että tälläisessä tapauksessa tietokantaan ei talletu mitään.
>
> Huomaa, että testin on luotava sovellukseen ainakin yksi panimo, jotta oluiden luominen olisi mahdollista.
>
> **HUOM:** ohjelmassasi saattaa olla bugi tilanteessa, jossa yritetään luoda epävalidin nimen omaava olut. Kokeile toiminnallisuutta selaimesta. Syynä tälle on selitetty viikon alussa, kohdassa https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko4.md#muutama-huomio. Korjaa vika koodistasi.
>
> Muista ongelmatilanteissa komento <code>save_and_open_page</code>!

> ## Tehtävä 6
>
> Tee testi joka varmistaa, että tietokannassa olevat reittaukset ja niiden lukumäärä näytetään sivulla _ratings_. Jos lukumäärää ei toteutuksessani näytetä, korjaa puute.
>
> **Vihje**: voit tehdä testin esim. siten, että luot aluksi FactoryBotillä reittauksia tietokantaan. Tämän jälkeen voit testata capybaralla sivun ratings sisältöä.
>
> Muista ongelmatilanteissa komento <code>save_and_open_page</code>!

> ## Tehtävä 7
>
> Tee testi joka varmistaa, että käyttäjän reittaukset näytetään käyttäjän sivulla. Käyttäjän sivulla tulee siis näkyä kaikki käyttäjän omat muttei muiden käyttäjien tekemiä reittauksia.
>
> Huomaa, että navigoidessasi käyttäjän <code>user</code> sivulle, joudut antamaan metodille <code>visit</code> polun määritteleväksi parametriksi <code>user_path(user)</code>, eli yleensä käytetty lyhempi muoto (olio itse) ei capybaran kanssa toimi.

> ## Tehtävä 8
>
> Tee testi, joka varmistaa että käyttäjän poistaessa oma reittauksensa, se poistuu tietokannasta.
>
> Jos sivulla on useita linkkejä joilla on sama nimi, ei <code>click_link</code> toimi. Joudut tälläisissä tilanteissa yksilöimään mikä linkeistä valitaan, ja se ei ole välttämättä ihan helppoa. Apua löytyy [capybaran dokumentaatiosta](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders) ja [täältä](http://stackoverflow.com/questions/6733427/how-to-click-on-the-second-link-with-the-same-text-using-capybara-in-rails-3)
>
> Vaikka ratkaisu onkin lyhyt, ei tehtävä ole välttämättä helpoimmasta päästä. Jos jäät jumiin, kannattanee tehdä viikon muut tehtävät ensin tai/ja kysyä apua pajassa/Telegramissa.

> ## Tehtävä 9
>
> Jos teit tehtävät 3-4, laajenna käyttäjän sivua siten, että siellä näytetään käyttäjän lempioluttyyli sekä lempipanimo. Tee ominaisuudelle myös capybara-testit. Monimutkaista laskentaa testeissä ei kannata testata, sillä yksikkötestit varmistavat toiminnallisuuden jo riittävissä määrin.

## Testauskattavuus

Testien rivikattavuus (line coverage) mittaa kuinka monta prosenttia ohjelman koodiriveistä tulee suoritettua testien suorituksen yhteydessä. Rails-sovelluksen testikattavuus on helppo mitata _simplecov_-gemin avulla, ks. https://github.com/colszowka/simplecov

Gem otetaan käyttöön lisäämällä Gemfilen test -scopeen rivi

    gem 'simplecov', require: false

**Huom** normaalin <code>bundle install</code>-komennon sijaan saatat joutua antamaan tässä vaiheessa komennon <code>bundle update</code>, jotta kaikista gemeistä saatiin asennetuiksi yhteensopivat versiot.

Jotta simplecov saadaan käyttöön tulee tiedoston rails_helper.rb alkuun, **kahdeksi ensimmäiseksi riviksi** lisätä seuraavat:

```ruby
require 'simplecov'
SimpleCov.start('rails')
```

Sitten ajetaan testit (ongelmatilanteessa ks. ylempi huomautus)

```ruby
$ rspec spec
..................................

Finished in 1.25 seconds (files took 1.95 seconds to load)
34 examples, 0 failures

Coverage report generated for RSpec to /Users/mluukkai/opetus/ratebeer/coverage. 161 / 333 LOC (48.35%) covered.
```

Testien rivikattavuus on siis 48.35 prosenttia. Tarkempi raportti on nähtävissä avaamalla selaimella tiedosto coverage/index.html. Kuten kuva paljastaa, on suuria osia ohjelmasta, erityisesti kontrollereista vielä erittäin huonosti testattu:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w4-1.png)

Suurikaan rivikattavuus ei tietysti vielä takaa että testit testaavat järkeviä asioita. Helposti mitattavana metriikkana se on kuitenkin parempi kuin ei mitään ja näyttää ainakin ilmeisimmät puutteet testeissä.

> ## Tehtävä 10
>
> Ota simplecov käyttöön ohjelmassasi. Tutki raportista (klikkaamalla punaisella tai keltaisella merkittyjä luokkia) mitä rivejä koodissasi on vielä täysin testaamatta.

## Jatkuva integraatio

[Jatkuvalla integraatiolla](http://martinfowler.com/articles/continuousIntegration.html) (engl. continuous integration) tarkoitetaan käytännettä, jossa ohjelmistokehittäjät integroivat koodiin tekemänsä muutokset yhteiseen kehityshaaraan mahdollisimman usein. Periaatteena on pitää ohjelman kehitysversio koko ajan toimivana eliminoiden näin raskas erillinen integrointivaihe. Toimiakseen jatkuva integraatio edellyttää kattavaa automaattisten testien joukkoa. Yleensä jatkuvan integraation yhteydessä käytetään keskitettyä palvelinta, joka tarkkailee repositorioa, jolla kehitysversio sijaitsee. Kun kehittäjä integroi koodin kehitysversioon, integraatiopalvelin huomaa muutoksen, buildaa koodin ja ajaa testit. Jos testit eivät mene läpi, tiedottaa integraatiopalvelin tästä tavalla tai toisella asianomaisia.

Github tarjoaa kehittäjien käyttöön [Github Actionsin](https://github.com/features/actions), joka onkin saanut paljon jalansijaa muiden CI:tä tarjoavien palveluiden joukossa. Github Actionsin puolesta puhuu sen integraatio suoraan githubiin, sekä actionsien marketplace josta löytyy CI:hin lisättäviä actioneja. Näistä lisää myöhemmin.

Githubissa olevat projektit on helppo asettaa Github actionsin tarkkailtaviksi.

> ## Tehtävä 11
>
> ### Tämän ja parin seuraavan tehtävän tekeminen ei ole välttämätöntä viikon jatkamisen kannalta. Voit tehdä tämän tehtävän myös viikon muiden tehtävien jälkeen.
>
> Mene oman projektisi repositorioon ja paina yläpalkista Actions-painiketta. Jos sinulla ei ole olemassaolevia actioneja github vie sinut suoraan sivulle, jossa ehdotetaan valmiita pohjia valittavaksi.
>
> ![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w4-2.png)
>
> Valitaan Ruby on Rails painamalla Configure-nappia. Tämän seurauksena github vie sivulle jossa muokataan <code>rubyonrails.yml</code> nimistä tiedostoa. Tämä workflow tiedosto kertoo Github actionsille mitä CI:n tulee tehdä.
>
> Tiedoston sisältö ei sellaisena kuitenkaan toimi, joten vaihdetaan sisällöksi aluksi seuraava:
>
> ```
> # This workflow uses actions that are not certified by GitHub. They are
> # provided by a third-party and are governed by separate terms of service,
> # privacy policy, and support documentation.
> #
> # This workflow will install a prebuilt Ruby version, install dependencies, and
> # run tests and linters.
> name: "Ruby on Rails CI"
> on:
>   push:
>     branches: [ "main" ] # Jos repositoriosi päähaara ei ole main, muuta nämä
>   pull_request:
>     branches: [ "main" ]
> jobs:
>   test:
>     runs-on: ubuntu-22.04
>     services:
>       postgres:
>         image: postgres:11-alpine
>         ports:
>           - "5432:5432"
>         env:
>           POSTGRES_DB: rails_test
>           POSTGRES_USER: rails
>           POSTGRES_PASSWORD: password
>     env:
>       RAILS_ENV: test
>       DATABASE_URL: "postgres://rails:password@localhost:5432/rails_test"
>     steps:
>       - name: Checkout code
>         uses: actions/checkout@v3
>       # Add or replace dependency steps here
>       - name: Install Ruby and gems
>         uses: ruby/setup-ruby@v1
>         with:
>           bundler-cache: true
>       - name: Run tests
>         run: bundle exec rspec
> ```
>
> Erona defaultina tarjottavaan versioon tässä on se, että sekä Ubuntusta ja Rubyn setuppaavasta actionista käytetään uusimpia versiota, jotta Rubyn 3.1.2. versio toimii.
>
> Vaihdettuasi sisällön valitse Start commit ja lisää tiedosto versionhallintaasi. GitHub Actions lähteekin suoraan käyntiin ja suorittaa testit. 
>
> Jos jokin testi ei toimi GitHub actionseissa korjaa se!

> ## Tehtävä 12
>
> Lisätään nyt myös Rubocop GitHub Actioniin. Käytetään tässä avuksemme [marketplacesta valmiiksi löytyvää actionia](https://github.com/marketplace/actions/rubocop-linter-action), jonka voimme liittää omaamme.
>
> Lisää <code>rubyonrails.yml</code> tiedostoon seuraava sisältö:
>
> ```
>  lint:
>    runs-on: ubuntu-22.04
>    steps:
>      - name: Checkout code
>        uses: actions/checkout@v3
>      - name: Install Ruby and gems
>        uses: ruby/setup-ruby@v1
>        with:
>          bundler-cache: true
>      - name: RuboCop Linter Action
>        uses: andrewmcodes-archive/rubocop-linter-action@v3.3.0
>        env:
>          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
> ```
>
> GITHUB_TOKEN rivillä käytetään [Githubin tarjoamaa automaattista tokenia](https://docs.github.com/en/actions/security-guides/automatic-token-authentication), jolla pystytään autentikoimaan githubin sovellukset.
>
> Nyt Github Actionsin pitäisi suorittaa sekä testit, että Rubocop sovellukselle joka kerta kun GitHubiin lisätään muutoksia.

## Continuous delivery

Jatkuvaa integraatiota vielä askeleen eteenpäin viety käytäntö on jatkuva toimittaminen eng. continuous delivery http://en.wikipedia.org/wiki/Continuous_delivery jonka yhtenä osatekijänä on jatkuva deployaus, eli idea, jonka mukaan sovelluksen uusin versio aina integroimisen yhteydessä myös deployataan eli käynnistetään tuotantoympäristön kaltaiseen ympäristöön tai parhaassa tapauksessa suoraan tuotantoon.

Eriyisesti Web-sovellusten yhteydessä jatkuva deployaaminen saattaa olla hyvinkin vaivaton operaatio.

> ## Tehtävä 13
>
> ### Tämän ja seuraavan tehtävän tekeminen ei ole välttämätöntä viikon jatkamisen kannalta. Voit tehdä tämän tehtävän myös viikon muiden tehtävien jälkeen.
>
> Toteuta sovelluksellesi jatkuva automaattinen deployaaminen Fly.io:n tai Herokuun
>
> Fly.io:n ohje seuraavasta https://fly.io/docs/app-guides/continuous-deployment-with-github-actions/
>
> Herokuun ohje seuraavasta https://devcenter.heroku.com/articles/github-integration
>
> **_HUOM_** Käyttäessäsi Herokua, muista valita "Wait for CI to pass before deploy"
>
> Voit testata toimiiko CI/CD-putkesi tekemällä jonkin muutoksen sovellukseesi ja lisäämllä muutoksen GitHubiin ja seuraamalla tuleeko muutos myös Fly.io:ssa/Herokussa olevaan sovellukseesi. Herokun tapauksessa näet Overview välilehdeltä "Latest Activity" syötteestä mitä putkessa tapahtuu.

## Koodin laatumetriikat

Testauskattavuuden lisäksi myös koodin laatua kannattaa valvoa. SaaS-palveluna toimivan Codeclimaten https://codeclimate.com avulla voidaan generoida Rails-koodista erilaisia laatumetriikoita.

> ## Tehtävä 14
>
> ### Tämän tehtävän tekeminen ei ole välttämätöntä viikon jatkamisen kannalta. Voit tehdä tämän tehtävän myös viikon muiden tehtävien jälkeen.
>
> Codeclimate on ilmainen opensource-projekteille. Kirjaudu sovelukseen ositteessa https://codeclimate.com/login/github/join ja lisää projektisi "Open source"-osista.
>
> Codeclimate valittelee todennäköisesti koodissa olevasta samanlaisuudesta. Kyseessä on kuitenkin Rails scaffoldingin luoma hieman ruma koodi, joten sen voi jättää paikalleen.
>
> Linkitä myös laatumetriikkaraportti repositorion README-tiedostoon:
>
> Löydät linkin raporttiin seuraavasti
>
> ![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w4-4c.png)
>
> Nyt myös codeclimate aiheuttaa sovelluskehittäjälle sopivasti painetta pitää koodi koko ajan hyvälaatuisena!

Sovelluskehittäjän elämää helpottavien pilvipalveluiden määrä kasvaa kovaa vauhtia. Simplecov:in sijaan tai lisäksi testauskattavuuden raportoinnin voi delegoida Coveralls https://coveralls.io/ -nimiselle pilvipalvelulle. Jätämme sen kuitenkin tälläkertaa tekemättä.

## Kirjautuneiden toiminnot

Jätetään testien teko hetkeksi ja palataan muutamaan aiempaan teemaan. Viikolla 2 rajoitimme http basic -autentikaation avulla sovellustamme siten, että ainoastaan admin-salasanan syöttämällä oli mahdollista poistaa panimoita. [Viikolla 3](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko3.md#vain-omien-reittausten-poisto) rajoitimme sovelluksen toiminnallisuutta siten, että reittausten poistaminen ei ole mahdollista kuin reittauksen tehneelle käyttäjälle. Sen sijaan esim. olutkerhojen ja oluiden luominen, poistaminen ja editointi on tällä hetkellä mahdollista jopa ilman kirjautumista.

Luovutaan http basic -autentikoinnin käytöstä ja muutetaan sovellusta siten, että oluita, panimoita ja olutkerhoja voivat luoda, muokata ja poistaa ainoastaan kirjautuneet käyttäjät.

Aloitetaan poistamalla http basic -autentikaatio. Eli poistetaan panimokontrollerista rivi

    before_action :authenticate, only: [:destroy]

sekä metodi <code>authenticate</code>. Nyt kuka tahansa voi jälleen poistaa panimoita.

Aloitetaan suojauksen lisääminen.

Näkymistä on helppo poistaa oluiden, olutkerhojen ja panimoiden muokkaus -ja luontilinkit siinä tapauksessa, jos käyttäjä ei ole kirjautunut järjestelmään.

Esim. näkymästä views/beers/index.html.erb voidaan nyt poistaa kirjautumattomilta käyttäjiltä sivun lopussa oleva oluiden luomislinkki:

```erb
<% if not current_user.nil? %>
  <%= link_to "New beer", new_beer_path %>
<% end %>
```

Eli linkkielementti näytetään ainoastaan jos <code>current_user</code> ei ole <code>nil</code>. Voimme myös hyödyntää if:in kompaktimpaa muotoa:

```erb
<%= link_to('New Beer', new_beer_path) if not current_user.nil? %>
```

Nyt siis <code>link_to</code> metodi suoritetaan (eli linkin koodi renderöityy) ainoastaan jos if:in ehto on tosi. if not -muotoiset ehtolauseet eivät ole kovin hyvää Rubya, parempi olisikin käyttää <code>unless</code>-ehtolausetta:

```erb
<%= link_to('New Beer', new_beer_path) unless current_user.nil? %>
```

Eli renderöidään linkki **ellei** <code>current_user</code> ei ole <code>nil</code>.

Oikeastaan <code>unless</code> on nyt tarpeeton, Rubyssä nimittäin <code>nil</code> tulkitaan epätodeksi, eli kaikkien siistein muoto komennosta on

```erb
<%= link_to('New Beer', new_beer_path) if current_user %>
```

Poistamme lisäys-, poisto- ja editointilinkit pian, ensin kuitenkin tarkastellaan kontrolleritason suojausta, nimittäin vaikka kaikki linkit rajoitettuihin toimenpiteisiin poistettaisiin, ei mikään estä tekemästä suoraa HTTP-pyyntöä sovellukselle ja tekemästä näin kirjautumattomilta rajoitettua toimenpidettä.

On siis vielä tehtävä kontrolleritasolle varmistus, että jos kirjautumaton käyttäjä jostain syystä yrittää tehdä suoraan HTTP:llä kiellettyä toimenpidettä, ei toimenpidettä suoriteta.

Päätetään ohjata rajoitettua toimenpidettä yrittävä kirjautumaton käyttäjä kirjautumissivulle.

Määritellään luokkaan <code>ApplicationController</code> seuraava metodi:

```ruby
def ensure_that_signed_in
  redirect_to signin_path, notice: 'you should be signed in' if current_user.nil?
end
```

Eli jos metodia kutsuttaessa käyttäjä ei ole kirjautunut, suoritetaan uudelleenohjaus kirjautumissivulle. Koska metodi on sijoitettu luokkaan <code>ApplicationController</code> jonka kaikki kontrollerit perivät, on se kaikkien kontrollereiden käytössä.

Lisätään metodi esifiltteriksi (ks. http://guides.rubyonrails.org/action_controller_overview.html#filters ja https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#yksinkertainen-suojaus) olut- ja panimo- ja olutkerhokontrollerille kaikille metodeille paitsi index:ille ja show:lle:

```ruby
class BeersController < ApplicationController
  before_action :ensure_that_signed_in, except: [:index, :show]

  #...
end
```

Esim. uutta olutta luotaessa, ennen metodin <code>create</code> suorittamista, Rails suorittaa esifiltterin <code>ensure_that_signed_in</code>, joka ohjaa kirjautumattoman käyttäjän kirjautumissivulle. Jos käyttäjä on kirjautunut järjestelmään, ei filtterimetodi tee mitään, ja uusi olut luodaan normaaliin tapaan.

Kokeile selaimella, että muutokset toimivat, eli että kirjautumaton käyttäjä ohjautuu kirjautumissivulle kaikilla esifiltterillä rajoitetuilla toiminnoilla mutta että kirjautuneet pääsevät sivuille ilman ongelmaa.

> ## Tehtävä 15
>
> Estä esifiltterien avulla kirjautumattomilta käyttäjiltä panimoiden ja olutseurojen suhteen muut toiminnot paitsi kaikkien listaus ja yksittäisen resurssin tietojen tarkastelu (eli metodit <code>show</code> ja <code>index</code>)
>
> Kun olet varmistanut että toiminnallisuus on kunnossa, poista näkymistä ylimääräiset luomis-, poisto- ja editointilinkit kirjautumattomilta käyttäjiltä

> ## Tehtävä 16
>
> Tehtävää 15 ennen tekemiemme laajennustan takia muutama ohjelman testeistä menee rikki. Korjaa testit

## Sovelluksen ulkoasun hienosäätö

Voit halutessasi tehdä hienosäätöä sovelluksen näkymiin, esim. poistaa resurssien poisto- ja editointilinkit listaussivulta. Nämä muutokset eivät ole välttämättömiä ja tulevat viikotkaan eivät muutoksiin nojaa.

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi GitHubiin. Deployaa myös uusin versio Fly.io:n tai Herokuun. Muista myös testata Rubocopilla, että koodisi noudattaa edelleen määriteltyjä tyylisääntöjä.

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/stats/courses/rails2022/
