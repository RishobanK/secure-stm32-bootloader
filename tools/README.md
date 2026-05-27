# Tools

This folder contains Python scripts used to prepare firmware images for the bootloader update workflow.

## Included / Planned Tools

### `app_final.py`

Generates a firmware image with metadata such as:

- magic number
- firmware size
- CRC32
- firmware version

### `bin_to_header.py`

Converts a binary firmware image into a C header array for testing the bootloader update process.

## Future Tool Improvements

Planned improvements include:

- UART firmware sender using Python and pyserial
- firmware packet generator
- HMAC-SHA256 firmware authentication generator
- version metadata generator
- update image validation tool
