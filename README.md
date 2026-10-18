# ATX Power Supply Controller (`psu-controller`)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![KiCad Version](https://img.shields.io/badge/KiCad-8.0-blue?logo=kicad)](https://www.kicad.org)
[![PCB Stackup](https://img.shields.io/badge/PCB-Single--Sided%20(1--Layer)-brightgreen)](#pcb-layout--manufacturing-specs)
[![Through-Hole](https://img.shields.io/badge/BOM-100%25%20Through--Hole-orange)](#bill-of-materials-bom)

A discrete, analog hardware controller designed to sequence and manage standard ATX power supplies for non-standard motherboards, repurposed compute platforms (such as the AMD BC-250 mining board), single-board computers, and custom electronics fixtures.

---

## 📖 Table of Contents

- [Motivation & Problem Solved](#-motivation--problem-solved)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Theory of Operation](#-theory-of-operation)
- [Header Pinout & Wiring](#-header-pinout--wiring)
- [Bill of Materials (BOM)](#-bill-of-materials-bom)
- [PCB Layout & Manufacturing Specs](#-pcb-layout--manufacturing-specs)
- [Bench Testing & Verification](#-bench-testing--verification)
- [Host BIOS Configuration](#-host-bios-configuration)
- [Repository Structure](#-repository-structure)
- [Documentation Index](#-documentation-index)
- [License](#-license)

---

## ⚡ Motivation & Problem Solved

Repurposed compute hardware (such as headless crypto-mining boards like the AMD BC-250, embedded industrial boards, and retro computing conversions) often lack standard front-panel ATX power management circuits. 

Without a dedicated controller:
1. **The Green Wire Hack:** Operators typically short the ATX `PS_ON#` wire (green) to Ground (black) with a jumper clip or switch.
2. **24/7 Power Waste & Wear:** The power supply and all chassis fans remain powered continuously at high wattage—even after the operating system cleanly halts and unmounts disks.
3. **Loss of ACPI Control:** Pressing a chassis button cannot trigger an ACPI graceful shutdown, risking filesystem corruption.

The **ATX Power Supply Controller** eliminates jumpers by providing an intelligent, fully autonomous, discrete hardware sequency bridge between any ATX power supply, the host motherboard, and your chassis buttons/LEDs.

---

## ✨ Key Features

- **Cold Boot RC Sequencing:** Momentary chassis button tap charges a timing capacitor ($C_{hold} = 22\ \mu\text{F}$ via $R_{bleed} = 120\text{ k}\Omega$), asserting ATX `PS_ON#` for ~3.5 seconds to allow the host to spin up.
- **Active Telemetry Latching:** Continuously monitors host power state via a 3.3V telemetry pin (e.g., TPM header pin `HOST_ON`). Once booted, host voltage locks `PS_ON#` low indefinitely.
- **Safe ACPI Button Mirroring:** Routes button pulses to the host's power button pad (`BTN_OUT`) through an open-drain N-channel MOSFET (`2N7000`), allowing:
  - **Short tap (200 ms):** Graceful OS ACPI shutdown without dropping ATX main power prematurely.
  - **Long press (4.0 seconds):** Hardware-level emergency forced power cut when OS kernel freezes.
- **Automated Standby Transition:** When the operating system finishes shutting down, `HOST_ON` drops to 0V. The controller de-asserts `PS_ON#` within 300 ms, putting the ATX power supply back into standby ($< 50\ \mu\text{A}$ quiescent draw).
- **Remote Microcontroller Wake (`ESP_WAKE`):** Accepts a 3.3V logic pulse from an ESP32, ESP8266, or Raspberry Pi Pico to power on the system remotely over WiFi/Bluetooth.
- **Hardware Remote Inhibit:** Uses an NPN transistor clamp to hardware-shunt `ESP_WAKE` directly to ground whenever the system is running, preventing accidental resets or wake loops.
- **Chassis Power LED Driver:** Built-in low-side switched driver (~15 mA) indicates true host system state.
- **100% Discrete Through-Hole Parts:** Built entirely with cheap, universally available "jellybean" components (TO-92 transistors, axial 1/4W resistors, DO-41 diodes, radial capacitors).
- **DIY Single-Sided PCB Design:** 1-layer bottom copper routing (`B.Cu` only, zero vias, zero top-side copper traces) designed specifically for home etching (toner transfer / photoresist) or desktop CNC milling.

---

## 🏗 System Architecture

```text
+---------------------+               +-----------------------+
|   Chassis Front     |               |    ATX Power Supply   |
|       Panel         |               |      (24-Pin Main)    |
|                     |               |                       |
| [Power Button]      |               |  Pin 9  (+5VSB, Purp) |
| [Power LED Ring]    |               |  Pin 16 (PS_ON#, Grn) |
|                     |               |  Pin 3  (GND, Blk)    |
+----------+----------+               +-----------+-----------+
           |                                      |
           | 4 wires                              | 3 wires
           v                                      v
     +-----+--------------------------------------+-----+
     |                                                  |
     |         ATX POWER SUPPLY CONTROLLER PCB          |
     |          (55 mm x 45 mm Single-Sided)            |
     |                                                  |
     +-----+--------------------------------------+-----+
           ^                                      ^
           | 2-3 wires                            | 2 wires
           |                                      |
+----------+----------+               +-----------+-----------+
| ESP32 / ESP8266     |               |   Host Motherboard    |
| Microcontroller     |               |     (AMD BC-250)      |
|                     |               |                       |
| - 5V / VIN (Power)  |               | - TPMS1 Pin 1 (3.3V)  |
| - WAKE (GPIO Out)   |               | - PWR_BTN# Solder Pad |
| - GND (Common)      |               | - Common Ground       |
+---------------------+               +-----------------------+
```

---

## 🔬 Theory of Operation

```text
========================================================================================
                          CIRCUIT SCHEMATIC OVERVIEW
========================================================================================

                                  +5VSB (J1 Pin 1, ATX Pin 9)
                                       |
                +----------------------+-------------------------------+
                |                      |                               |
            [R1: 10k]             Source [Q1: BS250]               [R_led: 330R]
                |                    (P-Ch MOSFET)                     |
  BTN_IN -------+------------------+ Gate                              v Anode
  (J3 Pin 1)    |                      | Drain                      [Power LED]
             [C_deb: 100nF]        [D2: 1N5819]                     (J4 Pin 1)
                |                      |                               | Cathode
  GND ----------+                      v                               (J4 Pin 2)
                                 Node [V_GATE]                         |
  HOST_ON (3.3V TPM) -----[D1: 1N5819]-+                               |
  (J2 Pin 1)                           |                               |
                                       |                               |
  ESP_WAKE (3.3V) ----[R5: 1k]---------+-----[D3: 1N5819]--+          |
  (J5 Pin 2)          |                                    |          |
                   Collector                               v          |
                      |                             +-------------+   |
  HOST_ON ---[R6: 10k]+ Base [Q5: 2N3904]           | [C_hold]    |   |
  (Inhibit)           | (Inhibit Switch)            | (22uF)      |   |
                   Emitter                          | [R_bleed]   |   |
                      |                             | (120k)      |   |
                     GND                            +------+------+   |
                                                           |          |
                                                          GND         |
                                                                    Gate
                                                                      v
                                                             +-----------------+
                                                             | Q2: 2N7000      |
                                ATX PS_ON# <-----------------+ (N-Ch MOSFET)   |
                                (J1 Pin 2, ATX Pin 16)       | Source          |
                                                             +--------+--------+
                                                                      |
                                                                     GND

  Q1 Drain (5V Pulse) --------[R2: 1k]-------------> Gate (Q3: 2N7000) -> BTN_OUT (J2.2)
  HOST_ON (3.3V TPM) ---------[R4: 4.7k]-----------> Base (Q4: 2N3904) -> LED- (J4.2)
========================================================================================
```

1. **Standby (Off):** ATX mains is plugged in. `+5VSB` is active. Node `V_GATE` is 0V, $Q_2$ is OFF, `PS_ON#` floats at 5V, power draw is $< 50\ \mu\text{A}$.
2. **Cold Power-On:** Pressing the chassis button pulls $Q_1$ gate low, turning $Q_1$ ON. A 5V pulse charges $C_{hold}$ ($22\ \mu\text{F}$) through Schottky diode $D_2$ in $< 2\text{ ms}$, immediately turning ON $Q_2$ and pulling `PS_ON#` to GND.
3. **Hold-Up Timer:** Once released, $C_{hold}$ discharges through $R_{bleed}$ ($120\text{ k}\Omega$). $Q_2$ remains fully saturated for ~3.5 seconds, giving the host ample time to power on and initialize.
4. **Active Latching:** As the host boots, its 3.3V TPM header asserts `HOST_ON`. Diode $D_1$ conducts, locking `V_GATE` to ~3.05V indefinitely.
5. **ACPI Button Mirroring:** While running, pressing the front panel button turns on $Q_3$, pulling the motherboard power pad (`BTN_OUT`) to ground. The OS registers the ACPI event and initiates a clean shutdown. `PS_ON#` stays grounded by `HOST_ON` during shutdown.
6. **Graceful Power Cut:** When the OS halts, the motherboard turns off and `HOST_ON` drops to 0V. $D_1$ stops conducting, $C_{hold}$ discharges in ~300 ms, $Q_2$ turns off, and the ATX supply turns off main rails.
7. **Emergency 4-Second Override:** If the OS freezes, holding the button for 4.0 seconds holds `BTN_OUT` low, triggering the motherboard hardware supervisor force-off.
8. **Remote Wake & Hardware Inhibit:** A 3.3V pulse on `ESP_WAKE` charges $C_{hold}$ via $D_3$, turning on the PSU. When the host is already running, $Q_5$ clamps the wake input directly to GND, preventing unexpected glitches.

---

## 🔌 Header Pinout & Wiring

| Header | Pin | Signal | Target Connection | Description |
| :--- | :--- | :--- | :--- | :--- |
| **J1: ATX PSU** | 1 | `+5VSB` | ATX 24-Pin Pin 9 (Purple) | Standby +5V DC power |
| | 2 | `PS_ON#` | ATX 24-Pin Pin 16 (Green) | Active-low main rail enable |
| | 3 | `GND` | ATX 24-Pin Pin 3/5/7/15/17 (Black) | Common system ground return |
| **J2: Host Board** | 1 | `HOST_ON` | Motherboard TPM Header Pin 1 (3.3V) | Active power telemetry sense line |
| | 2 | `BTN_OUT` | Motherboard `PWR_BTN#` Solder Pad | Open-drain ACPI button pulse output |
| **J3: Chassis Button**| 1 | `BTN_IN` | Pushbutton Terminal 1 | Momentary switch active-low input |
| | 2 | `GND` | Pushbutton Terminal 2 | Switch ground return |
| **J4: Power LED** | 1 | `LED+` | Power LED Anode (+) | +5V DC current-limited via onboard 330Ω |
| | 2 | `LED-` | Power LED Cathode (-) | Low-side switched ground via $Q_4$ |
| **J5: Remote MCU** | 1 | `+5VSB` | ESP32 `VIN` / `5V` | Standby power for external MCU |
| | 2 | `ESP_WAKE`| ESP32 GPIO (Output) | Remote wake active-high (3.3V) pulse |
| | 3 | `GND` | ESP32 `GND` | Shared ground return |

> [!TIP]
> Use an ATX 24-pin extension cable to tap Pin 9 (`+5VSB`), Pin 16 (`PS_ON#`), and Ground without altering or cutting your power supply harness.

For full pinouts, motherboard solder points, and step-by-step assembly advice, see [docs/wiring-guide.md](docs/wiring-guide.md).

---

## 📦 Bill of Materials (BOM)

All components are standard through-hole parts with wide global availability:

| Ref | Qty | Part / Value | Description | Package | Example MPN |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Q1** | 1 | BS250 | P-Channel Logic MOSFET (-45V, 180mA) | TO-92 | `BS250P` (or 2N3906 PNP) |
| **Q2, Q3** | 2 | 2N7000 | N-Channel Logic MOSFET (60V, 200mA) | TO-92 | `2N7000` |
| **Q4, Q5** | 2 | 2N3904 | NPN General Purpose BJT (40V, 200mA) | TO-92 | `2N3904` |
| **D1, D2, D3** | 3 | 1N5819 | Schottky Barrier Diode (40V, 1A, $V_F \approx 0.25\text{V}$) | DO-41 Axial | `1N5819` |
| **C_hold** | 1 | 22 µF, 25V | Radial Aluminum Electrolytic Capacitor | Radial Ø5mm | `ECA-1EHG220` |
| **C_deb, C_byp**| 2 | 100 nF (0.1 µF) | Multilayer Ceramic Capacitor (50V) | Radial 2.54mm | `C315C104K5R5TA` |
| **R1, R6** | 2 | 10 kΩ, 1/4W | Metal / Carbon Film Resistor (5%) | Axial 7.62mm | `CFR-25JB-52-10K` |
| **R2, R5** | 2 | 1 kΩ, 1/4W | Metal / Carbon Film Resistor (5%) | Axial 7.62mm | `CFR-25JB-52-1K` |
| **R4** | 1 | 4.7 kΩ, 1/4W | Metal / Carbon Film Resistor (5%) | Axial 7.62mm | `CFR-25JB-52-4K7` |
| **R_bleed** | 1 | 120 kΩ, 1/4W | Metal / Carbon Film Resistor (5%) | Axial 7.62mm | `CFR-25JB-52-120K` |
| **R_led** | 1 | 330 Ω, 1/4W | Metal / Carbon Film Resistor (5%) | Axial 7.62mm | `CFR-25JB-52-330R` |
| **J1, J5** | 2 | 3-Pin Header | 2.54mm (0.1") Male Pin Header | 1x03 THT | Standard 2.54mm Header |
| **J2, J3, J4** | 3 | 2-Pin Header | 2.54mm (0.1") Male Pin Header | 1x02 THT | Standard 2.54mm Header |

For detailed component substitutions, pinouts, and purchasing links, see [docs/bom.md](docs/bom.md).

---

## 🛠 PCB Layout & Manufacturing Specs

The PCB is built in KiCad 8 as a **true single-sided layout**:
- **Board Dimensions:** $55.0\text{ mm} \times 45.0\text{ mm}$
- **Layer Stackup:** 1-Layer Copper (`B.Cu` bottom solder side only). Zero copper traces on `F.Cu`, zero vias.
- **Top-Side Crossover Strategy:** Resistor `R2` ($10.16\text{ mm}$ axial span) and diode `D1` ($7.62\text{ mm}$ axial span) act as natural component bridges across bottom copper traces on `B.Cu`, avoiding the need for wire jumpers.
- **DIY Etch / Desktop CNC Milling Tolerances:**
  - Power / Ground Traces: `0.80 mm`
  - Signal Traces: `0.60 mm`
  - Minimum Copper Clearance: $\ge 0.45\text{ mm}$
  - Component Pads: `1.80 mm` outer diameter (`0.8 mm` drill)
  - Connector Pads: `2.00 mm` outer diameter (`1.0 mm` drill)
  - Expanded TO-92 Footprints: $\ge 2.54\text{ mm}$ lead spread for easy hand soldering and generous isolation.

### Manufacturing Files (`kicad/gerbers/`)
- `atx-psu-controller-B_Cu.gbr` — Bottom copper layer (traces & ground plane)
- `atx-psu-controller-B_Mask.gbr` — Solder mask (bottom)
- `atx-psu-controller-F_SilkS.gbr` — Top silkscreen (component placement & polarity)
- `atx-psu-controller-Edge_Cuts.gbr` — Board outline
- `atx-psu-controller-PTH.drl` — Excellon drill file (component holes only, no via drills)

---

## 🧪 Bench Testing & Verification

Before installing into your system, complete bench verification using a DC power supply and multimeter:

1. **Pre-Power Continuity Check:** Measure resistance between `+5VSB` and `GND` ($> 100\text{ k}\Omega$), and between `PS_ON#` and `GND` ($> 1\text{ M}\Omega$).
2. **Standby Current Draw:** Apply 5.0V to `J1.1`. Quiescent current must measure $< 50\ \mu\text{A}$ (typically $\sim 12\ \mu\text{A}$).
3. **Cold Boot Hold-Up Time:** Tap `BTN_IN` to `GND`. `PS_ON#` drops to $< 0.1\text{V}$ instantly and remains low for $2.5\text{s}$ to $3.5\text{s}$ before releasing.
4. **Telemetry Latch:** Apply 3.3V to `HOST_ON` within 2 seconds of button press. `PS_ON#` stays latched low indefinitely; the status LED lights up.
5. **ACPI Button Mirroring:** Tap `BTN_IN`. `BTN_OUT` pulses low ($< 0.1\text{V}$) while `PS_ON#` remains held low without interruption.
6. **Graceful Shutdown:** Remove 3.3V from `HOST_ON`. `PS_ON#` releases within 300 ms, and the LED turns off.
7. **Remote Wake & Inhibit:** 3.3V pulse on `ESP_WAKE` starts the PSU. When `HOST_ON` is 3.3V, `ESP_WAKE` is clamped to ground by $Q_5$.

For full testing protocols and verification logs, see [docs/testing.md](docs/testing.md).

---

## ⚙ Host BIOS Configuration

To take full advantage of automated cold-boot sequencing:
1. Enter your motherboard BIOS setup during boot (`DEL` or `F2`).
2. Go to **Advanced $\rightarrow$ ACPI Configuration / Power Management**.
3. Set **"Restore on AC Power Loss"** (or **"State After G3"**) to **`Power On`** (or **`Always On`**).
4. Save and exit (`F10`).

The host motherboard will now automatically boot the moment the ATX controller supplies +12V power!

---

## 📁 Repository Structure

```text
psu-controller/
├── docs/
│   ├── schematic.md         # Full ASCII schematic, netlist & circuit theory
│   ├── bom.md               # Complete Bill of Materials, footprints & substitutions
│   ├── wiring-guide.md      # Pinout, cable harnesses & motherboard solder points
│   ├── testing.md           # Bench test protocol, checklist & measurements
│   └── circuit-ideas.md     # Reference comparison (555 timers, flip-flops, latches)
├── kicad/
│   ├── atx-psu-controller.kicad_pro   # KiCad 8 project file
│   ├── atx-psu-controller.kicad_sch   # KiCad schematic
│   ├── atx-psu-controller.kicad_pcb   # KiCad single-sided PCB layout
│   └── gerbers/                       # Gerber RS-274X & Excellon drill files
├── openspec/
│   ├── config.yaml          # OpenSpec workspace configuration
│   ├── specs/               # Active system specifications
│   │   ├── atx-power-control/spec.md
│   │   └── pcb-layout/spec.md
│   └── changes/archive/     # Archived design proposals and implementation tasks
└── README.md                # Project documentation and user guide
```

---

## 📚 Documentation Index

- [Circuit Schematic & Netlist](docs/schematic.md)
- [Bill of Materials & Component Substitutions](docs/bom.md)
- [Hardware Wiring & Integration Guide](docs/wiring-guide.md)
- [Bench Testing & Verification Protocol](docs/testing.md)
- [Circuit Topology Research & Comparison](docs/circuit-ideas.md)
- [OpenSpec: ATX Power Control Spec](openspec/specs/atx-power-control/spec.md)
- [OpenSpec: PCB Layout Spec](openspec/specs/pcb-layout/spec.md)

---

## 📄 License

This hardware project is open-source and released under the [MIT License](LICENSE).
