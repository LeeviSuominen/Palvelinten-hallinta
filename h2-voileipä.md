## x) Tiivistelmät
### Karvinen 2026: Sudo without password
- Sudoers tiedostoa käytetään siihen, että käyttäjälle voidaan antaa sudo oikeudet ilman salasanan kysymistä.
- `%sudoless ALL = (ALL) NOPASSWD: ALL` määrittää ryhmälle oikeuden käyttää sudoa ilman salasanaa.
- Muutokset kannattaa tehdä visudo-komennolla, joka tarkistaa tiedoston syntaksin.
#### Omat Huomiot
- Ryhmäpohjanen ratkaisu helpompi ylläpitää kuin käyttäjäkohtaisten sääntöjen lisääminen.

### Munroe 2006: xkcd 149 - Sandwich
- Sarjakuvassa käyttäjällä ei ole oikeuksia, mutta ratkaisuksi neuvotaan käyttämään sudoa.
#### Omat Huomiot
- Sudoa kannattaa käyttää harkiten, vaikka se välillä ratkaiseekin ongelmat.

### Karvinen 2026: Passwordless Sudo with Ansible
- Passwordless sudo voidaan automatisoida Ansiblella.
- Ansiblella voidaan luoda käyttäjiä, ryhmiä sekä sudoers-tiedostoja automaattisesti.
- Ensimmäisellä ajokerralla tarvitaan usein `--ask-become-password`, koska passwordless sudo ei vielä ole käytössä.
#### Omat Huomiot
- Ansible vähentää manuaalista työtä huomattavasti, kun sama määritys voidaan tehdä usealle koneelle kerralla.


### ansible-doc
#### ansible-doc copy
##### Johdanto
copy moduulia käytetään tiedostojen kopiointiin tai sisällön kirjoittamiseen kohdekoneelle.
- content = kirjoittaa annetun tekstin tiedoston sisällöksi.
- dest = kohdetiedoston sijainti.
- src = lähdetiedosto, joka kopioidaan kohteeseen.
- owner = tiedoston omistaja.
- group = tiedoston omistava ryhmä.
- mode = tiedoston oikeudet.

Esimerkki:
```
- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'
```

#### ansible-doc apt
##### Johdanto
apt moduulilla hallitaan Debian pohjaisten järjestelmien paketteja.
name = paketin nimi.
state = paketin tila.
update_cache = päivittää pakettivarastot.

Esimerkki:
```
- name: Remove "foo" package
  ansible.builtin.apt:
    name: foo
    state: absent
```

#### ansible-doc file
##### Johdanto
file moduulilla hallitaan tiedostojen ja hakemistojen ominaisuuksia.
path = polku tiedostoon.
recurse = käsittelee myös alihakemistot.
src = lähde linkeille.
state = tiedoston tila.
owner = omistaja.
group = ryhmä.
mode = oikeudet.

Esimerkki:
```
- name: Change file group using gid
  ansible.builtin.file:
    path: /etc/bar.conf
    group: "1000"
```

#### ansible-doc user
##### Johdanto
user moduulilla luodaan ja hallitaan käyttäjiä.
name = käyttäjän nimi.
create_home = luo kotihakemiston.
comment = käyttäjän kuvaus.
groups = käyttäjäryhmät.
shell = käytettävä komentotulkki.
state = käyttäjän tila.
system = järjestelmäkäyttäjä.

Esimerkki:
```
- name: Create a user 'johnd' with a home directory
  ansible.builtin.user:
    name: johnd
    create_home: yes
```

#### ansible-doc authorized_key
##### Johdanto
authorized_key moduulilla lisätään SSH-julkisia avaimia käyttäjän authorized_keys-tiedostoon.
user = käyttäjä.
key = lisättävä julkinen avain.

Esimerkki: 
```
- name: Set authorized keys taken from url
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: https://github.com/charlie.keys
```

## a) Sudoless
Tavoitteena oli luoda käyttäjä, joka voi käyttää sudoa ilman salasanan syöttämistä
Aloitin luomalla uuden käyttäjän

`sudo adduser antero`

Sitten loin uuden ryhmän 

`sudo groupadd sudoless`

Lisäsin käyttäjän ryhmään

`sudo adduser antero sudoless`

<img width="1282" height="890" alt="1" src="https://github.com/user-attachments/assets/8c4ebe33-3d24-4a63-af28-9f7919118847" />


Seuraavaksi loin sudoers säännön tiedostoon /etc/sudoers.d/sudoless Karvisen 2026: Sudo without password ohjeen mukaan:
`%sudoless ALL = (ALL) NOPASSWD: ALL`

<img width="1280" height="881" alt="2" src="https://github.com/user-attachments/assets/7d46e178-3351-426b-91c7-33eba5d8863a" />


Tämän jälkeen vaihdoin käyttäjäksi antero ja testasin passwordless sudon toiminnan:

`sudo -k`

`sudo echo "See you at TeroKarvinen.com`

<img width="628" height="262" alt="3" src="https://github.com/user-attachments/assets/5149aa92-2ea9-4fbe-b622-72cf0939d2bc" />


Komento suoritettiin ilman salasanakyselyä, joten määritys onnistui.


## b) Antero
Tehtävän tavoitteena oli automatisoida käyttäjän luominen sekä SSH avaimen lisääminen Ansiblella.

Aloitin luomalla uuden Ansible roolin komennolla:

`mkdir -p roles/antero/tasks`

Sitten loin tiedoston Karvisen 2026: Passwordless Sudo with Ansible ohjeen mukaan: roles/antero/tasks/main.yml ja kirjoitin sinne Ansible tehtävät, joiden tarkoitus oli:

- luoda ryhmä sudoless
- luoda käyttäjä antero
- lisätä käyttäjälle SSH-julkinen avain
- mahdollistaa sudon käyttö ilman salasanan syöttämistä

Ennen tätä jouduin hakemaan oman SSH avaimeni komennolla

`cat ~/.ssh/id_ed25519.pub`

<img width="1098" height="495" alt="4" src="https://github.com/user-attachments/assets/1761fcd8-83bf-480b-a779-a31575bbf526" />


Sitten muokkasin site.yml tiedostoa lisäämällä sinne anteron roolin, jonka jälkeen ajoin playbookin komennolla:

`ansible-playbook site.yml -K`

<img width="1018" height="503" alt="6" src="https://github.com/user-attachments/assets/78e55501-5b01-474a-8724-f48ea4a35993" />

## c) Package
Tehtävän tavoitteena oli asentaa kaksi pakettia Ansiblella.

Loin uuden roolin:
`mkdir -p roles/package/tasks`

Roolissa käytin apt moduulia

```
- apt:
    name:
      - tree
      - sl
    state: present
    update_cache: yes
```

<img width="427" height="252" alt="7" src="https://github.com/user-attachments/assets/347d0a19-ee97-4bd3-bcd5-10b7e5310823" />


Paketit asennettiin suorittamalla playbook

`ansible-playbook site.yml -K`

<img width="1040" height="722" alt="9" src="https://github.com/user-attachments/assets/66fa4c50-21bd-4a26-98a8-c50525067c3b" />


Asennuksen jälkeen testasin paketteja

`tree --version`

Testasin myös sl-pakettia, joka näyttää terminaalissa junan/veturin 

<img width="955" height="557" alt="10" src="https://github.com/user-attachments/assets/8b5b3091-165e-4da4-be76-b2fe0809ef5a" />


## d) File
Tehtävän tavoitteena oli luoda useamman rivin tiedosto ja määrittää sille omistaja, ryhmä sekä oikeudet.

Loin uuden roolin: 

`mkdir -p roles/file/tasks`

Roolissa käytin copy moduulia:

```
- copy:
    dest: /tmp/terveiset.txt
    content: |
      lebron
      james
    owner: antero
    group: antero
    mode: "0600"
```

<img width="396" height="256" alt="11" src="https://github.com/user-attachments/assets/67720b33-1cf1-4356-96e3-59c8f9fbcf21" />


Playbook suoritettiin:

`ansible-playbook site.yml -K`

Tämän jälkeen tarkistin tiedoston: 

`ls -l /tmp/terveiset.txt`

Tulosteen mukaan oikeudet olivat

-rw-------

Vastaa siis arvoa 0600

<img width="1030" height="755" alt="13" src="https://github.com/user-attachments/assets/6f3b7570-f6ae-46d8-92ac-ba7418ea7812" />

## e) Jotain muuta
Valitsin uudeksi moduuliksi service moduulin, jota ei ollut käytetty aikaisemmissa tehtävissä.

Loin uuden roolin:

`mkdir -p roles/service/tasks`

Roolin sisältö:

```
- service:
    name: apache2
    state: started
```
<img width="318" height="82" alt="14" src="https://github.com/user-attachments/assets/506f250c-a4f2-42a5-b53a-0ddfd2e4546b" />


Lisäsin roolin playbookiin ja suoritin sen:

`ansible-playbook site.yml -K`

Lopuksi tarkistin Apache palvelun tilan:

`systemctl status apache2`

Ja tilan mukaan Apache oli käynnissä:

`active (running)`

<img width="1022" height="722" alt="16" src="https://github.com/user-attachments/assets/61c66286-1307-4548-afb7-47f9a8c41891" />

## Lähteet
Karvinen, T. 2026. Sudo without password.
 https://terokarvinen.com/passwordless-sudo/

Munroe, R. 2006. xkcd 149: Sandwich.
 https://xkcd.com/149/

Karvinen, T. 2026. Passwordless Sudo with Ansible.
 https://terokarvinen.com/passwordless-sudo-with-ansible/

Ansible Documentation. ansible-doc copy

Ansible Documentation. ansible-doc apt

Ansible Documentation. ansible-doc file

Ansible Documentation. ansible-doc user

Ansible Documentation. ansible-doc authorized_key







