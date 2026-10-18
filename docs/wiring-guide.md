# ATX Power Supply Controller - Hardware Integration & Wiring Guide

This guide details how to physically wire the ATX Power Supply Controller to your ATX power supply, host motherboard (such as the AMD BC-250), chassis front panel, and optional ESP microcontroller.

---

## 1. System Interconnect Overview

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

## 2. Connector Wiring Details

### J1: ATX Power Supply Interface (3 Pins)
Connects directly to the 24-pin ATX power supply main harness:

| Header Pin | Wire Signal | ATX 24-Pin Location | Wire Color | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Pin 1** | `+5VSB` | **Pin 9** | Purple | Always +5V DC standby power |
| **Pin 2** | `PS_ON#` | **Pin 16** | Green | Active-low enable line for main PSU rails |
| **Pin 3** | `GND` | **Pin 3, 5, 7, 15, 17, 18, 19, or 24** | Black | System DC ground return |

> [!TIP]
> You can tap these wires using an ATX 24-pin extension cable to avoid splicing the original power supply cables.

---

### J2: Host Motherboard Interface (2 Pins)
Connects to the AMD BC-250 (or equivalent host board):

| Header Pin | Signal Name | Motherboard Connection Point | Wire Color Suggestion | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Pin 1** | `HOST_ON` | **`TPMS1` Header Pin 1** (or pin 2 on some revisions) | Blue | Outputs 3.3V when the host board is powered ON |
| **Pin 2** | `BTN_OUT` | **Power Button Solder Pad** (Bottom side of BC-250) | Orange / Yellow | Active-low pad that triggers ACPI power/sleep events |

#### Finding the BC-250 Connection Points:
1. **`HOST_ON` (TPMS1 Header)**:
   - Locate the 14-pin `TPMS1` header on the motherboard.
   - Use a digital multimeter on DC voltage range. Measure the pins while the board is running: Pin 1 (keyed adjacent to the missing pin or triangle) reads **3.3V when running**, and **0.0V when unpowered/off**.
2. **`BTN_OUT` (Power Button Pad)**:
   - On the bottom side of the BC-250 board under the SMD tactile power/reset button, there are 4 solder pads.
   - One pad is connected to ground. The other pad is the active `PWR_BTN#` line pulled up to 3.3V by the chipset.
   - Solder a thin 28–30 AWG wire from the active pad to J2 Pin 2 (`BTN_OUT`).

---

### J3: Chassis Front Panel Power Button (2 Pins)
Connects to any standard momentary pushbutton (e.g. PC case power button, arcade button, or 16mm vandal-resistant push switch):

| Header Pin | Signal Name | Pushbutton Connection | Notes |
| :--- | :--- | :--- | :--- |
| **Pin 1** | `BTN_IN` | Button Terminal A (Normally Open) | Internal pull-up to 5VSB; pulled low on press |
| **Pin 2** | `GND` | Button Terminal B (Common) | Tied to chassis/system ground |

---

### J4: Front Panel Power LED (2 Pins)
Connects to an illuminated switch LED ring or case power LED:

| Header Pin | Signal Name | LED Connection | Notes |
| :--- | :--- | :--- | :--- |
| **Pin 1** | `LED+` | LED Anode (+) | Provides +5V through onboard $330\ \Omega$ resistor (~15mA) |
| **Pin 2** | `LED-` | LED Cathode (-) | Switched to GND by NPN transistor when `HOST_ON` = 3.3V |

---

### J5: Auxiliary ESP Microcontroller Interface (3 Pins)
Connects to an ESP32 or ESP8266 development board for WiFi/Bluetooth remote wake:

| Header Pin | Signal Name | ESP Connection | Notes |
| :--- | :--- | :--- | :--- |
| **Pin 1** | `+5VSB` | `5V` or `VIN` pin on ESP board | Powers ESP board directly from standby power |
| **Pin 2** | `ESP_WAKE`| Any GPIO configured as Output (e.g. GPIO4) | ESP pulses HIGH (3.3V) for 200–500 ms to wake |
| **Pin 3** | `GND` | `GND` pin on ESP board | Shared ground return |

---

## 3. Host BIOS Configuration

For optimal automated power sequencing, configure the host motherboard BIOS settings:
1. Boot into BIOS setup (press `DEL` or `F2` during boot).
2. Navigate to **Advanced $\rightarrow$ ACPI Configuration / Power Management**.
3. Set **"Restore on AC Power Loss"** (or **"State After G3"**) to **`Power On`** (or **`Always On`**).
4. Save changes and exit (`F10`).

With this setting enabled, the host board will automatically initiate its boot sequence as soon as the ATX controller supplies +12V power!

---

## 4. Single-Sided PCB Assembly & Soldering Guidelines

Because the PCB (`kicad/atx-psu-controller.kicad_pcb`) is a true single-sided (`B.Cu`-only) design compatible with unplated through-holes (home-etched copper-clad FR1/FR4 or desktop CNC-milled boards):

1. **Insert Components from the Top Side (`F.SilkS`)**:
   - Insert all passive components (`R1`, `R2`, `R4`, `R5`, `R6`, `R_bleed`, `R_led`, `D1`–`D3`, `C_hold`, `C_deb`, `C_byp`), transistors (`Q1`–`Q5`), and pin headers (`J1`–`J5`) from the **top non-copper side** so their leads protrude through to the bottom copper (`B.Cu`) pads.
2. **Soldering on Unplated Holes (`B.Cu`)**:
   - All electrical connections are made exclusively on the bottom copper layer (`B.Cu`). No top-side soldering under plastic header bodies or radial capacitors is required.
   - Ensure each solder fillet wets the entire `1.8 mm` / `2.0 mm` bottom copper annular ring and component lead.
3. **Mechanical Strain Relief for Connectors (`J1`–`J5`)**:
   - On unplated single-sided boards, copper pads are held only by adhesive bond to the substrate. When plugging or unplugging Dupont cables on `J1`–`J5`, hold the plastic header shroud with fingers to avoid peeling the bottom copper pads, or apply a small dab of hot glue / epoxy along the top plastic shroud of `J1`–`J5` after bench testing.

