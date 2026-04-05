# 8. External References

> **Project:** ParkSense — Full-Stack IoT Parking Occupancy System
> **Date:** 2026-04-03
> **Author:** Arturo Vargas Cuevas
> **↑ Parent:** [[README]]

All external datasheets, user manuals, and tool documentation are linked here. 

---

## 8.1 MCU — STM32U585AII6Q

| Document | Link |
| -------- | ---- |
| STM32 32-bit Arm Cortex MCUs — product family | https://www.st.com/en/microcontrollers-microprocessors/stm32-32-bit-arm-cortex-mcus.html |
| STM32U585AI — product page | https://www.st.com/en/microcontrollers-microprocessors/stm32u585ai.html |
| Product Specification (Datasheet) | https://www.st.com/resource/en/datasheet/stm32u585ai.pdf |
| Reference Manual — STM32U5 Series (RM0456) | https://www.st.com/resource/en/reference_manual/rm0456-stm32u5-series-armbased-32bit-mcus-stmicroelectronics.pdf |

---

## 8.2 Development Board — B-U585I-IOT02A

| Document | Link |
| -------- | ---- |
| B-U585I-IOT02A — product page | https://www.st.com/en/evaluation-tools/b-u585i-iot02a.html |
| Product Specification (Data Brief) | https://www.st.com/resource/en/data_brief/b-u585i-iot02a.pdf |
| Discovery Kit for IoT Node with STM32U5 — product page | https://www.st.com/en/microcontrollers-microprocessors/stm32u585ai.html |
| User Manual (UM2839) | https://www.st.com/resource/en/user_manual/um2839-discovery-kit-for-iot-node-with-stm32u5-series-stmicroelectronics.pdf |

---

## 8.3 RF Module — STM32WB5MMGH6TR

| Document | Link |
| -------- | ---- |
| STM32WB5MMG — product page | https://www.st.com/en/microcontrollers-microprocessors/stm32wb5mmg.html |
| Product Specification (Datasheet) | https://www.st.com/resource/en/datasheet/stm32wb5mmg.pdf |
| User Manual — STM32WB Series Zigbee Cluster Library API (UM2977) | https://www.st.com/resource/en/user_manual/um2977-stm32wb-series-zigbee-cluster-library-api-stmicroelectronics.pdf |
| User Manual — STM32WB Series BLE Low Level Driver (LLD) | https://www.st.com/en/embedded-software/stm32wb-lld.html |
| STM32CubeWB — Zigbee Copro Binaries | https://github.com/STMicroelectronics/STM32CubeWB/tree/master/Projects/STM32WB_Copro_Wireless_Binaries |

---

## 8.4 ToF Sensor — VL53L5CXV0GC/1

| Document | Link |
| -------- | ---- |
| VL53L5CX — product page | https://www.st.com/en/imaging-and-photonics-solutions/vl53l5cx.html |
| Product Specification (Datasheet) | https://www.st.com/resource/en/datasheet/vl53l5cx.pdf |

---

## 8.5 Magnetometer — IIS2MDCTR

| Document | Link |
| -------- | ---- |
| IIS2MDC — product page | https://www.st.com/en/mems-and-sensors/iis2mdc.html |
| Product Specification (Datasheet) | https://www.st.com/resource/en/datasheet/iis2mdc.pdf |

---

## 8.6 Network Module

### WiFi — EMW3080B

| Document | Link |
| -------- | ---- |
| X-WIFI-EMW3080B — product page | https://www.st.com/en/development-tools/x-wifi-emw3080b.html |
| Wi-Fi Firmware Update for MXCHIP EMW3080B on STM32 Boards (Data Brief) | https://www.st.com/resource/en/data_brief/x-wifi-emw3080b.pdf |
| MX-WIFI Component Driver (STM32CubeU5) | https://github.com/STMicroelectronics/STM32CubeU5/tree/main/Drivers/BSP/Components/mx_wifi |
| STM32 Network Library (STM32CubeU5) | https://github.com/STMicroelectronics/STM32CubeU5/tree/main/Middlewares/ST/STM32_Network_Library |

### Ethernet — WIZnet W5500

| Document | Link |
| -------- | ---- |
| W5500 Datasheet | https://docs.wiznet.io/Product/iEthernet/W5500/datasheet |
| WIZnet ioLibrary Driver (STM32 HAL) | https://github.com/Wiznet/ioLibrary_Driver |
| W5500 Hardware Design Guide | https://docs.wiznet.io/Product/iEthernet/W5500/design-guide |

---

## 8.7 Vendor SDK — STM32CubeU5

| Document | Link |
| -------- | ---- |
| STM32CubeU5 GitHub repository (submodule) | https://github.com/STMicroelectronics/STM32CubeU5 |
| STM32U5 HAL and LL Drivers — User Manual (UM2911) | https://www.st.com/resource/en/user_manual/um2911-description-of-stm32u5-hal-and-lowlayer-drivers-stmicroelectronics.pdf |

---

## 8.8 Build Tools & Compiler

| Tool | Version Used | Link |
| ---- | ------------ | ---- |
| Arm GNU Toolchain (arm-none-eabi-gcc) | 13.x | https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads |
| CMake | ≥ 3.25 | https://cmake.org/download/ |
| Ninja | latest | https://ninja-build.org |
| Docker | latest | https://www.docker.com/products/docker-desktop |

---

## 8.9 Test Frameworks

| Tool | Link |
| ---- | ---- |
| Unity (unit test framework) | https://github.com/ThrowTheSwitch/Unity |
| CMock (mock generator) | https://github.com/ThrowTheSwitch/CMock |

---

## 8.10 Static Analysis

| Tool | Link |
| ---- | ---- |
| Cppcheck | https://cppcheck.sourceforge.io |
| Cppcheck Manual | https://cppcheck.sourceforge.io/manual.pdf |

---

## 8.11 Standards Referenced

| Standard | Title | Used In |
| -------- | ----- | ------- |
| IEEE 29148-2018 | Systems and Software Engineering — System Requirements Specification | docs/2-system-requirements-specification/ |
| IEEE 42010:2011 | Systems and Software Engineering — Architecture Description | docs/4-system-architecture-design/ |
| IEEE 1016-2009 | Software Design Descriptions | docs/5-firmware-architecture-design/ |
| ISO/IEC/IEEE 12207:2017 | Software Lifecycle Processes | docs/5-firmware-architecture-design/ |
| MISRA-C:2012 | Guidelines for the Use of the C Language in Critical Systems | docs/1-development-guidelines/1.4-coding-standards.md |
| CMSIS 5 | ARM Cortex Microcontroller Software Interface Standard | firmware/libs/STM32CubeU5/CMSIS/ |
| Zigbee 3.0 Specification | ZigBee Alliance — Core specification | docs/3-hardware-selection/3.2-rf-wireless-module-selection.md |
| IEEE 802.15.4-2015 | Low-Rate Wireless Networks (MAC/PHY for Zigbee) | docs/5-firmware-architecture-design/5.11-rf-driver-api.md |
