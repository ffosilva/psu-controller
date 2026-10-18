# Design

## Context

See `proposal.md` for overall motivation. Repurposed SBCs (like AMD BC-250) or industrial single-board computers lack native power management circuitry to sequence standard ATX switch-mode power supplies. Standby power (+5VSB) is always present on ATX Pin 9, while the main rails (+12V, +5V, +3.3V) are commanded by pulling `PS_ON#` (Pin 16) low. The host provides status feedback via a 3.3V TPM header pin (`HOST_ON`) and accepts power/ACPI events via a momentary pull-down pad (`PWR_BTN#` / `BTN_OUT`). The design must be realized using standard through-hole discrete components on a dedicated PCB.

## Goals / Non-Goals

**Goals:**
- Implement a pure analog/discrete hardware controller using jellybean through-hole components (no firmware or microcontroller programming required).
- Provide a ~3.5-second boot pulse on ATX `PS_ON#` triggered by a momentary chassis pushbutton closure to GND.
- Interlock `PS_ON#` with `HOST_ON` (3.3V TPM telemetry) so that `PS_ON#` remains asserted while the host runs, and releases automatically when the host OS shuts down.
- Mirror chassis button presses to `BTN_OUT` using an open-drain stage to safely trigger ACPI graceful shutdown or a 4-second emergency forced shutdown.
- Provide a dedicated active-high 3.3V `ESP_WAKE` input for WiFi/Bluetooth remote wake, with hardware lockout while `HOST_ON` is active.
- Provide a switched low-side power LED output rated for ~15mA.

**Non-Goals:**
- Microcontroller firmware development (avoiding toolchain dependencies, programmer hardware, and code maintenance).
- Software-configurable timing (fixed analog RC timing is sufficient and cannot hang).
- Multi-rail voltage monitoring (handled internally by ATX `PWR_OK` supervisor).

## Decisions

### 1. Pure Discrete Architecture vs. Microcontroller
- **Decision**: Use a discrete dual-MOSFET / BJT analog timing architecture instead of an MCU (like ATtiny13A or CH32V003).
- **Rationale**: Eliminates firmware flashing, programmers, compiler toolchains, and firmware lockup/hang risks. Standby power consumption is in the microamp range (< 50µA).
- **Alternatives Considered**: ATtiny85/13A was evaluated. While it offers programmable delays, it requires flashing equipment and introduces firmware failure modes.

### 2. High-Side Inverter for Chassis Button Input
- **Decision**: Use a P-channel MOSFET (`BS250`) or PNP BJT (`2N3906`) driven by an active-low button line (`BTN_IN` pulled down to GND).
- **Rationale**: Standard PC chassis buttons connect between a signal pin and chassis/GND. The high-side switch inverts the button closure into a positive 5V pulse to rapidly charge the hold capacitor $C_{hold}$.
- **Alternatives Considered**: Active-high button switching +5VSB directly. Rejected because it would conflict with chassis buttons where one terminal is tied to chassis metal or shared ground.

### 3. RC Hold-Up Network & Telemetry Handshake
- **Decision**: $22\ \mu\text{F}$ radial electrolytic capacitor $C_{hold}$ discharged through a $120\text{ k}\Omega$ resistor $R_{bleed}$, clamped by `HOST_ON` via a Schottky diode $D_1$ (`1N5819`).
- **Rationale**:
  $$\tau = R \times C = 120\text{ k}\Omega \times 22\ \mu\text{F} \approx 2.64\text{ seconds}$$
  With logic-level MOSFET $Q_2$ (`2N7000`, $V_{GS(th)} \approx 1.0\text{V} - 2.1\text{V}$), $Q_2$ remains latched ON for ~3.5 seconds. This provides ample headroom for ATX rail stabilization (100–200 ms) and host BIOS initialization before `HOST_ON` takes over.
- **Alternatives Considered**: 555 monostable/bistable multivibrator. As documented in `docs/circuit-ideas.md`, 555 timers suffer from power-on race conditions, output polarity mismatch during standby, and clumsy external reset steering.

### 4. Open-Drain Isolation on `BTN_OUT`
- **Decision**: Drive `BTN_OUT` using an N-channel MOSFET ($Q_3$: `2N7000`) or Schottky diode steering.
- **Rationale**: The host motherboard power button pad operates at 3.3V logic (or 1.8V on some chipsets). Direct connection to the 5V-pulled button circuit would inject overvoltage into the host chipset. An open-drain N-FET ensures complete high-impedance isolation when idle and pulls to true 0.0V when pressed.

### 5. Hardware Inhibit Gate for `ESP_WAKE`
- **Decision**: Use an NPN transistor ($Q_5$: `2N3904`) whose base is driven by `HOST_ON` through a $10\text{ k}\Omega$ resistor to clamp the `ESP_WAKE` line to ground when the host is powered.
- **Rationale**: When the host is running (`HOST_ON` = 3.3V), $Q_5$ is saturated, clamping any incoming ESP pulse to < 0.1V. This prevents accidental power cycling, ignores spurious WiFi commands, and ensures the remote wake pin cannot prevent normal OS shutdown.

---

## Detailed Circuit Schematic

```text
========================================================================================
             THROUGH-HOLE ATX CONTROLLER COMPLETE SCHEMATIC
========================================================================================

                                  +5VSB (ATX Pin 9, Purple)
                                      |
                +---------------------+-------------------------------+
                |                     |                               |
            [R1: 10k]            Source [Q1: BS250]               [R_led: 330R]
                |                   (or 2N3906 PNP)                   |
                |                Gate |                               |
                |                  |  | Drain                         v Anode
  BTN_IN -------+------------------+  |                            [Power LED]
  (To Button)   |                     |                               | Cathode
                |                     |                               |
             [C_deb]              [D2: 1N5819]                        |
             (100nF)                  |                               |
                |                     |                               |
  GND ----------+                     v                               |
                                Node [V_GATE]                         |
                                      |                               |
  HOST_ON (3.3V TPM) -----[D1: 1N5819]+                               |
                                      |                               |
  ESP_WAKE (3.3V) ----[R5: 1k]--------+-----[D3: 1N5819]--+          |
                      |                                    |          |
                   Collector                               |          |
                      |                                    |          |
  HOST_ON ---[R6: 10k]+ Base [Q5: 2N3904]                  |          |
  (Inhibit)           | (Inhibit Switch)                   |          |
                   Emitter                                 |          |
                      |                                    |          |
                     GND                                   |          |
                                                           v          |
                                                    +-------------+   |
                                                    |             |   |
                                                 [C_hold]     [R_bleed]
                                                  (22uF)       (120k) |
                                                    |             |   |
                                                   GND           GND  |
                                                                      |
                                                                    Gate
                                                                      |
                                                                      v
                                                             +-----------------+
                                                             | Q2: 2N7000      |
                                ATX PS_ON# <-----------------+ (N-Ch MOSFET)   |
                                (ATX Pin 16)                 | Source          |
                                                             +--------+--------+
                                                                      |
                                                                     GND

----------------------------------------------------------------------------------------
                             BTN_OUT & LED DRIVERS
----------------------------------------------------------------------------------------

  Q1 Drain (5V Pulse) --------[R2: 1k]-------------> Gate
                                                     +-----------------+
  Host Power Pad (BTN_OUT) <-------------------------+ Q3: 2N7000      |
  (3.3V ACPI Button Pad)                             | (N-Ch MOSFET)   |
                                                     | Source          |
                                                     +--------+--------+
                                                              |
                                                             GND

  HOST_ON (3.3V TPM) ---------[R4: 4.7k]-----------> Base
                                                     +-----------------+
                                                     | Q4: 2N3904      |
                                                     | (NPN BJT)       |<---+ (LED-)
                                                     | Emitter         |
                                                     +--------+--------+
                                                              |
                                                             GND
========================================================================================
```

---

## Bill of Materials (Through-Hole)

| Reference | Component | Recommended Part | Package | Rating / Value | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Q1** | P-Ch MOSFET / PNP | `BS250` *(or `2N3906`)* | TO-92 | $-45\text{V}, 180\text{mA}$ | Button pulse inverter |
| **Q2, Q3** | N-Ch Logic MOSFET | `2N7000` *(or `BS170`)* | TO-92 | $60\text{V}, 200\text{mA}$ | `PS_ON#` pull-down and `BTN_OUT` mirror |
| **Q4, Q5** | NPN BJT | `2N3904` *(or `BC547`)* | TO-92 | $40\text{V}, 200\text{mA}$ | $Q_4$: LED driver; $Q_5$: ESP wake inhibit clamp |
| **D1, D2, D3** | Schottky Diodes | `1N5819` *(or `1N5817`)* | DO-41 | $40\text{V}, 1\text{A}$ ($V_F \approx 0.25\text{V}$) | Steering & `HOST_ON` isolation |
| **C_hold** | Electrolytic Cap | Radial | $\varnothing 5\text{mm}$, 2.5mm | $22\ \mu\text{F}$, 16V / 25V | Sets ~3.5s boot hold-up time |
| **C_deb, C_byp**| Ceramic Caps | Radial Disc / MLCC | 2.54mm pitch | $100\text{ nF}$, 50V | Switch debounce & 5VSB rail bypass |
| **R1, R6** | Resistor | Axial | 1/4W, 7.62mm pitch | $10\text{ k}\Omega$, 5% | $R_1$: Button pull-up; $R_6$: $Q_5$ base drive |
| **R2, R5** | Resistor | Axial | 1/4W, 7.62mm pitch | $1\text{ k}\Omega$, 5% | $R_2$: $Q_3$ gate drive; $R_5$: ESP current limiter |
| **R4** | Resistor | Axial | 1/4W, 7.62mm pitch | $4.7\text{ k}\Omega$, 5% | $Q_4$ base drive resistor |
| **R_bleed** | Resistor | Axial | 1/4W, 7.62mm pitch | $120\text{ k}\Omega$, 5% | $C_{hold}$ bleeder discharge resistor |
| **R_led** | Resistor | Axial | 1/4W, 7.62mm pitch | $330\ \Omega$, 1/4W | Power LED current limit (~15mA) |
| **J1-J4** | Pin Headers | Standard 2.54mm pitch | THT Male Headers | 2-pin / 3-pin | Connectors for ATX, Host, Button, ESP |

---

## Risks / Trade-offs

- **[Risk] Host takes longer than 3.5 seconds to assert `HOST_ON`**
  $\rightarrow$ *Mitigation*: If the host board has unusually long POST firmware delays (> 3s), increase $C_{hold}$ from $22\ \mu\text{F}$ to $33\ \mu\text{F}$ or $47\ \mu\text{F}$ to extend the hold-up window up to 5–7 seconds.
- **[Risk] Pinout mismatch between `2N7000` and `BS170`**
  $\rightarrow$ *Mitigation*: `2N7000` pinout is Source-Gate-Drain (pins 1-2-3), whereas `BS170` is Drain-Gate-Source. PCB silk should clearly designate Pin 1 and pin names (S, G, D).
- **[Risk] Stray noise on unshielded button wiring**
  $\rightarrow$ *Mitigation*: $C_{deb}$ ($100\text{ nF}$) directly across the button input filters high-frequency electromagnetic interference from chassis fans and cables.
