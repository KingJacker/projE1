- [ ] Power Management
	- [x] Batterie Modul
	- [ ] Notstrom Umschaltung

- [x] Sensor Verarbeitung
	- [x] Sensor gibt 1 oder 0 als Signal zum RPI (RaspberryPi)

- [ ] Output
	- [ ] Signal über Ethernet

- [ ] (Gehäuse)

# Pflichtenheft
## Anforderungen
- [ ] Der Prototyp muss in einem 30qm Raum erkennen, ob sich dort Personen befinden. Es sollen Personen erkannt werden, die bei Bewusstsein sind und sich bewegen. 
- [ ] Der Prototyp wird an Netzspannung (230 V, 50 Hz) angeschlossen (Stecker und Steckdose nach Norm SN 441011). 
- [ ] Bei Ausfall der Netzspannung soll der Prototyp 1h (mit einem 3000mAh - 4000mAh Li- Ionen-Akku) autonom funktionsfähig sein. 
- [ ] Der Prototyp liefert ein Signal über ein Medium, welches von ausserhalb des Gebäudes auslesbar ist. Das Signal liefert 2 Zustände liefert (Personen anwesend, keine Personen anwesend).


# Gütekriterien
- [ ] Der Prototyp soll Bewegungslose Personen erkennen können. 
- [ ] Der Prototyp soll trotz Rauchentwickelung Menschen erkennen können. 
- [ ] Der Prototyp sollte einen möglichst grossen Abdeckungsbereich haben. 
- [ ] Der Prototyp soll möglichst lange autonom funktionsfähig sein. 
- [ ] Ausgangssignal kann kabellos übertragen werden. 
- [ ] Das System sollte kein false-negative geben. o Mehrere Systemeinheiten sollen in der Lage sein untereinander zu kommunizieren. o Die Systemeinheiten vergleichen untereinander ihre Werte und erkennen Unstimmigkeiten als Fehler.