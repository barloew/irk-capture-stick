# Customizing

## Screens

| Screen | Shows |
|---|---|
| 1. Status | Advertising on or off, BLE name, MAC address and the first 12 characters of the IRK |
| 2. Latest IRK | The full key on two lines of 16 characters, and the address it came from |
| 3. System | SSID, IP address and the active BLE profile |

The screens are drawn by lambdas in the `display` block. Change fonts, coordinates and colors there.

## LED

The `interval` block at the end of the configuration sets the LED color every 2 seconds and overrides manual control. Remove the block to control `RGB LED` as a regular light from Home Assistant.

`spi_led_strip` has no `rgb_order` option and assumes BGR, which matches the APA102 on this dongle.

## Pinout

The screen and LED use separate SPI buses.

| Function | Pin |
|---|---|
| LCD SPI clock | GPIO5 |
| LCD SPI MOSI | GPIO3 |
| LCD CS | GPIO4 |
| LCD DC | GPIO2 |
| LCD reset | GPIO1 |
| LCD backlight (PWM, inverted) | GPIO38 |
| LED SPI clock | GPIO39 |
| LED SPI MOSI | GPIO40 |
| Button (inverted) | GPIO0 |
