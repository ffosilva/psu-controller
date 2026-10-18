# Tasks

## 1. Circuit & Component Documentation

- [x] 1.1 Generate complete electrical netlist, pinout reference, and theory of operation document in `docs/schematic.md` and verify all signal names align with `design.md`.
- [x] 1.2 Document comprehensive through-hole Bill of Materials (BOM) with manufacturer part numbers, packages, and substitution options in `docs/bom.md` and verify 100% component availability.
- [x] 1.3 Document hardware integration guide covering ATX 24-pin tapping, BC-250 TPM header connection, and chassis front panel wiring in `docs/wiring-guide.md`.

## 2. EDA Schematic & PCB Design (KiCad)

- [x] 2.1 Create KiCad schematic capturing the dual-MOSFET sequencer, ACPI mirror, LED driver, and ESP wake inhibit block.
- [x] 2.2 Assign standard through-hole footprints (TO-92, DO-41, axial 7.62mm, radial 2.5mm, 0.1" pin headers) and run Electrical Rules Check (ERC) with zero errors.
- [x] 2.3 Lay out compact 2-layer PCB with copper ground pours and clear silkscreen labelling for all headers and TO-92 pinouts (Source, Gate, Drain).
- [x] 2.4 Run Design Rules Check (DRC) and export Gerber, drill, and manufacturing files.

## 3. Bench Testing & Verification Protocol

- [x] 3.1 Document bench test procedure and step-by-step verification checklist in `docs/testing.md`.
- [x] 3.2 Validate standby state: verify quiescent current on `5VSB` is < 50µA and `PS_ON#` stays floating high with host off.
- [x] 3.3 Validate cold boot sequence: verify a momentary button press (< 200ms) holds `PS_ON#` low for 2.5s–3.5s and successfully latches once 3.3V is applied to `HOST_ON`.
- [x] 3.4 Validate ACPI shutdown and force-off: verify pressing the button while `HOST_ON` is 3.3V grounds `BTN_OUT` without dropping `PS_ON#`, and verify holding for 4.0s maintains a continuous ground on `BTN_OUT`.
- [x] 3.5 Validate ESP remote wake: verify a 3.3V pulse on `ESP_WAKE` starts the PSU when `HOST_ON` is 0V, and verify the pulse is completely clamped and ignored when `HOST_ON` is 3.3V.

## Workflow follow-up

- Review design and Gerber package.
- Archive the change after verification using `openspec archive`.
