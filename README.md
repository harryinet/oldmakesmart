---
title: TA250 Smart machen
created_date: 2022-10-03
created_time: 10:27

project: Fichtenweg
subproject: Heizung Smart
summary:

type: 
Aliases: 
Tags: homeassistant esp32

---
# TA250 Smart machen
Immer die Einstellungen am TA250 prüfen 

## Idee
Steueren des TA250 für z.B. Junkers Gasheizungen über den Homeassiatant.

An dieser über 20 Jahre alten Steuerung soll folgendes über Homeassistant gesteuert werden, jede Funktion wird durch einen kurze Tastendruck auf die jeweilige Taste aktiviert.

* Sparbetrieb toggle ON/OFF
* Warmwasser toggle JA/NEIN + (plus Taste

Der Gedanke ist, die jeweiligen Tasten per Optokoppler für einen kurzen Moment, ca. 700 msec, zu überbrücken um den Tstendruck zu simulieren.

Es ergeben sich dann folgende Möglichkeiten, die im Homeassistant zu programmieren sind.

#### Sparbetrieb
* Sparbetrieb einschalten, Gas aus, abschalten aller Pumpen. 
	* Die Pumpen brauchen, wenn kein Sprbetrieb, zusammen 140W permament. 
	* Die Elektronik der Heizung benötigt 11W.
* Sparbetrieb ausschalten, 
	* Wiederherstellen Heizbetrieb gemäß der Einstellungen am TA250 und zusätzlicher Einstellungen im Homeassistant, siehe [[Generic Thermostate]] im Dashboard.

Der Sparbetrieb wird automatisch um Mitternacht beendet, das ist eine feste Programmierung des Gerätes.

Die Erfahrung zeigt, dass es in der Übergangsjahreszeit nicht schädlich ist nachts den Sparbetrieb aus AUS zu schalten. Das ist das kleine Einstellrad hinter der Abdeckung.

#### Warmwasser
* Warmwasser heizen
	* gemäß Einstellung am TA250 für 150 Liter auf 45° C.
* Warmwasser heizen für z.B. nur 10 Minuten
	* um Duschen für 1 Person zu ermöglichen, ohne die ganzen 150 Liter zu erwärmen.
* Warmwasser heizen abschalten

Gemäß Anleitung kann man das Erhitzen von Wasser mit der +/- Taste starten oder beenden. Die Regel ist, dass man nur aus + (plus) drückt und dann der obere Teil des Pufferspeichers mit 150 Liter Wasser auf einmalig eine vorgegeben Temperatur erhitz wird, hier ca. 45°. Das reicht für 2x Duschen.
Nochmaliges Drücken der + (plus) Taste schaltet das Heizen wieder ab.

#### Bilder Gerät
![[20221003_113725.jpg|500]]

![[20221003_113735 1.jpg|500]]

## Erweiterung TA250
Per Homeassistant soll Tasten "gedrückt" werden.

* Die Taste Warmwasser (+) gedrückt werden. Die Taste toggelt das Heizen für Warmwasser auf JA oder NEIN, angezeigt im Display.
* Sparbetrieb gelbe Taste. Die Taste toggelt den Sparbetrieb auf EIN oder AUS, angezeigt über die geleb LED "Ein".

### Verdrahtung auf der Platine
![[20220717_094846.jpg|500]]
TA250 front

![[20220717_094826.jpg|500]]
TA250 back

* Schwarz = Masse (ground)
* Orange = Sparbetrieb geleb Taste. Die Taste toggelt den Sparbetrieb auf EIN oder AUS.
* Blau = Warmwasser + (plus)

![[20220717_105049.jpg|300]]  

![[20220717_105055.jpg|300]]

![[20220717_104908.jpg|300]]

### Material aus der Schublade
* NodeMCU [ESP8266 NodeMcu Pinout - ESP8266 Shop (esp8266-shop.com)](https://esp8266-shop.com/esp8266-guide/esp8266-nodemcu-pinout/)
* 2 Kanal Relay Board
##### Aufbau
![[20221008_120255.jpg|400]]
NodeMCU
![[20221008_121334.jpg|400]]
Relaisboard

##### Einige Zeilen Code für ESPhome
```yaml
esphome:
  name: mcu-a
esp8266:
  board: nodemcuv2
# Enable logging
logger:
# Enable Home Assistant API
api:
  encryption:
    key: "******"
ota:
  password: "******"
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Mcu-A Fallback Hotspot"
    password: "******"

captive_portal:

switch:   #HA -) ESP -) Pin

  - platform: gpio
    pin:
      number: D4 #Internal LED switching
      inverted: True  
    name: "MCU-A Internal LED"
    id: internal_led

  - platform: gpio
    pin:
      number: D1 #Output 1
      inverted: True  
    name: "MCU-A OUT-D1"
    id: output_d1

  - platform: gpio
    pin:
      number: D2 #Output 2
      inverted: True  
    name: "MCU-A OUT-D2"
    id: output_d2 
```


### Funktionen in Homeassistant
* [x] Energiesparen AN mit Flag [prio:: A]
* [x] Energiesparen AUS mit Flag [prio:: A]
* [x] Flag setzen per Button "TA250 Sparbetrieb an" [prio:: A]
* [x] Wasser AN mit Flag [prio:: A]
* [x] Wasser AUS mit Flag [prio:: A]
* [x] Flag setzen per Button "TA250 Wasser an" [prio:: A]
* [x] Wasser 13 Minuten AN [prio:: A]
#### Wasser
* [ ] Wasser getrennte Flags für Kurz und Lang Wasser. Das geht, allerdings bremst die Wartzeit die weitere Bearbeitung aus für den vollständigen Speicher. 
	* [ ] Wasser Vollständig deaktivieren, solange Kurz läuft.
	* [ ] Wasser erhöhen wenn nur Kurz geschaltet ist
	* [ ] Mit Timer dazu was bauen
	* [ ] Wie geht Timer noch ...
* [ ] Wasser Temperatur prüfen und dann schon ausschalten, als Signal Button aus
* [x] Wenn Wasser warm ist über 43, dann Zirkulation einschalten
* [ ] Wasser Big Button Funktion prüfen [due:: 2022-11-06]
#### Heizung
* [x] Sparbetrieb Bedingung permanent prüfen
	* [x] Wenn die Anzahl der Thermostate auf 0 ist dann soll der Sparbetrieb geschaltet werden.
* [x] Rücksetzen in der Nacht, im Mitternacht wird der Sparbetrieb aufgehoben in dem TA250. Das muss im Homeassistant nachgeführt werden.
	* [x] Überprüfung also nur zwischen 00:30 Uhr und 23:45, damit es keine Überschneidungen gibt
* [ ] Doku überprüfen der Zeit im TA250
* [x] TA250 Relais in Box [due:: 2022-10-16] ✅ 2022-10-16

### Ausblick
Gerne würde ich noch die Status LED des TA250 für "Energiesparen" als Input in den NodeMCU aufnehmen, über den Optokoppler. Jedoch fehlt mir hierzu die Information, wo der 5V Wert abzugreifen ist.


### ToDo´s
- [ ] Schaltplan zeichen für TA250

