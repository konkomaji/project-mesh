# Design Brief: Betar

I wrote this brief for Betar's visual and interaction design. Hand it to a designer, or paste
it into a design tool, whole. I made it self contained on purpose. Nobody needs to read the
rest of the repo to start work.

I pivoted Betar in October 2026. It is no longer a disaster-safety app. It is now a
censorship-resistant social and chat platform. Nobody can ban an account or remove a post,
because there is no server to pressure or subpoena. My inspiration is Bitchat, Jack Dorsey's
Bluetooth-mesh messenger. I built Betar for Gen Z, for journalists, for activists, and for
anyone who wants real freedom of speech.

I built 4 visual-direction concepts, each applying a different design system to the same
screen set below. **I picked Civic Calm (Concept D)** as the locked direction, decided
2026-10-31. All 4 live in the [`betar` repo's `docs/design-concepts/`
folder](https://github.com/konkomaji/betar/tree/main/docs/design-concepts), not here. This is
the protocol repo; app-design artifacts belong with the app.

---

## 1. What Betar is

Betar moves messages and posts phone to phone over Bluetooth and Wi-Fi Direct. A post or a
message hops from phone to phone until it reaches people nearby, or until a phone with internet
access bridges it to the wider network. There is no server anywhere, no account, no login, and
no phone number.

The whole design serves one sentence:

> **Betar. Say it. Nobody can take it down.**

I built Betar for:

- **Journalists and activists** who need to publish and talk without a platform that can ban
  them or a government that can pressure a server.
- **Gen Z users** who want a social feed and real chat, without a company controlling either.
- **Anyone under an internet shutdown or a platform ban**, who still has Bluetooth and a phone
  nearby.

The tone is confident and plain. This is a tool for people who want to speak freely and stay
safe while they do it. It is not a disaster app, and it carries no safety-infrastructure
framing.

## 2. The name

*Betar* (বেতার) is Bengali for wireless. Literally *be-* (without) and *tar* (wire).

It honours Jagadish Chandra Bose. In November 1895, in Calcutta, he sent radio waves through
two walls and a seated man's body to ring a bell and set off gunpowder in the next room. That
was a public demonstration of wireless transmission before Marconi's. He worked at millimetre
wavelengths a century before ordinary phones got there. He refused to patent any of it, because
he thought the work belonged to everyone.

Betar is open source under AGPL-3.0. I built it on the same idea, 130 years later.

Put the tribute on the About screen. Write it warmly and short. Do not overclaim. He
demonstrated wireless publicly before Marconi and gave it away unpatented. Do not write that he
invented radio. That claim is genuinely contested, and I do not make claims I cannot back.

## 3. How to frame this app

### Say what it is, plainly

Betar is a censorship-resistant social and chat platform. Say that directly, in the store
listing, the onboarding screens, and the About page. Do not euphemize censorship, free speech,
or government bans. This product's whole point is standing against them.

### Chat and Feed are both first-class

Betar carries two kinds of communication, and neither one is secondary:

- **Chats** is for direct messages, groups, and passphrase channels.
- **Feed** is for public posts in geohash-scoped rooms (see section 9).

Put both in the bottom tab bar, each with equal visual weight. Do not bury either one inside
a menu.

### Words to use

free speech, censorship-resistant, no server to ban, nobody can take it down, works without
the internet, local mesh, bridged, encrypted, carried.

### Words to avoid

Avoid vague euphemisms for what this app actually does. Do not write "connectivity solution"
when you mean "censorship resistance." Say the real thing.

### Credit

Add a short footer line on the About screen and in this repository's `README.md`:
**Developed by [Konko Maji](https://konkomaji.com/)**. I reversed my own earlier
no-developer-byline rule. Credit me by name now, every time, with that link.

## 4. Who uses it

- Gen Z users who already know Instagram and X, and expect a feed, a composer, and a chat
  list to look and feel familiar.
- Journalists and activists who need real E2E encryption and a panic wipe, and who read the
  Nearby screen's mesh status as a safety signal, not a novelty.
- Regular people in an internet shutdown, who have never used a mesh app before and need the
  app to work the first time with no account and no tutorial they must read.
- A wide range of reading ability. I do not require literacy to send a post, send a message,
  or trigger the panic wipe.

## 5. Hard constraints

1. **No onboarding friction.** No signup, no account, no email, no phone number, no login.
   Betar generates a keypair in the background. Install to first post takes under 30 seconds.
2. **Offline is the normal state.** No screen waits on a network. Betar shows no "no internet"
   error, because offline is the expected condition, not a failure.
3. **Delivery status must be honest.** A post or message travels when phones happen to meet.
   That can take minutes or hours. Never show a checkmark that implies guaranteed delivery.
   Build a 4-state indicator:

   | State | Means |
   |---|---|
   | Waiting | Still on your phone, nobody nearby yet |
   | Local | Reached phones in Bluetooth/Wi-Fi Direct range |
   | Bridged | Reached a Nostr relay through someone's internet connection |
   | Confirmed | The recipient's device acknowledged it |

4. **One reserved danger colour.** Reserve a single colour for the panic wipe control and
   destructive confirmations only. Never use it for errors, decoration, or anything else. This
   is a safety rule, not a style preference.
5. **Never rely on colour alone.** Every state uses at least two of colour, shape, icon, and
   text. Colour blindness must never hide a state.
6. **Accessibility.** Touch targets 48dp or larger. Body text 16sp minimum. Layouts survive
   200 percent font scaling.
7. **Cheap phones.** Keep motion restrained. Avoid heavy blur and large uncompressed images.
   Keep the install size small. Honour the system reduce-motion setting.
8. **Image posts are thumbnail-only over the mesh.** A post with an image carries a small,
   lossy, EXIF-stripped thumbnail over Bluetooth/Wi-Fi Direct. The full-resolution image
   uploads only once a relay bridge exists. A mesh-only user who never reaches a relay sees
   the thumbnail only. Show this distinction visibly on the image, do not hide it.
9. **The Feed has no ranking algorithm.** Feed order is reverse-chronological, always. There
   is no server to run a ranking engine on, and I do not want one. Do not design a "For You"
   sort, ever.
10. **Location sharing in a chat is one-shot only.** A user can share their current location
    once, inside a specific chat thread, sent only through that thread's encrypted channel.
    Betar has no continuous live-location-tracking feature. Do not design a "share live
    location" toggle, that feature does not exist yet and needs its own safety design first.

## 6. Design system

**Locked: Civic Calm**, decided 2026-10-31. Warm, soft, mainstream-friendly, muted neutral
palette (sage, warm cream, soft charcoal, one gentle accent), generous whitespace, calm rounded
shapes, gentle elevation. Built to feel like a trustworthy civic service, not a hacker tool or a
protest app, so a wider range of people, not just activists, feel safe using it. Full spec and
a working mobile mockup are in the `betar` repo at
[`docs/design-concepts/betar-design-D-civic-calm.html`](https://github.com/konkomaji/betar/blob/main/docs/design-concepts/betar-design-D-civic-calm.html).
The other 3 concepts considered and not picked:

- **Ground Signal**: dark, terminal and mesh-radio aesthetic, one signal-green accent.
- **Open Wire**: bright, familiar to Instagram/X users, keeps Betar's existing brand blue
  `#4BA3E0`.
- **Samizdat**: editorial, zine, underground-press aesthetic, high contrast, one loud accent.

The locked system needs:

1. A full light and dark theme.
2. A type scale with a 16sp body base.
3. A reserved danger colour, used only for the panic wipe control (see section 5, rule 4).
4. Noto Sans, plus Noto Sans Bengali, bundled with the app so Bengali text renders correctly.

## 7. The logo

The mark is designed and locked. Files are in `docs/assets/`.

| File | Use |
|---|---|
| `betar-logo.svg` | Primary mark |
| `betar-logo-compact.svg` | 24dp and below |
| `betar-logo-mono.svg` | Single colour, takes `currentColor` |

It is a solid field with a wire cut clean out of it, broken in the middle. The wire is present
only as its own absence, which is what the name means.

## 8. Information architecture

5 destinations in the bottom tab bar.

1. **Feed**, geohash-scoped public rooms (see section 9)
2. **Nearby**, who and what is in mesh range right now, and whether a relay bridge is active
3. **Chats**, direct messages, groups, and passphrase channels
4. **Notes**, Community Notes attached to posts, open to rate and reply
5. **Profile**, identity, settings, and the panic wipe control

## 9. Screens

### Onboarding

1. One illustrated panel, plain language, no reading requirement to proceed: "Betar works
   with no account. Your phone talks to nearby phones directly, and bridges to the wider
   network when it can."
2. Nickname, optional and skippable. Betar auto-generates one.
3. Permissions explained in plain language, then the system dialogs.

### Feed

4. Current geohash room, with a visible room indicator (for example `#block`, `#city`,
   `#region`) and a control to change precision or join another room.
5. Post composer: text, an optional image attachment (thumbnail preview, full-resolution
   notice per rule 8 in section 5), and an optional geohash tag.
6. Post detail screen, with the attached Community Notes fact-check UI: existing notes, their
   helpful/not-helpful rating, and a control to add a new note.

### Nearby

7. Mesh status screen: local peers in range, hop count, and whether a relay bridge is
   currently active for this cluster.

### Chats

8. Conversation list, with delivery state and conversation type on each row.
9. Chat thread, with a one-shot "share my location" action (rule 10 in section 5) and a
   voice-note recording control.
10. New conversation: scan a code, join a channel by passphrase, or start a group.

### Notes

11. Feed of posts with open or recent Community Notes, for users who want to review and rate
    notes directly rather than finding them inline in the Feed.

### Profile

12. Identity card: nickname, scannable code, copy action.
13. Privacy: panic wipe, with a hold-to-confirm gesture.
14. Appearance: light, dark, follow system.

### Documents

Design these as real, final, readable pages. Use no placeholder text.

15. **Privacy policy.** State plainly: Betar collects nothing, there is no server, nothing a
    user writes is sent anywhere Betar controls. Cover what the mesh layer can see, what the
    relay bridge can see, and how to erase everything.
16. **Terms of use.** Short. Acceptable use, no warranty, community maintained.
17. **Safety and limits.** Delivery is best-effort. Image posts are thumbnail-only until
    bridged. Location sharing is one-shot only. Betar's security has not had an independent
    audit yet.
18. **About Betar.** What it is, in 3 sentences. The Bose tribute. The licence. The
    repository link. **Developed by [Konko Maji](https://konkomaji.com/)**.

## 10. Deliverables

1. A locked design system: one theme pair (light and dark), the type scale, and every
   component state.
2. Every screen in section 9, at phone size, in both themes.
3. Component specs for the 4-state delivery indicator, the Community Notes rating control,
   and the mesh-status indicator.
4. Empty, error, and low-battery states for every screen.
5. All document pages, with real final copy.

Developed by [Konko Maji](https://konkomaji.com/)
