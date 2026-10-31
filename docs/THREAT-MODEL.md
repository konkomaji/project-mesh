# Threat Model

Companion to `WHITEPAPER.md` §4. I wrote this document to state who Betar defends against, what
it protects, what attacks I considered, how I mitigate them, and, critically, what I do **not**
defend against. Honesty about non-goals is a core value of this project.

---

## 1. Assets to protect

| Asset | Why it matters |
|---|---|
| **Message content** | The private substance of communication |
| **Social graph** | Who talks to whom (often more sensitive than content) |
| **Location / presence** | Where a user is and that they are active |
| **Identity** | Linking a key or device to a real person |
| **Device data at rest** | Everything above, if a device is seized |

**A note on location sharing specifically.** Betar ships one-shot location sharing in V1, sent
only through the existing end-to-end channel of one specific chat. I deliberately do not ship
continuous live-location tracking in V1. A continuous feed of a user's real-time position is the
single highest-risk feature I could add for a journalist or activist user: if the recipient's
device is compromised, or the recipient themselves is coerced, a live feed exposes where that
user is right now, not just where they were at one past moment. A one-shot share has the same
endpoint-compromise exposure as any other message, no more. I will not add continuous live
tracking until it has its own dedicated design for geofencing and automatic expiry.

## 2. Adversaries **in scope**

### A1: Passive local eavesdropper
Sniffs BLE / Wi-Fi / LoRa within radio range.
- **Mitigations:** end-to-end AEAD (content unreadable); Noise identity-hiding (long-term keys
  not exposed in handshake); rotating randomized link identifiers; size bucketing.
- **Hard limit: BLE physical-layer fingerprinting (verified, IEEE S&P 2022; see
  `RESEARCH-FINDINGS.md` §6):** hardware manufacturing imperfections (**Carrier Frequency
  Offset** and **I/Q offset**) let an attacker fingerprint a specific BLE radio at the physical
  layer, **bypassing MAC-address randomization entirely**. A **~$150 software-defined radio**
  suffices, and phones advertise constantly (an idle iPhone emits ~872 BLE advertisements per
  minute). Reliability is **limited** (thermal drift, chipset variation, configurable TX power,
  cheap-receiver noise), so it is **feasible-but-not-always-reliable**, not guaranteed. **I
  cannot fully defend against this.** MAC rotation raises the bar against casual tracking; it does
  **not** make a device untrackable by a determined local RF adversary. No "untraceable" claim is
  made anywhere in this project.

### A2: Malicious participant (modified node)
Runs altered software to **inject**, **drop**, **replay**, **flood**, or **corrupt**.
- **Mitigations:** AEAD integrity (corruption/injection rejected); content-derived
  `envelope_id` + seen-set (replays and duplicates ignored); TTL/hop limits and bounded store
  (flood bounded); per-peer rate limiting; optional client puzzle; key-based local blocking.

### A3: Local active attacker
Attempts **jamming** or **Sybil flooding** to deny service locally.
- **Mitigations (partial):** priority eviction keeps urgent traffic (direct messages ahead of
  bulk feed sync) moving under pressure; rate limits and puzzles raise Sybil cost; LoRa backbone
  provides an alternative path on a different band.
- Deliberate **radio jamming** of the 2.4 GHz band (BLE/Wi-Fi) is a physical attack no app can
  fully defeat; the sub-GHz LoRa backbone (Phase 3) is a mitigation because it uses a different
  band, not a cure.

### A4: Curious/malicious relay
An intermediate node trying to learn **who is talking to whom**.
- **Mitigations:** relays see only opaque sealed envelopes; optional **Sphinx onion routing** so
  no single relay learns both endpoints of a direct message; size bucketing.

### A5: Coercive physical adversary
Seizes and attempts to search a device.
- **Mitigations:** encryption at rest (`redb` + AEAD, Keystore-wrapped); optional passphrase;
  **panic-wipe** and optional **duress passphrase** (decoy state).
- Against a well-resourced forensic adversary with an unlocked device or a device-OS exploit, no
  application-level measure is a guarantee.

## 3. Adversaries **out of scope** (non-goals)

No user should be misled about these:

- **G-PA: Global passive adversary.** An entity able to observe *all* radio emissions across a
  wide area and perform end-to-end **timing/traffic correlation**. Onion routing and bucketing
  raise the cost but do **not** defeat this class. Not defended.
- **EP: Endpoint compromise.** Malware or an OS/firmware exploit on the user's own device. If
  the endpoint is owned, its plaintext is exposed. Not defended (beyond standard at-rest and
  key-hygiene measures).
- **SC: Supply-chain / build compromise.** Mitigated *procedurally* by reproducible builds and
  open source (see `DISTRIBUTION.md`), not by the protocol.

## 4. Attack → mitigation summary

| Attack | Class | Primary mitigation | Residual risk |
|---|---|---|---|
| Read message content | A1, A4 | E2E AEAD, Double Ratchet | Endpoint compromise (out of scope) |
| Learn long-term identity from handshake | A1 | Noise XX identity hiding | None |
| Track device by link address | A1 | Rotating randomized identifiers | Fingerprinting by other radio features (partial) |
| Inject / forge messages | A2 | AEAD integrity, signatures | None |
| Replay old messages | A2 | Content-ID + seen-set | None |
| Flood / exhaust storage | A2, A3 | TTL, hop cap, bounded store, rate limit, puzzle | Sophisticated distributed flood (partial) |
| Correlate sender↔recipient | A4 | Sphinx onion routing, bucketing | Global passive correlation (out of scope) |
| Jam the radio | A3 | Priority eviction; LoRa alt-band | Physical jamming (fundamental) |
| Seize device | A5 | At-rest encryption, panic-wipe, duress | Forensic w/ unlocked device (out of scope) |
| Recover past msgs after key leak | n/a | Forward secrecy (ratchet) | None |
| Read future msgs after key leak | n/a | Post-compromise security (ratchet) | Window before self-heal |
| Illegal content spreads (CSAM, doxxing, terrorism material) | A2, A4, structural | Client-side filtering via the Community Notes web-of-trust layer; relay operators may independently refuse to carry flagged content | No server or relay can delete a post everywhere once it has spread; a government may respond by banning the whole app or repo instead of one post (see below) |
| App or repo itself targeted for a ban | n/a, structural | F-Droid + direct APK as the primary distribution channel, not app-store-only | A government can still block GitHub, F-Droid mirrors, or ISPs directly; not fully defended |

## 5. Trust model

- **Root of trust is human.** Identity verification is in-person (QR / safety string). There is
  no certificate authority, no server, and no directory to compromise or coerce.
- **No central operator** means there is no single party holding anyone's data. There is nothing
  to request, seize, or leak from a server, because no such server or data store exists. This is
  a structural property of the design, not a policy promise (see `COMPLIANCE.md`).

## 6. Abuse, illegal content, and moderation

A server-less network cannot centrally moderate. No relay or server in this design can delete a
post everywhere once the mesh and the Nostr relay bridge have spread it. I decided on this
policy instead, after looking at what happened to a comparable project with no moderation at
all:

- **Client-side filtering via Community Notes.** Any user can attach a note to a post. Notes and
  their ratings are signed events, synced the same way posts are. The scoring algorithm is public
  and deterministic, so any client computes the same score from the same data, with no central
  verdict. A post flagged as illegal or harmful can be locally hidden or warned-on by each user's
  own client. This does not delete the post from the network. It removes it from view for users
  whose client applies the score, without creating a new party who can be pressured to delete
  content everyone sees.
- **Relay-level opt-out.** A relay operator can refuse to carry specific content on their own
  relay, the same way real Nostr relay operators already do today. This does not guarantee a
  post disappears everywhere, since another relay may still carry it, but it means no single
  relay operator is forced to carry everything either.
- **Rate limits and puzzles** raise the cost of automated abuse and flooding, same as before.
- **Local blocking and key reputation:** users can block keys; clients can locally de-prioritize
  or drop traffic from blocked keys.

**The honest limit, stated plainly: none of this is content removal.** A government cannot force
Betar to take down one post, because there is no server to serve that order to. Bitchat, the
project that inspired Betar's mesh-plus-relay design, ran into exactly this in July 2026: India's
Home Ministry did not ask for one post removed, it ordered GitHub to restrict three Bitchat
repositories entirely, arguing the architecture itself made lawful interception impossible.
Betar carries the same structural property and the same structural risk. I am not hiding that
trade-off. A network nobody can censor one message on is also a network nobody can be forced to
remove the worst message from, and the response a government has available is to target the
whole app instead of one post. This is acknowledged as an open problem, not a solved one, and is
also listed among the limitations in `WHITEPAPER.md` §15.

## 7. Distribution as an attack surface

An app store is a single party a government can pressure to remove an app, the same structural
weakness a server has. Bitchat was removed from Google Play and the Apple App Store in India on
3 October 2026, and from the Apple App Store in China in April 2026, before this document was
written. Betar's primary distribution channel is F-Droid and a direct APK download, not an app
store listing. A store listing may exist later as a convenience, never as the only path to
install Betar.

## 8. What this threat model does not claim

No independent security audit of Betar has happened yet. Nothing in this document, or anywhere
else in this project, should be read as a claim that one has. An audit is a hard gate before any
public claim of security, not a step I have completed.

---

Developed by [Konko Maji](https://konkomaji.com/).
