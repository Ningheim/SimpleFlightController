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

1. **Pull in libraries:** run `lcsc.py` as above so the schematic/PCB resolve correctly.
2. **Open the project** in KiCad (`hardware/SimpleFlightController.kicad_pro`).
3. **Fabricate:** export gerbers, drill files, and pick-and-place data from the PCB editor and send them to JLCPCB for a quote and print.
4. **Assemble:** generate a BOM from the schematic for sourcing. Most footprints are 0201/0402, so I recommend hand assembly with a stencil, paste, and hot air or reflow.
5. **Power up:** connect a 2S LiPo/Li-ion pack to `J2`, balance lead included, or run off `USB1` alone for bring-up without a battery.
6. **Flash firmware:** hold `SW2` (BOOT0), reset or power-cycle, then flash over `USB1` in DFU mode.
