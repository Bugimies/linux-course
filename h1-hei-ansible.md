
# h1-hei-ansible
## X) Tiivistelmä
* Julkisen avaimen luominen ja käyttäminen poistaa tarpeen salasanan syöttämiselle palvelimia hallitessa SSH yhteydellä.
* SSH on salattu yhteys palvelimien välisiin yhteyksiin.
* Ansiblen asennus ja käyttäminen, sillä voidaan tehdä kurssin ydintä eli infrastruktuuria koodina (Iac), eli hallita useita koneita. Se toimii SSH kautta.
* Ansiblea käyttävälle tilille määritetään salasanaton SSH autentikointi yhteyksien helpottamiseksi, jatkuvaa kijautumistietojen syöttämistä ei enää tarvita.

## A) SSH-demonin asentaminen
Aloitan 09:50 SSH-demonin asentamisen
En ole asentanut sudoa, joten komennolla ”su- ” mennään ensiksi ROOT käyttäjäksi.
Sshsecrets, asennetaan SSH-demoni ja testataan sitä kirjautumalla.
Ensiksi toki päivititetään järjestelmä 
apt-get update
Asennan SSH, sekä varmistan että se on päällä ja latautuu järjestelmän käynnistyksessä, komennot:

apt-get install openssh-server
systemctl enable --now ssh
systemctl status ssh

<img width="474" height="431" alt="image" src="https://github.com/user-attachments/assets/eed9fa58-84be-4163-9b57-7cc5577e8160" />

Testataan että SSH toimii, kirjaudun localhostiini.
ssh kalle@localhost

<img width="574" height="239" alt="image" src="https://github.com/user-attachments/assets/acc244d0-01a3-4f39-ac07-3328b6f1b029" />

yhteys muodostuu joten SSH näyttää toimivan.


## B) Julkisen avaimen luominen, kopiointi ja käyttäminen
Luodaan SSH julkinen avain.
ssh-keygen

Kopioidaan avain locahostiin niin saadaan testattua että authenticointi toimii.
ssh-copy-id localhost

<img width="398" height="272" alt="image" src="https://github.com/user-attachments/assets/2cb128d0-be28-4d72-b061-eb3cff46eda8" />

ssh yhteys avaimen kanssa.

<img width="940" height="277" alt="image" src="https://github.com/user-attachments/assets/8c03e8b9-22fb-42da-9bba-f3b0e63e1f1f" />
Salasanaa ei tarvita, eli kirjautuminen on automaattinen.

## C) Hei Ansible.
Asennetaan Ansible, sitä varten pitää olla sudo userina.
Komennolla su – päästään root käyttäjäksi hänen ympäristöönsä
Asennan Ansible, tree ja micro ohjelmat.
Ne auttavat tämän tehtvän teossa
apt-get install ansible
apt-get install micro
apt-get tree

Tämän voisi tehdä yhdellä komennolla mutta harjoittelun vuoksi teen vaikeammin. En ole itse aiemmin Debiania käyttänyt joten hieman komennot ovat uusia.
Kun nämä ovat asennettu tehdään ansible hakemisto, jonne sijoitetaan hosts.ini tiedosto. Tätä varten micro ohjelma asennettiin. 
olen kotihakemistossa. Komennoilla saa tehtyä asiat. hosts.ini tulee konfiguroida ansiblen yhteys tietoja mm. millä otetaan yhteys ja minne. Tiedostostoon ansible_connection=ssh, ansible_host=localhostansible all
mkdir ansible
cd ansible
micro hosts.ini
Kun teen hosts.ini ja kaikki tarpeellinen on sinne kirjoitettu se tallennetaan ctrl+s ja poistutaan ctrl+q. 

testataan ansiblen yhteys

ansible all-a ‘uptime’ -I hosts.ini

<img width="638" height="116" alt="image" src="https://github.com/user-attachments/assets/2d64d6d1-f79e-439b-848f-dfb07fb36753" />

Yhteys toimii

Luodaan ansible.cfg nini voidaan ajaa uptime ilman hosts.ini polkua.
micro ansible.cfg
Sen sisällöksi

<img width="396" height="108" alt="image" src="https://github.com/user-attachments/assets/e800b540-245f-44db-a3b2-f1c5112cb359" />

ansible all -a ‘uptime’


<img width="615" height="59" alt="image" src="https://github.com/user-attachments/assets/e215b845-15f7-4a43-bb9c-011dded2ca45" />

Luodaan playbookin pääohjaustiedosto site.yml
micro site.yml
Tässä tulee hieman haasteita että sisennykset ovat kunnossa.
Luodaan rooleille kansio ja myös muut tarvittavat kansiot

mkdir -p roles/hello/tasks
Luon roolille main.yml
micro roles/hello/tasks/main.yml

<img width="391" height="106" alt="image" src="https://github.com/user-attachments/assets/806f39d9-7b6a-4703-ba9a-69190d947eca" />

Ajetaan playbook
ansible-playbook site.yml

<img width="940" height="149" alt="image" src="https://github.com/user-attachments/assets/1d203cca-a988-4936-b391-105c9addf947" />

Minulla on käynyt kirjotusvirhe, kansio task pitäisi olla tasks
Korjaan sen komennolla
mv roles/hello/task roles/hello/tasks

<img width="494" height="196" alt="image" src="https://github.com/user-attachments/assets/c059a6b6-12af-4aeb-9c91-897580c86c5d" />

Ajetaan playbook site.yml uudestaan

<img width="940" height="196" alt="image" src="https://github.com/user-attachments/assets/a2e28c40-64f8-4340-a047-8645a09b35f6" />

Teen vielä idempotenssitestin, eli ajan playbookin uudestaan, näin nähdään ettei toinen ajo tee muutoksia.

<img width="940" height="230" alt="image" src="https://github.com/user-attachments/assets/a4d7f216-f5a0-414f-88e6-11b860bd124a" />

Testaan että mitä tapahtuu ssh yhteyden kautta. Komennolla.
ssh localhost ’cat/tmp/hello-ansible’

<img width="651" height="90" alt="image" src="https://github.com/user-attachments/assets/30e7dea5-b36c-49f1-9485-d4a6e20f47e4" />

Teksti printtaantuu!

## Lähteet
* Tero Karvinen 2026. Ansible ja sen konfaukset. https://terokarvinen.com/palvelinten-hallinta/#laksyt 

## Tekijätiedot
* Kalle Kyröhonka, Haaga-Helia Ammattikorkeakoulu, tietojenkäsittely
* Tätä dokumenttia saa kopioida ja muokata GNU General Public License (versio 2 tai uudempi) mukaisesti. http://www.gnu.org/licenses/gpl.html
* Pohjana Tero Karvinen 2026: Palvelinten hallinnan kurssi, http://terokarvinen.com 
