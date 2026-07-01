# Secure STM32 Bootloader & Firmware Update System

A custom bootloader implementation for the STM32 Blue Pill, developed to understand embedded bootloader design, flash memory partitioning, relocated application execution, application validation, CRC32-based firmware integrity checking, update flag handling, and firmware update workflow.

This project was developed as a pure embedded firmware learning project focused on ARM Cortex-M boot flow, memory layout, flash programming, validation logic, and firmware update mechanisms.

---

## Overview

The STM32 starts execution from the internal flash base address `0x08000000`. In this project, that region is reserved for a custom bootloader. The main application is relocated to a later flash address so that the bootloader can run first, validate the application, and then jump to it.

The bootloader performs the following main tasks:

1. Starts from `0x08000000`
2. Checks whether a firmware update is requested
3. Validates the application header
4. Checks application size and CRC32
5. Performs firmware update when required
6. Jumps to the relocated application firmware if validation passes

---

## Key Features

- Custom STM32 bootloader located at `0x08000000`
- Relocated application firmware starting at `0x08004400`
- Reserved application header region at `0x08004000`
- Bootloader-to-application jump using stack pointer and reset handler
- Application validation using magic number and reset handler sanity check
- Application size validation
- CRC32-based firmware integrity verification
- Flash-based update request flag mechanism
- Full firmware update workflow using flash erase/write operations
- Application header update after successful firmware flashing
- LED-based bootloader, application, and error-state indication
- ST-Link debugging and STM32CubeProgrammer memory verification
- ESP32-C3 Super Mini used as a temporary USB-to-UART bridge for UART console testing

---

## Hardware Used

- STM32 Blue Pill board, STM32F103C8T6
- ST-Link V2 programmer/debugger
- ESP32-C3 Super Mini used as temporary USB-to-UART bridge
- PC running STM32CubeIDE and STM32CubeProgrammer

---

## Software Tools

- STM32CubeIDE
- STM32CubeProgrammer
- ARM GCC toolchain
- Python 3
- ST-Link debugger
- Git and GitHub

---

## Flash Memory Map

The STM32 internal flash is divided into three main regions.

| Region | Start Address | Purpose |
|---|---:|---|
| Bootloader | `0x08000000` | Runs first after reset |
| Application Header | `0x08004000` | Stores update flag, magic, size, CRC, and version |
| Application Firmware | `0x08004400` | Main application code |

```text
0x08000000  ┌────────────────────────┐
            │ Bootloader              │
            │ 16 KB                   │
0x08004000  ├────────────────────────┤
            │ Application Header      │
            │ 1 KB                    │
0x08004400  ├────────────────────────┤
            │ Application Firmware    │
            │                         │
0x08010000  └────────────────────────┘
```

The application is relocated to `0x08004400` to prevent it from overwriting the bootloader region. The bootloader remains at `0x08000000`, because STM32 starts execution from the beginning of internal flash after reset.

---

## Application Header

The application header is stored at `0x08004000`. It contains metadata used by the bootloader before executing the application.

```c
typedef struct {
    uint32_t ota_flag;
    uint32_t magic;
    uint32_t size;
    uint32_t crc;
    uint32_t version;
} app_header_t;
```

| Field | Purpose |
|---|---|
| `ota_flag` | Requests firmware update mode |
| `magic` | Identifies a valid application header |
| `size` | Stores application image size |
| `crc` | Stores expected CRC32 of application firmware |
| `version` | Tracks firmware version |

The bootloader reads this header before jumping to the application. If the header is missing or invalid, the bootloader remains in bootloader/error mode instead of executing invalid firmware.

---

## Boot Flow

```text
Reset
  ↓
Bootloader starts from 0x08000000
  ↓
Check update request flag
  ↓
If update requested:
    perform firmware update
Else:
    validate application header
  ↓
Check magic number
Check reset handler address
Check application size
Check CRC32
  ↓
If valid:
    set application stack pointer
    jump to application reset handler at 0x08004400
Else:
    remain in bootloader error state
```

---

## Bootloader-to-Application Jump

Every ARM Cortex-M application starts with a vector table.

At the application start address:

| Address | Content |
|---|---|
| `APP_START_ADDR` | Initial stack pointer |
| `APP_START_ADDR + 4` | Reset handler address |

The bootloader reads these two values from the application region. It then sets the main stack pointer to the application stack and jumps to the application reset handler.

```c
appStack = *(volatile uint32_t*)APP_START_ADDR;
appResetHandler = *(volatile uint32_t*)(APP_START_ADDR + 4);

__disable_irq();

SysTick->CTRL = 0;
SysTick->LOAD = 0;
SysTick->VAL  = 0;

__set_MSP(appStack);
appEntry();
```

This transfers control from the bootloader firmware to the relocated application firmware.

---

## Firmware Update Flow

The firmware update process is triggered by a flash-based update request flag.

```text
Update flag detected
  ↓
Read prepared firmware image
  ↓
Validate image magic and size
  ↓
Erase existing application region
  ↓
Write new firmware chunks to flash
  ↓
Calculate CRC32 during/after write
  ↓
Compare calculated CRC with expected CRC
  ↓
Update application header
  ↓
Clear update flag
  ↓
Validate and jump to new application
```

The update process allows the bootloader to replace the existing application firmware with a new firmware image while keeping the bootloader region protected.

---

## Validation Mechanism

The bootloader validates the application before jumping to it.

Validation checks include:

- Magic number check
- Reset handler address sanity check
- Application size check
- CRC32 integrity check

### Magic Number Check

The magic number confirms that the application header exists and is valid.

If the magic number does not match the expected value, the bootloader rejects the application.

### Reset Handler Check

The bootloader reads the reset handler address from:

```text
APP_START_ADDR + 4
```

The reset handler must point to a valid flash address. If it is outside the expected flash region, the bootloader rejects the application.

### Size Check

The application size must be:

- greater than zero
- smaller than the available application flash region

This prevents the bootloader from validating or writing beyond the allocated application space.

### CRC32 Check

CRC32 is used to detect accidental corruption or incomplete firmware writes.

The bootloader calculates CRC32 over the application firmware region and compares it with the CRC stored in the application header.

```text
Expected CRC from header
        ↓
Compare
        ↑
Calculated CRC from flash
```

If both values match, the firmware is considered intact. If they do not match, the firmware is rejected.

> Note: CRC32 is an integrity check, not cryptographic authentication. It can detect accidental corruption, but it does not prove that the firmware came from an authorized source. Authenticated firmware validation using HMAC-SHA256 or digital signatures is planned as a future improvement.

---
## Security Notes

This project currently uses CRC32 for firmware integrity checking. CRC32 is useful for detecting accidental corruption or incomplete flash writes, but it is not a cryptographic security mechanism. It does not prove that the firmware came from an authorized source.

Planned security improvements include:

* HMAC-SHA256 based firmware authentication
* Digital signature verification using public-key cryptography
* Firmware version checking
* Anti-rollback protection
* Application health confirmation after update

The goal of the project is to gradually evolve from basic firmware integrity checking toward authenticated and secure firmware update concepts.

---

## Update Flag Mechanism

The update flag allows the application to request firmware update mode.

The flag is stored in flash memory inside the application header.

### Why Flash Is Used

RAM is cleared after reset. Flash memory is non-volatile, so the flag remains available after the MCU resets.

This allows the application to set an update request and then reset the MCU.

```text
Application running
  ↓
Update requested
  ↓
Application sets update flag in flash
  ↓
Application resets MCU
  ↓
Bootloader starts
  ↓
Bootloader detects update flag
  ↓
Bootloader enters firmware update process
```

After detecting the flag, the bootloader clears it to prevent repeatedly entering update mode on every reset.

---

## Verification Method

Initial validation was performed without a dedicated USB-to-TTL adapter. Bootloader behavior was verified using LED-coded status indicators, ST-Link breakpoints, watch variables, and STM32CubeProgrammer flash memory inspection.

UART console testing was later performed using an ESP32-C3 Super Mini configured as a temporary USB-to-UART bridge.

### LED Status Indicators

| LED Pattern | Meaning |
|---|---|
| Fast blink | Bootloader running |
| Slow blink | Application running |
| 1 blink + pause | Magic validation failed |
| 2 blinks + pause | Reset handler validation failed |
| 3 blinks + pause | Application size validation failed |
| 4 blinks + pause | CRC validation failed |

### Debugger-Based Verification

The following were verified using ST-Link debugging in STM32CubeIDE:

- Breakpoint hit inside bootloader startup
- Breakpoint hit inside application validation function
- Breakpoint hit inside `JumpToApplication()`
- Application `main()` reached after bootloader jump

Watch variables inspected:

- `magic`
- `size`
- `crc`
- `reset_handler`
- validation return code
- application stack pointer
- application reset handler

### Flash Memory Verification

STM32CubeProgrammer was used to inspect flash memory and confirm the expected memory layout.

| Region | Address |
|---|---:|
| Bootloader | `0x08000000` |
| Application header | `0x08004000` |
| Application code | `0x08004400` |

---

## ESP32-C3 UART Bridge

A dedicated USB-to-TTL converter was not initially available. For UART console testing, an ESP32-C3 Super Mini was configured as a temporary USB-to-UART bridge.

Connection used:

| ESP32-C3 Super Mini | STM32 Blue Pill |
|---|---|
| GPIO21 / TX | PA10 / USART1_RX |
| GPIO20 / RX | PA9 / USART1_TX |
| GND | GND |

The ESP32-C3 bridge was first verified using a loopback test by connecting GPIO21 to GPIO20 and confirming that typed serial data was echoed back in the serial monitor.

---

## Repository Structure

```text
secure-stm32-bootloader/
│
├── bootloader/       # STM32 custom bootloader firmware
├── application/      # Relocated STM32 application firmware
├── tools/            # Python scripts for firmware image generation
├── docs/             # Technical documentation
└── media/            # Screenshots, demo images, and videos
```

---

## Build and Flash Notes

### Bootloader

The bootloader is built as a separate STM32CubeIDE project and placed at the default flash start address:

```text
0x08000000
```

The bootloader linker script limits the bootloader region to the first 16 KB of flash.

### Application

The application is built as a separate STM32CubeIDE project and relocated to:

```text
0x08004400
```

The application linker script is modified so that the application code starts after the bootloader and application header region.

### Application Header

The application header is placed at:

```text
0x08004000
```

This is done using a custom linker section so that the bootloader can always find the header at a fixed address.

---

## Current Status

Completed up to full bootloader firmware update workflow:

- Bootloader jump to application
- Application validation
- Size and CRC32 validation
- Flash-based update flag mechanism
- Full firmware update process using flash erase/write
- LED/debugger/memory-based verification
- ESP32-C3 UART bridge validation for serial console testing

---

## Test Plan Summary

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

---

## Planned Improvements

The following improvements are planned for future development:

### UART-Based Firmware Transfer from PC

The current update workflow uses a prepared firmware image. A future improvement is to transfer firmware from a PC to the bootloader over UART using a Python firmware sender.

Planned features:

- Python firmware sender tool
- UART packet protocol
- ACK/NACK response handling
- Chunk-based firmware transfer
- Timeout and retry mechanism
- CRC verification after transfer

### OTA Update via TCP Server

A future extension is to support firmware update from a TCP server.

Planned flow:

```text
Bootloader enters update mode
  ↓
Connect to TCP server through network interface
  ↓
Receive firmware image in chunks
  ↓
Write chunks to flash
  ↓
Verify CRC
  ↓
Update application header
  ↓
Boot new application
```

This will demonstrate network-based firmware update concepts.

### Wireless Update Using ESP8266 and External SPI Flash

Another planned extension is wireless firmware update using an ESP8266 module and external SPI flash.

Planned architecture:

```text
ESP8266 downloads firmware over Wi-Fi
  ↓
Firmware stored in external SPI flash
  ↓
STM32 bootloader reads firmware from SPI flash
  ↓
STM32 verifies and flashes new application
```

This separates wireless communication from the STM32 bootloader and keeps the STM32 firmware focused on validation and flashing.

### HMAC-SHA256 Based Firmware Authentication

CRC32 detects accidental corruption, but it does not prove that firmware is authorized.

A future improvement is to add HMAC-SHA256 based authentication.

Planned flow:

```text
Python tool calculates HMAC using secret key
  ↓
Firmware package includes HMAC
  ↓
Bootloader calculates HMAC over firmware
  ↓
Bootloader compares expected and calculated HMAC
  ↓
Firmware is accepted only if HMAC matches
```

This will demonstrate authenticated firmware validation.

### Digital Signature Verification

A stronger future security improvement is digital signature verification using public-key cryptography.

This would allow the bootloader to verify firmware authenticity without storing a shared secret key in firmware.

Planned concept:

```text
Firmware signed using private key
  ↓
Bootloader verifies signature using public key
  ↓
Only correctly signed firmware is accepted
```

### Version Checking and Anti-Rollback

Version checking will prevent older firmware from replacing newer firmware.

Example:

```text
Installed version: 3
Incoming version: 2
Result: rejected
```

This prevents accidental downgrade and helps protect against rollback to vulnerable firmware.

### Application Health Check and Rollback

A future improvement is to support application health confirmation.

Planned flow:

```text
New firmware installed
  ↓
Bootloader marks app as PENDING_VERIFY
  ↓
Application boots and confirms health
  ↓
Bootloader marks app as VALID
```

If the application does not confirm successful boot, the bootloader can remain in update mode or roll back to a previous valid image.

### Improved UART Logging

UART logging will be improved using either:

- dedicated USB-to-TTL adapter
- ESP32-C3 USB-to-UART bridge
- ST-Link Virtual COM Port, if available

This will improve runtime diagnostics and make validation easier to observe.

### Demo Video and Debugger Screenshots

Planned documentation improvements:

- bootloader-to-application jump demo
- validation failure LED patterns
- flash memory layout screenshots
- debugger watch window screenshots
- firmware update workflow demonstration
