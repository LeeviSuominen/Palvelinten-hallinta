## x) Tiivistelmät
### Karvinen 2026: Apache installed with Ansible - quick notes
- Komennolla `sudo apt-get install apache2` ladataan apache.
- Apache palvelimen automatisointi perustuu package file service malliin
- Ensiksi asennetaan ohjelma, sitten muokataan asetustiedostoja ja lopuksi palvelu käynnistetään uudelleen.
- Jos asetustiedostoihin tulee muutoksia, Apache käynnistetään uudelleen Handlerien avulla.
- Jos haluat muokata apache tiedostoja -> `sudoedit /etc/apache2/...`

### Ansible Community Documentation: Handlers: running operations on change
#### Handlers: running operations on change
- Handler suoritetaan ainoastaan, jos on tapahtunut jotain muutoksia.
- Yleinen käyttötapa on esimerkiksi juuri asetustiedoston muutoksen jälkeinen uudelleenkäynnistys
- Jos ei ole muutoksia -> handleria ei suoriteta.

#### Notifying handlers
- Notify avainta käyttäessä tehtävä voi käynnistää yhden tai useamman handlerin.
- Sama handler suoritetaan vain kerran.
- Handlerit suoritetaan niiden määrittelyjärjestyksessä.

### ansible-doc service
#### Johdanto
- service moduulilla hallitaan palveluiden tilaa kohdejärjestelmässä.
- Moduulia voi käyttää esim palvelun käynnistämiseen, pysäyttämiseen tai uudelleenkäynnistykseen.

#### enabled
Määrittää käynnistyykö palvelu automaattisesti järjestelmän käynnistyessä.

#### name
Palvelun nimi.

#### state
Palvelun tila, esim: started, stopped.

#### ESIMERKKI:
```
- service:
    name: apache2
    state: started
    enabled: true
```

## y) Infra-uutiset
Valittu uutinen:  https://news.ycombinator.com/item?id=47865822
- Agent Vault on suunniteltu hallitsemaan salasanoja, API avaimia ja muita käyttöoikeuksia keskitetysti.
- Agent Vault on suunniteltu hallitsemaan salasanoja, API-avaimia ja muita käyttöoikeuksia keskitetysti.
- Järjestelmä toimii välikerroksena palveluiden ja salaisuuksien välillä, jolloin tunnuksia voidaan hallita turvallisemmin.
- Artikkelissa korostuu automaatio ja turvallisuus suurissa ympäristöissä

## a) Apassi
Tehtävän tavoitteena oli asentaa Apache käsin ja julkaista käyttäjän kotihakemistossa sijaitseva verkkosivu.

Olin jo aikaisemmin joskus asentanut apachen komennolla `sudo apt-get install apache2`, joten menin suoraan selaimeen ja kirjoitin osoitteeksi `http://localhost`.

Selain näytti default Apache2 sivun.

<img width="1266" height="828" alt="1" src="https://github.com/user-attachments/assets/52bee566-c4f4-475c-964d-a5e13bd7a4a5" />

Seuraavaksi loin käyttäjän kotihakemistoon uuden sivun: 

`micro ~/publicsite/index.html`

Määritin itse sivun sisällön.

<img width="386" height="230" alt="2" src="https://github.com/user-attachments/assets/99e4dcff-39cd-4e5b-b5a6-ff73618ab68e" />

Jotta Apache pystyi lukemaan tiedostot käyttäjän kotihakemistosta, määritin oikeudet:
```
chmod ugo+x /home/leevi
chmod ugo+x /home/leevi/publicsite
chmod ugo+r /home/leevi/publicsite/index.html
```

Tarkistin oikeudet ls komennolla

<img width="721" height="261" alt="3" src="https://github.com/user-attachments/assets/a02264e9-8d0a-4918-851f-0447bb2caaad" />

Apache asetettiin käyttämään uutta sivua muuttamalla DocumentRoot arvo:

`DocumentRoot /home/leevi/publicsite`

<img width="412" height="55" alt="5" src="https://github.com/user-attachments/assets/6290f2c3-c88b-46b5-b273-7298fdb5bdee" />

Lisäsin lisäksi Apache konfiguraatioon seuraavan:

```
<Directory /home/leevi/publicsite>
    Require all granted
</Directory>
```

Tämän jälkeen Apache sai oikeuden näyttää sivun.

<img width="803" height="722" alt="7" src="https://github.com/user-attachments/assets/f7729558-3532-4b9d-b892-1e05ef6022c3" />

Tarkistin asetusten oikeellisuuden:
`sudo apache2ctl configtest`

Tuloksena oli:

`Syntax OK`

<img width="1116" height="122" alt="6" src="https://github.com/user-attachments/assets/a63f1e94-c04d-4466-818a-0152e131b108" />

Lopuksi käynnistin Apache palvelimen uudelleen ja testasin sivun selaimessa.

<img width="551" height="350" alt="8" src="https://github.com/user-attachments/assets/d24124a8-16c8-46fd-ab2d-f7aa840f5c38" />

## b) Moottorix
Tehtävän tavoitteena oli asentaa Nginx käsin.

Ensin pysäytin Apache palvelimen, koska kaksi palvelinta ei voi käyttää samaa porttia samanaikaisesti.

`sudo systemctl stop apache2`

Tarkistin tilan:

`systemctl status apache2`

Palvelu oli tilassa:

`inactive (dead)`

<img width="952" height="281" alt="9" src="https://github.com/user-attachments/assets/5b76d61a-8a18-453d-b9cc-668046b5de1c" />

Asensin ja käynnistin Nginxin:
```
sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

Tarkistin tilan:

`systemctl status nginx`

<img width="1107" height="383" alt="10" src="https://github.com/user-attachments/assets/3d45155b-ef3e-48ba-90c0-38aafd3a3cce" />

Loin uuden sivun:

`micro ~/nginxsite/index.html`

<img width="447" height="263" alt="11" src="https://github.com/user-attachments/assets/9f8ea1c4-e745-417a-ac98-9f2ec65ed9ad" />


Määritin käyttöoikeudet: 

```
chmod ugo+x /home/leevi
chmod ugo+x /home/leevi/nginxsite
chmod ugo+r /home/leevi/nginxsite/index.html
```

<img width="706" height="88" alt="12" src="https://github.com/user-attachments/assets/01c14099-99fc-4db1-8bf9-92a27bd2271b" />

Muutin Nginxin juurihakemistoksi:

`root /home/leevi/nginxsite;`

<img width="540" height="157" alt="14" src="https://github.com/user-attachments/assets/3f1eb103-bdf1-4138-a0de-73aadb98a219" />

Tarkistin asetukset: 

`sudo nginx -t`

Tuloksena:

```
syntax is ok
test is successful
```

<img width="695" height="86" alt="15" src="https://github.com/user-attachments/assets/ec112d7c-671e-47e7-a0fb-e37921dd4cef" />

Lopuksi käynnistin Nginxin uudelleen ja testasin sivun selaimessa

<img width="513" height="332" alt="16" src="https://github.com/user-attachments/assets/1479cc71-6a4c-4668-bab1-491f7dad144c" />

## c) Automoottorix
Tehtävän tavoitteena oli automatisoida Nginxin asennus Ansiblella.

Loin uuden roolin:

`mkdir -p roles/nginx/tasks`

Roolissa käytin apt ja service moduuleita:

```
- apt:
    name: nginx
    state: present
    update_cache: yes

- service:
    name: nginx
    state: started
    enabled: true
```

<img width="366" height="211" alt="17" src="https://github.com/user-attachments/assets/625ab5b3-35c0-4798-b9ba-c3ef2c6daaf2" />

Lisäsin rooli playbookiin:

`- nginx`

<img width="306" height="218" alt="18" src="https://github.com/user-attachments/assets/9656da2c-b710-4f12-8b59-34bc9007b739" />


ja suoritin playbookin:

`ansible-playbook site.yml -K`

Suoritus onnistui ilman virheitä.

<img width="1112" height="737" alt="19" src="https://github.com/user-attachments/assets/281357ec-b5b1-44b4-87a5-d8fa499183c4" />

Tarkistin lopuksi Nginxin tilan:

`systemctl status nginx`

Nginx oli tilassa:

`active (running)`

ja käynnistyi automaattisesti järjestelmän käynnistyessä.


## Lähteet
Karvinen, T. 2026. Apache installed with Ansible - quick notes. https://terokarvinen.com/apache-ansible/

Ansible Community Documentation. Handlers: running operations on change. https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html

Hacker News. 2026. Agent Vault discussion. https://news.ycombinator.com/item?id=47865822

Infisical. 2026. Agent Vault: The Open Source Credential Proxy and Vault for Agents. https://infisical.com/blog/agent-vault-the-open-source-credential-proxy-and-vault-for-agents

Ansible Documentation. ansible-doc service

















