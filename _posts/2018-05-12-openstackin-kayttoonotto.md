---
ID: 487
post_title: OpenStackin käyttöönotto
author: jussi
post_excerpt: ""
layout: post
permalink: >
  http://jisosomppi.me/2018/05/12/openstackin-kayttoonotto/
published: true
post_date: 2018-05-12 11:39:23
---

# OpenStack-pohjainen pilvitallennusratkaisu
**Projektimme on osa Haaga-Helian Pilviteknologiat-kurssia keväällä 2018**

**Jussi Isosomppi, Samuli Kinnunen, Mikko Knutas, Eino Kupias, Saku Kähäri**

###### **DevStack asennettu alusta loppuun uudestaan `16` kertaa**

## Mitä, Miksi
Projektimme tavoitteena on alustavasti ollut pystyttää koulun palvelimelle oma pilvipalvelu (Openstack) ja luoda sinne eri käyttöjärjestelmillä toimivia virtuaalikoneita. Tämä saattaa vielä kestää tovin, joten ei mennä vielä sen pidemmälle.

Openstack on kokoelma eri työkaluja julkisten ja yksityisten pilvipalveluiden rakentamiseen.
Pilvipalveluiden tarkoituksena on tarjota resursseja asiakkaan käytettäväksi etänä. Tämä tarkoittaa että softa toimii palveluna luotettavalla ja helposti skaalautuvalla serverillä, eikä asiakkaan / käyttäjän omalla tietokoneella. Openstack voidaan siis sijoittaa IaaS -kategoriaan (Infrastructure as a service).

Openstack on avoimen lähdekoodin ohjelma, joten kuka tahansa voi tarkastella sen eri osia ja tehdä niihin haluttavia muutoksia tai lisätä kokonaan uusia osia.
Sen vakiokokoonpano koostuu seuraavista osista:

* **Nova** on pääasiallinen laskentaa suorittava ohjelma OpenStackin taustalla. Se vastaa laskentatehon jakamisesta instansseille ja OpenStackin muille moduleille.
* **Swift** on OpenStackin ylemmän tason tiedostojenhallintakomponentti, joka vastaa kaiken OpenStackin käyttämän tiedon tallennuksesta, varmistuksesta ja varastoinnista (myös erillisten fyysisten palvelinten välillä).
* **Cinder** on Block storage -komponentti, jonka kautta instansseille annetaan käyttöön levytilaresursseja.
* **Neutron** on OpenStackin verkkoyhteyspalvelu, joka mahdollistaa modulien ja instanssien kommunikoinnin keskenään sekä ulkomaailman kanssa.
* **Horizon** tarjoaa graafisen (selain-) käyttöliittymän OpenStackiin.
* **Keystone** käytetään käyttäjien ja käyttöoikeuksien hallintaan.
* **Glance** mahdollistaa levykuvien ottamisen ja niiden käyttämisen uusien instanssien pohjana.

## Konsepti / virtuaalitoteutus

Projektimme tarkoituksena on luoda useasta nodesta koostuva OpenStack -asennus, jossa on huomiotu tuotantotason ratkaisun vaatima tietoturva ja vikasietoisuus. Loppuratkaisun tulisi olla käytännössä yrityksen käyttöön soveltuva, "valmis" ratkaisu.

### Virtuaalikoneen luominen CloudPlatformiin

Päätimme toteuttaa projektin ensimmäisenä vaiheena virtuaaliasennuksen, jotta pääsisimme tutustumaan tarkemmin OpenStackin toimintaan. Asensimme OpenStackin kehittäjäversion, DevStackin, Haaga-Helian CloudPlatform -alustalle. Virtuaalikoneen isäntäkäyttöjärjestelmänä toimii Ubuntu Server 16.04.3 LTS, johon on asennuksen jälkeen tehty tarvittavat ohjelmisto- ja turvallisuuspäivitykset sekä asennettu vaadittavat ohjelmat. Loimme tarvittavat palomuurisäännöt CloudPlatformiin, jonka jälkeen toisesta koneestasai otettua SSH-yhteyden suoraan virtuaalikoneeseen hitaan selainkonsolin käyttämisen sijaan.

### DevStackin asennus

DevStackin asennus onnistuu helposti seuraamalla [virallisia asennusohjeita](https://docs.openstack.org/devstack/latest/), kunhan muistaa vaihtaa asetustiedostoon turvallisemmat salasanat *secret*in tilalle. Omalle kotipalvelimelle asentaminen sujui suoraan ohjeiden mukaan, mutta koulun CloudPlatform -alustassa asennus keskeytyi, kun yhteyttä GitHubiin ei voitu asennusskriptin aikana muodostaa. Syynä tähän ovat ilmeisesti koulun verkon palomuurin tai proxyn asetukset, jotka estävät yhdistämisen git:// -alkuisiin osoitteisiin. Kyseiset osoitteet voi onneksi muuttaa perinteisiksi https:// -osoitteeksi antamalla komento `git config --global url."https://".insteadOf git://`, joka korjasi ongelman heti seuraavalla asennusyrityksellä.

DevStackin asennusskriptin ajaminen CloudPlatformiin kesti n. 27min, jonka aikana asennusskripti latasi ajan tasalla olevat versiot DevStackin sisältämistä OpenStack -moduleista.

DevStackin asennuttua teimme sen vaatimat port forwardit CloudPlatformiin, jonka jälkeen OpenStackin graafinen käyttöliittymä Horizon löytyi selaimella.

```
CloudPlatform IP: 10.207.3.232
CloudPlatform VM SSH: 10.207.3.232:1001
DevStack Horizon (GUI): 10.207.3.232:1002
```

### DevStackin käyttökokeilu CloudPlatformissa

Teimme tämän jälkeen jokaiselle ryhmän jäsenelle omat käyttäjätunnukset, ja varmistimme niiden toimivan kunkin omalla selaimella. Näin varmistuimme port forwardien ja muiden verkkoasetusten sekä DevStackin verkkohallintaliittymän toimivuudesta. Yritimme myös luoda jokaiselle omaa virtuaalikonetta (instanssia), mutta valitettavasti CloudPlatform-virtuaalikoneestamme loppui ilmeisesti muisti kesken. CloudPlatformissa olemassa olevan virtuaalikoneen laitteistokokoonpanoa voi kyllä muuttaa, mutta vain sammuttamalla virtuaalikoneen ja vain ennalta määräitettyihin kokoonpanoihin. Tässä yhteydessä edes CloudPlatformiin määritelty *Large Instance* ei riittänyt tarpeisiimme, joten totesimme CloudPlatform-kokeilun lähestyvän loppuaan.

Totesimme lisäksi myös, että DevStack-instanssien koekäyttö olisi turhan vaikeaa, koska niille olisi pitänyt määritellä peräkkäisiä port forward -sääntöjä sekä CloudPlatformin että DevStackin virtuaalireitittimiin. Tämän lisäksi CloudPlatform sallii yhdelle projektille vain rajallisen määrän port forwardeja (10kpl?), eikä se olisi pidemmän päälle riittänyt tarpeisiimme.

## Asennus fyysiselle palvelimelle

Siirryimme projektin seuraavaan vaiheeseen toteuttamalla asennuksen fyysiselle palvelimelle koulun Servulassa. Kyseinen palvelin on HP:n ProLiant DL360p Gen8, jonka 2x Xeon E5-2603 -prosessoria ja 132Gb muistia riittänevät projektiin paremmin kuin CloudPlatformin tarjoamat. Palvelimeen on asennettu 8kpl 146Gb 15K RPM SAS-levyjä, jotka riittävät niin nopeudeltaan kuin kapasiteetiltaankin projektiimme hyvin. Tyhjensimme palvelimen kovalevyt, ja siirryimme käyttöjärjestelmän asennukseen.

### Käyttöjärjestelmän asennus

Asensimme palvelimelle DevStackin pohjaksi suositellun Ubuntu Server 16.04.4 LTS:n, jonka paketit päivitettiin heti asennuksen jälkeen uusimpiin. Otimme välittömästi käyttöön myös palomuurin (`sudo ufw allow 22/tcp &amp;&amp; sudo ufw enable`) ja poistimme root-tunnuksen kokonaan käytöstä (estimme salasanalla kirjautumisen sekä käytön SSH-yhteyden yli). Kokeilimme SSH-yhteyden toimivuuden sekä labraluokan 5004 koneilta että omilta läppäreiltä (Tielab -WLAN on osa labraverkkoa).

### Verkkoasetukset

Jouduimme taistelemaan jonkin aikaa koulun labraverkon asetuksien kanssa, mutta tuloksena saimme omat osoitteet sekä itse palvelimelle, että sen iLO-etäkäyttöliittymälle. iLO:n kautta voimme hallita etänä koko palvelinta, emmekä vain sen käyttöjärjestelmää. Määritimme iLO:on projektia varten uuden käyttäjän, jonka avulla voimme esimerkiksi käynnistää palvelimen miltä tahansa labraverkon koneelta. Verkkoyhteyden toimivuus tarkistettiin yksinkertaisesti asentamalla Apache2, avaamalla sille palomuurista portti 80 ja toteamalla placeholder-sivun näkyvän muillakin koneilla.

```
Palvelimelle varatut IP:t: 172.28.230.6-10
iLO:n etäkäyttö-IP: 172.28.230.6
Palvelimen GUI IP: 172.28.230.7
Instanssien exit point: 172.28.230.8
```

### Palvelimen levyasetukset

Yritimme muuttaa palvelimen levyasetuksia niin, että näkisimme sen kovalevyt itsenäisinä laitteina käyttöjärjestelmässä. Ideana oli käyttää hardware RAIDin sijaan ZFS tiedostojärjestelmää, jolloin olisimme saaneet itse päättää levyjen pariteettitiedot. Valitettavasti HP:n Smart Array -RAID-kortti ei kuitenkaan tue enempää kuin kahta loogista levyä ilman lisämuistin asennusta.

Lopullinen levyratkaisu oli seuraava:

* 2x 146Gb levyä RAID 1 -tilassa (mirror) -&gt; käyttöjärjestelmä (136Gb formatoituna)
* 6x 146Gb levyä RAID 1 -tilassa (mirror) -&gt; tallennustila (410Gb formatoituna)

### DevStackin asennus

DevStackia asentaessa kohtasimme taas pari ongelmaa:
* Unohdimme ajaa tuon aiemmin mainitun komennon, joka muuttaa git:// -osoitteet https:// -osoitteiksi
* ZFS-asennusta yrittäessä lisätty apt-repository oli jollain tavalla puutteellinen, jonka takia DevStack antoi virheilmoituksen ja keskeytti asennuksen
* DevStackin local.conf:issa annettu salasana sisälsi ääkkösiä, joka johti asennuksen virheilmoitukseen *"Not UTF-8 compliant"*

Nämä ongelmat korjattuamme saimme taas DevStackin toimimaan samalla tavalla, kuin aiemminkin. Tässä vaiheessa kuitenkin huomattiin, että palvelimelle oli jäänyt väärä IP-osoite.

#### IP-osoitteen vaihto

Jouduimme vaihtamaan palvelimen IP-osoitteen yhdeksi allokoiduista osoitteista, joka taas puolestaan sekoitti DevStackin toiminnan kokonaan. Hallintaliittymä Horizon latautui normaalisti, mutta sisäänkirjautuminen ei enää onnistunut ollenkaan. Syyksi paljastui parin Google-haun jälkeen se, että DevStackin asennuksessa käytetään palvelimen paikallista IP-osoitetta `localhost`in sijaan, jotta asennus tukisi sellaisenaan myös useamman noden kokoonpanoja. DevStackiin aiemmin kuulunut `rejoin-stack.sh` -skripti on valitettavasti poistettu, joten enää ei ole keinoa sammuttaa stackia, muuttaa sen asetuksia ja käynnistää sitä uudelleen. Jouduimme siis koneen IP-osoitteen vaihdon jälkeen ajamaan koko DevStackin asennusprosessin uudelleen.

```
Oikeat IP-asetukset (lisättiin /etc/network/interfaces):

auto eno1
iface eno1 inet static
address 172.28.230.7
netmask 255.255.0.0
network 172.28.1.0
broadcast 172.28.1.255
gateway 172.28.1.254
dns-nameservers 8.8.8.8

auto eno2
iface eno2 inet static
address 172.28.230.8
netmask 255.255.0.0
network 172.28.1.0
broadcast 172.28.1.255
gateway 172.28.1.254

```

### DevStackin käyttöönotto ja ensimmäinen kokeilu

DevStackiin lisättiin taas ryhmän jäsenille omat käyttäjätunnukset, ja määritettiin pääsy vakiona mukana tulevaan demo-projektiin. Omien ISO-tiedostojen lisääminen onnistuu helposti, ja uusien instanssien luominen on helppoa valmiiden laitteistokokoonpanojen ansiosta.

Instanssien käyttöönotto helpottui huomattavasti, kun avattiin portti `6080` NoVNC-virtuaalikonsolia varten. Tämän jälkeen selainkäyttöliittymän virtuaalikonsoli toimi moitteettomasti, ja yllättävän pienellä viiveellä. Käyttöjärjestelmien asennus onnistuu tämän jälkeen normaalisti. Jokaiselle instanssille on saatavilla ainoastaan sen laitteistokokoonpanon (flavor) mukaisesti levytilaa.

### Ohjelmistopäivitysten hitaus

Syystä tai toisesta ohjelmistojen päivitys ja asennus (`apt-get` -komennot) toimivat palvelimellamme todella hitaasti DevStackin asennuksen jälkeen (DevStackin virtuaalireititin vaikuttaa ehkä tähän?). Ongelma kuitenkin syntyy ainoastaan IPv6-yhteyttä käyttäessä, joten siitä pääsee eroon pakottamalla käyttöön IPv4-osoitteet päivityksiin. Tämä onnistuu lisäämällä `apt-get` -komentoihin tarkennus `-o Acquire::ForceIPv4=true`.

### Verkkoasetukset, osa 2
Koulun labraverkko itsessään ei tuottanut projektille juurikaan ongelmia, vaan lähinnä sen tarkasti määritelty IP-avaruus. DevStack ei tue ulkoisen DHCP-palvelimen käyttöä, vaan tahtoo itse jakaa julkiset, "oikeat" IP-osoitteet instansseilleen. Tästä johtuen jouduimme taistelemaan melko pitkään sopivien asetusten kanssa, ettemme sekoittaisi muun labraverkon toimintaa omilla virtuaalireitittimillämme. OpenStackin virtuaalireitittimen _ei pitäisi_ vastata ulkopuolisiin DHCP-pyyntöihin, mutta lähestymme asiaa mielummin varovasti, koska emme voi tätä varmistaa.

## Instanssien käyttöönotto

OpenStackiin on helppo luoda erilaisia virtuaalikoneita (instanssi, instance) ennakkoon määriteltyjen laitteistokokoonpanojen (flavor) mukaan. DevStackin asennuksessa tulee jo valmiiksi mukana kattava valikoima kokoonpanoja erilaisilla prosessori-, muisti-, ja levytilamäärittelyillä. Tämän lisäksi riittävillä oikeuksilla varustettu käyttäjä voi luoda myös mukautettuja kokoonpanoja omien tarpeidensa mukaan.

Kohtasimme paljon ongelmia juuri instanssien käyttöönotossa, kun syystä tai toisesta emme saaneet virtuaalikoneita toimimaan. Syistä selkeimpiä olivat:

* väärän tyyppiset levykuvat (OpenStack toimii parhaiten QCOW2-levykuvien kanssa)
* QCOW2-imaget ovat täydellisiä levykuvia, joten niitä joutuu käsittelemään toimivuuden varmistamiseksi. Tähän käytetään yleensä Guestfish-työkalua, jolla voidaan mm. poistaa verkkokortin MAC-osoite ja vaihtaa käyttäjätunnukset imageen.
* levytilan loppuminen kesken
* Lukuisista yrityksistä huolimatta emme saaneet määriteltyä palvelimen toista loogista levyä DevStackin käyttöön, joten käytössämme oli vain 146Gb levytilaa. Tämä levytila loppui pariin kertaan kesken, kun instansseja poistettiin, mutta virtuaalilevyt jäivät vielä talteen.

## 23/04 tapahtunutta

Päätimme asentaa koko serverin uudestaan. Ensimmäisellä kerralla tikkua ei voitu lukea ja toisella yrityksellä vika löytyi ilmeisestikin tikun asennusohjelmasta. Serveriä asentaessa virheilmoitukset viittasivat siihen, että Unetbootin ei olisi kaikkein luotettavin ohjelma kyseiseen tarkoitukseen.
Kolmas yritys on käynnissä. Tällä kertaa tikun luomiseen käytetty Rufusta ja itse tiedosto on edelleen sama (Ubuntu Server 16.04.4 LTS). Päätimme tarkastaa tikun viottuneisuuden varalta, mikä vei lähes tunnin.
