# 4. System Architecture Design

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-02-28
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[README]]

---

## 1. Purpose of This Document Set

This Architecture Description (AD) documents the system architecture of ParkSense in accordance with **IEEE 42010:2011**. The standard defines architecture description as a work product expressing the architecture of a system, organized by viewpoints that address specific stakeholder concerns.

This index document defines:
- The system of interest (SoI)
- Stakeholders and their concerns
- Selected viewpoints and the view document that addresses each
- Traceability to the System Requirements Specification (SyRS)

---

## 2. System of Interest

**ParkSense** is a battery-powered IoT parking occupancy system. It consists of wireless sensor nodes deployed in individual parking spaces, a gateway that aggregates node state, a server that processes and stores occupancy data, and a web dashboard that displays real-time lot availability.

The scope of this Architecture Description covers:

| In Scope | Out of Scope |
| -------- | ------------ |
| Firmware architecture (node + gateway) | Cloud hosting platform |
| Hardware deployment topology | Server backend implementation details |
| Inter-device communication protocols | UI framework internals |
| Security architecture (device level) | Network infrastructure (routers, switches) |
| Power architecture (node + gateway) | Physical installation (enclosures, conduit) |

---

## 3. Stakeholders and Concerns

Per IEEE 42010:2011 §4.2, stakeholders and their concerns must be identified before selecting viewpoints.

| Stakeholder | Role | Key Concerns |
| ----------- | ---- | ------------ |
| **Embedded Firmware Engineer** | Implements node + gateway firmware | Layer decomposition, driver APIs, module boundaries, build targets |
| **Security Engineer** | Reviews security posture | Trust boundaries, encryption points, key lifecycle, secure boot chain |
| **Systems Engineer** | Validates against SyRS | Requirements traceability, interface definitions, data flow correctness |
| **Power / Hardware Engineer** | Designs power supply and battery selection | Power domains, sleep states, average current, battery life |
| **Hiring Manager / Reviewer** | Evaluates architecture quality | Clarity of design decisions, traceability, professional presentation |
| **Student / Learner** | Studies embedded development lifecycle | Step-by-step documentation of each architectural decision |

---

## 4. Architectural Viewpoints

Per IEEE 42010:2011 §5.3, each viewpoint addresses a set of stakeholder concerns and is realized by one or more views (documents).

| Viewpoint | Concern Addressed | Stakeholders | View Document |
| --------- | ----------------- | ------------ | ------------- |
| **System Context** | System boundary, external actors, interactions with environment | Systems Engineer, Reviewer | [[4.1-system-context]] |
| **Physical Architecture** | Hardware deployment, network topology, board-level connections | HW Engineer, Firmware Engineer | [[4.2-physical-architecture]] |
| **Logical Architecture** | Software structure, module decomposition, layer dependencies | Firmware Engineer, Reviewer | [[4.3-logical-architecture]] |
| **Interface Architecture** | All external and internal interfaces, protocols, signal types | Firmware Engineer, Systems Engineer | [[4.4-interface-architecture]] |
| **Data Flow** | End-to-end data path from sensor reading to dashboard display | Systems Engineer, Reviewer | [[4.5-data-flow]] |
| **Security Architecture** | Trust boundaries, encryption points, key hierarchy, secure boot | Security Engineer, Systems Engineer | [[4.6-security-architecture]] |
| **Power Architecture** | Power domains, sleep states, duty cycle, battery life budget | Power Engineer, Firmware Engineer | [[4.7-power-architecture]] |

---

## 5. Architecture Models Used

Per IEEE 42010:2011 §5.5, each view uses architecture models appropriate to its viewpoint.

| Model Type | Used In |
| ---------- | ------- |
| Context diagram (Mermaid) | 4.1 System Context |
| Deployment / topology diagram (Mermaid + ASCII) | 4.2 Physical Architecture |
| Layer decomposition diagram (ASCII) | 4.3 Logical Architecture |
| Interface table + sequence diagram (Mermaid) | 4.4 Interface Architecture |
| Data flow diagram (Mermaid) | 4.5 Data Flow |
| Trust boundary + boot chain diagram (ASCII + Mermaid) | 4.6 Security Architecture |
| Power state machine + budget table | 4.7 Power Architecture |

---

## 6. Relationship to Other Documents

```
SyRS (IEEE 29148-2018)
  └── Requirements that this architecture satisfies
        ├── 4.1 System Context     ← SYS-F-001, SYS-I-001
        ├── 4.2 Physical           ← SYS-F-002, SYS-P-001 through SYS-P-004
        ├── 4.3 Logical            ← SYS-F-003 through SYS-F-007, SYS-NF-001
        ├── 4.4 Interfaces         ← SYS-I-001 through SYS-I-013
        ├── 4.5 Data Flow          ← SYS-F-001, SYS-F-004, SYS-P-001
        ├── 4.6 Security           ← SYS-S-001 through SYS-S-013
        └── 4.7 Power              ← SYS-C-001 through SYS-C-004, SYS-P-005

Hardware Selection (docs/3-hardware-selection/)
  └── Confirms physical components used in 4.2, 4.4, 4.7
        ├── 3.1 MCU → STM32U585AII6Q
        ├── 3.2 RF  → STM32WB5MMGH6TR (Zigbee 3.0)
        ├── 3.3 ToF → VL53L5CXV0GC/1
        ├── 3.4 Mag → IIS2MDCTR
        ├── 3.5 Network Module → net_api.h (EMW3080B WiFi Phase 1 / W5500 Ethernet Phase 2)
        └── 3.7 Board → B-U585I-IOT02A Discovery Kit
```

---

## 7. Architecture Decisions Log

Key decisions recorded here for traceability. Full rationale is in the hardware selection documents.

| ID | Decision | Rationale | Source |
| -- | -------- | --------- | ------ |
| AD-001 | Single source tree, two compile targets (`TARGET_NODE` / `TARGET_GATEWAY`) | Code reuse, portability (SYS-NF-001) | [[README]] §5 |
| AD-002 | Cooperative bare-metal super loop (no RTOS) | Deterministic timing, low RAM overhead, simpler security model (SYS-F-005) | [[3.2-rf-wireless-module-selection]] |
| AD-003 | Zigbee 3.0 as RF protocol (BLE 5.4 as Plan B) | All SyRS security + reliability + scalability requirements met; STM32WB5MMG on dev kit | [[3.2-rf-wireless-module-selection]] |
| AD-004 | Star topology (coordinator + sleepy end devices) | Battery life requirement ≥ 5 years; mesh relay eliminates battery feasibility (SYS-I-001) | [[3.2-rf-wireless-module-selection]] |
| AD-005 | ECDSA P-256 firmware signing + CRC-32 in image header | Secure boot chain integrity (SYS-S-001 through SYS-S-004) | [[4.6-security-architecture]] |
| AD-006 | Abstract `net_api.h` network driver (Layer 3c) with two compile-time implementations: EMW3080B WiFi (Phase 1, pre-installed on dev kit) and WIZnet W5500 Ethernet (Phase 2, breakout via SPI). CPM calls `NET_connect / NET_send / NET_recv` — transport is invisible to upper layers. Selected via `-DNET_TRANSPORT_WIFI` / `-DNET_TRANSPORT_ETHERNET`. | Eliminates 2.4 GHz Zigbee co-interference for Phase 2; single CPM codebase; field-selectable without PCB respin | [[3.5-network-module-selection]] |
| AD-007 | Hardware crypto (AES, PKA, SHA, RNG) via STM32U585 HW accelerators | Performance + side-channel resistance vs software AES | [[3.1-mcu-selection]] |
| AD-008 | Docker containerized build environment | Reproducible builds, identical local and CI toolchain | [[1.7-build-and-cicd]] |
