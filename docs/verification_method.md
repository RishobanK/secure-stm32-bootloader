# Verification Method

Initial validation was performed without a dedicated USB-to-TTL adapter.

Instead, the project was verified using:

- LED-coded status indicators
- ST-Link debugger breakpoints
- debugger watch variables
- STM32CubeProgrammer flash memory inspection

UART console testing was later performed using an ESP32-C3 Super Mini configured as a temporary USB-to-UART bridge.

---

## LED Status Indicators

| LED Pattern | Meaning |
|---|---|
| Fast blink | Bootloader running |
| Slow blink | Application running |
| 1 blink + pause | Magic validation failed |
| 2 blinks + pause | Reset handler validation failed |
| 3 blinks + pause | Application size validation failed |
| 4 blinks + pause | CRC validation failed |

These LED patterns allowed the bootloader state and validation failures to be observed without a serial console.

---

## Debugger-Based Verification

The following were verified using ST-Link debugging in STM32CubeIDE:

- bootloader startup
- application validation function execution
- `JumpToApplication()` execution
- application `main()` reached after bootloader jump

## Watch Variables

The following variables were inspected during debugging:

- `magic`
- `size`
- `crc`
- `reset_handler`
- validation return code
- application stack pointer
- application reset handler

## Flash Memory Verification

STM32CubeProgrammer was used to inspect flash memory and confirm the expected layout.

| Region | Address |
|---|---:|
| Bootloader | `0x08000000` |
| Application header | `0x08004000` |
| Application code | `0x08004400` |

## ESP32-C3 UART Bridge

A dedicated USB-to-TTL adapter was not initially available. An ESP32-C3 Super Mini was configured as a temporary USB-to-UART bridge.

Connection used:

| ESP32-C3 Super Mini | STM32 Blue Pill |
|---|---|
| GPIO21 / TX | PA10 / USART1_RX |
| GPIO20 / RX | PA9 / USART1_TX |
| GND | GND |

The ESP32-C3 bridge was verified using a loopback test by connecting GPIO21 to GPIO20 and confirming that typed serial data was echoed back in the serial monitor.
