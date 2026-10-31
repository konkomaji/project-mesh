# Roadmap

I wrote this roadmap as a companion to `WHITEPAPER.md` §16. I sequence phases so the project
delivers real value early, with phones only, before it takes on new hardware (LoRa). I leave out
timelines on purpose. This is a project I build alone and with occasional contributors, so
sequencing and exit criteria matter more than dates.

**2026-10-31 update: I pivoted the product.** Betar is no longer a disaster-response safety app.
It is a censorship-resistant mesh social and chat platform, inspired by Bitchat. I removed every
disaster-specific feature (SOS, the disaster bulletin board, the resource board, offline maps)
from scope. They are not repurposed, they are gone. The Rust core, the crypto stack, and the mesh
transport stay exactly as they were. Only the app-level feature set and framing changed.

---

## Phase 0: Design and specification *(complete)*

- I wrote the full documentation set: white paper, architecture, transport, routing,
  cryptography, threat model, features, UX, hardware, distribution, governance, compliance.
- I froze the core protocol decisions for v1: the crypto suite and the envelope format.

**Exit criteria met:** the design docs exist. The crypto suite and envelope format are frozen.

## Phase 1: Core engine and chat, Android first *(in progress)*

**What I already built and reuse without change:**
- **Rust core:** identity, Noise XX, Double Ratchet, X3DH and PQXDH async bootstrap, MLS groups
  (RFC 9420), passphrase channels, the envelope format, the store-carry-forward engine, dedup,
  TTL, and encryption at rest. 191 tests pass. See `docs/IMPLEMENTATION-STATUS.md` for the exact
  state of every component.
- **Android app:** Kotlin and Compose UI, a real BLE GATT driver, a real Wi-Fi Direct driver, and
  a foreground-service relay. OEM battery-whitelist guidance for the device vendors that need it.
- **Messaging:** Direct, Group, Channel, and a Broadcast primitive I built for the app's earlier
  features. I reuse Broadcast as a starting point for Feed posts in Phase 1 below, though posts
  need new work on top of it (signing, geohash tagging, image references).
- **Localization foundation:** the framework exists. English ships today. Bengali is paused, not
  deleted, see `docs/PROGRESS.md` for why.
- **Distribution:** F-Droid and a signed direct APK. App stores are not the primary channel,
  since a store listing is itself a ban chokepoint a government can pressure.
- **Testing:** a DTN simulation harness, a wire-parser fuzzing harness, and on-device tests on
  low-end hardware.

**What the pivot adds, not yet built:**
- **Feed, as geohash-scoped rooms.** Posts carry a geohash tag at a precision the user picks,
  from a city block up to a country. This is Bitchat's own pattern, not an invention of mine.
- **Nostr relay bridge.** Any device in a local mesh cluster that reaches the internet, even
  briefly, publishes the cluster's posts to the public Nostr relay network and pulls down
  everything from it. I chose real Nostr over a custom protocol because Bitchat already runs on
  it at scale, and reusing it gives Betar global reach from day one.
- **Community Notes-style fact-check.** Any user attaches a note to a post. The scoring
  algorithm is public and deterministic, so any client or relay computes the same score from the
  same data. No server holds the only verdict.
- **Images in posts.** A small, lossy, EXIF-stripped thumbnail travels over the raw mesh. The
  full-resolution image uploads to a content-addressed blob store only once a relay bridge
  exists. A mesh-only user who never reaches a relay stays at thumbnail quality. I state this
  limit here plainly, I do not hide it.
- **Location sharing, one-shot only.** A user can share their location once inside a specific
  chat, through that chat's own encrypted channel. Continuous live tracking is not in this phase.
  It is the single highest-risk feature for a journalist or an activist, and it needs its own
  safety design before I build it.
- **Client-side content filtering.** No server or relay can delete a post. A user's own client
  can locally hide or warn on a post that the Community Notes layer has flagged, using the same
  web-of-trust data as the fact-check feature.

**Exit criteria:** a reliable Android-to-Android mesh carrying the full new feature set, with
forward-secret messaging and a green reproducible build on F-Droid.

## Phase 2: Reach and hardening

> **2026-07-27 decision, still standing:** I paused iOS. I deprioritized it to last among the
> remaining phases. I did not drop it. I started Phase 3 (LoRa) planning ahead of it instead, see
> `docs/HARDWARE-LORA.md` §8. This is a deliberate reordering of this roadmap's stated Phase 2 to
> Phase 3 sequence, and I track it here rather than silently departing from the original order.

- **iOS app:** Swift and SwiftUI, a CoreBluetooth driver, MultipeerConnectivity for iOS
  clusters. I use an honest foreground relay-mode model here. I do not claim background relay
  that does not exist.
- I validate Android to iOS BLE bridging.
- I add more languages and tune performance and battery across device classes.
- **Onion routing (Sphinx), as a fallback only.** It engages when a device has internet
  connectivity and traffic would otherwise leave the offline mesh, or when a connection has been
  exposed and needs an anonymity layer on top. It is not the default path for every message.
- **Independent external security audit.** This is a hard gate before any general-availability
  release, not a nice-to-have.

**Exit criteria:** a cross-platform mesh with audit findings resolved and acceptable performance
on target low-end devices.

## Phase 3: Hybrid, LoRa backbone

- A LoRa companion-node bridge in the IN865 (865 to 868 MHz) band, built to interoperate with or
  extend Meshtastic firmware.
- A reference solar relay-node design, with a bill of materials and a build guide.
- An open contribution model for community node builders, with conservative duty-cycle defaults
  for band compliance.
- Validated phone-mesh to LoRa-mesh bridging across representative terrain.

**Exit criteria:** a community can stand up a city or valley backbone from documented kits, and
small high-priority messages cross long distances reliably within band limits.

## Phase 4: Ecosystem

- Community node networks in pilot regions, with digital-rights and independent-journalism
  partners rather than disaster-response partners, matching the pivot.
- Governance maturity: a non-profit structure, funding, a contributor community, a translation
  community.
- A reproducible-build verification community and an ongoing security review cadence.
- A published field evaluation: delivery ratio and latency against density, LoRa range and
  throughput, and battery cost, measured, not asserted.

**Exit criteria:** sustained, community-maintained deployments and a real evidence base for the
system's performance.

---

## Cross-phase invariants

- **Outage and shutdown resilience exist from Phase 1,** as a direct consequence of the design.
  The mesh needs no cell tower, no internet provider, and no central server to function, see
  `docs/COMPLIANCE.md`.
- **No claim ships unproven.** Every "yes" in the comparison table in `WHITEPAPER.md` §14
  becomes true only once I implement it, and for security claims, only once it is audited.
- **De-Googled and offline-first from the first build.**

---

Developed by [Konko Maji](https://konkomaji.com/).
