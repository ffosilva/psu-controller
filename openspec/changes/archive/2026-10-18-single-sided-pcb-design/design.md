# Design

## Context

The initial `v1.0` PCB (`kicad/atx-psu-controller.kicad_pcb`) placed traces on `F.Cu` and a ground plane on `B.Cu`, and omitted several passive footprints (`D1`–`D3`, `R1`–`R6`, `R_bleed`, `R_led`, `C_deb`, `C_byp`) from the physical board layout. See `proposal.md` for the motivation to support single-sided DIY etching, CNC isolation milling, and single-layer FR1/FR4 manufacturing.

## Goals / Non-Goals

**Goals:**
- Achieve a planar, zero-crossing copper topology on `B.Cu` for all 14 nets (`GND`, `+5VSB`, `PS_ON#`, `HOST_ON`, `BTN_IN`, `BTN_OUT`, `ESP_WAKE`, `V_GATE`, `V_PULSE`, `LED+`, `LED-`, `N_Q3_G`, `N_Q4_B`, `N_Q5_B`, `N_ESP_CLAMP`) without requiring any top-side copper traces, plated through-holes, or wire jumpers.
- Place all 21 BOM components (`J1`–`J5`, `Q1`–`Q5`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`, `R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`) on a compact single-sided board (`55mm x 45mm`).
- Enforce generous copper geometry rules suitable for home toner-transfer/photoresist etching and desktop 3018 CNC engraving bits.

**Non-Goals:**
- Changing the electrical circuit topology, component values, or external connector pinouts defined in `openspec/specs/atx-power-control/spec.md`.
- Adding SMD components or multi-layer stackups.

## Decisions

### 1. Standard Single-Sided THT Orientation (`F.Cu` Component Side, `B.Cu` Solder/Copper Side)
- **Decision**: Mount all through-hole components on the top surface (`F.Cu` footprint insertion side, with component outlines on `F.SilkS`) and route all copper traces and the `GND` pour exclusively on `B.Cu`.
- **Rationale**: In standard single-sided through-hole construction, component leads pass through non-plated holes and are soldered directly to copper pads on the bottom (`B.Cu`) without requiring top-side soldering under component bodies (such as headers `J1`–`J5` or electrolytic capacitor `C_hold`).
- **Alternatives Considered**:
  - *Routing on `F.Cu`*: Would require soldering component leads on the top side underneath plastic header housings and radial capacitors, which is impractical on non-plated single-sided boards.

### 2. Zero-Jumper Planar Routing via Axial Component Span Crossovers
- **Decision**: Arrange headers along the board perimeter (`J1` ATX top-left, `J3` Button bottom-left, `J5` ESP bottom-center, `J2` Host top-right, `J4` LED bottom-right) and orient axial resistors (`R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`, `7.62mm` pitch) and axial DO-41 diodes (`D1`, `D2`, `D3`, `7.62mm` pitch) so their bodies span across `B.Cu` signal traces where topological crossings occur.
- **Rationale**: A `7.62mm` (300 mil) axial lead pitch provides a `5.0mm` clearance window between its two pads on `B.Cu`, easily allowing one or two `0.6mm`–`0.8mm` traces with `0.4mm` clearance to pass underneath the component body on `B.Cu` without adding dedicated 0-ohm jumper links.
- **Alternatives Considered**:
  - *Dedicated 0-ohm wire jumpers*: Adds extra BOM entries and manual wire links during assembly; unnecessary because existing axial resistors (`R1`–`R6`, `R_led`) and diodes (`D1`–`D3`) already bridge the required crossings naturally.

### 3. DIY-Friendly Pad, Trace, and Clearance Rules
- **Decision**:
  - Through-hole pads: `1.8mm`–`2.0mm` diameter (`0.8mm`–`1.0mm` drill), providing a `0.45mm`–`0.55mm` annular ring.
  - Power traces (`+5VSB`, `GND`, `LED+`, `LED-`): `0.8mm`–`1.0mm` width.
  - Signal traces (`PS_ON#`, `HOST_ON`, `BTN_IN`, `BTN_OUT`, `ESP_WAKE`, `V_GATE`, `V_PULSE`): `0.6mm` width.
  - Copper-to-copper and `B.Cu` ground zone clearance: `0.4mm` minimum (`0.5mm` on zone fill).
- **Rationale**: Small pads (`< 1.5mm`) delaminate or lift during hand drilling and soldering on unplated FR1/FR4 copper-clad boards, and tight clearances (`< 0.3mm`) risk solder bridges or incomplete etching.

## Risks / Trade-offs

- **[Risk] Non-plated hole pad lifting during rework** → Enlarge all THT pads to $\ge 1.8\text{ mm}$ diameter (and oval `2.2mm x 1.6mm` for TO-92 `1.27mm`-pitch pads, or staggered triangular TO-92 footprint with `2.54mm` pitch) so copper adhesion is maximized.
- **[Risk] TO-92 tight `1.27mm` inline pin pitch on DIY boards** → Use a staggered/formed TO-92 footprint (`2.54mm` lead pitch or offset center pin) so each transistor pad can have a full `1.8mm` circular pad with `>= 0.4mm` clearance on single-sided boards.
