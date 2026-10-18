# Tasks

## 1. Single-Sided KiCad PCB Layout (`B.Cu` Only)

- [x] 1.1 Update `kicad/atx-psu-controller.kicad_pcb` to place all 21 through-hole BOM footprints (`J1`–`J5`, `Q1`–`Q5`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`, `R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`) on the top component side with pad outer dimensions >= 1.8mm, and verify every component reference from `docs/bom.md` exists in the PCB file
- [x] 1.2 Route all electrical nets (`GND`, `+5VSB`, `PS_ON#`, `HOST_ON`, `BTN_IN`, `BTN_OUT`, `ESP_WAKE`, `V_GATE`, `V_PULSE`, `LED+`, `LED-`, and internal transistor base/gate/clamp nodes) exclusively on `"B.Cu"` using power traces >= 0.8mm, signal traces >= 0.6mm, and clearances >= 0.4mm, and verify `atx-psu-controller.kicad_pcb` has zero `(segment ... (layer "F.Cu"))` and zero `(via ...)` elements
- [x] 1.3 Configure the `"B.Cu"` `GND` copper pour zone in `kicad/atx-psu-controller.kicad_pcb` with `>= 0.4mm` clearance and verify all ground pads connect cleanly without isolating signal traces

## 2. Single-Sided Gerber & Drill Manufacturing Package

- [x] 2.1 Regenerate the Gerber RS-274X and Excellon drill files in `kicad/gerbers/` (`B_Cu.gbr`, `B_Mask.gbr`, `F_SilkS.gbr`, `Edge_Cuts.gbr`, `PTH.drl`) for the single-sided layout and verify `B_Cu.gbr` contains all routed traces and pads while `PTH.drl` contains all 51 component lead holes and zero via holes
- [x] 2.2 Update or remove `F_Cu.gbr` and `F_Mask.gbr` in `kicad/gerbers/` so that single-layer fabrication/CNC milling packages contain zero top-side copper traces, and verify no `F.Cu` trace apertures exist in `kicad/gerbers/`

## 3. Documentation Updates for Single-Sided Fabrication & Assembly

- [x] 3.1 Update `docs/bom.md` and `docs/wiring-guide.md` with single-sided PCB physical layout dimensions, staggered TO-92 lead-forming notes, and bottom-side (`B.Cu`) soldering guidelines for unplated holes, and verify all component references match `kicad/atx-psu-controller.kicad_pcb`
- [x] 3.2 Update `docs/testing.md` with pre-assembly visual and continuity checks specific to single-sided home-etched or CNC-milled boards (checking `0.4mm` isolation channels and bottom-only solder joints), and verify all net names match `docs/schematic.md`
