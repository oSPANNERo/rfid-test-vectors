# MIFARE Classic 1K state matrix

This test vector exercises the readable-Key-B fix plus explicit unresolved/error-state controls.

## Files

- `mfc1k_iceman_state_matrix.bin` — synthetic 1K card image
  - UID: `12 34 56 78`
  - ATQA: `00 04`
  - SAK: `08`
  - SHA-256: `77e1025e5aae72e68be03152edf18715e8dcc870971283e0598fa4e166bfe895`
- `mfc1k_iceman_state_matrix-key.bin` — Proxmark3 binary key file, 192 bytes
  - SHA-256: `a7132872475aab92d247cc64c01c247402c94577a55d289d9a5f21dd2dc61bde`

## Baseline

Even sectors use `FF 07 80 69`, making Key B readable data (`E0 E1 E2 E3 E4 xx`) while the supplied key file deliberately contains `D0 D1 D2 D3 D4 xx` in the B slot.

Odd sectors use `F7 8F 00 69`, with protected authentication Key B values `B0 B1 B2 B3 B4 xx`.

## Special controls

### Sector 2

The card contains Key A `A0 A1 A2 A3 A4 02`, but the supplied Key A is deliberately wrong: `C0 C1 C2 C3 C4 02`.

Expected result: authentication fails. The readable B field cannot rescue authentication because it is data under this ACL, not an authentication key.

### Sector 11

The supplied Key A is correct, so the sector remains readable, but the supplied protected Key B is deliberately wrong (`D0 D1 D2 D3 D4 0B`).

Expected result: current `hf mf dump` reconstruction may place that supplied value into the trailer even though it was not read from the card or validated as a B authentication key. This documents existing behavior rather than the readable-Key-B fix itself.

## Test

```text
hf mf dump --ns --keys mfc1k_iceman_state_matrix-key.bin
```

## Pass criteria

- Ordinary even sectors preserve actual card-returned `E0E1E2E3E4xx` B bytes.
- Sector 2 remains unreadable with the deliberately wrong Key A.
- Ordinary odd sectors reconstruct protected `B0B1B2B3B4xx` values from the supplied key file.
- Sector 11 remains readable via Key A; any displayed `D0...0B` protected B value came from the supplied key file, not from a card read.
