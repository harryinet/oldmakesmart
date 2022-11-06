# TA250 Smart machen
## Idee
Steueren des TA250 von Bosch für z.B. Junkers Gasheizungen über den Homeassiatant.

An dieser über 20 Jahre alten Steuerung soll folgendes über Homeassistant gesteuert werden, jede Funktion wird durch einen kurze Tastendruck auf die jeweilige Taste aktiviert.

* Sparbetrieb toggle ON/OFF
* Warmwasser toggle JA/NEIN + (plus Taste)

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
![[20221003_113725.jpg|350]]

![[20221003_113735.jpg|350]]

## Erweiterung TA250
Per Homeassistant soll Tasten "gedrückt" werden.

* Die Taste Warmwasser (+) gedrückt werden. Die Taste toggelt das Heizen für Warmwasser auf JA oder NEIN, angezeigt im Display.
* Sparbetrieb gelbe Taste. Die Taste toggelt den Sparbetrieb auf EIN oder AUS, angezeigt über die geleb LED "Ein".

### Verdrahtung auf der Platine

![[20220717_094846.jpg|350]]
TA250 front

![[20220717_094826.jpg|350]]
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

Oben seht ihr den Prototyp, später wurde ein kombiniertes 2-Relay-ESP01 Board verwendet wie dieses hier drunter. Die Programmierung des ESP01 ist im Internet merhfach beschrieben, ich nutzte einen USB Adapter für die erste Programmierung über ESPhome im Homeassistant.

![[Screenshot 2022-11-06 123633.png|300]]

##### Einige Zeilen Code für ESPhome
```yaml
esphome:
  name: esp01-2way-a
esp8266:
  board: esp01_1m
# Enable logging
logger:
# Enable Home Assistant API
api:
  encryption:
    key: "5QrKf43uLMvlgXqGMgvKB1edummy"
ota:
  password: "979bc85d45963dummy"
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "esp01-2way-a Fallback Hotspot"
    password: "2kVGhdummy"

captive_portal:

switch:   #HA -) ESP -) Pin
  - platform: gpio
    pin:
      number: 2 #Internal LED switching
      inverted: True  
    name: "esp01-2way-a Internal LED"
    id: internal_led
  - platform: gpio
    pin:
      number: 0 #Output 0
      inverted: True  
    name: "esp01-2way-a Out 0"
    id: output_d0
```

## Homeassistant programmieren
### Heizung
* Energiesparen AN mit Flag
* Energiesparen AUS mit Flag
* Energiesparen AN wenn keine Heizung benötigt wird


### Warmwasser



## Worauf zu achten ist
Da es keine Rückmeldung von dem TA250 an einen Optokoppler oder ähnlich gibt, ist der TA250 ab und zu in Augenschein zu nehmen, ob die Einstellungen mit dem Dashboard des Homeassistant übereinstimmen. Man korrigiert das dann mit einem Druck der Taste auf dem TA250. Dies geschieht bei uns im Vorbeigehen, da das Dashboard auf dem Google Nest Hub 2 dicht bei dem TA250 steht.

Hier ein älteres Video zu einigen Dashboards im Google Nest Hub 2 [Beloved lovelaces ... dashboards ... no cloud needed. - YouTube](https://www.youtube.com/watch?v=Owj9DGhnhpk)

### Funktionen in Homeassistant

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
- [ ] Rückmeldung für LED "Sparbetrieb" irgenwie herstellen
