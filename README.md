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
