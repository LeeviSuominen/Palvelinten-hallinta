x) SSH public key
- asennus onnistuu komennolla: sudo apt-get -y install ssh
- sudo systemctl enable --now ssh käynnistää demonin heti ja myös bootissa
- ssh-keygen luo avainparin
- ssh-copy-id localhost lisää julkisen avaimen ~/.ssh/authorized_keys-tiedostoon
- julkisen avaimen käyttö on turvallisempaa kuin salasanakirjautuminen

Oma huomio: Julkisella avaimella kirjautuminen nopeuttaa työskentelyä huomattavasti

  Hello Ansible
- ansible on agentiton, orjakoneella tarvitaan vain SSH ja Python
- hosts.ini toimii inventaariona
- ansible all -a 'uptime' testaa yhteyden, ansible-playbook site.yml ajaa roolin.
- playbookeilla voidaan määrittää haluttu lopputulos, jonka Ansible toteuttaa automaattisesti.

Oma huomio: Ansiblen avulla saman asetuksen voi tehdä usealle palvelimelle yhdellä komennolla

a) Linkki raporttiin: https://github.com/LeeviSuominen/Palvelinten-hallinta/blob/main/h0-Oikeasti-netiss%C3%A4.md

b) Sshecrets: SSH-demoni käsin
Käytin ohjeena Tero Karvisen 2026 "SSH public key - Login without password" ohjetta https://terokarvinen.com/ssh-public-key-login-without-password/.
Aloitin tehtävän komennoilla:
sudo apt-get update (päivittää pakettiluettelon)
sudo apt-get -y install ssh (asentaa OpenSSH palvelimen ja -y vastaa kyllä kysymyksiin)
sudo systemctl enable --now ssh (käynnistää demonin heti ja myös bootissa)
ssh localhost (kirjautuu omalle konelle SSH:lla)
w (näyttää kirjautuneet käyttäjät)
exit (sulkee SSH istunnon)

c) Sitten kun olin varmistunut siitä, että SSH on latautunut ja toimii oikein oli aika luoda avainpari, jotta voin kirjautua julkisella avaimella. Siihen käytin
komentoja:
ssh-keygen (luo avainparin)
ssh-copy-id localhost (kopioi julkisen avaimen tiedostoon)
ssh localhost (kirjautuu nyt ilman salasanaa)

d) Tässäkin tehtävässä seurasin Karvisen ohjetta, joka löytyy täältä https://terokarvinen.com/hello-ansible/
Aloitin tehtävän lataamalla Ansiblen, sekä sen apuohjelmat:
sudo apt-get install ansible micro bash-completion tree (asentaa Ansiblen ja apuohjelmat)
mkdir ansible (luo työhakemiston)
cd ansible (siirtyy työhakemistoon)
Tämän jälkeen loin ansible työhakemiston sisälle hosts.ini tiedoston, johon määritin hallittavan koneen sekä Python tulkin sijainnin:
localhost

[all:vars]
ansible_python_interpreter=/usr/bin/python3

Testasin Ansiblen toiminnan komennolla: ansible all -a 'uptime' -i hosts.ini
Komento suoritti uptime-komennon localhostilla ja palautti onnistuneen vastauksen, joten Ansible pystyi käyttämään SSH-yhteyttä.

Seuraavaksi loin asetustiedoston:
micro ansible.cfg
Määritin sen sisällöksi vastaavan:
[defaults]
inventory = hosts.ini
Tämän ansiosta hosts.ini-tiedostoa ei tarvinnut enää määrittää jokaisen komennon yhteydessä.
Sitten oli aika luoda playbook komnnolla: micro site.yml ja sisällöksi:
- hosts: all
  roles:
    - hello
 
Seuraavaksi loin hello roolin: mkdir -p roles/hello/tasks ja roolin tehtävän: micro roles/hello/tasks/main.yml ja sen sisällöksi:
- copy:
    dest: /tmp/hello-ansible
    content: "See you at TeroKarvinen.com!\n"

Playbook suoritettiin komennolla: ansible-playbook site.yml
Ansible loi tiedoston /tmp/hello-ansible määritellyllä sisällöllä. Lopuksi tarkistin tiedoston sisällön: cat /tmp/hello-ansible
Tuloksena näkyi: See you at TeroKarvinen.com!

Ohjeita seuraamalla tehtävä oli suhteellisen simppeli. Kuvat minulla on erillisessä kansiossa "Picture", jos niitä kiinnostaa katsoa.

Lähteet: 
Karvinen, T. 2026. SSH public key - Login without password. Luettavissa: https://terokarvinen.com/ssh-public-key-login-without-password/. Luettu: 4.11.2026.
Karvinen, T. 2026. Hello Ansible. Luettavissa: https://terokarvinen.com/hello-ansible/. Luettu: 4.11.2026.


