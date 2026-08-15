# OpenVTX

An open 5.8 GHz analog video transmitter for the OpenDrone line: 20x20 mm,
2S-6S direct, MSP controlled, running the open [OpenOSD-X](https://github.com/OpenOSD-X)
firmware. Sits between an OpenFC and the camera; the FC talks to it over one
UART. Parts are chosen and the project-local library is drawn; schematic and
layout are not started.

[![Status](https://img.shields.io/endpoint?url=https://opendrone.be/api/status/OpenVTX.json)](https://github.com/OpenDrone-hw/.github/blob/main/CONTRIBUTING.md#the-life-of-a-project)
[![Discord](https://img.shields.io/badge/Discord-join-5865F2?logo=discord&logoColor=white)](https://discord.gg/v3sWmTcx3R)

Nobody holds this board yet: claim it on Discord.

## Why

Analog VTXs are cheap and plentiful, but pilots keep returning
the same complaints: output power that does not match the label, adjacent
channel bleed, no thermal protection, proprietary and flaky control protocols,
and firmware nobody can fix. OpenVTx, the one open firmware, stalled in 2022.
OpenOSD-X gives us an open stack on an STM32G4 with a
closed-loop power detector, MSP as the native protocol and a software OSD that
replaces the MAX7456. What is missing is a board built for it that people can
buy: this repo.

## Specifications

| | |
|---|---|
| Size | 20 x 20 mm, M2 holes at 16 x 16 mm |
| Layers | 4 |
| Input | 2-6S (7-36 V) |
| Output | 5 V for the camera |
| RF | RTC6705 synthesizer, 40 channels A/B/E/F/R |
| PA | SKY85743-21 FEM, about 500 mW CW |
| Power control | Closed loop, logarithmic detector, pit mode |
| MCU | STM32G431KBU6 |
| Control | MSP DisplayPort, SmartAudio V2.1 fallback |
| OSD | Software defined, SD and HD character modes, PAL/NTSC auto |
| Antenna | IPEX u.FL |
| Firmware | OpenOSD-X, GPL-3.0 |

Targets, not measurements. Part list, pinouts, RF chain, power tree and layout
rules: [research/DESIGN.md](research/DESIGN.md).

## Constraints

- Firmware is OpenOSD-X as it exists. The STM32G431KBU6 is a native target;
  the VPD loop needs adapting because the SKY85743 detector is logarithmic
  where the reference RTC6671 is linear.
- The SKY85743-21 runs from 5 V and takes at most +10 dBm in; the RTC6705 puts
  out +13 dBm, so a pad sits between them.
- JLCPCB assembly from LCSC parts, 0402 basics for passives. The RTC6705 has
  no LCSC stock and no alternative, so it is a consigned part.
- 4-layer JLC04161H-7628 stackup, unbroken ground under the RF section, 50 R
  microstrip to the antenna.
- Pads for battery, camera 5 V, video in, UART, SmartAudio, button and WS2812;
  JST-SH 4P to the FC; SWD on test points.

## Prior art

- [research/DESIGN.md](research/DESIGN.md): the design reference this board is drawn from.
- [OpenOSD-X](https://github.com/OpenOSD-X): firmware and the reference schematic, [reference/043_OpenOSD-X_REFERENCE.pdf](reference/043_OpenOSD-X_REFERENCE.pdf).
- [reference/vpd_table.pdf](reference/vpd_table.pdf): VPD calibration table format.
- [reference/Connection.png](reference/Connection.png): system block diagram.
- tinyFINITY: STM32F3 + RTC6705 with a software OSD at 16 x 16 mm, no external PA. Closest open hardware precedent.

## Open questions

- Software OSD on the VTX or plain passthrough? Pilots mostly want the FC to
  draw the OSD; the point of OpenOSD-X is a software OSD with no
  MAX7456. Decide before the video path is drawn.
- Antenna connector: IPEX is chosen for size, pilots rate MMCX and u.FL both
  poorly for crash survival. A pigtail to MMCX or SMA is the current answer.
- Thermal budget: about 500 mW CW in 20 x 20 mm needs a heatsink path. Bare
  board, enclosure as heatsink, or auto-throttle only?
- The 6 dB pad and the VPD calibration table need a spectrum analyzer on the
  bench; who has one?
- Antenna detection or VSWR foldback, the most asked-for missing feature: is
  it doable with the SKY85743 detector alone?

## In the line

What pairs with what, and what is available:
[opendrone.be](https://opendrone.be).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
