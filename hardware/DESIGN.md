# OpenVTX Hardware Design Document

Open-source 5.8GHz analog VTX with software-defined OSD. 20x20mm, 4-layer, 2S-6S input. Firmware: OpenOSD-X (STM32G4). The schematic and layout are not started yet; this document is the design reference for them.

## Key ICs

| IC | Part | LCSC | Package | Notes |
|----|------|------|---------|-------|
| MCU | STM32G431KBU6 | C529358 | QFN-32 5x5mm | Native OpenOSD-X target. |
| RF Synth | RTC6705 | C913074 | QFN-40 6x6mm | Consign. No alternative exists. |
| FEM | SKY85743-21 | C5348950 | LGA-24 3x5mm | 5V supply! Max +10dBm input! |
| Buck 5V | LMR51420YDDCR | C5383002 | SOT-23-6 | 1.1MHz, PFM. |
| LDO 3.3V | TLV75733PDRVR | C2868428 | WSON-6 2x2mm | 1A, 6.5uVrms noise |
| Rev Pol | AO3401A | C15127 | SOT-23 2.9x1.6mm | -30V, BASIC part, $0.05 |
| NTC | NCP15XH103F03RC | C77131 | 0402 | 10kOhm, B=3380K |
| Crystal | 8MHz 3225 | C367183 | SMD 3225 | For RTC6705 |
| ESD (x2) | PESD5V0S1BL | C2912556 | DFN1006-2 (0402) | One per UART line |
| Zener 12V | LNZ8F12VT5G-MS | C3018901 | SOD-882 (0402) | Gate clamp for AO3401A |
| Antenna | IPEX u.FL | C920232 | SMD | With pigtail to MMCX/SMA |
| FC conn | JST-SH 4P 1mm | C160404 | SMD | 5V, GND, TX, RX |
| WS2812 | WS2812C-2020-V1 | C2976072 | 2x2mm | Status LED |
| LED | 16-213/G6C-AK2L2VY/3T | C471422 | 0402 green | Power indicator |

## Passive BOM (0402 unless noted, JLCPCB basic parts)

| Value | LCSC | Use |
|-------|------|-----|
| 100pF C0G 50V | C1546 | RF bypass, crystal loads |
| 1nF X7R 50V | C1523 | SKY85743 VDD decoupling |
| 10nF X7R 50V | C15195 | HF decoupling |
| 100nF X7R 16V | C1525 | IC decoupling (all), CBOOT |
| 1uF X5R 25V | C52923 | RTC6705 REG1D8 decoupling |
| 4.7uF X5R 10V | C23733 | General bulk (5V and 3.3V rails) |
| 4.7uF X7R 50V (1206) | C29823 | Buck CIN (2x) |
| 10uF X5R 6.3V | C15525 | Buck COUT, SKY85743 VCC2 |
| 10pF C0G | - | RF DC blocking (SKY85743 TX_IN, DET) |
| 75R | C25133 | Video termination |
| 100R | C25076 | Series damping |
| 270R | C25187 | Video divider |
| 1k | C11702 | LED current limit |
| 1.2k | C25862 | Video divider |
| 10k | C25744 | NTC pullup, general |
| 13.7k | - | Buck FB bottom (RFBB) |
| 47k | C25792 | RTC6705 bias |
| 100k | C25741 | Buck FB top (RFBT), pulldowns |
| 510k | C11616 | Video bias |
| 560k | C25864 | Video bias |

## Consign Parts (1, source externally and supply to JLCPCB)

- **RTC6705**: 0 stock LCSC (checked 2026-08-05). No alternative. Source AliExpress/Taobao.

STM32G431KBU6 (C529358) and LMR51420YDDCR (C5383002) are in stock at LCSC and in the
JLCPCB assembly library (checked 2026-08-05): order via JLCPCB assembly, no consign needed.

## SKY85743-21 Pinout and Design Info

### Pinout (LGA-24, 3x5mm, top view)

| Pin | Name | Connect to |
|-----|------|-----------|
| 1 | GND | Ground |
| 2 | TX_IN | RTC6705 PAOUT via 6dB atten + 10pF DC block |
| 3 | GND | Ground |
| 4 | PA_EN | Tie to +5V (always TX) |
| 5 | GND | Ground |
| 6 | GND | Ground |
| 7 | GND | Ground |
| 8 | DET | MCU ADC (power detector output) |
| 9 | GND | Ground |
| 10 | C1 | Tie to +5V (TX mode) |
| 11 | GND | Ground |
| 12 | ANT | IPEX connector (50R trace) |
| 13 | C0 | Tie to GND (TX mode) |
| 14 | RX_OUT | NC (unused) |
| 15 | GND | Ground |
| 16 | LNA_IN | NC (unused) |
| 17 | LNA_OUT | NC (unused) |
| 18 | GND | Ground |
| 19 | VDD | 1nF to GND (LNA supply, still needs decoupling) |
| 20 | GND | Ground |
| 21 | GND | Ground |
| 22 | VCC3 | +5V, 100nF to GND |
| 23 | VCC2 | +5V, 10uF to GND |
| 24 | VCC1 | +5V, 2x 100nF to GND |
| 25 | GND | Exposed pad, thermal via array (0.3mm drill, 0.6mm pitch) |

### Critical specs

- **Supply: 4.2-5.5V** (powered from +5V buck rail)
- **Max TX_IN: +10 dBm absolute max**. RTC6705 outputs +13dBm, MUST attenuate.
- **TX mode truth table: PA_EN=1, C0=0, C1=1**
- **Detector: logarithmic, 0.33V@+5dBm to 0.93V@+28dBm, ~23mV/dB**
- **CW Psat: ~+27 dBm (500mW)** from MCS0 HT20 rating
- **Current: 190mA quiescent, 410mA @ +27dBm**
- **TX_IN is DC-shorted to GND internally, needs 10pF DC blocking cap**
- **Rugged: survives +10dBm input with 10:1 VSWR mismatch**

### Attenuator design (6dB pi-pad, 50R)

RTC6705 PAOUT (+13dBm) -> shunt 150R to GND -> series 37.4R -> shunt 150R to GND -> SKY85743 TX_IN (+7dBm).
Nearest standard 0402 values: 150R shunts, 36R or 39R series. Tune on bench.

## RTC6705 Pinout and Design Info

### Key pins (QFN-40, 6x6mm)

| Pin | Name | Connect to |
|-----|------|-----------|
| 1 | SPI_SE | Tie HIGH (SPI mode) |
| 5 | SPIDATA | MCU GPIO (bit-bang SPI) |
| 6 | SPILE | MCU GPIO (SPI latch/CS) |
| 7 | SPICLK | MCU GPIO (SPI clock) |
| 10 | VT_MOD | Video input (75R, AC-coupled, 1Vpp) |
| 23 | REG1D8_1 | 470nF to GND (internal 1.8V regulator) |
| 24 | XTAL1 | 8MHz crystal + load cap |
| 25 | XTAL2 | 8MHz crystal + load cap |
| 26 | REG1D8 | 100nF to GND (internal 1.8V regulator) |
| 27 | CP | PLL charge pump -> loop filter -> VT |
| 29 | VT | VCO tune voltage (from loop filter) |
| 34 | PAOUT2 | RF output (+13dBm, both PAOUTs connected) |
| 35 | PAOUT1 | RF output (connect with PAOUT2 for full power) |
| 33 | RFGND | Direct via to ground plane |
| 8,9,13,17,22,30,31,32,39,40 | Supply | All 3.3V, each 100nF decoupling |

### SPI protocol

- 3-wire: DATA, LE (latch), CLK
- 25-bit frame: 4 addr + 1 R/W + 20 data, LSB first
- Data on rising CLK edge, latch on rising LE edge
- Internal pullups on SPI pins

### Frequency formula

```
F_RF = 2 x (N x 64 + A) x (F_osc / R)
F_osc = 8MHz, R = 400 (default) -> 20kHz step
```

### Pit mode

Register 0x07, set PD_Q5G=1 to kill pre-driver (PA off, PLL stays locked).

## LMR51420YDDCR Design Values (5V output, 1.1MHz)

| Component | Value | Notes |
|-----------|-------|-------|
| L | 4.7uH | Shielded ferrite, Isat >= 4A, DCR < 100mOhm |
| CIN | 2x 4.7uF/50V X7R | Plus 100nF HF cap close to VIN/GND |
| COUT | 2x 10uF/16V X7R | |
| RFBT | 100k 1% | FB top (VOUT to FB) |
| RFBB | 13.7k 1% | FB bottom (FB to GND) |
| CBOOT | 100nF X7R 16V | CB to SW |
| EN | Tied to VIN | Always-on |

### Thermal note

At 6S/1A: marginal (110C rise). Fine at 500mA or less. VTX draws ~400mA input at 6S.

## AO3401A Reverse Polarity Protection

- SOT-23, -30V, 4A, 44mOhm Rdson
- Source on +BATT side, drain to buck VIN
- Gate: 100k pulldown to GND + 12V zener (LNZ8F12VT5G) to source
- Upstream TVS diode clamps transients before the MOSFET sees >30V
- At 400mA: 7mW loss. Negligible.

## Power Tree

```
+BATT -> AO3401A (rev pol) -> TVS clamp -> LMR51420YDDCR -> +5V
    +5V -> TLV75733P -> +3.3V -> STM32G431 + RTC6705
    +5V -> SKY85743-21 (VCC1/VCC2/VCC3)
    +5V -> Camera 5V output pad
```

## RF Chain

```
Camera (CVBS) -> 75R term -> STM32G4 OPAMP1 (OSD overlay) -> RTC6705 VT_MOD (pin 10)
RTC6705 PAOUT (+13dBm) -> 6dB pi-attenuator -> 10pF DC block -> SKY85743 TX_IN (pin 2)
SKY85743 ANT (pin 12) -> 50R microstrip -> IPEX connector
```

## VPD Power Control Loop

```
RTC6705 register 0x07 (PA power settings) -> adjusts RF drive level
SKY85743 DET (pin 8) -> MCU ADC
Firmware: read DET voltage -> compare to target VPD -> adjust RTC6705 PA register
Detector: logarithmic, ~23mV/dB, 0.33V (+5dBm) to 0.93V (+28dBm)
```

## Planned Schematic Structure

KiCad 9 hierarchical, root `OpenVTX.kicad_sch` plus 4 sub-sheets (not created yet):

- `mcu.kicad_sch`: STM32G431KBU6, crystal, SWD, UART, SPI, ADC, DAC, OPAMP
- `rf.kicad_sch`: RTC6705, attenuator, SKY85743-21, IPEX, loop filter
- `power.kicad_sch`: AO3401A, TVS, LMR51420, TLV75733, power indicator LED
- `pads.kicad_sch`: All user-facing solder pads, JST connector, test points

## User-Facing Interface

| Pad | Function | Notes |
|-----|----------|-------|
| +BATT | Battery positive (2-6S) | From PDB or ESC power lead |
| GND | Battery ground | Multiple pads for low impedance |
| 5V | Camera power output | Regulated from buck |
| VIN | Video input (CVBS) | From camera, 75R |
| TX | UART TX to FC | MSP DisplayPort |
| RX | UART RX from FC | MSP DisplayPort |
| SA | SmartAudio | Single-wire, optional fallback |
| BTN | Button | Channel/power cycle |
| LED | WS2812 data out | Status: channel, power, pit |
| IPEX | Antenna | 5.8GHz, use pigtail to MMCX/SMA |
| SWDIO | Debug | Test point only |
| SWCLK | Debug | Test point only |
| NRST | Reset | Test point only |

## PCB Specifications

- **Size**: 20x20mm, M2 mounting holes at 16x16mm
- **Layers**: 4-layer (JLCPCB JLC04161H-7628 stackup)
- **Layer assignment**: L1 = signal + RF, L2 = GND (unbroken under RF), L3 = power, L4 = signal
- **RF traces**: 50R microstrip on L1, solid L2 ground reference
- **Thermal vias**: 3x3 array under SKY85743 pad (0.3mm drill, 0.6mm pitch)
- **Copper**: 1oz standard (2oz outer if budget allows)
- **Antenna**: IPEX on board edge, keepout under connector

## Firmware

- Target: OpenOSD-X (STM32G4, GPL-3.0)
- Build: CMake + arm-none-eabi-gcc
- MCU is a native QFN-32 target, no pin remap needed
- VPD calibration table required per board (spectrum analyzer needed)
- VPD loop needs adaptation: SKY85743 logarithmic detector replaces RTC6671 linear detector
- Discord: https://discord.gg/9qxJNTPSPs

## Reference Documents

- `reference/043_OpenOSD-X_REFERENCE.pdf`: original schematic (RTC6705 + RTC6671)
- `reference/vpd_table.pdf`: VPD calibration table format and example
- `reference/Connection.png`: system block diagram
- `reference/datasheets/SKY85743-21_full_extraction.md`: complete pinout and specs
- `reference/datasheets/*.pdf`: component datasheets (STM32G431, RTC6705, LMR51420, NTC)
