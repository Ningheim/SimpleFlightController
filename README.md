# SimpleFlightController

<img width="733" height="611" alt="image" src="https://github.com/user-attachments/assets/fc7a2fe0-a7a5-4bca-8948-eb34f9dd7004" />

This is a USB-C powered 2-layer flight controller built around the STM32F722. It's 2S LiPo/Li-ion battery, a 6-axis IMU, a barometer, microSD logging, and two servo outputs.

## List of Materials

- MCU: STM32F722RET6, no SWD header, flash via USB DFU (BOOT0 + USB-C)
- IMU: ICM-20948 (SPI) — barometer: BME280 (I2C)
- Charger: BQ25887, 2S, I2C, cell balancing (needs the MID tap on J2)
- Power: TPS63070 buck-boost → VSYS, LMR51430 buck → 3.3V rail
- microSD: 4-bit SDIO + card detect
- J1/J3: servo/PWM headers, 3-pin (+5V, PWM, GND)
- J2: 2S battery screw terminal (BAT+, MID, GND)
- SW1 reset, SW2 boot, D1 status LED
- 25MHz HSE + 32.768kHz LSE crystals
- Board: 50x55mm, 2-layer, KiCad 
- the hardware folder has the KiCad files. Run lcsc.py first to get parts KiCAD didn't have. This came from the hack club guide.

## Power architecture

```
2S Battery ----> TPS63070 ----> VSYS ----> LMR51430 ----> 3.3V rail

USB-C VBUS ----> BQ25887 Charger
```

## How To Make It Yourself

1. Run `lcsc.py` so the libraries resolve.
2. Open the .kicad_pro file in KiCad.
3. Export gerbers/drill/pick-and-place and send to JLCPCB for a quote.
4. Generate a BOM and assemble with a stencil, paste, and reflow.
5. Power up: 2S LIPO pack on the battery pad, `J2`.
6. Flash: hold `SW2`, reset with `SW1`, flash over `USB1`.
