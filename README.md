# Installatiestappen Raspberry Pi datalogger Oktober 2026

> Installatie Raspberry Pi 3 met 64 bit Debian Trixie met Raspberry Pi Desktop. Er is een DHT22 geïnstalleerd met data naar GPIO22.  Data wordt gelogd in Mysql-database (mariadb)

## ---==== 1. Voorbereiding ====---
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
    
## ---==== 2. Boot van SD-kaartje ====---
* Plaats je SD-kaartje in de Raspberry Pi  (aansluitingscontactjes van SD naar boven)
* Opgelet: vooraleer je stroom aansluit eerst de overige aansluitingen aansluiten (netwerk, scherm indien nodig, usb toetsenbord+muis)
* Indien jouw Raspberry Pi goed opgestart is, kan je vanaf je laptop met PuTTY een ssh-verbinding opzetten naar jouw Raspi

## ---==== 3. Updates ====---
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


## ---==== 4. Aansluiting DHT22 ====---
Let op: we schakelen de Raspbery Pi volledig uit met onderstaand commando wanneer we de DHT sensor aansluiten.
```bash
sudo shutdown now
```
De pinnen van een Raspberry Pi noemen we GPIO (General Purpose Input/Output)
[Meer info](https://nl.wikipedia.org/wiki/General_Purpose_Input/Output)
![GPIO van Raspberry Pi](https://raw.githubusercontent.com/jmo2300/pythonscripts/refs/heads/main/GPIO.png)


We sluiten de DHT op deze manier aan:
<!-- Afbeelding aansluiting DHT22 -->
![Aansluiting van DHT22 op Raspberry Pi](https://raw.githubusercontent.com/jmo2300/pythonscripts/refs/heads/main/AansluitingDHT22.png)



## ---==== 5. Installatie Linux pakketten ====---
Na de update kunnen we starten met de installatie. Wij doen dit klassikaal via PuTTY. 

Voer volgende Linuxcommmando's na elkaar uit:
```bash
sudo apt install apache2 -y
sudo apt install php8.4 php8.4-mysql mariadb-server -y
sudo apt install swig liblgpio-dev
```

## ---==== 6. Database instellen ====---
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

## ---==== 7. Pythonscripts + venv ====---
Voer vanuit je homedirectory volgende commando's **één voor één** uit:
```bash
cd
git clone https://github.com/jmo2300/pythonscripts
cd pythonscripts
python -m venv dhtvenv
source dhtvenv/bin/activate
pip install adafruit-circuitpython-dht
pip install matplotlib mysql-connector-python
```
Je kan vanuit je directory **pythonscripts** al een eerste test **leesdht.py** uitvoeren om de temperatuur en luchtvochtigheid te lezen van de sensor DHT22
```bash
python leesdht.py
```
elke 2 seconden worden de waarden uitgelezen en getoond. Je kan dit onderbreken met ctrl-c


Met het Pythonscript **temperatuurlogger.py** kan je al een waarde wegschrijven naar de database
```bash
python temperatuurlogger.py
```


Met het Pythonscript **toondata.py** kan eenvoudig de data van onze "temperatures"-database tonen
```bash
python toondata.py
```

## ---==== 8. cronjob datalogger ===---
Met volgend commando kan je elk kwartier de datalogger opstarten vanuit crontab
```bash
crontab -e
```
Bij de eerste keer kiezen we "1. /bin/nano"  als editor in crontab.

Voeg volgende lijn **onderaan** toe bij de cronjobs
```bash
0,15,30,45 * * * * ~/pythonscripts/dhtvenv/bin/python ~/pythonscripts/temperatuurlogger.py
```

vanaf nu wordt elk kwartier data (tijdstip, temperatuur en luchtvochtigheid geschreven in de database.

## ---==== 9. Webpagina ====---
De locatie van de webserver staat op /var/www/html.  Met onderstaand commando kan je de voorbeeldpagina index.php beschikbaar maken op jouw website.
```bash
sudo rm /var/www/html/index.html
sudo cp ~/pythonscripts/index.php /var/www/html/
```
Surf met een webbrowser vanaf je laptop (op ons "STEM"-netwerk) naar jouw ipadres  10.16.10.25  (het laatste cijfer is hetzelfde als de nummer van jouw Raspberry Pi)


