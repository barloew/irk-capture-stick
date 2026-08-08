# IRK Capture op LilyGo T-Dongle-S3

ESPHome-configuratie die het [IRK Capture](https://github.com/DerekSeaman/irk-capture)-project van Derek Seaman draait op een **LilyGo T-Dongle-S3**, uitgebreid met ondersteuning voor het ingebouwde LCD-schermpje en de RGB-LED.

## Wat doet dit?

De dongle doet zich voor als een Bluetooth-apparaat (standaard met de advertentienaam `IRK Capture`). Koppel je er met je telefoon mee, dan vangt hij de **IRK** (Identity Resolving Key) op.

Die sleutel heb je nodig om in Home Assistant een telefoon met roterend MAC-adres toch betrouwbaar te volgen — bijvoorbeeld via de [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/)-integratie of Bermuda BLE Trilateratie-achtige aanwezigheidsdetectie.

Het verschil met de standaardconfiguratie: hier lees je de IRK **direct van het schermpje** af, zonder logs of de HA-webinterface erbij te hoeven pakken.

## Hardware

- LilyGo T-Dongle-S3 (ESP32-S3, met ST7735 160×80 LCD en APA102 RGB-LED)
- Verder niets — geen soldeerwerk, geen extra componenten

## Functies

| Functie | Toelichting |
|---|---|
| **Drie schermen** | Blader erdoor met de knop op de dongle (GPIO0) |
| **Dimbare backlight** | Via PWM als lamp beschikbaar in Home Assistant, standaard aan bij boot |
| **RGB-LED als statuslampje** | Elke 2 seconden bijgewerkt op basis van de actuele toestand |
| **Twee losse SPI-bussen** | LCD en LED delen geen pinnen en krijgen daarom elk een eigen bus |

### De schermen

1. **Status** — advertising aan/uit, ingestelde BLE-naam, MAC-adres, en de eerste 12 tekens van de IRK
2. **Laatste IRK** — de volledige sleutel over twee regels van 16 tekens, plus het adres waar hij van kwam
3. **Systeem** — SSID, IP-adres en het actieve BLE-profiel

### LED-kleuren

| Kleur | Betekenis |
|---|---|
| 🟢 Groen | IRK opgevangen |
| 🔵 Blauw | Aan het uitzenden (advertising) |
| 🔴 Rood | Inactief |

## Pinout

| Functie | Pin |
|---|---|
| LCD — SPI clock | GPIO5 |
| LCD — SPI MOSI | GPIO3 |
| LCD — CS | GPIO4 |
| LCD — DC | GPIO2 |
| LCD — RESET | GPIO1 |
| LCD — backlight (PWM, inverted) | GPIO38 |
| LED — SPI clock | GPIO39 |
| LED — SPI MOSI | GPIO40 |
| Knop (inverted) | GPIO0 |

## Installatie

### 1. Vul je `secrets.yaml`

```yaml
wifi_ssid: "JouwNetwerk"
wifi_password: "JouwWachtwoord"
api_key: "..."         # base64-encoded encryption key
ota_password: "..."
```

Een API-key genereer je op [esphome.io/components/api](https://esphome.io/components/api/#configuration-variables) of met `openssl rand -base64 32`.

### 2. Pas de BLE-naam aan (optioneel)

```yaml
substitutions:
  ble_name: "IRK Capture"    # <<< je eigen advertentienaam
```

### 3. Flashen

Sluit de dongle aan en installeer via de ESPHome-dashboard, of:

```bash
esphome run irk-dongle.yaml
```

### 4. IRK opvangen

1. Zorg dat advertising aan staat (scherm 1 toont **Adv: AAN**, LED is blauw)
2. Ga op je telefoon naar de Bluetooth-instellingen en koppel met het apparaat
3. De LED wordt groen — druk op de knop tot scherm 2 en lees de IRK af
4. Vul de sleutel in bij de Private BLE Device-integratie in Home Assistant

## Aanpassen

**LED zelf besturen vanuit HA?** Het `interval`-blok onderaan overschrijft handmatige bediening elke 2 seconden. Verwijder dat hele blok, dan blijft `RGB LED` een gewone bestuurbare lamp.

**Andere fonts of layout?** De schermen zijn opgebouwd met lambda's in het `display`-blok; coördinaten en kleuren pas je daar direct aan.

> [!NOTE]
> `spi_led_strip` heeft geen `rgb_order`-optie — de component gaat er zelf van uit dat de LED BGR is, wat precies overeenkomt met de APA102 op deze dongle.

## Credits

- [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture) — het onderliggende project, dit is enkel een device-configuratie erbovenop
- [ESPHome](https://esphome.io)
