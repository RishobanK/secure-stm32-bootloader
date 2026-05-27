# Application Validation

Application validation prevents the bootloader from jumping to invalid firmware.

Without validation, the bootloader might jump to empty flash, corrupted firmware, or an incorrectly flashed application image.

## Validation Checks

The bootloader validates the application using the following checks:

1. Magic number check
2. Reset handler address check
3. Application size check
4. CRC32 integrity check

## 1. Magic Number Check

The bootloader reads the application header from:

```text
0x08004000
```

It then checks whether the `magic` field matches the expected value.

If the magic number is wrong, the bootloader rejects the application.

This helps detect:

- missing application header
- erased flash
- incorrectly flashed firmware
- invalid firmware image

## 2. Reset Handler Address Check

The bootloader reads the reset handler address from:

```text
APP_START_ADDR + 4
```

The reset handler must point to a valid flash address.

If the reset handler is outside the expected flash region, the application is rejected.

## 3. Size Check

The application size must be valid.

The bootloader checks that the size is:

- not zero
- not larger than the available application flash region

This prevents the bootloader from reading outside the allowed memory area.

## 4. CRC32 Check

The bootloader calculates CRC32 over the application firmware and compares it with the CRC stored in the application header.

If both values match, the application is considered intact.

If they do not match, the application is rejected.

## Result

If all validation checks pass, the bootloader jumps to the application.

If any validation check fails, the bootloader remains in bootloader/error mode.
