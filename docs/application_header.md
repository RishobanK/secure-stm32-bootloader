# Application Header

The application header is stored at `0x08004000`.

It contains firmware metadata used by the bootloader before executing the application.

## Header Structure

```c
typedef struct {
    uint32_t ota_flag;
    uint32_t magic;
    uint32_t size;
    uint32_t crc;
    uint32_t version;
} app_header_t;
```

## Field Descriptions

| Field | Description |
|---|---|
| `ota_flag` | Indicates whether firmware update mode is requested |
| `magic` | Fixed value used to identify a valid application header |
| `size` | Size of the application firmware in bytes |
| `crc` | Expected CRC32 value of the application firmware |
| `version` | Application firmware version |

## Purpose of the Header

The bootloader reads the application header before jumping to the application.

It checks:

- whether the header exists
- whether the magic number is correct
- whether the application size is valid
- whether the CRC32 matches the firmware stored in flash
- whether update mode is requested

## Why the Header Is Separate

The actual application starts at `0x08004400`, while the header is stored at `0x08004000`.

This allows the bootloader to read metadata before executing the firmware.

```text
0x08004000  Application Header
0x08004400  Application Firmware
```

## Magic Number

The magic number is a fixed value used to confirm that the header is valid.

If the magic number does not match the expected value, the bootloader assumes that a valid application is not available.

## CRC

The CRC field stores the expected CRC32 value of the application firmware.

The bootloader calculates CRC32 over the application firmware region and compares it with this value.

If the CRC values do not match, the firmware is rejected.
