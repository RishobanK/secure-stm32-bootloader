# Full Firmware Update Workflow

This project implements a firmware update workflow inside the custom STM32 bootloader.

The bootloader can replace the existing application firmware with a new firmware image.

## Firmware Update Flow

```text
Update flag detected
  ↓
Read prepared firmware image
  ↓
Validate firmware image magic and size
  ↓
Erase old application region
  ↓
Write new firmware to application flash region
  ↓
Calculate CRC32
  ↓
Compare calculated CRC with expected CRC
  ↓
Write updated application header
  ↓
Clear update flag
  ↓
Validate new application
  ↓
Jump to new application
```

## Flash Erase and Write

STM32 flash memory must be erased before it can be rewritten.

The bootloader erases the application flash region before writing the new firmware.

The bootloader must protect the bootloader region from being erased or overwritten.

## Firmware Image Validation

Before writing the new firmware, the bootloader checks:

- firmware magic number
- firmware size
- expected CRC32

This prevents invalid firmware images from being written to flash.

## Writing Firmware Chunks

The firmware is written into flash in chunks.

During the write process, the bootloader keeps track of:

- current write address
- number of bytes written
- remaining firmware size
- CRC calculation

## Final CRC Verification

After writing the firmware, the bootloader calculates CRC32 over the written application region.

The calculated CRC is compared with the expected CRC from the firmware image/header.

If the CRC matches, the firmware is accepted.

If the CRC fails, the firmware is rejected.

## Application Header Update

After a successful update, the bootloader writes the updated application header.

The header contains:

- update flag
- magic number
- firmware size
- CRC32
- firmware version

## Result

If the update succeeds, the bootloader clears the update flag, validates the new application, and jumps to it.

If the update fails, the bootloader remains in bootloader/error mode.
