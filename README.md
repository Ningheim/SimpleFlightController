# SimpleFlightController

A compact, USB-C powered flight controller built around the STM32F722. It's got onboard 2S LiPo/Li-ion charging, a 6-axis IMU, a barometer, microSD logging, and two servo/PWM outputs — everything wired up on a 50 x 55 mm, 2-layer board.

Designed in KiCad 10.

## Features

- **MCU:** STM32F722RET6 (Arm Cortex-M7, LQFP-64) with native USB Full-Speed
- **IMU:** ICM-20948 6-axis accelerometer/gyroscope (SPI)
- **Barometer:** BME280 pressure/temperature/humidity sensor (I2C)
- **Onboard storage:** microSD slot wired over 4-bit SDIO, with card detect
- **USB-C** for power and USB FS data, used for DFU firmware flashing
- **2S battery charging** via the BQ25887, an I2C-controlled boost charger with built-in cell balancing
- **Power regulation:**
  - TPS63070 buck-boost converter builds the main system rail (VSYS) straight off the battery, so the rail stays regulated across the whole charge/discharge curve
  - LMR51430 buck converter steps that down to a secondary low-voltage rail for the MCU and sensors
- **Two servo/PWM outputs** (J1, J3) for driving servos or ESCs
- Status LED, reset button, and a boot (DFU) button
- 25 MHz HSE crystal for the main clock, plus a 32.768 kHz LSE crystal for the RTC

## Connectors & controls

| Ref | Type | Function | Pinout |
|---|---|---|---|
| `USB1` | USB-C | Power in + USB FS data | Standard USB-C (D+/D-, CC1/CC2) |
| `J2` | 3-pin screw terminal | 2S battery input | 1: BAT+, 2: MID (cell balance tap), 3: GND |
| `J1` | 2x03 header (3 pins used) | Servo/PWM output 1 | 1: +5V, 2: SERVO1 (PWM), 3: GND |
| `J3` | 2x03 header (3 pins used) | Servo/PWM output 2 | 1: +5V, 2: SERVO2 (PWM), 3: GND |
| `CARD1` | microSD | Onboard logging/storage | 4-bit SDIO + card detect |
| `SW1` | Tactile switch | Reset (NRST) | — |
| `SW2` | Tactile switch | BOOT0 (hold at power-up for USB DFU) | — |
| `D1` | LED | Status indicator | — |

`J2` breaks out the mid-point tap of a 2S pack (`MID`) alongside `BAT+`/`GND` — the BQ25887 needs that balance lead connected, not just the main pack leads, or cell balancing won't work.

There's no SWD header on this board; SWDIO/SWCLK are left unconnected on the MCU. Firmware goes on over USB instead: hold `SW2` (BOOT0) while powering up or resetting with `SW1`, then flash over `USB1` with STM32CubeProgrammer or `dfu-util`.

## Power architecture

```
2S Battery ──┬─────────────► TPS63070 (buck-boost) ──► VSYS ──► LMR51430 (buck) ──► 3.3V rail (MCU + sensors)
             │
USB-C VBUS ──┴──► BQ25887 (2S boost charger, I2C) ──► charges battery, balances cells
```

The BQ25887 shares the I2C bus with the BME280 barometer, since both are I2C devices.

## Sensor & peripheral buses

- **SPI:** ICM-20948 IMU (SCK, SDI, SDO, nCS, INT1, INT2, FSYNC)
- **I2C:** BME280 barometer + BQ25887 charger (SDA, SCL)
- **SDIO (4-bit):** microSD card (CLK, CMD, DAT0-3, card detect)
- **USB FS:** STM32's native USB peripheral, tied to `USB1`

## Repo layout

```
hardware/
  SimpleFlightController.kicad_pro   KiCad project file
  SimpleFlightController.kicad_sch   Schematic
  SimpleFlightController.kicad_pcb   PCB layout
  lcsc.py                            Pulls parts from LCSC into KiCad libraries
  lcsc.txt                           LCSC part numbers used by lcsc.py
lib/lcsc/
  easyeda2kicad.kicad_sym            Symbol library generated from lcsc.txt
```

Most passives, the STM32, and the crystals use standard KiCad libraries. The IMU, barometer, TPS63070, LMR51430, USB-C connector, and microSD socket don't ship with KiCad by default, so their symbols/footprints are pulled in from LCSC/EasyEDA instead. `lcsc.py` runs [easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py) against each part number in `lcsc.txt` and drops the results in `lib/lcsc/`:

```
python hardware/lcsc.py hardware/lcsc.txt --output ./lib/lcsc
```

Run that before opening the project for the first time, or KiCad will complain about missing symbols.

## Getting started

1. **Pull in libraries:** run `lcsc.py` as above so the schematic/PCB resolve correctly.
2. **Open the project** in KiCad 10 (`hardware/SimpleFlightController.kicad_pro`).
3. **Fabricate:** export gerbers, drill files, and pick-and-place data from the PCB editor and send them to your fab of choice.
4. **Assemble:** generate a BOM from the schematic for sourcing. Heads up — several passives are 0201/0402, so hand assembly is easiest with a stencil, paste, and hot air or reflow rather than a soldering iron alone.
5. **Power up:** connect a 2S LiPo/Li-ion pack to `J2`, balance lead included, or run off `USB1` alone for bring-up without a battery.
6. **Flash firmware:** hold `SW2` (BOOT0), reset or power-cycle, then flash over `USB1` in DFU mode.

## License

*Add your chosen hardware license here (e.g. CERN-OHL-S, TAPR OHL, MIT for any accompanying firmware/scripts).*
