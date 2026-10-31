# Routing Protocol and Wire Format

Store-carry-forward routing, envelope format, and flood controls. Companion to `WHITEPAPER.md`
§7. Values marked *(tunable)* are initial proposals to be validated by simulation and field
testing, not final constants.

---

## 1. Model: gossip on contact

Project Mesh is a **delay-tolerant network (DTN)**. There is no stable end-to-end path, no
addressing scheme, and no routing table. Instead:

1. Each node holds a bounded **store** of unexpired **envelopes** it has seen.
2. When two nodes come into contact, they exchange a compact **summary vector** describing which
   envelopes they hold.
3. Each node requests and receives only the envelopes it is missing.
4. Nodes carry envelopes as they move, spreading them on each new contact ("epidemic"
   dissemination), until an envelope reaches its recipient or expires.

This is the correct model for a network whose topology is unknown, sparse, and constantly
changing.

## 2. The envelope

An **envelope** is the unit of storage and transfer. Application data is always **sealed**
(encrypted) before it becomes an envelope payload; relays see only opaque bytes plus the minimal
routing header they need.

```
Envelope (matches core/src/envelope.rs as implemented)
┌───────────────────────────────────────────────────────────────┐
│ version           u8                                           │
│ envelope_id       32 bytes   (content-derived, BLAKE3)         │  ← dedup key
│ addressing_tag    u8         (0 Broadcast | 1 Channel |         │
│                               2 Group | 3 Direct)                │
│ addressing_target 32 bytes?  (present for Channel/Group/Direct, │
│                               absent for Broadcast)              │
│ priority_tag      u8         (0 Urgent | 1 Elevated |            │
│                               2 Normal | 3 Low)                  │
│ hops_remaining    u8         (decremented per relay)           │  ← TTL (hops)
│ expires_at        u64        (absolute expiry)                 │  ← TTL (time)
│ sealed_payload    bytes      (AEAD ciphertext; opaque to relays)│
└───────────────────────────────────────────────────────────────┘
```

Notes:

- **`envelope_id`** is derived from the sealed content, so it is stable, self-certifying against
  accidental corruption, and usable as the deduplication key.
- **`addressing_tag`** is the routing class: who the envelope is for. It never reveals
  plaintext. `priority_tag` is a separate axis, the relay-eviction urgency: lower ordinal is
  higher priority and evicted last under storage pressure (`envelope.rs`'s `Priority` enum).
  `Elevated` is the tier Community Notes use, so a fact-check correction resists eviction and
  propagates ahead of ordinary traffic. `Urgent` is unclaimed by any current feature, the top
  tier reserved for whatever needs it next.
- **What a post or a Community Note *is*, versus a chat message, lives inside the sealed
  payload, not in the envelope header.** The routing layer only ever sees `addressing_tag` and
  `priority_tag`; it never inspects or needs to know the sealed content's shape, by design, the
  same way it can't distinguish one Direct message's content from another's. Feed posts and
  Community Notes are not implemented in `core/` yet; when they are, this section needs a real
  update once their actual addressing choice (likely Broadcast for the local-mesh tier, a
  Channel-like geohash selector for the relay tier) is designed and built, not guessed at here
  ahead of that work.
- A channel selector or a Sphinx onion header (for future onion-routed Direct messages) would
  need its own wire field if implemented; neither exists in `core/` yet, so neither is in the
  diagram above. Don't add speculative fields for unbuilt features.

## 3. Contact exchange protocol

On each contact between nodes A and B:

```
A → B :  HELLO {proto_version, capabilities, nonce}
B → A :  HELLO {proto_version, capabilities, nonce}
A → B :  SUMMARY {bloom_filter_of_A_envelope_ids, high_water}
B → A :  SUMMARY {bloom_filter_of_B_envelope_ids, high_water}
A → B :  WANT {ids A lacks that B appears to have}   (from B's summary)
B → A :  WANT {ids B lacks that A appears to have}
A ⇄ B :  DATA {requested envelopes, ordered by priority_tag: Urgent → Elevated → Normal → Low}
```

- **Summary vectors** are **Bloom filters** (compact, false-positive-only) plus a small explicit
  recent-ID list to bound false positives on hot items.
- Transfers are **priority-ordered** by `priority_tag` so that in a short or lossy contact
  window, the highest-priority envelopes move first, whatever they contain.
- Both sides enforce all controls in §4 on every accepted envelope.

## 4. Flood and abuse controls

Naïve epidemic routing is expensive and trivially floodable. Controls:

### 4.1 Time and hop TTL
- Every envelope has `expires_at` (absolute) and `hops_remaining`. A node **drops** any envelope
  that is expired or has zero hops remaining, and never forwards it. *(tunable: default expiry
  e.g. 24–72 h; default max hops e.g. 16–32.)*

### 4.2 Deduplication
- A node maintains a **seen-set** (Bloom filter + bounded LRU of exact IDs). It never re-accepts,
  re-stores, or re-forwards an `envelope_id` it has already handled. This bounds re-transmission
  and breaks loops.

### 4.3 Bounded store with priority eviction
- The store has a fixed byte budget *(tunable)*. When full, eviction favours keeping
  high-priority and fresh envelopes and dropping low-priority, near-expiry, or widely-seen ones
  first.

### 4.4 Rate limiting
- Per-peer caps on envelopes/second and bytes/second during a contact, to stop a single peer
  saturating a node.

### 4.5 Sybil / injection resistance (lightweight)
- Optional **client puzzle** (small proof-of-work) bound to `envelope_id`, raising the cost of
  mass-producing distinct envelopes. `envelope_id` is itself content-derived over the whole
  envelope, so binding to it alone already commits the puzzle to the full content; no separate
  timestamp field exists to bind against (`core/src/puzzle.rs` explains this in detail).
  Calibrated to be negligible for a human sending a message and expensive for a flood.
  *(tunable / optional.)*
- Per-identity signing on non-anonymous addressing modes (Direct, Group, and signed Channel
  traffic, where the sender already reveals a fingerprint to make delivery possible) lets nodes
  locally rate-limit or block abusive keys without any central coordinator.

### 4.6 Optional directional spraying
- Where a coarse gradient exists (e.g. "toward a known LoRa gateway" or "toward a fixed relay
  node"), a node can bias forwarding (Spray-and-Wait / PRoPHET style) to cut redundant copies,
  without maintaining full routing state. Pure epidemic remains the fallback.

## 5. Addressing modes and sealing

| Mode | Who can read | Sealing | Routing tag |
|---|---|---|---|
| **Broadcast** | anyone nearby | signed, not encrypted (public) | empty |
| **Channel** | anyone with passphrase | AEAD under Argon2id-derived channel key | channel selector |
| **Group** | listed members | one AEAD copy sealed per member | per-copy |
| **Direct** | one recipient | Double Ratchet AEAD; optional Sphinx onion | onion header (opt) |

Cryptographic detail for each is in `CRYPTOGRAPHY.md`.

### 5.1 Prekey bundle transport

**Decided:** a device's current `PrekeyBundle` (`CRYPTOGRAPHY.md` §4.2: signed prekey + at most
one one-time prekey, the whole point being to let someone bootstrap a session with a peer who is
never simultaneously in range) travels as an ordinary **Broadcast** envelope, magic-byte-prefixed
exactly like the other envelope classes already are, rather than as a new `Addressing` variant.
Reasoning:

- In-person exchange (alongside the identity fingerprint, for trust verification) only works
  when both parties *are* simultaneously in range, which defeats the actual point of X3DH, so it
  can establish trust in a fingerprint but can't be the sole way a *current* rotating bundle
  reaches someone later.
- Broadcast already has exactly the properties this needs: no setup, reaches everyone the
  epidemic relay touches, and a bundle is meant to be discoverable by anyone who might want to
  initiate contact, not access-controlled like a Channel or Direct message would imply.
- The bundle is self-authenticating (the signed prekey's signature verifies against the identity
  key embedded in the bundle itself, see `PrekeyBundle::verify_signed_prekey`), so Broadcast's
  "signed, not encrypted" property is exactly right: no confidentiality is needed or wanted for a
  public bootstrap artifact, only tamper-evidence, which the signature already provides
  independent of the transport layer.
- A **shorter TTL** than ordinary broadcast traffic is appropriate: a stale prekey bundle
  either fails signature verification against a *rotated* signed prekey (harmless, `initiate`
  just returns an error) or, worse, references a one-time prekey the publisher already consumed
  responding to someone else (also harmless, `respond` simply won't have that secret, and X3DH
  degrades to three DH terms rather than failing outright per `CRYPTOGRAPHY.md` §4.2's design).
  Neither failure mode is dangerous, but propagating a long-stale bundle wastes relay bandwidth on
  something increasingly unlikely to still be useful: a native-layer/UX-tunable cadence (re-
  broadcast periodically, similar order of magnitude to the BLE link-identifier rotation interval
  in `CRYPTOGRAPHY.md` §7.1), not a hard protocol requirement.

**Not yet implemented:** `core/src/crypto/prekey.rs`'s `PrekeyBundle` has a wire format
(`to_bytes`/`from_bytes`) as of this pass, but no magic-byte class, UniFFI export, or Kotlin
messenger exists yet to actually compose/broadcast/receive one in the app: those are the next
increments, tracked in `docs/IMPLEMENTATION-STATUS.md`, not bundled silently into "the decision is
made" here.

## 6. LoRa considerations

The LoRa backbone has a tiny bit-rate, so over LoRa the engine:

- forwards **only small, `Urgent`/`Elevated`-priority envelopes** by default, whatever
  addressing mode or content they carry;
- keeps bulk payloads (images) on BLE/Wi-Fi;
- applies stricter TTL/hop budgets and respects the IN865 **duty-cycle** limits (see
  `HARDWARE-LORA.md`).

## 7. Delivery semantics

- Delivery is **best-effort and probabilistic** (inherent to DTN). There is no guaranteed
  delivery and no global acknowledgement.
- **End-to-end acknowledgements** are themselves envelopes: a recipient can send a small sealed
  ACK back through the mesh, giving the sender eventual (not immediate) delivery confirmation.
- Delivery ratio and latency depend heavily on node density and mobility; these are the primary
  quantities the simulation harness (`ARCHITECTURE.md` §7) measures.

---

Developed by [Konko Maji](https://konkomaji.com/)
