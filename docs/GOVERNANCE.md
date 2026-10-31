# Governance, Licensing, and Sustainability

Companion to `WHITEPAPER.md` §12.

---

## 1. Stewardship

Betar and Project Mesh are stewarded as a transparent, research-and-open-source initiative with a
**public mission centred on censorship-resistant communication and free expression**: no server
to subpoena, no account to ban, no central operator to pressure. The recommended legal form is a
**non-profit** structure (e.g. a Section-8 company or a registered society/trust in India) with
clear, accountable governance. A neutral, mission-clear non-profit is also the correct legal
posture (see `COMPLIANCE.md`).

## 2. Licensing

### 2.1 Source code, copyleft preferred
- Recommended: **GPLv3** or **AGPLv3**. Copyleft keeps improvements to public-good infrastructure
  open, preventing closed proprietary forks that take without giving back.
- Contrast: a permissive licence like MIT would permit closed-source derivatives. For civic
  commons we lean copyleft; the final choice is to be confirmed with counsel and community input,
  weighing whether commercial closed forks should be permitted (MIT) or not (GPL/AGPL).
- **AGPL** specifically closes the "network-service" loophole, relevant if any optional
  server-side helper is ever written (there is none in the core design).

### 2.2 Documentation
- **CC BY-SA 4.0** for all documentation (this repository).

### 2.3 Hardware / firmware
- Reference node designs and firmware profiles under an appropriate open hardware/firmware
  licence, interoperating with and contributing upstream to **Meshtastic** where practical.

## 3. Contribution model

- **Open specification and issue tracker.** Design happens in the open.
- **Contributor guide** covering code standards, the Rust-core boundary, security-review
  expectations, and the **framing discipline** below.
- **Security disclosure policy:** a responsible-disclosure process and, before any
  general-availability release, an **independent external security audit** (a hard gate).
- **Localization workflow:** community translation with a safety-term glossary
  (`LOCALIZATION-UX.md`).

## 4. Framing discipline (required of all contributors)

Positioning is load-bearing (`docs/DESIGN-BRIEF.md`, `docs/BETAR-TRANSITION.md` Part 2). Every
contributor agrees to consistent language in code, docs, commits, issues, and public
communication:

- **Do** describe Betar plainly as a censorship-resistant social and messaging platform: no
  server to pressure, no account a government can ban, no post a platform operator can be forced
  to remove everywhere at once.
- **Do** name the real-world problem directly: government internet shutdowns, platform bans, and
  speech restrictions are current events, not a hypothetical. Say so openly in docs, marketing,
  and design rationale.
- Betar credits its developer. Every doc, site, and app footer carries "Developed by Konko Maji"
  linked to `https://konkomaji.com/`. This replaces an older no-byline rule from the project's
  disaster-safety phase; that rule no longer applies.
- This is not about manufacturing controversy; it is about leading with the genuine primary
  purpose in plain, calm language.

## 5. Sustainability

- **Funding:** grants aligned with digital-rights, free-expression, and internet-freedom
  missions; individual and institutional donations. No venture model that would pressure the
  project away from its ethos or create a buyable, bannable chokepoint.
- **No data monetization:** there is no user data to monetize. By design there is no server,
  no account, and no directory.
- **Community infrastructure:** LoRa relay nodes and Nostr relays are community-owned and
  community-funded, distributing both cost and control.
- **Institutional partners:** digital-rights organisations, journalist-safety and press-freedom
  groups, and civil-liberties bodies as review and deployment partners.

## 6. Decision-making

- Technical decisions via open proposals (design docs / RFCs) and maintainer review.
- Security-relevant changes require review against the threat model (`THREAT-MODEL.md`).
- Roadmap (`ROADMAP.md`) is public and revised in the open.

---

Developed by [Konko Maji](https://konkomaji.com/).
