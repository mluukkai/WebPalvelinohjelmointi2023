Jos et ole käyttänyt aiemmin gitiä ja/tai githubia, seuraavat ohjeet auttavat alkuun

## Tunnus githubiin

Tee itsellesi tunnus GitHubiin
* mene osoitteeseen https://github.com/plans
* valitse create free account

Luo repositorio, esim. nimellä wadror 
* klikkaa yläpalkin oikeassa reunassa olevaa  "Create a new repo"-ikonia 
* **älä laita rastia** kohtaan Initialize this repository with a README 

Jos haluat, voit hakea akateemista tunnusta. Akateemisella tunnuksella saat käyttöösi (ilmaiseksi) yksityisiä repositorioita, normaalisti repositoriosi ovat julkisia 
* https://github.com/edu

Luo tarvittaessa paikalliselle koneellesi ssh-avain (tapahtuu komentoriviltä käsin)
* ks. ohje https://github.com/mluukkai/otm-2018/blob/master/tehtavat/viikko1.md#julkinen-avain

Lisää avaimen julkinen pari githubiin:
* https://github.com/settings/ssh

Näin pystyt käyttämään GitHubia ilman salasanan syöttämistä koneelta jossa juuri luodun avaimen salainen pari löytyy

Konfiguroi nimesi ja emailosoitteesi paikallisen koneesi git:iin antamalla komennot:

    git config --global user.name "Your Name"
    git config --global user.email my.address@gmail.com

Tee paikallsiella koneella olevasta Rails-sovelluksen sisältävästä hakemistosta git-repositorio antamalla hakemiston juuressa seuraavat komennot

    git init
    git add -A
    git commit -m"initial commit"

* toimi Githubiin luomasi uuden repositorion avausnäkymän kohdan *Push an existing repository from the command line* mukaan

## git-ohjeita

Kun olet tehnyt muutoksia koodiisi ja haluat synkronoida koneellasi olevan datan githubiin, riittää että suoritat seuraavat komennot sovelluksen hakemiston juuresta

    git add -A
    git commit -m"kirjoita tähän joku järkevä viesti, joka kertoo mitä repositorioon lisätään"
    git push origin master

## lisää gitiä

Jos git on uusi tuttavuus tai taidot ovat päässeet ruosteeseen, kannattaa käydä läpi Ohjelmistotekniikan menetelmien viikon 1 gitiin liittyvä [tehtäväsarja](https://github.com/mluukkai/otm-2018/blob/master/tehtavat/viikko1.md#versionhallinta)

Internetistä löytyy suuri määrä enemmän tai vähemmän hyviä git-tutoriaaleja. Seuraavassa muutama 

- https://www.atlassian.com/git/tutorials/
- http://learngitbranching.js.org
- http://ohshitgit.com
