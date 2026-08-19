# RFID Test Vectors

Reusable RFID/NFC test vectors and regression cases for Proxmark3, ChameleonUltra, and related tools.

The repository is intentionally small and test-focused. Each case includes the binary image, any companion key material needed to reproduce the scenario, and concise pass criteria.

## MIFARE Classic

- [`mifare-classic/keyb-regression`](mifare-classic/keyb-regression/) — verifies that readable sector-trailer Key B data is preserved instead of being overwritten by a supplied key-file value.
- [`mifare-classic/state-matrix`](mifare-classic/state-matrix/) — adds explicit authentication-failure and protected-Key-B control states, plus a 22-key dictionary and pass criteria for ChameleonUltraGUI readable-vs-authentication Key B regression testing.
- [`mifare-classic/clone-emulator-comparison`](mifare-classic/clone-emulator-comparison/) — records observed identity, memory-fidelity, magic-card, PRNG, and nonce-behavior differences between the original source card, Gen1a and Gen2/CUID clones, Chameleon Ultra, and Flipper Zero.

The state-matrix vectors intentionally distinguish between a Key B field being readable from a trailer and that same value being proven usable for authenticated memory access. Software should validate the latter rather than infer it solely from the trailer ACL.

These fixtures and observations are intended for authorized testing, software validation, and regression reproduction. Behavioral comparison results describe the specific tested hardware/software and should not be generalized to every implementation of the same class.

## License

MIT. See [`LICENSE`](LICENSE).
