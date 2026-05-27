# Flash Memory Map

This project uses a separated flash memory layout for the bootloader, application header, and application firmware.

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

## Reason for This Layout

The STM32 starts execution from the internal flash base address `0x08000000` after reset. Therefore, the custom bootloader is placed at this address.

The application firmware is relocated to `0x08004400` so that it does not overwrite the bootloader or application header region.

The application header is kept at `0x08004000` so that the bootloader can always read firmware metadata from a fixed address before deciding whether to run the application.

## Address Definitions

```c
#define BL_START_ADDR      0x08000000
#define APP_HEADER_ADDR    0x08004000
#define APP_START_ADDR     0x08004400
```

## Region Purpose

### Bootloader Region

The bootloader is responsible for:

- starting first after reset
- checking update request flag
- validating application firmware
- performing firmware update
- jumping to the application if validation passes

### Application Header Region

The header stores firmware metadata:

- update flag
- magic number
- application size
- CRC32
- version

### Application Firmware Region

The actual application code starts at `0x08004400`. The application linker script is modified so that the application is built for this relocated address.
