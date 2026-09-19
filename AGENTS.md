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
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| KiCad project | `hardware/OpenVTX.kicad_pro`; no `.kicad_sch` or `.kicad_pcb` exists |
| Design reference | `hardware/DESIGN.md` |
| Local library | `hardware/lib.kicad_sym` (14 symbols), `hardware/lib.pretty/` (14 footprints), `hardware/lib.3dshapes/`, nickname `lib` |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Board tools | `hardware/tools/add_mpn_fields.py`: fills MPN and Manufacturer symbol properties from LCSC numbers through the jlcsearch API with kicad-skip; dry run by default, `--write` to change files |
| Reference | `reference/`: OpenOSD-X reference schematic PDF, VPD table, connection drawing, component datasheets |
| Research | `research/`: sourcing notes, reference material not decisions |
| License | CERN-OHL-S-2.0 |

## Environment

There is no schematic or board to check yet. Once they exist, the ERC, DRC
and netlist commands are the template's, on `hardware/OpenVTX.kicad_sch` and
`hardware/OpenVTX.kicad_pcb`. On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`; `KPY` is KiCad's
bundled Python,
`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`.

## Rules

Identical in every OpenDrone board repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check the `OpenDrone` library and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it, and its symbol links to the exact committed
  datasheet: place it from `OpenDrone`. Draw a new part into `lib` only when
  the catalogue has nothing that fits, imported with
  `easyeda2kicad` from its LCSC number. Pulling a newer catalogue is a
  deliberate, reviewed commit: `git submodule update --remote
  hardware/KiCad-Library`, then DRC.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Run ERC and DRC before every pull request.** Existing approved findings
  may remain; a new type or increased count must be reviewed before merge.
  Commands are in Environment above.

## By task

Board-specific paths are in Environment above. `KPY` is KiCad's bundled
Python named there.

- Draw the schematic: only on request, from `hardware/DESIGN.md`, in KiCad, into `hardware/OpenVTX.kicad_sch`; then replace the Environment section above with the template's commands.
- Add a part: place it from the `OpenDrone` library if `hardware/KiCad-Library/PARTS-USED.md` lists it; otherwise import it into `lib` with `$KPY <hardware-tooling>/hardware/kicad/import_part.py --repo hardware` (read `--help` first), KiCad closed.
- Fill supplier fields once a schematic exists: `python3 hardware/tools/add_mpn_fields.py` to preview, `--write` to apply, KiCad closed.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, commit as its own reviewed change; run DRC once a board exists.
- Answer an RF or OSD question: read `hardware/DESIGN.md` and the PDFs in `reference/`; the RTC6705 sourcing question is in `research/rtc6705-replacement.md`.
