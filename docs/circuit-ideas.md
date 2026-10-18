# Latching a Momentary Pushbutton for ATX Power Supply PS_ON

16 DE JULIO DE 2026 • 19 MIN READ

*INGENIERO COLEGIADO (PE)* *¿Está trabajando sobre una máquina en producción?* Un ingeniero colegiado en Maine puede continuar desde aquí: incluido con hardware IMD, por horas para todo lo demás. [Reservar un ingeniero](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#66)

## Contents

1. [1. Overview: Latching a Momentary Pushbutton for ATX PS_ON](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#77)
2. [2. ATX Power Supply Control Signals](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#78)
3. [3. Why PS_ON# Cannot Be Driven Directly by a Momentary Switch](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#79)
4. [4. 555 Bistable Multivibrator Topology](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#80)
5. [5. NE555 / LM555 / TLC555 Pinout and the Pin-4 Reset Rule](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#81)
6. [6. Component Selection and RC Timing](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#82)
7. [7. Output Driver Stage: BJT vs Logic-Level MOSFET](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#83)
8. [8. Alternative: CD4013 D-Flip-Flop Toggle](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#84)
9. [9. Alternative: Dedicated Pushbutton Controller ICs](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#85)
10. [10. Debouncing, Power-On Default State, and Reset Behavior](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#86)
11. [11. Layout, Wiring, and ATX Safety Considerations](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#87)
12. [12. Verification and Test Procedure](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#88)
13. Frequently Asked Questions
* [Can a 555 timer replace a mechanical latching switch directly?](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#90)
* [How much current does the ATX PS_ON# line need to sink?](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#91)
* [What happens if I leave PS_ON# floating?](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#92)
* [Do I need to debounce the pushbutton with a 555 latch?](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#93)
* [Can I power the latch circuit from +5VSB?](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#94)



---

## 1. Overview: Latching a Momentary Pushbutton for ATX PS_ON

The ATX power supply standard, introduced by Intel in 1995 and revised through the ATX12V series, requires the system to control a single logic line (PS_ON#) to start and stop the main power rails. A momentary pushbutton on the ATX chassis is normally wired through the motherboard, which handles debouncing, the four-second override interval, and the latched PS_ON# generation. When an ATX supply is repurposed for a non-PC load such as the Atari STe, an industrial test fixture, a custom bench supply, or a small embedded chassis, the motherboard's power management is no longer present. The pushbutton then becomes a low-side SPST switch between the green PS_ON# wire and ground.

To start the supply, PS_ON# must be pulled below approximately 0.8 V. To stop it, PS_ON# must be released (allowed to float high) so the supply's internal pull-up to +5VSB can take it above 2.0 V. A bare momentary pushbutton can only hold the line low while the operator is pressing. The moment the button is released, the supply turns off. A latching circuit is required to convert the short, edge-triggered press into a sustained low output that is only released on the next press. This reference covers three practical implementations: a 555-bistable-based latch, a CD4013 toggle flip-flop, and a dedicated pushbutton controller IC.

## 2. ATX Power Supply Control Signals

A 24-pin ATX connector (per the ATX12V Power Supply Design Guide) carries the following control and housekeeping signals used by the latching circuit:

| Pin | Wire color | Signal | Function |
| --- | --- | --- | --- |
| 9 | Purple | +5VSB | +5 V standby, always present when mains is applied; rated 2.0-3.0 A; powers the latch circuit. |
| 16 | Green | PS_ON# | Active-low enable for main +3.3 V, +5 V, +12 V, -12 V rails. Pulled low to start. |
| 8 | Gray | PWR_OK | Open-collector output, goes high 100-500 ms after the main rails are in regulation. |
| 3, 5, 7, 15, 17, 18, 19, 24 | Black | GND | DC ground return; tied to internal DC common, NOT to mains protective earth. |

The +5VSB rail is the dedicated power source for the latching circuit. It must power the 555, the flip-flop, or the controller IC, the pull-up network on PS_ON# inside the ATX supply, and any indicator LEDs. The PS_ON# sink current is specified in most ATX supply datasheets at $\le 5\text{ mA}$, so a low-power BJT or logic-level MOSFET is sufficient to discharge the line. Powering the latch from +5VSB is mandatory: the main rails are not present until the supply is on, so a latch powered from the main rails could not self-start.

## 3. Why PS_ON# Cannot Be Driven Directly by a Momentary Switch

If the operator holds the pushbutton, the supply starts. Hold it for three seconds and the supply still runs. Release and the line floats back to +5VSB; the supply shuts down. The fundamental issue is that PS_ON# is a level-sensitive enable, not an edge-triggered command. Two solutions exist:

1. Replace the momentary pushbutton with a mechanical latching (Push-On/Push-Off) switch. This is the simplest and is often suggested for retro-computing, but it alters the chassis hardware and may not match the original aesthetic.
2. Insert an electronic latch between the pushbutton and PS_ON#. The latch must remember its state between button presses (bistable behavior), toggle state on each press (T-flip-flop behavior, not S-R), survive power-down and power-up cycles with a defined power-on default, and survive the contact bounce of the mechanical switch (debounce).

The remaining sections cover implementation of option 2.

## 4. 555 Bistable Multivibrator Topology

The classic 555 bistable (flip-flop) configuration ties the TRIG (pin 2) and THRES (pin 6) pins together. A negative-going edge on this shared node drives pin 2 below $1/3\text{ V}_{\text{CC}}$, setting the internal flip-flop and driving the output (pin 3) high. A positive-going edge that drives pin 6 above $2/3\text{ V}_{\text{CC}}$ resets the internal flip-flop and drives the output low. With TRIG and THRES tied together, a single RC time constant is shared by both thresholds.

The 555 in this configuration does not toggle on a single edge; it requires two separate pushbuttons, one to momentarily pull the node low (SET) and one to momentarily pull the node high (RESET). To get single-button toggle behavior from a single shared switch, additional steering is required: a diode-capacitor network (a differentiator and an integrator) separates the SET and RESET events from the same press.

A common implementation uses:

* A pull-up resistor (R2 in the table below) to +5VSB on the TRIG/THRES node.
* A pushbutton (SW1) to ground through a current-limiting resistor (R3).
* A capacitor (C1) from the TRIG/THRES node to ground that holds the node state between transitions.
* A coupling capacitor (C2) and steering diodes (D1, D2) that direct the negative-going edge from the switch to the trigger comparator and the positive-going edge to the threshold comparator at the appropriate time.
* The 555's DISCH pin (pin 7) or OUT pin (pin 3) drives a low-side transistor (Q1) whose collector/drain pulls the ATX PS_ON# line to ground.

The output of the 555 then drives the base of an NPN BJT (such as 2N3904) or the gate of a logic-level N-MOSFET (such as 2N7000) whose collector/emitter-drain pulls the ATX PS_ON# line to ground.

## 5. NE555 / LM555 / TLC555 Pinout and the Pin-4 Reset Rule

For a 555 in any 8-pin package (DIP-8, SOIC-8, VSSOP-8):

* NE555 / LM555 DIP-8 / SOIC-8 (top view)
* Pin 1: GND
* Pin 2: TRIG
* Pin 3: OUT
* Pin 4: RESET (Pin 4 MUST be tied to V+ - not optional)
* Pin 5: CTRL
* Pin 6: THRES
* Pin 7: DISCH
* Pin 8: V+

The single most common mistake when bringing up a 555 design is leaving pin 4 floating. On most 555s, pin 4 has an internal pull-down of $\sim 1\text{--}10\text{ }\mu\text{A}$. If left floating, pin 4 stays low, the internal flip-flop is held in reset, and the output never changes. Pin 4 must be tied directly to pin 8 (VCC) for normal operation, or to a logic signal if an external reset is desired. A small RC delay ($10\text{ k}\Omega$ in series with pin 4, then $100\text{ nF}$ to ground) is sometimes used to provide a clean power-on reset, ensuring the latch starts in a defined state on +5VSB power-up.

Recommended part numbers (all from TI):

| Part | Type | Supply range | Output drive | Quiescent current | Notes |
| --- | --- | --- | --- | --- | --- |
| [TI NE555](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#95) | Bipolar | 4.5–16 V | 200 mA sink/source | $\sim 10\text{ mA}$ | Classic 555. Pin-compatible with 7555/ICM7555. |
| [TI LM555](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#96) | Bipolar | 4.5–16 V | 200 mA sink/source | $\sim 10\text{ mA}$ | Improved temperature stability vs NE555. |
| [TI TLC555](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#97) | CMOS | 2–15 V | 100 mA sink/source | $\sim 1\text{ mA}$ | Recommended for ATX standby; rail-to-rail output. |

For ATX standby applications, the TLC555 (or its LinCMOS predecessor LMC555) is preferred because the standby rail is rated 2–3 A and the CMOS 555 draws only a small fraction of the available standby budget.

## 6. Component Selection and RC Timing

For a 555 bistable that toggles on a single pushbutton press, the component values depend on the specific topology. A common implementation is summarized below. Component tolerances of 5% (E24) are usually adequate; the 555 is forgiving.

| Reference | Value | Part / Type | Function |
| --- | --- | --- | --- |
| U1 | NE555 / TLC555 | [TI NE555](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#95) (DIP-8 or SOIC-8) | Bistable multivibrator |
| R1 | $10\text{ k}\Omega$ | 1/4 W, 5% | Pin 4 pull-up to +5VSB (provides 555 reset delay) |
| C0 | $100\text{ nF}$ | Ceramic X7R | Pin 4 RC delay for power-on reset |
| R2 | $100\text{ k}\Omega$ | 1/4 W, 5% | TRIG/THRES node pull-up to +5VSB |
| R3 | $1\text{ k}\Omega$ | 1/4 W, 5% | Pushbutton current limit |
| C1 | $100\text{ nF}$ | Ceramic X7R | TRIG/THRES node to ground (holds state) |
| C2 | $10\text{ nF}$ | Ceramic X7R | AC coupling from pushbutton to TRIG/THRES node |
| D1, D2 | 1N4148 | Small-signal switching diode | Steering: separate SET pulse from RESET pulse |
| C3 | $10\text{ nF}$ | Ceramic X7R | Pin 5 (CTRL) bypass to ground |
| R4 | $4.7\text{ k}\Omega$ | 1/4 W, 5% | Output pull-up (only if Q1 is a BJT) |
| Q1 | 2N7000 | N-MOSFET, 60 V, 200 mA, logic-level | PS_ON# pull-down switch |
| SW1 | SPST momentary | Tactile switch, 50 mA, 12 V | Power button |

Timing analysis: the TRIG/THRES node is held at +5VSB by R2. Pressing the button momentarily pulls the node low through R3, with C1 providing a short hold. The 555's output toggles. On release, C1 charges back to +5VSB through R2 with a time constant $\tau = R_2 \times C_1 = 100\text{ k}\Omega \times 100\text{ nF} = 10\text{ ms}$. This is short enough that successive presses register independently (operator must release for at least 10 ms), but long enough to reject switch bounce (typically less than 5 ms).

The steering diodes D1 and D2 direct the leading and trailing edges of the switch waveform to the appropriate comparator. D1 is wired so that a negative-going edge from the switch flows into the TRIG/THRES node, triggering pin 2. D2 is wired so that a positive-going edge from the switch is steered to ground through C2, bypassing the node. The asymmetry is created by choosing $C_1 \gg C_2$ and by using the diode's reverse-recovery characteristics. In practice, a single 1N4148 and a 10 nF cap with a $100\text{ k}\Omega$ pull-up is sufficient to demonstrate the toggle action, though a buffer stage (Schmitt trigger) is recommended for production designs.

## 7. Output Driver Stage: BJT vs Logic-Level MOSFET

The 555 output swings rail-to-rail (or nearly so on CMOS variants). On the bipolar NE555/LM555, the output high level is typically $V_{\text{CC}} - 1.7\text{ V}$ at 100 mA, which is enough to drive a small signal BJT or the gate of a logic-level N-MOSFET. On the CMOS TLC555, the output swings within $\sim 50\text{ mV}$ of either rail at low load.

For driving ATX PS_ON# (sink current $\le 5\text{ mA}$), an NPN BJT such as 2N3904 or a logic-level N-MOSFET such as 2N7000 works. The 2N7000 is preferred for low-power applications because it requires essentially no gate current, so the 555 output does not have to source anything to keep it on. The 2N3904 requires a base current of roughly $I_B = I_C / h_{\text{FE}} = 5\text{ mA} / 100 = 50\text{ }\mu\text{A}$, so a $4.7\text{ k}\Omega$ base resistor from the 555 output to the base is more than sufficient. The 2N3904 also requires a base-emitter resistor ($\sim 10\text{ k}\Omega$ to ground) to ensure it turns off cleanly when the 555 output goes low.

For the Atari STe application, the load is only the ATX supply's internal pull-up on PS_ON# and any indicator LED. The transistor should be wired as a low-side switch:

```text
+5VSB (PS_ON# pull-up inside supply) --- PS_ON# pin (green) --- collector (2N3904) or drain (2N7000)
                                                                        |
                                                                        +--- emitter (2N3904) or source (2N7000) --- GND

```

When the 555 output is HIGH, Q1 conducts and pulls PS_ON# low (active state). When the 555 output is LOW, Q1 is off and PS_ON# floats back to +5VSB. An optional indicator LED can be wired from +5VSB to the collector/drain of Q1, with a $1\text{ k}\Omega$ current-limit resistor, so the LED lights when the supply is running. This gives the operator a visible confirmation that the latch has toggled.

## 8. Alternative: CD4013 D-Flip-Flop Toggle

The CD4013B is a CMOS dual D-type flip-flop. With one half, it can implement a clean toggle flip-flop: wire the Q-bar output back to the D input. Each rising edge of the clock input toggles the Q output. A debounced pushbutton drives the clock input; Q drives the same low-side BJT/MOSFET that drives PS_ON#.

The CD4013B is well-suited for this because:

* It operates from 3 V to 18 V, so +5VSB is in the middle of the range.
* Static current consumption is in the nanoampere range, so the +5VSB rail is not stressed.
* The Q output swings rail-to-rail.
* The S (set) and R (reset) inputs are tied low for normal operation; they can be used to force a defined power-on state.
* The [TI CD4013B datasheet](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#98) provides the timing specifications and pinout.

Component values for a CD4013 toggle latch:

| Reference | Value | Function |
| --- | --- | --- |
| U1 | CD4013B | Dual D-FF, use one half; Q-bar to D for toggle |
| R1 | $100\text{ k}\Omega$ | Set-input pull-up for power-on default (Q = high at power-up if needed) |
| C0 | $100\text{ nF}$ | Set-input RC delay for power-on default |
| R2 | $100\text{ k}\Omega$ | CLK pull-up to +5VSB |
| R3 | $1\text{ k}\Omega$ | Pushbutton current limit |
| C1 | $100\text{ nF}$ | CLK to ground, provides debounce and AC coupling |
| C2 | $100\text{ nF}$ | +5VSB bypass, as close to U1 as possible |
| Q1 | 2N7000 | PS_ON# pull-down, gate driven by Q output |
| SW1 | SPST momentary | Power button |

The toggle is edge-triggered: the rising edge of the debounced CLK signal toggles Q. The duty cycle of the Q output is exactly 50 percent on average over many toggles, regardless of how long the operator holds the button. The power-on default is set by S (set) and R (reset) inputs: tie S low and R low for Q = low at power-up (PS_ON# floating, supply off), or pull S high through an RC delay to force Q = high at power-up (PS_ON# pulled low, supply starts automatically). For an ATX application, the desired default is Q = low (supply off) so the operator must press the button to start.

## 9. Alternative: Dedicated Pushbutton Controller ICs

For applications where board space, BOM count, or power-on default state are critical, dedicated pushbutton controller ICs eliminate the discrete latch and provide integrated debouncing, ESD protection, and watchdog logic. The two industry-standard families are:

* **Analog Devices LTC2950 / LTC2951 / LTC2954** (originally Linear Technology): Adjustable turn-on/turn-off debounce timers, pushbutton input, KILL input for hard power-down, INT output for software coordination. Operates from 2.7 V to 26 V. The LTC2950-1 and LTC2950-2 differ only in output polarity. The [LTC2950 product page](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#99) provides the datasheet and application notes.
* **Texas Instruments TPS3421 / TPS3430**: Adjustable pushbutton watchdog, similar architecture, integrated ESD protection on the PB pin. The [TI TPS3421 product page](https://industrialmonitordirect.com/es/blogs/knowledgebase/latching-a-momentary-pushbutton-for-atx-power-supply-ps-on?srsltid=AU7gw4U-G2Pm383OS4C-eSNobfYyFBuDmHUjI1UOJvYII0mUq92bPBo6#100) provides the datasheet and EVM reference design.

For an ATX PS_ON# application, the LTC2950-2 is the most common choice because its output is LOW on power-up and toggles HIGH on each button press. The output drives the gate of an N-MOSFET (e.g., 2N7000) which pulls PS_ON# low when the LTC2950-2 output is low. The KILL input can be wired to a secondary pushbutton for hard power-down, or to the ATX supply's PWR_OK signal for software-coordinated shutdown. The pushbutton can be wired with up to several meters of cable because the LTC2950 includes ESD protection up to $\pm 15\text{ kV}$ HBM on the PB pin.

The dedicated controller approach is overkill for a single-supply Atari STe application, but is the right choice for production runs or for designs that may expand to multiple ATX supplies in parallel (a server PSU redundancy controller, for example).

## 10. Debouncing, Power-On Default State, and Reset Behavior

Every mechanical switch bounces for 1–20 ms when actuated. Without debouncing, a single press could toggle the latch multiple times. The 555 bistable, the CD4013, and the LTC2950 all include some form of debouncing, but the mechanism differs:

* **555 bistable**: the RC time constant of the TRIG/THRES node ($R_2 \times C_1$ in the table above) provides a low-pass filter. As long as the time constant is longer than the bounce duration ($\sim 5\text{--}10\text{ ms}$) but shorter than the inter-press gap ($>200\text{ ms}$ typical), the circuit works.
* **CD4013**: edge-triggered, so it only responds to the first rising edge of a debounced waveform. An RC + Schmitt trigger buffer is recommended for clean edges, but in practice a simple RC works.
* **LTC2950**: integrated 8 ms (typical) debounce timer, factory-trimmed.

The power-on default state is a separate consideration:

* **555 bistable**: output is undefined on power-up (depends on which internal comparator wins the race). For an ATX application where the desired default is off (PS_ON# floating), this is acceptable. If the default should be on (PS_ON# held low), tie pin 4 to VCC through an RC delay so the 555 holds RESET for $\sim 10\text{ ms}$ after power-up; the 555 then enters its normal state but the network around the TRIG/THRES node must be designed to favor the SET state on the first clock cycle. In practice, the 555's power-on default is unreliable and a CD4013 is preferred if the power-on state matters.
* **CD4013**: drive the S (set) input HIGH at power-up via an RC delay to force Q high, or R (reset) to force Q low. The desired default for ATX is Q low (PS_ON# floating, supply off).
* **LTC2950**: factory-default output state is documented in the datasheet (LTC2950-2 starts low).

## 11. Layout, Wiring, and ATX Safety Considerations

**SAFETY**: An ATX power supply is a switch-mode supply with the hot line of mains at the input. The supply's internal capacitors store up to 400 VDC for several minutes after unplugging. Always discharge the bulk capacitors (typically 220–470 microfarads at 400 V) through a $10\text{ k}\Omega$ resistor before touching the PCB. The +5VSB rail is always live when the supply is plugged into mains.

1. **Mains voltage**: ATX supplies are switch-mode supplies with the hot line of mains at the input. The supply's internal capacitors store up to 400 VDC for several minutes after unplugging. Always discharge the bulk capacitors (typically 220–470 microfarads at 400 V) through a $10\text{ k}\Omega$ resistor before touching the PCB.
2. **+5VSB is always live**: The +5VSB rail is generated from the standby converter, which is powered as soon as the AC input is present. The purple wire is always at +5 V when the supply is plugged in. Even with PS_ON# floating, the standby converter is running.
3. **ATX ground vs mains ground**: Pins 3, 5, 7, 15, 17, 18, 19, 24 of the ATX connector are the DC ground return. They are tied to the supply's internal DC common, which is the negative terminal of the bulk capacitor and the COM of the switch-mode controller. They are NOT the protective earth (PE) of the mains cord. Treat the ATX ground as live-referenced to mains and use an isolation transformer if any external connection is required.
4. **PS_ON# current**: Most ATX supplies require PS_ON# to source $< 5\text{ mA}$ when released (i.e., the internal pull-up to +5VSB is around 5–10 k$\Omega$). When pulled low, the current sinked by the ATX is typically 1–2 mA. The 555 and CD4013 can source/sink this without issue.
5. **Wire gauge**: The 24-pin ATX connector is rated for $\sim 6\text{ A}$ per pin. The PS_ON# wire is signal-level; any 22–28 AWG wire is fine. The +5VSB wire is also signal-level since the standby rail is rated for 2–3 A and the latch draws $< 50\text{ mA}$.
6. **Bypass**: Place a 100 nF X7R ceramic bypass across the +5VSB and GND pins of the latch IC, as close to the IC as possible. A 10 microfarad aluminum or tantalum bulk cap on the same rail improves transient response when the load (the Atari STe) draws spikes on the +5VSB rail.

The latch circuit should be assembled on a small piece of perfboard or a custom PCB, and housed inside the ATX case or a separate small enclosure. The pushbutton wires should be kept short (preferably $< 30\text{ cm}$) to minimize noise pickup. If a longer cable is required, use shielded cable and tie the shield to ground at the latch end only. If the latch is mounted outside the ATX case, ensure the enclosure is properly grounded to the ATX ground, not to mains protective earth.

## 12. Verification and Test Procedure

1. **555 latch power-on test**: Apply +5VSB to the latch (pin 8 and pin 1). Measure the voltage on pin 3 (output) with a multimeter. It should be either 0 V or close to +5V. If it sits at $\sim V_{\text{CC}}/2$ or oscillates, pin 4 is floating; fix this by tying pin 4 to pin 8.
2. **Pushbutton test**: Connect SW1 and press it. The output (pin 3) should toggle between 0 V and $\sim +5\text{VSB}$ on each press. The transition should be clean with no ringing or chatter. If the output chatters, increase C1 or add a Schmitt trigger buffer between SW1 and the TRIG/THRES node.
3. **Transistor stage test**: With Q1 connected, measure the collector/drain voltage. When the 555 output is high, the collector should be pulled to within $\sim 0.1\text{ V}$ of ground (2N7000) or $\sim 0.2\text{ V}$ (2N3904). When the 555 output is low, the collector should float to +5VSB through the ATX supply's internal pull-up.
4. **PS_ON# current limit test**: Connect a $1\text{ k}\Omega$ resistor between +5VSB and PS_ON# in parallel with Q1. Press the button. The voltage at PS_ON# should drop to $< 0.5\text{ V}$. The current through the resistor should be $\sim 5\text{ mA}$, which is the maximum Q1 must sink. Verify the 555 output and Q1 stay within their safe operating area (SOA) at this current.
5. **ATX supply test**: Connect a dummy load (a $10\text{ }\Omega$, $10\text{ W}$ resistor across +5V and ground, drawing $\sim 500\text{ mA}$) to the ATX supply main rails. Press the button. The supply should start; the fan should spin; the +5V rail should read $+5.00\text{ V} \pm 5\%$. Release the button. The supply should stop; the fan should stop.
6. **End-to-end test with Atari STe**: Connect the ATX supply to the Atari STe power input. Press the button. The STe should boot. Press the button again. The STe should shut down cleanly. Repeat 50 times to verify reliability.
7. **Power cycle test**: With the ATX supply plugged into mains but PS_ON# floating, unplug the supply and plug it back in. The STe should NOT start (PS_ON# should be floating). Press the button. The STe should start. This verifies the power-on default state is off.
8. **ESD test**: With the STe running, touch the pushbutton case (or a grounded metal object near it). The supply should not glitch or restart. If it does, improve the chassis grounding or add ESD protection to the PB pin (TVS diode or RC snubber).

## Frequently Asked Questions

### Can a 555 timer replace a mechanical latching switch directly?

Yes, in bistable configuration. Tie TRIG (pin 2) and THRES (pin 6) together, drive them from a debounced toggle source, and use pin 3 (OUT) or pin 7 (DISCH) to switch the load. Pin 4 (RESET) must be tied to VCC; leaving it floating is the most common cause of "the 555 does not work" symptoms. A steering-diode network is needed if a single pushbutton is to toggle the output.

### How much current does the ATX PS_ON# line need to sink?

Typical ATX supplies require $< 5\text{ mA}$ of sink current on PS_ON# to recognize the active-low request. The internal pull-up to +5VSB is in the $5\text{--}10\text{ k}\Omega$ range. A 2N7000 or 2N3904 has at least 100x the required capacity, so a small signal transistor is sufficient.

### What happens if I leave PS_ON# floating?

The ATX supply stays in standby mode: only +5VSB is active. The main +3.3 V, +5 V, +12 V, and -12 V rails are off. This is the off state of an ATX supply. The latch must actively pull PS_ON# low to start the main rails.

### Do I need to debounce the pushbutton with a 555 latch?

Yes. Mechanical switches bounce for 1–20 ms. A 555 latch with an RC time constant of 10–100 ms on the trigger node will reject bounce. A CD4013 only responds to one edge per press, so it is naturally debounce-tolerant, but a small RC on the clock input is still recommended for clean edges.

### Can I power the latch circuit from +5VSB?

Yes, and you should. The +5VSB rail is always present when the ATX supply is plugged into mains, regardless of PS_ON# state. It is designed to source the standby load (typically 1–3 A). A 555 or CD4013 draws $< 5\text{ mA}$, well within the rail's capacity. Do not power the latch from the main +5 V or +12 V rails because those rails are not present until PS_ON# is pulled low, creating a chicken-and-egg startup problem.
