<div align="center">
<img src="docs/assets/mesh-logo.svg" width="108" height="108" alt="Project Mesh">

# Project Mesh

**An open protocol for communication that doesn't need a network.**

*Decentralized, delay-tolerant mesh communication over Bluetooth LE and Wi-Fi Direct,*
*with LoRa planned as a longer-range backbone.*

[![Core tests](https://img.shields.io/badge/core%20tests-191%20passing-brightgreen)](core/)
[![Code licence](https://img.shields.io/badge/code%20licence-AGPL--3.0--or--later-lightgrey)](LICENSE)
[![Docs licence](https://img.shields.io/badge/docs%20licence-CC%20BY--SA%204.0-lightgrey)](docs/GOVERNANCE.md)
[![Status](https://img.shields.io/badge/status-pre--alpha-orange)](docs/IMPLEMENTATION-STATUS.md)

**[Read the whitepaper](WHITEPAPER.md)** &middot; **[konkomaji.github.io/project-mesh](https://konkomaji.github.io/project-mesh/)**

</div>

---

Project Mesh is a decentralized, server-less, end-to-end encrypted protocol and Rust core for
communication that keeps working when centralized infrastructure is absent, destroyed, or
unavailable. Phones (and, in a later phase, low-cost long-range LoRa radio nodes) relay messages
directly for each other: **no cell tower, no internet service provider, no central server, and
no personal identifier.**

This repository holds the protocol, the research, and the Rust core (`core/`) that implements
both. **[Betar](https://github.com/konkomaji/betar)** is the reference Android application
built on top of it — the thing most people will actually install — and depends on this repo's
`core/` as a git submodule, so the protocol has one canonical source regardless of how many
applications end up built on it.

> Community-maintained and open source. No company, no developer byline: the source is the
> attribution.

---

### ✨ At a glance

| | |
|---|---|
| 🔐 **Crypto** | Noise `XX` → Double Ratchet · X3DH + hybrid post-quantum PQXDH (ML-KEM-1024) for offline first contact · MLS groups (RFC 9420) · Argon2id passphrase Channels · envelope-size padding against traffic analysis |
| 📡 **Transport** | BLE dual-role GATT + Wi-Fi Direct today, composed behind one mesh engine · store-carry-forward gossip, TTL, rate limiting, a client puzzle · LoRa (865–868 MHz, India's licence-free band) planned as a longer-range backbone |
| 🧪 **Confidence** | 191 Rust tests · a DTN simulation harness that found a real relay bug · a wire-parser fuzzing harness · a security-review pass |
| 🚫 **What it isn't** | No server, no account, no cloud, no independent security audit yet, see [`docs/THREAT-MODEL.md`](docs/THREAT-MODEL.md) |

---

## 🌩️ Why this exists

| Problem | Reality |
|---|---|
| 📡 **Coverage gaps** | Hill country, islands, forest and river regions, high valleys: weak, intermittent, or absent cellular and fibre coverage |
| 🌊 **Disasters** | Cyclones, floods, and earthquakes routinely destroy communication infrastructure exactly when people need it most, see [`docs/REFERENCES.md`](docs/REFERENCES.md) |
| 🔌 **Any outage** | Towers go down, for whatever reason, for as long as it lasts |

All three share one root cause: **communication that depends on centralized infrastructure fails
when that infrastructure is absent, destroyed, or unavailable.** Project Mesh removes the
dependency by design: no node is a hub, any node can relay for any other, and the network
survives losing any one of them.

## 📚 Documentation map

| Document | What it covers |
|---|---|
| 📖 [`WHITEPAPER.md`](WHITEPAPER.md) | The full research paper: problem, design, protocol, evaluation |
| 📊 [`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md) | **Live status**: what's actually built vs. still design-only, per component |
| 📅 [`docs/PROGRESS.md`](docs/PROGRESS.md) | **Running log**: dated, one entry per work session, what happened and why |
| 🔍 [`docs/RESEARCH-FINDINGS.md`](docs/RESEARCH-FINDINGS.md) | **Verified research + corrections** (deep multi-source, adversarially checked): read this for what changed and why |
| 🔤 [`docs/BETAR-TRANSITION.md`](docs/BETAR-TRANSITION.md) | The rename/repo-split tracker: what's decided, what's left, in what order |
| 🏗️ [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | System layers, Rust core, module boundaries |
| 📶 [`docs/TRANSPORT.md`](docs/TRANSPORT.md) | BLE mesh, Wi-Fi Direct, LoRa; iOS/Android radio realities |
| 🔀 [`docs/ROUTING-PROTOCOL.md`](docs/ROUTING-PROTOCOL.md) | Store-carry-forward routing, packet format, dedup, TTL |
| 🔐 [`docs/CRYPTOGRAPHY.md`](docs/CRYPTOGRAPHY.md) | Identity, Noise handshake, Double Ratchet, MLS, PQXDH, onion routing |
| 🛡️ [`docs/THREAT-MODEL.md`](docs/THREAT-MODEL.md) | Adversaries, assets, attacks, mitigations, non-goals |
| 📻 [`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md) | LoRa gateway spec, the 865 to 868 MHz band, node design |
| 📦 [`docs/DISTRIBUTION.md`](docs/DISTRIBUTION.md) | De-Googled builds, F-Droid, reproducibility |
| 🔁 [`docs/REPRODUCIBLE-BUILD.md`](docs/REPRODUCIBLE-BUILD.md) | Pinned toolchain, build steps, what's actually verified reproducible |
| ⚖️ [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) | Licence, community stewardship, contribution model |
| 📜 [`docs/COMPLIANCE.md`](docs/COMPLIANCE.md) | Spectrum compliance, licence, no-warranty position |
| 🗺️ [`docs/ROADMAP.md`](docs/ROADMAP.md) | Phased delivery plan and milestones |
| 📎 [`docs/REFERENCES.md`](docs/REFERENCES.md) | Citations, prior art, standards, further reading |

Docs written while the Android app still lived in this mono-repo (e.g.
[`docs/DESIGN-BRIEF.md`](docs/DESIGN-BRIEF.md), [`docs/LOCALIZATION-UX.md`](docs/LOCALIZATION-UX.md),
[`docs/FEATURES.md`](docs/FEATURES.md)) describe the app, not the protocol — kept here for now
since the split just happened, not yet moved to [Betar's own repo](https://github.com/konkomaji/betar).

## 📈 Status

**Pre-alpha.** Full detail, component-by-component, in
[`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md), this is the short version.

**Rust core (`core/`, 191 tests passing):** identity (Ed25519 + X25519), Noise `XX` handshake →
Double Ratchet for 1:1 messaging, X3DH and hybrid post-quantum PQXDH (ML-KEM-1024 + classical,
via the `ml-kem` crate) for async first-contact bootstrap when both parties are never online at
once, MLS groups (RFC 9420, via `openmls`) for larger groups, Argon2id passphrase-derived
Channels, envelope wire format + Bloom-filter gossip summaries, a store-carry-forward mesh engine
(gossip-on-contact, epidemic relay, TTL, rate limiting, an optional client puzzle), envelope-size
padding (metadata protection against relay traffic analysis) wired into every addressing mode,
encryption-at-rest (`redb` + AEAD, a deliberate substitution for the design docs' SQLCipher,
whose OpenSSL dependency wouldn't build in this dev environment), and a full UniFFI surface
covering all of it. A [DTN simulation harness](core/src/dtn_sim.rs) drives the real relay engine
through scripted contact schedules with no hardware needed: building it surfaced and fixed a real
TTL-decrement gap in the gossip-relay path. A [wire-parser fuzzing harness](core/fuzz/) covers
every untrusted-bytes-in parser (can't execute on this Windows dev box, a documented MSVC/`cdylib`
linker conflict, needs Linux/CI).

**The Android app moved to its own repository, [konkomaji/betar](https://github.com/konkomaji/betar),
on 2026-07-29.** It depends on this repo's `core/` as a git submodule. What's built, what's
released, and how to install it now lives in that repo's own README and release notes, not here.

**What's not done yet in the protocol itself:** no independent security audit (a hard release
gate, not attempted here, see [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) §3); the wire-parser
fuzzing harness has never actually run (Windows/MSVC linker blocker, needs Linux/CI); LoRa is
still design-only, no hardware acquired yet (see [`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md)).

Contributions to the protocol, the cryptography, and the research are welcome, see
[`docs/GOVERNANCE.md`](docs/GOVERNANCE.md).

## 🗂️ Repository layout

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

The Android app (`android/`) lived here until 2026-07-29; it's now
[konkomaji/betar](https://github.com/konkomaji/betar), consuming this `core/` as a submodule.

## 🛠️ Building

```sh
cargo test                       # mesh-core: run the full Rust test suite (191 tests)
cargo run --release --bin uniffi-bindgen -- generate --library target/release/mesh_core.dll \
  --language kotlin --out-dir core/generated                # regenerate Kotlin bindings
```

`rust-toolchain.toml` pins the exact Rust version this project builds with. No Android SDK/NDK is
required to build or test `core/`, it's a plain Rust crate. See
[`docs/REPRODUCIBLE-BUILD.md`](docs/REPRODUCIBLE-BUILD.md) for the full pinned toolchain and what
reproducibility has actually been verified (same-machine, not yet cross-machine). To build the
Android app itself, see [konkomaji/betar](https://github.com/konkomaji/betar)'s own README.

## ⚖️ Licence

Code: **[AGPL-3.0-or-later](LICENSE)** (copyleft, keeps forks open, see
[`docs/GOVERNANCE.md`](docs/GOVERNANCE.md) for the rationale). Documentation: **CC BY-SA 4.0**.

## 🕊️ A note on scope and honesty

This documentation is deliberately explicit about what is **hard or impossible**: iOS
background Bluetooth limits, the need for physical radio hardware to cover long distances, and
the security properties the system does **not** provide. Building resilient communication tools
is only worthwhile if the claims are true. Where something is aspirational rather than proven,
it is labelled as such.
