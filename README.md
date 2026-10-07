# Installatiestappen Raspberry Pi datalogger Oktober 2026

> Installatie Raspberry Pi 3 met 64 bit Debian Trixie met Raspberry Pi Desktop. Er is een DHT22 geïnstalleerd met data naar GPIO22.  Data wordt gelogd in Mysql-database (mariadb)

## 1. Voorbereiding
* Gebruik Raspberry Pi Imager met volgende instellingen
  * Kies als **device** Raspberry Pi3  (dezelfde als in onze klas)
  * Kies bij **OS** Raspberry Pi OS (64 bit) A port of Debian Trixie met Raspberry Pi Desktop
  * Selecteer je SD-kaartje bij **Storage**
  * Als hostnaam in ons netwerk vb Raspi25  (indien je ipadres eindigt met 25)
  * Bij **Localisation** kies je de default waarde voor BE
  * We maken SSID bij wifi leeg
  * Enable SSH
  * Raspberry Pi Connect is niet nodig op onze Raspberry Pi 3
  * Bevestig dat je SD-kaartje overschreven wordt
    
## 2. Boot van SD-kaartje
* Plaats je SD-kaartje in de Raspberry Pi  (aansluitingscontactjes van SD naar boven)
* Opgelet: vooraleer je stroom aansluit eerst de overige aansluitingen aansluiten (netwerk, scherm indien nodig, usb toetsenbord+muis)
* Indien jouw Raspberry Pi goed opgestart is, kan je vanaf je laptop met PuTTY een ssh-verbinding opzetten naar jouw Raspi

## 3. Updates
Updates kunnen best wel wat tijd nemen en daarom doen we vanaf de desktop op Raspberry Pi.  We doen dit klassikaal met volgend commando

```bash
sudo raspi-config
```
* Kies "3 Interface Options"
* VNC
* Enabled op Yes
* Ga terug naar hoofdmenu  (kies Back)
* Kies Finish

Via een terminal op je Raspi Desktop kan je volgend commando uitvoeren:

```bash
sudo apt update && sudo apt upgrade -y
```

## 4. Installatie Linux pakketten
Na de update kunnen we starten met de installatie. Wij doen dit klassikaal via PuTTY. 

Voer volgende Linuxcommmando's na elkaar uit:
```bash
sudo apt install apache2 -y
sudo apt install php8.4 php8.4-mysql mariadb-server -y
sudo apt install swig liblgpio-dev
```

## 5. Database instellen
```bash
sudo mariadb-secure-installation
```
* paswoord : (afgesproken in onze klas, LET OP: je typt dit blind in)
* unix_socket authentication [Y/n] n
* Change the root password? [Y/n] n
* Remove anonymous users? [Y/n] y
* Disallow root login remotely? [Y/n] y
* Remove test database and access to it? [Y/n] y
* Reload privilege tables now? [Y/n] n

###  Database en tabellen aanmaken
Eerst als sudo:
```bash
sudo mysql -u root -p
```
Geef ons afgesproken paswoord (blind: je ziet geen characters verschijnen) 
Kopieer onderstaande code om de database aan te maken
```bash
CREATE DATABASE temperatures;
USE temperatures;
CREATE USER 'logger'@'localhost' IDENTIFIED BY 'paswoord';
GRANT ALL PRIVILEGES ON temperatures.* TO 'logger'@'localhost';
FLUSH PRIVILEGES;
QUIT
```

Als gebruiker logger:
```bash
mysql -u logger -p
```
Geef het paswoord "blind" in en voor vervolgens onderstaande code uit.
```bash
USE temperatures;
CREATE TABLE temperaturedata (dateandtime DATETIME, sensor VARCHAR(32), temperature DOUBLE, humidity DOUBLE);
QUIT
```

## 6. Pythonscripts
Voer vanuit je homedirectory volgende commando's één voor één uit:
```bash
cd
git clone https://github.com/jmo2300/pythonscripts
cd pythonscripts

```





