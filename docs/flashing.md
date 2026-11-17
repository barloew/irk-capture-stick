# Flashing

## Factory or OTA image

| File | Contains | Flash at | Use for |
|---|---|---|---|
| `firmware.factory.bin` | Bootloader, partition table and app | `0x0` | First flash of a blank dongle or one with other firmware |
| `firmware.ota.bin` | App only | n/a | Over-the-air updates after the first flash |

Flashing the OTA image to a new dongle appears to succeed, but the device will not boot.

## Board configuration

```yaml
esp32:
  board: esp32-s3-devkitc-1
  variant: esp32s3
  flash_size: 16MB
```

Both `variant` and `flash_size` are required. Without `variant`, validation fails with "This board is unknown". Without `flash_size`, ESPHome uses 4 MB, which leaves too little room for OTA updates.

After changing `board`, `variant` or `flash_size`, delete the build folder before compiling again (`rm -rf .esphome/build/irk-dongle`, or **Clean Build Files** in the dashboard). Otherwise the old flash layout is kept.

## Build the factory image

- **Dashboard:** open the device, select **Install → Manual download** and choose **Factory format**.
- **Command line:** run `esphome compile irk-dongle.yaml`. The image is written to `.esphome/build/irk-dongle/.pioenvs/irk-dongle/firmware.factory.bin`. In Docker, this path is relative to the folder mounted as `/config`.

## Flash

- **Dashboard:** **Install → Plug into this computer** compiles and flashes in one step.
- **Browser:** open [web.esphome.io](https://web.esphome.io) in Chrome or Edge and select the `.factory.bin` file.
- **esptool:**

  ```bash
  esptool.py --chip esp32s3 --port /dev/ttyACM0 --baud 921600 \
    write_flash 0x0 firmware.factory.bin
  ```

  Use your own port, such as `COM5` on Windows or `/dev/cu.usbmodem*` on macOS. The dongle appears as USB VID `0x303a`, PID `0x1001`.

If no port appears, hold the button while plugging in the dongle to enter download mode.

## Verify

esptool prints the detected chip:

```
Chip is ESP32-S3 (QFN56) (revision v0.2)
Features: Wi-Fi, BLE
Flash ID: 1840ef
```

`1840ef` is a Winbond chip with capacity `0x18`, which is 16 MB.
