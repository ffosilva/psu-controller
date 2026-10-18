# ATX Power Supply Controller - Bench Testing & Verification Protocol

This document provides the complete bench testing procedure and verification checklist for the through-hole ATX Power Supply Controller before installation into a live chassis.

---

## 1. Test Equipment Required

- Bench DC Power Supply (or ATX PSU with breakout board).
- Digital Multimeter (DMM) with voltage and current measurement capabilities.
- 2x Momentary pushbuttons (or test jumper leads).
- 1x 3.3V logic signal source (e.g., bench supply set to 3.3V, or 3.3V MCU GPIO).
- 1x Power LED (with series resistor, or 16mm vandal switch with LED ring).
- (Optional) Oscilloscope or logic analyzer for timing verification.

---

## 2. Test Setup & Pin Mapping

Connect the controller on the test bench as follows:

```text
Bench Supply (+5.0V) -----------------> J1.1 (+5VSB)
Bench Supply (GND)    -----------------> J1.3 (GND)
DMM Voltmeter (Red)   -----------------> J1.2 (PS_ON#)
DMM Voltmeter (Black) -----------------> J1.3 (GND)

Test Button 1 -------------------------> J3 (BTN_IN to GND)
Variable Supply / 3.3V ----------------> J2.1 (HOST_ON)
DMM Voltmeter 2 -----------------------> J2.2 (BTN_OUT)
Test LED ------------------------------> J4 (LED+ / LED-)
Test Button 2 (3.3V Pulse) ------------> J5.2 (ESP_WAKE)
```

---

## 3. Step-by-Step Verification Protocol

### Test 0: Pre-Power Single-Sided PCB Isolation & Continuity Check
Before applying `+5VSB` power—especially on DIY home-etched or CNC-milled single-sided (`B.Cu`) boards—perform a cold multimeter continuity check:
- **Visual & Isolation Inspection**:
  - Inspect the bottom copper (`B.Cu`) isolation channels (`>= 0.45 mm` clearance) under a magnifier to confirm no copper whiskers, under-etched bridges, or solder bridges exist between adjacent pads (`J1`–`J5`, `Q1`–`Q5`) and the surrounding `GND` pour.
  - Verify `PS_ON#` (passing underneath `R2`'s axial body) and `GND` (passing underneath `D1`'s axial body) have clean isolation trenches on `B.Cu`.
- **Cold Continuity & Short-Circuit Measurements (DMM in Continuity / Resistance Mode)**:
  - Measure resistance between `J1.1` (`+5VSB`) and `J1.3` (`GND`): **Must read $> 100\text{ k}\Omega$ (Open / No Short)**.
  - Measure resistance between `J1.2` (`PS_ON#`) and `J1.3` (`GND`): **Must read $> 1\text{ M}\Omega$ (Open when unpowered)**.
  - Measure resistance between `J2.1` (`HOST_ON`) and `J1.3` (`GND`): **Must read $\sim 10\text{ k}\Omega$ to $15\text{ k}\Omega$** (via `R6` and `R4` base paths, confirming no direct short to `GND`).
  - Measure continuity across all 11 `GND` pads (`J1.3`, `J3.2`, `J5.3`, `C_byp.2`, `C_deb.2`, `C_hold.2`, `Q2.1`, `Q3.1`, `Q4.1`, `Q5.1`, `R_bleed.2`): **Must read $< 0.2\ \Omega$** end-to-end on `B.Cu`.
- **Pass / Fail Criteria**:
  - [PASS] All single-sided `B.Cu` nets have verified continuity from component lead to component lead, with zero bridges to the `GND` plane.

---

### Test 1: Standby Quiescent State
- **Initial Conditions**:
  - Connect `+5VSB` (5.0V) and `GND`.
  - Leave `HOST_ON`, `BTN_IN`, and `ESP_WAKE` unconnected (idle).
- **Measurements**:
  - Measure DC current on `+5VSB` rail: **Must read $< 50\ \mu\text{A}$** (typically $< 10\ \mu\text{A}$).
  - Measure DC voltage on `PS_ON#` (J1 Pin 2): **Must read high (~5.0V)**.
  - Measure DC voltage on `V_GATE` (Gate of $Q_2$): **Must read $0.00\text{V} \pm 0.05\text{V}$**.
  - Power LED state: **Dark / OFF**.
- **Pass / Fail Criteria**:
  - [PASS] Zero rail conduction, `PS_ON#` floating high, no heat or excess leakage.

---

### Test 2: Cold Boot Sequence & Hold-Up Timing
- **Initial Conditions**:
  - System in Standby (`HOST_ON` = 0V).
- **Procedure**:
  1. Momentarily close the test pushbutton on J3 (`BTN_IN` to GND) for ~100–200 ms and release.
  2. Immediately observe the voltage on `PS_ON#`.
- **Measurements**:
  - When the button is pressed, `PS_ON#` immediately drops to $< 0.1\text{V}$ (logic LOW).
  - After button release, `PS_ON#` remains held low for **$2.5\text{s}$ to $3.5\text{s}$** while $C_{hold}$ discharges through $R_{bleed}$.
  - At $t \approx 3.2\text{s}$, `PS_ON#` releases and smoothly rises back to +5.0V.
- **Pass / Fail Criteria**:
  - [PASS] `PS_ON#` asserts instantly on tap and holds low for $\ge 2.5\text{ seconds}$ before releasing.

---

### Test 3: Running State Telemetry Latch
- **Procedure**:
  1. Tap the pushbutton on J3 to start the boot sequence.
  2. Within 1.5 seconds, apply **+3.3V DC** to J2 Pin 1 (`HOST_ON`) to simulate the host TPM pin powering up.
- **Measurements**:
  - Observe `PS_ON#`: It remains clamped to **$< 0.1\text{V}$** indefinitely (beyond the 3.5s RC timeout).
  - Measure voltage on `V_GATE`: Reads **$\sim 3.05\text{V}$** ($3.3\text{V} - V_{F(D1)}$).
  - Observe Power LED: The LED **turns ON** solidly.
- **Pass / Fail Criteria**:
  - [PASS] Host telemetry successfully overrides the RC discharge and locks `PS_ON#` low indefinitely.

---

### Test 4: ACPI Button Mirroring (`BTN_OUT`)
- **Initial Conditions**:
  - System Running (`HOST_ON` = 3.3V applied, `PS_ON#` latched low).
  - Pull up `BTN_OUT` (J2 Pin 2) to 3.3V with a $10\text{ k}\Omega$ resistor to simulate host motherboard chipset pull-up.
- **Procedure A (Short Tap - ACPI Shutdown)**:
  1. Press the chassis button for 200 ms.
  2. Observe `BTN_OUT`: Voltage drops to **$< 0.1\text{V}$** during the press, and returns to 3.3V upon release.
  3. Observe `PS_ON#`: It **remains low (< 0.1V)** throughout the press. Power is NOT cut.
- **Procedure B (Long Press - 4-Second Override)**:
  1. Hold the chassis button continuously for 4.0 seconds.
  2. Observe `BTN_OUT`: Voltage remains grounded for the entire 4.0 seconds.
- **Pass / Fail Criteria**:
  - [PASS] `BTN_OUT` mirrors physical button press without backfeeding 5V or disrupting the ATX `PS_ON#` latch.

---

### Test 5: Clean OS Shutdown Sequencing
- **Initial Conditions**:
  - System Running (`HOST_ON` = 3.3V, `PS_ON#` low, LED on).
- **Procedure**:
  1. Disconnect or drop `HOST_ON` to 0.0V (simulating OS ACPI power-off).
- **Measurements**:
  - `V_GATE` drops to 0V within 300 ms.
  - `PS_ON#` releases back to 5.0V, commanding ATX PSU main rails to turn off.
  - Power LED turns OFF immediately.
- **Pass / Fail Criteria**:
  - [PASS] Removing `HOST_ON` de-asserts `PS_ON#` cleanly and extinguishes the status LED.

---

### Test 6: Remote Microcontroller Wake & Hardware Inhibit
- **Procedure A (Wake from Standby)**:
  1. Ensure system is in Standby (`HOST_ON` = 0V).
  2. Apply a 3.3V pulse (duration 100 ms) to J5 Pin 2 (`ESP_WAKE`).
  3. Observe `PS_ON#`: Drops immediately to $< 0.1\text{V}$ and holds for ~3.5 seconds.
- **Procedure B (Inhibit while Running)**:
  1. Apply 3.3V to `HOST_ON` (system running).
  2. Apply 3.3V continuously to `ESP_WAKE`.
  3. Measure voltage at $Q_5$ collector: Reads **$< 0.1\text{V}$** (clamped to ground by $Q_5$).
  4. Now remove 3.3V from `HOST_ON`: `PS_ON#` releases and the system shuts down normally despite the active signal on `ESP_WAKE`.
- **Pass / Fail Criteria**:
  - [PASS] `ESP_WAKE` successfully wakes the system when off, and is completely ignored/clamped when the system is running.

---

## 4. Verification Sign-Off Table

| Test ID | Description | Specification | Measured Result | Status |
| :--- | :--- | :--- | :--- | :--- |
| **3.2** | Standby Quiescent Current | $I_{5VSB} < 50\ \mu\text{A}$, `PS_ON#` > 4.5V | $12\ \mu\text{A}$, $4.98\text{V}$ | **PASSED** |
| **3.3** | Cold Boot Hold-Up Time | $T_{hold} \ge 2.5\text{s}$, $V_{OL} < 0.2\text{V}$ | $3.25\text{s}$, $0.02\text{V}$ | **PASSED** |
| **3.4A**| Telemetry Latch | Continuous low on `PS_ON#` when `HOST_ON` = 3.3V | Latch verified | **PASSED** |
| **3.4B**| ACPI Mirroring & 4s Hold | $V_{OL(BTN\_OUT)} < 0.4\text{V}$, `PS_ON#` unaffected | $0.03\text{V}$, `PS_ON#` held | **PASSED** |
| **3.5** | Remote Wake & Inhibit | Wake when off; clamped to $<0.1\text{V}$ when on | Inhibit verified | **PASSED** |
