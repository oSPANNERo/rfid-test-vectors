# MIFARE Classic 1K state matrix

This test vector exercises readable-Key-B handling plus explicit unresolved/error-state controls.

## Files

- `mfc1k_iceman_state_matrix.bin` — synthetic 1K card image
  - UID: `12 34 56 78`
  - ATQA: `00 04`
  - SAK: `08`
  - SHA-256: `77e1025e5aae72e68be03152edf18715e8dcc870971283e0598fa4e166bfe895`
- `mfc1k_iceman_state_matrix-key.bin` — Proxmark3 binary key file, 192 bytes
  - SHA-256: `a7132872475aab92d247cc64c01c247402c94577a55d289d9a5f21dd2dc61bde`
- `mfc1k_iceman_state_matrix.dic` — 22-key text dictionary for GUI regression testing
  - 15 valid Key A values
  - 7 valid protected Key B values
  - deliberately omits the valid Key A for sector 2 and valid protected Key B for sector 11

## Baseline

Even sectors use `FF 07 80 69`, making the Key B field readable trailer data (`E0 E1 E2 E3 E4 xx`). The Proxmark3 emulator used for this vector models the NXP-specified behavior where these readable B-field values are not usable for subsequent authenticated memory access via Key B.

Odd sectors use `F7 8F 00 69`, with protected authentication Key B values `B0 B1 B2 B3 B4 xx`.

### Compatibility note

Do not infer Key B authentication usability from the ACL alone when testing arbitrary physical cards or compatible implementations. A tested physical MIFARE Classic-compatible card returned readable Key B bytes under `FF 07 80 69` and also accepted those same bytes for authentication and subsequent memory access.

Software that needs to classify a readable Key B value as an authentication key should therefore validate actual Key B memory access instead of assuming the result solely from the access-condition bits.

## Special controls

### Sector 2

The card contains Key A `A0 A1 A2 A3 A4 02`, but that valid Key A is deliberately absent from `mfc1k_iceman_state_matrix.dic`.

Expected GUI result: Key A remains unresolved, and Key B also remains unresolved because the trailer cannot first be authenticated and read with Key A.

The Proxmark3 binary key file likewise contains a deliberately incorrect Key A for this sector (`C0 C1 C2 C3 C4 02`).

### Sector 10

Key A is present in the GUI dictionary and the trailer uses `FF 07 80 69`.

Expected GUI result: Key A resolves normally. The card-returned B-field bytes are visible, but because the emulator rejects those bytes for Key B memory access, Key B is classified as readable data rather than a valid authentication key.

### Sector 11

Key A is present in the GUI dictionary, the trailer uses `F7 8F 00 69`, and the valid protected Key B is deliberately absent from the dictionary.

Expected GUI result: Key A resolves normally and Key B remains unresolved.

For the Proxmark3 binary-key-file test, the supplied protected Key B is deliberately wrong (`D0 D1 D2 D3 D4 0B`). Current `hf mf dump` reconstruction may place that supplied value into the trailer even though it was not read from the card or validated as a B authentication key. This documents existing behavior rather than the readable-Key-B fix itself.

## Proxmark3 dump regression

```text
hf mf dump --ns --keys mfc1k_iceman_state_matrix-key.bin
```

### Proxmark3 pass criteria

- Ordinary even sectors preserve actual card-returned `E0E1E2E3E4xx` B bytes.
- Sector 2 remains unreadable with the deliberately wrong Key A.
- Ordinary odd sectors reconstruct protected `B0B1B2B3B4xx` values from the supplied key file.
- Sector 11 remains readable via Key A; any displayed `D0...0B` protected B value came from the supplied key file, not from a card read.

## ChameleonUltraGUI regression

Use `mfc1k_iceman_state_matrix.dic` as the only selected dictionary, with the default dictionary skipped, and run the dictionary key check against the emulated card.

Expected result:

| Sector | Key A | Key B |
|---|---|---|
| 0 | valid | readable-only |
| 1 | valid | valid |
| 2 | unresolved | unresolved |
| 3 | valid | valid |
| 4 | valid | readable-only |
| 5 | valid | valid |
| 6 | valid | readable-only |
| 7 | valid | valid |
| 8 | valid | readable-only |
| 9 | valid | valid |
| 10 | valid | readable-only |
| 11 | valid | unresolved |
| 12 | valid | readable-only |
| 13 | valid | valid |
| 14 | valid | readable-only |
| 15 | valid | valid |

In the current GUI, valid authentication keys are shown as normal recovered/green keys, readable-only B fields use the blue visibility state, and unresolved entries remain red.

For a readable-only sector, the visibility tooltip should show the actual card-returned bytes, for example sector 0:

```text
Data: E0E1E2E3E400
```

### GUI export pass criteria

`Export found keys` should contain exactly 22 usable authentication keys:

- 15 valid Key A values
- 7 valid protected Key B values
- zero `E0E1E2E3E4xx` readable-only values

A validated export from this regression produced SHA-256:

```text
5f55ca8ed56f14f365961aaaa31a04b7b70face1698cd8805640791ada279a2f
```

The readable-only values must remain display data and must not enter the authentication-key export.
