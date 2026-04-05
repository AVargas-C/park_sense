# 6. Cybersecurity

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-03-28
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[README]]
> **↑ Upstream:** [[4.6-security-architecture]] · [[5.6-error-handling]]

---

## 1. Purpose of This Document Set

This folder contains the **full Cybersecurity Engineering documentation** for ParkSense. It extends the Security Architecture view (docs/4.6) and error handling design (docs/5.6) into formal, standard-aligned security engineering artifacts.

### Relationship to Other Documents

| Existing Document | What It Covers | What docs/6.x Adds |
|--------------------|---------------|---------------------|
| [[4.6-security-architecture]] | *What* controls exist: TrustZone, ECDSA, Zigbee AES, TLS | *Why* — adversary model, algorithm selection rationale |
| [[5.6-error-handling]] | Fault handlers, recovery paths | Vulnerability response process, SDL security gates |
| [[2-system-requirements-specification]] §7 | Security requirements (SYS-S-001 through SYS-S-013) | Compliance evidence per ETSI EN 303 645 |

---

## 2. Governing Standards

### Primary: ETSI EN 303 645 v2.1.1 (2021)

*Cybersecurity for Consumer IoT: Baseline Requirements*

The primary IoT cybersecurity standard, mandatory under:
- **UK PSTI Act 2024** (Product Security and Telecommunications Infrastructure)
- **EU Cyber Resilience Act (CRA)** — effective 2027

This standard defines 13 provisions covering: no universal default passwords, vulnerability disclosure, software updates, credential storage, secure communication, minimised attack surface, integrity checking, personal data protection, resilience to outages, telemetry examination, ease of deletion, installation guidance, and input validation.

### Supporting Standards

| Standard | Role |
|----------|------|
| **IEC 62443-4-2:2019** | Component-level security capability requirements (FR 1–7) for the IoT node as an embedded device |
| **NIST SP 800-82 Rev. 3** | OT/IoT security guide — threat landscape, network segmentation, patching strategy |
| **OWASP IoT Top 10 (2018)** | Attack surface areas specific to IoT products |
| **NIST SP 800-57 Part 1 Rev. 5** | Key management recommendations — algorithm selection, key sizes, lifecycle |

---

## 3. Scope

This cybersecurity documentation covers:

| In Scope | Out of Scope |
|----------|-------------|
| IoT node firmware security | Server-side application security (separate codebase) |
| Gateway firmware security | Cloud hosting security (provider responsibility) |
| Zigbee 3.0 communication security | Physical installation security (operations scope) |
| WiFi gateway-to-server TLS | Network infrastructure (firewall, VLAN) |
| Firmware signing and OTA security | Web dashboard frontend security |
| Device commissioning and key lifecycle | End-user credential management |
| Secure development and release process | |

---

## 4. System Threat Profile

ParkSense is an infrastructure IoT system deployed in parking lots. Its cybersecurity risk profile:

| Attribute | Assessment |
|-----------|-----------|
| **Data sensitivity** | Low — occupancy state (occupied/free) is not personal data |
| **Safety criticality** | Low — misreported occupancy does not create physical hazard |
| **Availability requirement** | Medium — system failure reduces parking efficiency; not life-critical |
| **Attack motivation** | Low direct financial incentive; potential for competitive intelligence or facility disruption |
| **Physical accessibility** | High — nodes are deployed in open parking lots; accessible to any person |
| **RF accessibility** | High — 2.4 GHz transmissions are freely interceptable in the field |
| **Network accessibility** | Low — gateway is on a private LAN; server may be on a private network |

**Overall risk posture: Medium-Low.** Security controls are proportionate — strong firmware integrity and communication confidentiality; not hardened against nation-state actors.

---

## 5. Document Map

```
docs/6-cybersecurity/
├── 6-cybersecurity.md              ← This index (you are here)
├── 6.1-threat-model.md             ← STRIDE analysis, attack trees, adversary profiles
├── 6.2-cryptographic-design.md     ← Algorithm selection, key lifecycle, forbidden list
├── 6.3-secure-development-lifecycle.md  ← SSDLC, static analysis, CVD policy
├── 6.4-penetration-test-plan.md    ← Test cases per attack surface
└── 6.5-compliance-checklist.md     ← ETSI EN 303 645 provision-by-provision checklist
```

---

## 6. Security Controls Summary

| Control | Location | Standard Reference |
|---------|----------|--------------------|
| Unique per-device identity (IEEE EUI-64 burned in STM32WB5) | [[4.6-security-architecture]] §4.1 | ETSI EN 303 645 §5.1 |
| No universal default passwords | [[5.7-build-configuration]] §3 (per-device config) | ETSI EN 303 645 §5.1 |
| Secure boot (ECDSA P-256) | [[4.6-security-architecture]] §4.2 | ETSI EN 303 645 §5.4, IEC 62443-4-2 CR 7.4 |
| Encrypted communication (Zigbee AES-128-CCM, TLS 1.2) | [[4.6-security-architecture]] §4.3–4.4 | ETSI EN 303 645 §5.5, IEC 62443-4-2 CR 4.1 |
| No keys in plaintext / source code | [[4.6-security-architecture]] §5 | ETSI EN 303 645 §5.4, NIST SP 800-57 |
| SWD debug port lockout (RDP Level 2) | [[4.6-security-architecture]] §4.5 | ETSI EN 303 645 §5.6, IEC 62443-4-2 CR 5.2 |
| OTA update integrity (ECDSA + CRC) | [[4.6-security-architecture]] §4.6 | ETSI EN 303 645 §5.3 |
| Replay protection (Zigbee frame counter) | [[4.6-security-architecture]] §4.3 | SYS-S-006 |
| TrustZone isolation | [[5.2-memory-map]] §5 | ETSI EN 303 645 §5.6, IEC 62443-4-2 CR 3.4 |
| MISRA-C compliance (reduced attack surface) | [[5.7-build-configuration]] §12 | ETSI EN 303 645 §5.6 |
| Fault logging (post-mortem analysis) | [[5.6-error-handling]] §7 | ETSI EN 303 645 §5.7 |
| Vulnerability disclosure policy | [[6.3-secure-development-lifecycle]] | ETSI EN 303 645 §5.2 |

---

## 7. Traceability to SyRS Security Requirements

| SyRS ID | Requirement Summary | Evidence |
|---------|--------------------|---------|
| SYS-S-001 | Bootloader verifies FW signature | [[4.6-security-architecture]] §4.2, [[6.2-cryptographic-design]] §4 |
| SYS-S-002 | FW signed before deployment | [[5.7-build-configuration]] §10, [[6.3-secure-development-lifecycle]] §3 |
| SYS-S-003 | Boot halts on signature failure | [[4.6-security-architecture]] §4.2, [[6.4-penetration-test-plan]] test PT-FW-002 |
| SYS-S-004 | CRC + version in image header | [[5.5-data-structures]] §10 (`ps_image_header_t`) |
| SYS-S-005 | RF payload encrypted AES-128 | [[4.6-security-architecture]] §4.3, [[6.2-cryptographic-design]] §3 |
| SYS-S-006 | Replay protection | [[6.1-threat-model]] T-04; Zigbee frame counter |
| SYS-S-007 | Mutual authentication node ↔ gateway | Trust Center join + Install Code; [[6.2-cryptographic-design]] §5 |
| SYS-S-008 | TrustZone + MPU isolation | [[5.2-memory-map]] §5, [[6.4-penetration-test-plan]] test PT-HW-001 |
| SYS-S-009 | No plaintext key storage | [[6.2-cryptographic-design]] §6 |
| SYS-S-010 | MIC/HMAC on RF payloads | Zigbee AES-128-CCM MIC (4 bytes); [[6.2-cryptographic-design]] §3 |
| SYS-S-011 | Key management policy | [[6.2-cryptographic-design]] §6 |
| SYS-S-012 | Secure session establishment | Zigbee Trust Center join; [[6.1-threat-model]] §4 |
| SYS-S-013 | Commissioning security | Install Code enrollment; [[6.2-cryptographic-design]] §5 |
