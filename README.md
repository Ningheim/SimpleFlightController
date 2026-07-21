# SimpleFlightController

A compact, USB-C powered flight controller board built around the STM32F722, with onboard 2S LiPo/Li-ion charging, a 6-axis IMU, a barometer, microSD logging, and two servo/PWM outputs.

- **Board size:** 50 mm x 55 mm
- **Layers:** 2 (top/bottom copper)
- **Design tool:** KiCad (Pcbnew 10.0.2)

## Features

- **MCU:** STM32F722RET6 (Arm Cortex-M7, LQFP-64) with native USB Full-Speed
- **IMU:** ICM-20948 6-axis accelerometer/gyroscope (SPI)
- **Barometer:** BME280 pressure/temperature/humidity sensor (I2C)
- **Onboard storage:** microSD card slot wired over 4-bit SDIO, plus card-detect
- **USB-C:** power input and USB FS data (used for DFU firmware flashing)
- **2S battery charging:** BQ25887 I2C-controlled boost-mode charger with integrated cell balancing for 2S Li-ion/LiPo packs
- **Power regulation:**
  - TPS63070 buck-boost converter generating the main system rail (VSYS) directly from the battery, so output stays regulated across the full charge/discharge range
  - LMR51430 synchronous buck converter for a secondary low-voltage rail feeding the MCU and sensors
- **2x servo/PWM outputs** (J1, J3) for driving servos or ESCs
- **Status LED, reset button, and boot (DFU) button**
- **25 MHz HSE crystal** for the MCU's main clock and a **32.768 kHz LSE crystal** for RTC timekeeping

## Connectors & controls

| Ref | Type | Function | Pinout |
|---|---|---|---|
| `USB1` | USB-C | Power input + USB FS data | Standard USB-C (D+/D-, CC1/CC2) |
| `J2` | 3-pin screw terminal | 2S battery input | 1: BAT+, 2: MID (cell balance tap), 3: GND |
| `J1` | 2x03 header (3 pins used) | Servo/PWM output 1 | 1: +5V, 2: SERVO1 (PWM), 3: GND |
| `J3` | 2x03 header (3 pins used) | Servo/PWM output 2 | 1: +5V, 2: SERVO2 (PWM), 3: GND |
| `CARD1` | microSD (push-pull) | Onboard logging/storage | 4-bit SDIO + card detect |
| `SW1` | Tactile switch | Reset (NRST) | — |
| `SW2` | Tactile switch | BOOT0 (hold at power-up to enter USB DFU mode) | — |
| `D1` | LED | Status indicator | — |

**Note on the battery input:** `J2` breaks out the mid-point tap of a 2S pack (`MID`) in addition to `BAT+`/`GND`. This is required by the BQ25887's cell-balancing feature, so the battery pack needs its balance lead connected here, not just the main pack leads.

**Note on programming:** No SWD debug header is populated on this board — the MCU's SWDIO/SWCLK pins are unconnected. Firmware is intended to be flashed over USB using the STM32 built-in DFU bootloader: hold `SW2` (BOOT0) while powering up or resetting via `SW1`, then flash over `USB1` with a tool such as STM32CubeProgrammer or `dfu-util`.

## Power architecture

```
2S Battery ──┬─────────────► TPS63070 (buck-boost) ──► VSYS ──► LMR51430 (buck) ──► 3.3V rail (MCU + sensors)
             │
USB-C VBUS ──┴──► BQ25887 (2S boost charger, I2C) ──► charges battery, balances cells
```

The BQ25887 charger shares the I2C bus with the BME280 barometer, since both are I2C-addressable devices.

## Sensor/peripheral buses

- **SPI:** ICM-20948 IMU (SCK, SDI, SDO, nCS, INT1, INT2, FSYNC)
- **I2C:** BME280 barometer + BQ25887 charger (SDA, SCL)
- **SDIO (4-bit):** microSD card (CLK, CMD, DAT0-3, card detect)
- **USB FS:** STM32 native USB peripheral, tied to `USB1`

## What's in this repo

| File | Description |
|---|---|
| `bom.csv` | Full bill of materials (designators, footprints, values, LCSC part numbers) |
| `designators.csv` | Reference designator list with quantities, for quick component counts |
| `positions.csv` | Pick-and-place / component placement data (designator, X/Y, rotation, side) |
| `netlist.ipc` | IPC-D-356 netlist, useful for bare-board electrical test / continuity checks |
| `SimpleFlightController.zip` | Fabrication gerbers + drill files (copper, mask, silkscreen, paste, edge cuts, PTH/NPTH drills and drill maps) |

**Not yet included:** schematic (PDF or KiCad source), assembly drawings, and 3D/STEP model. If you add these later, this is a good place to link them.

## Getting started

1. **Fabricate:** send the contents of `SimpleFlightController.zip` to your PCB fab (Gerber X2 format, from KiCad).
2. **Assemble:** use `bom.csv` for sourcing and `positions.csv` for pick-and-place, or hand-assemble using the same files as a reference (note several passives are 0201/0402 — fine-pitch tweezers/hot air or a stencil + reflow oven are recommended).
3. **Power up:** connect a 2S LiPo/Li-ion pack to `J2` (including the balance/MID lead), or power solely from `USB1` for bring-up/testing without a battery attached.
4. **Flash firmware:** hold `SW2` (BOOT0), reset or power-cycle the board, then connect over `USB1` and flash via DFU.
