---
title: DALI
description: Instructions on how to integrate a DALI bus through a USB interface with Home Assistant.
ha_category:
  - Binary sensor
  - Button
  - Hub
  - Light
  - Sensor
ha_release: 2026.11
ha_iot_class: Local Polling
ha_codeowners:
  - '@markus-becker-tridonic-com'
ha_domain: dali
ha_config_flow: true
ha_platforms:
  - binary_sensor
  - button
  - diagnostics
  - light
  - number
  - select
  - sensor
ha_integration_type: hub
ha_quality_scale: gold
---

The **DALI** {% term integration %} connects Home Assistant directly to a
[DALI](https://www.dali-alliance.org/) (Digital Addressable Lighting Interface)
bus through a USB interface, such as the Lunatone DALI-USB2. DALI is the open
[IEC 62386](https://www.dali-alliance.org/) lighting control standard, and the
DALI Alliance certifies a large range of interoperable
[products](https://www.dali-alliance.org/products) from many manufacturers.

Unlike the [Lunatone](/integrations/lunatone/) and
[Sunricher DALI](/integrations/sunricher_dali/) integrations, which talk to a
networked DALI gateway over your local network, this integration drives a DALI
interface that is plugged directly into the Home Assistant host over USB and
speaks the DALI protocol on the bus itself.

{% note %}
Support for DALI-2 sensors and input devices is planned for future releases.
{% endnote %}

## Supported devices

- USB DALI interfaces based on the Lunatone DALI-USB2 (and compatible Tridonic
  DALI USB interfaces).
- DALI control gear of device type 6 (dimmable LED modules, IEC 62386-207).

The integration has been tested with a Lunatone DALI-USB2 interface and multiple
Tridonic devices on the DALI bus.

## Prerequisites

Before setting up the DALI integration, ensure you have:

1. A supported DALI USB interface connected to the Home Assistant host.
2. Your DALI control gear wired to the bus and the bus powered. The interface
   can power the bus itself if it has a built-in supply; otherwise use an
   external DALI bus power supply.

{% include integrations/config_flow.md %}

When the interface is plugged in, Home Assistant discovers it automatically. You
can also add it manually by selecting its serial port.

{% configuration_basic %}
Device:
    description: "The serial port of the DALI USB interface, for example `/dev/ttyACM0`. When the interface is discovered over USB this is filled in for you."
Power the DALI bus:
    description: "Whether Home Assistant should power the DALI bus from the interface. Only offered for interfaces that have a switchable bus supply. Leave this off if the bus is powered externally."
{% endconfiguration_basic %}

## Supported functionality

### Entities

#### Binary sensors

- **Problem**
  - **Description**: Reports failure conditions a control gear exposes in its
    control-gear and light-source diagnostics (IEC 62386 memory banks 205 and
    206 / DiiA Part 253), such as control-gear or light-source failures.
  - **Available for devices**: Control gear that implements the Part 253
    diagnostics memory banks.

- **Emergency status and failures**
  - **Description**: Reports the status and failure bits of self-contained
    emergency control gear (IEC 62386-202 memory bank 208), such as battery,
    lamp, and test failures.
  - **Available for devices**: DALI self-contained emergency control gear
    (device type 1).

#### Lights

- **Brightness**
  - **Description**: Turn dimmable LED control gear on and off and set its
    brightness. The integration follows the gear's configured dimming curve
    (logarithmic or linear) when translating between Home Assistant brightness
    and the DALI arc-power level.
  - **Available for devices**: DALI control gear of device type 6.

- **Color temperature**
  - **Description**: Set the color temperature of color control gear, within the
    gear's configured (or physical) warmest/coolest limits.
  - **Available for devices**: DALI color control gear of device type 8
    (IEC 62386-209) that support tunable white.

- **XY color**
  - **Description**: Set the CIE xy color of color control gear.
  - **Available for devices**: DALI color control gear of device type 8
    (IEC 62386-209) that support xy color.

#### Sensors

Diagnostic sensors expose the identification data a control gear reports in its
memory banks, for example its GTIN, identification number, serial and batch
number, firmware and hardware version, DALI short address, and — for devices
that provide it — luminaire information (IEC 62386-102 memory bank 1 / DiiA
Part 251). Most of these are disabled by default.

For control gear that implements the DiiA extended memory banks, additional
sensors expose control-gear and light-source **diagnostics** (memory banks 205
and 206 / Part 253) such as operating times, start counters and temperatures,
luminaire **maintenance** / rated life (memory bank 207 / Part 253), and active
**energy** (memory bank 202 / Part 252).

Self-contained emergency control gear (device type 1) additionally exposes
**emergency** sensors such as the emergency mode, battery charge, and the
various test durations and counters.

#### Numbers

Configuration entities let you adjust a control gear's **minimum level**,
**maximum level**, **system failure level**, and **extended fade time**.

#### Selects

Configuration entities let you adjust a control gear's **fade time** and
**fade rate**.

#### Buttons

- **Identify**
  - **Description**: Flash the control gear so you can locate it physically.
- **Rescan**
  - **Description**: Trigger a new scan of the DALI bus for control gear.
- **Emergency commands**
  - **Description**: Start and stop the emergency function and duration tests
    and other emergency commands on self-contained emergency control gear.
  - **Available for devices**: DALI self-contained emergency control gear
    (device type 1).

## Data updates

The integration {% term polling polls %} the DALI bus every 30 seconds to update
the state of the control gear.

## Troubleshooting

### The bus is not powered

If the interface reports that the DALI bus is unpowered or overloaded, Home
Assistant raises a repair issue that offers to power the bus from the interface
(when the interface supports it). If the bus is powered externally, check the
external DALI bus power supply.

### Control gear is not addressed

Control gear without a DALI short address cannot be controlled. When Home
Assistant detects unaddressed control gear, it raises a repair issue that offers
to assign short addresses. If two or more control gear share the same short
address, a repair issue offers to resolve the address collision.

## Removing the integration

This integration follows standard integration removal. No extra steps are
required.

{% include integrations/remove_device_service.md %}
