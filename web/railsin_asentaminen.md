**HUOM** sudoa ei tule käyttää rbenv:iä tai muita rubyn versiomanagereja käyttäessä (poislukien kirjastojen asentaminen), sillä asennus tehdään käyttäjän omaan kotihakemistoon.

Varaa asennukseen kunnolla aikaa ja tee se mieluusti joskus kun levypalvelinten käyttö on muutoin vähäistä. Älä jätä asennusta ohjauksen alkuun, jos haluat tehdä muutakin kuin pyöritellä peukaloita.

Kurssilla suositellaan ensisijaisesti käytettävän omaa tietokonetta.

--

Asennamme tässä Rubyn version 3.1.2 ja Railsin version 7.0.4 [rbenv-versiomanagerilla](https://github.com/sstephenson/rbenv)

7.0.4 on uusin versio tätä kirjoitettaessa (15.9.2022), voit katsoa [täältä](https://rubygems.org/gems/rails/versions) löytyykö vielä uudempaa versiota. Rubyn versiot selviävät [täältä](https://www.ruby-lang.org/en/downloads/releases/). 3.2 ilmestynee vielä syksyn aikana. Pärjäät kyllä kurssin 3.1.2:lla.

Voit halutessasi käyttää myös [RVM:ää](https://rvm.io/rvm/install) eli rbenvin lähisukulaista.

**Älä kuitenkaan missään tapauksessa asenna Rubyä/Railsia Linuxin paketinhallintajärjestelmän kautta!**

**Huom:** seuraavassa on ohjeet ainoastaan Linuxille ja OSX:lle.

## Rails Windowsille

Ruby on Railsin asentaminen Windowsiin onnistuu (ehkä) sivun [http://railsinstaller.org/en](http://railsinstaller.org/en) ohjeilla.

Windowsilla suositellaan käytettäväksi [WSL:ää](https://docs.microsoft.com/en-us/windows/wsl/install)

Jos haluat välttämättä käyttää Windowsia ja et suostu käyttämään osaston konetta tai omaa fuksiläppäriä, tapahtuu kurssille osallistuminen omalla vastuulla.

## Yliopiston etätyöpöytä

Yliopiston Linuxeille on nykyään mahdollista ottaa graafinen etätyöpöytäyhteys, ks  
https://helpdesk.it.helsinki.fi/ohjeet/tietokone-ja-tulostaminen/tyoasemapalvelu/etakaytettavat-tyopoydat-vdi-ja-vmware eli jos et saa asennettua Railsia omalle koneellesi, etätyöpöytä voi pelastaa päiväsi.

Myös etätyöpöytää käyttäessäsi joudut asentamaan Rubyn ja Railsin allaolevan ohjeen mukaan.

## rbenv Linuxille

Allaolevat on testattu osaston koneissa ja Ubuntun uusimman LTS version kanssa. Seuraavassa luvussa ohjeet OSX:lle. Windowsiin asentaminen ainoastaan omalla vastuulla!

**Huom:** koneella tulee olla muutamia kirjastoja, joiden asennus onnistuu Ubuntussa komennolla <code>sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev nodejs yarn</code>. **Osaston koneilla ja ajantasalla olevilla fuksikannettavilla kirjastot ovat valmiina.**

Toimitaan sivun [https://github.com/rbenv/rbenv#installation](https://github.com/rbenv/rbenv#installation) kohdan _Installation, Basic GitHub Checkout_ mukaan, eli annetaan terminaalissa seuraavat komennot:

- _git clone https://github.com/rbenv/rbenv.git ~/.rbenv_
- _echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile_
- _echo 'eval "$(rbenv init -)"' >> ~/.bash_profile_
- _echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc_
- _echo 'eval "$(rbenv init -)"' >> ~/.bashrc_

Uudelleenkäynnistä terminaali

- _mkdir -p "$(rbenv root)"/plugins_
- _git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build_

Käynnistä terminaali ja varmista seuraavan komennon avulla, että kaikki on kunnossa

- curl -fsSL https://github.com/rbenv/rbenv-installer/raw/main/bin/rbenv-doctor | bash

Komennon tulostuksen pitäisi näyttää suunilleen seuraavalta:

```
Checking for `rbenv' in PATH: /home/user/.rbenv/bin/rbenv
Checking for rbenv shims in PATH: OK
Checking `rbenv install' support: /home/user/.rbenv/plugins/ruby-build/bin/rbenv-install (ruby-build 20220610)
Counting installed Ruby versions: none
  There aren't any Ruby versions installed under `/home/user/.rbenv/versions'.
  You can install Ruby versions like so: rbenv install 3.1.2
Checking RubyGems settings: OK
Auditing installed plugins: OK
```

Siirry kohtaan [Rubyn ja railsin asennus](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/railsin_asentaminen.md#rubyn-ja-railsin-asennus)

## rbenv OSXlle

Rbenvin asennus onnistuu helpoiten homebrew:in avulla. Ohjeet homebrewin asennukseen löydät osoitteesta http://brew.sh/

Homebrewin asennuksen jälkeen

    brew update
    brew install rbenv
    brew install ruby-build

Lisää myös rivit

    export PATH="$HOME/.rbenv/bin:$PATH"
    eval "$(rbenv init -)"

Tiedostoon `.bash_profile` kotihakemistoosi. Voit mahdollisesti joutua luomaan sen. .-alkuiset tiedostot eivät oletuksena näy Finderissä.

Tiedoston luominen tapahtuu terminaalista käsin esim. seuraavasti

    cd
    nano .bash_profile

kopioi rivit tiedoston, tallenna tiedosto painamalla _control_ ja _o_, sulje se komennolla _control_ ja _x_.

Käynnistä tässä vaiheessa terminaali uudelleen.

**Huom:** joissain vanhoissa OSX:issä saattaa olla tarve tehdä em. rivien lisäys kotihakemistossa olevaan tiedostoon `.bashrc`

## Rubyn ja Railsin asennus

Kun _rbenv_ on asennettu, asennetaan ja määritellään käytettävä Ruby:n versio komennoilla

    rbenv install 3.1.2
    rbenv global 3.1.2

Komento asentaa Rubyn version 3.1.2, joka on Rubyn uusin versio. Voit tarkistaa asennettavissa olevat versiot komennolla <code>rbenv install --list</code>

Varmista, että komennon <code>which ruby</code> tulos on suunnilleen seuraava:

    /Users/kayttajatunnus/.rbenv/shims/ruby

Asennetaan sitten Rails antamalla komentoriviltä seuraavat komennot (vastaa mahdollisiin **Overwrite the executable?** -kyselyihin Y):

    echo 'gem: --no-ri --no-rdoc' >> ~/.gemrc
    gem install bundler
    gem install rspec
    gem install rake
    rbenv rehash
    gem install rails -v 7.0.4
    rbenv rehash
