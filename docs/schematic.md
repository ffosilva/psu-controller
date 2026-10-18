# ATX Power Supply Controller - Electrical Schematic & Netlist Reference

## 1. Overview

The ATX Power Supply Controller is a discrete, analog hardware controller designed to sequence an ATX power supply for non-standard host motherboards (such as the AMD BC-250 mining board). It provides:
- Cold power-on sequencing with an RC hold-up timer (~3.5 seconds).
- Active telemetry locking via the host's 3.3V TPM header (`HOST_ON`).
- Safe open-drain ACPI button mirroring (`BTN_OUT`) for graceful OS shutdown and 4-second emergency override.
- Hardware-inhibited remote wake input (`ESP_WAKE`) for WiFi/Bluetooth microcontrollers.
- Switched low-side Power LED driver (~15mA).

All components are standard through-hole parts (TO-92, DO-41, axial resistors, radial capacitors).

---

## 2. Complete ASCII Schematic

```text
========================================================================================
                          CIRCUIT SCHEMATIC (THROUGH-HOLE)
========================================================================================

                                  +5VSB (J1 Pin 1, ATX Pin 9, Purple)
                                      |
                +---------------------+-------------------------------+
                |                     |                               |
            [R1: 10k]            Source [Q1: BS250]               [R_led: 330R]
                |                   (or 2N3906 PNP)                   |
                |                Gate |                               |
                |                  |  | Drain                         v Anode
  BTN_IN -------+------------------+  |                            [Power LED]
  (J3 Pin 1)    |                     |                            (J4 Pin 1)
                |                     |                               | Cathode
             [C_deb]              [D2: 1N5819]                     (J4 Pin 2)
             (100nF)                  |                               |
                |                     |                               |
  GND ----------+                     v                               |
  (J3 Pin 2)                    Node [V_GATE]                         |
                                      |                               |
  HOST_ON (3.3V TPM) -----[D1: 1N5819]+                               |
  (J2 Pin 1)                          |                               |
                                      |                               |
  ESP_WAKE (3.3V) ----[R5: 1k]--------+-----[D3: 1N5819]--+          |
  (J5 Pin 2)          |                                    |          |
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
                                (J1 Pin 2, ATX Pin 16)       | Source          |
                                                             +--------+--------+
                                                                      |
                                                                     GND (J1 Pin 3)

----------------------------------------------------------------------------------------
                             BTN_OUT & LED DRIVER STAGE
----------------------------------------------------------------------------------------

  Q1 Drain (5V Pulse) --------[R2: 1k]-------------> Gate
                                                     +-----------------+
  Host Power Pad (BTN_OUT) <-------------------------+ Q3: 2N7000      |
  (J2 Pin 2)                                         | (N-Ch MOSFET)   |
                                                     | Source          |
                                                     +--------+--------+
                                                              |
                                                             GND

  HOST_ON (3.3V TPM) ---------[R4: 4.7k]-----------> Base
  (J2 Pin 1)                                         +-----------------+
                                                     | Q4: 2N3904      |
                                                     | (NPN BJT)       |<---+ (LED-)
                                                     | Emitter         |
                                                     +--------+--------+
                                                              |
                                                             GND
========================================================================================
```

---

## 3. Signal Netlist Reference

| Net Name | Origin / Node | Destination / Node | Signal Characteristic | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`+5VSB`** | J1.1 (ATX Pin 9) | Q1.S, R1.1, R_led.1, C_byp.1, J5.1 | +5.0V DC standby (always live) | Auxiliary rail powering controller & charging $C_{hold}$ |
| **`GND`** | J1.3 (ATX Pin 3/5/etc) | J2/J3/J4/J5 grounds, Q2.S, Q3.S, Q4.E, Q5.E, C_hold.2, R_bleed.2 | 0.0V DC Reference | System common ground return |
| **`PS_ON#`** | J1.2 (ATX Pin 16) | Q2.D | 0.0V active / 5.0V idle | Main PSU enable line (sinks to GND to start PSU) |
| **`HOST_ON`** | J2.1 (Host `TPMS1` Pin 1)| D1.Anode, R4.1, R6.1 | 3.3V DC logic (0V off / 3.3V on) | Motherboard power state telemetry sense line |
| **`BTN_IN`** | J3.1 (Front Panel Button)| R1.2, Q1.G, C_deb.1 | 5.0V idle / 0.0V pressed | Chassis momentary pushbutton active-low input |
| **`BTN_OUT`**| J2.2 (Host Power Pad) | Q3.D | Open-drain (< 0.2V active / Hi-Z)| Mirrors button press to host chipset without backfeed |
| **`ESP_WAKE`**| J5.2 (External MCU GPIO)| R5.1 | 3.3V active-high pulse | Remote wake signal from WiFi/Bluetooth microcontroller |
| **`V_GATE`** | Q2.G, C_hold.1, R_bleed.1| D1.Cathode, D2.Cathode, D3.Cathode | 0V idle / 3.0V–4.7V active | Core gate node controlling $Q_2$ conduction state |
| **`V_PULSE`**| Q1.D, D2.Anode | R2.1 | 0V idle / +5.0V during press | Inverted high-side pulse from button actuation |
| **`LED+`** | J4.1 | R_led.2 | +5.0V DC through $330\ \Omega$ | Current-limited anode supply for power LED (~15mA) |
| **`LED-`** | J4.2 | Q4.C | Switched GND (< 0.1V on / Hi-Z) | Low-side switched return for power LED |

---

## 4. Theory of Operation

### 4.1 Standby State (System Off)
Mains AC is connected to the ATX power supply. The standby converter outputs +5VSB on ATX Pin 9, powering the controller.
- `HOST_ON` is 0V because the host motherboard is unpowered.
- $C_{hold}$ is fully discharged through $R_{bleed}$ ($120\text{ k}\Omega$), holding node `V_GATE` at 0V.
- Transistor $Q_2$ is OFF.
- `PS_ON#` floats high to +5V through the ATX supply's internal pull-up resistor.
- Quiescent current draw from +5VSB is $< 50\ \mu\text{A}$ (only leakage through $Q_1$).

### 4.2 Cold Boot Sequence
When the operator presses the chassis pushbutton:
1. `BTN_IN` is pulled to GND, dropping $Q_1$'s gate to 0V.
2. $Q_1$ (P-channel MOSFET) turns on hard ($V_{GS} = -5\text{V}$), bringing node `V_PULSE` to +5VSB.
3. Node `V_PULSE` charges $C_{hold}$ ($22\ \mu\text{F}$) to ~4.7V through Schottky diode $D_2$ in less than 2 milliseconds.
4. $Q_2$ turns on immediately, pulling ATX `PS_ON#` to GND ($V_{DS} < 10\text{ mV}$).
5. The ATX power supply starts and ramps its +12V, +5V, and +3.3V rails within 100–200 ms.
6. When the operator releases the button, $Q_1$ turns off and $D_2$ is reverse-biased.
7. $C_{hold}$ discharges exclusively through $R_{bleed}$ ($120\text{ k}\Omega$):
   $$\tau = R_{bleed} \times C_{hold} = 120\text{ k}\Omega \times 22\ \mu\text{F} \approx 2.64\text{ seconds}$$
   $Q_2$ remains fully conductive for approximately 3.5 seconds.
8. The host motherboard powers up from the +12V rail (via BIOS "Restore on AC Power Loss: Always On").
9. Within 1.0 to 1.8 seconds, the host motherboard's TPM header pin asserts `HOST_ON` = 3.3V.
10. Current flows through Schottky diode $D_1$ ($1\text{N}5819$, $V_F \approx 0.25\text{V}$), clamping `V_GATE` to ~3.05V.
11. $Q_2$ remains locked ON indefinitely, powered by the host's running state.

### 4.3 Running State & ACPI Button Mirroring
While the host is active:
- $Q_4$ receives base current from `HOST_ON` through $R_4$ ($4.7\text{ k}\Omega$), turning ON and connecting `LED-` to GND, illuminating the chassis power LED.
- If the operator taps the chassis button (e.g. for 200 ms):
  - $Q_1$ sends a 5V pulse to the gate of $Q_3$ through $R_2$ ($1\text{ k}\Omega$).
  - $Q_3$ turns on and pulls `BTN_OUT` to GND ($V_{DS} < 50\text{ mV}$) for the duration of the physical press.
  - The host motherboard chipset detects the falling edge on its power button pad and triggers an ACPI graceful shutdown.
  - Crucially, `PS_ON#` remains held low by `HOST_ON`, ensuring the ATX power supply maintains full power while the OS flushes buffers and cleanly unmounts storage drives.

### 4.4 OS Power-Off Transition
When the operating system finishes shutting down:
1. The motherboard chipset cuts power to secondary and core regulators.
2. `HOST_ON` drops from 3.3V to 0.0V.
3. Diode $D_1$ ceases conduction. Since $C_{hold}$ has already discharged to the 3.0V clamp level, the remaining charge drains through $R_{bleed}$ to 0V in ~300 ms.
4. $Q_2$ turns OFF.
5. `PS_ON#` floats back to +5V.
6. The ATX power supply shuts down its main rails (+12V, +5V, +3.3V).
7. $Q_4$ turns off and the power LED extinguishes. The system returns cleanly to Standby.

### 4.5 Emergency 4-Second Forced Shutdown
If the operating system hangs or kernel-panics:
1. The operator holds the chassis button continuously for 4 seconds.
2. $Q_1$ and $Q_3$ remain on, holding `BTN_OUT` grounded continuously for 4 seconds.
3. The motherboard's hardware supervisor / Super I/O detects the 4-second hold condition and forces an emergency power cut.
4. `HOST_ON` immediately falls to 0V.
5. Upon button release, $Q_2$ turns off and ATX power shuts off.

### 4.6 Remote Microcontroller Wake & Inhibit Logic
An external microcontroller (such as an ESP32 or ESP8266) powered from `+5VSB` connects to `ESP_WAKE`:
- **When Host is OFF (`HOST_ON` = 0V)**:
  $Q_5$'s base is at 0V, so $Q_5$ is OFF. When the ESP asserts a 3.3V pulse on `ESP_WAKE`, current flows through $R_5$ ($1\text{ k}\Omega$) and Schottky diode $D_3$ into `V_GATE`, charging $C_{hold}$ and starting the ATX power supply exactly like a physical button press.
- **When Host is ON (`HOST_ON` = 3.3V)**:
  $Q_5$ receives base current via $R_6$ ($10\text{ k}\Omega$) and saturates ($V_{CE} < 0.1\text{V}$), clamping the node between $R_5$ and $D_3$ directly to GND. Any subsequent signal on `ESP_WAKE` is safely shunted to ground; diode $D_3$ remains reverse-biased, completely preventing remote wake pulses from disturbing the running system or preventing OS shutdown.
