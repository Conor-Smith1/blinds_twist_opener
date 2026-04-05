# Blinds Twist Opener

This project packages the pieces needed to motorize twist-style window blinds with an ESP32, a stepper motor, a hall effect home sensor, and a small 3D-printed enclosure/adapter set. The goal is to automate blinds without permanently modifying an apartment or rental space: the control box can be mounted with removable adhesive, and the printed coupler drives the existing blind wand/shaft.

## What is in this repository

- [`smartblinds.yaml`](smartblinds.yaml): ESPHome firmware for the ESP32, including homing, open/close/position control, and Home Assistant entities.
- [`3D Printer Models`](3D%20Printer%20Models): Printable `.stl` files for the enclosure and blind interface parts.
- [`CAD`](CAD): Source `.step` files for the printable parts.
- [`Wiring Diagarm`](Wiring%20Diagarm): WireViz source plus exported wiring images and reference photos.
- [`BoM.xlsx`](BoM.xlsx): Bill of materials with example parts and source links.

## Hardware overview

The included BOM and wiring files point to a build based on:

- ESP32 dev board
- TMC2209 stepper driver
- NEMA 14 bipolar stepper motor
- A3144 hall effect sensor
- 12 V power supply
- MP1584EN buck converter to power the ESP32
- 220 uF capacitor across the motor supply
- 10 mm x 3 mm magnet for homing
- 1/4 in aluminum rod plus jumper wiring

The spreadsheet title is "Home Assistant Ready Automated Twist Blind Opener", and the example parts list uses readily available hobby components rather than custom electronics.

## Wiring and pin map

The wiring diagram in [`Wiring Diagarm/smart_blinds_wireviz.yml`](Wiring%20Diagarm/smart_blinds_wireviz.yml) maps the ESP32 like this:

- `GPIO25`: motor enable
- `GPIO26`: step
- `GPIO27`: direction
- `GPIO33`: hall effect sensor output
- `3V3` / `GND`: hall sensor power
- `5V` / `GND`: ESP32 power from the buck converter

Power is split two ways:

- 12 V directly to the stepper driver motor supply (`VMOT` + `GND`)
- 12 V into the buck converter, reduced to 5 V for the ESP32

The motor driver is shown as a TMC2209, but the ESPHome config uses the `a4988` stepper platform. That is reasonable here because ESPHome is only generating `STEP`, `DIR`, and `EN` signals; the TMC2209 is effectively being treated like a standalone step/dir driver rather than configured over UART.

## Firmware behavior

The ESPHome config creates:

- A `cover` entity named `Bedroom Blinds`
- A `binary_sensor` named `Blind Home Sensor`
- A `sensor` named `Blind Position Percent`
- Buttons for `Home Blinds`, `Open One Turn`, and `Close One Turn`

On boot, the device automatically runs a homing script:

1. The motor is enabled.
2. The stepper position is seeded to a positive value.
3. The firmware searches toward the hall sensor in small negative moves.
4. Once the sensor triggers, it backs off by a configured close offset.
5. Position `0` is recorded as the fully closed reference point.

If homing never finds the sensor within the allowed travel window, the firmware sets a failure flag and stops the motor.

## Important tuning values

These values in [`smartblinds.yaml`](smartblinds.yaml) will likely need to be adjusted for your blinds and gearing:

- `steps_per_turn: 1600`
- `total_steps: 8000`
- `close_offset_steps: 2400`
- `max_speed: 1200 steps/s`
- `acceleration: 400`
- `deceleration: 400`

What they mean:

- `steps_per_turn`: how many motor steps make one full twist of the blind mechanism
- `total_steps`: full travel from closed to open
- `close_offset_steps`: how far to move after the hall sensor is detected so the blinds land on the true closed position

## How to use the ESPHome config

1. Copy or edit [`smartblinds.yaml`](smartblinds.yaml).
2. Replace the Wi-Fi placeholders with your SSID and password.
3. Update or remove the `manual_ip` block if it does not match your network.
4. Flash the ESP32 with ESPHome.
5. Verify the hall sensor polarity and motor direction before letting the blinds run a full cycle.
6. Tune `steps_per_turn`, `total_steps`, and `close_offset_steps` until the physical open/close positions match the reported cover position.

If the motor moves away from home during startup, reverse the motor direction in firmware or swap the appropriate motor wiring at the driver.

## Files you will probably want first

- Use [`smartblinds.yaml`](smartblinds.yaml) if you are building or modifying the firmware.
- Use [`Wiring Diagarm/smart_blinds_wireviz.svg`](Wiring%20Diagarm/smart_blinds_wireviz.svg) or [`Wiring Diagarm/smart_blinds_wireviz.png`](Wiring%20Diagarm/smart_blinds_wireviz.png) during assembly.
- Use the `.stl` files in [`3D Printer Models`](3D%20Printer%20Models) to print parts directly.
- Use the `.step` files in [`CAD`](CAD) if you want to modify the mechanical design.
- Use [`BoM.xlsx`](BoM.xlsx) if you want the original parts list and purchase references.

## Notes and assumptions

- The repository currently documents a single blind setup named `Bedroom Blinds`; you will probably want to rename entities for your room/device.
- The hall sensor input is configured with pull-up and inverted logic, with 20 ms debounce on both edges.
- The motor enable output is also inverted in firmware.
- This design appears intended for tilt/twist blinds, not lift-style blinds that raise and lower the whole shade.

## License

See [`LICENSE`](LICENSE).
