## 24/11/23
- git Setup für Dokumentation
- Raspi wifi config mit hotspot
- Raspi updates & installation für grove kit

## 27/11/23
- Mind Map vom System
  ![[imgs/20231127_104146-1.jpg|500]]
- Hauptprobleme analysieren
	- Power Management
		- Li-Ion
		- BMS
	- Sensor Processing
		- Sensor gibt 1 oder 0 als Signal zum RPI (RaspberryPi)
	- Signal Out
		- Signal über Ethernet

## 28/11/23
### Batterie Modul
- Batterie Modul gelötet
	- mit TP4065 Laderegler für die Li-ion Zellen
![[batteriemodul.jpeg|500]]

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
