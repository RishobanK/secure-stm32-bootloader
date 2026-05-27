# Bootloader Firmware

This folder contains the STM32 custom bootloader firmware.

The bootloader is located at `0x08000000` and is responsible for:

- starting first after reset
- checking the update request flag
- validating the application header
- checking firmware size and CRC32
- performing firmware update using flash erase/write operations
- jumping to the relocated application firmware at `0x08004400`

The bootloader was developed using STM32CubeIDE for the STM32 Blue Pill board.
