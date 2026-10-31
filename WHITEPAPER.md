# Project Mesh: A Decentralized Protocol for Communication No One Can Switch Off

**A technical white paper**

Project Mesh is the protocol and the Rust core this paper describes. The shipping consumer app
built on it is called **Betar**; see [`docs/DESIGN-BRIEF.md`](docs/DESIGN-BRIEF.md) for the
app's own identity and design.

**Developed by [Konko Maji](https://konkomaji.com/).**

Status: Phase 1 (core, Android-first) implemented and released, pre-alpha, no independent
security audit yet. This document was written at the design stage; §5 to §9 now describe a real,
tested implementation, not a plan. Live status: [`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md).
Licence: CC BY-SA 4.0 (this document); the code is AGPL-3.0-or-later (`LICENSE`).

---

## Abstract

A communication platform that depends on a server can be subpoenaed, throttled, or shut down.
I did not write that sentence as a hypothetical. In 2026 alone: Iran had an 88-day nationwide
internet shutdown, the longest Freedom House has recorded. France's Prime Minister asked the
country's four mobile carriers to be ready to block four social platforms for several hours.
India's Home Ministry ordered GitHub to restrict three repositories belonging to Bitchat, a
Bluetooth-mesh messenger, arguing its anonymous, decentralized design obstructs lawful
interception. This paper presents **Project Mesh**, a decentralized, server-less, end-to-end
encrypted protocol for a social feed and messaging platform that keeps working no matter who
tries to shut it off. Project Mesh forms an ad-hoc **delay-tolerant mesh network** directly
between users' phones over Bluetooth Low Energy and Wi-Fi Direct, bridges to the open **Nostr**
relay network the moment any device in a local cluster reaches the internet, and, in a later
phase, extends over a backbone of low-cost long-range **LoRa** radio nodes. It requires no cell
tower, no internet service provider, no central server, and no personal identifier (phone number
or email). I describe the system's design goals, layered architecture, transport strategy,
store-carry-forward routing protocol, cryptographic design, the application-level feature set
(a geohash-scoped social feed, direct/group/channel chat, Community Notes-style fact-checking
with client-side illegal-content filtering, and one-shot location sharing), localization and
accessibility strategy, distribution as a fully de-Googled, F-Droid-first application, an open
governance model, and a compliance position. I am explicit throughout about the system's
fundamental limitations, most importantly the severe constraints Apple's iOS places on
background Bluetooth operation, the physical necessity of dedicated radio hardware to cover long
distances, and the security properties the system deliberately does and does not provide.

---

## Table of contents

1. Introduction
2. Background and related work
3. Design goals and non-goals
4. Threat model (summary)
5. System architecture
6. Transport layer
7. Routing protocol
8. Cryptographic design
9. Application features
10. Localization, accessibility, and user experience
11. Distribution and the de-Googled requirement
12. Governance, licensing, and sustainability
13. Compliance
14. Evaluation and comparison with prior art
15. Limitations and open problems
16. Roadmap
17. Conclusion
18. References

Detailed protocol-level specifications for several sections live in the `docs/` directory and
are cross-referenced from here.

---

## 1. Introduction

### 1.1 The problem

I pivoted this project in October 2026, from a disaster-response safety app to what it is now:
a censorship-resistant social feed and chat platform. The reason is not abstract. Governments
and platform operators can and do remove apps, ban accounts, and shut off networks, in 2026,
not in some earlier, more repressive era. A platform that routes through a server has exactly
one thing in common across every failure mode: the server. Subpoena it, throttle it, pressure
the company that runs it, or order it blocked at the network level, and the platform stops
working for everyone downstream of that single point. This is true whether the platform is a
household-name social network or a small messaging app. India's order against Bitchat in 2026
makes the point concretely: a government did not need to find and remove individual messages,
it targeted the GitHub repositories the app's distribution depended on.

### 1.2 The approach

Project Mesh removes the server from the picture entirely. Instead of routing messages and
posts through a tower and a central service, it treats every participating device as a node
that stores, carries, and forwards content on behalf of the network. Two phones within radio
range exchange content directly; content addressed to someone out of range is carried by
intermediate devices until it reaches its destination, or is relayed onward through the open
Nostr network the moment any device in the cluster reaches the internet. No node is privileged;
there is no centre to fail, subpoena, or switch off. This is the **delay-tolerant networking
(DTN)** paradigm, adapted to consumer smartphones today and, later, to inexpensive long-range
radio nodes.

### 1.3 Contributions

This paper contributes:

- a coherent **system design** for a censorship-resistant social and messaging platform
  spanning phone-to-phone short-range links, an internet relay bridge, and a long-range LoRa
  backbone;
- an honest, engineering-first treatment of the **hard constraints** (iOS background
  Bluetooth, radio range, battery, low-end devices) that prior consumer mesh apps have tended
  to understate;
- a **cryptographic and routing design** that provides confidentiality, forward secrecy, and
  meaningful metadata protection without any server or registry;
- a **content and moderation model**, Community Notes-style fact-checking plus client-side
  illegal-content filtering, that works without a central moderator or a server that could be
  ordered to remove anything; and
- a **distribution strategy** that treats app stores themselves as a censorship surface, not
  a neutral channel, grounded in a real 2026 precedent rather than a hypothetical one.

### 1.4 What this project is not

Project Mesh is communication infrastructure whose resilience is uniform: it does not know or
care *why* a network is unavailable or *who* is trying to make it unavailable, only that
content keeps moving regardless. This is a sincere description of the design, not a slogan,
and I ask every contributor to keep public materials in the same plain, calm framing
(`docs/GOVERNANCE.md` §4).

---

## 2. Background and related work

### 2.1 Delay-tolerant and mesh networking

The intellectual foundations are **delay-tolerant networking** (Fall, 2003) and
**store-carry-forward** routing for intermittently connected networks. Classic DTN routing
schemes relevant here include **Epidemic routing** (Vahdat & Becker, 2000), **Spray-and-Wait**
(Spyropoulos et al., 2005), and **PRoPHET** (Lindgren et al.). These trade bandwidth and
storage for delivery probability in networks with no stable end-to-end path, exactly the
regime Project Mesh operates in.

### 2.2 Consumer mesh and censorship-resistant messaging apps

I researched six comparable projects before finalizing this pivot (full method and sourcing in
[`docs/RESEARCH-FINDINGS.md`](docs/RESEARCH-FINDINGS.md) §10). What each one contributed to
this design, and what I deliberately did not take from it:

- **Bitchat** (`permissionlesstech/bitchat`), the project's stated direct inspiration, bridges
  local BLE mesh (roughly 30 m per hop, multi-hop to roughly 300 m) to the public **Nostr**
  relay network the moment any device reaches the internet, with geohash-scoped public rooms.
  I took the entire three-tier shape, local mesh, relay bridge, geohash rooms, directly from
  it; Betar's Feed is this pattern, not an original design. It is actively maintained and
  audited in part by Cure53.
- **Briar** uses Tor as its default route with Bluetooth and Wi-Fi as a blackout fallback, and
  has forums and personal blogs. It confirms the forum/blog shape Betar's Channels and Feed
  already cover. I did not adopt Tor-as-default, since Project Mesh is mesh-primary for the
  opposite reason: it works with zero infrastructure first, and bridges to the internet
  second, rather than the other way around.
- **FireChat** (2014 Hong Kong protests) was pure BLE mesh with no relay fallback,
  unencrypted, and encouraged real names. A retrospective fact-check (Global Voices) found
  real mesh usage was lower than contemporary press coverage implied, and that most protesters
  defaulted to WhatsApp whenever any internet existed at all. This is the single clearest
  lesson behind Project Mesh's hybrid design: a mesh-only app will not be anyone's daily
  driver, only a true-outage fallback, so the relay bridge is not optional polish, it is why
  people will actually use this day to day. Encryption is mandatory everywhere in this design,
  with no exception, directly in response to FireChat's most-cited flaw.
- **Session** routes through 2,000-plus independent service nodes with onion routing and a
  keypair-derived ID, no phone number. It confirms the onion-routing-as-fallback design this
  project had already scoped independently (§4, §8.3). I did not adopt Session's own node
  network, and kept this project's existing Noise/MLS/PQXDH stack.
- **Nostr**, a dumb-relay pub/sub network reported at over 140 clients and 850 relays across
  40-plus countries, is adopted directly as the relay-bridge protocol in §6 and §7, revising an
  earlier plan to hand-roll a custom gossip layer. Reusing an already-live network means reach
  exists from day one, with no new infrastructure required.
- **Farcaster** tied its identity to an on-chain registration on the Optimism blockchain.
  Ownership of the protocol itself changed hands at least twice within under a year, and its
  feed was criticized for NFT and token-promotion spam after a reported pivot away from a
  social-first vision. This is a cautionary case, not a pattern: a blockchain or its operator
  is one more party that can be bought, sold, or regulated, which is exactly the kind of
  chokepoint this project exists to remove. No on-chain or blockchain-based identity appears
  anywhere in this design.
- **Bridgefy** was shown to be insecure (Albrecht et al., 2021, *"Mesh Messaging in Large-Scale
  Protests: Breaking Bridgefy"*), a cautionary example on cryptography, not a model.
- **Meshtastic** is the primary positive reference for the long-range LoRa backbone (Phase 3,
  §6.3), an open-source LoRa mesh network over inexpensive ESP32 hardware.

### 2.3 Supporting technology

- **The Noise Protocol Framework** (Perrin) for authenticated key exchange.
- **The Signal Double Ratchet** (Marlinspike & Perrin) for forward-secret, self-healing session
  encryption.
- **MLS, Messaging Layer Security** (RFC 9420) for efficient group key management.
- **Sphinx** (Danezis & Goldberg, 2009) and mix-network designs (Loopix; Nym) for
  metadata-protecting onion routing, used as an internet-connectivity fallback layer, not the
  primary path (`docs/THREAT-MODEL.md` §4).
- **Nostr** for the open, already-live relay network this project bridges to rather than
  reimplementing.
- **Rust** with **UniFFI** (Mozilla) for a single, memory-safe core shared across native
  Android and iOS front-ends.

### 2.4 Where Project Mesh differs

No existing project simultaneously offers: (a) a security-first design, (b) a hybrid
phone-plus-internet-relay-plus-LoRa transport, (c) a social feed and fact-checking layer with
no central moderator and no server that can be ordered to remove content, (d) first-class
multilingual accessibility, and (e) a fully de-Googled, store-independent distribution model
treated as a first-class requirement rather than an afterthought. Project Mesh targets that
combination.

---

## 3. Design goals and non-goals

### 3.1 Goals

- **G1: Infrastructure independence.** Core functions must work with zero internet, zero
  cellular service, and zero central server.
- **G2: Decentralization.** No node is required; no registry, directory, or coordinator
  exists. The network degrades gracefully as nodes join or leave.
- **G3: Security by default.** End-to-end encryption everywhere, including feed posts, forward
  secrecy, and no personal identifiers, with no configuration required from the user and no
  unencrypted mode.
- **G4: Metadata minimization.** Reduce what a passive observer or a relay can learn about who
  is talking to whom.
- **G5: No removable chokepoint.** Content, once posted, cannot be deleted by any single
  server, relay, or moderator. Illegal content is handled through client-side filtering
  instead (§9, §13), not a central takedown mechanism, because no central mechanism exists.
- **G6: Accessibility.** Usable by users with varying literacy, in multiple languages, on
  inexpensive phones with limited RAM, storage, and battery.
- **G7: De-Googled and libre.** No Google Play Services, no Firebase, no proprietary cloud
  dependency; distributable via F-Droid and direct download first, open source.
- **G8: Honesty.** Ship no claim I cannot substantiate; document every limitation.

### 3.2 Non-goals

- **N1: Anonymity against a global passive adversary.** This design provides meaningful
  metadata protection, not guarantees against an adversary who can observe the entire radio
  environment.
- **N2: Real-time, high-bandwidth media.** This is a store-carry-forward text-and-small-payload
  network, not a video-calling platform.
- **N3: Seamless iOS background operation.** Apple's platform constraints make this
  impossible; I design *around* it rather than pretending to solve it (§6).
- **N4: Guaranteed delivery.** Delivery is best-effort and probabilistic, as is inherent to
  DTN.
- **N5: Blockchain or on-chain identity.** Rejected explicitly, see §2.2's note on Farcaster.
  A blockchain dependency is one more party that can be captured, regulated, or sold.
- **N6: Interactive post formats or mini-apps.** Rejected explicitly, see §2.2's note on
  Farcaster's Frames. This is a communication platform, not a programmable application host.
- **N7: Algorithmic feed ranking.** The feed is reverse-chronological only. There is no server
  to run a ranking engine on, and building one on-device would contradict G5 and the spirit of
  this project regardless.

---

## 4. Threat model (summary)

The full treatment is in [`docs/THREAT-MODEL.md`](docs/THREAT-MODEL.md). In brief:

**Assets:** message and post content; the social graph (who talks to whom); a user's location
and presence; a user's identity; device data at rest.

**Adversaries considered:**
- a **passive local eavesdropper** sniffing Bluetooth/Wi-Fi/LoRa in radio range;
- a **malicious participant** running a modified node to inject, drop, replay, or flood;
- a **local active attacker** attempting jamming or Sybil flooding;
- a **curious or malicious relay** trying to learn who is talking to whom;
- a **coercive physical adversary** who seizes a device;
- a **government or platform operator** attempting to remove the app from distribution
  channels or pressure a chokepoint that does not exist in this design (§11, §13).

**Explicitly out of scope (non-goals):** a global passive adversary correlating all radio
emissions; endpoint compromise via a device operating-system exploit; and analysis that a
determined nation-state observer with pervasive sensing could perform.

**Core mitigations:** end-to-end authenticated encryption (content confidentiality/integrity);
Double Ratchet and MLS (forward secrecy and post-compromise security); rotating, unlinkable
link-layer identifiers; optional Sphinx onion routing (relationship anonymity within the mesh,
and an internet-connectivity fallback, not the default path); message TTL and deduplication
(flood control); client-side content filtering instead of server-side takedown (§9); and
on-device encryption at rest with a panic-wipe mechanism for the coercive-seizure scenario.

**One deliberate V1 scope limit, stated here because it is a safety decision, not an
oversight:** location sharing is one-shot only. Continuous live-location tracking is the
single highest-risk feature this design could add for a journalist or activist user, and it is
not shipped until it has its own dedicated geofencing and auto-expiry design
(`docs/THREAT-MODEL.md` §1).

---

## 5. System architecture

Project Mesh uses a **shared portable core with thin native front-ends**.

```
        ┌──────────────────────────────┐   ┌──────────────────────────────┐
        │        Android app           │   │           iOS app            │
        │   Kotlin + Jetpack Compose   │   │      Swift + SwiftUI         │
        │  (UI, notifications, radio   │   │  (UI, notifications, radio   │
        │   drivers, foreground svc)   │   │   drivers, CoreBluetooth,    │
        │                              │   │   MultipeerConnectivity)     │
        └───────────────┬──────────────┘   └───────────────┬──────────────┘
                        │        UniFFI bindings            │
        ┌───────────────┴───────────────────────────────────┴──────────────┐
        │                     Shared core  (Rust)                            │
        │  identity · Noise handshake · Double Ratchet · sealing/opening ·  │
        │  packet format · store-carry-forward engine · dedup · TTL/expiry · │
        │  channel/group logic · onion routing · encrypted persistence       │
        └───────────────┬───────────────────────────────────┬──────────────┘
                        │           transport abstraction    │
        ┌───────────────┴──────────┐        ┌────────────────┴─────────────┐
        │  Phone radios (per-OS)    │        │  Nostr relay bridge +         │
        │  BLE GATT · Wi-Fi Direct/ │        │  LoRa bridge (Phase 3)        │
        │  Aware · Multipeer (iOS)  │        │  BLE/USB link to ESP32/LoRa   │
        │                           │        │  node · IN865 865-868 MHz     │
        └───────────────────────────┘        └───────────────────────────────┘
```

### 5.1 The shared Rust core

All security-critical and protocol logic, cryptography, packet construction and parsing, the
routing/relay engine, deduplication, expiry, channel and group semantics, onion routing, and
encrypted persistence, lives in a single **Rust** library. Rationale:

- **Memory safety** for parsing untrusted network input, without a garbage collector. This
  matters specifically because the core parses bytes from strangers over BLE, the classic
  memory-corruption attack surface in exactly this kind of application.
- **One implementation** of the hard logic, shared byte-for-byte across platforms, eliminating
  a whole class of cross-platform divergence bugs and reducing the audit surface to one
  codebase.
- **No Google/Apple runtime dependency** in the core; it is plain portable Rust.
- **UniFFI** generates safe Kotlin and Swift bindings automatically.

The native layer is deliberately a **"dumb byte pipe"**: it advertises, discovers, connects,
and moves opaque byte buffers to and from the core. It contains no cryptography and no routing
decisions.

### 5.2 Why native UI, not a cross-platform runtime

I deliberately **reject** React Native, Expo, Flutter, and similar runtimes for this project,
despite their development-speed appeal, because:

- Radio behaviour (background BLE scanning/advertising, foreground services, OEM battery-manager
  interactions) is exactly where these runtimes are weakest and most fragile, mediated through
  plugins that lag platform changes.
- A large share of the real device population runs inexpensive, heavily OEM-customized Android
  phones (Xiaomi, realme, vivo, Oppo, Samsung budget lines) with aggressive background-process
  killers. Reliable background relay on these devices requires precise native control.
- The security-critical logic is already shared (in Rust); the remaining per-platform work is
  UI and radio, which is precisely what should be native.

Full detail: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

---

## 6. Transport layer

This section summarizes; the authoritative treatment, including the iOS background-BLE analysis,
is in [`docs/TRANSPORT.md`](docs/TRANSPORT.md).

### 6.1 Phone-to-phone links

- **Bluetooth Low Energy (BLE) GATT** is the universal baseline: it is the only transport that
  can bridge **Android ↔ iOS** directly. Each device acts as both a GATT peripheral
  (advertising a Project Mesh service UUID) and a central (scanning and connecting). Payloads
  are fragmented to fit the negotiated MTU and reassembled by the core.
- **Wi-Fi Direct / Wi-Fi Aware (NAN)** on Android provides much higher throughput for larger
  payloads (images) between Android devices.
- **Apple MultipeerConnectivity** provides fast iOS ↔ iOS links using Apple's combined
  Bluetooth/peer-to-peer-Wi-Fi stack, but is Apple-only and cannot bridge to Android.

### 6.2 The iOS constraint

Apple restricts background Bluetooth severely, and no application can remove these limits:

- A backgrounded iOS app **cannot place a custom service UUID in the main advertising packet**;
  the UUID is moved into a special "overflow" area (a hashed bit in a 128-bit bitmask) that is
  only discoverable by a device **explicitly scanning for that specific UUID**. An Android peer
  that knows the Mesh UUID **can** parse this and discover the iOS peripheral, cross-platform
  background discovery is *possible* but higher-latency.
- Overflow advertising is **screen-gated**: it transmits only while the iOS device's screen is
  illuminated. A screen-off backgrounded iOS device goes effectively silent.
- Background scanning is throttled and de-duplicated; since iOS 14 the advertised service set
  cannot be changed while backgrounded.

**Consequence:** iOS devices are strong mesh participants **while the app is in the foreground**,
degraded-but-reachable when backgrounded with the **screen on**, and effectively silent when
backgrounded with the screen off. I do not pretend otherwise. The UX is designed around this:
in gatherings and dead zones, users are guided to keep the app open ("relay mode"), and
iOS-to-iOS clusters use MultipeerConnectivity for much better behaviour. Android devices, which
*can* relay in a foreground service, carry the backbone of the phone-level mesh.

### 6.3 Internet relay bridge

The moment any one device in a local mesh cluster reaches the internet, even briefly, it
bridges that cluster's content to and from the open **Nostr** relay network. This is the second
half of the answer to "this only works when people in my radius are connected": a user never
needs their own device to be online, only to be near someone, at some point, who is. Nostr
relays are simple, independently operated servers; reach is not capped by any single relay's
uptime, since a client can switch relays freely. See §2.2 and §7 for why real Nostr was chosen
over a hand-rolled gossip protocol.

### 6.4 Long-range backbone (Phase 3): LoRa

Phone Bluetooth reaches roughly 10-100 m per hop, which cannot cross long distances without
either internet relay or dedicated hardware. For areas with neither cellular data nor Wi-Fi,
genuine long-range coverage requires **LoRa** radio:

- Long range (kilometres line-of-sight; useful non-line-of-sight range in the hundreds of
  metres to low kilometres), very low power, very low bit-rate.
- In India, LoRa operates in the **licence-free 865-868 MHz ISM band** (the **IN865** region in
  LoRaWAN/Meshtastic terminology), subject to the Indian regulator's power and duty-cycle
  limits, a hard regional design constraint, see [`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md).
  Other regions use different bands and their own local rules.
- Implemented as an inexpensive companion node (ESP32 + LoRa module such as those used by
  Meshtastic hardware) that a phone pairs with over BLE/USB. Community-owned nodes can form a
  local backbone.

The phone mesh and the internet relay bridge together handle the large majority of real-world
cases; the LoRa backbone extends coverage into areas with neither phones in range nor internet
access at all.

---

## 7. Routing protocol

Summary here; wire format and algorithms in
[`docs/ROUTING-PROTOCOL.md`](docs/ROUTING-PROTOCOL.md).

### 7.1 Model

Project Mesh uses **store-carry-forward, epidemic-style dissemination with controls**. Every
node keeps a bounded store of unexpired, sealed **envelopes** it has seen. When two nodes meet,
they exchange compact **summary vectors** (Bloom filters / ID digests) of what they hold and
then transfer only the envelopes the other lacks. This "gossip on contact" approach needs no
addresses, routing tables, or topology knowledge, the correct properties for a network with no
stable paths. This is also the mechanism that defeats the "only people in my radius" limit: a
message does not need sender and recipient to ever share radio range, only a chain of pairwise
range overlaps carried across devices over time.

### 7.2 Controls against the cost of flooding

Naïve epidemic routing is bandwidth- and storage-hungry and trivially floodable. I bound it:

- **TTL / hop limit:** every envelope carries a maximum hop count and an absolute expiry time;
  both are enforced at every node.
- **Deduplication:** each envelope has a content-derived ID; nodes never re-accept or re-forward
  an ID they have already stored (Bloom filter + bounded seen-set).
- **Store bounds and eviction:** a fixed storage budget with priority-aware eviction, so urgent
  traffic is not displaced by bulk feed sync under storage pressure.
- **Rate limiting and proof-of-work-lite:** per-peer transfer rate caps; optionally a small
  client puzzle on envelope creation to raise the cost of mass injection (Sybil/flood
  mitigation).
- **Directionality hints (optional):** where a coarse gradient exists, for example toward a
  known LoRa gateway, spraying can be biased to reduce waste, à la Spray-and-Wait / PRoPHET,
  without requiring full routing state.

### 7.3 Addressing modes

- **Broadcast** ("everyone nearby"): public local content, including feed posts before they
  are tagged to a specific geohash room.
- **Channel:** a passphrase-derived shared key; anyone with the passphrase can read and post; no
  owner or server.
- **Group:** a bounded set of members; the sender seals an individual copy per member.
- **Direct:** one-to-one, end-to-end encrypted between two identities.

---

## 8. Cryptographic design

Summary here; full specification, including handshake transcripts and key schedules, in
[`docs/CRYPTOGRAPHY.md`](docs/CRYPTOGRAPHY.md).

### 8.1 Identity

A user is a **self-generated key pair**, a long-term **Ed25519** signing identity and an
**X25519** key-agreement key, created entirely on-device at first launch. There is **no phone
number, no email, no account, and no registration.** A short, human-friendly ID (for example
`A3F9K2`) is derived as a fingerprint of this key pair, not a separately issued identifier; a
36-character alphanumeric space is not large enough to be collision-free if issued
independently at scale, so it must be a derivation of the already-unique key, not a new source
of uniqueness. A user may additionally set a cosmetic display name, shown alongside the ID,
explicitly not guaranteed unique, since no server exists to check for collisions. A user's
public identity fingerprint can also be shared out-of-band (QR code shown screen-to-screen) for
in-person verification. Trust is rooted in human verification, never in a server.

### 8.2 Session establishment and message encryption

- **Handshake:** the **Noise Protocol Framework** (the `XX` pattern for mutual authentication
  with identity hiding), using **X25519** for key agreement, **ChaCha20-Poly1305** for AEAD, and
  **BLAKE2s/SHA-256** for hashing.
- **Ongoing 1:1 messaging:** the **Signal Double Ratchet**, providing **forward secrecy** (past
  messages stay safe if a current key leaks) and **post-compromise security** (the session
  self-heals after a key compromise).
- **Groups:** the Double Ratchet is fundamentally two-party. Sealing one copy per member is O(N)
  and only acceptable for tiny groups; larger groups use **MLS (Messaging Layer Security, RFC
  9420)**, whose TreeKEM gives forward secrecy, post-compromise security, and **logarithmic**
  (not O(N²)) group key-update cost.
- **Post-quantum:** the handshake adds **PQXDH** (a post-quantum KEM, ML-KEM-1024 class, run
  alongside X25519) to defend against passive "harvest-now, decrypt-later" quantum attacks,
  important because store-and-forward envelopes are long-lived. It does **not** stop an active
  quantum attacker.
- **Channels:** symmetric keys derived from a passphrase using a memory-hard KDF (**Argon2id**),
  with per-message nonces and AEAD sealing.
- **Feed posts and Community Notes:** signed events, gossiped the same way as any other
  envelope. A post's or note's signature lets any recipient verify who signed it without
  trusting whichever relay or peer happened to deliver it.

### 8.3 Metadata protection

- **Rotating link identifiers:** BLE MAC/identifier randomization and periodic rotation of any
  advertised ephemeral identifier (on the order of every ~15 minutes), from a CSPRNG, so that a
  device cannot be trivially tracked across time and place by its link-layer address.
- **Fixed-size sealed envelopes:** padding envelopes to size buckets so that length does not
  leak message type.
- **Optional onion routing:** for direct messages, a **Sphinx-style** layered-encryption packet
  lets the sender route through several relays such that no single relay learns both source and
  destination, providing **relationship anonymity within the mesh** (subject to the non-goals
  in §3.2 and §4). This is an internet-connectivity fallback layer, engaged when a connection
  has been exposed or bridged across the internet, not the default path for every message.

### 8.4 At rest

All local storage (messages, posts, keys, contacts) is encrypted at rest with AEAD
(ChaCha20-Poly1305 over **redb**, a pure-Rust embedded store, a deliberate deviation from an
earlier SQLCipher plan, since `rusqlite`'s SQLCipher backend needs an OpenSSL build this
project's toolchain could not produce; same encrypted-at-rest security property, different
engine, see `docs/PROGRESS.md`), with the database key protected by the device keystore / Secure
Enclave where available (Android Keystore-wrapped identity, prekey pool, and MLS group state are
implemented; iOS Secure Enclave equivalent is Phase 2), and a **panic-wipe** mechanism that
erases everything on the device and retires the identity completely, so a fresh launch
afterward generates a brand new key pair and ID from nothing.

---

## 9. Application features

Project Mesh is the protocol underneath a social feed and messaging platform, not only a chat
transport. All features run on the one mesh substrate. Full detail in
[`docs/FEATURES.md`](docs/FEATURES.md).

- **Feed, as geohash-scoped public rooms.** A post's room runs from a city block up to a
  country in precision, the pattern taken directly from Bitchat (§2.2). Posts reach local
  phones immediately over the mesh, and reach the wider room the moment any device nearby
  bridges to the Nostr relay network. The feed is reverse-chronological only; there is no
  server to run an engagement-ranking algorithm on, and I decided against building one even
  where it would be technically possible (G7, N7).
- **Chat.** Direct messages (Noise XX into a Double Ratchet, X3DH plus PQXDH for first
  contact), groups (MLS), and passphrase-derived channels (Argon2id), all end-to-end encrypted
  with no exception.
- **Community Notes-style fact-checking.** Any user can attach a signed note to a post. The
  scoring algorithm that decides which notes surface is public and deterministic; any client
  or relay computes the same result locally from the same synced events, so no single scorer
  is authoritative.
- **Client-side illegal-content filtering.** No server or relay in this design can delete a
  post once it has spread. Instead, a reported or flagged post can be hidden or warned-on by
  each user's own client, driven by the same web-of-trust scoring as fact-checking. I chose
  this over full server-side moderation, impossible without a server to run it on, and over no
  moderation at all, Bitchat's current approach, which led to a government ordering its
  distribution restricted rather than any individual post removed (§2.2, §11, §13).
- **Images.** A post's image attaches as a small, lossy, EXIF-stripped thumbnail over the mesh.
  The full-resolution file uploads only once a relay bridge exists, to a content-addressed
  blob store (the Nostr ecosystem's Blossom protocol, or an equivalent), so no single host is
  required. A mesh-only post stays at thumbnail quality until a relay bridge reaches it.
- **Location sharing, one-shot only.** Sent once, inside a specific chat thread's existing
  encrypted channel. Continuous live tracking is deliberately not built in V1 (§4, N7, and
  `docs/THREAT-MODEL.md` §1).
- **Identity and panic wipe.** A key-pair identity with a short derived ID (§8.1), a
  user-editable but non-unique display name, and a panic-wipe control that erases everything
  and retires the identity for good.

---

## 10. Localization, accessibility, and user experience

Full detail in [`docs/LOCALIZATION-UX.md`](docs/LOCALIZATION-UX.md). Principles:

- **Multiple languages, done properly.** English and Bengali ship as the default languages,
  using open fonts (Noto Sans and Noto Sans Bengali) with correct script shaping, not merely
  English with a translated string table. Other languages are community-contributed.
- **Accessible by default.** Icon-led navigation and large touch targets, so reading fluency
  is not a hard prerequisite for using the app.
- **Low-end devices.** Target inexpensive Android phones (roughly 2 GB RAM), a small
  application size, and a small persistent footprint; minimize battery drain from scanning and
  relaying, with user-visible controls and clear "relay mode" indication.
- **Offline-first everywhere.** No screen ever blocks on a network call. Every interaction is
  designed for a device that may never touch the internet.

---

## 11. Distribution and the de-Googled requirement

Full detail in [`docs/DISTRIBUTION.md`](docs/DISTRIBUTION.md).

- **No Google Play Services, no Firebase, no proprietary cloud.** Notifications, background
  execution, and all platform integration are implemented without Google's proprietary layer.
  This is both a values requirement and a robustness requirement: the app must run on
  de-Googled and custom-ROM devices and in environments with no Google reachability.
- **F-Droid and direct signed APK are the primary channel, not a backup plan.** This is not a
  theoretical stance. Apple removed Bitchat, the mesh messenger that inspired this project,
  from its China App Store in April 2026. Google and Apple both removed Bitchat from Play and
  the App Store in India on 3 October 2026, following a government order. A censorship-
  resistant app distributed only through a centralized store is not fully censorship-
  resistant: the store itself is the chokepoint. An optional store listing, built from the
  same source, may exist later, for reach, never as the only or privileged channel.
- **Reproducible builds.** The published binary must be byte-for-byte reproducible from the
  public source, so any third party can verify the release was built from the code it claims.
  Same-machine build reproducibility is verified today (byte-identical SHA-256 across a clean
  rebuild); cross-machine reproducibility and an F-Droid submission are not yet done, see
  [`docs/REPRODUCIBLE-BUILD.md`](docs/REPRODUCIBLE-BUILD.md).
- **iOS delivery** via the App Store / TestFlight remains an unavoidable centralization point,
  documented honestly, not hidden.

---

## 12. Governance, licensing, and sustainability

Full detail in [`docs/GOVERNANCE.md`](docs/GOVERNANCE.md).

- **Licence.** **AGPL-3.0-or-later** (`LICENSE`), a copyleft licence over a permissive one so
  that improvements to this infrastructure remain open, with the network-use clause closing
  the "run it as a hosted service without sharing changes" gap a plain GPL leaves. Documentation
  is CC BY-SA 4.0.
- **Stewardship.** The project is stewarded as a transparent, research-and-open-source
  initiative with a public mission centred on censorship-resistant communication and free
  expression. Every doc, site, and app footer credits **Developed by [Konko
  Maji](https://konkomaji.com/)**.
- **Open contribution.** Public specification, open issue tracker, and a contributor guide,
  including a framing discipline every contributor agrees to (`docs/GOVERNANCE.md` §4): name
  the real-world problem directly, government shutdowns and platform bans are current events,
  not a hypothetical, and describe the project plainly as censorship-resistant infrastructure.
- **Sustainability.** Grant funding and donations aligned with digital-rights and
  free-expression missions; community-run LoRa and Nostr relay infrastructure; no data
  monetization, since there is no user data to monetize by design.

---

## 13. Compliance

Full detail in [`docs/COMPLIANCE.md`](docs/COMPLIANCE.md). This is not legal advice; qualified
counsel should review before any launch.

- **No server, no user database, no personal identifiers.** There is nothing to hand over and no
  service backend for a traceability obligation to attach to. The absence of a central operator
  is a deliberate design property, not only a compliance one.
- **Spectrum.** The LoRa backbone (§6.4) operates in India's licence-free IN865 band under the
  duty-cycle and power limits in [`docs/HARDWARE-LORA.md`](docs/HARDWARE-LORA.md); type approval
  and spectrum compliance for any deployment, in any country, is the deployer's responsibility.
- **Licence and warranty.** AGPL-3.0-or-later, provided as is, with no warranty; see
  `docs/COMPLIANCE.md` and `LICENSE` for the full terms.

---

## 14. Evaluation and comparison with prior art

| Property | FireChat | Bridgefy | Briar | Session | Nostr | Farcaster | Meshtastic | **Project Mesh** |
|---|---|---|---|---|---|---|---|---|
| Open source | No | Partly | Yes | Yes | Yes | Yes | Yes | **Yes** |
| Security-first / audited design | No | No (broken) | Yes | Partial | N/A | N/A | Yes | **Yes (goal)** |
| Forward secrecy (ratchet) | No | No | Yes | Partial | No | No | Channel keys | **Yes** |
| Works fully offline, phone-to-phone | Yes | Yes | Partial | No | No | No | Yes | **Yes** |
| Internet relay bridge for reach | No | No | Tor (default) | Onion nodes | Relays (native) | Hubs | No | **Yes (Nostr)** |
| Social feed, no central ranking | No | No | Blogs | No | Yes | Yes (ranked) | No | **Yes** |
| Fact-check without a moderator | No | No | No | No | No | No | No | **Yes** |
| No blockchain/on-chain identity | Yes | Yes | Yes | Yes | Yes | **No** | Yes | **Yes** |
| Long-range (LoRa) backbone | No | No | No | No | No | No | **Yes** | **Yes (Phase 3)** |
| Fully de-Googled, store-independent distribution | No | No | Yes | Partial | N/A | N/A | Yes | **Yes (F-Droid first)** |

Phase 1 (phone-to-phone core) is implemented, verified on a real Android emulator, not only
unit-tested in isolation, 191 core tests, plus a DTN simulation harness
(`core/src/dtn_sim.rs`) that drives the real relay engine through scripted contact schedules
rather than a reimplemented protocol. **No independent security audit has been done**, and the
Nostr relay bridge, Community Notes, and the LoRa backbone (Phase 3) remain in progress or not
yet built, see [`docs/IMPLEMENTATION-STATUS.md`](docs/IMPLEMENTATION-STATUS.md) for the exact,
row-by-row state.

Evaluation still pending: delivery ratio and latency versus node density in real (not
simulated) field trials; battery cost of relay on representative low-end Android hardware; a
genuine two-device over-the-air exchange (blocked so far by this project's dev environment
having no spare hardware and no capacity to run two emulators at once); LoRa range and
throughput across representative terrain; and an independent security audit before any
general-availability release.

---

## 15. Limitations and open problems

- **iOS background relay is fundamentally limited** by Apple policy and cannot be fully solved
  (§6.2). I mitigate with foreground "relay mode" and iOS-to-iOS MultipeerConnectivity, but
  Android devices necessarily carry the background mesh.
- **Long-range coverage requires hardware.** Phone-only mesh cannot cover large distances
  without an internet relay bridge or the LoRa backbone (Phase 3), which implies device cost
  and community deployment effort.
- **DTN delivery is probabilistic**, not guaranteed, and degrades in sparse networks with few
  relays.
- **No content-moderation lever beyond client-side filtering.** This is a deliberate design
  property (G5), not an oversight, but it means a genuinely illegal post, once it has spread
  across the mesh or a relay, cannot be centrally removed. I accept this tradeoff plainly rather
  than quietly building a hidden moderation backdoor that would reintroduce the exact
  chokepoint this project exists to remove.
- **A short derived ID is not collision-free if misused.** The 6-character ID (§8.1) is a
  fingerprint of a real key pair, not an independently issued identifier. It must never be
  treated as a standalone source of uniqueness; the key pair underneath is the real identity.
- **Battery cost** of continuous scanning and relaying is real and must be tuned per device
  class.
- **Anonymity is bounded** (§3.2, §4): this design does not defend against a global passive
  adversary or endpoint compromise.
- **Metadata is not fully hidden**; onion routing reduces but does not eliminate what relays and
  observers can infer.
- **Trust bootstrapping** relies on out-of-band human verification, which has a real usability
  cost.

These are documented, not hidden, in keeping with goal G8.

---

## 16. Roadmap

The phased plan is maintained in [`docs/ROADMAP.md`](docs/ROADMAP.md). In brief:

- **Phase 0, Design and specification. Complete.** The full documentation set, and the core
  protocol decisions (crypto suite, envelope format) frozen for v1.
- **Phase 1, Core engine and chat, Android first. In progress.** Everything reused without
  change: the Rust core (identity, Noise, Double Ratchet, PQXDH, MLS groups, Argon2id
  channels, the store-carry-forward engine, encryption at rest, 191 tests), the Android BLE/
  Wi-Fi Direct drivers, and Direct/Group/Channel messaging, all verified on-device. What the
  pivot adds, not yet built: the Feed as geohash-scoped rooms, the Nostr relay bridge,
  Community Notes-style fact-checking with client-side filtering, the image thumbnail/relay
  split, and one-shot location sharing.
- **Phase 2, Reach and hardening.** iOS front-end, paused but not dropped; additional
  languages; performance tuning for low-end devices; the onion-routing fallback option;
  an independent security audit before any general-availability release.
- **Phase 3, Hybrid (LoRa backbone).** LoRa companion-node bridge in the IN865 band (and
  regional equivalents elsewhere); reference node design; an open contribution model for
  community node builders.
- **Phase 4, Ecosystem.** Community relay and node networks, governance maturity,
  cross-machine reproducible-build verification, and partnerships with digital-rights and
  journalism organizations rather than disaster-response partners, matching the pivot.

---

## 17. Conclusion

A platform that depends on a server has exactly one real vulnerability: the server. Project
Mesh removes it. This paper has described a decentralized, server-less, end-to-end encrypted
protocol for a social feed and messaging platform that works phone-to-phone today, bridges to
the open internet the moment any device nearby can reach it, and extends across a long-range
LoRa backbone later, with a fact-checking layer that needs no central moderator and a
distribution strategy that treats app stores as a real attack surface, grounded in what
actually happened to this project's own direct inspiration in 2026, not a hypothetical. The
design is deliberately candid about what is hard and what is impossible; the value of a
resilience tool lies entirely in whether its claims are true. The next step is independent
scrutiny of both this design and the code that implements it.

---

## 18. References

A consolidated, linked bibliography, covering DTN routing, Noise, the Double Ratchet, MLS,
Sphinx/Loopix/Nym, the Bridgefy analysis, and the comparable projects this pivot drew on
directly, Bitchat, Briar, Session, Nostr, and Farcaster, alongside Meshtastic, the Indian LoRa
band, and relevant Indian telecommunications law, is maintained in
[`docs/REFERENCES.md`](docs/REFERENCES.md).
