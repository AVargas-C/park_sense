# 5. Firmware Architecture Design

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-03-14
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[5-firmware-architecture-design]]
> **↑ Upstream:** [[4-system-architecture-design]] (IEEE 42010:2011)

---

## 1. Purpose of This Document Set

This Firmware Architecture Design (FAD) constitutes the **Software Design Description (SDD)** for ParkSense firmware, written in accordance with **IEEE 1016-2009**. It refines the system-level architecture defined in `docs/4-system-architecture-design/` (IEEE 42010:2011) into implementation-level detail: memory layouts, execution timing, state machine logic, data structure definitions, error handling strategies, and build configuration.

**Relationship between docs/4.x and docs/5.x:**

| System Architecture (4.x — IEEE 42010) | Firmware Architecture (5.x — IEEE 1016) |
|----------------------------------------|------------------------------------------|
| *What* components exist and *why* | *How* each module is implemented in C |
| Deployment topology, protocol selection | Flash memory map, linker regions |
| PDM described as a black-box module | PDM FSM coded as states + transitions + guards |
| Data flow as a pipeline concept | ISR flags → volatile buffers → main-loop consumer |
| Interface as API signatures | Full `struct`/`enum` definitions, error codes, buffer sizes |
| Power states as a budget table | `PWR_EnterStop2()` implementation, RTC alarm setup |

---

## 2. Scope

This SDD covers the embedded firmware running on the **STM32U585AII6Q** application MCU for both the IoT Node and Gateway targets. It does not cover:

- Server backend software (separate repository)
- Web dashboard frontend (separate repository)
- STM32WB5MMGH6TR binary Zigbee stack (ST-provided, opaque binary)

---

## 3. Applicable Standards and Constraints

| Standard / Constraint | Applicability |
|-----------------------|---------------|
| IEEE 1016-2009 | Governs this SDD structure and viewpoints |
| ISO/IEC/IEEE 12207:2017 | Software lifecycle; SDD is a required output |
| MISRA-C:2012 | Coding constraints — no dynamic allocation, no recursion, restricted pointer arithmetic |
| CMSIS 5 | ARM Cortex-M33 register access and startup conventions |
| STM32U5 HAL v1.x | ST-provided peripheral drivers; ParkSense drivers wrap these |

---

## 4. Design Viewpoints (IEEE 1016 §5)

IEEE 1016-2009 requires the SDD to be organized by design viewpoints. Each viewpoint addresses a specific design concern and is realized by a view document.

| Viewpoint | IEEE 1016 Category | Concern Addressed | View Document |
|-----------|--------------------|-------------------|---------------|
| **Firmware Decomposition** | Composition | Module tree, file inventory, per-module responsibility, build-target inclusion | [[5.1-firmware-decomposition]] |
| **Memory Map** | Information | Flash layout, SRAM regions, TrustZone partitioning, linker script regions, stack/heap sizing | [[5.2-memory-map]] |
| **Execution Model** | Behavioral | Super-loop sequencing, ISR catalog + priorities, cooperative scheduling, timing constraints, watchdog | [[5.3-execution-model]] |
| **State Machines** | Behavioral | PDM FSM, Network FSM, OTA FSM — states, transitions, guards, actions | [[5.4-state-machines]] |
| **Data Structures** | Information | All key `struct`/`enum`/`typedef` types, global singletons, ISR-to-main communication, no-heap justification | [[5.5-data-structures]] |
| **Error Handling** | Behavioral | Fault handlers, error codes, watchdog reset flow, fault logging, error propagation rules | [[5.6-error-handling]] |
| **Build Configuration** | Composition | CMake targets, compile-time constants, runtime config (node ID / space ID), MISRA-C deviation list | [[5.7-build-configuration]] |
| **BSP / HAL Design** | Composition | Peripheral ownership, clock tree, GPIO pin table, TrustZone init sequence, Secure MPU, NS MPU, NSC veneer catalog, STOP2 entry/exit | [[5.21-bsp-hal-design]] |

---

## 5. Traceability to System Architecture and SyRS

### 5.1 From System Architecture (docs/4.x) to Firmware Design (docs/5.x)

```
4.3 Logical Architecture           5.1 Firmware Decomposition
    Layer definitions      ──────►     Module tree with file inventory
    Module responsibilities            Per-module API + implementation notes

4.4 Interface Architecture         5.5 Data Structures
    C API signatures       ──────►     struct/enum definitions
    Error return codes                 Error code enumerations

4.5 Data Flow                      5.3 Execution Model
    Stage pipeline         ──────►     Super loop step sequence
    Trigger conditions                 ISR → volatile flag → main loop

4.6 Security Architecture         5.2 Memory Map
    TrustZone partitioning ──────►     SAU region table
    Secure boot chain                  Flash sector assignments

4.7 Power Architecture            5.3 Execution Model
    Duty cycle diagram     ──────►     Sleep entry/exit code path
    Stop 2 wake source                 RTC alarm ISR
```

### 5.2 From SyRS (docs/2.x) to Firmware Design (docs/5.x)

| SyRS Requirement | Firmware Design Section |
|------------------|------------------------|
| SYS-F-001 through SYS-F-010 (node functional) | 5.1 (PDM module), 5.4 (PDM FSM) |
| SYS-F-005 (super loop cycle) | 5.3 (execution model) |
| SYS-F-020 through SYS-F-023 (gateway functional) | 5.1 (CPM module, gateway app) |
| SYS-P-001 (cycle ≤ 1 s) | 5.3 (timing budget) |
| SYS-P-003 (30 s sleep) | 5.3 (RTC alarm), 5.7 (`SLEEP_INTERVAL_S`) |
| SYS-I-004 (RF packet structure) | 5.5 (`cpm_packet_t`) |
| SYS-I-005 through SYS-I-009 (reliability) | 5.4 (Network FSM), 5.5 (`cpm_tx_ctx_t`) |
| SYS-S-001 through SYS-S-004 (secure boot) | 5.2 (boot flash layout, image header) |
| SYS-S-005 through SYS-S-013 (comm security) | 5.4 (Network FSM join sequence) |
| SYS-S-008 (TrustZone + MPU) | 5.2 (SAU region table) |
| SYS-NF-001 (driver swap at Layer 3) | 5.1 (driver abstraction), 5.7 (CMake targets) |
| SYS-NF-003 (MISRA-C) | 5.7 (MISRA deviation list) |
| SYS-NF-004 (reproducible builds) | 5.7 (Docker + CMake toolchain) |
| SYS-NF-005 (host-executable unit tests) | 5.7 (test build target) |

---

## 6. Firmware Design Decisions Log

| ID | Decision | Rationale | Traced To |
|----|----------|-----------|-----------|
| FD-001 | No dynamic memory allocation (`malloc`/`free` banned) | MISRA-C Rule 21.3; deterministic memory footprint; eliminates fragmentation and use-after-free risks | SYS-NF-003 |
| FD-002 | No recursion | MISRA-C Rule 17.2; deterministic stack depth; stack overflow prevention | SYS-NF-003 |
| FD-003 | ISR-to-main communication via volatile flags + static circular buffers | Avoids dynamic allocation; deterministic latency; lock-free for single producer / single consumer | AD-002 |
| FD-004 | All module-global state wrapped in a single `static` context struct per module | Encapsulation without heap; enables unit test state injection; single point of initialization | FD-001 |
| FD-005 | CRC-16/CCITT-FALSE for CPM packet integrity | Matches Zigbee stack CRC polynomial family; lightweight; no HW accelerator needed for 12-byte payload | SYS-I-007 |
| FD-006 | IWDG (Independent Watchdog) with 4 s timeout | Recovers from software hang; IWDG runs from LSI, independent of system clock — survives clock failure | SYS-P-001 |
| FD-007 | Linker-enforced stack size: 4 KB (node), 8 KB (gateway) | Static analysis of max call depth; gateway needs deeper stack for WiFi driver + Zigbee coordinator | FD-002 |
| FD-008 | Fault log stored in last 4 KB flash sector | Survives reset; allows post-mortem analysis via SWD read or OTA telemetry | — |

---

## 7. Document Map

```
docs/5-firmware-architecture-design/
├── 5-firmware-architecture-design.md       ← This index (you are here)
│
├── 5.01-firmware-decomposition.md          ← Module tree, file inventory, build inclusion matrix
├── 5.02-memory-map.md                      ← Flash + SRAM layout, SECWM, SAU, MPU, linker symbols
├── 5.03-execution-model.md                ← Super loop, ISRs, timing budget, watchdog
├── 5.04-state-machines.md                 ← PDM FSM, Network FSM, OTA FSM
├── 5.05-data-structures.md                ← struct/enum/typedef definitions, singletons
├── 5.06-error-handling.md                 ← Fault handlers, error codes, fault log ring buffer
├── 5.07-build-configuration.md            ← CMake targets, compile flags, MISRA deviations
│
├── 5.08-bootloader-design.md              ← Boot flow, ECDSA validation, slot swap logic
├── 5.09-module-headers-and-metadata.md    ← ps_image_header_t, magic, version, CRC, signature
│
├── 5.10-communication-protocol-module.md  ← CPM design: TX/RX FSM, retry, queue
├── 5.11-rf-driver-api.md                  ← rf_api.h contract: Zigbee SED + Coordinator
├── 5.12-rf-packet-design.md               ← cpm_packet_t wire format, CRC-16
├── 5.13-parking-detection-design.md       ← PDM: sensor fusion, FSM, calibration
├── 5.14-sensor-drivers-api.md             ← sensor_api.h: ToF (VL53L5CX), magnetometer (IIS2MDCTR)
│
├── 5.15-application-design.md             ← Node + Gateway main loops, super loop steps
├── 5.16-server-design.md                  ← Gateway HTTP client, server API, TLS config
├── 5.17-database-design.md                ← Server-side DB schema (reference)
├── 5.18-gui-design.md                     ← React dashboard (reference)
│
├── 5.19-power-management-design.md        ← STOP2 duty cycle, wake sources, power budget
├── 5.20-network-driver-api.md             ← net_api.h: WiFi (EMW3080B) + Ethernet (W5500)
└── 5.21-bsp-hal-design.md                 ← BSP init, peripheral ownership, clock, MPU, veneer
```
