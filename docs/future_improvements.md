# Future Improvements

The current implementation provides a working custom STM32 bootloader with application validation, CRC32 integrity checking, update flag handling, and firmware update workflow.

The following improvements are planned for future development.

---

## 1. UART-Based Firmware Transfer from PC

The current update workflow uses a prepared firmware image.

A future improvement is to transfer firmware from a PC to the bootloader over UART using a Python firmware sender.

Planned features:

- Python firmware sender tool
- UART packet protocol
- ACK/NACK response handling
- chunk-based firmware transfer
- timeout and retry mechanism
- CRC verification after transfer

---

## 2. OTA Update via TCP Server

A future extension is to support firmware update from a TCP server.

Planned flow:

```text
Bootloader enters update mode
  ↓
Connect to TCP server through network interface
  ↓
Receive firmware image in chunks
  ↓
Write chunks to flash
  ↓
Verify CRC
  ↓
Update application header
  ↓
Boot new application
```

This will demonstrate network-based firmware update concepts.

---

## 3. Wireless Update Using ESP8266 and External SPI Flash

Another planned extension is wireless firmware update using an ESP8266 module and external SPI flash.

Planned architecture:

```text
ESP8266 downloads firmware over Wi-Fi
  ↓
Firmware stored in external SPI flash
  ↓
STM32 bootloader reads firmware from SPI flash
  ↓
STM32 verifies and flashes new application
```

This separates wireless communication from the STM32 bootloader and keeps the STM32 firmware focused on validation and flashing.

---

## 4. HMAC-SHA256 Based Firmware Authentication

CRC32 detects accidental corruption, but it does not prove that firmware is authorized.

A future improvement is to add HMAC-SHA256 based authentication.

Planned flow:

```text
Python tool calculates HMAC using secret key
  ↓
Firmware package includes HMAC
  ↓
Bootloader calculates HMAC over firmware
  ↓
Bootloader compares expected and calculated HMAC
  ↓
Firmware is accepted only if HMAC matches
```

This will demonstrate authenticated firmware validation.

---

## 5. Digital Signature Verification

A stronger future security improvement is digital signature verification using public-key cryptography.

This would allow the bootloader to verify firmware authenticity without storing a shared secret key in firmware.

Planned concept:

```text
Firmware signed using private key
  ↓
Bootloader verifies signature using public key
  ↓
Only correctly signed firmware is accepted
```

---

## 6. Version Checking and Anti-Rollback

Version checking will prevent older firmware from replacing newer firmware.

Example:

```text
Installed version: 3
Incoming version: 2
Result: rejected
```

This prevents accidental downgrade and helps protect against rollback to vulnerable firmware.

---

## 7. Application Health Check and Rollback

A future improvement is to support application health confirmation.

Planned flow:

```text
New firmware installed
  ↓
Bootloader marks app as PENDING_VERIFY
  ↓
Application boots and confirms health
  ↓
Bootloader marks app as VALID
```

If the application does not confirm successful boot, the bootloader can remain in update mode or roll back to a previous valid image.

---

## 8. Improved UART Logging

UART logging will be improved using either:

- dedicated USB-to-TTL adapter
- ESP32-C3 USB-to-UART bridge
- ST-Link Virtual COM Port, if available

This will improve runtime diagnostics and make validation easier to observe.

---

## 9. Demo Video and Debugger Screenshots

Planned documentation improvements:

- bootloader-to-application jump demo
- validation failure LED patterns
- flash memory layout screenshots
- debugger watch window screenshots
- firmware update workflow demonstration
