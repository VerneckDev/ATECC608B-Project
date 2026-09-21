# ATECC608B — Hardware AES-128 Encryption with Arduino

A project that uses the **Microchip ATECC608B** secure element as an **AES-128** cryptographic accelerator, controlled by an Arduino over I²C. The AES key is written into a protected slot of the chip, which keeps it internally: the Arduino only sends data blocks and receives the encrypted or decrypted result, and never reads the key.

The project has three parts:

1. **Provisioning** of the chip (locking the configuration and data zones and writing the AES key).
2. An **interactive test** that encrypts/decrypts phrases through the Serial Monitor.
3. **File encryption/decryption**, with a Python script that uses the Arduino and the chip as a hardware "bridge".

> **Warning:** the configuration sketch performs **irreversible** operations (permanently locking the ATECC608B configuration and data zones). Read the [Provisioning](#1-provisioning--atecc608b_config) section before uploading it.

---

## Repository structure

```
ATECC608B-Project/
├── ATECC608B_config/
│   └── ATECC608B_config.ino    # Provisioning: verifies config, locks zones, writes the AES key to slot 8
├── ATECC608B_test/
│   └── ATECC608B_test.ino      # Interactive AES test (phrase typed in the Serial Monitor)
├── ATECC608B_test2/
│   ├── ATECC608B_test2.ino     # AES "server" firmware over serial (ENC:/DEC: protocol)
│   └── enc_dec_file.py         # Python client: encrypts/decrypts files through the Arduino
└── README.md
```

No external cryptography library (for example, CryptoAuthLib) is used: the sketches talk to the chip directly over I²C using only `Wire.h`, with packet construction and the CRC-16 calculation implemented in the code itself.

---

## Hardware

- Arduino board with `Wire` (I²C) and an I²C buffer of at least **41 bytes** (see [notes](#implementation-notes)).
- **ATECC608B** (7-bit I²C address **`0x60`**).
- Pull-up resistors on SDA and SCL (many breakout boards already include them).
- USB cable.

Wiring:

```
Arduino                ATECC608B
  SDA   ─────────────►  SDA
  SCL   ─────────────►  SCL
  VCC   ─────────────►  VCC   (supply per datasheet)
  GND   ─────────────►  GND
```

The I²C bus runs at 100 kHz and the serial link to the PC at 9600 baud.

## Software

- [Arduino IDE](https://www.arduino.cc/en/software) (only the `Wire` library, included by default).
- Python 3 and [`pyserial`](https://pypi.org/project/pyserial/) (only for `enc_dec_file.py`):

```bash
python -m pip install pyserial
```

---

## How it works

### Protocol with the chip

The sketches send commands to the ATECC608B in the datasheet format: **word address** + **packet** (count, opcode, parameters, data) + **CRC-16** (polynomial `0x8005`, least significant byte first).

| Element | Value | Use |
|---|---|---|
| I²C address | `0x60` | Device |
| Word address `0x01` | Sleep | Puts the chip to sleep |
| Word address `0x02` | Idle | Ends the session |
| Word address `0x03` | Command | Sending commands |
| Opcode `0x02` | Read | Read the configuration zone |
| Opcode `0x12` | Write | Write the key to the slot |
| Opcode `0x17` | Lock | Lock zones |
| Opcode `0x51` | AES | Encrypt/decrypt one 16-byte block |

Typical sequence for each operation: **wake** (a pulse on SDA at `0x00`, followed by a 1.5 ms wait and reading the `04 11 …` response, validated with CRC) → **command** → wait for the execution time → **read and validate the response** (count + CRC).

### AES key

- Slot **8** of the data zone.
- The slot is 32 bytes: the first 16 hold the AES-128 key and the remaining 16 are padded with zeros.
- The AES command processes **one 16-byte block at a time** (mode `0x00` = encrypt, `0x01` = decrypt), which corresponds to **AES-ECB**.

For data larger than one block, the padding is **PKCS#7** and blocks are processed in sequence.

---

## Usage

### 1. Provisioning — `ATECC608B_config`

Everything runs in `setup()`, in the following order:

1. Wakes the chip and reads the 128 bytes of the configuration zone.
2. **Verifies** that the configuration is the expected one, and aborts without locking anything if it is not:

   | Field | Expected value |
   |---|---|
   | `LockConfig` / `LockValue` | `0x55` / `0x55` (zones still unlocked) |
   | `SlotConfig[8]` | `0x0F8F` |
   | `KeyConfig[8]` | `0x0038` |
   | `SlotLocked` | `0xFFFF` |

3. **Locks the configuration zone** (irreversible) and confirms that `LockConfig` became `0x00`.
4. Writes the AES key (16 bytes) to **slot 8**.
5. **Locks the data zone** (irreversible) and confirms that `LockValue` became `0x00`.
6. Runs a closing test: encrypts and decrypts `"ATECC608B-AES-01"` and compares the result.

Steps:

1. Connect the hardware and open the Serial Monitor at **9600 baud**.
2. Upload `ATECC608B_config.ino`.
3. Follow the messages: at the end you should see `>>> AES TEST SUCCESS <<<`.

> You only need to run this sketch **once per chip**. It **does not write** the configuration zone: it only verifies and locks it. The `SlotConfig`/`KeyConfig` values must already be present on the chip (for example, written beforehand) for the process to proceed.

### 2. Interactive test — `ATECC608B_test`

1. Upload `ATECC608B_test.ino` (chip already provisioned).
2. Open the Serial Monitor (9600 baud, line ending **Newline** or **Carriage return**).
3. Type a phrase (max **240 characters**) and press Enter.

The sketch applies PKCS#7 padding, encrypts all blocks, decrypts them, removes the padding, and compares the result with the original. At each step it shows the text, the bytes in hexadecimal, and the number of blocks, ending with `AES TEST SUCCESS` or `AES TEST FAILED`. It then asks for another phrase.

### 3. File encryption — `ATECC608B_test2`

Here the Arduino acts as an **AES server** on the serial port and Python as the client.

1. Upload `ATECC608B_test2.ino`. The Arduino sends `WIRE STARTED` at startup and waits for lines.
2. **Close the Serial Monitor** (the port can only be open in one program at a time).
3. In `enc_dec_file.py`, set `SERIAL_PORT` (default `'COM11'`) to your port.
4. Run:

```bash
# Encrypt (default mode)
python enc_dec_file.py document.pdf document.enc -e

# Decrypt
python enc_dec_file.py document.enc document_recovered.pdf -d
```

**Serial protocol** (one line per request, terminated with `\n`):

| Direction | Format | Example |
|---|---|---|
| PC → Arduino | `ENC:<hex>` or `DEC:<hex>` | `ENC:414243…` |
| Arduino → PC | a line with the result in hexadecimal, no spaces | `9F03…` |
| Arduino → PC (error) | `ERROR_LENGTH`, `ERROR_ENCRYPT_LENGTH`, `ERROR_DECRYPT_LENGTH`, `ERROR_ENCRYPT`, `ERROR_DECRYPT` | |

How it works in detail:

- The file is read in binary mode and PKCS#7 padding is applied **in Python**; the Arduino only processes complete blocks (multiples of 16 bytes).
- Each request carries at most **240 bytes** (15 blocks), a limit set by the Arduino's SRAM.
- The chip is woken **once per request**, not once per block.
- When finished, the script prints the number of blocks processed and the total time.
- For decryption, the file size must be a multiple of 16 bytes.

---

## Implementation notes

- **I²C buffer:** writing the key generates a 40-byte transmission (1 word address + 37 packet + 2 CRC). The default `Wire` buffer on AVR boards is 32 bytes, so the configuration sketch requires `BUFFER_LENGTH ≥ 41` and aborts with `ERROR: Wire buffer too small!` if it is not enough. Use a board/core with a larger buffer or increase `BUFFER_LENGTH`.
- **Lock without summary CRC check:** the *lock* commands use the "ignore summary CRC" option (`0x80` and `0x81`), so the chip does not validate the configuration as a whole before locking. The sketch partially compensates by checking the relevant slot 8 fields before locking.
- **Wait times:** the code uses `delay(15)` after the AES command and `delay(40)` after *write*/*lock*, values chosen for the command execution time.
- **Performance:** file encryption is slow by nature: hexadecimal data at 9600 baud, one I²C command per 16-byte block, and a *wake* on every request.

## Limitations and security

This project is **educational/experimental**. Do not use it as-is to protect real data:

- **The AES key is written in the source code** (`ATECC608B_config.ino`) and, since the repository is public, it must be considered **compromised**. The chip prevents *reading* the key after provisioning, but that does not help if the key is already public. For real use, generate a random key, supply it outside the repository, or generate it on the chip itself.
- **AES-ECB:** identical blocks produce identical ciphertext, which reveals patterns in the data. There is also no IV and no authentication (no integrity). For real data, prefer an authenticated mode (for example, AES-GCM).
- **Irreversibility:** after provisioning, the configuration and the key **cannot be changed**. A wrongly provisioned chip becomes unusable for any other purpose.
- **Cleartext communication:** data travels unprotected over I²C and the serial port; anyone with physical access to the bus can use the chip as an encrypt/decrypt oracle.
- On decryption, `enc_dec_file.py` does not flag invalid padding (for example, with a corrupted file): it returns the data as it is.

---

## Author

**João Pedro Verneck** — [@VerneckDev](https://github.com/VerneckDev)

## References

- Microchip Technology — *ATECC608B CryptoAuthentication Device* (datasheet).
- [Arduino `Wire` library documentation](https://www.arduino.cc/reference/en/language/functions/communication/wire/).
- [pyserial](https://pyserial.readthedocs.io/).