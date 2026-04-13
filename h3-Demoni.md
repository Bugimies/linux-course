Aloitetaan 19:00

#### X)
* Karvinen 2026: Apache installed with Ansible - quick notes
Apache voidaan konfiguroida ja asentaan täysin automaattisesti ansiblen avulla.
Tyypillisimmät moduulit Playbookissa ovat apt, copy, service ja template.

* Handlereita ei ajeta joka kerta, vasta silloin kun jokin tehtävä ilmoittaa niille muutoksesta,  usein palvelun uudelleen käynnistys aka potkaistaan demonia
Ne ajatetaan playbookin lopussa, jotta riittää vain 1 potkaisu.

##### Huomioni. Tämä estää turhat restartit ja tehostaa ajoa.

* Notifying handlers: kun tehtävä saa tilan changed, silloin handler saa herätteen. 
Liitetään tehvävään kun se voi auheuttaa muutoksen.
Sama handleri voi olla useassa taskissä, mutta se ajatetaan vain kerran.
##### Huomioni. Tämä tekee mielestäni playbookista älykkään, palvelu käynnistetään vain silloin kun se on tarpeellista.

Ansible-doc service
* service moduuli hallitsee palveluiden tilaa (start, stop, restart ja reload)
  tätä käytretään lähes jokaisessa palvelin palybookissa.
  
* Enabled
  yes/no määrittelee, käynnistyykö palvelu bootissa vai ei. Hyödyllinen mikäli halutaan estää käynnistys vahingossa. Esimerkiksi jos halutaan että Apache ei käynnisty koska Nginx on käynnissä.
  ##### Huomioni. Varmistaa että palvelu myös bootin jälkeen kuten haluttu.

* Name
Määrittää palvelun nimen.
Nimen pitää vastata järjestelmän palvelun nimeä esim. apache2 tai nginx.
##### Huomioni. Pienikin kirjoitusvirhe kaataa koko ajon. Kannattaa tarkastaa nimi ennen playbookin luontia.

* State
Määrittää palvelun tilan.
started, stopped, restarted, reloaded. 
state on idempotentti, mikäli palvelu on halutussa tilassa Ansible ei tee mitään.
##### Huomioni. Handlerissa on lähes aina restarted, jotta muutokset saadaan voimaan.

* Examples
  Käynnistä palvelu
- name: Start nginx service
  service:
    name: nginx
    state: started

Pysäytä palvelu
- name: Stop apache2 service
  service:
    name: apache2
    state: stopped

Uudelleenkäynnistä palvelu
- name: Restart nginx after config change
  service:
    name: nginx
    state: restarted

Lataa konfiguraatio uudelleen ilman restarttia
- name: Reload sshd configuration
  service:
    name: ssh
    state: reloaded

Estä palvelua käynnistymästä bootissa
- name: Disable apache2 autostart
  service:
    name: apache2
    enabled: no

Ota palvelu käyttöön bootissa
- name: Enable nginx autostart
  service:
    name: nginx
    enabled: yes

##### Huomioni. Esimerkit ovat hyödynnettävissä suoraan kopioimalla playbookiin. Voi helposti tehdä cheatSheetin itselleen.




#### A) Apassi

<img width="940" height="260" alt="image" src="https://github.com/user-attachments/assets/375e6c93-a695-4922-9bce-d3a51e202780" />


Asensimme tunnilla Apachen jo käsin. Joten oletan sen riittävän enkä tee siitä dokumentaatiota, se myös tarkastettiin tunnilla että se toimii.
Tässä Apache2 kotisivu, joka tunnilla oli nähtävillä

<img width="814" height="464" alt="image" src="https://github.com/user-attachments/assets/e259f992-b555-45bb-9631-c0bcaf114711" />


#### B) Moottorix

Asennetaan Nginx käsin, pitää muistaa että olemme asentaneet Apache2 joka käyttää porttia 80, sammutan sen varmuuden vuoksi ettei se sotke tätä tehtävää.
``` 
sudo systemctl stop apache2
``` 
Ennen asennusta suositeltavaa päivittää paketit. sen jälkeen asennetaan itse nginx komennot:
``` 
 sudo apt update
 sudo apt install nginx -y
``` 
<img width="577" height="437" alt="image" src="https://github.com/user-attachments/assets/e4af7097-0733-445c-bab1-96a67940feb3" />

Luodaan käyttäjän oma DocumentRoot
Tein hakemiston www koska tämä kertoo helposti mistä on kyse, se voisi olla toki muukin. Tein sen kotihakemistooni, jotta oikeudet on varmasti olemassa ja löytyy helposti.
``` 
mkdir -p /home/kalle/www
echo "Kalle kokeilee uutta" > /home/kalle/www/index.html
``` 
<img width="455" height="386" alt="image" src="https://github.com/user-attachments/assets/83d00cb5-752c-45c6-b4e3-48a12b3a2850" />

Tehtävänannossa on että sivusto tulee olla muokattavissa ilman root ja sudo oikeuksia, tästäkin syystä se on kotihakemistossa jolloin niit voi muokata kuten tehtävännannossa on määritelty.
luon tätä varten webedit ryhmän, jonka jäsenet voi muokata tätä tiedostoa.
``` 
sudo groupadd webedit
``` 

Vaihdetaan tämän jälkeen websivun hakemiston ja tiedoston ryhmäksi äsken luotu ryhmä, jotta ryhmäläiset voivat tehdä muokkauksia
``` 
sudo chgrp -R webedit /home/kalle/nginx_www
``` 

Ryhmälle tulee vielä määritellä kirjoitusoikeus
``` 
sudo chmod -R g+w /home/kalle/nginx_www
``` 

Tarkastetaan että oikeudet ovat nyt kunnossa.
``` 
ls -l /home/kalle/nginx_www/index.html
``` 
<img width="602" height="106" alt="image" src="https://github.com/user-attachments/assets/26b429d7-0ace-4713-9982-541aac51b63a" />


Jotta Nginx tietää mitä sivua halutaan näyttää tulee tehdä sille oma server block. Se kertoo mistä hakemistosta sivu löytyy, minkä niminen tiedosto on. Teen sen NANOlla ja lopuksi tallennan tuotoksen ctrl+o ja poistu ctrl+x
``` 
sudo nano /etc/nginx/sites-available/kalle
``` 
sisältö:

<img width="602" height="215" alt="image" src="https://github.com/user-attachments/assets/a35a7182-25d1-4dc2-89c3-0a6c61308c0a" />

Otetaan sivu käyttöön, sille tehdään symbolinen linkki sites-enabled hakemistoon komennolla
``` 
sudo ln -s /etc/nginx/sites-available/kalle /etc/nginx/sites-enabled/
``` 
Poistan default tiedoston jotta se ei sotke, olisi ehkä pitänyt ottaa varmuuskopio mutta tällä kertaa unohdin.
``` 
sudo rm /etc/nginx/sites-enabled/default
``` 
Testataan konfiguraatio
``` 
sudo nginx -t
``` 
<img width="602" height="120" alt="image" src="https://github.com/user-attachments/assets/6eec6624-293e-4273-ac6a-80ec63626926" />


Testi näyttää hyvältä. Ei tällä kertaa tarvetta lisä konfiguroinnille.

Varmistan vielä että NGINX on käynnissä.

<img width="602" height="222" alt="image" src="https://github.com/user-attachments/assets/ac87f742-7c4f-4645-a4d0-4d86004f0995" />
``` 
Käynnistetään NGINX uudestaan ns. ”potkaistaan demonia” (Tero Karvinen 2026).
``` 
sudo systemctl restart nginx
``` 
Tämä ei tuonut mitään ruudulle, joten ei virheitä. Palvelun tulisi nyt toimia.
Testataan sitä. Virtuaalikoneen omalla internet selaimella, localhost tulee testata sen koneen selaimella koska tosen koneen localhost näyttää sen koneen localhost. 
``` 
http://localhost
``` 

<img width="369" height="173" alt="image" src="https://github.com/user-attachments/assets/9ea07a25-0563-4ce4-a957-514218e565a0" />

Kuvasta voi huomata että localhost on päivittynyt tässä tehtävässä olevaan index.html
Asennus toimi



#### C) Automoottorix: NGINX-automatisointi Ansiblella


Luodaan playbookille omat hakemistot
``` 
mkdir -p automoottorix/files
mkdir -p automoottorix/templates
``` 
<img width="451" height="211" alt="image" src="https://github.com/user-attachments/assets/ec1038cb-3839-40ab-abd1-bd657a7b10ba" />


Luodaan kalle.conf tiedosto
``` 
nano files/kalle.conf
``` 
<img width="562" height="286" alt="image" src="https://github.com/user-attachments/assets/ff8bfa0d-fb32-4149-a88c-d13b914b58b2" />


Luodaan ionvetory tiedosto eli hosts, jonne kirjataan ryhmän nimi sekä kohde (localhost). 

[webservers] localhost ansible_connection=local
``` 
nano hosts
``` 
Tehdään playbook.yml
``` 
nano playbook.yml
``` 
Sisältö:
<img width  ="312" height="268" alt="image" src="https://github.com/user-attachments/assets/ec2c60d6-34cb-41e9-a53a-d07a7bf62922" />

Ajetaan playbook.
``` 
ansible-playbook -i hosts playbook.yml --ask-become-pass
``` 
<img width="364" height="315" alt="image" src="https://github.com/user-attachments/assets/c151fed8-9f89-469e-9775-fa56234bff43" />

Playbook suorittui hyvin. Teen vielä käsin HTML sivun jotta saamme sisältöä.
Alussa tein hakemiston projektille se on /home/kalle/nginx_www käytän siis sitä.
Jotta websivut voidaan saada tehtyä tulee luoda index.html tiedosto.
``` 
nano /home/kalle/nginx_www/index.html
``` 
<img width="453" height="193" alt="image" src="https://github.com/user-attachments/assets/284aafc8-7903-4183-98fd-4e4f08df6742" />

sivulle pitäisi siis tulla teksti. ”Tämä sivu on asennettu Ansible-playbookilla ja tehty käsin. ”
Haluan varmistaa että oikeudet ovat Ok. Se onnistuu komennolla
``` 
ls -l /home/kalle/nginx_www
``` 
<img width="602" height="86" alt="image" src="https://github.com/user-attachments/assets/5ee36884-bae5-4960-9ca5-04ae24fd938f" />

Koska edellisessä tehtävässä oli myös sama demoni niin disabloin vanhan sivun, automoitu sivu näkyy.

<img width="602" height="86" alt="image" src="https://github.com/user-attachments/assets/0002056f-33ed-4db3-98dd-79b54f3c9c42" />

Poistan symlinkin vanhaan

```
sudo unlink /etc/nginx/sites-enabled/kalle
```


<img width="602" height="75" alt="image" src="https://github.com/user-attachments/assets/d78b64f1-ee4b-47dc-8bb0-249542ab2886" />

Enää on jäljellä kalle.conf Let’s kick it and test it.
```
sudo systemctl restart nginx
```


<img width="378" height="204" alt="image" src="https://github.com/user-attachments/assets/3524583e-83ea-40a1-8ce1-2a9a965e63a9" />

Toimii!!



#### Lähteet:
•	Karvinen 2026: Apache installed with Ansible - quick notes
•	Ansible Community Documentation: Handlers: running operations on change
•	Nginx: Linux packages https://nginx.org/en/linux_packages.html 
•	Nginx Documentation https://nginx.org/en/docs/ 
•	Debian wiki: https://wiki.debian.org/Nginx 
•	ComputingForGeeks: Your first Ansible playbook https://computingforgeeks.com/ansible-playbook-tutorial/ 
Käytetty apuna CO-pilot neuvojen kysymiseen










