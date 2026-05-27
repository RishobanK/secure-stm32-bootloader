# Update Flag Mechanism

The update flag allows the application to request firmware update mode.

The flag is stored inside the application header at `0x08004000`.

## Why an Update Flag Is Needed

Normally, the bootloader validates the application and jumps to it.

However, during a firmware update, the application needs a way to tell the bootloader:

```text
Do not boot the normal application next time.
Enter firmware update mode instead.
```

The update flag provides this communication method.

## Why Flash Is Used

RAM is cleared after reset.

Flash memory is non-volatile, so a value written to flash remains available even after reset or power cycle.

This allows the application to:

1. set the update flag in flash
2. reset the MCU
3. allow the bootloader to detect the flag after restart

## Update Request Flow

```text
Application running
  ↓
Update requested
  ↓
Application writes update flag to flash
  ↓
Application resets MCU
  ↓
Bootloader starts
  ↓
Bootloader reads update flag
  ↓
Bootloader enters firmware update mode
```

## Clearing the Flag

After detecting the update flag, the bootloader clears it.

This prevents the system from entering update mode repeatedly after every reset.

## Why Read-Modify-Write Is Important

The update flag is stored in the same flash page as the application header.

The header also contains:

- magic number
- size
- CRC
- version

Flash memory cannot be modified like RAM. Usually, the whole flash page must be erased before writing again.

Therefore, the firmware must:

1. read the current header into RAM
2. modify only the update flag
3. erase the flash page
4. write the full header back to flash

This prevents the other header fields from being lost.
