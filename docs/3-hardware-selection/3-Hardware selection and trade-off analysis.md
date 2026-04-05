# 3. Hardware Selection and Trade-Off Analysis

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-02-14
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[3-Hardware selection and trade-off analysis]]
> **↑ Parent:** [[README]]

---

## Overview

This document serves as the entry point for all hardware selection decisions. Each component is evaluated in a dedicated sub-document following a consistent trade-off methodology derived from the System Requirements Specification (SyRS).

### Selection Methodology

Each sub-document follows this structure:

| Section              | Purpose                                                                  |
| -------------------- | ------------------------------------------------------------------------ |
| Requirements Mapping | Which SyRS IDs this component satisfies                                  |
| Candidates Evaluated | 2–3 options compared on a common criteria table                          |
| Trade-Off Criteria   | Power, cost, interface compatibility, security features, availability    |
| Selection Rationale  | Why the chosen part wins                                                 |
| Risk Assessment      | Supply chain, driver complexity, documentation quality                   |

### Selection Order

Components are selected in dependency order — upstream decisions constrain downstream choices.

| #   | Component                        | Document                                                            | Status      |
| --- | -------------------------------- | ------------------------------------------------------------------- | ----------- |
| 3.1 | MCU                              | [[3.1-mcu-selection\|3.1 MCU Selection]]                            | Draft       |
| 3.2 | RF Wireless Module               | [[3.2-rf-wireless-module-selection\|3.2 RF Wireless Module]]        | Draft       |
| 3.3 | ToF Sensor                       | [[3.3-tof-sensor-selection\|3.3 ToF Sensor]]                        | Draft       |
| 3.4 | Magnetometer                     | [[3.4-magnetometer-selection\|3.4 Magnetometer]]                    | Draft       |
| 3.5 | Network Module                   | [[3.5-network-module-selection\|3.5 Network Module]]                | Draft       |
| 3.6 | Battery                          | [[3.6-battery-selection\|3.6 Battery]]                              | Not Started |
| 3.7 | Board vs Custom PCB              | [[3.7-board-vs-custom-pcb-selection\|3.7 Board vs Custom PCB]]      | Draft       |

> **Note:** Battery selection (3.6) is performed last because it depends on the full system power budget — which can only be calculated after all active components are selected.

