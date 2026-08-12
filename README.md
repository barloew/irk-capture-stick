# IRK Capture on LilyGo T-Dongle-S3

An ESPHome configuration that runs Derek Seaman's [IRK Capture](https://github.com/DerekSeaman/irk-capture) project on a **LilyGo T-Dongle-S3**, extended with support for the built-in LCD screen and RGB LED.

## What does this do?

The dongle presents itself as a Bluetooth device (advertising name `IRK Capture` by default). Pair your phone with it and it captures the **IRK** (Identity Resolving Key).

You need that key to reliably track a phone with a rotating MAC address in Home Assistant — for example via the [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/) integration, or Bermuda BLE trilateration-style presence detection.

The difference from the standard configuration: here you read the IRK **straight off the screen**, without having to dig through logs or the HA web interface.

## Hardware

- LilyGo T-Dongle-S3 (ESP32-S3, with ST7735 160×80 LCD and APA102 RGB LED)
- Nothing else — no soldering, no extra components

## Features

| Feature | Notes |
|---|---|
| **Three screens** | Cycle through them with the button on the dongle (GPIO0) |
| **Dimmable backlight** | Available as a light in Home Assistant via PWM, on by default at boot |
| **RGB LED as status indicator** | Updated every 2 seconds based on the current state |
| **Two separate SPI buses** | The LCD and LED share no pins, so each gets its own bus |

### The screens

1. **Status** — advertising on/off, the configured BLE name, MAC address, and the first 12 characters of the IRK
2. **Latest IRK** — the full key across two lines of 16 characters, plus the address it came from
3. **System** — SSID, IP address, and the active BLE profile

### LED colours

| Colour | Meaning |
|---|---|
| 🟢 Green | IRK captured |
| 🔵 Blue | Advertising |
| 🔴 Red | Idle |

## Pinout

| Function | Pin |
|---|---|
| LCD — SPI clock | GPIO5 |
| LCD — SPI MOSI | GPIO3 |
| LCD — CS | GPIO4 |
| LCD — DC | GPIO2 |
| LCD — RESET | GPIO1 |
| LCD — backlight (PWM, inverted) | GPIO38 |
| LED — SPI clock | GPIO39 |
| LED — SPI MOSI | GPIO40 |
| Button (inverted) | GPIO0 |

## Installation

### 1. Fill in your `secrets.yaml`

```yaml
wifi_ssid: "YourNetwork"
wifi_password: "YourPassword"
api_key: "..."         # base64-encoded encryption key
ota_password: "..."
```

You can generate an API key at [esphome.io/components/api](https://esphome.io/components/api/#configuration-variables) or with `openssl rand -base64 32`.

### 2. Change the BLE name (optional)

```yaml
substitutions:
  ble_name: "IRK Capture"    # <<< your own advertising name
```

### 3. Flashing

Plug in the dongle and install via the ESPHome dashboard, or:

```bash
esphome run irk-dongle.yaml
```

### 4. Capturing an IRK

1. Make sure advertising is on (screen 1 shows **Adv: ON**, the LED is blue)
2. Go to the Bluetooth settings on your phone and pair with the device
3. The LED turns green — press the button until you reach screen 2 and read off the IRK
4. Enter the key in the Private BLE Device integration in Home Assistant

## Customising

**Want to control the LED yourself from HA?** The `interval` block at the bottom overrides manual control every 2 seconds. Remove that whole block and `RGB LED` stays an ordinary controllable light.

**Different fonts or layout?** The screens are built with lambdas in the `display` block; adjust coordinates and colours directly there.

> [!NOTE]
> `spi_led_strip` has no `rgb_order` option — the component simply assumes the LED is BGR, which is exactly what the APA102 on this dongle uses.

## Credits

- [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture) — the underlying project; this is only a device configuration on top of it
- [ESPHome](https://esphome.io)
