# Troubleshooting

### "Chip is ESP32-S3 but device expects esp32c6"

The image was built for another chip. The upstream IRK Capture web installer serves an ESP32-C6 image. Build from this configuration instead.

### "'name' is a required option for [esphome]"

The upstream package expects your file to define the device name. Make sure the configuration has a top-level `esphome:` block with `name:`.

### "This board is unknown"

Add `variant: esp32s3` to the `esp32:` block.

### "Please specify flash_size within esp32 configuration only"

Move `flash_size` from `platformio_options` into the `esp32:` block.

### Sketch too big

The build still uses a 4 MB layout. Delete the build folder, compile again and check that `.esphome/build/irk-dongle/platformio.ini` contains `board_upload.flash_size = 16MB`.

### Strapping pin warnings for GPIO0 and GPIO3

Expected. These pins are wired to the button and the screen. Holding the button during a reset enters download mode.

### Blank, mirrored or shifted display

ST7735 panels differ between batches. Adjust `invert_colors`, `use_bgr`, `col_start` and `row_start` in the `display:` block.

### Undeclared identifiers such as `ble_advertising` or `last_irk`

The display code uses IDs from the upstream package, which may have been renamed. List the current IDs:

```bash
grep -nE "^\s+id: (ble_|last_|effective_)" \
  ".esphome/packages/*/ESPHome Devices/irk-capture-base.yaml"
```
