# 7. Development Plan

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-04-04
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[README]]

---

## 1. Purpose

This document defines the implementation sequence for ParkSense firmware, tools, and backend. Milestones are ordered by hard dependency: each milestone's exit criteria must be met before the next begins. No timeline estimates are given — the plan governs order, not schedule.

---

## 2. Dependency Overview

```
M0 — Doc stubs
  └── M1 — HAL / BSP  ←── linker scripts from memory map (5.02)
        └── M2 — CI/CD Basic  ←── HAL compiles cleanly
              ├── M3 — Bootloader + Tools  ←── flash map finalised
              ├── M4 — Sensor Drivers
              │     └── M5 — Parking Detection Module
              ├── M6 — RF Drivers
              │     └── M8 — Communication Protocol Module
              └── M7 — Network Driver (Gateway)
                    └── M8 — Communication Protocol Module
                          └── M9 — Integration Test
                                └── M10 — Application Layer
                                      └── M11 — CI/CD Full Pipeline
                                            └── M12 — Server & UI
                                                  └── M13 — OTA
```

---

## 3. Milestones

---

### M0 — Close Documentation Stubs

**Depends on:** Nothing
**Blocks:** M1 (linker scripts cannot be written without 5.02)

Three critical design docs are stubs. They must be completed before any code is written — each feeds directly into the files that will be created in M1.

| Doc | Content Required | Why It Blocks Code |
|-----|------------------|--------------------|
| [[5.01-firmware-decomposition]] | Complete module tree, per-file list, build-target inclusion table | Defines the folder/file structure for `firmware/src/` |
| [[5.02-memory-map]] | Flash layout (bootloader / app A / app B / OTA staging / fault log), SRAM regions, SAU table, linker script regions | Determines every address used in linker scripts and `ps_image_header_t` |
| [[5.08-bootloader-design]] | Boot flow, image validation sequence (CRC-32 + ECDSA P-256), slot swap logic, jump-to-application | Required before bootloader code can be structured |
| [[5.09-module-headers-and-metadata]] | `ps_image_header_t` struct layout in full | Required by both bootloader and `inject_header.py` |

**Exit criteria:** All four docs contain substantive content, no TBD sections. Memory map includes all flash sector addresses and sizes. `ps_image_header_t` is fully defined.

---

### M1 — HAL / BSP

**Depends on:** M0 (5.02 memory map → linker scripts)
**Blocks:** M2, M3, M4, M6, M7

Bring up the STM32U585AII6Q on the B-U585I-IOT02A dev kit. This milestone delivers the foundation every other module builds on.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/bsp/bsp.c` / `bsp.h` | Clock tree init (160 MHz via PLL), GPIO init, peripheral ownership table |
| `firmware/src/bsp/stm32u585aii6q/startup_stm32u585aiixq.s` | Startup file (copied from STM32CubeU5 CMSIS) |
| `firmware/src/bsp/stm32u585aii6q/node.ld` | Linker script — Node binary |
| `firmware/src/bsp/stm32u585aii6q/gateway.ld` | Linker script — Gateway binary |
| `firmware/src/bsp/stm32u585aii6q/bootloader.ld` | Linker script — Bootloader binary |
| `firmware/CMakeLists.txt` | Top-level CMake; all three targets declared |
| `firmware/cmake/arm-none-eabi.cmake` | Cross-compilation toolchain file |

#### Tasks

1. Set up `firmware/` source tree as defined in [[5.01-firmware-decomposition]]
2. Configure STM32CubeU5 submodule selective compilation in CMake (HAL + CMSIS only at this stage)
3. Implement clock tree: HSI16 → PLL → SYSCLK 160 MHz, verify with SysTick
4. Configure GPIO for status LED and UART debug output (SWO or USART)
5. Write peripheral ownership table — assign SPI, I2C, UART instances to drivers
6. Write all three linker scripts from [[5.02-memory-map]] sector addresses
7. Confirm all three targets (`parksense_node.elf`, `parksense_gateway.elf`, `parksense_boot.elf`) link to correct flash regions

#### Proof of completion

- LED blinks at 1 Hz from `main_node.c` super loop
- SWO / UART output prints `SYSCLK: 160 MHz` and stack pointer value at boot
- All three `.elf` files built by CMake without warnings (`-Werror` enforced)
- `arm-none-eabi-size` output confirms each binary fits within its flash region

---

### M2 — CI/CD Build Pipeline (Basic)

**Depends on:** M1 (HAL compiles cleanly)
**Blocks:** All subsequent milestones (CI validates every commit from here)

Stand up the Docker-based build pipeline. From this point, every push is automatically validated. This milestone is intentionally narrow — only the build + static analysis + empty test suite stages.

#### Deliverables

| File | Description |
|------|-------------|
| `Dockerfile` | `parksense-build` image: arm-none-eabi-gcc 13.x, CMake, Ninja, cppcheck, Unity |
| `.github/workflows/ci.yml` | GitHub Actions pipeline |
| `firmware/test/CMakeLists.txt` | Host-compiled test target (empty Unity suite) |

#### Pipeline stages at this milestone

| Stage | Gate |
|-------|------|
| Docker build | Image builds successfully |
| CMake configure | All three targets configure without error |
| cppcheck static analysis | Zero errors (warnings permitted at this stage) |
| Build — all targets | `parksense_node.elf` + `parksense_gateway.elf` + `parksense_boot.elf` compile, zero warnings |
| Unit tests | Empty Unity suite runs and exits 0 |

#### Proof of completion

- `git push` to `main` triggers the pipeline automatically
- All stages pass green on GitHub Actions
- Build artefacts (`.elf` + `.map`) are uploaded as pipeline artefacts

---

### M3 — Bootloader + Image Tools

**Depends on:** M1 (HAL, flash map finalised), M0 (5.08 + 5.09 docs complete)
**Blocks:** M10 (OTA slot swap), M11 (signed release artefacts)

Implement the secure bootloader and the host-side image processing tools. From this point, every firmware image carries a validated header and signature.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/bootloader/btl_validate.c` | CRC-32 verification + ECDSA P-256 signature check |
| `firmware/src/bootloader/btl_jump.c` | MPU lockdown + jump to application |
| `firmware/src/bootloader/btl.h` | Bootloader API |
| `tools/inject_header.py` | Prepends 256-byte `ps_image_header_t` to `.bin` |
| `tools/sign_image.py` | ECDSA P-256 signing — appends or embeds signature |
| `tools/keygen.py` | Generates ECDSA P-256 key pair (run once, private key never committed) |

#### Tasks

1. Implement `ps_image_header_t` exactly as defined in [[5.09-module-headers-and-metadata]]
2. Implement CRC-32 over application image; compare against header field
3. Implement ECDSA P-256 signature verification using STM32U585 PKA hardware accelerator
4. Implement slot selection: boot from primary slot; fall back to secondary if CRC fails
5. Implement jump: set MSP from app vector table, branch to reset handler
6. Write `inject_header.py`: parse `.bin`, build header struct, prepend
7. Write `sign_image.py`: compute ECDSA P-256 signature over header+image, embed in header
8. Validate end-to-end: tool-signed `.bin` → flashed → bootloader verifies → jumps to LED blink app
9. Validate rejection: corrupt one byte of image → bootloader halts, does not jump

#### Proof of completion

- Bootloader verifies a correctly signed image and jumps to the LED blink app from M1
- Bootloader rejects a corrupted image (no jump; fault logged)
- `inject_header.py` + `sign_image.py` produce a correctly structured binary verified by bootloader
- Unit tests cover CRC computation and header parsing with mock flash reads

---

### M4 — Sensor Drivers

**Depends on:** M1 (HAL I2C configured)
**Blocks:** M5

Implement VL53L5CX ToF driver and IIS2MDCTR magnetometer driver behind the `sensor_api.h` abstraction. Both drivers adapted from the STM32CubeU5 BSP/B-U585I-IOT02A reference.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/drivers/sensor/sensor_api.h` | Common sensor interface: `SENSOR_Init()`, `SENSOR_Read()` |
| `firmware/src/drivers/sensor/tof/tof_driver.c/.h` | VL53L5CX ULD — 8×8 ranging, returns `VL53L5CX_ResultsData` |
| `firmware/src/drivers/sensor/magnetometer/mag_driver.c/.h` | IIS2MDCTR — continuous mode, returns 3-axis mGauss |
| `firmware/test/test_sensor_drivers.c` | Unit tests with mocked I2C HAL |

#### Tasks

1. Bring STM32CubeU5 VL53L5CX ULD into CMake (selective compile from `libs/STM32CubeU5/`)
2. Implement ToF driver: configure 8×8 mode, `tof_driver_init()`, `tof_driver_read()`
3. Implement magnetometer driver: configure continuous mode, `mag_driver_init()`, `mag_driver_read()`
4. Expose both behind `sensor_api.h` — PDM only calls the abstract interface
5. Print raw sensor readings over SWO to validate on dev kit
6. Write unit tests with mocked `HAL_I2C_Master_Transmit` / `HAL_I2C_Master_Receive`

#### Proof of completion

- Both sensors initialise without HAL error on dev kit
- ToF readings change when hand placed 5–30 cm above sensor
- Magnetometer readings (mGauss) change when magnet brought near sensor
- Unit tests pass on host (no hardware required)

---

### M5 — Parking Detection Module (PDM)

**Depends on:** M4 (sensor drivers deliver `sensor_api.h`)
**Blocks:** M8 (CPM consumes `pdm_state_t`)

Implement the occupancy state machine and sensor fusion logic as defined in [[5.13-parking-detection-design]].

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/pdm/pdm.c/.h` | PDM public interface: `PDM_Init()`, `PDM_Update()`, `PDM_GetOccupancy()` |
| `firmware/src/pdm/pdm_tof.c` | ToF zone processing, min-distance extraction |
| `firmware/src/pdm/pdm_mag.c` | Magnetometer delta computation vs calibration baseline |
| `firmware/src/pdm/pdm_fsm.c` | FSM: FREE / DEBOUNCE / OCCUPIED with hysteresis counters |
| `firmware/src/pdm/pdm_calibration.c` | Baseline calibration on first boot |
| `firmware/test/test_pdm.c` | Unit tests — FSM transitions with injected sensor frames |

#### Tasks

1. Implement `pdm_tof.c`: average 8×8 zones, compare minimum distance to `VEHICLE_PRESENCE_THRESHOLD_MM`
2. Implement `pdm_mag.c`: compute magnitude delta from baseline, compare to `MAGNETIC_DISTURBANCE_THRESHOLD_MG`
3. Implement `pdm_fsm.c`: N_ENTER consecutive triggers → OCCUPIED; M_EXIT consecutive clears → FREE; DEBOUNCE state absorbs transients
4. Implement `pdm_calibration.c`: record magnetometer baseline from N samples at boot
5. Write unit tests: drive FSM through all state transitions with synthetic sensor frames; verify hysteresis counts

#### Proof of completion

- Placing hand over sensor → `PDM_GetOccupancy()` returns `OCCUPIED` within 1 s
- Removing hand → returns `FREE` after M_EXIT clear samples
- Unit tests pass all FSM transitions and edge cases on host

---

### M6 — RF Drivers

**Depends on:** M1 (HAL IPCC / SPI configured)
**Blocks:** M8

Implement the Zigbee Sleepy End Device (Node) and Coordinator (Gateway) drivers behind `rf_api.h` as defined in [[5.11-rf-driver-api]].

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/drivers/rf/rf_api.h` | Common RF interface: `RF_Init()`, `RF_JoinNetwork()`, `RF_Send()`, `RF_SetRxCallback()` |
| `firmware/src/drivers/rf/rf_module/rf_driver.c/.h` | SED mode (Node): join, TX, RX callback |
| `firmware/src/drivers/rf/rf_coordinator.c` | Coordinator mode (Gateway): PAN start, join window, RX dispatch |
| `firmware/src/drivers/rf/rf_zigbee_config.c` | Channel, PAN ID, TX power, security config |
| `firmware/src/drivers/rf/rf_ipcc.c` | IPCC mailbox wrappers for STM32WB copro communication |
| `firmware/test/test_rf_driver.c` | Unit tests with mocked IPCC layer |

#### Tasks

1. Flash STM32WB5MMG copro with correct Zigbee 3.0 binary from STM32CubeWB
2. Implement IPCC mailbox wrappers (`rf_ipcc.c`)
3. Implement coordinator: `RF_InitCoordinator()`, open join window, RX dispatch callback
4. Implement SED: `RF_JoinNetwork()` — scan, associate, install Install Code → Link Key
5. Configure Zigbee channel 25 or 26 (minimising WiFi 2.4 GHz overlap)
6. Write unit tests with mocked IPCC for packet serialisation / deserialisation paths

#### Proof of completion

- Node joins the gateway PAN (both on same dev kit boards)
- Gateway `RF_RxCallback` fires with correct source address when node transmits a raw payload
- Channel assignment confirmed via Zigbee sniffer or UART log
- Unit tests pass on host

---

### M7 — Network Driver (Gateway)

**Depends on:** M1 (HAL SPI2 configured for EMW3080B)
**Blocks:** M8

Implement `net_api.h` with the Phase 1 WiFi implementation as defined in [[5.20-network-driver-api]].

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/drivers/net/net_api.h` | Abstract interface: `NET_Init()`, `NET_Connect()`, `NET_Send()`, `NET_Recv()`, `NET_Disconnect()` |
| `firmware/src/drivers/net/wifi/wifi_driver.c/.h` | EMW3080B via STM32_Network_Library |
| `firmware/test/mocks/mock_net_api.c` | CMock-generated mock for CPM unit tests |

#### Tasks

1. Pull STM32_Network_Library from STM32CubeU5 into CMake (gateway target only)
2. Implement `NET_Init()`: `MX_WIFI_Init()` → associate to AP using provisioned SSID/password
3. Implement `NET_Connect()`: open TLS socket, validate server CA certificate
4. Implement `NET_Send()` / `NET_Recv()`: delegate to `MX_WIFI_Socket_send()` / `recv()`
5. Implement `NET_Disconnect()`: close socket cleanly
6. Verify TLS handshake succeeds against a test HTTPS server (`curl` on laptop)
7. Generate CMock mock from `net_api.h` for CPM unit testing

#### Proof of completion

- Gateway sends `POST /api/v1/events/test` with a JSON body to a local HTTPS test server
- Server responds HTTP 200; gateway logs success over SWO
- TLS certificate rejection: swap cert → connection refused as expected
- `mock_net_api.c` generated and used in CPM unit tests (M8)

---

### M8 — Communication Protocol Module (CPM)

**Depends on:** M6 (RF drivers deliver `rf_api.h`), M7 (net driver delivers `net_api.h` mock)
**Blocks:** M9

Implement CPM as defined in [[5.10-communication-protocol-module]]: packet assembly, AES-128-CCM encryption, CRC-16, ACK/retry, replay protection, and gateway server forwarding.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/cpm/cpm.c/.h` | CPM core: init, TX state machine, RX callback dispatch |
| `firmware/src/cpm/cpm_serialize.c` | Packet pack / unpack (`cpm_packet_t`) |
| `firmware/src/cpm/cpm_crc.c` | CRC-16/CCITT-FALSE computation and validation |
| `firmware/src/cpm/cpm_queue.c` | Circular TX queue |
| `firmware/src/cpm/cpm_retry.c` | Retry timer, exponential backoff |
| `firmware/src/cpm/cpm_server.c` | Gateway-only: server TX queue, JSON serialisation, `CPM_Server_Tick()` |
| `firmware/test/test_cpm.c` | Unit tests — all paths exercised with mocked `rf_api.h` and `net_api.h` |

#### Tasks

1. Implement `cpm_packet_t` serialisation / deserialisation per [[5.05-data-structures]]
2. Implement CRC-16/CCITT-FALSE; test against known vectors
3. Implement AES-128-CCM encrypt/decrypt using STM32U585 AES hardware accelerator
4. Implement TX state machine: IDLE → SENDING → WAIT_ACK → (RETRY | DONE)
5. Implement replay protection: sequence number window per node ID
6. Implement `cpm_server.c`: server TX queue (depth 16), `CPM_Server_Tick()`, JSON format, retry ≤ 3×
7. Write unit tests: serialise → transmit (mock RF) → receive → deserialise; gateway forward path; retry exhaustion; replay rejection

#### Proof of completion

- Unit tests pass 100% on host with mocked `rf_api.h` and `net_api.h`
- On dev kit: node sends a CPM packet → gateway receives, decrypts, CRC valid, ACK sent back to node
- Replay attack test: duplicate packet rejected
- Retry test: RF TX fails twice, third succeeds

---

### M9 — Integration Test

**Depends on:** M8 (CPM complete), M5 (PDM complete)
**Blocks:** M10

End-to-end hardware validation before the application layer is assembled. This milestone de-risks the app layer by confirming the full data path works on real hardware.

#### Test scenarios

| # | Scenario | Pass Criteria |
|---|----------|---------------|
| 1 | Node detects vehicle → CPM packet → gateway → server POST | Server receives JSON event; HTTP 200 returned |
| 2 | Node sends heartbeat every 30 s | 3 consecutive heartbeats received by server; timestamps plausible |
| 3 | Gateway RF RX fails (node out of range) | CPM retry fires; after exhaustion, fault logged; gateway continues |
| 4 | Gateway WiFi drops mid-session | Server TX queue fills (≤ 16 events buffered); reconnects; queue flushes |
| 5 | New node joins the PAN | Gateway RF join window accepts it; subsequent packets routed correctly |
| 6 | Corrupted packet (bit flip in CRC) | CPM discards packet; NACK sent; node retries; second attempt succeeds |

#### Proof of completion

All 6 scenarios pass with results logged over SWO. No assertion failures, no watchdog resets during the 30-minute unattended soak run.

---

### M10 — Application Layer

**Depends on:** M9 (full chain validated)
**Blocks:** M11

Assemble the final super loop for Node and Gateway; integrate watchdog and power management.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/targets/node/main_node.c` | Node super loop: Init → Wake → PDM_Update → CPM_Send → Sleep |
| `firmware/targets/gateway/main_gw.c` | Gateway super loop: Init → RF_RX → CPM_Forward → NET_Tick |
| `firmware/src/bsp/bsp.c` (update) | IWDG init + pet in super loop; STOP2 sleep entry/exit |

#### Tasks

1. Implement full node super loop per [[5.15-application-design]] timing budget
2. Implement full gateway super loop
3. Implement IWDG with 4 s timeout; pet at top of each loop iteration
4. Implement STOP2 sleep entry on node; RTC alarm wake per `SLEEP_INTERVAL_S`

#### Proof of completion

- Node runs autonomously for 24 hours; no watchdog resets; occupancy changes logged on server
- Gateway runs autonomously for 24 hours; server receives all events; no dropped packets

---

### M11 — CI/CD Full Pipeline

**Depends on:** M10 (signed release images + tools complete)
**Blocks:** M12 (reproducible artefacts for server)

Extend the M2 pipeline with image signing, version management, changelog generation, and GitHub Release publishing.

#### Additional pipeline stages

| Stage | Tool | Gate |
|-------|------|------|
| Image processing | `inject_header.py` + `sign_image.py` | Signed `.bin` produced for all three targets |
| Version bump | Conventional Commits analysis | `fix:` → PATCH, `feat:` → MINOR, `BREAKING CHANGE:` → MAJOR |
| Changelog | `gen_changelog.py` | `CHANGELOG.md` generated from commit history between tags |
| GitHub Release | GitHub Actions | Tag created; signed `.bin` artefacts attached |

#### Proof of completion

- `git push` with `feat:` prefix → MINOR version bump → signed artefacts on GitHub Release page
- `CHANGELOG.md` updated automatically
- Downloaded `.bin` from GitHub Release → flashed → bootloader validates → boots

---

### M12 — Server & UI

**Depends on:** M9 (integration test confirms gateway → server POST format), M11 (CI/CD artefacts)
**Blocks:** M13 (OTA requires server to deliver images to the gateway)

Implement the Python Flask backend and React dashboard as defined in [[5.16-server-design]], [[5.17-database-design]], and [[5.18-gui-design]].

#### Deliverables

| Component | Description |
|-----------|-------------|
| `server/` | Flask REST API: `/api/v1/events/occupancy` POST, WebSocket broadcast |
| `server/db/` | SQLite or PostgreSQL schema: spaces, events, gateway registrations |
| `gui/` | React dashboard: real-time lot map, occupancy status per space |

#### Proof of completion

- Dashboard shows live occupancy state updated within 2 s of physical detection
- Historical event query returns correct occupancy timeline for a given space ID
- Gateway token authentication enforced: unauthenticated POST returns HTTP 401

---

### M13 — OTA

**Depends on:** M10 (application layer running end-to-end), M3 (bootloader + slot mechanism), M12 (server can push images to gateway)
**Blocks:** Nothing (final milestone)

Implement over-the-air firmware update: server delivers a signed image to the gateway, gateway writes it to the secondary flash slot, bootloader validates and swaps. This milestone is last because it exercises the entire stack — server delivery, RF relay, flash write, bootloader validation, and application recovery — and has no value until everything beneath it is stable.

#### Deliverables

| File | Description |
|------|-------------|
| `firmware/src/ota/ota.c/.h` | OTA receive loop: buffer chunks → write to secondary slot → set boot flag → trigger reset |
| `server/ota/` | Server-side OTA push endpoint: `POST /api/v1/ota/push` — streams signed `.bin` to gateway |

#### Tasks

1. Implement `ota.c`: receive binary chunks from CPM, write sequentially to secondary flash slot using HAL flash write
2. Validate full image CRC before setting boot flag — do not set flag on incomplete transfer
3. Set boot flag in OTA config sector; trigger `NVIC_SystemReset()`
4. Validate: bootloader selects secondary slot on next boot, validates signature, jumps to new app
5. Validate rollback: write corrupt image to secondary slot → bootloader rejects → boots primary
6. Implement server-side OTA push: chunk the signed `.bin` and deliver via existing gateway TCP/TLS connection

#### Proof of completion

- Updated firmware image delivered from server → gateway → secondary flash slot → bootloader validates → new firmware running after one reset
- OTA rollback: corrupt image written to secondary slot → bootloader rejects → primary firmware still running on next boot
- Partial transfer (simulated by cutting power mid-OTA) → CRC check fails → boot flag not set → primary firmware still running

---

## 4. Exit Criteria Summary

| Milestone | Hard Exit Criteria |
|-----------|-------------------|
| M0 | 5.01, 5.02, 5.08, 5.09 — no TBD sections; memory map with exact sector addresses |
| M1 | LED blinks; 3 targets link to correct flash regions; `-Werror` clean |
| M2 | `git push` → all pipeline stages green |
| M3 | Bootloader validates signed image; rejects corrupt image; tools produce correct binary |
| M4 | Both sensors read on hardware; unit tests pass on host |
| M5 | Occupancy state changes correctly on hand test; unit tests pass all FSM transitions |
| M6 | Node joins gateway PAN; gateway RX callback fires; unit tests pass |
| M7 | TLS HTTPS POST to test server succeeds; cert rejection works |
| M8 | CPM unit tests 100%; encrypted node→gateway packet ACK'd on hardware |
| M9 | All 6 integration scenarios pass; 30-minute soak run clean |
| M10 | 24-hour autonomous node + gateway run clean; no watchdog resets |
| M11 | `git push` → signed artefacts on GitHub Release; downloaded `.bin` boots |
| M12 | Dashboard shows live state; auth enforced |
| M13 | OTA image delivered and running; corrupt image rejected; partial transfer safe |

---

## 5. Related Documents

| Document | Relevance |
|----------|-----------|
| [[README]] §11 | High-level milestone roadmap |
| [[5-firmware-architecture-design]] | SDD governing all firmware modules |
| [[5.01-firmware-decomposition]] | File inventory for M1 source tree setup |
| [[5.02-memory-map]] | Flash layout for M1 linker scripts and M3 bootloader |
| [[5.07-build-configuration]] | CMake targets and compile-time constants |
| [[5.08-bootloader-design]] | Bootloader design for M3 |
| [[5.09-module-headers-and-metadata]] | `ps_image_header_t` layout for M3 tools |
| [[1.7-build-and-cicd]] | CI/CD pipeline specification for M2 and M11 |