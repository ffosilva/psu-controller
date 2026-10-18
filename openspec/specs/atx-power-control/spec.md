# atx-power-control Specification

## Purpose

Provides automated sequencing and control of ATX power supply rails for headless or non-PC host boards, managing power-on hold timing, ACPI button mirroring, host state tracking, and remote wake signals.

## Requirements

### Requirement: ATX Power Supply Sequencing on Button Press
The controller SHALL pull ATX PS_ON# below 0.8V for a minimum hold-up duration of 2.5 seconds upon momentary closure of the power button input (BTN_IN to GND) while the host system is off.

#### Scenario: Cold power-on from chassis button
- **WHEN** the host is unpowered (HOST_ON is 0V) and the momentary pushbutton is pressed for >= 100ms
- **THEN** PS_ON# is driven to GND immediately and remains held low for at least 2.5 seconds after button release

#### Scenario: Host fails to boot before timeout
- **WHEN** the pushbutton is pressed but HOST_ON does not rise above 2.0V within 3.5 seconds
- **THEN** PS_ON# is released to float high and the ATX power supply returns to standby mode

### Requirement: Host Power State Telemetry and Latch
The controller SHALL continuously latch PS_ON# to GND whenever HOST_ON is energized at 3.3V, and SHALL immediately release PS_ON# when HOST_ON transitions to 0V.

#### Scenario: Sustained system operation
- **WHEN** the host finishes booting and asserts 3.3V on HOST_ON
- **THEN** PS_ON# remains asserted low independently of the initial boot capacitor state

#### Scenario: Clean operating system shutdown
- **WHEN** the host operating system shuts down and removes power from HOST_ON (0V)
- **THEN** PS_ON# is released within 500ms, causing the ATX power supply main rails to turn off

### Requirement: ACPI Power Button Event Mirroring
The controller SHALL provide an open-drain output (BTN_OUT) that pulls the host board's power button pad to GND when the chassis button is pressed, without backfeeding voltage when released.

#### Scenario: ACPI graceful shutdown request
- **WHEN** the host is running (HOST_ON is 3.3V) and the user presses the chassis button for 200ms
- **THEN** BTN_OUT is pulled to GND (< 0.4V) for the duration of the button press while PS_ON# remains held low

#### Scenario: Emergency forced shutdown
- **WHEN** the host is running and the user holds the chassis button continuously for 4.0 seconds
- **THEN** BTN_OUT remains grounded continuously for 4.0 seconds, allowing the host hardware supervisor to force a power cut

### Requirement: Remote Microcontroller Wake Trigger
The controller SHALL accept an active-high 3.3V logic pulse on ESP_WAKE from an external microcontroller to turn on the ATX power supply when the host is off.

#### Scenario: Remote wake from standby
- **WHEN** the host is off (HOST_ON is 0V) and a 3.3V pulse >= 50ms is received on ESP_WAKE
- **THEN** PS_ON# is pulled low and held for at least 2.5 seconds, starting the ATX power supply

### Requirement: Remote Wake Hardware Inhibit when Running
The controller SHALL hardware-clamp and ignore all signals on the ESP_WAKE input whenever HOST_ON is energized at 3.3V.

#### Scenario: Remote signal received while host is running
- **WHEN** the host is active (HOST_ON is 3.3V) and ESP_WAKE is driven to 3.3V
- **THEN** the ESP_WAKE input is shunted to GND via hardware clamp, no change occurs to PS_ON#, and BTN_OUT is not triggered

### Requirement: Chassis Power Status LED Indication
The controller SHALL switch a low-side driver to illuminate a connected power LED whenever HOST_ON is energized at 3.3V.

#### Scenario: System running LED state
- **WHEN** HOST_ON is 3.3V
- **THEN** the LED cathode pin (LED-) is connected to GND, sinking ~15mA through an onboard current-limiting resistor

#### Scenario: System standby LED state
- **WHEN** HOST_ON is 0V
- **THEN** the LED cathode pin (LED-) is high-impedance and the LED remains dark
