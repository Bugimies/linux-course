# H2 Voileipä

### X)
1. Karvinen 2026
Sudo without password, sudo salasanan automatisointi on aikaa säästävä ja jopa tietoturvallisuus asia. Näin ei tarvitse, jokaiselle käyttäjälle omaa sudo salasanaa pitää yllä ja muistaa, usein kun on kymmeniä tai jopa satoja salasanoja ihminen rupeaa käyttämään samoja. Tulee kohdentaa tietyille rooleille tai komennoille ei koko käyttäjäryhmille.
2. Munroe 2006: xkcd 149: Sandwich
Sarjakuva jossa kuvataa ja havainnollistetaan loogista ajattelua, pitäisi käyttää helposti ymmärrettäviä esimerkkejä eikä aina vaikeita teknillisiä termejä.
3. Karvinen 2026: Passwordless Sudo with Ansible
Käytä visudo, visudo tarkastaa syntaksivirheet ja lukitsee suders tiedoston muokkauksen ajaksi, estää tällä tavalla yhtäaikaiset muokkaukset. Opastaa käytännön tasolla sudo-konfiguraation automatisointiin ansiblella. 
4.  ansible-doc copy
sisällön kopiointi ohjauskoneelta orjakoneelle. Tai luo content kentän avulla tiedoston suoraan. Content, Tiedoston sisältö suoraan playbookissa. Tukee monirivistä tekstiä.
Dest, destination kohdepolku orjakoneella. 
SRC, lähdetiedosto
owner, tiedoston omistaja
group, tiedoston ryhmä
mode, tiedoston oikeudet oktaalimuodossa.
- copy:
    src: foo.txt
    dest: /tmp/foo.txt
    owner: antero
    group: antero
    mode: '0600'
5. ansible-doc apt
Voidaan asentaa, poistaa, päivittää paketteja ja päivittää välimuisti. Hallinnoi paketteja
name, paketin nimi. State (present, latest tai absent) missä tilassa paketti on.
Update_cache, päivittää apt-välimuistin ennen toimenpidettä.
- apt:
    name: htop
    state: present
    update_cache: yes
6. ansible-doc file
varmistaa oikeudet, omistajuuden ja hakemiston olemassaolon. Voidaan poistaa tai luoda tiedostoja ja hakemistoja. Hallinnoi tiedostojen ja hakemistojen tilaa
path, kohdepolku
recurse, omistajuus ja oikeudet myös alihakemistoihin mikäli päällä (yes)
state, määrittelee mitä polulle tehdään
Owner, hakemistojen ja tiedostojen omistja
Group, tiedoston tai hakemiston omistajaryhmä
Mode, oideoikeudet oktaalimuodossa esim. 0755


7. ansible-doc user
Hallitsee käyttäjiä, voidaan luoda kotihakemiston, luoda käyttäjiä sekä lisätä ryhmiin.
- user:
    name: antero
    create_home: yes
    groups: "sudo,developers"
    shell: /bin/bash
    state: present
8. ansible-doc authorized_key
Käytetään generoimaan todennusavaimia, joilla voi automatisoida käyttäjien yhteyksien todentamista. Esimerkiksi ettei tarvitse käyttää salasanaa.
- authorized_key: 
       user: antero
key: “ssh-rsa AKKFKFfkfkf….” 
### A)
Sudoless
Tein itselleni tunnuksen kalleans sekä lisään käyttäjän sudo-ryhmään
avaimen kopiointi onkin tuttua edelisestä harjoitteesta niin en siitä nyt ota kuvia.
uuden tunnuksen luonti ja lisääminen ryhmään tapahtuu komennoilla
adduser kalleans
sudo adduser kalleans
Käytän samaa avainta kun toisella jo aktivoidulla käyttäjällä ja avaimen kopiointi
ssh-copy-id kalleans@localhost

<img width="940" height="318" alt="image" src="https://github.com/user-attachments/assets/74429c6a-a9e4-4eb3-a9a1-b8114004fa43" />

ssh kirjautuminen ilman salasanaa toimii.
Käyttäjä kalleans saadaan vapautettua sudo salasanan näppäilemisestä lisäämällä se sudo listalle ja määrittelemällä että ei tarvita salasanaa. 
sudo visudo
Päästään muokkaamaan turvallisesti sudo listaa, visudo tarkastaa syntaksivirheet ja lukitsee suders tiedoston muokkauksen ajaksi, estää tällä tavalla yhtäaikaiset muokkaukset. Sudoers tiedosto sijaitsee /etc/sudoers/ ja lisä konfiguraatiot /etc/sudoers.d/ (Sudo Manual).

<img width="542" height="303" alt="image" src="https://github.com/user-attachments/assets/8dbcd4c7-d190-4c82-9d65-b0e3d00a732a" />


 Poistin käyttäjän sudo listalta koska se saattaa aiheuttaa ristiriitaa. 
sudo gpasswd –-delete kalleans sudo 
Lisäsin visudoon Root kohtaan, tämä on se missä määritellään mikä käyttäjä, missä koneissa, millä tunnuksilla, älä kysy salasanaa ja mitä komentoja käyttäjä saa ajaa ilman salananaa.

kalleans ALL=(ALL) NOPASSWD:ALL

 ### B) Antero

Tee salasanaton automaattisesti ssh:lla kirjautuva tunnus Anseblella.
Tehdään rooli antero, sitä varten tulee tehdä roles hakemistoon uusi hakemisto ja sinne main.yml
mkdir -p roles/antero/tasks

 <img width="544" height="270" alt="image" src="https://github.com/user-attachments/assets/2911de45-fffc-43b7-aa43-e01992882589" />

Jotta tätä voidaan ajaa tarvitaan playbook. Teen antoro playbookin, 
micro antero.yml

<img width="325" height="245" alt="image" src="https://github.com/user-attachments/assets/47d6a574-474f-4339-a153-527abcabff10" />

Testataan ajamalla anto.yml playbook
ansible-playbook antero.yml

<img width="777" height="275" alt="image" src="https://github.com/user-attachments/assets/afe0094b-9749-4233-a2a0-3be1a3f86c7a" />

Ja sama ajo uudestaan, jotta varmistutaan ohjelma yritä tehdä uudestaan samaa.

<img width="731" height="259" alt="image" src="https://github.com/user-attachments/assets/343b32f5-f227-4711-a73d-24c9f590a338" />

Testataan anteron kirjautuminen
ssh antero@localhost

<img width="940" height="209" alt="image" src="https://github.com/user-attachments/assets/e28950dc-74cf-4e51-a4aa-0412be6eb4c2" />


Kirjauduin anteron ssh, salasanaa ei kysytty, joten ajamalla playbook todellakin syntyi käyttäjä antero ja ssh kirjautuminen on salasanaton.
### C)
Asennetaan kaksi pakettia, ei ole tarkemmin määritelty mitä asennetaan ansiblella. Asenann htop sekä 
Teen hakemistot kuten hello worldissä tähän htopille.
mkdir -p roles/htop/tasks
Luodaan main.yml
micro roles/htop/tasks/main.yml

<img width="727" height="227" alt="image" src="https://github.com/user-attachments/assets/56d53f4b-b397-4b28-8373-ee1752cade9c" />

Pitää tehdä myös playbook nimeän sen install.yml
micro install.yml
<img width="403" height="203" alt="image" src="https://github.com/user-attachments/assets/bcb8e9ba-cc3b-470f-b3f0-2d04e326782b" />

ajetaan testi 
ansible-playbook install.yml

<img width="522" height="258" alt="image" src="https://github.com/user-attachments/assets/ef9b1f7d-4590-4db2-abd4-62955661bef6" />

Virhe rivillä 2, sarakkeessa 9. väärä sisennys. Korjataan tämä ja ajaetaan playbook uudestaan.

<img width="940" height="303" alt="image" src="https://github.com/user-attachments/assets/e7289651-c44a-4d5d-84c6-16aad0486a1b" />

tiedostossa roles/htop/tasks/main.yml on myös sisennys virhe. Tässä on hyvä esimerkki että pitää testailla välissä niin saa ympäristön toimimaan ilman suuria vianselvityksiä. Käy ilmi että alussa  puuttuu välilyönti – sekä name välistä sekä become on väärin sillä se oli name sanan kanssa samassa niin sitäkin pitää siirtää.
ajetaan playbook

<img width="940" height="356" alt="image" src="https://github.com/user-attachments/assets/b3d3fab9-ee3c-49d0-b05a-9e3d742d2168" />

Paikallistan Co-pilotin kanssa virheeksi että roles/htop/tasks/main.yml on kirjoitus virhe. Olen kirjoittanut uodate_cache kun piti kirjoittaa update_cache. Korjataan ja ajetaan playbook uudestaan.

<img width="820" height="335" alt="image" src="https://github.com/user-attachments/assets/2db67124-bb9d-4891-82b7-07a645524adb" />

Ensimmäinen ajo onnistui, tulee testata idempotenssi ja ajaa toisen kerran ja toivoa ettei muutoksia tapahdu.

<img width="705" height="421" alt="image" src="https://github.com/user-attachments/assets/2d67073a-de6d-4a7a-ab9d-567159a5b83f" />

Toinen ajo toimi oikein sillä htop oli jo asennettu. Nyt kun htop on asennettu asennan toisen paketin, valitsen että se on curl koska se on käyttökelpoinen työkalu. Kokeilen yhdistää nämä samaan playbookiin niin ei tule kahta erillis asennusta. Yritykseni on että htop ei asennu uudelleen eli asennus olisi idempotenti.
micro packages.yml
<img width="383" height="332" alt="image" src="https://github.com/user-attachments/assets/36204d4d-2146-4d5b-97aa-ea22a0fd13bf" />

Ajetaan playbook packages.yml. Unohdin ottaa kuvankaappauksen, siinä oli virhe ja jälleen kerran sisennyksessä. 
<img width="336" height="359" alt="image" src="https://github.com/user-attachments/assets/c772fcb7-f231-4b59-8c0d-88cb4c6d5b69" />

 ajo uudemman kerran.
<img width="940" height="388" alt="image" src="https://github.com/user-attachments/assets/b49c1c2b-afdb-4d53-b6ef-7b7f0e3e8cc4" />

 Changed 2. Eli ajo päivitti cachen ja suoritti curl asennuksen.
 
### C)
Kirjoita orjalle useamman rivin mittainen tiedosto Ansiblella, sille tulee määritellä omistaja, omistaja ryhmä ja oikeudet. Käytetään oktaalinumeroa 0600. 
Aloitan tekemällä ansibleen tarvittavat hakemistot, joita pelikirjan tekoon tarvitaan.
mkdir -p  roles/filecopy/file/tasks

<img width="344" height="351" alt="image" src="https://github.com/user-attachments/assets/0dbda0e2-fac9-4056-aed3-04fee1f1200a" />

mirco roles/filecopy/files/foo.txt
mirco roles/filecopy/tasks/main.yml
<img width="450" height="312" alt="image" src="https://github.com/user-attachments/assets/49defb76-a7d7-49c0-9064-ef7b300cea86" />

sekä tehdään playbook filecopy.yml sen sisältö on
- hosts: all
  roles:
  - filecopy
Ajetaan juuri tehty filecopy playbook
<img width="632" height="249" alt="image" src="https://github.com/user-attachments/assets/2f509726-dc06-4bdf-968b-c08cfaebbc03" />
ja toisen kerran
<img width="460" height="275" alt="image" src="https://github.com/user-attachments/assets/12018f67-e08d-4f30-a8c0-5959b8e4b28b" />

Tarkastetaan että tiedosto on luotu.
<img width="670" height="144" alt="image" src="https://github.com/user-attachments/assets/d3c4b675-8267-4f89-a29d-7a448300f9a7" />

Kuvassa näkyy että ls- l se näkyy mutta en pääse katsomaan sitä koska annoin sen ROOTille. Omistaja sekä ryhmä on ROOTllä on oikeuksina -rw-------- eli omistaja voi lukea ja kirjoittaa mutta muut ei.

### E)
Jotain muuta, löysin moduulin CRON, joka on Linuxin ajastinpalvelu. Cron‑moduulilla voidaan hallita ajastettuja tehtäviä orjakoneella.
Tätäkin varten tulee tehdä uusi playbook. Teen service_test.yml.

<img width="331" height="250" alt="image" src="https://github.com/user-attachments/assets/729df755-24ac-429f-a30d-9efd13f9ec85" />
Ajetaan playbook

<img width="740" height="288" alt="image" src="https://github.com/user-attachments/assets/436a45e7-c604-43e2-823a-5bb55492f2d5" />

Lopetin tekemisen 22:35


## Lähteet:
Sudo Manual https://www.sudo.ws/docs/man/1.9.3/visudo.man/ : Viitattu 2.4.2026
Tero Karvinen https://terokarvinen.com/passwordless-sudo/ : Viitattu 2.4.2026
Tero Karvinen https://terokarvinen.com/palvelinten-hallinta/ 
Co-pilot LLM
