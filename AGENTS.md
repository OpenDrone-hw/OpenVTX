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

## Environment

There is no schematic or board to check yet. Once they exist, the ERC, DRC
and netlist commands in Board rules, Commands, apply to the schematic and board
files beside `hardware/OpenVTX.kicad_pro`.

## Rules

The [Board rules](#board-rules) at the end of this file apply, a CI-checked copy of the [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) section. Only what differs for this board is listed here.

## By task

- Draw the schematic: only on request, from `hardware/DESIGN.md`, in KiCad, into the schematic beside `hardware/OpenVTX.kicad_pro`.
- Fill supplier fields once a schematic exists: `python3 hardware/tools/add_mpn_fields.py` to preview, `--write` to apply, KiCad closed.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, commit as its own reviewed change; run DRC once a board exists.
- Answer an RF or OSD question: read `hardware/DESIGN.md` and the documents its Reference Documents section links; the RTC6705 sourcing question is in `research/rtc6705-replacement.md`.

## Board rules

Shared by every OpenDrone KiCad board repository, byte for byte, and checked
in CI against [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules).
Edit them there, never in a board.

### Files

Every board has this layout; its Repo table lists only what differs.

| | |
|---|---|
| Designed in | KiCad 10 |
| Status | The `status-*` topic on the repository, never written in a file |
| Local library | `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`, nickname `lib` |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

A repository holds design files only. Sourcing (suppliers, prices, quotes, RFQs, contacts) is handled by Incutec and never lives in OpenDrone repositories.

### Parts and datasheets

- **Per-repository part index:** the root schematic and its listed sub-sheets
  are authoritative for what this board fits. Export the netlist with the
  command below when a script-readable board index is needed; do not maintain
  a second hand-written BOM.
- **Proven shared parts:**
  `hardware/KiCad-Library/PARTS-USED.md` is the catalogue index. Its `Boards`
  column identifies every repository using each LCSC part; filter that column
  for the repository name to get its proven shared-part view.
- **Exact datasheets:**
  `hardware/KiCad-Library/datasheet/manifest.json` maps shared symbols to the
  committed PDFs and their SHA-256 hashes. The PDFs live in
  `hardware/KiCad-Library/datasheet/`, and symbol links resolve there through
  `OPENDRONE_LIB`.
- **Local-only parts:** inspect `hardware/lib.kicad_sym`,
  `hardware/lib.pretty/`, and `hardware/lib.3dshapes/`, then verify supplier
  fields in the board schematic. Do not duplicate a shared part or datasheet.

### Commands

`<name>` is the KiCad project named in the board's Repo table.

```sh
# schematic and board checks
kicad-cli sch erc hardware/<name>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/<name>.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/<name>.net hardware/<name>.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python, `KPY`:
`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`.
Reusable scripts for renders, STEP export and packaging art come from Incutec
hardware tooling. The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
Board-specific scripts, where a board has any, live in `hardware/tools/`.

### Rules

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
  kicad-skip cannot round-trip KiCad 10 schematics: use a full S-expression
  parser, `kicad-cli sch upgrade --force`, then an ERC and netlist diff.
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
  The commands are under Commands.

### By task

- Check the design: run the ERC and DRC commands in Commands before every pull request.
- Add a part: place it from the `OpenDrone` library if `hardware/KiCad-Library/PARTS-USED.md` lists it; otherwise import it into `lib` with `$KPY <hardware-tooling>/hardware/kicad/import_part.py` (read `--help` first), KiCad closed.
- Render the board for the README: `$KPY <hardware-tooling>/hardware/kicad/render_board.py hardware/<name>.kicad_pcb --outdir images`, KiCad closed.
- Analyse the netlist: export it with the netlist command in Commands, then read it with a script; never hand-write a second BOM.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, run DRC, commit as its own reviewed change.
