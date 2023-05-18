## Kisko afterparty pe 16.12. klo 16-18

Suomen johtava Rails-talo [Kisko](https://www.kiskolabs.com/) järjestää kurssilaisille illanvieton pe 16.12. klo 16-18

<img src="https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/kisko.png" width="600">

Jos haluat mukaan, kysy ilmoittautumislinkkiä matti.luukkainen@helsinki.fi tai Discordissa kurssikanavalla tai @mluukkai

## Web-sovellusten toimintaperiaatteita

Web-sovellusten toimintaperiaate on periaatteessa yksinkertainen. Käyttäjä avaa selaimen ja kirjoittaa osoiteriville haluamansa sivun URL:in, esim. https://www.cs.helsinki.fi/u/mluukkai/. URL:in ensimmäinen osa, eli esimerkissämme www.cs.helsinki.fi on yleensä DNS-nimi, jonka avulla pystytään selvittämään www-sivua hallinnoivan palvelimen ip-osoite. Selain lähettää web-palvelimelle pyynnön sivusta käyttäen HTTP-protokollan GET-metodia. Jos osoite on oikea, ja sivupyynnön lähettäjällä on oikeus URL:n polun määrittelemään resurssiin (esimerkissämme mluukkai/index.html), palvelin palauttaa selaimelle _statuskoodin_ 200 ja sivun sisällön HTML-muodossa. Selain renderöi sitten sivun käyttäjälle. Jos sivua ei ole olemassa, palvelin palauttaa selaimelle virheestä kertovan statuskoodin 404.

Palvelimen palauttama www-sivu voi olla **staattinen**, eli "käsin" palvelimella sijaitsevaan html-tiedostoon kirjoitettu tai **dynaaminen**, eli esim. palvelimen tietokannassa olevan datan perusteella pyynnön yhteydessä generoitu. Esim. [opintotarjonnan sivulla](https://studies.helsinki.fi/opintotarjonta?organisation=hy-org-116716376&page=0&period=hy-university-root-id%2F2022%2F0%2F1&searchText=&studyYear=2022) oleva kurssien lista luetaan tietokannasta ja sivun renderöivä html-koodi muodostetaan aina uudelleen sivulle mentäessä, senhetkisen tietokannassa olevan kurssien listan perusteella.

Toisinaan www-sivuilla tiedon kulun suunta muuttuu ja dataa lähetetään selaimelta palvelimelle. Useimmiten tämä tapahtuu siten, että sivustolla on _lomake_, jolle käyttäjä syöttää palvelimelle lähetettävät tiedot. Tietojen lähettämistä varten HTTP-protokolla tarjoaa metodin POST (myös HTTP:n GET-metodia voi käyttää tietojen lähettämiseen).

Esim. yliopiston www-sivujen yläkulmassa olevaa suurennuslasia painamalla avautuu lomake, jonka avulla sivun käyttäjä voi lähettää web-palvelimelle dataa. Kun käyttäjä kirjoittaa lomakkeelle tekstiä ja painaa enter, selain lähettää palvelimelle http://www.helsinki.fi POST-metodilla varustetun pyynnön, jonka mukana lähetetään käyttäjän hakulomakkeelle kirjoittama merkkijono. Palvelin vastaa lomakkeen lähetyksen yhteydessä tehtäviin POST-kutsuihin useimmiten palauttamalla uuden HTML-tiedoston, jonka selain sitten renderöi käyttäjälle. (Todellisuudessa POST-kutsuihin ei yleensä vastata palauttamalla html-sivua, vaan suoritetaan ns. uudelleenohjaus renderöitävän html-koodin sisältävälle sivulle ks. http://en.wikipedia.org/wiki/Post/Redirect/Get asiasta tarkemmin toisella viikolla)

HTTP-pyyntöihin ja vastauksiin liittyy osoitteen, datan (eli viestin rungon, engl. body) ja [statuskoodien](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes) lisäksi myös otsikoissa eli **headereissä** lähetettyä dataa (ks.
http://en.wikipedia.org/wiki/List_of_HTTP_header_fields), joiden avulla tarkennetaan pyyntöjä ja niihin liittyviä vastauksia, esim. määritellään minkä tyyppistä dataa selain on valmis vastaanottamaan.

Web-palvelinohjelmoinnilla tarkoitetaan juuri niitä toimia, miten web-palvelin muodostaa selaimelle näytettäviä web-sivuja ja käsittelee selaimen lomakkeen avulla lähettämää, käyttäjän syöttämää dataa.

Web-sivut eivät ole pelkkää html:ää. Html:nä pyritään kuvaamaan sivujen rakenne ja tietosisältö. Sivujen muotoilu on tapana hoitaa CSS-tyylitiedostojen avulla, ks. http://en.wikipedia.org/wiki/Cascading_Style_Sheets. Nykyisenä trendinä on myös sisällyttää www-sivuille yhä suurempi määrä **selaimessa** suoritettavaa ohjelmakoodia, joka taas on JavaScriptiä. On hieman veteen piirretty viiva, mitä toiminnallisuuksia kannattaa toteuttaa selaimen päässä ja mitä palvelimella.

Esim. jos www-sivu sisältää lomakkeen, jonka avulla suoritetaan sivustolle kirjautuminen, on selvää että salasanan ja käyttäjätunnuksen tarkastamisen täytyy tapahtua palvelimella. Sen sijaan selaimen päässä voidaan JavaScriptillä tehdä tarkistus onko käyttäjän salasanakenttä tyhjä kun käyttäjä yrittää kirjautua sivulle. Tälläisessä tilanteissa on turha vaivata palvelinta ollenkaan, sillä kirjautuminen ei tulisi kuitenkaan onnistumaan.

Viimeaikaisena trendinä on ollut pyrkiä saamaan web-sovellusten toiminta muistuttamaan mahdollisimman suuressa määrin normaalien työpöytäsovellusten toimintaa. Hyvänä esimerkkinä tälläisestä sovelluksesta on Google docs joka "matkii" mahdollisimman tarkasti wordin/openofficen toiminnallisuutta. Tälläisissä sovelluksissa sovelluslogiikasta suurin osa on toteutettu selaimessa. Palvelimessa toteutettua toiminnallisuutta tarvitaan kuitenkin aina, muutenhan tietoa ei voi jakaa sovellusta eri paikoissa käyttävien kesken.

Kun moderneissa sovelluksissa palvelimelta haetaan dataa, ei palvelin välttämättä palautakaan valmista HTML-sivua, vaan ainoastaan raakamuotoista dataa (yleensä json-muotoista), jonka selaimessa suoritettava JavaScript-koodi sitten sijoittaa käyttäjälle näytettävälle sivulle. Näin sivuista päivittyy ainoastaan tarpeellinen osa.

Tällä kurssilla keskitymme lähes yksinomaan web-sovellusten palvelinpuolen toiminnallisuuden toteuttamiseen. Viikoilla 6 ja 7 näemme muutaman esimerkin selaimen päässä JavaScriptillä toteutettavasta toiminnallisuudesta sekä sovelluksen ulkoasun muotoilusta CSS:n avulla.

Kurssilla [Full stack -websovellushehitys](https://fullstackopen.com/) taas keskitytään pääosin selaimen puolella toteutettavaan toiminnallisuuteen.

## Kurssin materiaalin ja tehtävät

Kurssilla kaikki tehtävät ovat upotettu tähän materiaaliin. Seuraavaa tehtävää lukuunottamatta kaikki tehtävät tullaan palauttamaan githubin kautta. github-palautusten lisäksi tehtävät merkataan tehdyksi tehtäväkirjanpitojärjestelmään. Tästä enemmän sivun lopussa. Aloitetaan kuitenkin nyt ensimmäisellä tehtävällä.

> ## Tehtävä 1: HTTP in action
>
> Erityisesti selainpuolen toiminnallisuuden toteuttamisessa selaimien developer-työkalut ovat erittäin tärkeä työskentelyväline. Selaimista kehittäjäystävällisin on Chrome, ja oletamme tässä että käytät Chromea. Vastaava toiminnallisuus löytyy muistakin selaimista.
>
> Avaa Chromen developer tool painamalla yhtä aikaa Shift, Control ja i (tai F12). Pääset developer tooliin myös valikon Tools-kautta. Avaa välilehti Network. Välilehti näyttää selaimen lähettämät HTTP-pyynnöt ja palvelimen niihin lähettämät vastaukset.
>
> Copypastea selaimen osoiteriville https://studies.cs.helsinki.fi/exampleapp/notes ja paina enter.
> Ylimpänä näet sivun lataamisen aiheuttaneen GET-pyynnön. Avaa se (klikkaamalla kutsua) ja tutki mitä kaikkea pyynnön mukana menee. Tutki erityisesti headereja ja response-osaa. Developer tools näyttää erikseen pyyntöön liittyvät (request headers) ja vastaukseen liittyvät (response headers) headerit.
>
> Pyyntö palauttaa siis välilehdellä response näytettävän HTML-koodin. Koodi sisältää viitteitä css-tyylitiedostoihin, JavaScript-tiedostoihin sekä kuviin. Sivua renderöitäessä selain hakee kunkin näistä omalla GET-pyynnöllä.
>
> Pidä edelleen sama networking-välilehti auki. Tyhjennä developer toolsin välilehti painamalla vasemman reunan halkaistu pallo -symbolia (:no_entry_sign:). Kirjoita jotain tekstikenttään ja paina "Save". Tekstikenttä on toteutettu html:n *lomakkeena* (engl. form). Lomakkeen tietojen lähetys palvelimelle tapahtuu HTTP-protokollan POST-metodin sisältävän pyynnön avulla.
>
> Tutki POST-pyynnön sisältöä (pyynnön nimi on new\_note). Välilehdeltä *Payload* löytyy *Form data*, eli lomakkeen mukana lähtetety tieto. Huomaat, että pyyntöön vastattiin statuskoodilla 302, joka taas tarkoittaa sitä, että palvelin tekee selaimelle **uudelleenohjauksen**, eli pyytää selainta menemään vastauksen headereissa ilmoittamaan osoitteeseen. POST-pyynnön vastaus ei siis sisällä ollenkaan HTML-koodia jonka selain voisi renderöidä käyttäjälle. Heti POST-kutsun perään selain tekeekin automaattisesti GET-kutsun POST:in vastauksen headerissa **Location** olevaan osoitteeseen. Vasta tämän uudelleenohjauksen aiheuttaman pyynnön vastauksena tullut sivu renderöidään käyttäjälle.
>
> Tutki vielä joillekin muille www-sivuille tekemisesi pyyntöjen aiheuttamaa HTTP-protokollan viestintää.

## Ruby on Railsin perusteita

Tällä kurssilla käytämme Web-sovellusten toteuttamiseen Ruby on Rails -sovelluskehystä.

Rails-sovellukset noudattavat [MVC-mallia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) (tai WebMVC:tä, joka poikkeaa hiukan alkuperäisestä MVC:stä), jossa ideana on jakaa sovelluksen data- ja sovelluslogiikka (Model), näyttöjen muodostaminen (View) ja toiminnan koordinointi (Controller) selkeästi eriytettyihin osiin. Valtaosa web-palvelinpuolen sovelluskehityksestä tapahtuu nykyään MVC-periaatetta noudattaen.

Tutkitaan mitä tapahtuu kun käyttäjä menee Railsilla toteutetulle web-sivulle, olkoon sivun URL esim. <https://ratebeer22.fly.dev/breweries>, eli kurssin aikana tekemämme esimerkkisovelluksen sivu, joka listaa kaikki esimerkkisovelluksen tuntemat panimot.

![mvc-kuva](http://www.cs.helsinki.fi/u/mluukkai/rails_mvc.png)

1. Käyttäjän kirjoitettua URL:n selaimen osoiteriville, tekee selain HTTP GET-pyynnön palvelimelle ratebeer22.herokuapp.com

2. Usein palvelimella suoritetaan web-palvelinohjelmistoa (esim. Apache tai Nginx), joka toimii välityspalvelimena ohjaten pyynnön osoitteeseen rekisteröityyn Rails-sovellukseen. Sovellus selvittää mikä sovelluksen _kontrolleri_ on rekisteröity huolehtimaan resurssiin breweries kohdistuvista GET-kutsista. Tätä vaihetta sanotaan Rails-sovelluksen sisäiseksi reititykseksi (routing), eli etsitään "reitti minkä varrella pyyntö käsitellään". On myös mahdollista ohittaa välityspalvelin, jolloin Rails-sovellus on ensimmäinen pyynnön vastaanottaja. Useasti kuitenkin välityspalvelinta käytetään kevyenä kerroksena mm. nopeuttamaan suorituskykyä ja jakelemaan pyyntöjä useammalle palvelimelle, joilla varsinainen Rails-sovellus on.

3. Kun oikea kontrolleri (esimerkissämme panimoista huolehtiva kontrolleri) ja sen pyynnöstä huolehtiva metodi selviää, kutsuu sovellus metodia ja antaa sille parametriksi HTTP-pyynnön mukana mahdollisesti tulleen datan. Kontrolleri hoitaa sitten operaatioon liittyvät toimenpiteet, yleensä toimenpiteiden suorittaminen edellyttää joihinkin sovelluksen dataa ja sovelluslogiikkaa sisältäviin _modeleihin_ tapahtuvaa metodikutsua.

4. Esimerkissämme kontrolleri pyytää panimoista huolehtivaa model-luokkaa lataamaan kaikkien panimoiden listan tietokannasta.

5. Saatuaan kaikkien panimoiden listan, kontrolleri pyytää panimoidem listan muodostavaa _näkymää_ renderöimään itsensä.

6. Näkymä renderöityy eli kontrolleri saa kaikki panimot listaavan HTML-sivun

7. Kontrolleri palauttaa HTML-sivun web-palvelimelle

8. ja web-palvelin palauttaa generoidun HTML-sivun ja siihen liittyvät headerit selaimelle.

MVC-mallissa modelit ovat useimmiten olioita, joiden tila talletetaan tietokantaan. Tietokannan käsittely on yleensä abstrahoitu siten, että ohjelmakoodin tasolla on harvoin tarve kirjoittaa SQL-kieltä tai tietokannan konfiguraatioita. Detaljit hoituvat [Object Relational Mapping (ORM)](https://en.wikipedia.org/wiki/Object-relational_mapping) -kirjaston avulla. Railsissa käytettävä ORM on nimeltään ActiveRecord, joka toimii hieman eri tavalla kuin joillekin ehkä Javamaailmasta tutut JPA-standardia noudattavat EclipseLink ja Hibernate.

Railsin taustalla on vahvana periaatteena **convention over configuration**, mikä tarkoittaa tapaa, jolla Rails pyrkii minimoimaan konfiguraatioiden tekemisen tarpeen määrittelemällä joukon konventioita, eli "sääntöjä", esim. tiedostojen nimennälle ja niiden sijainnille tiedostohierarkiassa. Tulemme pian näkemään mitä CoC-periaate tarkoittaa käytännössä sovellusohjelmoijan kannalta. Rails mahdollistaa toki konventiosta poikkeamisen, mutta siinä tapauksessa ohjelmoijan on jossain määrin konfiguroitava asioita käsin.

Railsilla sovellusten tekeminen edellyttää luonnollisesti jonkinasteista Rubyn hallintaa. Ruby on dynaamisesti tyypitetty tulkattu oliokieli, joka mahdollistaa myös funktionaalisen ohjelmointityylin. Ruby-koodia ei siis käännetä ollenkaan, vaan tulkki suorittaa koodia komento komennolta. Koska kääntäjää ei ole, ilmenevät myös monet koodiin tehdyt virheet vasta ajon aikana toisin kuin käännettävillä kielillä. Modernit kehitysympäristöt auttavat hiukan, tarjoten lähinnä lennossa tapahtuvaa "syntaksitarkastusta", mutta kehitysympäristön tuki ei ole läheskään samaa luokkaa kuin esim. Javalla.

> ## Tehtävä 2: Rubyn alkeet
>
> Tee/käy läpi seuraava
>
> - [http://www.ruby-lang.org/en/documentation/quickstart/](http://www.ruby-lang.org/en/documentation/quickstart/)
>
> Jos aikaa ja intoa riittää, voit käydä läpi myös osoitteessa
> [https://github.com/HY-TKTL/ruby-materiaali/blob/master/Perusteet.md](https://github.com/HY-TKTL/ruby-materiaali/blob/master/Perusteet.md) olevan materiaalin

## Komentorivi

Railsilla tapahtuvassa sovelluskehityksessä komentorivin käyttön hallinta on suhteellisen tärkeää. Jos rutiinisi komentorivin käyttöön on huono, kannattaa tehdän Ohjelmistotekniikka-kurssin [viikon 1 tehtävät](https://ohjelmistotekniikka-hy.github.io/python/viikko1)

## Kurssin suoritusmuoto

Kurssin rakenne poikkeaa jossain määrin Tietojenkäsittelytieteen osaston kurssistandardista. Kurssilla tehdään ainoastaan yksi sovellus, samaa sovellusta tehdään sekä kurssimateriaalissa että materiaalin sekaan upotetuissa tehtävissä. Kurssin materiaalia ei pystykään pelkästään lukemaan. Materiaalia seuratessa tulee itse rakentaa matkan varrella täydentyvää sovellusta, sillä muuten tehtävien tekeminen on mahdotonta. 

Kurssi jakautuu seitsemään "viikkoon", eli osaan jotka tehtiin kurssin aiemmissa versiossa yhden viikon aikana. Tällä kurssilla on kuitenkin ainoastaan yksi deadline, 31.12.2022 klo 23.59.

Jokainen "viikko" palateaan erikseen kurssin palautussovellukseen. Palautettuasi yhden viikon tehtävät, pääset näkemään viikon esimerkkivastauksen. Seuraavalla viikolla on mahdollista jatkaa joko oman sovelluksen rakentamista tai ottaa pohjaksi edellisen viikon esimerkkivastaus.

Osa viikon tehtävistä on käytännössä pakollisia, muuten eteneminen pysähtyy viikon osalta. Osa tehtävistä taas on vapaaehtoisia, ei-kriittisten ominaisuuksien toteutuksia. Osa näistä ominaisuuksista oletetaan olevan ohjelmistossa seuraavalla viikolla, joten jos et ole tehnyt kaikkia viikon tehtäviä, kannattaa aloittaa esimerkkivastauksesta tai vaihtoehtoisesti copypasteta sieltä tarvittavat asiat koodiisi.

## Railsin asennus

Asennusohje osoitteessa https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/railsin_asentaminen.md

## Sovelluksen luominen

Teemme kurssilla olutharrastajille tarkoitetun palvelun, jonka avulla olutharrastajat voivat selata olemassa olevia panimoja, oluita, oluttyylejä sekä "reitata" juomiaan oluita (eli antaa oluille oman mieltymyksensä mukaisen pistemäärän). Kurssin lopussa sovellus näyttää suunnilleen seuraavalta <https://ratebeer22.fly.dev/>

Rails tarjoaa sovelluskehittäjän avuksi useita generattoreita (ks. [http://guides.rubyonrails.org/generators.html](http://guides.rubyonrails.org/generators.html)), joiden avulla on helppo generoida hieman valmista toiminnallisuutta sisältäviä tiedostopohjia.

Uusi Rails-sovellus luodaan generaattorilla new. Mene sopivaan hakemistoon ja luo sinne sovellus nimeltään ratebeer antamalla komentoriviltä komento

```bash
rails new ratebeer
```

Komennon suoritus luo sovelluksen sisältämän hakemiston _ratebeer_.

Huom: jatkon kannalta on kätevintä, että luodusta hakemistosta tehdään git-repositorio. Älä siis sijoita sovellusta minkään muun git-repositorion sisälle!

Siirry hakemistoon antamalla komento _cd ratebeer_

Komennolla <code>tree</code> saat tuntumaa siitä, mitä kaikkea new-generaattorin suorittaminen sai aikaan. Huom: OSX:ssä ei ole oletusarvoisesti asennettuna tree-komentoa. Saat asennettua treen [homebrew:llä](http://brew.sh/) komennolla <code>brew install tree</code>

Seuraavassa hieman lyhennelty näkymä:

<pre>
.
|-- Gemfile
|-- Rakefile
|-- app
|   |-- assets
|   |-- controllers
|   |   |-- application_controller.rb
|   |   `-- concerns
|   |-- helpers
|   |   `-- application_helper.rb
|   |-- mailers
|   |-- models
|   |   `-- concerns
|   `-- views
|       `-- layouts
|           `-- application.html.erb
|-- bin
|-- config
|   |-- routes.rb
|-- db
|-- lib
|-- log
|-- public
|-- test
38 directories, 39 files
</pre>

Hakemistoista tärkein on sovelluksen ohjelmakoodin sisältävä **app**. Hakemiston **config** alla on erilaista sovelluksen konfigurointiin liittyvää dataa, mm. routes.rb, joka määrittelee miten sovellus käsittelee erilaiset sille kohdistuneet HTTP-pyynnöt. Tietokannan konfiguraatiot tulevat hakemistoon **db**. Gemfile taas määrittelee sovelluksen käyttämät kirjastot. Tulemme pikkuhiljaa tutustumaan sovelluksen hakemiston rakenteeseen tarkemmin.

Hakemistorakenne on tärkeä osa Railsin Convention over Configuration -periaatetta, jokaiselle komponentille (esim. panimoista huolehtivalle kontrollerille) on oma tarkasti määritelty paikkansa, josta Rails osaa etsiä komponentin ilman että sovelluskehittäjän tarvitsee erikseen kertoa Railsille missä hakemistossa ja tiedostossa komponentti sijaitsee.

Käynnistä sovellus antamalla komentoriviltä komento

```bash
rails server
```

Saman asian ajaa lyhennetty muoto _rails s_

Komento käynnistää oletusarvoisesti Puma HTTP-palvelimen (ks. https://github.com/puma/puma), joka alkaa suorittamaan hakemistossa olevaa Rails-sovellusta paikallisen koneen (eli localhost:in) portissa 3000.

Kokeile selaimella osoitteessa [http://localhost:3000](http://localhost:3000) että sovellus on käynnissä.

HUOM: **Tarkoituksena on, että materiaalia lukiessasi teet koko ajan samat asiat itse omaan sovellukseesi kuin mitä materiaalissa esimerkkisovellukselle tehdään**. Osa toteutettavista asioista on muotoiltu tehtäviksi, kuten seuraava kohta, ja osa askelista taas tulee tehdä, jotta materiaalissa eteneminen on ylipäätään mahdollista.

> ## Tehtävä 3
>
> Talletamme kurssilla tehtävän sovelluksen Githubissa sijaitsevaan repositorioon.
>
> Oletusarvoisesti komento _rails new_ myös luo sovelluksen hakemistosta uuden git-repositorion. Jos näin ei tapahtunut, tee sovelluksesi hakemistosta (eli siitä hakemistosta jonka _rails new_ loi) git-repositorio suorittamalla hakemistossa komento <code>git init</code>
>
> Luo sovellusta varten repositorio GitHubiin ja liitä se etärepositorioksi sovelluksesi hakemiston repositorioon
>
> Ohjeita gitin ja Githubin käyttöön esim. kurssin Tietokone työvälineenä materiaalissa https://tkt-lapio.github.io/ sekä Ohjelmistotekniikka-kurssin [viikon 1 tehtävissä](https://ohjelmistotekniikka-hy.github.io/python/viikko1)
>
> Tämän dokumentin lopussa on ohje varsinaisen palautuksen tekemiseksi

Aloitetaan sovelluksen rakentaminen. Päätetään aloittaa panimoista, eli:

- luodaan tietokantataulu panimoita varten
- tehdään toiminnallisuus, joka listaa kaikki panimot
- tehdään toiminnallisuus, joka mahdollistaa uuden panimon lisäyksen
- saamme myös kaupan päälle toiminnallisuuden panimon tietojen muuttamiseen ja panimon poistamiseen

Railsissa konventiona on, että (melkein) jokaista tietokantaan talletettavaa 'asiaa' varten sovelluksessa on oma model-luokka, kontrolleri-luokka sekä joukko omia näytön muodostavia tiedostoja.

Luodaan kaikki nämä Railsin valmista scaffold-generaattoria käyttäen. Panimolla on nimi (merkkijono) ja perustusvuosi (kokonaisluku). Annetaan komentoriviltä (sovelluksen sisältävästä hakemistosta) seuraava komento:

```bash
rails g scaffold brewery name:string year:integer
```

Syntyy joukko tiedostoja. Tärkeimmät näistä ovat

- app/models/brewery.rb
- app/controllers/breweries_controller.rb
- app/views/breweries/index.html.erb
- app/views/breweries/show.html.erb
- views-hakemistoon tulee näiden lisäksi muutama muukin tiedosto.

Railsin scaffold-generaattori luo siis kaikki tarvittavat tiedostopohjat nimettyinä ja sijoiteltuna Railsin konvention mukaisesti.

Loimme koodin generaattorilla <code>rails g scaffold brewery name:string year:integer</code>. Generaattorissa kirjoitimme luotavan asian, eli panimotietokantataulun ja siihen liittyvät asiat yksikössä (brewery). Railsin nimeämiskäytäntöjen mukaan tästä syntyy

- tietokantataulu nimeltään breweries
- kontrolleri nimeltään BreweriesController (tiedosto breweries_controller.rb)
- model eli yhtä olutpanimoa edustava luokka Brewery (tiedosto brewery.rb)

Alussa saattaa olla hieman sekavaa milloin ja missä käytetään yksikkö- ja milloin monikkomuotoa, miten tiedostot on nimetty ja mikä niiden sijainti on. Pikkuhiljaa kuitenkin käytänteet juurtuvat selkärankaan ja alkavat vaikuttamaan loogisilta.

Jos sovellus ei ole jo käynnissä, käynnistetään se uudelleen antamalla komentoriviltä komento <code>rails s</code>. Huom: sovelluksen uudelleenkäynnistys on Railsissa tarpeen melko harvoin. Esim. koodin muuttelu ja lisääminen ei aiheuta uudelleenkäynnistystarvetta.

Railsin konventioiden mukaan kaikkien oluiden lista näkyy osoitteessa breweries, eli mennään sivulle <http://localhost:3000/breweries>.

Tästä aiheutuu kuitenkin virheilmoitus:

```ruby
Migrations are pending; run 'bin/rails db:migrate RAILS_ENV=development' to resolve this issue.
```

Syynä virheelle on se, että panimot tallettavan tietokantataulun luomisesta huolehtiva _tietokantamigraatio_ on suorittamatta.

Scaffoldin suorittaminen luo hieman erikoisella tavalla nimetyn tiedoston

    db/migrate/20220808075536_create_breweries.rb

Kyseessä on ns. migraatiotiedosto, joka sisältää ohjeen breweries-tietokantataulun luomiseksi. Tietokantataulu saadaan luotua suorittamalla migraatio antamalla komentoriviltä (eli terminaalista) komento <code>rails db:migrate</code>:

```ruby
== 20220808075536 CreateBreweries: migrating ==================================
-- create_table(:breweries)
   -> 0.0216s
== 20220808075536 CreateBreweries: migrated (0.0220s) =========================
```

Panimot tallettava tietokantataulu on nyt luotu ja sovelluksen pitäisi toimia.

Refreshaa panimot näyttävä sivu [http://localhost:3000/breweries](http://localhost:3000/breweries) ja lisää sitä käyttäen tietokantaan nyt kolme panimoa.

Kuten huomaamme, on Railsin scaffoldingilla saatu jo melko paljon valmista toiminnallisuutta. Scaffoldingilla luotu toiminnallisuus on hyvä tapa päästä nopeasti alkuun. Mikään silver bullet scaffoldingit eivät kuitenkaan ole, sillä suurin osa scaffoldingeilla valmiiksi luodusta toiminnallisuudesta tullaan ajan myötä korvaamaan itse kirjoitetulla koodilla. Luomme kurssin aikana, viikosta 2 alkaen toiminnallisuutta myös kokonaan käsin, joten myös scaffoldingien automaattisesti generoima koodi tulee tutuksi.

**Huom:** Railsin generaattorin luomat tiedostot on mahdollista poistaa komennolla _destroy_:

```bash
rails destroy scaffold brewery
```

Jos olet suorittanut jo migraation ja huomaat että generaattorin luoma koodi onkin tuhottava, on **erittäin tärkeää** ensin perua migraatio komennolla

```bash
rails db:rollback
```

## Konsoli

Rails-sovelluskehittäjän yksi tärkeimmistä työkaluista on Rails-konsoli. Konsoli on interaktiivinen komentotulkki, joka on yhteydessä myös sovelluksen tietokantaan.

Avaa konsoli antamalla komentoriviltä (sovelluksen sisältävästä hakemistosta) komento

```bash
rails c
```

Tee kaikki seuraavat komennot myös itse (komento on merkin > jälkeen oleva merkkijono):

```ruby
irb(main):001:0> Brewery.all
   (4.2ms)  SELECT sqlite_version(*)
  Brewery Load (2.7ms)  SELECT "breweries".* FROM "breweries"
=>
[#<Brewery:0x00007f4b628be9a8
  id: 1,
  name: "Weihenstephan",
  year: 1040,
  created_at: Mon, 08 Aug 2022 14:39:47.235924000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 14:39:47.235924000 UTC +00:00>,
 #<Brewery:0x00007f4b62775d80
  id: 2,
  name: "Schlenkerla",
  year: 1682,
  created_at: Mon, 08 Aug 2022 14:40:11.159471000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 14:40:11.159471000 UTC +00:00>,
 #<Brewery:0x00007f4b62775b00
  id: 3,
  name: "Sinebrychoff",
  year: 1879,
  created_at: Mon, 08 Aug 2022 14:40:35.385354000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 14:40:35.385354000 UTC +00:00>]
irb(main):002:0> Brewery.count
irb(main):002:0> Brewery.count
   (0.2ms)  SELECT COUNT(*) FROM "breweries"
=> 3
irb(main):003:0> Brewery
=> Brewery(id: integer, name: string, year: integer, created_at: datetime, updated_at: datetime)
irb(main):004:0>
```

Komento <code>Brewery.all</code> siis näyttää kaikki tietokannassa olevat panimot. Konsoli näyttää ensin tietokantaoperaation aiheuttaman SQL-kyselyn ja sen jälkeen kannasta saatavat panimo-oliot. Komento <code>Brewery.count</code> näyttää kannassa olevien panimoiden määrän.

Yhteys breweries-tietokantatauluun siis saadaan luokan <code>Brewery</code> kautta. Railsin scaffold -generaattori loi luokan koodin valmiiksi.

Jos katsotaan miltä luokka (eli tiedosto app/models/brewery.rb) näyttää, huomaamme että se sisältää varsin niukasti koodia:

```ruby
class Brewery < ApplicationRecord
end
```

Kuten äskeinen konsolisessio paljasti, on luokalla kuitenkin metodit all ja count, nämä ja todella suuren määrän muita metodeja luokka saa **perimästään** luokasta <code>ApplicationRecord</code> joka taas perii toiminnallisuutensa luokasta <code>ActiveRecord</code>.

Rails-guiden (http://guides.rubyonrails.org/active_record_basics.html) sanoin:

<blockquote>
Active Record is the M in MVC - the model - which is the layer of the system responsible for representing business data and logic. Active Record facilitates the creation and use of business objects whose data requires persistent storage to a database. It is an implementation of the Active Record pattern (https://en.wikipedia.org/wiki/Active_record_pattern) which itself is a description of an Object Relational Mapping system.
</blockquote>

Periaatteena ActiveRecordissa on lyhyesti sanottuna se, että jokaista tietokannan taulua (esim. breweries) vastaa koodissa oma luokka (Brewery). Luokka tarjoaa **luokkametodeina** metodit, joiden avulla tietokantaa käsitellään. Kun tietokannasta haetaan rivillinen dataa (yhden panimon tiedot), luodaan siitä luokan instanssi (eli Brewery-olio).

ActiveRecordissa luokilla on siis kaksoisrooli, luokkametodien (joita Rubyssä kutsutaan luokan nimen kautta tyyliin <code>Brewery.all</code>) avulla hoidetaan suurin osa tietokantaoperaatioista, esim. tietokantakyselyt. Tietokantaan talletetut rivit taas mäppäytyvät ActiveRecord-luokkien instansseiksi.

Jatketaan konsolista tapahtuvia kokeiluja. Luodaan uusi panimo:

```ruby
Brewery.new(name: "Stadin Panimo", year: 1997)
```

Railsissa siis konstruktoria kutsutaan hieman eri tyyliin kuin esim. Javassa. Huomaa, että sulkujen käyttö konstruktori- tai metodikutsussa ei ole välttämätöntä, edellinen oltaisiinkin voitu antaa muodossa

```ruby
Brewery.new name: "Stadin Panimo", year: 1997
```

Listaa nyt panimot ja tarkista niiden lukumäärä metodeilla <code>Brewery.all</code> ja <code>Brewery.count</code>. Huomaat, että vaikka loimme uuden olion, ei se mene kuitenkaan tietokantaan!

Olio saadaan talletettua tietokantaan seuraavasti:

```ruby
b = Brewery.new name: "Stadin Panimo", year: 1997
b.save
```

Eli otettiin luotu olio talteen muuttujaan <code>b</code> ja kutsuttiin oliolle metodia <code>save</code>. Save on ActiveRecordilta peritty oliometodi, joka kuten arvata saattaa, tallettaa olion tietokantaan. Huomaa, että muuttujan tyyppiä ei tarvitse (eikä voi) määritellä sillä Ruby on dynaamisesti tyypitetty kieli.

Olion voi myös luoda ja tallettaa suoraan kantaan käyttämällä new:n sijaan luokan metodia create:

```ruby
Brewery.create name: "Weihenstephan", year: 1040
```

Kun olio luodaan komennolla <code>new</code>, huomaamme, että olio sisältää kenttiä joiden arvoa ei ole asetettu:

```ruby
irb(main):007 > b = Brewery.new(name: "Stadin Panimo", year: 1997)
 => #<Brewery id: nil, name: "Stadin Panimo", year: 1997, created_at: nil, updated_at: nil>
```

Kun olio sitten talletetaan, tulee näillekin kentille arvo:

```ruby
irb(main):008 > b.save
irb(main):009 > b
 => #<Brewery id: 4, name: "Stadin Panimo", year: 1997, created_at: "2022-08-09 13:21:37", updated_at: "2022-08-09 13:21:37">
```

Kuten arvata saattaa, oliomuuttujat eli olion kentät vastaavat tietokantataulun sarakkeita. Kun olio tallettuu tietokantaan, generoi kanta automaattisesti oliolle pääavaimena (engl. primary key) toimivan id:n sekä pari aikaleimaa. Id on siis juuri luodun oluen yksikäsitteinen tunnus.

Katso tilannetta taas [selaimesta](http://localhost:3000/breweries). Luotujen panimoiden pitäisi nyt olla näkyvissä sivulla.

Konstruktorin <code>new</code> ja metodien <code>create</code> kutsut näyttivät hieman erikoiselta

```ruby
Brewery.new name: "Stadin Panimo", year: 1997
```

Olemme tässä hyödyntäneet Rubyn vapaamielistä suhtautumista sulkujen käyttöön. Eli sulkujen kanssa kutsu näyttää seuraavalta:

```ruby
Brewery.new( name: "Stadin Panimo", year: 1997 )
```

Myös parametri on hieman erikoisessa formaatissa. Kyseessä on symboleilla indeksöity assosiatiivinen taulukko eli _hash_, ks. https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#hash-ja-symbolit

Kuten yo. linkistä selviää, hashit määritellään aaltosuluissa:

```ruby
{ name: "Stadin Panimo", year: 1997 }
```

Konstruktorin kutsun voisi siis kirjoittaa myös muodossa

```ruby
Brewery.new( { name: "Stadin Panimo", year: 1997 } )
```

Metodin/konstruktorin parametrina olevassa hashissa ei ole kuitankaan pakko käyttää aaltosulkuja kaikissa tapauksissa, joten usein ne jätetäänkin pois. Jos metodilla on useita parametreja, ovat aaltosulkeet joissain tilanteissa tarpeen.

Huom: Rubyssä on myös vaihtoehtoinen syntaksi hashien määrittelyyn, sitä käyttäen edellinen tulisi muodossa

```ruby
Brewery.new :name => "Stadin Panimo", :year => 1997
```

Jos haluat luoda tietueita tai muuten harjoitella Railsin käyttöä konsolista käsin ilman pysyviä muutoksia tietokantaan, voit suorittaa konsolin hiekkalaatikko-tilassa komennolla `rails console --sandbox` tai lyhyemmin `rails c -s`.

## ActiveRecordin hakurajapinta

ActiveRecord tarjoaa monipuoliset mahdollisuudet tietokantahakujen tekemiseen ohjelmallisesti eli SQL:ää kirjoittamatta, ks. http://guides.rubyonrails.org/active_record_querying.html

Seuraavassa muutamia esimerkkejä, kokeile kaikkia konsolista:

```ruby
Brewery.find 1                 # palauttaa olion, jonka id on 1

b = Brewery.find 2             # palauttaa olion, jonka id on 2 ja sijoittaa sen muuttujaan b
b.year                         # muuttujaan b talletetun olion kentän year arvo
b.name                         # muuttujaan b talletetun olion kentän name arvo

Brewery.find_by name: "Koff"    # palauttaa olion, jonka nimi on Koff

Brewery.where name: "Koff"      # palauttaa taulukon, johon on sijoitettu kaikki Koff-nimiset panimot

Brewery.where year: 1900..2000  # palauttaa taulukon, jossa vuosina 1900-2000 perustetut panimot

Brewery.where "year<1900"      # palauttaa taulukon, jossa ennen vuotta 1900 perustetut panimot

b = Brewery.where name: "Koff"
b.year                         # operaatio ei toimi, sillä where palauttaa taulukon, jossa Koff sijaitsee

t = Brewery.where name: "Koff"
t.first.year                   # t.first tarkoittaa kuin t[0] eli taulukon 1. alkiota
```

Lisää Rubyn taulukosta ks. https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#taulukko

Huomaa, että jätimme edellä kaikissa esimerkeissä metodikutsuista sulut pois. <code>Brewery.find 1</code> siis tarkoittaa samaa kuin <code>Brewery.find(1)</code>

## Alaviiva

Edellisen metodikutsun palauttamaan arvoon voi konsolissa viitata alaviivalla, eli merkillä <code>\_</code>, eli jos konsolissa työskennellessä unohtuu sijoittaa metodin tulos muuttujaan, saa tuloksen vielä kiinni alaviivalla:

```ruby
irb(main):013 > Brewery.where "year<1900"
```

tuloksen tallettaminen muuttujaan unohtui... käytetään alaviivaa

```ruby
irb(main):014 > vanhat = _
irb(main):015 > vanhat.count
 => 3
irb(main):016:0> vanhat.first
=>
#<Brewery:0x00007f4b622e1008
 id: 1,
 name: "Weihenstephan",
 year: 1040,
 created_at: Mon, 08 Aug 2022 14:39:47.235924000 UTC +00:00,
 updated_at: Mon, 08 Aug 2022 14:39:47.235924000 UTC +00:00>
irb(main):017 >
```

> ## Tehtävä 4
>
> Lue http://guides.rubyonrails.org/active_record_basics.html#crud-reading-and-writing-data
>
> Tee kaikki seuraavat Rails-konsolista:
>
> - Luo panimo nimeltä "Kumpulan panimo", jonka perustamisvuosi on 2012 <br />
> - Hae panimo kannasta <code>find_by</code>-metodilla nimen perusteella<br />
> - Muuta panimon perustamisvuodeksi 2018 <br />
> - Hae panimo kannasta uudelleen <code>find_by</code>:lla ja varmista että perustamisvuoden muutos tapahtui <br />
> - Tarkista myös, että panimon kentän <code>updated_at</code> arvo on muuttunut, eli ettei se ole enää sama kuin <code>created at</code> <br />
> - Tuhoa panimo <br />
> - Varmista, että panimo tuhoutui

Vilkaistaan vielä panimon koodia:

```ruby
class Brewery < ApplicationRecord
end
```

Pystymme siis koodista käsin pääsemään käsiksi panimo-olioiden kaikkiin kenttiin "pistenotaatiolla" ja voimme asettaa niille vastaavalla tavalla uudet arvot:

    b = Brewery.first         # hakee tietokannasta ensimmäiseksi lisätyn
    b.created_at              # näyttää luontihetken aikaleiman
    b.name = "Sinebrychoff"   # vaihtaa kentän arvoa, huom: muutos menee kantaan vasta olion talletuksen yhteydessä!

Taustalla on hieman Railsin magiaa, sillä Rails luo automaattisesti kaikille olioiden tietokantataulun sarakkeille setteri- ja getterimetodit, joiden nimi on täsmälleen sama kuin tietokannan sarake.

Kun sanomme konsolissa <code>b.created_at</code> suoritetaan siis todellisuudessa <code>Brewery</code>:lle automaattisesti lisätty <code>created_at</code>-metodi, joka palauttaa samannimisen kentän arvon. Vastaavasti komento <code>b.name = "Sinebrychoff"</code> aiheuttaa Brewerylle automaattisesti lisätyn kentän <code>name</code> arvoa muuttavan metodin <code>name=</code>-suorittamisen.

## Oluet ja yhden suhde moneen -yhteys

Laajennetaan sovellustamme seuraavaksi oluilla. Jokainen olut liittyy yhteen panimoon, ja panimoon luonnollisesti liittyy useita oluita. Laajennuksen jälkeen sovelluksemme domainin (eli bisneslogiikkaa sisältävien tietokantaan talletettavien olioiden) luokkamalli näyttää seuraavalta:

![Panimot ja oluet](http://yuml.me/4f643b44.png)

Luodaan oluita varten malli, kontrolleri ja valmiit näkymät Railsin scaffold-generaattorilla (komento annetaan komentoriviltä):

    rails g scaffold Beer name:string style:string brewery_id:integer

jotta saamme tietokannan päivitettyä, suoritetaan tietokantamigraatio antamalla komentoriviltä komento

    rails db:migrate

Nyt siis on luotu

- oluet tallettava tietokantataulu beers
- tietokantamäppäykseen käytettävä luokka Beer tiedostoon app/models/beer.rb
- oluista huolehtiva kontrolleri BeersController tiedostoon app/controllers/beers_controller.rb
- sekä näkymätiedostoja hakemistoon app/views/beers/

Loimme oluelle string-tyyppiset nimen ja tyylin tallettavat kentät <code>name</code> ja <code>style</code>. Loimme myös integer-tyyppisen kentän <code>brewery_id</code>, jonka tarkoitus on toimia **vierasavaimena** (engl. foreign key), jonka liittää oluen panimoon.

Tarvittaessa kentät voi tarkistaa kirjoittamalla tietokantataulua vastaavan luokan nimi Rails-konsoliin:

```ruby
irb(main):035:0> Beer
=> Beer(id: integer, name: string, style: string, brewery_id: integer, created_at: datetime, updated_at: datetime)
```

Jos et ole vielä hakenut oluita kannasta, edellinen komento ei välttämättä toimi. Suorita ensin esim. komento <code>Beer.count</code> ja tämän jälkeen myös pelkkä <code>Beer</code> toimii.

Oluella on siis luonnollisesti myös kaikille ActiveRecord-olioille automaattisesti lisättävät kentät eli <code>id</code>, <code>created_at</code> ja <code>updated_at</code>.

Luodaan konsolista käsin muutama olut ja liitetään ne panimoon vierasavaimen <code>brewery_id</code> avulla (huom: jos konsolisi oli jo auki, saatat joutua antamaan konsolissa komennon <code>reload!</code>, joka lataa oluiden ohjelmakoodin konsolin käytettäväksi):

```ruby
irb(main):043 > koff = Brewery.first
irb(main):044 > Beer.create name: "iso 3", style: "Lager", brewery_id: koff.id
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:09", updated_at: "2022-08-09 13:54:09">
irb(main):045 > Beer.create name: "Karhu", style: "Lager", brewery_id: koff.id
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:20", updated_at: "2022-08-09 13:54:20">
irb(main):046 >
```

Luodut oluet **iso 3** ja **Karhu** siis liitetään panimoon Koff. Tietokannan tasolla oluiden ja panimon välillä on liitos. Koodin tasolla liitos ei kuitenkaan vielä toimi.

Jotta saamme liitokset toimimaan myös koodin tasolla, muokataan modeleja seuraavasti:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
end

class Brewery < ApplicationRecord
  has_many :beers
end
```

eli olut liittyy yhteen panimoon ja panimolla on useita oluita. Huomaa monikko ja yksikkö!

Mennään taas konsoliin. Jos konsoli oli auki kun teit muutokset koodiin, anna ensin komento <code>reload!</code> jotta koodin uusi versio latautuu konsolin käyttöön.

Kokeillaan ensin miten pääsemme käsiksi panimon oluisiin:

```ruby
irb(main):047 > koff = Brewery.find_by name: "Koff"
irb(main):048 > koff.beers.count
 => 2
irb(main):049> koff.beers
  Beer Load (2.0ms)  SELECT "beers".* FROM "beers" WHERE "beers"."brewery_id" = ?  [["brewery_id", 1]]
=>
[#<Beer:0x00007efe39e64ec8
  id: 1,
  name: "Iso 3",
  style: "Lager",
  brewery_id: 1,
  created_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00>,
 #<Beer:0x00007efe39e41180
  id: 2,
  name: "Karhu",
  style: "Lager",
  brewery_id: 1,
  created_at: Mon, 08 Aug 2022 17:13:09.139694000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 17:13:09.139694000 UTC +00:00>]
```

<code>Brewery</code>-olioille on siis ilmestynyt metodi <code>beers</code>, joka palauttaa panimoon liittyvät <code>Beer</code>-oliot. Rails generoi automaattisesti tämän metodin nähtyään <code>Brewery</code>-luokassa rivin <code>has_many :beers</code>. 

Yksittäisiin panimoon liittyviin oluisiin pääsee käsiksi seuraavasti:

```ruby
irb(main):050 > koff = Brewery.find_by name: "Koff"
irb(main):051 > koff.beers.first
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:09", updated_at: "2022-08-09 13:54:09">
irb(main):052 > koff.beers.last
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:20", updated_at: "2022-08-09 13:54:20">
irb(main):053 > koff.beers[1]
 => #<Beer id: 2, name: "Karhu", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:20", updated_at: "2022-08-09 13:54:20">
```

Panimoon liittyvän olutkokoelman läpikäynti onnistuu esim. <code>each</code>-iteraattorilla:

```ruby
irb(main):055 > koff.beers.each { |beer| puts beer.name }
iso 3
Karhu
```

Myös olueeseen liittyvään panimoon pääsee käsiksi helposti kooditasolla:

```ruby
irb(main):056 > bisse = Beer.first
 => #<Beer id: 1, name: "iso 3", style: "Lager", brewery_id: 8, created_at: "2022-08-09 13:54:09", updated_at: "2022-08-09 13:54:09">
irb(main):057 > bisse.brewery
 => #<Brewery id: 8, name: "Koff", year: 1897, created_at: "2022-08-09 13:51:26", updated_at: "2022-08-09 13:51:26">
irb(main):058 >
```

Eli <code>Beer</code>-luokkaan lisätty rivi <code>belongs_to :brewery</code> lisää oluille metodin <code>brewery</code>, joka palauttaa olueeseen tietokannassa liitetyn panimo-olion.

## Tietokannan alustus

Ohjelmiston kehitysvaiheessa saattaa joskus olla hyödyksi generoida tietokantaan "kovakoodattua" dataa. Oikea paikka tälläiselle datalle on tiedosto db/seeds.rb

Kopioi seuraava sisältö sovelluksesi seeds.rb-tiedostoon:

```ruby
b1 = Brewery.create name: "Koff", year: 1897
b2 = Brewery.create name: "Malmgard", year: 2001
b3 = Brewery.create name: "Weihenstephaner", year: 1040

b1.beers.create name: "Iso 3", style: "Lager"
b1.beers.create name: "Karhu", style: "Lager"
b1.beers.create name: "Tuplahumala", style: "Lager"
b2.beers.create name: "Huvila Pale Ale", style: "Pale Ale"
b2.beers.create name: "X Porter", style: "Porter"
b3.beers.create name: "Hefeweizen", style: "Weizen"
b3.beers.create name: "Helles", style: "Lager"
```

Poistetaan kaikki vanha data tietokannasta antamalla komentoriviltä komento:

    rails db:reset

Komento "seedaa" kannan automaattisesti eli vanhan datan poistamisen lisäksi suorittaa myös tiedoston seeds.rb sisällön.

**Sovellus sekä Rails-konsoli kannattaa uudelleenkäynnistää seedauksen jälkeen.**

**HUOM:** et välttämättä tarvitse sovelluksessasi ollenkaan tiedostoon _seeds.rb_ määriteltyä dataa. Seediin määritelty data voi olla tarpeen jos sovellus tarvitsee esim. käynnistyäkseen joitan valmiksi määriteltyjä olioita. Tällöin seedin olemassaolo helpottaa uuden sovelluskehittäjän työtä, hän saa sovelluksen heti toimimaan omalla koneellaan ilman potentiaalisesti vaivalloista välttämättömien olioiden luomista.

## lisää konsolin käyttöä

Tutkitaan uutta dataa konsolista käsin:

```ruby
irb(main):003:0> koff = Brewery.first
  Brewery Load (2.9ms)  SELECT "breweries".* FROM "breweries" ORDER BY "breweries"."id" ASC LIMIT ?  [["LIMIT", 1]]
=>
#<Brewery:0x00007efe3abb1ea0
...
irb(main):004:0> koff.beers
  Beer Load (3.2ms)  SELECT "beers".* FROM "beers" WHERE "beers"."brewery_id" = ?  [["brewery_id", 1]]
=>
[#<Beer:0x00007efe3ab2d1a0
  id: 1,
  name: "Iso 3",
  style: "Lager",
  brewery_id: 1,
  created_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00>,
 #<Beer:0x00007efe3ab2d010
  id: 2,
  name: "Karhu",
  style: "Lager",
  brewery_id: 1,
  created_at: Mon, 08 Aug 2022 17:13:09.139694000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 17:13:09.139694000 UTC +00:00>,
 #<Beer:0x00007efe3ab2ce80
  id: 3,
  name: "Tuplahumala",
  style: "Lager",
  brewery_id: 1,
  created_at: Mon, 08 Aug 2022 17:13:09.171706000 UTC +00:00,
  updated_at: Mon, 08 Aug 2022 17:13:09.171706000 UTC +00:00>]
irb(main):005:0>
```

Luodaan uusi olut-olio. Käytetään tällä kertaa new-metodia, jolloin olio ei vielä talletu tietokantaan:

```ruby
irb(main):005:0> b = Beer.new name: "Lite", style: "Lager"
=> #<Beer:0x00007efe396c2aa8 id: nil, name: "Lite", style: "Lager", brewery_id: nil, created_at: nil, updated_at: nil>
irb(main):006:0>
```

Olut ei ole tietokannassa, eikä myöskään liity vielä mihinkään panimoon:

```ruby
irb(main):006:0> b.brewery
=> nil
```

Oluen voi liittää panimoon muutamallakin tavalla. Voimme asettaa oluen panimokentän arvon käsin:

```ruby
irb(main):008:0> b.brewery = koff
=>
#<Brewery:0x00007efe3abb1ea0
...
irb(main):009:0> b
=> #<Beer:0x00007efe396c2aa8 id: nil, name: "Lite", style: "Lager", brewery_id: 1, created_at: nil, updated_at: nil>
```

Kuten huomaamme, tulee oluen brewery_id-vierasavaimeksi panimon id. Olut ei ole vielä tietokannassa, eikä panimokaan vielä tästä syystä tiedä, että luotu olut liittyy siihen:

```ruby
irb(main):064:0> koff.reload
irb(main):065:0> koff.beers.include? b
 => false
```

Huom: kutsuimme ensin varalta panimon tietokannasta uudelleenlataavaa metodia <code>reload</code>, muuten olion tila ei olisi ollut tuore, ja siihen liittyvä olutlistakin olisi vastannut olion lataushetken oluiden listaa.

Olut saadaan tallettumaan tuttuun tapaan komennolla <code>save</code>. Tämän jälkeen myös panimon mielestä olut liittyy panimoon (jälleen lataamme olion ensin kannasta uudelleen):

```ruby
irb(main):066:0> b.save
 => true
irb(main):067:0> koff.reload
irb(main):068:0> koff.beers.include? b
 => true
```

Hieman kätevämpi tapa on liittää olut panimon oluiden joukkoon <code><<</code> operaattorilla:

```ruby
irb(main):069:0> b = Beer.new name: "IVB", style: "Lager"
 => #<Beer id: nil, name: "IVB", style: "Lager", brewery_id: nil, created_at: nil, updated_at: nil>
irb(main):070:0 > koff.beers << b
   (0.1ms)  begin transaction
  Beer Create (0.3ms)  INSERT INTO "beers" ("name", "style", "brewery_id", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["name", "IVB"], ["style", "Lager"], ["brewery_id", 1], ["created_at", "2018-09-01 16:46:01.643854"], ["updated_at", "2018-09-01 16:46:01.643854"]]
```

Vaikka luotua olutta ei tässä eksplisiittisesti talletettu <code>save</code>-metodilla, tallentuu olut kantaan operaattorin <code><<</code> käytön ansiosta.

Kolmas tapa on tiedostossa <code>seeds.rb</code> käytetty tyyli, jossa metodia <code>create</code> kutsutaan suoraan panimon beers-kokoelmalle:

```ruby
irb(main):071:0> koff.beers.create name: "Extra Light Triple Brewed", style: "Lager"
```

> ## Tehtävä 5: Panimoja ja oluita
>
> Tee konsolista käsin seuraavat toimenpiteet:
>
> - Luo panimo Hartwall ja sille kolme olutta kaikkia kolmea yllä demonstroitua tapaa käyttäen.
> - Päädymme kuitenkin siihen että Hartwall on huonon laatunsa takia poistettava. Ennen poistamista, ota muistiin Hartwall-olion id
> - Hartwallin poistaminen jättää tietokantaan olut-olioita, jotka liittyvät jo poistettuun panimoon
> - Hae orvoksi jääneet oluet komennolla <code>Beer.where tähänsopivaparametri</code>
> - Tuhoa operaation palauttamat oluet. Ohjeita oluiden listan läpikäyntiin esim. seuraavasta https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#taulukko

## Kontrollerin ja viewien yhteys

Tutkitaan hieman panimon valmiiksi generoitua kontrolleria app/controller/breweries_controller.rb

Kontrolleri on siis nimetty Railsin konvention mukaan monikkomuodossa. Kontrollerissa on Railsin konventioiden mukaan 6 metodia, tutkitaan niistä aluksi kaikkien oluiden näyttämisestä huolehtivaa metodia <code>index</code>:

```ruby
class BreweriesController < ApplicationController
  # ...

  def index
    @breweries = Brewery.all
  end

  # ...
end
```

Metodi sisältää ainoastaan komennon

```ruby
@breweries = Brewery.all
```

eli sijoittaa kaikkien panimoiden listan <code>@breweries</code>-nimiseen muuttujaan, jonka avulla se välittää panimoiden listan näkymälle. Tämän jälkeen metodi index renderöi näkymätemplatessa app/views/breweries/index.html.erb määritellyn html-sivun. Metodi ei missään vaiheessa viittaa näkymätemplateen tai sisällä käskyä sen renderöintiin. Kyse on jälleen Railsin convention over configuration -periaatteesta, eli jos ei muuta sanota, renderöidään kontrollerin index-metodin lopussa index.html.erb-näkymätemplate.

Renderöintikomento voitaisiin kirjoittaa myös eksplisiittisesti:

```ruby
def index
  @breweries = Brewery.all
  render :index   # renderöi hakemistossa view/breweries olevan näkymätemplaten index.html.erb
end
```

Näkymätemplatet, eli erb-tiedostot ovat html:ää, joihin on upotettu Ruby-koodia.

Tarkastellaan valmiiksigeneroitua näkymätemplatea eli tiedostoa app/views/breweries/index.html.erb

```html
<p style="color: green"><%= notice %></p>

<h1>Breweries</h1>

<div id="breweries">
  <% @breweries.each do |brewery| %> 
    <%= render brewery %>
    <p><%= link_to "Show this brewery", brewery %></p>
  <% end %>
</div>

<%= link_to "New brewery", new_brewery_path %>
```

Näkymätemplate muodostaa listan, jossa jokainen muuttujan @breweries sisältämä panimo tulee omalle rivilleen.

Näkymätemplateen upotettu Ruby-koodi tulee <% %> merkkien sisälle. <%= %> taas aiheuttaa Ruby-komennon arvon tulostumisen ruudulle.

Tutustumme listan generointiin kohta hieman tarkemmin. Lisätään ensin sivulle (eli erb-templateen) tieto panimoiden yhteenlasketusta määrästä. Eli lisää johonkin kohtaan sivua, esim. heti h1-tagien sisällä olevan otsikon jälkeen seuraava rivi

```html
<p>Number of breweries: <%= @breweries.count %></p>
```

Mene nyt selaimella [panimot listaavalle sivulle](http://localhost:3000/breweries) ja varmista, että lisäys toimii.

Palataan sitten tarkemmin panimot listaavaan koodiin. Jokainen panimo tulostuu omalle rivilleen listaan Rubyn <code>each</code>-iteraattoria käyttäen:

```html
  <% @breweries.each do |brewery| %>
    <%= render brewery %>
    <p>
      <%= link_to "Show this brewery", brewery %>
    </p>
  <% end %>
```

Muuttujaan `@breweries` talletettu panimoiden lista käydään läpi `each`-iteraattorin avulla. (lisää eachista ks. https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#each). 

Rails luo rivit käyttäen <code>render</code> metodia jota se kutsuu jokaiselle panimolle: 

```html
  <%= render brewery %>
```

<code>render</code> metodi käyttää hyväkseen [Partial templateja](https://guides.rubyonrails.org/layouts_and_rendering.html#using-partials) eli tutummin "partialeja". 

Rails on luonut yksittäiselle panimolle automaattisesti partials-tiedoston (app/views/breweries/\_brewery.html.erb). Partiaali näyttää seuraavalta

```html
<div id="<%= dom_id brewery %>">
  <p>
    <strong>Name:</strong>
    <%= brewery.name %>
  </p>

  <p>
    <strong>Year:</strong>
    <%= brewery.year %>
  </p>

</div>
```

Jokaista yksittäistä panimoa kohti siis luodaan listaan div-tagien sisällä olevat rivit. Ensimmäiselle riville tulee panimon nimi ja toiselle perustamisvuosi.

Partials-tiedostot nimetään käyttäen tiedoston alussa alaviivaa, jotta ne pystytään jo silmäyksellä erottamaan normaaleista näkymistä.

Partialin määrittelyssä ja käytössä on hieman Rails-magiaa. Partialia siis kutsutaan seuraavasti:

```html
  <% @breweries.each do |brewery| %>
    <%= render brewery %>
    ...
  <% end %>
```

Render-metodi saa parametrikseen muuttujaan brewery tallennetun panimo-olion. Olion tyypistä Rails osaa päätellä, että sen tulee käyttää tiedostoon \_brewery.html.erb talletettua partialia.

Partialin koodissa panimo-olioon viitataan muuttujan brewery kautta:

```html
<div id="<%= dom_id brewery %>">
  <p>
    <strong>Name:</strong>
    <%= brewery.name %>
  </p>

  ...

</div>
```

Panimon tietojen kolmannelle riville luodaan linkki panimon tiedot näyttävälle sivulle. Linkin generoiva Ruby-koodi on `<%= link_to "Show this brewery", brewery %>` .

Kyseessä on oikeastaan lyhennysmerkintä seuraavasta:

```
<%= link_to "Show this brewery", brewery_path(brewery.id) %>
```

joka generoi sivulle seuraavanlaisen HTML-koodin (seuraavassa oleva numero riippuu olion id-kentän arvosta):

```
<a href="/breweries/1">Show this brewery</a>
```

eli linkin osoitteeseen "breweries/1". Komennon `link_to` ensimmäinen parametri siis on a-tagiin tuleva nimi, ja toinen on linkin osoite.

Itse osoite luodaan tässä pitemmässä muodossa apumetodilla `brewery_path(brewery.id)`, joka palauttaa polun id:n `brewery.id` omaavan panimon sivulle. Saman asian siis metodin <code>link_to</code> parametrina saa aikaan olio itse, eli esimerkkimme tapauksessa muuttuja <code>brewery</code>

Linkin generoivan komennon voisi myös "kovakoodata" muodossa `<%= link_to "Show this brewery", "breweries/#{brewery.id}" %>`, mutta kovakoodaus ei ole yleensä eikä tässäkään tapauksessa kovin järkevää.

Mitä tarkoittaa `"breweries/#{brewery.id}"`? Kyseinen merkkijonon alussa on ensiksi kaikkiin panimoihin viittaava "breweries", jonka jälkeen siihen tulee yksilöivän panimon id muttujana. Muuttuja asetetaan käyttäen <code>#{}</code>-notaatiota, jonka avulla merkkijonoon voidaan upottaa muuttajia.

> ## Tehtävä 6
>
> muuta panimon nimi klikattavaksi (partials-tiedostossa) ja poista listasta show-kenttä linkkeineen

Tehtävän jälkeen sovelluksesi panimot näyttävien sivujen tulisi näyttää seuraavalta

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/brewery-w1-0a.png)

## Oluiden listaaminen panimon sivulla

Tutkitaan yksittäisen panimon näyttämistä. Url panimon sivulle on muotoa "breweries/3", missä numero on panimon id. Panimon sivulle menemisestä huolehtii breweries-kontrollerin metodi show:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: %i[ show, edit, update, destroy ]

  # muita metodeja...

  def show
  end

end
```

Metodi ei sisällä mitään koodia! Huomaamme kuitenkin, että luokan määrittelyn alussa on rivi

```ruby
before_action :set_brewery, only: %i[ show edit update destroy ]
```

Tämä taas saa aikaan sen, että ennen jokaista lueteltua metodia (show, edit, update ja destroy) suoritetaan metodin <code>set_brewery</code> koodi. Metodin määrittely löytyy luokan loppupuolelta:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: %i[ show, edit, update, destroy ]

  # ...

  def show
  end

  # ...

  private
    def set_brewery
      @brewery = Brewery.find(params[:id])
    end

end
```

Ennen metodin <code>show</code> suoritusta siis suoritetaan komento

```ruby
@brewery = Brewery.find(params[:id])
```

joka viittaa muuttujaan `params`, joka taas sisältää suorituksen alla olevaan HTTP-kutsuun liittyvät tiedot. Muuttuja <code>params</code> on tyypiltään assosiatiivinen taulukko eli hash. Erityisesti muuttujan arvo avaimella <code>:id</code> eli `params[:id]` kertoo tässä tapauksessa tarkasteltavana olevan panimon id:n, eli polun breweries/#{id}, kenoviivan jälkeisen osan.

Panimo haetaan tietokannasta tutulla komennolla `Brewery.find` ja sijoitetaan muuttujaan `@brewery`.
Metodi <code>show</code> renderöi lopuksi näkymätemplaten `show.html.erb`. Näkymätemplaten generointi tapahtuu jälleen automaattisesti Railsin konvention perusteella, eli panimokontrollerin metodin `show` suorituksen lopussa renderöidään näkymä views/breweries/show.html.erb ellei koodi määrää muuta.

Eksplisiittisesti auki kirjoitettuna metodin <code>show</code> suorituksen yhteydessä suoritettava koodi on siis seuraava:

```ruby
@brewery = Brewery.find(params[:id])
render :show
```

Näkymätemplaten views/breweries/show.html.erb koodi on seuraavassa:

```
<p style="color: green"><%= notice %></p>

<%= render @brewery %>

<div>
  <%= link_to "Edit this brewery", edit_brewery_path(@brewery) %> |
  <%= link_to "Back to breweries", breweries_path %>

  <%= button_to "Destroy this brewery", @brewery, method: :delete %>
</div>
```

Sivun yläosassa oleva id:llä **notice** varustettu osa on tarkoitettu näyttämään panimon luomiseen tai muutokseen liittyviä viestejä, asiasta lisää myöhemmin.

Sivulla käytetään samaa partialia render-metodissa, kuin kaikkien painimoiden sivulla. Aiemman muutoksen seurauksena sivun otsikko on nyt linkki sivuun itseensä.

Muutama tarkentava huomio partialiin liittyen. Panimon tiedot näyttävä partial siis on seuraava:

```html
<div id="<%= dom_id brewery %>">
  <p>
    <strong>Name:</strong>
    <%= brewery.name %>
  </p>

  <p>
    <strong>Year:</strong>
    <%= brewery.year %>
  </p>

</div>
```

Partialin parametriksi saama panimo on muuttujassa brewery. Partialia siis kutsuttaan seuraavasti

```html
<%= render @brewery %>
```

Partialille voitaisiin antaa enemmänkin parametreja, mutta nyt annettiin ainoastaan yksi. Koska parametri on tyypiltään panimo eli Brewery, renderöidään _brewery.html.erb partiaali ja välitetään sille parametri muuttujan nimeltään brewery kautta. Parametrin välittävä muuttuja saa nimensä automaattisesti partialin nimen perusteella.

Koska muutamme pian yksittäisen panimoin näkymää, mutta emme halua samaa templatea käyttävän kaikkien panimoiden näkyvän muuttuvan, luovutaan paritalin käytöstä kaikkien panimoiden sivulla ja muutetaan tiedosto app/views/breweries/index.html.erb seuraavaan muotoon:

```html
<p style="color: green"><%= notice %></p>

<h1>Breweries</h1>

<p>Number of breweries: <%= @breweries.count %></p>

<div id="breweries">
  <% @breweries.each do |brewery| %>
    <div id="<%= dom_id brewery %>">
      <p>
        <%= link_to brewery.name, brewery %>
      </p>
    
      <p>
        <strong>Year:</strong>
        <%= brewery.year %>
      </p>
    
    </div>
  <% end %>
</div>

<%= link_to "New brewery", new_brewery_path %>
```

Partialin \_brewery.html.erb koodi on siis copypastettu suoraan kaikkien panimoiden templaten sisään.

> ## Tehtävä 7: Panimon sivun hiominen
>
> Lisätään sivulle tieto panimoon liittyvien oluiden määrästä eli renderöi sivun sisällä <code>@brewery.beers.count</code>
>
> Muokkaa valmista sivua siten, että panimon nimestä tulee h2-tason otsikko ja vuosi ilmoitetaan kursivoituna tyyliin "_Established_ _in_ _1897_". Tämän tehdäksesi on otettava partialsin renderöinti pois käytöstä ja tehtävä renderöinti ilman sen apua.
>
> Nämä, ja seuraavan tehtävän muutokset siis tehdään yksittäisen panimon tietojen näyttämisestä huolehtivaan partialiin \_brewery.html.erb

Jatketaan muutosten tekemistä.

> ## Tehtävä 8: Oluet panimon sivulle
>
> Lisätään nyt panimon sivulle lista panimoon liittyvistä oluista. Lisää aluksi sivulle seuraava <code><%= brewery.beers.to_a %></code> ja katso aikaansannosta.
>
> Listaa seuraavaksi ainoastaan oluiden nimet käyttäen each-toistoa:
>
> ```ruby
> <p>
>  <% brewery.beers.each do |beer| %>
>    <%= beer.name %>
>  <% end %>
> </p>
> ```
>
> Muuta vielä oluiden nimet klikattavaksi metodin <code>link_to</code> avulla

Sivusi tulisi näyttää tehtävän jälkeen seuraavalta

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/brewery-w1-1.png)

Parannellaan vielä hieman sovelluksemme navigaatiota.

> ## Tehtävä 9
>
> Lisää kaikkien panimojen sivulle linkki oluiden sivulle ja vastaavasti oluiden sivulle linkki panimoiden sivulle, esim. linkki oluiden sivulle saadaan komennolla `link_to 'list of beers', beers_path`

Viritellään lopuksi kaikkien oluiden listaa.

> ## Tehtävä 10
>
> Tällä hetkellä kaikkien oluiden listalla näytetään olueeseen liittyvän panimon id
>
> Muuta sivua siten että panimon id:n sijaan näytetään olueeseen liittyvän panimon nimi, ja että nimeä klikkaamalla päästään panimon sivulle
>
> Muuta myös oluen nimi klikattavaksi ja poista show-linkki
>
> Huom: jos törmäät ongelmiin, kannattaa lukea seuraava luku!

Lopputuloksen pitäisi näyttää seuraavalta:

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/brewery-w1-2.png)

## nil

Saatat törmätä seuraavanlaiseen virheilmoitukseen

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/brewery-w1-3.png)

Kyse on oikeastaan klassisesta nullpointer-poikkeuksesta, tai sen Ruby-versiosta nilpointer-poikkeuksesta. Rails vihjaa, että olet yrittänyt kutsua nil:ille (joka on Rubyssä olio!) metodia name, ja että sellaista ei ole. Syynä tälle on todennäköisesti se, että tietokannassasi on oluita joihin ei liity panimoa tai että niihin liittyvä panimo on poistettu.

Voit poistaa ongelman aiheuttavat oluet käsin tai konsolin avulla. Saat haettua orvot oluet konsolista komennolla:

```ruby
orvot_oluet = Beer.all.select{ |b| b.brewery.nil? }
```

ja niiden poistaminen onnistuu sitten each-iteraattorin avulla

```ruby
orvot_oluet.each{ |orpo| orpo.delete }
```

koska jokaiselle iteroitavalla oliolle kutsutaan ainoastaan metodia, onnistuu edellinen myös seuraavalla, hieman erikoisen näköisellä syntaksilla:

```ruby
orvot_oluet.each(&:delete)
```

## Kertausta: polkujen ja kontrollerien nimentäkonventiot

Loimme siis sovellukseemme tietokantataulut panimoille ja oluille sekä molempien hallinnointiin tarkoitetut kontrollerit ja näkymät. Kerrataan vielä Railsin nimentäkonventioita, joihin tottumiseen saattaa aloittelijalla mennä hetki.

Panimo ja siihen liittyvät kontrollerit ja näkymät luotiin Railsin scaffold-generaattorilla seuraavasti:

```bash
rails g scaffold Brewery name:string year:integer
```

Tästä seurauksena syntyi

- tietokantataulu <code>breweries</code>
- kontrolleri <code>BreweriesController</code> hakemistoon app/controllers/
- model <code>Brewery</code> hakemistoon app/models/
- joukko näkymiä hakemistoon app/views/breweries
- tietokannan muodostamisesta huolehtiva migraatiotiedosto hakemistoon /db/migrate

Railsin konvention mukaan kaikkien panimoiden sivun URL on breweries, yksittäisten panimoiden sivujen URLit taas ovat muotoa _breweries/3_, missä numerona on panimon id.

URLeja ei itse kannata kirjoittaa näkymätemplateihin sillä Rails tarjoaa _path_helper_-metodeja (ks. http://guides.rubyonrails.org/routing.html#path-and-url-helpers), joiden avulla URLit saa generoitua.

Kaikkien panimoiden URLin (tai oikeastaan vain URLin jälkiosan) generoi metodi <code>breweries_path</code>, yksittäisen panimon URL saadaan generoitua metodilla <code>brewery_path(id)</code>, missä parametrina on linkin kohteena olevan panimon id.

Helppereitä käytetään usein yhdessä apumetodin <code>link_to</code> kanssa. link_to generoi HTML-sivulle linkin eli a-tagin.

Linkin panimon <code>brewery</code> sivulle voi generoida seuraavasti:

```ruby
<%= link_to "linkki panimoon #{brewery.name}", brewery_path(brewery.id) %>
```

Ensimmäisenä parametrina on siis linkin teksti ja toisena kohteena oleva osoite.

Usein tehtäessä linkkiä yksittäisen olion sivulle käytetään edellisestä lyhempää muotoa:

```ruby
<%= link_to "linkki panimoon #{brewery.name}", brewery %>
```

Nyt toisena parametrina on siis suoraan olio, jonka sivulle linkki johtaa. Kun toinen parametri on olio, korvaa Rails sen automaattisesti todellisen polun generoimalla koodilla <code>brewery_path(brewery.id)</code>

Railsin automaattisesti generoiduissa kontrollereissa on valmiina kuusi metodia. Kaikkien panimoiden listaa, eli osoitetta /breweries hallinnoi metodi <code>index</code>, yksittäisen panimon osoitetta, esim. /breweries/3 hallinnoi kontrollerin metodi <code>show</code>. Tutustumme myöhemmin kontrollerin muihin metodeihin.

Kontrollerien metodit renderöivät lopuksi käyttäjälle palautettavan HTML-sivun muodostavan templaten. Oletusarvoisesti panimokontrollerin metodi <code>index</code> renderöi näkymätemplaten app/views/breweries/index.html.erb ja metodi <code>show</code> renderöi näkymätemplaten app/views/breweries/show.html.erb

Kontrollereiden ei siis tarvitse erikseen kutsua renderöintiä suorittavaa metodia <code>render</code> jos ne renderöivät oletustemplaten. Eli koodi

```ruby
class BreweriesController < ApplicationController
  def index
    @breweries = Brewery.all
    render :index
  end
```

toimii täsmälleen samalla tavalla kuin seuraava

```ruby
class BreweriesController < ApplicationController
  def index
    @breweries = Brewery.all
  end
```

Eksplisiittinen <code>render</code>-metodin kutsuminen on siis tarpeen vain silloin kun kontrolleri renderöi jonkin muun kuin oletusnäkymän.

## Kertausta: näkymät ja näkymäpartiaalit

Kerrataan vielä muutamaa asiaa näkymätemplateista. Tarkastellaan tilannetta, missä käyttäjä navigoi yksittäisen oluen sivulle esim. osoitteeseen beers/1. Sovellus suorittaa olutkontrollerin metodin show, joka renderöi oletusarvoisen templaten views/beers/show.html.erb. Kontrolleri asettaa tarkasteltavan oluen muuttujaan @beer kiitos before\_action määrittelyn:

```ruby
class BeersController < ApplicationController
  before_action :set_beer, only: %i[ show edit update destroy ]

  # suoritetaan mentäessä yksittäisen oluen sivulle /beers/1
  def show
  end

  private
    # tämä suoritetaan metodin show suoritusta ennen
    def set_beer
      @beer = Beer.find(params[:id])
    end

end
```

Näkymätemplate siis näkee käsiteltävän oluen muutujan @beer kautta: 

```html
<%= render @beer %>

<div>
  <%= link_to "Edit this beer", edit_beer_path(@beer) %> |
  <%= link_to "Back to beers", beers_path %>

  <%= button_to "Destroy this beer", @beer, method: :delete %>
</div>
```

Näkymätemplate ei hoida kaikkea renderöintiä itse, se hoitaa oluen tietojen renderöinnin partiaalin views/beers/\_beer.html.erb avulla. Partial renderöidään metodin render avulla:

```html
<%= render @beer %>
```

Koska metodikutsun parametrin tyyppi on Beer, tietää Rails, että on tarkoitus renderöidä nimenomaan partiaali _beer.html.erb. Partiaali näkee parametrin muuttujana beer, muuttujan nimi on Rails-magian avulla automaattisesti partialin nimen perusteella määrittyvä.

```html
<div id="<%= dom_id beer %>">
  <p>
    <%= link_to beer.name, beer %>
  </p>

  <p>
    <strong>Style:</strong>
    <%= beer.style %>
  </p>

  <p>
    <strong>Brewery:</strong>
    <%= link_to beer.brewery.name, beer.brewery %>
  </p>

</div>
```

> ## Tehtävä 11
>
> Muuta tilapäisesti panimokontrollerin <code>index</code>-metodia seuraavasti
>
> ```ruby
>  def index
>    @breweries = Brewery.all
>
>    render :panimot
>  end
> ```
>
> Kokeile mitä tapahtuu kun menet panimoiden sivulle eli osoitteeseen [http://localhost:3000/breweries](http://localhost:3000/breweries)
>
> Lisää nyt hakemistoon app/views/breweries tiedosto panimot.html.erb ja lisää sinne esim.
>
> ```ruby
>  panimoita <%= @breweries.count %>
> ```
>
> Mene panimoiden sivulle.
>
> Palauta metodi nyt entiselleen.

## Sovellus internettiin

Railsissa sovellukset käyttävät oletusarvoisesti sqlite-tietokantaa, mutta tuotantokäytössä on syytä käyttää jotain todellista tietokantaa, kuten PostgreSQL:ää. Rails-sovelluksen käyttämät kirjastot eli Rubyn termein gemit on määritelty sovelluksen juuressa olevassa Gemfile-nimisessä tiedostossa. Jotta saamme PostgreSQLn käyttöön, joudumme tekemään muutoksen Gemfileen.

Poista rivi

```ruby
gem 'sqlite3'
```

ja lisää johonkin kohtaa tiedostoa seuraavat

```ruby
group :development, :test do
  gem 'sqlite3'
end

group :production do
   gem 'pg'
end
```

Suoritetaan komentoriviltä komento <code>bundle install</code>, jotta muutokset tulevat käyttöön:

```ruby
$ ratebeer git:(master) ✗ bundle install
Fetching gem metadata from https://rubygems.org/..........
Resolving dependencies...
Using rake 13.0.6
Using concurrent-ruby 1.1.10
Using minitest 5.16.1
Using builder 3.2.4
...
Using rails 7.0.3
Installing pg 1.4.1 with native extensions
Bundle complete! 16 Gemfile dependencies, 75 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

Jos <code>bundle install</code> aiheuttaa virheen (näin voi käydä todennäköisesti vain OS X:llä), voit joko

- suorittaa komennon lisäparametrilla <code>bundle install --without production</code>
  - tämä tapa on suositeltava ellet välttämättä halua asentaa _postgresql_ tietokantaa koneellesi (tällä kurssilla sille ei ole tarvetta, koska _postgresql_ ei ole käytössä lokaalisti)
  - huomaa, että jos päädyt tähän vaihtoehtoon, joudut jatkossa suorittamaan komennon aina näin
- tai vaihtoehtoisesti asentaa ensin _postgresql_ tietokannan omalle koneellesi ja suorittaa komennon uudelleen

Committoidaan kaikki muutokset versionhallintaan antamalla komennot:

```ruby
git add -A
git commit -m"updated Gemfile for Internet"
```

Sovellusten hostaamiseen, eli "internettiin laittamiseen" on olemassa lukematon määrä erilaisia ratkaisuja. Helpoimpia näistä sovelluskehittäjän kannalta ovat ns PaaS (eli Platform as a Service) -palvelut, jotka huolehtivat sovelluskehittäjän puolesta tietokannan ja suoritusympäristön asentamisen.

Kymmenen vuoden ajan PaaS-ratkaisujen ykkönen on ollut [Heroku](http://heroku.com). Elokuun 2022 lopussa Heroku ilmoitti että 27.11.2022 alkaen alustan maksuttomat palvelut loppuvat. Tämän takia esittelemme seuraavassa myös lupaavan korvaajan [Fly.io](https://fly.io/). Voit käyttää kumpaa vaan kunhan muistat sen että Herokun ilmaisuus loppuu pian. Heroku on lupaillut jonkinlaista ilmaista käyttömahdollisuutta opiskelijoille, mutta sen varaan ei kannata liiaksi tässä vaiheessa laskea.

### Fly.io

Luo tunnus [Fly.io](https://fly.io/)-palveluun. Palveluun voi kirjautua myös GitHub-tunnuksella.

Asenna Fly.io [tämän ohjeen](https://fly.io/docs/getting-started/installing-flyctl/) mukaan. Asennuksen yhteydessä sinua voidaan pyytää manuaalisesti tallentamaan Fly.ion vaatimat ympäristömuuttujat. Toimi kehoitteen mukaan.

Edetään sitten noudattaen Fly.io:n [Rails](https://fly.io/docs/rails/getting-started/)-ohjetta.

Mene sovelluksen hakemistoon ja luo Fly.io-sovellus komennolla `fly launch`. Samalla tapahtuu Postgres-tietokannan luominen. Vastaile kysymyksiin "sopivasti". (Sovelluksen nimen tulee sisältää vain numeroita, pieniä kirjaimia ja väliviivoja.) Huomaa, että joudut hyväksymään tietokannan luomisen, oletusarvoisesti kysymykseen on vastattu 'No':

![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/fly-pg.png)
    
Vie sovellus tuotantoon komennolla `fly deploy`. Tämä komento toistetaan aina kun sovellukseen uusi versio halutaan internettiin.

Sovelluksen saa avattua selaimeen komennolla `fly open`.

**Huomaa**, että sovelluksen juuressa, eli esim. omassa tapauksessani osoitteessa https://ratebeer.fly.dev/ ei ole (tällä hetkellä) mitään. Sovelluksemme oluet tulevat löytymään osoitteesta https://ratebeer.fly.dev/beers ja panimot osoitteesta https://ratebeer.fly.dev/breweries

Jos jokin ei toimi, vikaa kannattaa etsiä palvelimen lokeista jotka näkee komennolla `fly logs`.

Myös palvelimella olevan sovelluksen Rails-konsoliin on mahdollista ottaa yhteys, se tapahtuu menemällä ensin sovellusta pyörittävään Docker-konttiin komennolla `fly ssh console` ja antamalla konsolissa komento `/app/bin/rails c`.

Ennen komentoa joudut todennäköisesti autentikoitumaan komennolla `fly log` 

Myös sovelluksen postgres-tietokantaan on mahdollista ottaa yhteys komennolla `flyctl postgres connect -a tietokannan_nimi` . Tietokannan nimi on sovelluksen nimi, jonka perään on lisätty `-db`. Oman sovelluksen nimi on ratebeer, joten tietokantayhteys onnistuu seuraavasti

`flyctl postgres connect -a ratebeer-db`

Kokemusta Fly.io:n käytöstä laitoksen kursseilla ei vielä juurikaan ole, jos törmään ongelmiin, kysy apua Discordissa.

Fly.io:ssa tapahtuu kaikenlaista mielenkiintoista pellin alla. Sovellus suoritetaan ns. [Docker](https://www.docker.com/)-kontissa. Hakemistoosi syntyy kontin muodostamisen määrittelevä tiedosto Dockerfile. Docker on tämän päivän ja tulevaisuuden ykkösteknologia sovellusten tuotantokäytössä ja monessa muussa. Tällä kurssilla pärjäät vielä ilman konepellin alle kurkkimista, mutta syvällisempi tutustuminen Dockeriin on ohjelmistoalan ihmisille jo lähes välttämättömyys. Kun aikaa jää, kannattaa suorittaa kurssi [DevOps with Docker](https://devopswithdocker.com/)

### Heroku

Sovelluksen deployaaminen Herokuun onnistuu helpoiten jos sovelluksen hakemisto on oma git-repositorionsa.

Jos et ole käyttänyt aiemmin Herokua

- luo Herokuun tunnus.
- luo ssh-avain ja lisää se herokuun sivulla https://dashboard.heroku.com/account
  - ohje ssh-avaimen luomiseen https://github.com/mluukkai/otm-2018/blob/master/tehtavat/viikko1.md#julkinen-avain
- Asenna komentoriviliittymän sisältävä Heroku CLI sivun https://devcenter.heroku.com/articles/heroku-cli ohjeiden mukaan.

Asennettuasi komentorivikäyttöliittymän mene sovelluksen juurihakemistoon, ja luo sovellusta varten heroku-instanssi komennolla <code>heroku create</code>:

```ruby
$ heroku create
Creating app... done, ⬢ young-escarpment-87255
https://young-escarpment-87255.herokuapp.com/ | https://git.heroku.com/young-escarpment-87255.git
```

Syötä pyydettäessä Heroku-tunnuksesi.

Sovelluksen URL tulee olemaan tässä tapauksessa https://young-escarpment-87255.herokuapp.com/. Sovelluksen URLin alkuosan saa haluamaansa muotoon antamalla komennon muodossa **heroku create urlin_alkuosa**.

**Huomaa**, että sovelluksen juuressa, eli osoitteessa https://young-escarpment-87255.herokuapp.com/ ei ole (tällä hetkellä) mitään. Sovelluksemme oluet tulevat löytymään osoitteesta https://young-escarpment-87255.herokuapp.com/beers ja panimot osoitteesta https://young-escarpment-87255.herokuapp.com/breweries


Nyt olemme valmiina käynnistämään sovelluksen Herokussa. Sovellus käynnistetään suorittamalla komentoriviltä operaatio <code>git push heroku main</code>

```ruby
$ git push heroku main
Counting objects: 136, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (122/122), done.
Writing objects: 100% (136/136), 30.06 KiB | 615.00 KiB/s, done.
Total 136 (delta 14), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
...
remote: -----> Discovering process types
remote:        Procfile declares types     -> (none)
remote:        Default types for buildpack -> console, rake, web, worker
remote:
remote: -----> Compressing...
remote:        Done: 47.7M
remote: -----> Launching...
remote:        Released v5
remote:        https://young-escarpment-87255.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/young-escarpment-87255.git
 * [new branch]      main -> main
```

Sovelluksen käynnistys näytti onnistuneen ongelmitta.

Avataan nyt selaimella panimoiden listan näyttävä sivu https://young-escarpment-87255.herokuapp.com/breweries

Seurauksena on kuitenkin ikävä virheilmoitus "We're sorry, but something went wrong.".

Voimme koittaa selvittää vikaa katsomalla herokun lokeja komennolla <code>heroku logs</code>. Tulostusta tulee aika paljon, mutta pienen etsinnän jälkeen syy selviää:

<pre>
2018-09-01T18:20:29.445705+00:00 app[web.1]: [ac5c97e8-2c40-4e97-9bf4-6d763bfd189f]
2018-09-01T18:20:29.453053+00:00 app[web.1]: [ac5c97e8-2c40-4e97-9bf4-6d763bfd189f] ActionView::Template::Error (PG::UndefinedTable: ERROR:  relation "breweries" does not exist
2018-09-01T18:20:29.453056+00:00 app[web.1]: LINE 8:                WHERE a.attrelid = '"breweries"'::regclass
2018-09-01T18:20:29.453058+00:00 app[web.1]: ^
2018-09-01T18:20:29.453060+00:00 app[web.1]: :               SELECT a.attname, format_type(a.atttypid, a.atttypmod),
2018-09-01T18:20:29.453064+00:00 app[web.1]: c.collname, col_description(a.attrelid, a.attnum) AS comment
2018-09-01T18:20:29.453065+00:00 app[web.1]: FROM pg_attribute a
2018-09-01T18:20:29.453062+00:00 app[web.1]: pg_get_expr(d.adbin, d.adrelid), a.attnotnull, a.atttypid, a.atttypmod,
</pre>

Syynä on siis se, että tietokantaa ei ole luotu _PG::UndefinedTable: ERROR: relation "breweries" does not exist_. Meidän on siis suoritettava migraatiot Herokussa olevalle sovellukselle. Tämä onnistuu komennolla <code>heroku run rails db:migrate</code>

Ja nyt sovellus toimii!

Kuten huomaat, tietokannassa olevat oluet ja panimot eivät siirry Herokuun. Jos haluat <code>seed.rb</code> tiedostossa määritellyt oliot tietokantaan, voit antaa komennon

    heroku run rails db:seed

Jatkossakin on siis aina muistettava suorittaa migraatiot deployatessamme sovellusta Herokuun.

Voimme myös avata Rails-konsolin Herokussa sijaitsevalle sovellukselle komennolla <code>heroku run console</code>:

```ruby
$ heroku run console
irb(main):001:0> Brewery.all
=> #<ActiveRecord::Relation [#<Brewery id: 1, name: "Koff", year: 1897, created_at: "2022-08-09 14:57:30", updated_at: "2022-08-09 14:57:30">, #<Brewery id: 2, name: "Malmgard", year: 2001, created_at: "2022-08-09 14:57:30", updated_at: "2022-08-09 14:57:30">, #<Brewery id: 3, name: "Weihenstephaner", year: 1040, created_at: "2022-08-09
```

Kyseessä on normaali Rails-konsolisessio, eli voit esim. tutkia Herokuun deployatun sovelluksen tietokannan tilaa session avulla.

## Riippuvuuksien hallinta

Kuten edellisessä luvussa mainittiin Rails-sovelluksen käyttämät kirjastot eli _gemit_ on määritelty sovelluksen juuressa olevassa Gemfile-nimisessä tiedostossa.

Ennen edellisessä luvussa tekemiämme muutoksia Gemfilen alku näyttää seuraavalta (poiskommentoidut osat on jätetty allaolevasta pois):

```ruby
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.1.2"

gem "rails", "~> 7.0.3"
gem "sprockets-rails"
gem "puma", "~> 5.0"
gem "importmap-rails"
gem "turbo-rails"
gem "stimulus-rails"
gem "jbuilder"
```

Gemfile siis listaa joukon gemejä, joita sovellus käyttää. Kuten huomaamme, on Rails itsekin gem. Joissain tapauksissa gemin yhteydessä määritellään käytettävä versio tai minimissään käyvä versionumero.

Riippuvuudet ladataan osoitteesta https://rubygems.org Bundler-ohjelmaa, ks. http://bundler.io/ käyttäen antamalla komentoriviltä komento <code>bundle install</code>. Bundler lataa gemit ja niiden riippuvuudet rubygems.org:ista ja tämän jälkeen sovellus on valmiina käytettäväksi.

Kun <code>bundle install</code> on suoritettu ensimmäisen kerran, syntyy tiedosto <code>Gemfile.lock</code> joka määrittelee tarkasti mitkä versiot gemeistä on asennettu. Gemfilehän ei määrittele välttämättä tarkkoja versioita. Tämän jälkeen kutsuttaessa <code>bundle install</code> asennetaan Gemfile.lock tiedostossa määritellyt versiot. Suorittamalla <code>bundle update</code> saadaan tarvittaessa ladattua uusimmat gemit ja luodaan uusi Gemfile.lock-tiedosto. Katso tarkemmin Bundlerin toiminnasta ositteesta https://bundler.io/docs.html

## Suoritusympäristöt

Rails tarjoaa oletusarvoisesti kolme eri suoritusympäristöä

- development eli sovelluskehitykseen tarkoitettu ympäristö
- test eli testien suorittamiseen tarkoitettu ympäristö
- production eli tuotantokäyttöön tarkoitettu ympäristö

Jokaisessa suoritusympäristössä on käytössä oma tietokanta ja Rails toimii myös hieman eri tavalla eri ympäristöissä.

Normaalisti ohjelmoija työskentelee siten, että sovellusta suoritetaan development-ympäristössä. Tällöin Rails tarjoaa mm. sovelluskehittäjän työtä helpottavia virheilmoituksia. Myös sovelluksen koodi ladataan aina suoritettaessa uudelleen. Tämän ansiosta sovellusta ei tarvitse käynnistää uudelleen koodia muutettaessa vaan muutettu ja lisätty koodi on aina "automaattisesti" sovelluksen käytössä.

Herokuun deployattaessa sovellus alkaa toimia production-ympäristössä, joka sisältää useita suorituskykyä optimoivia eroja development-ympäristöön nähden. Myös sovelluksen virheilmoitukset ovat erilaiset, virheen syyn ja sijainnin sijaan ilmoitetaan ainoastaan "Something went wrong...".

Testausympäristöön tutustumme kurssin viikolla 4.

Joskus eri ympäristöt tarvitsevat erilaisia riippuvuuksia, esim. kun sovellusta suoritetaan Herokussa production-ympäristössä on käytössä PostgreSQL-tietokanta, kun taas sovelluskehityksessä käytetään kevyempää sqlite3-tietokantaa. Samat gemit eivät siis sovellu kaikkiin suoritusympäristöihin.

Eri ympäristöjen käyttämät gemit voidaan määritellä Gemfilessä group-lohkojen avulla. Seuraavassa sovelluksemme Gemfile Herokun edellyttämien muutosten jälkeen:

```ruby
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "3.1.2"

gem "rails", "~> 7.0.3"
gem "sprockets-rails"
gem "puma", "~> 5.0"
gem "importmap-rails"
gem "turbo-rails"
gem "stimulus-rails"
gem "jbuilder"

group :development, :test do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
end

```

sqlite3 gem on siis käytössä ainoastaan development- ja test-ympäristöissä. Ainoastaan tuotantoympäristössä taas käytössä ovat gemit pg ja rails_12factor.

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi Githubiin. Lisää GitHubin Readme-tiedostoon linkki sovelluksen Fly.io- tai Heroku-instanssiin. Oletusarvoisesti Rails-sovelluksen readme-tiedostoon generoituvan sisältö kannattanee poistaa. _Sovelluksen vieminen Fly.io:hon/Herokuun ei ole edellytys palautuksen tekeminen, se on kuitenkin ehdottoman suositeltavaa ja hyödyllistä._ 

Jos käytät yksityistä repositoriota, lisää käyttäjät mluukkai ja kaltsoon collaboraattoreiksi!

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/stats/courses/rails2022

Jatketaan tykitystä: [viikko 2](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/viikko2.md).
