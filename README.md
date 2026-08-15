# OpenVTX

Open-source 5.8GHz analog video transmitter with software-defined OSD, based on [OpenOSD-X](https://github.com/OpenOSD-X) firmware.

Early stage: parts are selected and project-local libraries are in place. Schematic and layout are not started yet. The full design reference lives in [hardware/DESIGN.md](hardware/DESIGN.md).

## Planned Features

### RF
- **RTC6705** 5.8GHz FM synthesizer: 40+ channels (A/B/E/F/R bands)
- **SKY85743-21** FEM: PA with integrated power detector, ~500mW CW output
- IPEX/u.FL antenna connector

### OSD
- Software-defined analog OSD via STM32G4 OPAMP/DAC/DMA, no MAX7456 needed
- SD (30x13/16) and HD (45x26/32) character modes
- NTSC/PAL auto-detection

### MCU
- **STM32G431KBU6**: ARM Cortex-M4F @ 170MHz, QFN-32
- MSP DisplayPort protocol (Betaflight native)
- SmartAudio V2.1 fallback for legacy FCs

### Power
- **2S-6S direct input** (7-36V) via LMR51420 buck to 5V
- 5V regulated output for camera
- 3.3V LDO (TLV757, low-noise) for MCU and RF synth
- Reverse polarity protection
- TVS input transient protection

### Safety
- NTC thermistor on PA for thermal auto-throttle
- Boot-to-pit-mode
- VPD closed-loop power control with logarithmic detector

## Repository Structure

```
OpenVTX/
├── README.md
├── hardware/
│   ├── DESIGN.md           ← Full design document (BOM, pinouts, RF chain, PCB specs)
│   ├── OpenVTX.kicad_pro
│   ├── lib.kicad_sym       ← 14 symbols (project-local)
│   ├── lib.pretty/         ← 14 footprints (project-local)
│   ├── lib.3dshapes/       ← 3D models
│   └── tools/              ← Python scripts
└── reference/
    ├── 043_OpenOSD-X_REFERENCE.pdf
    ├── vpd_table.pdf
    ├── Connection.png
    └── datasheets/         ← Component datasheets
```

## User Interface

| Pad/Connector | Function |
|---------------|----------|
| +BATT / GND | Battery input (2-6S) |
| 5V / GND | 5V output for camera |
| VIN / GND | Video input (CVBS) |
| TX / RX | UART to FC (MSP DisplayPort) |
| SA | SmartAudio (single-wire fallback) |
| IPEX | Antenna (5.8GHz via pigtail) |
| BTN | Channel/power cycle button |
| LED | WS2812 status indicator |
| SWD | Debug test points (SWDIO, SWCLK, NRST) |

## Key ICs

| IC | Part | LCSC | Package |
|----|------|------|---------|
| MCU | STM32G431KBU6 | C529358 | QFN-32 5x5mm |
| RF Synth | RTC6705 | C913074 | QFN-40 6x6mm |
| FEM | SKY85743-21 | C5348950 | LGA-24 3x5mm |
| Buck | LMR51420YDDCR | C5383002 | SOT-23-6 |
| LDO | TLV75733PDRVR | C2868428 | WSON-6 2x2mm |

## License

Licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt).
