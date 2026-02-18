<p align="center">
  <img src="https://raw.githubusercontent.com/avp-protocol/spec/main/assets/avp-shield.svg" alt="AVP Shield" width="80" />
</p>

<h1 align="center">avp-hardware</h1>

<p align="center">
  <strong>Hardware secure element for Agent Vault Protocol</strong><br>
  FIPS 140-3 Level 3 · Tamper resistant · Keys never leave silicon
</p>

<p align="center">
  <a href="https://github.com/avp-protocol/avp-hardware/releases"><img src="https://img.shields.io/github/v/release/avp-protocol/avp-hardware?style=flat-square&color=00D4AA" alt="Release" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache_2.0-blue?style=flat-square" alt="License" /></a>
  <a href="LICENSE-HARDWARE"><img src="https://img.shields.io/badge/hardware-CERN_OHL_S_2.0-blue?style=flat-square" alt="Hardware License" /></a>
</p>

---

## Overview

`avp-hardware` provides firmware and hardware designs for AVP-compatible secure elements. Keys are generated, stored, and used entirely within tamper-resistant silicon — they never touch the host computer's memory.

## Why Hardware?

| Threat | File | Keychain | Hardware |
|--------|:----:|:--------:|:--------:|
| Infostealer malware | ✗ | ✓ | ✓ |
| Credential phishing | ✗ | ✓ | ✓ |
| Full host compromise | ✗ | ✗ | ✓ |
| Memory dump | ✗ | ✗ | ✓ |
| Physical device theft | — | — | ✓ |
| Supply chain attack | ✗ | ✗ | ✓ |
| Insider threat | ✗ | ⚠ | ✓ |

**Only hardware protects against all seven threat categories.**

## Supported Devices

### Reference Design (Recommended)

| Device | Secure Element | Interface | Status |
|--------|---------------|-----------|--------|
| AVP-SE1 | ATECC608B | USB-C | 🔨 In development |
| AVP-SE2 | Infineon SLE97 | USB-C | 📋 Planned |

### Compatible Third-Party Devices

| Device | Notes |
|--------|-------|
| YubiKey 5 | Via FIDO2/PIV (limited operations) |
| Nitrokey 3 | Full AVP support (community firmware) |
| OnlyKey | Full AVP support (community firmware) |

## Hardware Operations

The AVP Hardware extension adds three operations:

### HW_CHALLENGE — Device Attestation

Verify the device is genuine and untampered:

```bash
avp hw-challenge
# Challenge: 0x7f3a...
# Response: 0x9c2b...
# Verified: true
# Manufacturer: AVP Reference
# Model: SE1
# Firmware: 1.0.0
```

### HW_SIGN — Sign Without Export

Sign data without the key ever leaving the device:

```bash
echo "payload" | avp hw-sign signing_key
# Signature: 0x3d7e...
```

### HW_ATTEST — Compliance Proof

Generate cryptographic proof that a secret is stored in hardware:

```bash
avp hw-attest anthropic_api_key
# Attestation Certificate:
# -----BEGIN CERTIFICATE-----
# ...
# -----END CERTIFICATE-----
#
# This proves the secret "anthropic_api_key" is stored in
# FIPS 140-3 Level 3 certified hardware and has never been exported.
```

## Firmware

The reference firmware is written in Rust (no_std) and runs on ARM Cortex-M microcontrollers.

### Building

```bash
cd firmware
cargo build --release --target thumbv7em-none-eabihf
```

### Flashing

```bash
cargo flash --chip ATSAME54P20A --release
```

### Security Features

- **Secure boot** — Firmware signature verification
- **Anti-tamper** — Zeroization on physical intrusion
- **Side-channel resistance** — Constant-time operations
- **Memory protection** — MPU isolation of key material
- **Watchdog** — Automatic reset on fault

## Hardware Design

### Schematic

The reference schematic is available in KiCad format:

```
hardware/
├── avp-se1/
│   ├── avp-se1.kicad_sch    # Schematic
│   ├── avp-se1.kicad_pcb    # PCB layout
│   ├── avp-se1-bom.csv      # Bill of materials
│   └── avp-se1-gerbers.zip  # Manufacturing files
```

### Key Components

| Component | Part Number | Purpose |
|-----------|-------------|---------|
| MCU | ATSAME54P20A | Main controller |
| Secure Element | ATECC608B | Key storage & crypto |
| USB | USB-C connector | Host interface |
| ESD | TPD2E2U06 | Protection |

### Manufacturing

Gerber files and BOM are ready for JLCPCB/PCBWay. Estimated cost: ~$15/unit at 100 qty.

## Compliance

- **FIPS 140-3 Level 3** — In progress (ATECC608B is FIPS certified)
- **Common Criteria EAL5+** — Planned
- **PCI PTS** — Planned

## Protocol

The hardware communicates using the AVP protocol over USB CDC (serial):

```
Host                          Device
  │                              │
  │──── DISCOVER ───────────────>│
  │<─── capabilities ────────────│
  │                              │
  │──── AUTHENTICATE (PIN) ─────>│
  │<─── session_id ──────────────│
  │                              │
  │──── STORE (encrypted) ──────>│
  │<─── ok ──────────────────────│
  │                              │
  │──── HW_SIGN (payload) ──────>│
  │<─── signature ───────────────│
```

## Contributing

We need:

- **Hardware engineers** — Review schematic, suggest improvements
- **Firmware developers** — Rust embedded experience
- **Security researchers** — Audit firmware and protocol
- **Beta testers** — Test pre-production units

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

- **Firmware**: Apache 2.0
- **Hardware**: CERN Open Hardware License Version 2 - Strongly Reciprocal (CERN-OHL-S-2.0)

---

<p align="center">
  <a href="https://github.com/avp-protocol/spec">Specification</a> ·
  <a href="https://github.com/avp-protocol/avp-hardware/issues">Issues</a>
</p>
