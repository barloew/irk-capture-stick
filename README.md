# IRK Capture on LilyGo T-Dongle-S3

An ESPHome configuration that runs Derek Seaman's
[IRK Capture](https://github.com/DerekSeaman/irk-capture) project on a
**LilyGo T-Dongle-S3**, extended with support for the built-in LCD screen and
RGB LED.

## What does this do?

The dongle presents itself as a Bluetooth device. Pair your phone with it and
it captures the **IRK** (Identity Resolving Key).

You need that key to reliably track a phone with a rotating MAC address in Home
Assistant — for example via the
[Private BLE Device](https://www.home-assistant.io/integrations/private_ble_device/)
integration, or Bermuda BLE trilateration-style presence detection.

The difference from the standard configuration: here you read the IRK **straight
off the screen**, without having to dig through logs or the HA web interface.

> [!WARNING]
> An IRK is a credential, not an identifier. It is the key that makes a phone's
> rotating MAC address permanently resolvable — exactly what the rotation is
> designed to prevent. It only changes if the pairing is removed and recreated
> from scratch. Don't leave it displayed on a desk, and don't paste screenshots
> of screen 2 into issues or forum posts.

## Hardware

- LilyGo T-Dongle-S3 (ESP32-S3, 16 MB flash, no PSRAM, ST7735 160×80 LCD, APA102 RGB LED)
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

You can generate an API key at
[esphome.io/components/api](https://esphome.io/components/api/#configuration-variables)
or with `openssl rand -base64 32`.

### 2. Change the BLE name (optional)

```yaml
substitutions:
  ble_name: "irk-dongle"    # <<< your own advertising name
```

This is the name your phone will see in its Bluetooth list. Some phones pair
more readily with a name that looks like a familiar accessory, so if pairing
gives you trouble, this is the first thing worth changing.

### 3. Board configuration

The relevant part of the config, and why it looks the way it does:

```yaml
esp32:
  board: esp32-s3-devkitc-1
  variant: esp32s3      # ESPHome doesn't recognise the board by name
  flash_size: 16MB      # the board definition would default to 4 MB
```

Both lines are required. Without `variant`, validation fails with *"This board
is unknown"*; without `flash_size`, three quarters of the chip goes unused and
there is too little room for OTA updates.

### 4. Flash it

See [Building and flashing a factory image](#building-and-flashing-a-factory-image)
below. The short version, if the dongle is plugged into the machine running
ESPHome:

```bash
esphome run irk-dongle.yaml
```

### 5. Capture an IRK

1. Make sure advertising is on (screen 1 shows **Adv: ON**, the LED is blue)
2. Go to the Bluetooth settings on your phone and pair with the device
3. The LED turns green — press the button until you reach screen 2 and read off the IRK
4. Enter the key in the Private BLE Device integration in Home Assistant

## Building and flashing a factory image

Two kinds of binary come out of an ESPHome build, and picking the wrong one is
the most common reason a first flash appears to succeed but the device never
boots:

| File | Contains | Flash at | Use for |
|---|---|---|---|
| `firmware.factory.bin` | bootloader + partition table + app | `0x0` | first flash of a blank or repurposed dongle |
| `firmware.ota.bin` | app only | — | over-the-air updates once the device is on WiFi |

A T-Dongle-S3 that shipped with other firmware, or that previously ran a build
for a different chip, needs the **factory** image. After that, ESPHome updates
it over WiFi and you never touch USB again.

### Build it

**From the ESPHome dashboard:** open the device, choose **Install → Manual
download**, and pick **Factory format** when asked. The browser downloads
`irk-dongle.factory.bin`.

**From the command line:**

```bash
esphome compile irk-dongle.yaml
```

The result lands at:

```
.esphome/build/irk-dongle/.pioenvs/irk-dongle/firmware.factory.bin
```

If ESPHome runs in Docker, that path is inside the container — prefix it with
whatever host directory you mounted as `/config`:

```bash
docker exec -it esphome esphome compile /config/irk-dongle.yaml
ls -l ./esphome/.esphome/build/irk-dongle/.pioenvs/irk-dongle/firmware.factory.bin
```

> [!IMPORTANT]
> After changing `board`, `variant`, or `flash_size`, delete the build
> directory before recompiling. The generated `platformio.ini` is not fully
> regenerated on an existing build, so the old flash layout survives and you
> end up flashing a binary that does not match your config.
>
> ```bash
> rm -rf .esphome/build/irk-dongle
> ```
>
> In the dashboard the equivalent is the three-dot menu → **Clean Build Files**.

### Flash it

The simplest route is a direct USB install from the ESPHome dashboard
(**Install → Plug into this computer**), which compiles and flashes in one step
and always uses the binary it just built.

To flash a downloaded `.factory.bin` instead, use
[web.esphome.io](https://web.esphome.io) in Chrome or Edge — Firefox and Safari
have no WebSerial support. Connect, select the file, and let it write.

Or with esptool:

```bash
esptool.py --chip esp32s3 --port /dev/ttyACM0 --baud 921600 \
  write_flash 0x0 firmware.factory.bin
```

Replace `/dev/ttyACM0` with your port (`COM5` on Windows, `/dev/cu.usbmodem*`
on macOS). The dongle enumerates as USB VID `0x303a`, PID `0x1001`.

If the port never appears, hold the button on the dongle while plugging it in
to force download mode.

### Verifying you flashed the right thing

A successful connection prints the chip it found:

```
Chip is ESP32-S3 (QFN56) (revision v0.2)
Features: Wi-Fi, BLE
Flash ID: 1840ef
```

`Flash ID: 1840ef` decodes as Winbond, capacity `0x18` — 2^24 bytes, so
**16 MB**. That is the number `flash_size: 16MB` in the config refers to.

## Troubleshooting

**"Chip is ESP32-S3 but device expects esp32c6"** — you are flashing a binary
built for a different chip. The upstream irk-capture project targets an
ESP32-C6 dongle, so its ready-made web installer serves a C6 image and will
refuse your S3. Build from this configuration instead; do not use the upstream
installer page.

**"'name' is a required option for [esphome]"** — the upstream package supplies
an `esphome:` block without a name and expects your device file to provide one.
Make sure your config has a top-level `esphome:` block with `name:` in it.

**"This board is unknown"** — ESPHome's board list and PlatformIO's don't always
agree. Naming the variant explicitly satisfies the validator.

**"Please specify flash_size within esp32 configuration only"** — recent ESPHome
versions promoted `flash_size` to a first-class option, so it can no longer be
set through `platformio_options`. Move it into the `esp32:` block.

**Sketch too big** — the build is still using a 4 MB layout. Delete the build
directory and recompile, then confirm the generated
`.esphome/build/irk-dongle/platformio.ini` contains
`board_upload.flash_size = 16MB`.

**Strapping pin warnings for GPIO0 and GPIO3** — expected and safe to ignore.
Those pins are hard-wired on this dongle to the button and the LCD data line
respectively, so there is no alternative. The only practical consequence is
that holding the button during a reset boots into download mode.

**Blank, mirrored, or shifted display** — the ST7735 panels vary between
batches. Adjust `invert_colors`, `use_bgr`, and the `col_start` / `row_start`
offsets in the `display:` block; the rest of the configuration is unaffected.

**Compile errors about undeclared identifiers** (`ble_advertising`, `last_irk`,
`effective_mac`, and so on) — the display lambdas reference IDs defined by the
upstream package. If upstream renames one, the failure shows up here. Check the
current names with:

```bash
grep -nE "^\s+id: (ble_|last_|effective_)" \
  ".esphome/packages/*/ESPHome Devices/irk-capture-base.yaml"
```

## Customising

**Want to control the LED yourself from HA?** The `interval` block at the bottom
overrides manual control every 2 seconds. Remove that whole block and `RGB LED`
stays an ordinary controllable light.

**Different fonts or layout?** The screens are built with lambdas in the
`display` block; adjust coordinates and colours directly there.

> [!NOTE]
> `spi_led_strip` has no `rgb_order` option — the component simply assumes the
> LED is BGR, which is exactly what the APA102 on this dongle uses.

## Credits

- [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture) — the underlying project; this is only a device configuration on top of it
- [ESPHome](https://esphome.io)

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
