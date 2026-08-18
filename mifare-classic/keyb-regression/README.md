# MIFARE Classic 1K readable Key B regression

This test vector validates `hf mf dump` behavior when a MIFARE Classic 1K contains alternating readable and protected Key B fields.

## Files

- `mfc1k_iceman_keyb_regression.bin` — synthetic 1K card image
  - UID: `12 34 56 78`
  - ATQA: `00 04`
  - SAK: `08`
  - SHA-256: `77e1025e5aae72e68be03152edf18715e8dcc870971283e0598fa4e166bfe895`
- `mfc1k_iceman_keyb_regression-key.bin` — Proxmark3 binary key file, 192 bytes
  - Layout: 16 Key A values followed by 16 Key B values
  - SHA-256: `5d8f6ff07f73d5a361b5de73a92502f79b3fa631894003757d48e928c13dc2bc`

## Design

Even sectors (`0, 2, 4, ... 14`) use trailer ACL/GPB `FF 07 80 69`. Key B is readable data and cannot authenticate. The actual card-returned B field is `E0 E1 E2 E3 E4 xx`, while the supplied key file deliberately contains the wrong value `D0 D1 D2 D3 D4 xx`.

Odd sectors (`1, 3, 5, ... 15`) use trailer ACL/GPB `F7 8F 00 69`. Key B is protected and usable for authentication. The supplied key file contains the correct `B0 B1 B2 B3 B4 xx` value.

## Test

Load or emulate the card image, then run:

```text
hf mf dump --ns --keys mfc1k_iceman_keyb_regression-key.bin
```

## Pass criteria

- Even-sector trailers preserve the card-returned `E0E1E2E3E4xx` bytes.
- Even-sector trailers do **not** contain the deliberately bogus `D0D1D2D3D4xx` bytes from the key file.
- Odd-sector trailers contain the protected `B0B1B2B3B4xx` Key B values supplied by the key file.

The historical failure mode is immediately visible because an affected even sector changes from `E0...` to `D0...`.
