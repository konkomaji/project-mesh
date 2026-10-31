# Distribution and the De-Googled Requirement

Companion to `WHITEPAPER.md` §11. De-Googling is both a values choice and a robustness
requirement: the app must run where Google's services are absent, unreliable, or unwanted.

---

## 1. No proprietary cloud layer

Betar uses **no Google Play Services, no Firebase, and no third-party proprietary
cloud**. Concretely:

| Common Google dependency | What we use instead |
|---|---|
| Firebase Cloud Messaging (push) | No server push at all: the mesh is local; local notifications from the foreground service |
| Google Maps SDK | **MapLibre GL Native + OpenStreetMap** offline tiles |
| Play Services location | Android platform location APIs directly (opt-in, coarse) |
| Play Integrity / SafetyNet | None (not needed; would add a Google dependency) |
| Crashlytics / analytics | None, or a self-hostable/opt-in libre crash reporter with no PII |
| Play-only distribution | F-Droid + IzzyOnDroid + direct APK (Play optional) |

Because there is **no server and no push**, the largest reason apps depend on Google
(notifications) simply does not apply.

## 2. Android distribution

- **F-Droid and direct signed APK are the primary channel, not a backup plan.** A government or
  platform operator can remove an app from Google Play or the App Store in hours. It cannot
  remove a reproducible-build APK already on thousands of devices, or force every F-Droid mirror
  offline at once.
- **IzzyOnDroid** repository for faster updates.
- **Direct signed APK** download from the project site, with published signing-key fingerprints.
- **Optional Google Play** listing built from the *same* source, for reach, never the only or
  privileged channel.
- Runs on **de-Googled ROMs** (GrapheneOS, LineageOS, /e/OS, CalyxOS) and on stock devices
  without Google connectivity.

## 3. Reproducible builds (mandatory for a security tool)

- The published binary must be **byte-for-byte reproducible** from the public source, so any
  third party can verify the release was built from the code it claims, closing the
  supply-chain gap that source-availability alone leaves open.
- F-Droid's reproducible-build verification is used where possible.
- Build inputs (toolchain versions, dependencies) are pinned; the build is documented and
  scripted.

## 4. iOS distribution

- iOS apps are distributed through the **App Store** (and **TestFlight** for testing). This is an
  **unavoidable centralization point**: Apple controls the channel.
- iOS distribution is not decentralized, and this doc does not pretend otherwise. It affects
  availability (Apple could remove the app), not the on-device security or the offline operation
  of the mesh.
- This is not a theoretical risk. Apple removed Bitchat, the mesh messenger that inspired this
  project, from its China App Store in April 2026. Google and Apple also removed Bitchat from
  Play and the App Store in India on 3 October 2026, following a government order. A
  censorship-resistant app distributed only through a centralized store is not fully
  censorship-resistant.

## 5. Update and integrity

- Releases are **signed**; users can verify signatures and reproducible-build attestations.
- No silent auto-update through a proprietary channel is required; F-Droid and direct APK give
  transparent, verifiable updates.

## 6. Why this matters for the mission

- **Runs anywhere:** a de-Googled app works on cheap devices, custom ROMs, and in environments
  with no Google reachability, including a device seized or wiped mid-protest or mid-shutdown.
- **No hidden dependency to fail or leak:** every network capability is either local (the mesh)
  or explicitly optional; there is no background beacon to a proprietary cloud.
- **Verifiable trust:** open source + reproducible builds let the security community, not a
  vendor, vouch for the binary.
- **No single distribution chokepoint:** the Bitchat precedent above is the reason this matters
  in practice, not just in principle.

---

Developed by [Konko Maji](https://konkomaji.com/).
