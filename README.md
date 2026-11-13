# Usagi Core

Flight computer core board by **Usagi Avionica**. An STM32G474 with on-board flash logging, CAN, USB-C and connectors for expansion boards, on a 50 × 50 mm four-layer board designed to be stacked.

<p align="center">
  <img src="https://github.com/usagi-avionica/usagi-core/releases/latest/download/isometric_top.png" width="45%" alt="Usagi Core, top">
  <img src="https://github.com/usagi-avionica/usagi-core/releases/latest/download/isometric_bottom.png" width="45%" alt="Usagi Core, bottom">
</p>

## Features

- **MCU:** STM32G474RE (Arm Cortex-M4F, 170 MHz), 24 MHz crystal, 32.768 kHz RTC
- **Storage:** 256 Mbit (32 MB) SPI NOR flash (Infineon S25FL256L)
- **CAN:** classic CAN up to 1 Mbit/s (TI SN65HVD231Q), split termination on board, ESD protection
- **USB:** USB-C, ESD protected; can also power the board through a jumper for debug
- **Power:** 5 V input or USB into a 3.3 V / 1.2 A LDO
- **Expansion:** I2C, SPI, an 8-pin GPIO/ADC connector and 13 GPIO solder pads on the bottom side
- **Feedback:** buzzer, status LED, power indicator LEDs
- **Connectors:** Molex Micro-Lock Plus 1.25 mm with positive locks

## Specifications

| | |
|---|---|
| Board size | 50 × 50 mm |
| Mounting | 4 × M3, 3.5 mm from each corner (43 × 43 mm pattern) |
| Layers | 4, 1.6 mm, ENIG |
| Input voltage | 5 V nominal, **5.5 V maximum** |
| Logic level | 3.3 V |

## Connectors

All board-to-wire connectors are Molex Micro-Lock Plus 1.25 mm right-angle headers (220098 series). Mating housing: Molex 204532 series.

### J6: Power input

| Pin | Signal |
|---|---|
| 1 | VIN (5 V nominal, 5.5 V max) |
| 2 | GND |

### J3: CAN

| Pin | Signal |
|---|---|
| 1 | CANL |
| 2 | CANH |
| 3 | GND |

Split termination (2 × 60.4 Ω + 4.7 nF) is fitted on the board.

### J4: I2C (I2C3)

| Pin | Signal |
|---|---|
| 1 | SCL |
| 2 | SDA |
| 3 | GND |

1.8 kΩ pull-ups to 3.3 V on board.

### J5: SPI (SPI2)

| Pin | Signal |
|---|---|
| 1 | SCK |
| 2 | MISO |
| 3 | MOSI |
| 4 | GND |

There are no chip-select lines on this connector. Use pins from J7 or the solder pads for chip selects. Each SPI device should pull its own CS line up, so it stays deselected while the MCU is in reset.

### J2: Debug

| Pin | Signal |
|---|---|
| 1 | 3.3 V (VTref, reference only; does not power the board) |
| 2 | SWDIO |
| 3 | SWCLK |
| 4 | SWO |
| 5 | UART TX (USART2, from the MCU) |
| 6 | UART RX (USART2, to the MCU) |
| 7 | NRST |
| 8 | GND |

### J7: GPIO / ADC

| Pin | MCU pin | ADC channel |
|---|---|---|
| 1 | PC4 | ADC2_IN5 |
| 2 | PC5 | ADC2_IN11 |
| 3 | PC2 | ADC12_IN8 |
| 4 | PC1 | ADC12_IN7 |
| 5 | PC0 | ADC12_IN6 |
| 6 | PA8 | ADC5_IN1 |
| 7 | PB12 | ADC1_IN11 / ADC4_IN3 |
| 8 | PB11 | ADC12_IN14 |

Every pin can be a GPIO or an ADC input, and each uses a different EXTI line, so all eight can be interrupts at the same time. This connector has no ground pin; each daughter board takes its ground from its power or CAN connection.

> [!WARNING]
> J7 has **no ESD protection or filtering**. Keep signals between 0 and 3.3 V. In analog mode none of these pins are 5 V tolerant.

### GPIO solder pads (bottom side)

The remaining free GPIOs are on solder pads on the bottom of the board, labelled on the silkscreen:

PA10, PA15, PB4, PB7, PB9, PB10, PC6, PC7, PC10, PC11, PC12, PC13, PD2

These pads are unprotected and meant for development and bodge wires. Secure any wire that stays on in flight. PC13 can only source or sink a few mA, so use it only for slow signals.

### J1: USB-C

USB 2.0 full-speed device. It powers the board only when the USB EN jumper is fitted.

## Power

The board runs from J6 (5 V) or USB. The two are diode-ORed (Schottky) into a 3.3 V LDO, so both can be connected at once.

| Jumper | Function |
|---|---|
| USB EN | Fitted: USB powers the board. Removed: USB carries data only. |

| LED | Meaning |
|---|---|
| PWR IN | External power (J6) present |
| USB PWR | USB power present (J21 fitted) |
| 3.3V | 3.3 V rail on |
| STATE | Status LED, driven by firmware |

## Programming

- **SWD** through J2 with any Arm debug probe (ST-LINK, J-Link, etc.).
- **USB DFU bootloader:** set the **BOOT0** switch (SW1) to the BOOT0 position and reset the board. It should then show up on USB as an ST DFU device (STM32CubeProgrammer or `dfu-util`).

## Manufacturing

Designed in **KiCad 10**. The project includes design rules (`usagi-core.kicad_dru`) for PCBWay's standard 4-layer service.

## Releases

Releases are built automatically by GitHub Actions. When a change merged to `main` comes with a new board revision, the workflow:

1. Runs ERC and DRC (errors block the release)
2. Generates the outputs below
3. Publishes a release named **Revision X** with everything in one `release.zip`

| File | Contents |
|---|---|
| `gerber.zip` | Gerbers and drill files, ready to upload to a board house |
| `odb++.zip` | ODB++ fabrication data |
| `schematic.pdf` | Schematic |
| `pcb.pdf` | Copper, mask and fabrication layers |
| `bom.csv` | Bill of materials, grouped by manufacturer part number |
| `ibom.html` | Interactive BOM for assembly |
| `netlist.net` | Netlist |
| `isometric_top.png`, `isometric_bottom.png` | 3D renders |
| `erc.rpt`, `drc.rpt` | Check reports |

**To publish a new revision:** change the Revision field in the Page Settings of both the schematic and the PCB (e.g. `A` → `B`), then merge to `main`. Merges that don't change the revision don't produce a release.

## License

Hardware licensed under **CERN-OHL-S**. See [LICENSE](LICENSE).