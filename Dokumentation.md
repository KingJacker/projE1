## 24/11/23
- git Setup für Dokumentation
- Raspi wifi config mit hotspot
- Raspi updates & installation für grove kit

## 27/11/23
- Mind Map vom System
  ![[imgs/20231127_104146-1.jpg|500]]
- Hauptprobleme analysieren
	- Power Management
		- Batterie Modul
		- Notstrom Umschaltung
	- Sensor Verarbeitung
		- Sensor gibt 1 oder 0 als Signal zum RPI (RaspberryPi)
	- Output
		- Signal über Ethernet

Wir notieren unseren Fortschritt in [[Tasks]]
## 28/11/23
### Batterie Modul
- Batterie Modul gelötet
- mit TP4065 Laderegler für die Li-ion Zellen
![[batteriemodul.jpeg|500]]
**- Das Batterie Modul ist somit fertig**
**- Für den Abschluss des Power Management Problems fehlt nun noch ein weg von Netzstrom zu der Batterie umzuschalten.**
### Radar Sensor Kommunikation
- Radar Sensor Pinout: [Dokumentation](https://github.com/limengdu/Seeed-Studio-MR24FDB1-Sensor)
![[Pasted image 20231128111346.png|500]]
- Stecker an Radar Modul angelötet und beim RaspberryPi eingesteckt.
```
RaspberryPi | Radar Modul
5v --------------- 5v
GND -------------- GND
GPIO 14 (TX) ----- RX
GPIO 15 (RX) ----- TX
GPIO 20 ---------- S1
GPIO 21 ---------- S2
```
- RaspberryPi Pinout:
![[imgs/Pasted image 20231128115723.png|700]]
- Als erstes versuchen wir die S1 und S2 als digital signal auf dem RPI zu empfangen, ohne Erfolg. Beide gaben kein Signal. Diese beiden Ausgänge sollten wenn Personen erkannt werden eine digitale 1 ausgeben.
- Als nächstes versuchen wir eine Serielle Verbindung zum RaspberryPi mit Python und dem Modul *PySerial* herzustellen.
- Wir konfigurieren die Serial Verbindung wie im Datenblatt vorgegeben.
```
Interface level: TTL 
Baud rate: 9600bps
Stop bits: 1
Data bits: 8
Parity: None
```
- Wir erhalten ein Signal welches nicht der vom Datenblatt gegebenen Struktur entspricht. Der Startbyte sollte *55* sein. 
```bash
>>> ser.readline()
b'\x04\x03\x06\x00\x00\x80?\xfcEU\x0b\x00\x04\x03\x06\x00\x00\x80?\xfcEU\n'
>>> ser.readline()
b'\x00\x04\x03\x05\x01\x00\xff\x1d\x14U\n'
>>> ser.readline()
b'\x00\x04\x03\x05\x01\x00\xff\x1d\x14U\x0b\x00\x04\x03\x06\x00\x00\x80?\xfcEU\n'
```
![[imgs/Screen Shot 2023-11-28 at 13.33.58.png]]
- Nach einigen Versuchen die erhaltenen Antworten zu entziffern gaben wir mit dieser Methode auf. Weil alle vom Händler verlinkten Setup-Guides auf einen Arduino library verwiesen.

#### Arduino
- Arduino Pinout
![[imgs/A000066-pinout.png|500]]
- Nach dem installieren der [Library](https://github.com/limengdu/Seeed-Studio-MR24FDB1-Sensor) und Ausführen des Beispiel Sketch *CRC_Checksum_Generation.ino* erhalten wir die Ausgabe:
```
The datas send to the radar: 0x55 0x08 0x00 0x05 0x01 0x04 0x03 0x0c 0x80 
The CRC16 values is: 0c80
```


## 29/11/23
### Sensor Abfrage
- Mit neuem [Code](https://github.com/Seeed-Studio/Seeed_Arduino_24GHz_Radar_Sensor) hat es nun geklappt die Ausgabe vom Sensor mit dem Arduino über Serial auszulesen. Der nächste Schritt ist den Code für den RaspberryPi anzupassen.
- Das Radar Modul ermöglicht eine einfache Status abfrage dadurch dass zwei output pins vom Radar Modul den Status aktuellen Status ausgeben über zwei Pins. S1 gibt an ob der Sensor Personen erkennt und S2 ob diese sich bewegen. 

- Dieser Code gibt den Status des Radarsensors auf dem RaspberryPi aus.
```python
# Code für Statusabfrage vom Radarsensor mit dem Raspi
from gpiozero import DigitalInputDevice
from time import sleep

occupancy = DigitalInputDevice(24) # Radarmodul S1
movement = DigitalInputDevice(23)  # Radarmodul S2

while True:
        out = f'Occupied: {occupancy.value}\nMovement: {movement.value}'
        print(out)
        sleep(1)
```

- **Die Verarbeitung von dem Sensorinput ist somit erreicht!**

### Notstrom Umschaltung
- Nach kurzer Recherche fanden wir ein Produkt das unseren Anforderungen perfekt erfüllt:[**WaveShare** USV HAT für Raspberry Pi Unterbrechungsfreie Stromversorgung](https://www.digitec.ch/de/s1/product/waveshare-usv-hat-fuer-raspberry-pi-unterbrechungsfreie-stromversorgung-entwicklungsboard-kit-25007122?supplier=8244233)
- Mit einem Preis von 35.90 CHF passt dieser noch gut in unser Budget. --> Bestellt.

### Plan für Morgen
- Zweiten Raspi mitnehmen um Output zu machen

## 30/11/2023
### Signalübertragung Ethernet
- Signalübertragung Ethernet mit zwei Raspi testen
	- 1. Raspi gibt ein Output Signal und sendet Signal an IP-Adresse
	- Auf einem zweiten Raspi ist ein Netcat server eingerichtet um das Zustands Signal zu empfangen.
- Problem: Solange Raspis im Wifi sind funktioniert die Verbindung via Ethernet nicht, aber wenn sie nicht mit wifi verbunden sind können wir nicht drauf zugreifen
- Solution: Raspi mit Tastatur und Bildschirm ansteuern. Einmal dient der Projektor als Bildschirm und einmal ein Bildschirm, beide werden mit der gleichen Tastatur bedient.

Netcat ist ein vielseitiges Kommandozeilen-Tool, das in Unix-ähnlichen Betriebssystemen (einschließlich Linux) sowie in Windows verfügbar ist. Es wurde entwickelt, um einfach Daten über Netzwerke zu übertragen.

- Beide Raspis wurden mit statischer IP konfiguriert und können nun über das Ethernetkabel miteinander kommunizieren.

### Plan für Morgen
- Python Programm von der Sensorauslesung ergänzen mit Code zum senden der Information.
- Python Programm für den Empfang auf dem zweiten Pi, sowie Ansterung einer LED zur Visualisierung der übertragenen Daten.