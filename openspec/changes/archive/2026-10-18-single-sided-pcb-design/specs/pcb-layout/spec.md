# Spec Delta

## Purpose

Defines the physical PCB layout, single-sided copper routing constraints, through-hole pad and clearance tolerances for DIY and single-layer manufacturing, and Gerber fabrication output requirements.

## ADDED Requirements

### Requirement: Single-Sided Bottom Copper Routing Only
The PCB layout SHALL route 100% of electrical copper traces and copper zones exclusively on the bottom copper layer (`B.Cu`), with zero copper traces on the front copper layer (`F.Cu`) and zero interlayer vias.

#### Scenario: Inspecting copper layer assignments in PCB layout
- **WHEN** the KiCad PCB file (`atx-psu-controller.kicad_pcb`) is inspected for copper track segments, vias, and filled zones
- **THEN** all `(segment ...)` and `(zone ...)` elements reside exclusively on `"B.Cu"`, no `(via ...)` elements exist, and `"F.Cu"` contains zero electrical routing

#### Scenario: Component lead bridging without top-side copper
- **WHEN** two signal nets must cross on the board
- **THEN** the layout routes one net on `B.Cu` underneath the top-mounted body of an axial resistor or diode (`F.Cu` placement side) rather than using a via or `F.Cu` trace

### Requirement: Complete Through-Hole BOM Footprint Placement
The PCB layout SHALL include placed through-hole footprints on the top assembly side (`F.Cu`) for all 21 components in the Bill of Materials (`J1`–`J5`, `Q1`–`Q5`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`, `R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, and `R_led`), with every pad assigned to its corresponding net from the schematic.

#### Scenario: Verifying footprint and net completeness against schematic
- **WHEN** the footprints and pad net assignments in `atx-psu-controller.kicad_pcb` are compared against `docs/schematic.md` and `docs/bom.md`
- **THEN** all 5 headers, 5 TO-92 transistors, 3 DO-41 diodes, 3 radial capacitors, and 7 axial resistors are present with zero unrouted nets

### Requirement: DIY Etching and CNC Milling Geometry Tolerances
The PCB layout SHALL use minimum trace widths of 0.6mm for signal nets, 0.8mm for power nets (`+5VSB`, `GND`), minimum copper-to-copper clearance of 0.4mm, and minimum through-hole pad diameter of 1.8mm to ensure reliable home etching, CNC isolation milling, and single-sided pad soldering without plated barrels.

#### Scenario: Checking trace width and clearance rules
- **WHEN** trace segments, copper zone clearances, and through-hole pad sizes are measured in `atx-psu-controller.kicad_pcb`
- **THEN** all power traces are >= 0.8mm wide, all signal traces are >= 0.6mm wide, zone and trace clearances are >= 0.4mm, and all through-hole pads have a minimum outer dimension >= 1.8mm

### Requirement: Single-Sided Gerber and Excellon Manufacturing Outputs
The Gerber and drill export package (`kicad/gerbers/`) SHALL provide a complete single-sided fabrication set where active copper routing exists only in `B_Cu.gbr` and the drill file contains only component lead holes with zero via drills.

#### Scenario: Inspecting Gerber package for single-layer fabrication
- **WHEN** the generated files in `kicad/gerbers/` are inspected for CNC milling, home etching, or single-layer board house submission
- **THEN** `atx-psu-controller-B_Cu.gbr` contains all routed traces, pads, and the ground plane, `atx-psu-controller-PTH.drl` contains all component lead holes with zero via holes, and no top-side copper trace routing exists
