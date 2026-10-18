# ATX Power Supply Controller - Through-Hole Bill of Materials (BOM)

## 1. Primary Components Table

All components specified below are standard through-hole parts with wide global availability across distributors (Mouser, DigiKey, LCSC, Farnell, Tayda).

| Ref | Qty | Value / Specs | Description | Package / Footprint | Example MPN | Alternate Subsitutes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Q1** | 1 | $-45\text{V}, 180\text{mA}$ | P-Channel Logic MOSFET | TO-92 (D-G-S) | `BS250P` (Diodes Inc.) / `BS250` | `2N3906` (PNP BJT with 1k series base resistor), `TP2104N3-G`, `VP2106N3-G` |
| **Q2** | 1 | $60\text{V}, 200\text{mA}$ | N-Channel Logic MOSFET | TO-92 (S-G-D) | `2N7000` (onsemi / Microchip) | `BS170` *(Note: D-G-S pinout)*, `TN0604N3-G`, `VN2222LL` |
| **Q3** | 1 | $60\text{V}, 200\text{mA}$ | N-Channel Logic MOSFET | TO-92 (S-G-D) | `2N7000` (onsemi / Microchip) | `BS170`, `2N3904` (NPN with 1k base resistor) |
| **Q4** | 1 | $40\text{V}, 200\text{mA}$ | NPN General Purpose BJT | TO-92 (E-B-C) | `2N3904` (onsemi) | `BC547` *(Note: C-B-E pinout)*, `2N2222A` |
| **Q5** | 1 | $40\text{V}, 200\text{mA}$ | NPN General Purpose BJT | TO-92 (E-B-C) | `2N3904` (onsemi) | `BC547`, `2N2222A` |
| **D1** | 1 | $40\text{V}, 1\text{A}$ ($V_F \approx 0.25\text{V}$) | Schottky Barrier Diode | DO-41 (Axial) | `1N5819` (Vishay / onsemi) | `1N5817` ($V_F \approx 0.22\text{V}$), `BAT46`, `BAT48` |
| **D2** | 1 | $40\text{V}, 1\text{A}$ ($V_F \approx 0.25\text{V}$) | Schottky Barrier Diode | DO-41 (Axial) | `1N5819` (Vishay / onsemi) | `1N5817`, `1N4148` (DO-35, higher $V_F \approx 0.65\text{V}$ but acceptable for charge path) |
| **D3** | 1 | $40\text{V}, 1\text{A}$ ($V_F \approx 0.25\text{V}$) | Schottky Barrier Diode | DO-41 (Axial) | `1N5819` (Vishay / onsemi) | `1N5817`, `BAT46` |
| **C_hold**| 1 | $22\ \mu\text{F}, 25\text{V}$ | Radial Aluminum Electrolytic | Radial $\varnothing 5\text{mm}$, 2.0/2.5mm lead pitch | Panasonic `ECA-1EHG220` | $22\ \mu\text{F}$–$47\ \mu\text{F}$ (16V or 25V, 105°C), Tantalum radial $22\ \mu\text{F}$ |
| **C_deb** | 1 | $100\text{ nF}$ ($0.1\ \mu\text{F}$), 50V | Ceramic Disc / MLCC | Radial 2.54mm lead pitch | KEMET `C315C104K5R5TA` | Any 100nF 50V ceramic disc / MLCC |
| **C_byp** | 1 | $100\text{ nF}$ ($0.1\ \mu\text{F}$), 50V | Ceramic Disc / MLCC | Radial 2.54mm lead pitch | KEMET `C315C104K5R5TA` | Any 100nF 50V ceramic disc / MLCC |
| **R1** | 1 | $10\text{ k}\Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-10K` | Standard 1/4W or 1/8W axial |
| **R2** | 1 | $1\text{ k}\Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-1K` | Standard 1/4W or 1/8W axial |
| **R4** | 1 | $4.7\text{ k}\Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-4K7` | Standard 1/4W or 1/8W axial |
| **R5** | 1 | $1\text{ k}\Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-1K` | Standard 1/4W or 1/8W axial |
| **R6** | 1 | $10\text{ k}\Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-10K` | Standard 1/4W or 1/8W axial |
| **R_bleed**| 1 | $120\text{ k}\Omega$, 1/4W (0.25W), 5%| Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-120K` | $100\text{ k}\Omega$ to $150\text{ k}\Omega$ |
| **R_led** | 1 | $330\ \Omega$, 1/4W (0.25W), 5% | Carbon / Metal Film Resistor | Axial, 7.62mm pitch | Yageo `CFR-25JB-52-330R` | $270\ \Omega$ to $470\ \Omega$ (adjusts LED brightness) |

---

## 2. Connectors & Hardware

| Ref | Description | Header Type | Pin Pitch | Pins | Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **J1** | ATX Power Supply Interface | Male Pin Header or Screw Terminal | 2.54mm (0.1") | 3 | Pin 1: `+5VSB`, Pin 2: `PS_ON#`, Pin 3: `GND` |
| **J2** | Host Board Interface | Male Pin Header / Dupont | 2.54mm (0.1") | 2 | Pin 1: `HOST_ON` (3.3V TPM), Pin 2: `BTN_OUT` |
| **J3** | Chassis Power Button | Male Pin Header / Dupont | 2.54mm (0.1") | 2 | Pin 1: `BTN_IN`, Pin 2: `GND` |
| **J4** | Front Panel Power LED | Male Pin Header / Dupont | 2.54mm (0.1") | 2 | Pin 1: `LED+` (+5V), Pin 2: `LED-` (Switched GND) |
| **J5** | ESP Microcontroller Interface | Male Pin Header / Dupont | 2.54mm (0.1") | 3 | Pin 1: `+5VSB` (Power), Pin 2: `ESP_WAKE` (Input), Pin 3: `GND` |

---

## 3. Package Pinout Reference

```text
       TO-92 (Flat face facing operator, leads pointing downwards)
                         +---------+
                         |         |
                         +---------+
                           |  |  |
                           1  2  3

   Part             Pin 1          Pin 2          Pin 3
   --------------------------------------------------------
   2N7000 (N-FET)   Source         Gate           Drain
   BS170  (N-FET)   Drain          Gate           Source
   BS250  (P-FET)   Drain          Gate           Source
   2N3904 (NPN BJT) Emitter        Base           Collector
   2N3906 (PNP BJT) Emitter        Base           Collector
   BC547  (NPN BJT) Collector      Base           Emitter
```

> [!WARNING]
> Pay close attention to pinouts when substituting components! For example, `2N7000` has Source on Pin 1 and Drain on Pin 3, whereas `BS170` and `BS250` reverse Drain and Source! Always verify the pinout printed on the PCB silkscreen or check the datasheet before soldering.

---

## 4. Single-Sided PCB Fabrication & Assembly Notes

- **Board Dimensions & Stackup**: `55.0 mm × 45.0 mm` (`1.6 mm` FR1 phenolic or FR4 fiberglass), 1-layer copper (`B.Cu` bottom solder side only, zero top copper traces, zero vias).
- **Pad & Trace Geometry (DIY Etching / CNC Milling Friendly)**:
  - Header pads (`J1`–`J5`): `2.0 mm` outer diameter (`1.0 mm` drill).
  - Discrete component pads (`Q1`–`Q5`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`, `R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`): `1.8 mm` outer diameter (`0.8 mm` drill).
  - Power & ground trunk traces (`+5VSB`, `GND`, `LED+`, `LED-`): `0.8 mm` width.
  - Signal traces (`PS_ON#`, `HOST_ON`, `BTN_IN`, `BTN_OUT`, `ESP_WAKE`, `V_GATE`, `V_PULSE`): `0.6 mm` width.
  - Minimum copper clearance: `>= 0.45 mm` (`0.45 mm` ground zone clearance, `>= 0.54 mm` pad-to-pad clearance).
- **Staggered TO-92 Lead Forming (`Q1`–`Q5`)**:
  - To accommodate `1.8 mm` circular pads with generous isolation channels on a single-sided board, the TO-92 footprints (`TO-92_HandSolder`) use a `>= 2.54 mm` staggered/expanded lead spacing instead of the tight `1.27 mm` inline pitch. Gently spread the outer leads (`Pin 1` and `Pin 3`) or offset the center lead (`Pin 2`) by `1.5 mm–2.5 mm` before inserting into the top side of the board.
- **Zero-Jumper Top-Side Axial Crossovers**:
  - All components insert from the **Top (Component) Side** (`F.Cu` / `F.SilkS`), and all leads are soldered exclusively on the **Bottom (Copper) Side** (`B.Cu`).
  - Axial resistor `R2` (`1 kΩ`, `10.16 mm` span) and Schottky diode `D1` (`1N5819`, `7.62 mm` span) naturally bridge across bottom-side `PS_ON#` and `GND` traces on the top side of the PCB, eliminating the need for any wire jumpers or plated through-holes.

