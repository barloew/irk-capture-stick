# IRK Capture Stick

ESPHome firmware for the LilyGo T-Dongle-S3 that shows a phone's IRK on screen.

Runs Derek Seaman's [IRK Capture][irk-capture] on a LilyGo T-Dongle-S3 and adds support for its built-in screen and RGB LED. Pair a phone with the dongle and read its Identity Resolving Key (IRK) directly from the display, without searching logs. Use the key with the [Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/) integration or Bermuda to track phones that rotate their MAC address.

> [!WARNING]
> An IRK is a credential. It makes a phone's rotating MAC address permanently traceable until the pairing is removed. Do not leave it on screen or share screenshots of it.

## Features

- **Three screens**, cycled with the button: status, latest IRK and system information
- **Status LED:** green when an IRK is captured, blue while advertising, red when idle
- **Dimmable backlight**, exposed as a light in Home Assistant
- **No extra hardware:** no soldering or additional components

## Requirements

- LilyGo T-Dongle-S3 (ESP32-S3, 16 MB flash)
- ESPHome

## Installation

1. Add your credentials to `secrets.yaml`:

   ```yaml
   wifi_ssid: "YourNetwork"
   wifi_password: "YourPassword"
   api_key: "..."       # generate with: openssl rand -base64 32
   ota_password: "..."
   ```

2. Optionally, change `ble_name` in `irk-dongle.yaml`. This is the name your phone shows. If pairing fails, try a different name first.
3. Plug in the dongle and flash it:

   ```bash
   esphome run irk-dongle.yaml
   ```

   For a first flash of a dongle with other firmware, use the factory image. See [Flashing](docs/flashing.md).

## Usage

1. Check that advertising is on: screen 1 shows **Adv: ON** and the LED is blue.
2. Pair your phone with the dongle in its Bluetooth settings.
3. When the LED turns green, press the button to go to screen 2 and read the IRK.
4. Enter the key in the Private BLE Device integration.

## Documentation

- [Flashing](docs/flashing.md): factory image, esptool, verifying the flash
- [Customizing](docs/customizing.md): pinout, screens, LED control
- [Troubleshooting](docs/troubleshooting.md)

## Credits

Built on [IRK Capture][irk-capture] by Derek Seaman and [ESPHome](https://esphome.io).

[irk-capture]: https://github.com/DerekSeaman/irk-capture
