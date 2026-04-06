# Zigbee M0+ Co-processor Firmware Binaries

Pre-compiled ST binary images for the STM32WB5MMGH6TR M0+ radio co-processor.
Flash via STM32CubeProgrammer before the WB5MM M4 relay firmware runs.

| Field | Value |
|-------|-------|
| Source repo | https://github.com/STMicroelectronics/STM32CubeWB.git |
| Source path | `Projects/STM32WB_Copro_Wireless_Binaries/STM32WB5x/` |
| Source commit | `caf4df647a95c4a21f0d788c3857c96954c563cf` |
| Vendor-copy date | 2026-04-05 |

## Binary selection

| File | Target | Description |
|------|--------|-------------|
| `stm32wb5x_Zigbee_FFD_fw.bin` | Gateway | Zigbee FFD — PAN Coordinator mode |
| `stm32wb5x_BLE_Zigbee_RFD_static_fw.bin` | Node | Zigbee RFD — SED mode (BLE unused) |
| `stm32wb5x_FUS_fw.bin` | Bring-up only | FUS update binary; flash once before first stack flash |

## Flash procedure (once per chip)

1. Connect STM32CubeProgrammer via SWD to the WB5MM.
2. Flash FUS if version < 1.2.0: `stm32wb5x_FUS_fw.bin` at `0x080EC000`.
3. Flash stack binary at address from Release Notes.
4. Flash WB5MM M4 relay firmware from `firmware/targets/wb_relay/`.
