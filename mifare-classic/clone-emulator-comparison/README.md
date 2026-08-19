# MIFARE Classic clone and emulator comparison

This table records observed behavior from a controlled comparison of one original MIFARE Classic 1K source card against a Gen1a clone, Gen2/CUID clone, Chameleon Ultra emulation, and Flipper Zero emulation.

The comparison is an observation set for the specific hardware and software tested. It should not be read as a guarantee that every device or compatible card of the same class will exhibit identical behavior.

Source identity used for the comparison:

- UID: `42DCAEF2`
- ATQA: `00 04`
- SAK: `08`

The machine-readable source for the table is [`MIFARE_Classic_Clone_Emulator_Comparison.csv`](MIFARE_Classic_Clone_Emulator_Comparison.csv).

## Comparison

| **Test** | **Original source** | **Gen1a clone** | **Gen2/CUID clone** | **Chameleon Ultra** | **Flipper Zero** |
|---|---|---|---|---|---|
| UID | 42DCAEF2 | Match | Match | Match | Match |
| ATQA | 00 04 | Match | Match | Match | Match |
| SAK | 08 | Match | Match | Match | Match |
| RATS | Unsupported | Unsupported | Unsupported | Unsupported | Unsupported |
| Block 0 | Canonical | Match | Match | Match | Match |
| Blocks 20–22 | Canonical | Match | Match | Match | Match |
| Diagnostic B-fields | Canonical | 7/7 match | 7/7 match | 7/7 match | 7/7 match |
| Crypto1 authentication | Yes | Yes | Yes | Yes | Yes |
| PM3 fingerprint | n/a | n/a | n/a | n/a | n/a |
| PM3 magic detection | `<n/a>` | Gen1a | Gen2/CUID | `<n/a>` | `<n/a>` |
| PRNG classification | Weak | Static nonce | Weak | Weak | Hard |
| Special nonce behavior | Static encrypted nonce | Static 01200145 | Normal weak PRNG | Does not reproduce source SEN behavior | Hard PRNG |
| Memory fidelity | Baseline | Yes | Yes | Yes | Yes |
| Donor-independent behavior tested? | — | — | — | — | Yes |
| Best distinguishing signal | Baseline | Gen1a probe / static nonce | Gen2/CUID probe | Missing source SEN behavior | Hard PRNG |
| Easy to distinguish from original? | — | Yes | Yes | Yes, behaviorally | Yes, PRNG |

## Interpretation

All four tested replacements reproduced the source identity and the tested memory contents, including the seven diagnostic B-field values. The strongest differences appeared in implementation behavior rather than memory fidelity:

- the Gen1a clone exposed Gen1a magic behavior and a static nonce
- the Gen2/CUID clone exposed Gen2/CUID magic behavior
- Chameleon Ultra matched memory but did not reproduce the source static-encrypted-nonce behavior
- Flipper Zero matched memory but presented a hard PRNG

These distinctions make the tested replacements separable from the original source even when the application-visible memory is faithful.
