# TA250 Smart machen - oldmakesmart
## Idee
Steueren des TA250 von Bosch für z.B. Junkers Gasheizungen über den Homeassiatant.

An dieser über 20 Jahre alten Steuerung soll folgendes über Homeassistant gesteuert werden, jede Funktion wird durch einen kurze Tastendruck auf die jeweilige Taste aktiviert.

* Sparbetrieb toggle ON/OFF
* Warmwasser toggle JA/NEIN + (plus Taste)

Der Gedanke ist, die jeweiligen Tasten per Optokoppler für einen kurzen Moment, ca. 700 msec, zu überbrücken um den Tstendruck zu simulieren.

Es ergeben sich dann folgende Möglichkeiten, die im Homeassistant zu programmieren sind.

## Ein Ergebnis vorne Weg
Durch konsquentes Abschalten und zielgereichtetes Einschalten der Heizung und des Warmen Wassers, welches ebenfalls über Solarthemie gewonnen wird, konnten in 12 Monaten ca. 1.000m³ an Gas eingespart werden.

![[Screenshot_20221106_162811.jpg|450]]

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

#### Ein einfache Gehäuse 3D gedruckt
Ein passendes Gehäuse kann man z.B. hier finden und ist schnell gedruckt ...
[Waterproof electronic box / enclosure by pbtec - Thingiverse](https://www.thingiverse.com/thing:4921568)


#### Einige Zeilen Code für ESPhome
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
Die Programmierung des Energiesparens ist relativ einfach.
* Energiesparen AN/AUS mit Flag, ob schon gesetzt.
* Energiesparen AN wenn keine Heizung benötigt wird.

##### Energiesparen AN/AUS mit Flag, ob schon gesetzt.

Zunächst der Tastendruck für 0,7 sec.
```YAML
alias: TA250 Sparbetrieb an D1
description: Toggle Taste Sparbetrieb für 0,7 sec über Pin D1
trigger: []
condition: []
action:
  - type: turn_on
    device_id: 090e8bda57306f97f7fe1fee3f272733
    entity_id: switch.esp01_2way_a_internal_led
    domain: switch
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 700
  - type: turn_off
    device_id: 090e8bda57306f97f7fe1fee3f272733
    entity_id: switch.esp01_2way_a_internal_led
    domain: switch
mode: single
```

Setzen des Flag, je nach Zustand
```YAML
alias: TA250 Sparbetrieb AN/AUS
description: "Wechsel zwischen AN und AUS"
trigger: []
condition: []
action:
  - choose:
      - conditions:
          - condition: state
            entity_id: input_boolean.ta250_sparbetrieb_an
            state: "off"
        sequence:
          - service: input_boolean.turn_on
            data: {}
            target:
              entity_id: input_boolean.ta250_sparbetrieb_an
          - service: automation.trigger
            data: {}
            target:
              entity_id: automation.ta250_sparbetrieb_an_d1
    default:
      - service: input_boolean.turn_off
        data: {}
        target:
          entity_id: input_boolean.ta250_sparbetrieb_an
      - service: automation.trigger
        data: {}
        target:
          entity_id: automation.ta250_sparbetrieb_an_d1
mode: queued
max: 10
```

##### Button im Dashboard mit Code
Wenn man diesen antippt, dann wechselt je nach Status die Farbe und der Text, und die Taste wird über das Relay für 0,7 sec. gedrückt.

![[Screenshot 2022-11-06 153520.png|150]]

```YAML
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: call-service
      service: automation.trigger
      target:
        entity_id: automation.ta250_sparbetrieb_an_aus_2
      data: {}
    entity: input_boolean.ta250_sparbetrieb_an
    name: 'Sparbetrieb '
    show_state: true
```

### Sparbetrieb AN wenn kein Bedarf an Heizung
Um die Sache an dieser Stelle noch ganz rund zu machen zähle ich die Generic Thermostate, die überheupt eine Heizung benötigen. Das hilft besonders in der Übergangsjahreszeit. Es gibt Räume mit großen Fenstern, die bei Sonneneinstrahlung schon von selbst über 22° C erreichen, oder es kann sein, dass die Aussentemperatur an einem Tag ausreicht und am anderen Tag wieder niedrig ist. Vielleicht komme ich noch später dazu die Steuerung der Fussbodenheizung zu beschreiben, die eben aus 10x Generic Thermostat und 5x ON/OFF Schaltern besteht.

Basis sind diverse XIAOMI Raumthermometer und 8-Relay Tasmota Platinen, das mit 230V Aktoren, hier ein Beispiel.
![[20211127_123720.jpg|500]]

Auf jeden Fall nutze ich den Sparbetrieb um auch den letzten Gas- und Stromverbauch der Heizung so gering wie möglich zu halten.

Genereic Thermostat und ON/OFF
![[Screenshot 2022-11-06 154814.png|450]]

Zählen der Thermostate
![[Screenshot 2022-11-06 124401.png|300]]

Sensor zum Zählen es können Genereic Thermostate über friendly_name ausgeschlossen werden
```YAML
- platform: template
  sensors:
    number_heaters_on:
      friendly_name: Anzahl Heizungen AN
      value_template: >-
        {{ states.climate
            | rejectattr('attributes.hvac_action','eq','idle')
            | rejectattr('attributes.hvac_action','eq','off')
            | rejectattr('attributes.friendly_name','eq','Elise Wand')
            | list | count }}
      icon_template: mdi:home-thermometer-outline
```

Zyklisch prüfen, ob es möglich ist den Sparbetrieb einzuschalten
```YAML
alias: TA250 Sparbetrieb zyklisch prüfen
description: "Prüfe alle 6 min. ob Räume geheizt werden müssen."
trigger:
  - platform: time_pattern
    minutes: /6
condition:
  - condition: time
    after: "00:15:00"
    before: "23:45:00"
action:
  - if:
      - condition: and
        conditions:
          - condition: state
            entity_id: input_boolean.ta250_sparbetrieb_an
            state: "off"
          - condition: numeric_state
            entity_id: sensor.number_heaters_on
            below: 1
    then:
      - service: input_boolean.turn_on
        data: {}
        target:
          entity_id: input_boolean.ta250_sparbetrieb_an
      - service: automation.trigger
        data: {}
        target:
          entity_id: automation.ta250_sparbetrieb_an_d1
    else:
      - if:
          - condition: and
            conditions:
              - condition: state
                entity_id: input_boolean.ta250_sparbetrieb_an
                state: "on"
              - condition: numeric_state
                entity_id: sensor.number_heaters_on
                above: 0
        then:
          - service: input_boolean.turn_off
            data: {}
            target:
              entity_id: input_boolean.ta250_sparbetrieb_an
          - service: automation.trigger
            data: {}
            target:
              entity_id: automation.ta250_sparbetrieb_an_d1
mode: single
```

Die Zentralheizung hebt von sich aus den Sparbetrieb nachts auf, somit ist das Flag entsprechend zu setzen.
```YAML
alias: TA250 Sparbetrieb aus 00:05 Uhr
description: "Nachts das Flag gemäß dem Status der Heizung setzen."
trigger:
  - platform: time
    at: "00:05:00"
condition: []
action:
  - service: input_boolean.turn_off
    data: {}
    target:
      entity_id: input_boolean.ta250_sparbetrieb_an
mode: single
```

### Warmwasser
Das Thema Warmwasser birgt ebenfalls Einsparpotential, die Systematik ist nahezu identisch.
Hier noch nicht beschrieben ist eine Logic, die anhand der vorhandenen Temperatur im Wasserspeicher eine Zeitdauer schätzt, die überhaupt nötig ist nachzuheizen.

Die Logik der alten Heizung ist, wenn Warmwasser AN, dann Heize alle 150 Liter bis 50° C.

Wenn eine Einzelperson Duschen möchte ist das ca. nur zur Hälfte nötig. So ergibt sich folgende Funktion für das Dashboard.

![[Screenshot 2022-11-06 164341.png|450]]

##### Warmwasser AN/AUS mit Flag, ob schon gesetzt.
```YAML
alias: TA250 Wasser an D2
description: Toggle Taste Wasser + über Pin D2
trigger: []
condition: []
action:
  - type: turn_on
    device_id: 090e8bda57306f97f7fe1fee3f272733
    entity_id: switch.esp01_2way_a_out_0
    domain: switch
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 700
  - type: turn_off
    device_id: 090e8bda57306f97f7fe1fee3f272733
    entity_id: switch.esp01_2way_a_out_0
    domain: switch
mode: single
```

##### Warmwasser kurz einschalten und Zirkulation aktivieren
```YAML
alias: TA250 Warmwasser 1 Person
description: TA250 Warmwasser 1 Pers
trigger: []
condition: []
action:
  - choose:
      - conditions:
          - condition: state
            entity_id: input_boolean.ta250_wasser_kurz
            state: "off"
        sequence:
          - service: input_boolean.turn_on
            data: {}
            target:
              entity_id: input_boolean.ta250_wasser_kurz
          - service: automation.trigger
            data: {}
            target:
              entity_id: automation.ta250_wasser_an
          - delay:
              hours: 0
              minutes: 12
              seconds: 0
              milliseconds: 0
          - service: automation.trigger
            data: {}
            target:
              entity_id: automation.ta250_zirkulation_einschalten
          - delay:
              hours: 0
              minutes: 8
              seconds: 0
              milliseconds: 0
          - service: automation.trigger
            data: {}
            target:
              entity_id: automation.ta250_wasser_an
          - service: input_boolean.turn_off
            data: {}
            target:
              entity_id: input_boolean.ta250_wasser_kurz
    default: []
mode: single

```

##### Eine Logik mit Timer muss programmiert werden
* [ ] Wasser getrennte Flags für Kurz und Lang Wasser. Das geht, allerdings bremst die Wartzeit die weitere Bearbeitung aus für den vollständigen Speicher. 
	* [ ] Wasser Vollständig deaktivieren, solange Kurz läuft.
	* [ ] Wasser erhöhen wenn nur Kurz geschaltet iste
	* [ ] Mit Timer dazu was programmieren
* [ ] Wasser Temperatur prüfen und dann schon ausschalten, als Signal Button aus
* [ ] Wasser Big Button Funktion prüfen [due:: 2022-11-06]

##### Zirkulationspumpe steuern
Des Weiteren lässt sich ein kleiner, lange vermisster Luxus einbauen.
Mit einem Shelly 1L kann man nun anhand der Wasserspeicher Temparatur erlauben, dass diese aben zirkuliert.

Zur Geschichte, vor 20 Jahren sagte der Heizungsmonteur, die Warmwasser- Zirkulationspumpe dürfe nicht immer laufen, da diese sonst die Sonnenenergie schmälert. Somit hatten wir eine Art Treppenlichtschalter dafür, der vor dem Duschen gedrückt wurde. 

Unser Warmwasser wird möglichst evorranging über einen Solarthermie Kollektoren erwärmt.

So erlaub ich es, sofern das Wasser über 33° C hat, in gewisssen Abständen die Warmwasser Zirkulationspumpe laufen zu lassen.

```YAML
alias: TA250 Zirkulation einschalten
description: "Stündlich prüfen ob das Wasser warm genug ist für Zirkulation."
trigger:
  - platform: time_pattern
    minutes: /58
condition:
  - condition: and
    conditions:
      - type: is_temperature
        condition: device
        device_id: 8703e4909362c00b173698d63913deaf
        entity_id: sensor.keller_wasser_temperature
        domain: sensor
        above: 33
      - condition: time
        after: "06:30:00"
        before: "21:00:00"
action:
  - service: input_boolean.turn_on
    data: {}
    target:
      entity_id: input_boolean.ta250_zirkulation
  - type: turn_on
    device_id: fb76b70507019505eef79666f9bac099
    entity_id: switch.zirkulationwasser
    domain: switch
  - delay:
      hours: 0
      minutes: 20
      seconds: 0
      milliseconds: 0
  - service: input_boolean.turn_off
    data: {}
    target:
      entity_id: input_boolean.ta250_zirkulation
  - type: turn_off
    device_id: fb76b70507019505eef79666f9bac099
    entity_id: switch.zirkulationwasser
    domain: switch
mode: single
```

## Worauf zu achten ist
Da es keine Rückmeldung von dem TA250 an einen Optokoppler oder ähnlich gibt, ist der TA250 ab und zu in Augenschein zu nehmen, ob die Einstellungen mit dem Dashboard des Homeassistant übereinstimmen. Man korrigiert das dann mit einem Druck der Taste auf dem TA250. Dies geschieht bei uns im Vorbeigehen, da das Dashboard auf dem Google Nest Hub 2 dicht bei dem TA250 steht.

Hier ein älteres Video zu einigen Dashboards im Google Nest Hub 2 [Beloved lovelaces ... dashboards ... no cloud needed. - YouTube](https://www.youtube.com/watch?v=Owj9DGhnhpk)

### Funktionen in Homeassistant

### Ausblick
Gerne würde ich noch die Status LED des TA250 für "Energiesparen" als Input in den NodeMCU aufnehmen, über den Optokoppler. Jedoch fehlt mir hierzu die Information, wo der 5V Wert abzugreifen ist.
- [ ] Rückmeldung für LED "Sparbetrieb" irgenwie herstellen
