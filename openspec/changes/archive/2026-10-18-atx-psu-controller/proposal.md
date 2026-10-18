# Proposal

## Why

Repurposed compute boards (such as the AMD BC-250 mining board or embedded systems) lack native ATX `PS_ON#` power supply sequencing. Without a dedicated power controller, operators must permanently jump the ATX green wire (`PS_ON#`) to ground, leaving high-wattage power supplies and cooling fans running 24/7 even after the host OS has cleanly shut down. This change introduces an ACPI-coordinated hardware controller circuit using standard discrete through-hole components to automatically sequence ATX power, support chassis pushbutton control (including ACPI shutdown and 4-second emergency force-off), and enable remote wake via WiFi/Bluetooth microcontrollers (e.g. ESP32).

## What Changes

- Add specification and discrete circuit architecture for an ATX Power Supply Controller.
- Implement momentary pushbutton input support (`BTN_IN`) to trigger a ~3.5-second boot hold-up pulse on ATX `PS_ON#`.
- Implement host telemetry feedback monitoring (`HOST_ON` from a 3.3V TPM pin) to latch `PS_ON#` active while the host runs, and de-assert `PS_ON#` when the host shuts down.
- Implement an open-drain ACPI button mirror output (`BTN_OUT`) to pulse the host power button pad for graceful OS shutdown and 4-second emergency override.
- Implement an auxiliary remote wake input (`ESP_WAKE`) that accepts a 3.3V HIGH pulse from an ESP32/ESP8266 to wake the system from standby, with automatic hardware inhibition when the host is already running.
- Implement a switched low-side driver for a chassis power status LED.
- Define a complete through-hole Bill of Materials (BOM) using jellybean components (TO-92 transistors, axial resistors, DO-41 diodes, and radial capacitors).

## Capabilities

### New Capabilities
- `atx-power-control`: Covers the functional logic and electrical requirements for sequencing ATX power, processing manual and remote wake triggers, monitoring host power state, and mirroring ACPI button events.

### Modified Capabilities
None.

## Impact

- Hardware: New dedicated controller PCB board using through-hole components.
- Interfaces: Connects between ATX 24-pin connector (`5VSB`, `PS_ON#`, `GND`), host motherboard (`TPMS1` 3.3V, `PWR_BTN#` pad), chassis front panel (power button, power LED), and optional ESP32 microcontroller.
- Software: No firmware or software changes required on the host system; host BIOS should ideally be configured for "Restore on AC Power Loss: Always On".
