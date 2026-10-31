# Application Features

I pivoted Betar in October 2026, from a disaster-safety app to a censorship-resistant social
and chat platform. This document lists what I am building for version 1, and what I decided
not to build.

Every feature runs on the same mesh substrate (`ROUTING-PROTOCOL.md`) and the same
cryptography (`CRYPTOGRAPHY.md`) as before. The pivot changed the product, not the protocol.

---

## 1. Chat

- **Direct messages.** 1:1, end-to-end encrypted with Noise XX into a Double Ratchet, with
  X3DH plus PQXDH (ML-KEM-1024) for first contact.
- **Groups.** MLS (RFC 9420), bounded membership.
- **Channels.** Argon2id passphrase-derived key, open local topic rooms.
- **Location sharing.** One-shot only. A user sends their current location once, inside a
  specific chat thread, through that thread's encrypted channel. I decided against
  continuous live-location tracking for version 1. See "Not building" below.

## 2. Feed

- **Geohash-scoped public rooms.** I copied this pattern directly from Bitchat. A room's
  precision runs from a city block up to a country, and the user picks it.
- **Local-mesh first.** A post reaches phones in Bluetooth/Wi-Fi Direct range immediately,
  with no internet required.
- **Relay-bridged reach.** When any device in a local mesh cluster has internet access, it
  publishes that cluster's posts to the public Nostr relay network, and pulls down posts for
  the cluster in return. One device's internet connection, even briefly, extends reach past
  the local radius for everyone nearby.
- **No ranking algorithm.** Feed order is reverse-chronological. There is no server to run an
  engagement-ranking algorithm on, and I decided against building one even where it would be
  technically possible.
- **Images.** A post's image attaches as a small, lossy, EXIF-stripped thumbnail over the
  mesh. The full-resolution file uploads only once a relay bridge exists, to a
  content-addressed blob store (the Nostr ecosystem's Blossom protocol, or an equivalent),
  so no single host is required. A mesh-only post stays at thumbnail quality until a relay
  bridge reaches it.

## 3. Fact-check and illegal-content handling

- **Community Notes model.** Any user can attach a note to a post. Notes are signed events,
  gossiped the same way posts are.
- **Decentralized scoring.** The scoring algorithm is public and deterministic. Any client or
  relay computes the same score locally from the same synced events. No single scorer is
  authoritative.
- **Client-side filtering for illegal content.** Betar has no server or relay that can delete
  a post. Instead, a reported or flagged post can be hidden or warned-on by each user's own
  client, driven by the same web-of-trust scoring used for fact-checking. I chose this over 2
  alternatives: full moderation (impossible without a server to run it on) and no moderation
  at all (Bitchat's current approach, which led to India's government ordering GitHub to
  restrict 3 Bitchat repositories in July 2026, rather than removing individual posts).

## 4. Security

- **End-to-end encryption everywhere**, including Feed posts, not only direct messages.
  Betar has no unencrypted mode.
- **Panic wipe.** A hold-to-confirm gesture erases all local data immediately.
- **Pubkey identity only.** Betar requires no phone number and no email address.

## 5. Transport

- **Local mesh.** Bluetooth Low Energy and Wi-Fi Direct, with store-carry-forward gossip. A
  message or post does not need sender and recipient to ever be in range at the same time. It
  needs a chain of pairwise range overlaps over time, the same way a phone carried between 2
  locations bridges them.
- **Relay bridge.** The Nostr protocol, not a custom gossip layer. Bitchat, my stated
  inspiration, already runs on real Nostr relays. Reusing the existing network means Betar's
  online tier works from day 1, with no new infrastructure required.

## 6. Distribution

- **F-Droid and direct APK first.** App stores are themselves a ban chokepoint a government
  can pressure. Betar's store listing comes later, as a convenience channel, never as the
  only path to install it.

## 7. Not building

I decided against each of the following, with the reason stated:

- **Blockchain or on-chain identity.** Farcaster tied its identity to the Optimism
  blockchain, and the protocol itself sold twice in under a year. A blockchain dependency is
  one more thing a government or a buyer can capture. It contradicts the point of Betar.
- **Interactive mini-apps or post "frames."** Farcaster's frames turned its feed into a
  platform for NFT and token promotion, not better discourse. I am not building a programmable
  post format.
- **An unencrypted or public-only mode.** FireChat shipped without encryption and encouraged
  real names. That design was its most-cited criticism. Betar encrypts everything, with no
  exception.
- **Continuous live-location tracking.** This is the single highest-risk feature for a
  journalist or an activist. A compromised device or a compromised recipient exposes
  real-time whereabouts. Version 1 ships one-shot location sharing only. A continuous version
  needs its own safety design, including geofencing and auto-expiry, before I build it.

## 8. What stayed the same

- **Envelope format and routing engine**, shared by every feature above, same as before the
  pivot.
- **Offline-first behaviour.** No feature blocks on having a network connection.

Developed by [Konko Maji](https://konkomaji.com/)
