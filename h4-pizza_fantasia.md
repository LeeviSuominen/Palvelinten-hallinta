## x) Tiivistelmät
### Karvinen 2023: Configuration Management of Distributed Systems over Unreliable and Hostile Networks

#### 4.12.1 Size and Complexity of Some DSLs
- Konfiguraationhallinnan DSL kielet ovat isoja: Saltissa on 510 funktiota ja Puppetissa 113.
- Saltin dokumentaatio on yli 75 000 sanaa,
- Ominaisuuksien määrä vaihteli järjestelmittäin huomattavasti.
- Käytännössä suuri määrä ominaisuuksia ei tarkoita, että niitä kaikkia käytettäisiin aktiivisesti.

#### 4.12.2 Use of DSL Functions in Case Configuration
- Kaikkia DSL-kielten tarjoamia toimintoja käytetään käytännössä vain harvoin.
- Käytössä oli vain pieni osa funktioista. File, package, service ja exec olivat yleisimpiä.
- Tyypillinen kaava on package-file-service. Siinä asennetaan ohjelma, asetetaan asetustiedosto ja käynnistetään palvelu.

#### 4.12.3.1 Dependencies Between Main Functions
- Tärkeimmät funktiot liittyivät usein toisiinsa.
- Palvelun käynnistys riippui yleensä siitä, että paketti oli ensin asennettu ja asetustiedostot olivat kunnossa.
- Vain exec ja file muuttavat kohdejärjestelmää suoraan.
- Idempotenssi tarkoittaa, että muutos tehdään vain, jos tila ei ole jo oikea.


## a) Räpylä

Tehtävän tarkoituksena oli asentaa uusi demoni käsin. Valitsin demoniksi Caddyn, koska sitä ei ollut käytetty aiemmissa tehtävissä.

Asensin caddyn:

`sudo apt install caddy`

Ensimmäisellä yrityksellä Caddy ei käynnistynyt, koska Apache käytti edelleen porttia 80. Pysäytin Apache-palvelimen:

`sudo systemctl stop apache2`

Tän jälkeen käynnistin Caddyn: 

`sudo systemctl restart caddy`

Tarkistin palveluiden tilan:

`systemctl status apache2`

`systemctl status caddy`

Apache oli tilassa:

`inactive (dead)`

ja Caddy:

`active (running)`

<img width="970" height="553" alt="1" src="https://github.com/user-attachments/assets/262d5826-ba43-4ba0-bcc1-676dbde86535" />

Lopuksi testasin palvelun selaimella osoitteessa: `http://localhost`

Selain näytti Caddyn oletusivun.

<img width="1105" height="775" alt="2" src="https://github.com/user-attachments/assets/525d3877-512c-41dd-bd6f-97d9b5f8bbf1" />

## b) Automaatti 

Tehtävän tavoitteena oli automatisoida demonin asennus Ansiblella.

Loin uuden roolin:

`mkdir -p roles/caddy/tasks`

ja tiedoston: 

`micro roles/caddy/tasks/main.yml`

Roolissa käytettiin apt ja service moduuleita:

```
- apt:
    name: caddy
    state: present
    update_cache: yes

- service:
    name: caddy
    state: started
    enabled: true
```

<img width="377" height="217" alt="3" src="https://github.com/user-attachments/assets/a83594cc-b343-49cd-8613-2ff181d24eb2" />


Lisäsin roolin playbookiin ja suoritin sen:

`ansible-playbook site.yml -K`

Playbook suoritettiin onnistuneesti ilman virheitä.

<img width="1203" height="690" alt="4" src="https://github.com/user-attachments/assets/fb656f45-7d76-42a9-930c-c8e683ad8253" />

## c) Asetus
Tehtävän tavoitteena oli muuttaa asetustiedostoa masterilla ja osoittaa, että muutos tuli käyttöön.

Tarkistin ensin alkuperäisen Caddyn asetustiedoston:

`sudo cat /etc/caddy/Caddyfile`

<img width="742" height="542" alt="5" src="https://github.com/user-attachments/assets/d47b3f45-0e55-4fd0-9d32-3b7b275f813d" />

Loin tämän jälkeen uuden asetustiedoston masterille:

`micro roles/caddy/files/Caddyfile`

Sisältö:

```
:80 {
    respond "H4 toimii!"
}
```

<img width="368" height="97" alt="6" src="https://github.com/user-attachments/assets/b8ae4e4c-579b-4303-9a45-9cee419de35d" />

Muokkasin roolia siten, että uusi asetustiedosto kopioitiin palvelimelle:

```
- copy:
    src: Caddyfile
    dest: /etc/caddy/Caddyfile
    owner: root
    group: root
    mode: "0644"
```

<img width="476" height="403" alt="7" src="https://github.com/user-attachments/assets/bdd87c34-37dd-4470-9e4e-1c6d271ba90c" />

Suoritin playbookin uudelleen: 

`ansible-playbook site.yml -K`

Tämän jälkeen avasin selaimen osoitteessa: `http://localhost`

Selain näytti tekstin:

`H4 toimii!`

<img width="435" height="160" alt="9" src="https://github.com/user-attachments/assets/d1907947-62cf-4643-9c5b-5fd4463a1334" />

Tämä osoitti, että uusi asetus siirtyi onnistuneesti palvelimelle ja tuli käyttöön.

## d) Paikka remonttiin

Tehtävän tavoitteena oli rikkoa järjestelmä ja osoittaa, että Ansible pystyy palauttamaan sen toimintakuntoon.

Poistin Caddyn kokonaan:

`sudo apt purge caddy`

Tämän jälkeen tarkistin tilanteen: 

`systemctl status caddy`

Tuloksena oli:

`Unit caddy.service could not be found.`

<img width="850" height="418" alt="10" src="https://github.com/user-attachments/assets/07de4be6-9966-4163-8565-31a5b4547ef0" />

Suoritin tän jälkeen playbookin uudestaan:

`ansible-playbook site.yml -K`

Ansible asensi puuttuvan ohjelman takaisin sekä palautti asetukset.

<img width="1196" height="736" alt="11" src="https://github.com/user-attachments/assets/c7d7f571-39c1-4450-9ab6-ac3db9ec3677" />

Tarkistin lopuksi palvelun tilan:

`systemctl status caddy`

ja palvelu oli jälleen käynnissä.

<img width="1100" height="536" alt="12" src="https://github.com/user-attachments/assets/73276527-732b-472b-a925-6b639701df64" />

## e) Idempotentti

Tehtävän tavoitteena oli osoittaa, että playbook on idempotentti.

Ensimmäisellä yrityksellä käytin asetusta:

`state: restarted`

minkä vuoksi Ansible teki muutoksen jokaisella ajokerralla.

Muutin asetuksen muotoon:

`state: started`

jonka jälkeen suoritin playbookin uudestaan

`ansible-playbook site.yml -K`

Tuloksena oli

```
changed=0
failed=0
```

<img width="1122" height="715" alt="14" src="https://github.com/user-attachments/assets/fbf7709c-afc2-46ae-9528-ba68de670c03" />


Ansible ei tehnyt enää mitään muutoksia, koska järjestelmä oli jo halutussa tilassa.

Tämä osoittaa, että tila on idempotentti.

## Lähteet

Karvinen, T. 2023. Configuration Management of Distributed Systems over Unreliable and Hostile Networks. https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks. Luettu: 24.11.2026

Karvinen, T. 2023. Apache installed with Ansible - quick notes. https://terokarvinen.com/apache-ansible/. Luettu: 24.11.2026

Ansible Community Documentation. Handlers: running operations on change. Luettu: 24.11.2026.

Caddy Documentation. https://caddyserver.com/docs/. Luettu: 24.11.2026.
















