# Proposal

## Why

The current ATX Power Supply Controller PCB layout uses a two-layer copper stackup (`F.Cu` signal traces with a `B.Cu` ground plane), which requires plated through-holes (PTH) and commercial two-layer fabrication. Converting the board to a true single-sided PCB layout—with all through-hole components mounted on the top side and all copper traces and ground pour exclusively on the bottom copper layer (`B.Cu`)—enables low-cost DIY home etching (toner transfer or photoresist), desktop CNC isolation milling, and single-layer FR1/FR4 manufacturing without plated vias or top-side soldering.

## What Changes

- Re-architect the KiCad PCB layout (`kicad/atx-psu-controller.kicad_pcb`) from a 2-layer design to a 100% single-sided (`B.Cu`-only) copper layout with zero traces on `F.Cu` and zero vias.
- Place all discrete through-hole components from the BOM (`J1`–`J5`, `Q1`–`Q5`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`, `R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`) on the top component side (`F.Cu` footprint insertion side) and use axial resistor/diode spans as natural top-side crossovers so all copper routing remains strictly on `B.Cu`.
- Widen trace widths ($\ge 0.8\text{ mm}$ for power/ground, $\ge 0.6\text{ mm}$ for signals), enlarge annular rings ($\ge 1.8\text{ mm}$ pad diameter), and increase copper clearances ($\ge 0.4\text{ mm}$) to support DIY etching and CNC milling tolerances.
- Regenerate Gerber and Excellon manufacturing files (`kicad/gerbers/`) for single-sided fabrication, removing active `F.Cu` copper features and via drills.
- Update project documentation (`docs/bom.md`, `docs/wiring-guide.md`, `docs/testing.md`) with single-sided PCB assembly and fabrication notes.

## Capabilities

### New Capabilities
- `pcb-layout`: Physical PCB stackup, single-sided `B.Cu` routing rules, THT pad/clearance geometry for DIY and commercial single-layer fabrication, and Gerber output requirements.

### Modified Capabilities

## Impact

- `kicad/atx-psu-controller.kicad_pcb`: Complete single-sided component placement and `B.Cu`-only routing for all nets and footprints.
- `kicad/gerbers/`: Updated Gerber RS-274X (`B_Cu`, `B_Mask`, `F_SilkS`, `Edge_Cuts`) and Excellon (`PTH.drl`) outputs reflecting single-sided copper geometry.
- `docs/`: Added single-sided fabrication, etching/CNC milling design rules, and non-plated through-hole (NPTH/single-sided pad) soldering instructions.
