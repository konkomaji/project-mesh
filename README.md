<div align="center">
<img src="docs/assets/mesh-logo.svg" width="108" height="108" alt="Project Mesh">

# Project Mesh

**The protocol behind Betar: a social and chat network no government can switch off.**

*Decentralized, delay-tolerant communication over Bluetooth LE and Wi-Fi Direct,*
*bridged to the open Nostr relay network when any device reaches the internet.*

[![Core tests](https://img.shields.io/badge/core%20tests-191%20passing-brightgreen)](core/)
[![Code licence](https://img.shields.io/badge/code%20licence-AGPL--3.0--or--later-lightgrey)](LICENSE)
[![Docs licence](https://img.shields.io/badge/docs%20licence-CC%20BY--SA%204.0-lightgrey)](docs/GOVERNANCE.md)
[![Status](https://img.shields.io/badge/status-pre--alpha-orange)](docs/IMPLEMENTATION-STATUS.md)

**[Read the whitepaper](WHITEPAPER.md)** &middot; **[Betar, the reference app](https://github.com/konkomaji/betar)** &middot; **[betar's website](https://konkomaji.github.io/betar/)**

**Developed by [Konko Maji](https://konkomaji.com/)**

</div>

---

I started Project Mesh to remove a single point of failure from communication: the server.
A server can be subpoenaed. A server can be ordered offline. A server can be sold to someone
who changes the rules. Project Mesh has none, by design.

This repository holds the protocol and the shared Rust core (`core/`) that implements it.
**[Betar](https://github.com/konkomaji/betar)** is the reference Android application built on
top of it. Betar depends on this repo's `core/` as a git submodule, so every app built on this
protocol shares one canonical implementation of the cryptography and the mesh engine.

---

### At a glance

| | |
|---|---|
| **Crypto** | Noise `XX` to Double Ratchet. X3DH plus hybrid post-quantum PQXDH (ML-KEM-1024) for offline first contact. MLS groups (RFC 9420). Argon2id passphrase channels. Envelope-size padding against traffic analysis. |
| **Transport** | BLE dual-role GATT plus Wi-Fi Direct today, composed behind one mesh engine. Store-carry-forward gossip, TTL, rate limiting, a client puzzle. A Nostr relay bridge extends any local mesh cluster to the public internet the moment one device in it goes online. LoRa (865 to 868 MHz, India's licence-free band) is planned as a longer-range backbone. |
| **Confidence** | 191 Rust tests. A DTN simulation harness that found a real relay bug. A wire-parser fuzzing harness. A security-review pass. |
| **What it isn't** | No server, no account, no cloud, no independent security audit yet. See [`docs/THREAT-MODEL.md`](docs/THREAT-MODEL.md). |

---

## Why this exists

A server is a single point of control. Whoever runs it, or whoever can pressure whoever runs
it, decides who gets to speak and who gets cut off. In 2026 alone, Iran ran an 88-day national
internet shutdown, the longest on record. France's government asked its own mobile carriers to
be ready to block four social platforms for several hours. India ordered GitHub to restrict
three repositories of Bitchat, a Bluetooth-mesh messenger with no server to subpoena, because
its anonymous design made lawful interception harder for the state. None of these actions
targeted one bad post. Each one targeted an entire platform, because no narrower lever existed.

Project Mesh removes the lever. No node is a hub. Any node can relay for any other. Killing one
device, one relay, or even one country's internet access does not kill the network, because the
network was never one thing to kill.

Bitchat, built by Jack Dorsey's team, is the direct inspiration for this approach: Bluetooth
mesh for local reach, a public relay network for everything beyond it, no account and no phone
number required. Project Mesh and Betar build on the same idea, with an independent
cryptographic core and a wider feature set: posts, fact-checking, and group chat, not only
direct messages.

## Documentation map

| Document | What it covers |
|---|---|
| [`WHITEPAPER.md`](WHITEPAPER.md) | The full research paper: problem, design, protocol, evaluation |
| [`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md) | Live status: what's actually built versus still design-only, per component |
| [`docs/PROGRESS.md`](docs/PROGRESS.md) | Running log: dated, one entry per work session, what happened and why |
| [`docs/RESEARCH-FINDINGS.md`](docs/RESEARCH-FINDINGS.md) | Verified research and corrections, adversarially checked |
| [`docs/BETAR-TRANSITION.md`](docs/BETAR-TRANSITION.md) | The rename and repo-split tracker: what's decided, what's left, in what order |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | System layers, the Rust core, module boundaries, the mesh-to-relay bridge |
| [`docs/TRANSPORT.md`](docs/TRANSPORT.md) | BLE mesh, Wi-Fi Direct, LoRa; iOS and Android radio realities |
| [`docs/ROUTING-PROTOCOL.md`](docs/ROUTING-PROTOCOL.md) | Store-carry-forward routing, packet format, dedup, TTL |
| [`docs/CRYPTOGRAPHY.md`](docs/CRYPTOGRAPHY.md) | Identity, Noise handshake, Double Ratchet, MLS, PQXDH, onion routing |
| [`docs/THREAT-MODEL.md`](docs/THREAT-MODEL.md) | Adversaries, assets, attacks, mitigations, non-goals |
| [`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md) | LoRa gateway spec, the 865 to 868 MHz band, node design |
| [`docs/DISTRIBUTION.md`](docs/DISTRIBUTION.md) | F-Droid and direct-APK distribution, de-Googled builds, reproducibility |
| [`docs/REPRODUCIBLE-BUILD.md`](docs/REPRODUCIBLE-BUILD.md) | Pinned toolchain, build steps, what's actually verified reproducible |
| [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) | Licence, community stewardship, contribution model |
| [`docs/COMPLIANCE.md`](docs/COMPLIANCE.md) | Spectrum compliance, licence, no-warranty position |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Phased delivery plan and milestones |
| [`docs/REFERENCES.md`](docs/REFERENCES.md) | Citations, prior art, standards, further reading |

Docs written while the Android app still lived in this mono-repo, such as
[`docs/DESIGN-BRIEF.md`](docs/DESIGN-BRIEF.md), [`docs/LOCALIZATION-UX.md`](docs/LOCALIZATION-UX.md),
and [`docs/FEATURES.md`](docs/FEATURES.md), describe the app, not the protocol. They stay here
for now, not yet moved to [Betar's own repo](https://github.com/konkomaji/betar).

## Status

**Pre-alpha.** Full detail, component by component, is in
[`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md). This is the short version.

**Rust core (`core/`, 191 tests passing):** identity (Ed25519 plus X25519), Noise `XX` handshake
to Double Ratchet for 1:1 messaging, X3DH and hybrid post-quantum PQXDH (ML-KEM-1024 plus
classical, via the `ml-kem` crate) for async first-contact bootstrap when both parties are never
online at once, MLS groups (RFC 9420, via `openmls`) for larger groups, Argon2id
passphrase-derived channels, envelope wire format plus Bloom-filter gossip summaries, a
store-carry-forward mesh engine (gossip-on-contact, epidemic relay, TTL, rate limiting, an
optional client puzzle), envelope-size padding (metadata protection against relay traffic
analysis) wired into every addressing mode, encryption at rest (`redb` plus AEAD, a deliberate
substitution for the design docs' SQLCipher, whose OpenSSL dependency would not build in this
dev environment), and a full UniFFI surface covering all of it. A
[DTN simulation harness](core/src/dtn_sim.rs) drives the real relay engine through scripted
contact schedules with no hardware needed. Building it surfaced and fixed a real TTL-decrement
gap in the gossip-relay path. A [wire-parser fuzzing harness](core/fuzz/) covers every
untrusted-bytes-in parser. It cannot run on this Windows dev box, a documented MSVC/`cdylib`
linker conflict, and needs Linux or CI.

**The Android app moved to its own repository, [konkomaji/betar](https://github.com/konkomaji/betar),
on 2026-07-29.** It depends on this repo's `core/` as a git submodule. What's built, what's
released, and how to install it now lives in that repo's own README and release notes, not here.

**What's not done yet in the protocol itself:** no independent security audit. This is a hard
release gate, not attempted here, see [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) section 3. The
wire-parser fuzzing harness has never actually run, blocked on the Windows/MSVC linker conflict
above. LoRa is still design-only. No hardware has been acquired yet, see
[`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md). The Nostr relay bridge is specified in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) but not yet implemented in `core/`.

Contributions to the protocol, the cryptography, and the research are welcome. See
[`docs/GOVERNANCE.md`](docs/GOVERNANCE.md).

## Repository layout

```
project-mesh/
├── WHITEPAPER.md, README.md    : design docs (see table above)
├── LICENSE                     : AGPL-3.0-or-later
├── docs/                       : specifications, live status, progress log, and this site
└── core/                       : mesh-core, the shared Rust core
    ├── fuzz/                   : cargo-fuzz targets for every wire-format parser
    └── src/
        ├── identity.rs           : Ed25519 + X25519 identity, fingerprint
        ├── crypto/                : Noise XX, Double Ratchet, X3DH, PQXDH, Channels, padding
        ├── groups.rs              : MLS groups (RFC 9420, via openmls)
        ├── envelope.rs            : wire format, content-derived envelope IDs
        ├── bloom.rs                : Bloom filter (compact gossip summary vectors)
        ├── engine.rs               : in-memory store, dedup, TTL, priority eviction
        ├── persistence.rs          : encrypted-at-rest envelope store (redb + AEAD)
        ├── durable.rs               : engine.rs + persistence.rs wired together (restart-safe)
        ├── puzzle.rs                : client puzzle (optional proof-of-work anti-flood)
        ├── relay.rs                 : mesh engine loop, gossip, relay, rate limiting, puzzle
        ├── dtn_sim.rs                : DTN simulation harness (scripted contact schedules)
        ├── transport.rs              : radio abstraction trait
        ├── ffi.rs, ffi_groups.rs,     : UniFFI-exported surface: identity, crypto, MLS,
        │   ffi_transport.rs,         :   transport callbacks, prekey/PQXDH, mesh node
        │   ffi_node.rs, ffi_prekey.rs
```

The Android app (`android/`) lived here until 2026-07-29. It is now
[konkomaji/betar](https://github.com/konkomaji/betar), consuming this `core/` as a submodule.

## Building

1. `cargo test` runs the full Rust test suite (191 tests) for `mesh-core`.
2. `cargo run --release --bin uniffi-bindgen -- generate --library target/release/mesh_core.dll --language kotlin --out-dir core/generated` regenerates the Kotlin bindings.

`rust-toolchain.toml` pins the exact Rust version this project builds with. No Android SDK or
NDK is required to build or test `core/`. It is a plain Rust crate. See
[`docs/REPRODUCIBLE-BUILD.md`](docs/REPRODUCIBLE-BUILD.md) for the full pinned toolchain and what
reproducibility has actually been verified so far: same-machine, not yet cross-machine. To build
the Android app itself, see [konkomaji/betar](https://github.com/konkomaji/betar)'s own README.

## Distribution

Betar ships first through F-Droid and direct APK download, not an app store. App stores are a
single point of control too: Apple and Google have both already removed Bitchat from their
stores in specific countries at government request. A direct APK cannot be pulled the same way.
Store listings may follow later as a convenience channel, never as the only path. See
[`docs/DISTRIBUTION.md`](docs/DISTRIBUTION.md).

## Licence

Code: **[AGPL-3.0-or-later](LICENSE)** (copyleft, keeps forks open; see
[`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) for the rationale). Documentation: **CC BY-SA 4.0**.

## A note on scope and honesty

I write this documentation to be explicit about what is hard or impossible: iOS background
Bluetooth limits, the need for physical radio hardware to cover long distances, the fact that a
pure mesh-local user without ever reaching a relay stays limited to local reach and thumbnail
images, and the security properties the system does not provide. Building a communication tool
that people's safety can depend on is only worth doing if the claims about it are true. Where
something is aspirational rather than proven, I label it as such.

**Developed by [Konko Maji](https://konkomaji.com/)**
