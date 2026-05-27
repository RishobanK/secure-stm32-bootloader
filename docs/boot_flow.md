# Boot Flow

The bootloader always starts first after reset because it is located at `0x08000000`, which is the default STM32 flash start address.

## Overall Boot Flow

```text
Reset
  ↓
Bootloader starts from 0x08000000
  ↓
Read application header at 0x08004000
  ↓
Check update request flag
  ↓
If update requested:
    run firmware update process
Else:
    validate application
  ↓
If application valid:
    jump to application at 0x08004400
Else:
    stay in bootloader/error state
```

## Bootloader Responsibilities

The bootloader performs these tasks:

1. Starts after reset
2. Checks whether firmware update is requested
3. Validates the application header
4. Checks application size and CRC32
5. Performs update if needed
6. Jumps to the application if validation passes

## Bootloader-to-Application Jump

Every ARM Cortex-M application starts with a vector table.

At the application start address:

| Address | Content |
|---|---|
| `APP_START_ADDR` | Initial stack pointer |
| `APP_START_ADDR + 4` | Reset handler address |

The bootloader reads both values:

```c
appStack = *(volatile uint32_t*)APP_START_ADDR;
appResetHandler = *(volatile uint32_t*)(APP_START_ADDR + 4);
```

Then it sets the Main Stack Pointer and jumps to the application reset handler.

```c
__set_MSP(appStack);
appEntry();
```

## Why This Is Needed

The bootloader and application are two separate firmware images. The bootloader cannot simply call `main()` of the application. It must transfer control in the same way the CPU normally starts an application:

1. load the application stack pointer
2. jump to the application reset handler

This allows the relocated application firmware to start correctly.
