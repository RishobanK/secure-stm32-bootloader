# CRC32 and Size Validation

This project uses application size validation and CRC32 validation to improve firmware reliability.

## Why Size Validation Is Needed

The application header stores the size of the application firmware.

The bootloader uses this value to know how many bytes of application firmware should be checked.

The size must be:

- greater than zero
- less than the maximum allowed application region

If the size is invalid, the bootloader rejects the application.

## Why CRC32 Validation Is Needed

CRC32 is used to detect accidental corruption.

It can detect problems such as:

- incomplete firmware update
- corrupted firmware image
- wrong binary file
- flash write error

## CRC32 Flow

```text
Application firmware in flash
        ↓
Bootloader calculates CRC32
        ↓
Compare with CRC stored in header
        ↓
If match: application is accepted
If mismatch: application is rejected
```

## Important Note

CRC32 is not cryptographic security.

CRC32 can detect accidental corruption, but it does not prove that the firmware came from an authorized source.

For firmware authentication, future improvements include:

- HMAC-SHA256 validation
- digital signature verification

## Validation Logic

The bootloader performs the following:

```text
Read application header
  ↓
Check size
  ↓
Calculate CRC32 over application firmware
  ↓
Compare calculated CRC with stored CRC
  ↓
Accept or reject firmware
```

## Purpose in This Project

CRC32 and size validation make the bootloader safer by ensuring that the application firmware is complete and correctly written before execution.
