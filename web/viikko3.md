## Kisko afterparty pe 16.12. klo 16-18

Suomen johtava Rails-talo [Kisko](https://www.kiskolabs.com/) järjestää kurssilaisille illanvieton pe 16.12. klo 16-18

<img src="https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/kisko.png" width="600">

Jos haluat mukaan, kysy ilmoittautumislinkkiä matti.luukkainen@helsinki.fi tai Discordissa kurssikanavalla tai @mluukkai

Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 2 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit täydentää ratkaisusi tehtävien palautusjärjestelmän kautta näkyvän esimerkivastauksen avulla.

## Rails-ohjelmoijan workflow

Railsia tehtäessä optimaalinen työskentelytapa poikkeaa merkittävästi esim. Java-ohjelmoinnista. Railsia _ei_ yleensä kannata ohjelmoida siten, että editoriin yritetään kirjoittaa paljon valmista koodia, jonka toimivuus sitten testataan menemällä koodin suorittavalle sivulle. Osittain syy tähän on kielen dynaaminen tyypitys ja tulkattavuus, joka tekee parhaillekin IDE:ille koodin kattavan tarkastuksen haasteelliseksi. Toisaalta kielen tulkattavuus ja konsolityökalut (konsoli ja debuggeri) mahdollistavat pienempien koodinpätkien toiminnallisuuden testaamisen ennen niiden siirtämistä editoitavaan kooditiedostoon.

Tarkastellaan esimerkkinä viime viikolla toteutetun oluiden reittausten keskiarvon toteuttamista luontevaa Rails-workflowta noudattaen.

Jokainen olut siis sisältää kokoelman reittauksia:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings
end
```

Tehtävänämme on luoda oluelle metodi <code>average</code>

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    # code here
  end
end
```

Voisimme toteuttaa keskiarvon laskemisen "Javamaisesti" laskemalla summan käymällä reittauksen läpi alkio alkiolta ja jakamalla summan alkioden määrällä.

Kaikki Rubyn kokoelmamaiset asiat (mm. taulukko ja <code>has_many</code>-kenttä) sisältävät Enumerable-moduulin (ks. http://ruby-doc.org/core-2.5.1/Enumerable.html) tarjoamat apumetodit. Päätetäänkin hyödyntää apumetodeja keskiarvon laskemisessa.

Koodin kirjoittamisessa kannattaa _ehdottomasti_ hyödyntää konsolia. Oikeastaan konsoliakin parempi vaihtoehdo on debuggerin käyttö. Debuggerin avulla saadaan avattua konsoli suoraan siihen kontekstiin, johon koodia ollaan kirjoittamassa. Lisätään metodikutsuun debuggerin käynnistävä komento <code>binding.pry</code>:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    binding.pry
  end
end
```

Avataan sitten Rails konsoli (eli komento _rails c_ komentoriviltä), luetaan tietokannasta reittauksia sisältävä olio ja kutsutaan sille metodia <code>average</code>:

```ruby
irb(main):026:0> b = Beer.first
   (0.1ms)  SELECT sqlite_version(*)
  Beer Load (5.0ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
=>
#<Beer:0x00007f044a848a00
...
irb(main):027:0> b.average
[7, 14] in /myapp/app/models/beer.rb
     7|   def to_s
     8|     "#{name} #{brewery.name}"
     9|   end
    10|
    11|   def average
=>  12|     binding.pry
    13|   end
    14| end
=>#0	Beer#average at /myapp/app/models/beer.rb:12
  #1	<main> at (irb):27
  # and 28 frames (use `bt' command for all frames)
(rdbg)
```

eli saamme auki debuggerisession, joka avautuu metodin sisälle. Pääsemme siis käsiksi kaikkiin oluen tietoihin.

Olioon itseensä päästään käsiksi viitteellä <code>self</code>

```ruby
(rdbg) self
#<Beer:0x00007f044a848a00
 id: 1,
 name: "Iso 3",
 style: "Lager",
 brewery_id: 1,
 created_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00,
 updated_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00>
```

ja olioiden kenttiin pistenotaatiolla tai pelkällä kentän nimellä:

```ruby
(ruby) self.name
"Iso 3"
(rdbg) style
"Lager"
(rdbg)
```

Huomaa, että jos metodin sisällä on tarkotus muuttaa olion kentän arvoa, on käytettävä pistenotaatiota:

```ruby
def metodi
  # seuraavat komennot tulostavat olion kentän name arvon
  puts self.name
  puts name

  # alustaa metodin sisälle muuttujan name ja antaa sille arvon
  name = "StrongBeer"

  # muuttaa olion kentän name arvoa
  self.name = "WeakBeer"
end
```

Voimme siis viitata oluen reittauksiin oluen metodin sisältä kentän nimellä <code>ratings</code>:

```ruby
(rdbg) ratings
  Rating Load (3.6ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
[#<Rating:0x00007f044927afe8
  id: 2,
  score: 22,
  beer_id: 1,
  created_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00,
  updated_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00>,
 #<Rating:0x00007f0449285f38
  id: 3,
  score: 17,
  beer_id: 1,
  created_at: Fri, 19 Aug 2022 14:09:11.750743000 UTC +00:00,
  updated_at: Fri, 19 Aug 2022 14:09:11.750743000 UTC +00:00>]
```

Katsotaan yksittäistä reittausta:

```ruby
(ruby) ratings.first
#<Rating:0x00007f044927afe8
 id: 2,
 score: 21,
 beer_id: 1,
 created_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00,
 updated_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00>
```

summataksemme reittaukset, tulee siis jokaisesta reittausoliosta ottaa sen kentän <code>score</code> arvo:

```ruby
(ruby) ratings.first.score
21
```

Enumerable-modulin metodi <code>map</code> tarjoaa keinon muodostaa kokoelman perusteella uusi kokoelma, jonka alkiot saadaan alkuperäisen kokelman alkioista, suorittamalla jokaiselle alkiolle mäppäys-funktio.

Jos alkuperäisen kokoelman alkioon viitataan nimellä <code>r</code>, mäppäysfunktio on yksinkertainen:

```ruby
(ruby) r = ratings.first
#<Rating:0x00007f044927afe8>
(ruby) r.score
21
```

Nyt voimme kokeilla mitä <code>map</code> tuottaa:

```ruby
(ruby) ratings.map { |r| r.score }
[22, 17]
```

mäppäysfunktio siis annetaan metodille <code>map</code> parametriksi aaltosulkein erotettuna koodilohkona. Koodilohko voitaisiin erottaa myös <code>do end</code>-parina, molemmat tuottavat saman lopputuloksen:

```ruby
(ruby) ratings.map do |r| r.score end
[22, 17]
```

Metodin map avulla saamme siis muodostettua reittausten kokoelmasta taulukon reittausten arvoja. Seuraava tehtävä on summata nämä arvot.

Rails on lisännyt kaikille Enumerableille metodin
[sum](http://apidock.com/rails/Enumerable/sum), kokeillaan sitä mapilla aikansaamaamme taulukkoon.

```ruby
(ruby) ratings.map { |r| r.score }.sum
39
```

Jotta saamme vielä aikaan keskiarvon, on näin saatava summa jaettava alkioiden kokonaismäärällä. Varmistetaan ensin kokonaismäärän laskevan metodin <code>count</code> tominta:

```ruby
(ruby) ratings.count
  Rating Count (2.2ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
2
```

ja muodostetaan sitten keskiarvon laskeva onelineri:

```ruby
(ruby) ratings.map { |r| r.score }.sum / ratings.count
  Rating Count (2.2ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
19
```

huomaamme että lopputulos pyöristyy väärin. Kyse on tietenkin siitä että sekä jaettava että jakaja ovat kokonaislukuja. Muutetaan toinen näistä liukuluvuksi. Kokeillaan ensin miten kokonaisluvusta liukuluvun tekevä metodi toimii:

```ruby
> 1.to_f
=> 1.0
```

Jos et tiedä miten joku asia tehdään Rubyllä, google tietää.

Mieti sopiva hakusana niin saat melko varmasti vastauksen. Kannattaa kuitenkin olla hiukan varovainen ja tutkia ainakin muutama googlen vastaus. Ainakin kannattaa varmistaa että vastauksessa puhutaan riittävän tuoreesta Rubyn tai Railsin versiosta.

Rubyssä ja Railsissa on useimmiten joku valmis metodi tai gemi melkein kaikkeen, eli pyörän uudelleenkeksimisen sijaan kannattaa aina Googlata tai vilkuilla dokumentaatiota.

Muodostetaan sitten lopullinen versio keskiarvon laskevasta koodista:

```ruby
(ruby) ratings.map { |r| r.score }.sum / ratings.count.to_f
  Rating Count (2.4ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
19.5
```

Nyt koodi on valmis ja testattu, joten se voidaan kopioida metodiin:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    ratings.map{ |r| r.score }.sum / ratings.count.to_f
  end

end
```

Testataan metodia, eli poistutaan debuggerista, _lataamalla_ uusi koodi, hakemalla olio ja suorittamalla metodi:

```ruby
irb(main):001:0> b = Beer.first
irb(main):002:0> b.average
  Rating Load (1.8ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
  Rating Count (2.3ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=> 19.5
```

Jatkotestaus kuitenkin paljastaa että kaikki ei ole hyvin:

```ruby
irb(main):003:0> b = Beer.second
irb(main):004:0> b.average
  Rating Load (2.3ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
  Rating Count (3.1ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
=> NaN
```

eli kannassa toisena olevan oluen reittausten keskiarvo on <code>NaN</code>. Turvaudutaan jälleen debuggeriin. Laitetaan komento <code>binding.pry</code> keskiarvon laskevaan metodiin, uudelleenladataan koodi ja kutsutaan metodia ongelmalliselle oliolle:

```ruby
irb(main):008:0> b.average
[7, 15] in /myapp/app/models/beer.rb
     7|   def to_s
     8|     "#{name} #{brewery.name}"
     9|   end
    10|
    11|   def average
=>  12|     binding.pry
    13|     ratings.map{ |r| r.score }.sum / ratings.count.to_f
    14|   end
    15| end
```

Evaluoidaan lausekkeen osat debuggerissa:

```ruby
(ruby) ratings.map{ |r| r.score }.sum
  Rating Load (2.5ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
0
(ruby) ratings.count.to_f
  Rating Count (2.1ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
0.0
```

Olemme siis jakamassa kokonaisluku nollaa luvulla nolla, katsotaan mikä laskuoperaation tulos on:

```ruby
(ruby) 0/0.0
NaN
```

eli estääksemme nollalla jakamisen, tulee metodin käsitellä tapaus erikseen:

```ruby
def average
  return 0 if ratings.empty?
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

Käytämme oneliner-if:iä ja kokoelman metodia <code>empty?</code> joka evaluoituu todeksi kokoelman ollessa tyhjä. Kyseessä on Rubymainen tapa toteuttaa tyhjyystarkastus, joka "javamaisesti" kirjotettuna olisi:

```ruby
def average
  if ratings.count == 0
    return 0
  end
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

Kutakin kieltä käytettäessä tulee kuitenkin mukautua kielen omaan tyyliin, varsinkin jos on mukana projekteissa joita ohjelmoi useampi ihminen.

Jos et ole jo rutinoitunut debuggerin käyttöön, kannattaa ehdottomasti kerrata viime viikon debuggeria käsittelevä materiaali.

## Rubocop: tyyli ratkaisee

Isommissa ohjelmistoprojekteissa on tapana sopia yhtenäisestä koodityylistä, eli esim. tavoista miten asioita nimetään, mihin aaltosulkeet sijoitetan, missä on välilyönti ja missä ei. Railsin konventiot määrittelevät jo jossain määrin koodin tyyliäkin, lähinnä luokkien ja metodien nimennän tasolla.

Otetaan nyt käyttöön [Rubocop](https://github.com/rubocop-hq/rubocop), jonka avulla voimme määritellä koodilemme tyylisäännöstön ja seurata, että pidättäydymme säännöstön mukaisessa koodissa. Rubocop on vastaavanlainen _staattisen analyysin_ työkalu kuin JavaScript-maailmassa käytetty [ESLint](https://eslint.org/) tai Javan [checkstyle](http://checkstyle.sourceforge.net/).

Rubocop asennetaan antamalla komentoriviltä komento

    gem install rubocop

Rubocopin tarkastama säännöstö määritellään projektin juureen sijoitettavassa tiedostossa _.rubocop.yml_. Luo tiedosto projektiisi (huomaa, että tiedoston nimen alussa on piste) ja kopioi sille sisältö [täältä](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/misc/.rubocop.yml)

Tiedoston määrittelemä säännöstö perustuu [Relaxed Ruby](https://relaxed.ruby.style/) -tyyliin, jota se tiukentaa muutamien sääntöjen osalta. Tiedostossa myös jätetään osa projektin tiedostoista tyylitarkastuksen ulkopuolelle.

Tyylitarkastus suoritetaan komentoriviltä komennolla _rubocop_.

Koodista löytyy melko paljon ongelmia, esim. seuraava:

<pre>
app/models/beer.rb:8:5: C: Layout/EmptyLineAfterGuardClause: Add empty line after guard clause.
    return 0 if ratings.empty?
    ^^^^^^^^^^^^^^^^^^^^^^^^^^
</pre>

Tiedoston _beer.rb_ rivillä 8 rikotaan sääntöä [Layout/EmptyLineAfterGuardClause](http://docs.rubocop.org/en/latest/cops_layout/#layoutemptylineafterguardclause).

Sääntöjen [dokumentaatio](http://docs.rubocop.org/en/latest/cops/) selvittää mistä on kyse, eli nyt ongelmana on se, että äsken määrittelemässämme metodissa _average_ ensimmäisen koodirivin, joka on ns. _guard clause_, jälkeen ei ole tyhjää riviä:

```ruby
def average
  return 0 if ratings.empty?
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

Seuraava virhe

<pre>
app/models/concerns/rating_average.rb:9:38: C: Layout/SpaceAroundOperators: Surrounding space missing for operator +.
    ratings.reduce(0.0){ |sum, r| sum+r.score } / ratings.count    
                                     ^
</pre>

taas rikkoo sääntöä, jonka mukaan [matemaattisen operaattorin vasemmalla ja oikealla puolella on oltava välilyönti](http://docs.rubocop.org/en/latest/cops_layout/#layoutspacearoundoperators)

Ongelmistamme monet liittyvätkin ylimääräisiin tai puuttuviin välilyönteihin tai rivinvaihtoihin:

<pre>
app/models/concerns/rating_average.rb:10:6: C: Layout/TrailingWhitespace: Trailing whitespace detected.
  end
     ^^
app/models/concerns/rating_average.rb:11:1: C: Layout/EmptyLinesAroundModuleBody: Extra empty line detected at module body end.
app/models/concerns/rating_average.rb:12:4: C: Layout/TrailingBlankLines: Final newline missing.
end

app/models/rating.rb:7:1: C: Layout/TrailingWhitespace: Trailing whitespace detected.
</pre>

> ## Tehtävä 1
>
> Korjaa koodistasi kaikki määrittelymme mukaiset tyylivirheet.
>
> HUOM: voit suorittaa tarkastuksen vain yksittäiselle tiedostolle tai hakemiston sisällölle. Esim. komento _rubocop app/models/beer.rb_ tekee tarkastuksen tiedostolle _beer.rb_
>
> HUOM2: jos ei suoraan ymmärrä mistä kussakin sääntörikkeessä on kyse, tarkasta asia [dokumentaatiosta](https://docs.rubocop.org/rubocop/)

> ## Tehtävä 2
>
> Ota käyttöön sääntö, joka estää yli 15 riviä pitkät metodit. Varmista, että _rubocop_ ilmoittaa, jos koodiisi tulee liian pitkä metodi.
>
> Löydät ohjeita säännön määrittelyyn [dokumentaation](https://docs.rubocop.org/rubocop/cops_metrics.html) Metrics-osuudesta.

Rubocop saattaa mainitaa raportissaan, että osa virheistä on automaattisesti korjattavissa:

```bash
31 files inspected, 19 offenses detected, 19 offenses autocorrectable
```

Näiden virheiden automaattinen korjaaminen onnistuu komennolla `rubocop -A`. 

Tästä lähtien kannattaa pitää huoli, että kaikki koodi mitä teet säilyy Rubocopin sääntöjen mukaisena. Voit halutessasi muokata konfiguroituja sääntöjä mielesi mukaiseksi.

## Käyttäjä ja sessio

Laajennetaan sovellusta seuraavaksi siten, että käyttäjien on mahdollista rekisteröidä itselleen järjestelmään käyttäjätunnus.
Tulemme hetken päästä muuttamaan toiminnallisuutta myös siten, että jokainen reittaus liittyy sovellukseen kirjautuneena olevaan käyttäjään:

![mvc-kuva](http://yuml.me/4abc9b51.png)

Tehdään käyttäjä ensin pelkän käyttäjätunnuksen omaavaksi olioksi ja lisätään myöhemmin käyttäjälle myös salasana.

Luodaan käyttäjää varten model, näkymä ja kontrolleri komennolla <code>rails g scaffold user username:string</code>

Uuden käyttäjän luominen tapahtuu Rails-konvention mukaan osoitteessa <code>users/new</code> olevalla lomakkeella. Olisi kuitenkin luontevampaa jos osoite olisi <code>signup</code>. Lisätään routes.rb:hen vaihtoehtoinen reitti

```ruby
get 'signup', to: 'users#new'
```

eli myös osoitteeseen signup tuleva HTTP GET -pyyntö käsitellään Users-kontrollerin metodin <code>new</code> avulla.

HTTP on tilaton protokolla, eli kaikki HTTP-protokollalla suoritetut pyynnöt ovat toisistaan riippumattomia. Jos Web-sovellukseen kuitenkin halutaan toteuttaa tila, esim. tieto kirjautuneesta käyttäjästä tai vaikkapa verkkokauppaan ostoskori, tulee jonkinlainen tieto websession "tilasta" välittää jollain tavalla jokaisen selaimen tekemän HTTP-kutsun mukana. Yleisin tapa tilatiedon välittämiseen ovat evästeet, ks. <http://en.wikipedia.org/wiki/HTTP_cookie>

Lyhyesti sanottuna evästeiden toimintaperiaate on seuraava: kun selaimella mennään jollekin sivustolle, voi palvelin lähettää vastauksessa selaimelle pyynnön evästeen tallettamisesta. Jatkossa selain liittää evästeen kaikkiin sivustolle kohdistuneisiin HTTP-pyyntöihin. Eväste on käytännössä pieni määrä dataa, ja palvelin voi käyttää evästeessä olevaa dataa haluamallaan tavalla evästeen omaavan selaimen tunnistamiseen.

Railsissa sovelluskehittäjän ei ole tarvetta työskennellä suoraan evästeiden kanssa, sillä Railsiin on toteutettu evästeiden avulla hieman korkeammalla abstraktiotasolla toimivat **sessiot** ks.
<http://guides.rubyonrails.org/action_controller_overview.html#session> joiden avulla sovellus voi "muistaa" tiettyyn selaimeen liittyviä asioita, esim. käyttäjän identiteetin, useiden HTTP-pyyntöjen ajan.

Kokeillaan ensin sessioiden käyttöä muistamaan käyttäjän viimeksi tekemä reittaus. Rails-sovelluksen koodissa HTTP-pyynnön tehneen käyttäjän (tai tarkemmin ottaen selaimen) sessioon pääsee käsiksi hashin kaltaisesti toimivan olion <code>session</code> kautta.

Talletetaan reittaus sessioon tekemällä seuraava lisäys reittauskontrolleriin:

```ruby
def create
  # otetaan luotu reittaus muuttujaan
  rating = Rating.create params.require(:rating).permit(:score, :beer_id)

  # talletetaan tehty reittaus sessioon
  session[:last_rating] = "#{rating.beer.name} #{rating.score} points"

  redirect_to ratings_path
end
```

jotta edellinen reittaus saadaan näkyviin kaikille sivuille, lisätään application layoutiin (eli tiedostoon app/views/layouts/application.html.erb) seuraava:

```erb
<% if session[:last_rating].nil? %>
  <p>no ratings given</p>
<% else %>
  <p>previous rating: <%= session[:last_rating] %></p>
<% end %>
```

Kokeillaan nyt sovellusta. Aluksi sessioon ei ole talletettu mitään ja <code>session[:last_rating]</code> on arvoltaan <code>nil</code> eli sivulla pitäisi lukea "no ratings given". Tehdään reittaus ja näemme että se tallentuu sessioon. Tehdään vielä uusi reittaus ja havaitsemme että se ylikirjoittaa sessiossa olevan tiedon.

Avaa nyt sovellus incognito-ikkunaan tai toisella selaimella. Huomaat, että toisessa selaimessa session arvo on <code>nil</code>. Eli sessio on selainkohtainen.

## Kirjautuminen

Ideana on toteuttaa kirjautuminen siten, että kirjautumisen yhteydessä talletetaan sessioon kirjautuvaa käyttäjää vastaavan <code>User</code>-olion </code>id</code>. Uloskirjautuessa sessio nollataan.

Huom: sessioon voi periaatteessa tallennella melkein mitä tahansa olioita, esim. kirjautunutta käyttäjää vastaava <code>User</code>-olio voitaisiin myös tallettaa sessioon. Hyvänä käytänteenä (ks. <http://guides.rubyonrails.org/security.html#session-guidelines>) on kuitenkin tallettaa sessioon mahdollisimman vähän tietoa (oletusarvoisesti Railsin sessioihin voidaan tallentaa korkeintaan 4kB tietoa), esim. juuri sen verran, että voidaan identifioida kirjautunut käyttäjä, johon liittyvät muut tiedot saadaan tarvittaessa haettua tietokannasta.

Tehdään nyt sovellukseen kirjautumisesta ja uloskirjautumisesta huolehtiva kontrolleri. Usein Railsissa on tapana noudattaa myös kirjautumisen toteuttamisessa RESTful-ideaa ja konvention mukaisia polkunimiä.

Voidaan ajatella, että kirjautumisen yhteydessä syntyy sessio, ja tätä voidaan pitää jossain mielessä samanlaisena "resurssina" kuin esim. olutta. Nimetäänkin kirjautumisesta huolehtiva kontrolleri <code>SessionsController</code>iksi

Sessio-resurssi kuitenkin poikkeaa esim. oluista siinä mielessä että tietyllä ajanhetkellä käyttäjä joko ei ole tai on kirjaantuneena. Sessioita ei siis ole yhden käyttäjän näkökulmasta oluiden tapaan useita vaan maksimissaan yksi. Kaikkien sessioiden listaa ei nyt reittien tasolla ole mielekästä olla ollenkaan olemassa kuten esim. oluiden tilanteessa on. Reitit kannattaakin kirjoittaa yksikössä ja tämä saadaan aikaan kun session retit luodaan routes.rb:hen komennolla <code>resource</code>:

    resource :session, only: [:new, :create, :destroy]

**HUOM: varmista että kirjoitat määrittelyn routes.rb:hen juuri ylläkuvatulla tavalla, eli <code>resource</code>, ei _resources_ niinkuin muiden polkujen määrittelyt on tehty.**

Kirjautumissivun osoite on nyt **session/new**. Osoitteeseen **session** tehty POST-kutsu suorittaa kirjautumisen, eli luo käyttäjälle session. Uloskirjautuminen tapahtuu tuhoamalla käyttäjän sessio eli tekemällä POST-delete kutsu osoitteeseen **session**.

Tehdään sessioista huolehtiva kontrolleri (tiedostoon app/controllers/sessions_controller.rb):

```ruby
class SessionsController < ApplicationController
  def new
    # renderöi kirjautumissivun
  end

  def create
    # haetaan usernamea vastaava käyttäjä tietokannasta
    user = User.find_by username: params[:username]
    # talletetaan sessioon kirjautuneen käyttäjän id (jos käyttäjä on olemassa)
    session[:user_id] = user.id if not user.nil?
    # uudelleen ohjataan käyttäjä omalle sivulleen
    redirect_to user
  end

  def destroy
    # nollataan sessio
    session[:user_id] = nil
    # uudelleenohjataan sovellus pääsivulle
    redirect_to :root
  end
end
```

Huomaa, että vaikka sessioiden reitit kirjoitetaan nyt yksikössä **session** ja **session/new**, on kontrollerin ja näkymien hakemiston kirjoitusasu kuitenkin Railsin normaalia monikkomuotoa noudattava.

Kirjautumissivun app/views/sessions/new.html.erb koodi on seuraavassa:

```erb
<h1>Sign in</h1>

<%= form_with url: session_path, method: :post do |form| %>
  <%= form.text_field :username %>
  <%= form.submit "Log in" %>
<% end %>
```

Toisin kuin reittauksille tekemämme formi (kertaa asia [viime viikolta](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#lomake-ja-post)), nyt tekemämme lomake ei perustu olioon ja lomake luodaan <code>form_with</code>-metodilla, ks. http://guides.rubyonrails.org/form_helpers.html#dealing-with-basic-forms

Lomakkeen lähettäminen siis aiheuttaa HTTP POST -pyynnön session_pathiin (huomaa yksikkömuoto!) eli osoitteeseen **session**.

Pyynnön käsittelevä metodi ottaa <code>params</code>-olioon talletetun käyttäjätunnuksen ja hakee sitä vastaavan käyttäjäolion kannasta ja tallettaa olion id:n sessioon jos käyttäjä on olemassa. Lopuksi käyttäjä _uudelleenohjataan_ (kertaa [viime viikolta](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#uudelleenohjaus) mitä uudelleenohjauksella tarkoitetaan) omalle sivulleen. Kontrollerin koodi vielä uudelleen seuraavassa:

```ruby
def create
  user = User.find_by username: params[:username]
  session[:user_id] = user.id if not user.nil?
  redirect_to user
end
```

Huom1: komento <code>redirect_to user</code> siis on lyhennysmerkintä seuraavalla <code>redirect_to user_path(user)</code>, ks. [viikko 1](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko1.md#kertausta-polkujen-ja-kontrollerien-niment%C3%A4konventiot).

Huom2: Rubyssa yhdistelmän <code>if not</code> sijaan voidaan käyttää myös komentoa <code>unless</code>, eli metodin toinen rivi oltaisiin voitu kirjoittaa muodossa

```ruby
  session[:user_id] = user.id unless user.nil?
```

Paras muoto komennolle on kuitenkin

```ruby
  session[:user_id] = user.id if user
```

Rubyssä nimittäin kaikki muut arvoit paitsi _nil_ ja _false_ tulkitaan todeksi. Eli nyt komento suoritetaan jos _user_ on jotain muuta kuin _nil_ ja se on täsmälleen haluamamme toiminto.

Lisätään application layoutiin seuraava koodi, joka lisää kirjautuneen käyttäjän nimen kaikille sivuille (edellisessä luvussa lisätyt sessioharjoittelukoodit voi samalla poistaa):

```erb
<% if not session[:user_id].nil? %>
  <p><%= User.find(session[:user_id]).username %> signed in</p>
<% end %>
```

menemällä osoitteeseen [/session/new](http://localhost:3000/session/new) voimme nyt kirjautua sovellukseen (olettaen, että sovellukseen on luotu käyttäjiä osoitteesta http://localhost:3000/signup). Uloskirjautuminen ei vielä toistaiseksi onnistu.

**HUOM:** jos saat virheilmoituksen <code>uninitialized constant SessionsController></code> **varmista että määrittelit reitit routes.rb:hen oikein, eli**

```ruby
  resource :session, only: [:new, :create, :destroy]
```

> ## Tehtävä 3
>
> Tee kaikki ylläesitetyt muutokset ja varmista, että kirjautuminen onnistuu (eli kirjautunut käyttäjä näytetään sivulla) olemassaolevalla käyttäjätunnuksella (jonka siis voit luoda osoitteessa [/signup](http://localhost:3000/signup) ). Vaikka uloskirjautuminen ei ole mahdollista, voit kirjautua uudella tunnuksella kirjautumisosoitteessa ja vanha kirjautuminen ylikirjoittuu.

## Kontrollerien ja näkymien apumetodi

Tietokantakyselyn tekeminen näkymän koodissa (kuten juuri teimme application layoutiin lisätyssä koodissa) on todella ruma tapa. Lisätään luokkaan <code>ApplicationController</code> seuraava metodi:

```ruby
class ApplicationController < ActionController::Base
  # määritellään, että metodi current_user tulee käyttöön myös näkymissä
  helper_method :current_user

  def current_user
    return nil if session[:user_id].nil?
    User.find(session[:user_id])
  end
end
```

Koska kaikki sovelluksen kontrollerit perivät luokan <code>ApplicationController</code>, on määrittelemämme metodi kaikkien kontrollereiden käytössä. Määrittelimme lisäksi metodin <code>current_user</code> ns. helper-metodiksi, joten se tulee kontrollerien lisäksi myös kaikkien näkymien käyttöön. Voimme nyt muuttaa application layoutiin lisätyn koodin seuraavaan muotoon:

```erb
<% if not current_user.nil? %>
  <p><%= current_user.username %> signed in</p>
<% end %>
```

Voimme muotoilla ehdon myös tyylikkäämmin:

```erb
<% if current_user %>
  <p><%= current_user.username %> signed in</p>
<% end %>
```

Pelkkä <code>current_user</code> toimii ehtona, sillä arvo <code>nil</code> tulkitaan Rubyssä epätodeksi.

Kirjautumisen osoite **sessions/new** on hieman ikävä. Määritelläänkin kirjautumista varten luontevampi vaihtoehtoinen osoite **signin**. Määritellään myös reitti uloskirjautumiselle. Lisätään siis seuraavat routes.rb:hen:

```ruby
get 'signin', to: 'sessions#new'
delete 'signout', to: 'sessions#destroy'
```

eli sisäänkirjautumislomake on nyt osoitteessa [/signin](http://localhost:3000/signin) ja uloskirjautuminen tapahtuu osoitteeseen _signout_ tehtävän _HTTP DELETE_ -pyynnön avulla.

Olisi periaatteessa ollut mahdollista määritellä myös

```ruby
get 'signout', to: 'sessions#destroy'
```

eli mahdollistaa uloskirjautuminen HTTP GET:in avulla. Ei kuitenkaan pidetä hyvänä käytänteenä, että HTTP GET -pyyntö tekee muutoksia sovelluksen tilaan ja pysyttäydytään edelleen REST-filosofian mukaisessa käytänteessä, jonka mukaan resurssin tuhoaminen tapahtuu HTTP DELETE -pyynnöllä. Tässä tapauksessa vaan resurssi on hieman laveammin tulkittava asia eli käyttäjän sisäänkirjautuminen.

> ## Tehtävä 4
>
> Muokkaa nyt sovelluksen application layoutissa olevaa navigaatiopalkkia siten, että palkkiin tulee näkyville sisään- ja uloskirjautumislinkit. Huomioi, että uloskirjautumislinkin yhteydessä on määriteltävä käytettäväksi HTTP-metodiksi DELETE. Railsin versiossa 7 linkeille ei enää ole suoraa tukea deleten käyttöön, [linkit saa kuitenkin käyttämään delete-metodia](https://github.com/heartcombo/devise/issues/5439#issuecomment-997927041)
>
> Edellisten lisäksi lisää palkkiin linkki kaikkien käyttäjien sivulle, sekä kirjautuneen käyttäjän nimi, joka toimii linkkinä käyttäjän omalle sivulle. Käyttäjän ollessa kirjaantuneena tulee palkissa olla myös linkki uuden oluen reittaukseen.
>
> Muistutus: näet järjestelmään määritellyt routet ja polkuapumetodit komentoriviltä komennolla <code>rails routes</code> tai menemällä mihin tahansa sovelluksen osoitteeseen, jota ei ole olemassa, esim. [http://localhost:3000/wrong](http://localhost:3000/wrong)

Tehtävän jälkeen sovelluksesi näyttää suunnilleen seuraavalta jos käyttäjä on kirjautuneena:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-1.png)

ja seuraavalta jos käyttäjä ei ole kirjautuneena (huomaa, että nyt näkyvillä on myös uuden käyttäjän rekisteröitymiseen tarkoitettu signup-linkki):

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-2.png)

## Reittaukset käyttäjälle

Muutetaan seuraavaksi sovellusta siten, että reittaus kuuluu kirjautuneena olevalle käyttäjälle, eli tämän vaiheen jälkeen olioiden suhteen tulisi näyttää seuraavalta:

![kuva](http://yuml.me/4abc9b51.png)

Modelien tasolla muutos kulkee tuttuja latuja:

```ruby
class User < ApplicationRecord
  has_many :ratings   # käyttäjällä on monta ratingia
end

class Rating < ApplicationRecord
  belongs_to :beer
  belongs_to :user   # rating kuuluu myös käyttäjään

  def to_s
    "#{beer.name} #{score}"
  end
end
```

Ratkaisu ei kuitenkaan tällaisenaan toimi. Yhteyden takia _ratings_-tietokantatauluun riveille tarvitaan vierasavaimeksi viite käyttäjän id:hen. Railsissa kaikki muutokset tietokantaan tehdään Ruby-koodia olevien _migraatioiden_ avulla. Luodaan nyt uuden sarakkeen lisäävä migraatio. Generoidaan ensin migraatiotiedosto komentoriviltä komennolla:

    rails g migration AddUserIdToRatings

Hakemistoon _db/migrate_ ilmestyy tiedosto, jonka sisältö on seuraava

```ruby
class AddUserIdToRatings < ActiveRecord::Migration[7.0]
  def change
  end
end
```

Huomaa, että hakemistossa on jo omat migraatiotiedostot kaikkia luotuja tietokantatauluja varten. Jokaiseen migraatioon sisällytetään tieto sekä tietokantaan tehtävästä muutoksesta että muutoksen mahdollisesta perumisesta. Jos migraatio on riittävän yksinkertainen, eli sellainen että Rails osaa päätellä suoritettavasta lisäyksestä myös sen peruvan operaation, riittää että migraatiossa on määriteltynä ainoastaan metodi <code>change</code>. Jos migraatio on monimutkaisempi, on määriteltävä metodit <code>up</code> ja <code>down</code> jotka määrittelevät erikseen migraation tekemisen ja sen perumisen.

Tällä kertaa tarvittava migraatio on yksinkertainen:

```ruby
class AddUserIdToRatings < ActiveRecord::Migration[7.0]
  def change
    add_column :ratings, :user_id, :integer
  end
end
```

Jotta migraation määrittelemä muutos tapahtuu, suoritetaan komentoriviltä tuttu komento <code>rails db:migrate</code>

Migraatiot ovat varsin laaja aihe ja harjoittelemme niitä vielä lisää myöhemmin kurssilla. Lisää migraatiosta löytyy osoitteesta http://guides.rubyonrails.org/migrations.html

Huomaamme nyt konsolista, että yhteys olioiden välillä toimii:

```ruby
> u = User.first
> u.ratings
  Rating Load (0.3ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
=> []
```

Toistaiseksi antamamme reittaukset eivät liity mihinkään käyttäjään:

```ruby
> r = Rating.first
> r.user
 => nil
>
```

Päätetään että laitetaan kaikkien olemassaolevien reittausten käyttäjäksi järjestelmään ensimmäisenä luotu käyttäjä:

```ruby
> u = User.first
> Rating.all.each{ |r| u.ratings << r }
>
```

**HUOM:** reittausten tekeminen käyttöliittymän kautta ei toistaiseksi toimi kunnolla, sillä näin luotuja uusia reittauksia ei vielä liitetä mihinkään käyttäjään. Korjaamme tilanteen pian.

> ## Tehtävä 5
>
> Lisää käyttäjän sivulle eli näkymään app/views/users/show.html.erb
>
> - käyttäjän reittausten määrä ja keskiarvo (huom: käytä edellisellä viikolla määriteltyä moduulia <code>RatingAverage</code>, jotta saat keskiarvon laskevan koodin käyttäjälle!)
> - lista käyttäjän reittauksista ja mahdollisuus poistaa reittauksia
>
> Tee partialin sijaan muutokset suoraan tiedostoon app/views/users/show.html.erb, poista partialin kutsu kokonaan ko. tiedostosta!

Käyttäjän sivu siis näyttää suunilleen seuraavalta:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-3.png)

Uusien reittausten luominen www-sivulta ei siis tällä hetkellä toimi, koska reittaukseen ei tällä hetkellä liitetä kirjautuneena olevaa käyttäjää. Muokataan siis reittauskontrolleria siten, että kirjautuneena oleva käyttäjä linkitetään luotavaan reittaukseen:

```ruby
def create
  rating = Rating.new params.require(:rating).permit(:score, :beer_id)
  rating.user = current_user
  rating.save
  redirect_to current_user
end
```

Huomaa, että <code>current_user</code> on luokkaan <code>ApplicationController</code> äsken lisäämämme metodi, joka palauttaa kirjautuneena olevan käyttäjän eli suorittaa koodin:

```ruby
User.find(session[:user_id])
```

Reittauksen luomisen jälkeen kontrolleri on laitettu uudelleenohjaamaan selain kirjautuneena olevan käyttäjän sivulle.

> ## Tehtävä 6
>
> Muuta sovellusta vielä siten, että kaikkien reittausten sivulla ei ole enää mahdollisuutta reittausten poistoon ja että reittauksen yhteydessä näkyy reittauksen tekijän nimi, joka myös toimii linkkinä reittaajan sivulle.

Kaikkien reittausten sivun tulisi siis näyttää edellisen tehtävän jälkeen seuraavalta:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-4.png)

## Kirjautumisen hienosäätöä

Tällä hetkellä sovellus käyttäytyy ikävästi, jos kirjautumista yritetään olemassaolemattomalla käyttäjänimellä.

Muutetaan sovellusta siten, että uudelleenohjataan käyttäjä takaisin kirjautumissivulle, jos kirjautuminen epäonnistuu. Eli muutetaan sessiokontrolleria seuraavasti:

```ruby
def create
  user = User.find_by username: params[:username]
  if user.nil?
    redirect_to signin_path
  else
    session[:user_id] = user.id
    redirect_to user
  end
end
```

muutetaan edellistä vielä siten, että lisätään käyttäjälle kirjautumisen epäonnistuessa, sekä onnistuessa näytettävät viestit:

```ruby
def create
  user = User.find_by username: params[:username]
  if user.nil?
    redirect_to signin_path, notice: "User #{params[:username]} does not exist!"
  else
    session[:user_id] = user.id
    redirect_to user, notice: "Welcome back!"
  end
end
```

Jotta viesti saadaan näkyville kirjautumissivulle, lisätään näkymään `app/views/sessions/new.html.erb` seuraava elementti:

```erb
<p style="color: red"><%= notice %></p>
```

Elementti on jo valmiina käyttäjän sivun templatessa (ellet vahingossa poistanut sitä), joten viesti toimii siellä.

Sivulla tarvittaessa näytettävät, seuraavaan HTTP-pyyntöön muistettavat eli uudelleenohjauksenkin yhteydessä toimivat viestit eli **flashit** on toteutettu Railssissa sessioiden avulla, ks. lisää http://guides.rubyonrails.org/action_controller_overview.html#the-flash

## Olioiden kenttien validointi

Sovelluksessamme on tällä hetkellä pieni ongelma: on mahdollista luoda useita käyttäjiä, joilla on sama käyttäjätunnus. User-kontrollerin metodissa <code>create</code> pitäisi siis tarkastaa, ettei <code>username</code> ole jo käytössä.

Railsiin on sisäänrakennettu monipuolinen mekanismi olioiden kenttien validointiin, ks http://guides.rubyonrails.org/active_record_validations.html

Käyttäjätunnuksen yksikäsitteisyyden validointi onkin helppoa, pieni lisäys User-luokkaan riittää:

```ruby
class User < ApplicationRecord
  include RatingAverage

  validates :username, uniqueness: true

  has_many :ratings
end
```

Jos nyt yritetään luoda uudelleen jo olemassaoleva käyttäjä, huomataan että Rails osaa generoida sopivan virheilmoituksen automaattisesti.

Rails (tarkemmin sanoen ActiveRecord) suorittaa oliolle määritellyt validoinnit juuri ennen kuin olio yritetään tallettaa tietokantaan esim. operaatioiden <code>create</code> tai <code>save</code> yhteydessä. Jos validointi epäonnistuu, oliota ei tallenneta.

Lisätään saman tien muitakin validointeja sovellukseemme. Lisätään käyttäjälle vaatimus, että käyttäjätunnuksen pituuden on oltava vähintään 3 merkkiä, eli lisätään User-luokkaan rivi:

```ruby
  validates :username, length: { minimum: 3 }
```

samaa attribuuttia koskevat validointisäännöt voidaan myös yhdistää, yhden <code>validates :attribuutti</code> -kutsun alle:

```ruby
class User < ApplicationRecord
  include RatingAverage

  validates :username, uniqueness: true,
                       length: { minimum: 3 }

  has_many :ratings
end
```

Railsin scaffold-generaattorilla luodut kontrollerit toimivat siis siten, että jos validointi onnistuu ja olio on tallentunut kantaan, uudelleenohjataan selain luodun olion sivulle. Jos taas validointi epäonnistuu, näytetään uudelleen olion luomisesta huolehtiva lomake ja renderöidään virheilmoitukset lomakkeen näyttävälle sivulle.

Mistä kontrolleri tietää, että validointi on epäonnistunut? Kuten mainittiin, validointi tapahtuu tietokantaan talletuksen yhteydessä. Jos kontrolleri tallettaa olion metodilla <code>save</code>, voi kontrolleri testata metodin paluuarvosta onko validointi onnistunut:

```ruby
@user = User.new(parametrit)
if @user.save
  # validointi onnistui, uudelleenohjaa selain halutulle sivulle
else
  # validointi epäonnistui, renderöi näkymätemplate :new
end
```

Scaffoldin generoima kontrolleri näyttää hieman monimutkaisemmalta:

```ruby
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save
      format.html { redirect_to user_url(@user), notice: "User was successfully created." }
      format.json { render :show, status: :created, location: @user }
    else
      format.html { render :new, status: :unprocessable_entity }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end
```

Ensinnäkin mistä tulee olion luonnissa parametrina käytettävä <code>user_params</code>? Huomaamme, että tiedoston alalaitaan on määritelty metodi

```ruby
def user_params
  params.require(:user).permit(:username)
end
```

eli metodin <code>create</code> ensimmäinen rivi on siis sama kuin

```ruby
@user = User.new(params.require(:user).permit(:username))
```

Entä mitä metodin päättävä <code>respond_to</code> tekee? Jos olion luonti tapahtuu normaalin lomakkeen kautta, eli selain odottaa takaisin HTML-muotoista vastausta, on toiminnallisuus oleellisesti seuraava:

```ruby
if @user.save
  redirect_to user_url(@user), notice: "User was successfully created."
else
  render :new, status: :unprocessable_entity 
end
```

eli suoritetaan komentoon (joka on oikeastaan metodi) <code>respond_to</code> liittyvässä koodilohkossa merkintään (joka on jälleen teknisesti ottaen metodikutsu) <code>format.html</code> liittyvä koodilohko. Jos taas käyttäjä-olion luova HTTP POST -kutsu olisi tehty siten, että vastausta odotettaisiin json-muodossa (näin tapahtuisi esim. jos pyyntö tehtäisiin toisesta palvelusta tai Web-sivulta JavaScriptillä), suoritettaisiin <code>format.json</code>:n liittyvä koodi. Syntaksi saattaa näyttää aluksi oudolta, mutta siihen tottuu pian.

Jatketaan sitten validointien parissa. Määritellään että oluen reittauksen tulee olla kokonaisluku väliltä 1-50:

```ruby
class Rating < ApplicationRecord
  belongs_to :beer
  belongs_to :user

  validates :score, numericality: { greater_than_or_equal_to: 1,
                                    less_than_or_equal_to: 50,
                                    only_integer: true }

  # ...
end
```

Jos luomme nyt virheellisen reittauksen, ei se talletu kantaan. Huomamme kuitenkin, että emme saa virheilmoitusta. Ongelmana on se, että loimme lomakkeen käsin ja se ei sisällä scaffoldingin yhteydessä automaattisesti generoituvien lomakkeiden tapaan virheraportointia ja että kontrolleri ei tarkista millään tavalla validoinnin onnistumista.

Muutetaan ensin reittaus-kontrollerin metodia <code>create</code> siten, että validoinnin epäonnistuessa se renderöi uudelleen reittauksen luomisesta huolehtivan lomakkeen:

```ruby
def create
  @rating = Rating.new params.require(:rating).permit(:score, :beer_id)
  @rating.user = current_user

  if @rating.save
    redirect_to user_path current_user
  else
    @beers = Beer.all
    render :new, status: :unprocessable_entity
  end
end
```

Metodissa luodaan siis ensin Rating-olio <code>new</code>:llä, eli sitä ei vielä talleteta tietokantaan. Tämän jälkeen suoritetaan tietokantaan tallennus metodilla <code>save</code>. Jos tallennuksen yhteydessä suoritettava olion validointi epäonnistuu, metodi palauttaa epätoden, ja olio ei tallennu kantaan. Tällöin renderöidään new-näkymätemplate. Näkymätemplaten renderöinti edellyttää, että oluiden lista on talletettu muuttujaan <code>@beers</code>. [Rails 7 ei renderöi erroreita näkymään](https://stackoverflow.com/questions/71751952/rails-7-signup-form-doesnt-show-error-messages), ellei palautamme myös symbolia :unprocessable_entity käyttäen HTTP-statuskoodia 422. Statuskoodeista voi lukea lisää esim. [wikipediasta](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) tai kuvien kanssa [täältä](https://http.cat/).

Kun nyt yritämme luoda virheellisen reittauksen, käyttäjä pysyy lomakkeen näyttävässä näkymässä (joka siis teknisesti ottaen renderöidään uudelleen POST-kutsun jälkeen). Virheilmoituksia ei kuitenkaan vielä näy.

Validoinnin epäonnistuessa Railsin validaattori tallettaa virheilmoitukset <code>@ratings</code> olion kenttään <code>@rating.errors</code>.

Muutetaan lomaketta siten, että lomake näyttää kentän <code>@rating.errors</code> arvon, jos kenttään on asetettu jotain:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
    <%= @rating.errors.inspect %>
  <% end %>

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```

Kun nyt luot virheellisen reittauksen, huomaat että virheen syy selviää kenttään <code>@rating.errors</code> talletetusta oliosta.

Otetaan sitten mallia esim. näkymätemplatesta views/users/\_form.html.erb ja muokataan lomakettamme (views/ratings/new.html.erb) seuraavasti:

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

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```

Validointivirheitä löytyessä, näkymätemplate renderöi nyt kaikki joukossa <code>@rating.errors.full_messages</code> olevat virheilmoitukset.

**Huom:** validoinnin epäonnistuessa ei siis suoriteta uudelleenohjausta (miksi se ei tässä tapauksessa toimi?), vaan renderöidään näkymätemplate, johon tavallisesti päädytään <code>new</code>-metodin suorituksen yhteydessä.

Apuja seuraaviin tehtäviin löytyy osoitteesta
http://guides.rubyonrails.org/active_record_validations.html ja https://apidock.com/rails/v4.2.7/ActiveModel/Validations/ClassMethods/validates

> ## Tehtävä 7
>
> Lisää ohjelmaasi seuraavat validoinnit
>
> - oluen ja panimon nimi on epätyhjä
> - panimon perustamisvuosi on kokonaisluku väliltä 1040-2022
> - käyttäjätunnuksen eli User-luokan attribuutin username pituus on vähintään 3 mutta enintään 30 merkkiä

Jos yrität luoda oluen tyhjällä nimellä, seurauksena on virheilmoitus:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-9.png)

Mistä tämä johtuu? Jos oluen luonti epäonnistuu validoinnissa tapahtuneen virheen takia, olutkontrollerin metodi <code>create</code> suorittaa else-haaran, eli renderöi uudelleen oluiden luomiseen käytettävän lomakkeen. Oluiden luomiseen käytettävä lomake käyttää muuttujaan <code>@styles</code> talletettua oluttyylien listaa lomakkeen generointiin. Virheilmoituksen syynä onkin se, että muuttujaa ei ole nyt alustettu (toisin kuin jos lomakkeeseen mennään kontrollerimetodin <code>new</code> kautta). Lomake olettaa myös, että muuttujaan <code>@breweries</code> on talletettu kaikkien panimoiden lista. Eli ongelma korjautuu jos alustamme muuttujat else-haarassa:

```ruby
def create
  @beer = Beer.new(beer_params)

  respond_to do |format|
    if @beer.save
      format.html { redirect_to beers_path, notice: 'Beer was successfully created.' }
      format.json { render :show, status: :created, location: @beer }
    else
      @breweries = Brewery.all
      @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
      format.html { render :new }
      format.json { render json: @beer.errors, status: :unprocessable_entity }
    end
  end
end
```

> ## Tehtävä 8
>
> ### tehtävän teko ei ole viikon jatkamisen kannalta välttämätöntä eli ei kannata juuttua tähän tehtävään. Voit tehdä tehtävän myös viikon muiden tehtävien jälkeen.
>
> Parannellaan tehtävän 7 validointia siten, että panimon perustamisvuoden täytyy olla kokonaisluku, jonka suuruus on vähintään 1040 ja korkeintaan menossa oleva vuosi. Vuosilukua ei siis saa kovakoodata.
>
> Huomaa, että seuraava ei toimi halutulla tavalla:
>
> validates :year, numericality: { less_than_or_equal_to: Time.now.year }
>
> Nyt käy siten, että <code>Time.now.year</code> evaluoidaan siinä vaiheessa kun ohjelma lataa luokan koodin. Jos esim. ohjelma käynnistetään vuoden 2021 lopussa, ei vuoden 2022 alussa voida rekisteröidä 2022 aloittanutta panimoa, sillä vuoden yläraja validoinnissa on ohjelman käynnistyshetkellä evaluoitunut 2021
>
> Eräs kelvollinen ratkaisutapa on oman validointimetodin määritteleminen http://guides.rubyonrails.org/active_record_validations.html#custom-methods

## Monen suhde moneen -yhteydet

Yhteen olueeseen liittyy monta reittausta, ja reittaus liittyy aina yhteen käyttäjään, eli olueeseen liittyy monta reittauksen tehnyttä käyttäjää. Vastaavasti käyttäjällä on monta reittausta ja reittaus liittyy yhteen olueeseen. Eli käyttäjään liittyy monta reitattua olutta. Oluiden ja käyttäjien välillä on siis **monen suhde moneen -yhteys**, jossa ratings-taulu toimii liitostaulun tavoin.

Saammekin tuotua tämän many to many -yhteyden kooditasolle helposti käyttämällä jo [edellisen viikon lopulta tuttua](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#olioiden-ep%C3%A4suora-yhteys) tapaa, eli **has_many through** -yhteyttä:

```ruby
class Beer < ApplicationRecord
  include RatingAverage

  belongs_to :brewery
  has_many :ratings, dependent: :destroy
  has_many :users, through: :ratings

  # ...
end

class User < ApplicationRecord
  include RatingAverage

  has_many :ratings
  has_many :beers, through: :ratings

  # ...
end
```

Ja monen suhde moneen -yhteys toimii käyttäjästä päin:

```ruby
User.first.beers
=> [#<Beer:0x00007fbe23b8a770
  id: 1,
  name: "Iso 3",
  style: "Lager",
  brewery_id: 1,
  created_at: Sun, 21 Aug 2022 15:25:05 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:25:05 UTC +00:00>,
 #<Beer:0x00007fbe23b8a608
  id: 1,
  # ...
```

ja oluesta päin:

```ruby
irb(main):007:0> Beer.first.users
   (0.2ms)  SELECT sqlite_version(*)
  Beer Load (2.3ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.0ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf15b47aa0
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>,
 #<User:0x00007faf15b4dc20
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>
]
```

Vaikuttaa ihan toimivalta, mutta tuntuu hieman kömpeltä viitata oluen reitanneisiin käyttäjiin nimellä <code>users</code>. Luontevampi viittaustapa oluen reitanneisiin käyttäjiin olisi kenties <code>raters</code>. Tämä onnistuu vaihtamalla yhteyden määrittelyä seuraavasti

```ruby
has_many :raters, through: :ratings, source: :user
```

Oletusarvoisesti `has_many` etsii liitettävää taulun nimeä ensimmäisen parametrinsa nimen perusteella. Koska <code>raters</code> ei ole nyt yhteyden kohteen nimi, on se määritelty erikseen \_source*-option avulla.

Yhteytemme uusi nimi toimii:

```ruby
irb(main):009:0> Beer.first.raters
   (0.2ms)  SELECT sqlite_version(*)
  Beer Load (2.2ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.0ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf160f7748
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>,
 #<User:0x00007faf160bad48
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>]
```

Koska sama käyttäjä voi tehdä useita reittauksia samasta oluesta, näkyy käyttäjä useaan kertaan oluen reittaajien joukossa. Jos haluamme yhden reittaajan näkymään ainoastaan kertaalleen, onnistuu tämä esim. seuraavasti:

```ruby
irb(main):010:0> Beer.first.raters.uniq
  Beer Load (1.7ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.2ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf15cfd020
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>]
irb(main):011:0>
```

On myös mahdollista määritellä, että oluen <code>raters</code> palauttaa oletusarvoisesti vain kertaalleen yksittäisen käyttäjän. Tämä onnistuisi asettamalla <code>has*many</code>-määreelle [rajoite](https://guides.rubyonrails.org/association_basics.html#scopes-for-has-many) \_distinct*, joka rajoittaa niiden olioiden joukkoa, jotka näytetään assosiaatioon liittyviksi siten, että samaa oliota ei näytetä kahteen kertaan:

```ruby
class Beer < ApplicationRecord
  #...

  has_many :raters, -> { distinct }, through: :ratings, source: :user

  #...
end
```

Lisää asiaa yhteyksien määrittelemisestä normaaleissa ja hieman monimutkaisemmissa tapauksissa löytyy sivulta https://guides.rubyonrails.org/association_basics.html

Huom: Railsissa on myös toinen tapa many to many -yhteyksien luomiseen <code>has_and_belongs_to_many</code> ks. http://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association jonka käyttö saattaa tulla kyseeseen jos liitostaulua ei tarvita mihinkään muuhun kuin yhteyden muodostamiseen.

Trendinä kuitenkin on, että metodin has_and_belongs_to_many sijaan käytetään (sen monien ongelmien takia) has_many through -yhdistelmää ja eksplisiittisesti määriteltyä yhteystaulua. Mm. Chad Fowler kehottaa kirjassaan [Rails recepies](http://pragprog.com/book/rr2/rails-recipes) välttämään has_and_belongs_to_many:n käyttöä, sama neuvo annetaan Obie Fernandezin autoritiivisessa teoksessa [Rails 5 Way](https://leanpub.com/tr5w)

> ## Tehtävät 9-10: Olutseurat
>
> ### Tämän ja seuraavan tehtävän tekeminen ei ole välttämätöntä viikon jatkamisen kannalta. Voit tehdä tämän tehtävän myös viikon muiden tehtävien jälkeen.
>
> Laajennetaan järjestelmää siten, että käyttäjillä on mahdollista olla eri _olutseurojen_ jäseninä.
>
> Luo scaffoldingia hyväksikäyttäen model <code>BeerClub</code>, jolla on attribuutit <code>name</code> (merkkijono) <code>founded</code> (kokonaisluku) ja <code>city</code> (merkkijono)
>
> Muodosta <code>BeerClub</code>in ja <code>User</code>ien välille monen suhde moneen -yhteys. Luo tätä varten liitostauluksi model <code>Membership</code>, jolla on attribuutteina vierasavaimet <code>User</code>- ja <code>BeerClub</code>-olioihin (eli <code>beer_club_id</code> ja <code>user_id</code>.Tämänkin modelin voit luoda scaffoldingilla.
>
> Voit toteuttaa tässä vaiheessa jäsenien liittämisen olutseuroihin esim. samalla tavalla kuten oluiden reittaus tapahtuu tällä hetkellä, eli lisäämällä navigointipalkkiin linkin "join a club", jonka avulla kirjautunut käyttäjä voidaan liittää johonkin listalla näytettävistä olutseuroista.
>
> Listaa olutseuran sivulla kaikki jäsenet ja vastaavasti henkilöiden sivulla kaikki olutseurat, joiden jäsen henkilö on. Lisää navigointipalkkiin linkki kaikkien olutseurojen listalle.
>
> Tässä vaiheessa ei ole vielä tarvetta toteuttaa toiminnallisuutta, jonka avulla käyttäjän voi poistaa olutseurasta.
>
> Tässä tehtävässä joutuu olemaan tarkkana Railsin nimentäkäytänteiden suhteen. Olutseuran määrittelemä luokka kirjoitetaan BeerClub, sitä vastaava vierasavain taas beer_club_id ja muissa olioissa, esim. Membership:eissä olutseuraan viitataan muodossa beer_club

> ## Tehtävä 11
>
> Hio edellisessä tehävässä toteuttamaasi toiminnallisuutta siten, että käyttäjä ei voi liittyä useampaan kertaan samaan olutseuraan.
>
> Tämän tehtävän tekemiseen on monia tapoja, validointien käyttö ei ole välttämättä järkevin tapa tehtävän toteuttamiseen. Liittymislomakkeella tuskin kannattaa edes tarjota sellasia seuroja joiden jäsenenä käyttäjä jo on.

Seuraavat kaksi kuvaa antavat suuntaviivoja sille miltä sovelluksesi voi näyttää tehtävien 9-11 jälkeen.

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-5.png)

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-6.png)

## Salasana

Muutetaan sovellusta vielä siten, että käyttäjillä on myös salasana. Tietoturvasyistä salasanaa ei tule missään tapauksessa tallentaa tietokantaan. Kantaan talletetaan ainoastaan salasanasta yhdensuuntaisella funktiolla laskettu tiiviste. Tehdään tätä varten migraatio:

    rails g migration AddPasswordDigestToUser

migraation (ks. hakemisto db/migrate) koodiksi tulee seuraava:

```ruby
class AddPasswordDigestToUser < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :password_digest, :string
  end
end
```

huomaa, että lisättävän sarakkeen nimen on oltava <code>password_digest</code>.

Tehdään seuraava lisäys luokkaan <code>User</code>:

```ruby
class User < ApplicationRecord
  include RatingAverage

  has_secure_password

  # ...
end
```

<code>has_secure_password</code> (ks. http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) lisää luokalle toiminnallisuuden, jonka avulla *salasanan tiiviste* talletetaan kantaan ja käyttäjä voidaan tarpeen vaatiessa autentikoida.

Rails käyttää tiivisteen tallettamiseen <code>bcrypt-ruby</code> gemiä. Otetaan se käyttöön lisäämällä Gemfile:en rivi

    gem 'bcrypt', '~> 3.1.7'

Tämän jälkeen annetaan komentoriviltä komento <code>bundle install</code> jotta gem asentuu.

Kokeillaan nyt hieman uutta toiminnallisuutta konsolista. Uudelleenkäynnistä konsoli, jotta se saa käyttöönsä uuden gemin. Myös Rails-sovellus kannattaa tässä vaiheessa uudelleenkäynnistää. Muista myös suorittaa migraatio!

Salasanatoiminnallisuus <code>has_secure_password</code> lisää oliolle attribuutit <code>password</code> ja <code>password_confirmation</code>. Ideana on, että salasana ja se varmistettuna sijoitetaan näihin attribuutteihin. Kun olio talletetaan tietokantaan esim. metodin <code>save</code> kutsun yhteydessä, lasketaan tiiviste joka tallettuu tietokantaan olion sarakkeen <code>password_digest</code> arvoksi. Selväkielinen salasana eli attribuutti <code>password</code> ei siis tallennu tietokantaan, vaan on ainoastaan olion muistissa olevassa representaatiossa.

Talletetaan käyttäjälle salasana:

```ruby
> u = User.first
> u.password = "salainen"
> u.password_confirmation = "salainen"
> u.save
  TRANSACTION (0.1ms)  begin transaction
  User Exists? (2.9ms)  SELECT 1 AS one FROM "users" WHERE "users"."username" = ? AND "users"."id" != ? LIMIT ?  [["username", "mluukkai"], ["id", 1], ["LIMIT", 1]]
  User Update (15.3ms)  UPDATE "users" SET "updated_at" = ?, "password_digest" = ? WHERE "users"."id" = ?  [["updated_at", "2022-08-25 11:41:12.367244"], ["password_digest", "[FILTERED]"], ["id", 1]]
  TRANSACTION (10.8ms)  commit transaction
=> true
```

Autentikointi tapahtuu <code>User</code>-olioille lisätyn metodin <code>authenticate</code> avulla seuraavasti:

```ruby
>  u.authenticate "salainen"
=>
#<User:0x00007f320cdbba38
 id: 1,
 username: "mluukkai",
 created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
 updated_at: Thu, 25 Aug 2022 11:41:12.367244000 UTC +00:00,
 password_digest: "[FILTERED]">
irb(main):006:0>
```

eli metodi <code>authenticate</code> palauttaa <code>false</code>, jos sille parametrina annettu salasana on väärä. Jos salasana on oikea, palauttaa metodi olion itsensä.

Lisätään nyt kirjautumiseen salasanan tarkistus. Muutetaan ensin kirjautumissivua (app/views/sessions/new.html.erb) siten että käyttäjätunnuksen lisäksi pyydetään salasanaa (huomaa että lomakkeen kentän tyyppi on nyt _password_field_, joka näyttää kirjoitetun salasanan sijasta ruudulla ainoastaan tähtiä):

```erb
<h1>Sign in</h1>

<p id="notice"><%= notice %></p>

<%= form_with url: session_path, method: :post do |form| %>
  username <%= form.text_field :username %>
  password <%= form.password_field :password %>
  <%= form.submit "Log in" %>
<% end %>
```

ja muutetaan sessions-kontrolleria siten, että se varmistaa metodia <code>authenticate</code> käyttäen, että lomakkeelta on annettu oikea salasana.

```ruby
def create
  user = User.find_by username: params[:username]
  # tarkastetaan että käyttäjä olemassa, ja että salasana on oikea
  if user && user.authenticate(params[:password])
    session[:user_id] = user.id
    redirect_to user_path(user), notice: "Welcome back!"
  else
    redirect_to signin_path, notice: "Username and/or password mismatch"
  end
end
```

Kokeillaan toimiiko kirjautuminen (**huom: jotta bcrypt-gem tulisi sovelluksen käyttöön, käynnistä Rails server uudelleen**). Kirjautuminen onnistuu toistaiseksi vain niiden käyttäjien tunnuksilla joihin olet lisännyt salasanan konsolista käsin.

Lisätään vielä uuden käyttäjän luomiseen (eli näkymään view/users/\_form.html.erb) salasanan syöttökenttä:

```erb
<div>
  <%= form.label :password, style: "display: block"%>
  <%= form.password_field :password %>
</div>

<div>
  <%= form.label :password_confirmation, style: "display: block"%>
  <%= form.password_field :password_confirmation %>
</div>
```

Käyttäjien luomisesta huolehtivan kontrollerin apumetodia <code>user_params</code> on myös muutettava siten, että lomakkeelta lähetettyyn salasanaan ja sen varmenteeseen päästään käsiksi:

```erb
 def user_params
   params.require(:user).permit(:username, :password, :password_confirmation)
 end
```

Kokeile mitä tapahtuu, jos password confirmatioksi annetaan eri arvo kuin passwordiksi.

Huom: jos saat sisäänkirjautumisyrityksessä virheilmoitusen <code>BCrypt::Errors::InvalidHash</code> johtuu virhe melko varmasti siitä että käyttäjälle ei ole asetettu salasanaa. Eli aseta salasana konsolista ja yritä uudelleen.

> ## Tehtävä 12
>
> Tee luokalle User-validointi, joka varmistaa, että salasanan pituus on vähintää 4 merkkiä, ja että salasana sisältää vähintään yhden ison kirjaimen (voit unohtaa skandit) ja yhden numeron.

**Huom**: Säännöllisiä lausekkeita voi testailla Rubular sovelluksella: http://rubular.com/ Tehtävän tekeminen onnistuu toki muillakin tekniikoilla.

## Vain omien reittausten poisto

Tällä hetkellä kuka tahansa voi poistaa kenen tahansa reittauksia. Muutetaan sovellusta siten, että käyttäjä voi poistaa ainoastaan omia reittauksiaan. Tämä onnistuu helposti tarkastamalla asia reittauskontrollerissa:

```ruby
def destroy
  rating = Rating.find params[:id]
  rating.delete if current_user == rating.user
  redirect_to user_path(current_user)
end
```

eli tehdään poisto-operaatio ainoastaan, jos `current_user` on sama kuin reittaukseen liittyvä käyttäjä.

Reittauksen poistolinkkiä ei oikeastaan ole edes syytä näyttää muuta kuin kirjaantuneen käyttäjän omalla sivulla. Eli muutetaan käyttäjän show-sivua seuraavasti:

```erb
<ul>
  <% user.ratings.each do |rating| %>
    <li><%= "#{rating.to_s}" %>
      <% if @user == current_user %>
        <%= link_to "Delete", rating, data: {turbo_method: :delete} %>
      <% end %>
    </li>
  <% end %>
</ul>
```

Huomaa, että pelkkä **delete**-linkin poistaminen ei estä poistamasta muiden käyttäjien tekemiä reittauksia, sillä on erittäin helppoa tehdä HTTP DELETE -operaatio mielivaltaisen reittauksen urliin. Tämän takia on oleellista tehdä kirjaantuneen käyttäjän tarkistus poistamisen suorittavassa kontrollerimetodissa.

> ## Tehtävä 13
>
> Jokaisen käyttäjän omalla sivulla [http://localhost:3000/user/1](http://localhost:3000/user/1) on nyt painike **destroy this user**, jonka avulla käyttäjän voi tuhota, sekä linkki **edit** käyttäjän tietojen muuttamista varten.
>
> Näytä editointi- ja tuhoamislinkki vain kirjautuneen käyttäjän itsensä sivulla. Muuta myös User-kontrollerin metodeja <code>update</code> ja <code>destroy</code> siten, että olion tietojen muutosta tai poistoa ei voi tehdä kuin kirjaantuneena oleva käyttäjä itselleen.

> ## Tehtävä 14
>
> Luo uusi käyttäjätunnus, kirjaudu käyttäjänä ja tuhoa käyttäjä. Käyttäjätunnuksen tuhoamisesta seuraa ikävä virhe. **Pääset virheestä eroon tuhoamalla selaimesta cookiet.** Mieti mistä virhe johtuu ja korjaa asia myös sovelluksesta siten, että käyttäjän tuhoamisen jälkeen sovellus ei joudu virhetilanteeseen.

> ## Tehtävä 15
>
> Laajenna vielä sovellusta siten, että käyttäjän tuhoutuessa käyttäjän tekemät reittaukset tuhoutuvat automaattisesti. Ks. https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#orvot-oliot
>
> Jos teit tehtävät 9-11 eli toteutit järjestelmään olutkerhot, tuhoa käyttäjän tuhoamisen yhteydessä myös käyttäjän jäsenyydet olutkerhoissa

## Lisää hienosäätöä

Käyttäjän editointitoiminto mahdollistaa nyt myös käyttäjän <code>username</code>:n muuttamisen. Tämä ei ole ollenkaan järkevää. Poistetaan tämä mahdollisuus.

Uuden käyttäjän luominen ja käyttäjän editoiminen käyttävät molemmat samaa, tiedostossa views/users/\_form.html.erb määriteltyä lomaketta. Myös scaffoldin generoimat formit ovat Railsissa partiaaleja, joita liitetään muihin templateihin <code>render</code>-kutsun avulla.

Käyttäjän editointiin tarkoitettu näkymätemplate on seuraavassa:

```erb
<h1>Editing user</h1>

<%= render 'form' %>

<%= link_to "Show this user", @user %> |
<%= link_to "Back to users", users_path %>
```

eli ensin se renderöi \_form-templatessa olevat elementit ja sen jälkeen pari linkkiä. Lomakkeen koodi on seuraava:

```erb
<%= form_with(model: user) do |form| %>
  <% if user.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(user.errors.count, "error") %> prohibited this user from being saved:</h2>

      <ul>
        <% user.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :username, style: "display: block" %>
    <%= form.text_field :username %>
  </div>

  <div>
    <%= form.label :password, style: "display: block"%>
    <%= form.password_field :password %>
  </div>

  <div>
    <%= form.label :password_confirmation, style: "display: block"%>
    <%= form.password_field :password_confirmation %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

Haluaisimme siis poistaa lomakkeesta seuraavat

```erb
<div>
  <%= form.label :username, style: "display: block" %>
  <%= form.text_field :username %>
</div>
```

_jos_ käyttäjän tietoja ollaan editoimassa, eli käyttäjäolio on jo luotu aiemmin.

Lomake voi kysyä oliolta <code>@user</code> onko se vielä tietokantaan tallentamaton metodin <code>new_record?</code> avulla. Näin saadaan <code>username</code>-kenttä näkyville lomakkeeseen ainoastaan silloin kuin kyseessä on uuden käyttäjän luominen:

```erb
<% if @user.new_record? %>
  <div>
    <%= form.label :username, style: "display: block" %>
    <%= form.text_field :username %>
  </div>
<% end %>
```

Nyt lomake on kunnossa, mutta käyttäjänimeä on edelleen mahdollista muuttaa lähettämällä HTTP POST -pyyntö suoraan palvelimelle siten, että mukana on uusi username.

Tehdään vielä User-kontrollerin <code>update</code>-metodiin tarkastus, joka estää käyttäjänimen muuttamisen:

```ruby
def update
  respond_to do |format|
    if user_params[:username].nil? and @user == current_user and @user.update(user_params)
      format.html { redirect_to @user, notice: 'User was successfully updated.' }
      format.json { head :no_content }
    else
      format.html { render action: 'edit' }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end
```

Muutosten jälkeen käyttäjän tietojen muuttamislomake näyttää seuraavalta:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-7.png)

> ## Tehtävä 16
>
> Ainoa käyttäjään liittyvä tieto on nyt salasana, joten muuta käyttäjän tietojen muuttamiseen tarkoitettua lomaketta siten, että se näyttää allaolevassa kuvassa olevalta. Huomaa, että uuden käyttäjän rekisteröitymisen (signup) on edelleen näytettävä samalta kuin ennen.

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-8.png)

## Sovellus internetiin

Viikon lopuksi on taas aika deployata sovellus Fly.io:n Herokuun. Deployment Fly.io:n onnistuu ehkä ongelmitta, sillä Fly.io suorittaa automaattisesti sovellukseen määritellyt tietokantamigraatiot. Herokun suhteen tilanne on toisin.

### Ongelmia Herokussa

Kun ohjelman päivitetty versio deployataan herokuun, törmätään jälleen ongelmiin. Kaikkien reittausten ja kaikkien käyttäjien sivu ja signup-linkki saavat aikaan tutun virheen:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w2-12.png)

Kuten [viime viikolla](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md#ongelmia-herokussa) jo totesimme, tulee ongelman syy selvittää herokun lokeista.

Kaikkien käyttäjien sivu aiheuttaa seuraavan virheen:

ActionView::Template::Error (PG::UndefinedTable: ERROR: relation "users" does not exist

eli tietokantataulua _users_ ei ole olemassa koska sovelluksen uusia migraatioita ei ole suoritettu herokussa. Ongelma korjaantuu suorittamalla migraatiot:

    heroku run rails db:migrate

Myös signup-sivu toimii migraatioiden suorittamisen jälkeen.

Reittausten sivun ongelma ei korjaantunut migraatioiden avulla ja syytä on etsittävä lokeista:

```ruby
2022-08-24T16:28:33.610096+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4] ActionView::Template::Error (undefined method `name' for nil:NilClass):
2022-08-24T16:28:33.610221+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     2:
2022-08-24T16:28:33.610225+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     3: <ul>
2022-08-24T16:28:33.610227+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     4:  <% @ratings.each do |rating| %>
2022-08-24T16:28:33.610229+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     5:    <li> <%= rating %> <%= link_to rating.user.username, rating.user %></li>
2022-08-24T16:28:33.610231+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     6:  <% end %>
2022-08-24T16:28:33.610232+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     7: </ul>
2022-08-24T16:28:33.610234+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     8:
2022-08-24T16:28:33.610239+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]
2022-08-24T16:28:33.610241+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4] app/models/rating.rb:10:in `to_s'
```

Syy on jälleen tuttu, eli näkymäkoodi yrittää kutsua metodia <code>username</code> nil-arvoiselle oliolle. Syyn täytyy olla <code>link_to</code> metodissa oleva parametri

```ruby
rating.user.username
```

eli järjestelmässä on reittauksia joihin ei liity user-olioa.

Vaikka tietokantamigraatio on suoritettu, on osa järjestelmän datasta edelleen vanhan tietokantaskeeman mukaista. Tietokantamigraation yheyteen olisikin ollut järkevää kirjoittaa koodi, joka varmistaa että myös järjestelmän data saatetaan migraation jälkeen sellaiseen muotoon, mitä koodi olettaa, eli että esim. jokaiseen olemassaolevaan reittaukseen liitetään joku käyttäjä tai käyttäjättömät reittaukset poistetaan.

Luodaan järjestelmään käyttäjä ja laitetaan Herokun konsolista kaikkien olemassaolevien reittausten käyttäjäksi järjestelmään ensimmäisenä luotu käyttäjä:

```ruby
> u = User.first
> Rating.all.each{ |r| u.ratings << r }
```

Nyt sovellus toimii.

Toistetaan vielä viikon lopuksi edellisen viikon "ongelmia Herokussa"-luvun lopetus

<quote>
Useimmiten tuotannossa vastaan tulevat ongelmat johtuvat siitä, että tietokantaskeeman muutosten takia jotkut oliot ovat joutuneet epäkonsistenttiin tilaan, eli ne esim. viittaavat olioihin joita ei ole tai viitteet puuttuvat. *Sovellus kannattaakin deployata tuotantoon mahdollisimman usein*, näin tiedetään että mahdolliset ongelmat ovat juuri tehtyjen muutosten aiheuttamia ja korjaus on helpompaa.
</quote>

## Rubocop

Muista testata Rubocopilla, että koodisi noudattaa edelleen määriteltyjä tyylisääntöjä.

Jos käytät Visual Studio Codea, voit asentaa [ruby-rubocop](https://marketplace.visualstudio.com/items?itemName=misogi.ruby-rubocop) laajennuksen, jolloin editori huomauttaa heti jos teet koodiin tyylivirheen:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-10.png)

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi GitHubiin. Deployaa myös uusin versio Fly.io:n tai Herokuun.

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/stats/courses/rails2022/

[Viikko 4](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko4.md)
