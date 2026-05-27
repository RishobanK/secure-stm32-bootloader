# Relocated Application Firmware

This folder contains the STM32 application firmware relocated to `0x08004400`.

The application includes an application header located at `0x08004000`, which stores:

- update flag
- magic number
- firmware size
- CRC32
- firmware version

The bootloader reads this header before deciding whether to jump to the application.
