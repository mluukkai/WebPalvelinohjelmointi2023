## perusteita

Ruby on dynaamisesti tyypitetty tulkattu erittäin olio-orientoitunut kieli.

Kieli on melko vapaamielinen ja monien asioiden tekemiseen on useita erilaisia tapoja. Tämä tekee joskus asiat hankalaksi aloittelijan kannalta

Koodi kirjoitetaan rb-päätteisiin tiedostoihin ja suoritetaan (normaalin Ruby-koodin ollessa kyseessä) komentoriviltä Ruby-komennon avulla.

Ruby-ohjelmoijan paras ystävä on interaktiivinen tulkki IRB joka käynnistyy komennolla irb. Railsin konsoli vastaa irb:tä, mutta sitä suoritetaan Rails-sovelluksen kontekstissa.

Rubyn harjoitteluun tarkoitettua materiaali [ensimmäinen osa](https://github.com/HY-TKTL/ruby-materiaali/blob/master/Perusteet.md) ja [toinen osa](https://github.com/HY-TKTL/ruby-materiaali/blob/master/LuokkiaJaOlioita.md)

Verkosta löytyy runsaasti Ruby-tutoriaaleja, ks. http://www.ruby-lang.org/en/

Heti alkuun kannattaa lukea
* http://www.ruby-lang.org/en/documentation/quickstart/

Hieman aikaavievämpi ja kattavampi, mutta kovin hyödyllinen Rubykoans löytyy seuraavasta
* http://rubykoans.com/


Hyväksi tai vähintään siedettäväksi havaittuja tutoriaaleja ovat mm:
* https://www.railstutorial.org/book/rails_flavored_ruby#sec-strings_and_methods
* http://rubylearning.com/satishtalim/tutorial.html
* http://ruby.learncodethehardway.org/book/
* http://www.techotopia.com/index.php/Ruby_Essentials

The Ruby-kirjan "Programming Ruby" ensimmäisen painoksen online-versio:
* http://ruby-doc.com/docs/ProgrammingRuby/

Jos aiot ostaa yhden kirjan Rubystä, erittäin suositeltava on Russ Olsenin __Eloquent Ruby__, ks http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104 Kirja ei ole tutoriaali mutta sopii hyvin jo ohjelmointitaustaa omaaville.

Seuraavassa listataan muutamia Railsin kannalta oleellisimpia asioita Rubystä

## hash ja symbolit

Seuraavassa irb-sessio joka luo hashin eli assosiatiivisen taulukon ja lisää sinne kahdella eri avaimella dataa. Avaimina käytetään Rubyn symboleja, eli :-alkuisia "vakion tapaan" käyttäytyviä merkkijonoja:

```ruby
irb(main):001:0> h = {}
=> {}
irb(main):002:0> h[:nimi] = "Pekka"
=> "Pekka"
irb(main):003:0> h[:ika] = 25
=> 25
irb(main):004:0> h
=> {:nimi=>"Pekka", :ika=>25}
irb(main):005:0> h[:ika]
=> 25
irb(main):006:0> h[:osoite]
=> nil
irb(main):007:0>
```

Seuraavassa luodaan hash joka saa sisältönsä luonnin yhteydessä:

```ruby
irb(main):010:0> h2 = { nimi:"Arto", ika:36 }
=> {:nimi=>"Arto", :ika=>36}
irb(main):011:0> h2[:nimi]
=> "Arto"
irb(main):012:0> h2.keys
=> [:nimi, :ika]
irb(main):013:0> h2.values
=> ["Arto", 36]
irb(main):014:0>
```

Edellisessä esimerkissä luotiin hash käyttäen Ruby 1.9:stä alkaen käytössä ollutta uutta syntaksia symbooliavaimisten hashien luomiseen. Seuraavassa vanhaa "hashrocket"-syntaksia käyttävä esimerkki:

```ruby
irb(main):014:0> h3 = { :nimi => "Esko", :osoite => "Westend" }
=> {:nimi=>"Esko", :osoite=>"Westend"}
```

lue seuraavasta linkistä lisää:
* http://nicholasjohnson.com/ruby/ruby-course/exercises/hashes-and-symbols/

## taulukko

Seuraavassa IRB-sessio, jossa luodaan taulukko, lisätään taulukkoon dataa parilla eri tavalla, aksessoidaan dataa  ja tulostetaan taulukon sisältö kahdella eri tavalla (erittäin harvoin Rubyssä käytetyllä) for-lauseella sekä rubymaisesti each-iteraattorin avulla:

```ruby
irb(main):015:0> t = []
=> []
irb(main):016:0> t[0] = "eka"
=> "eka"
irb(main):017:0> t << "toka"
=> ["eka", "toka"]
irb(main):018:0> t << 23
=> ["eka", "toka", 23]
irb(main):019:0> t
=> ["eka", "toka", 23]
irb(main):020:0> t.first
=> "eka"
irb(main):021:0> t.last
=> 23
irb(main):022:0> for alkio in t do
irb(main):023:1* puts alkio
irb(main):024:1> end
eka
toka
23
=> ["eka", "toka", 23]
irb(main):025:0> t.each { |alkio| puts alkio }
eka
toka
23
=> ["eka", "toka", 23]
irb(main):026:0>
```

lue seuraavasta linkistä lisää:
* http://www.techotopia.com/index.php/Understanding_Ruby_Arrays

## each

Vaikka Rubystä löytyvät tutut for- ja while-lauseet niitä käytetään erittäin harvoin. Erityisesti erilaisten kokoelmien, esim. taulukoiden läpikäynti hoidetaan käytännössä aina each-iteraattorin avulla.

each-iteraattorin toimintaperiaate on seuraava. Kutsuttaessa eachia taulukolle, iteraattori palauttaa taulukon alkiot yksi kerrallaan ja antaa ne nimetyn muuttujan avulla iteraattoria seuraavalle koodilohkolle. Esim. seuraavassa {}:lla määritellyssä lohkossa oleva muuttuja <code>alkio</code> saa arvokseen yksi kerrallaan kunkin taulukon <code>t</code> alkiosta:


```ruby
irb(main):027:0> t = [1,2,3,4,5]
 => [1, 2, 3, 4, 5]
irb(main):028:0> t.each { |alkio| puts alkio }
1
2
3
4
5
```

Rubyssä vaihtoehtoinen tapa määritellä koodilohko on seuraava

```ruby
t.each do |alkio|
 puts alkio
end
```

Yleensä käytäntönä on määritellä lohko aaltosulkeilla jos kyseessä on yhdelle riville mahtuva lohko, pitemmät lohkot taas määritellään yleensä do:n ja end:in avulla.

lisätietoja
* https://www.railstutorial.org/book/rails_flavored_ruby#sec-blocks

# merkkijonot

ks.
* https://www.rubyguides.com/2019/07/ruby-string-concatenation/

# moduuli

ks.

* http://juixe.com/techknow/index.php/2006/06/15/mixins-in-ruby/


