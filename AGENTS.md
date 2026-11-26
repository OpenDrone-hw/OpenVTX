# OpenVTX

5.8 GHz analog video transmitter with a software-defined OSD, built for the
[OpenOSD-X](https://github.com/OpenOSD-X) firmware on an STM32G431. Parts are
selected and the project-local libraries exist; there is no schematic and no
board yet. The part list, pinouts, RF chain and board targets are in
[hardware/DESIGN.md](hardware/DESIGN.md), the design reference for the
schematic to come. Do not turn its values into specifications until the KiCad
files carry them.

## Repo

| | |
|---|---|
| KiCad project | `hardware/OpenVTX.kicad_pro`; no `.kicad_sch` or `.kicad_pcb` exists |
| Design reference | `hardware/DESIGN.md` |
| Local library | `hardware/lib.kicad_sym` (14 symbols), `hardware/lib.pretty/` (14 footprints), `hardware/lib.3dshapes/`, nickname `lib` |
| Board tools | `hardware/tools/add_mpn_fields.py`: fills MPN and Manufacturer symbol properties from LCSC numbers through the jlcsearch API with kicad-skip; dry run by default, `--write` to change files |
| Reference | `hardware/DESIGN.md` "Reference Documents": links to the OpenOSD-X reference schematic, VPD table, block diagram and component datasheets; `reference/datasheets/SKY85743-21_full_extraction.md` |
| Research | `research/`: technical notes, reference material not decisions |

## Rules

The shared [Board rules](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) apply. Only what differs for this board is listed here.

## By task

- Draw the schematic: only on request, from `hardware/DESIGN.md`, in KiCad, into the schematic beside `hardware/OpenVTX.kicad_pro`.
- Fill supplier fields once a schematic exists: `python3 hardware/tools/add_mpn_fields.py` to preview, `--write` to apply, KiCad closed.
- Answer an RF or OSD question: read `hardware/DESIGN.md` and the documents its Reference Documents section links; the RTC6705 sourcing question is in `research/rtc6705-replacement.md`.
