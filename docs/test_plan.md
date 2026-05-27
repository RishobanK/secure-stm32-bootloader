# Test Plan

This document summarizes the test cases used to verify the STM32 bootloader project.

| Test Case | Verification Method | Expected Result |
|---|---|---|
| Bootloader startup | LED pattern + debugger breakpoint | Bootloader starts from `0x08000000` |
| Application jump | LED changes from fast blink to slow blink | Application runs from `0x08004400` |
| Missing application | LED error pattern | Bootloader does not jump |
| Invalid magic number | 1 blink + pause | Bootloader rejects application |
| Invalid reset handler | 2 blinks + pause | Bootloader rejects application |
| Invalid size | 3 blinks + pause | Bootloader rejects application |
| CRC mismatch | 4 blinks + pause | Bootloader rejects application |
| Flash layout | STM32CubeProgrammer memory view | Header and application are located at correct addresses |
| Update flag set | LED update-mode indication | Bootloader enters firmware update path |
| Successful firmware update | LED/application behavior + flash inspection | New application is written and executed |

## Notes

The project was initially verified without a dedicated USB-to-TTL adapter.

LED status indicators, ST-Link debugging, watch variables, and flash memory inspection were used to confirm the behavior of each bootloader stage.

UART console testing was later performed using an ESP32-C3 Super Mini as a temporary USB-to-UART bridge.
