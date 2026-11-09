# Shambhavi & Arunav — invitation from the bride's family

Static site for **save-the-date.arunav.work**. The page is written in the voice of
the bride's parents, **Sonia & Tarun Sharma**, and collects RSVPs for both days.

```
index.html          Whole site — markup, styles and behaviour in one file
assets/             Photography and the Evara logo
assets/gallery/     Gallery photographs (see below)
CNAME               Custom domain for GitHub Pages
```

## Sections

Envelope → hero → invitation (Ganesh shloka and the parents' words) → venue →
countdown → events (Day One and Day Two) → RSVP → gallery → footer.

## Behaviour

- **Intro envelope** — a drawn burgundy envelope, no photograph: tapping the wax
  seal swings the flap open and lifts the screen away. The animation is the one
  from `arunav-rsvp-invitation`, recoloured. Scroll is locked until it is opened,
  and Enter, Space or Escape open it too.
- **Music** — starts as the envelope opens, with a mute button at the bottom left.
  `AUDIO_SRC` takes either a YouTube link (played through a hidden player) or a
  file in `assets/`. It is currently the YouTube link
  `https://www.youtube.com/watch?v=uJrRbYEaMVM`. Browsers block sound until a tap,
  so the first note always follows the tap on the seal; leave `AUDIO_SRC` empty to
  keep the page silent and hide the button.
- **Countdown** — ticks to 27 Jan 2027, 12:00 PM IST (Welcome Lunch). Change
  `TARGET` at the top of the script block to retarget it at the Vivah Sanskar
  (`2027-01-28T11:00:00+05:30`).
- **Gallery** — four portrait photographs in `assets/gallery/` (`01.jpg` to
  `04.jpg`), listed in `GALLERY`. To add more, drop them in that folder, resize to
  about 1500px on the long side, and add the file names to that list. The frames
  are 4:5, so portrait photographs sit best; while the list is empty, six empty
  frames hold the section's shape.
- **RSVP** — posted to the shared Apps Script backend; a copy is kept in the
  guest's own browser under `localStorage` `std-wedding-rsvps`.

## The invite link

The page only accepts replies from guests who open it with the key in the query
string: `https://save-the-date.arunav.work/?k=<key>`. The key is stored in the
guest's browser, removed from the address bar, and sent with every request.
Without it the form is replaced by "please use the link from your invitation", and
the family's phone numbers are not shown.

The key already exists. To see the link again, open the Apps Script project
(below) and run **`setupBrideInviteKey`** — it only logs the existing link.
Run `rotateBrideInviteKey` if the link leaks, then send guests the new one.

## What the form collects

| Field | Notes |
|-------|-------|
| Name, phone | Required; one reply per family |
| Days attending | Day 1 · Wed 27 Jan, Day 2 · Thu 28 Jan — at least one. Stored with a hyphen (`Day 1 - Wed 27 Jan`): see the note below |
| Guests | How many are coming, counting themselves (1 to 10) |
| Mode of transport | Road, train or flight |
| Arrival date and time | 26–28 Jan |
| Special assistance | Accommodation for driver · Pick-up at the railway station (then which station, and the train) · Special medical care (then what to arrange) |
| ID proof per guest | **Optional.** Name as on ID, ID type, and a photo or PDF up to 5 MB |

**ID proofs never block a reply.** Each guest card may be left empty. Files are
uploaded first; if one fails, the RSVP is still saved and the thank-you screen
says which to send to Vedika or Tarun instead. The sheet records how many
arrived, and links the folder when there is one.

**Keep validated values ASCII.** Any value the backend checks against a fixed list
must be plain ASCII on both sides. The day values first used a `·` and every reply
was rejected with "Please choose the day." — the character does not survive the
POST into Apps Script. Labels on the page can still use `·`; only the `value=`
attribute matters. Free text (names, stations, notes) is stored as sent.

## The backend

Replies go to the **same Apps Script web app the groom's side uses** —
`apps-script/Code.gs` in the sibling `arunav-rsvp-invitation` repo, bound to the
RSVP Google Sheet. This page posts `kind: "rsvp-bride"` (and `kind: "file"` with
`side: "bride"` for ID proofs), which the script writes to its own **"Bride side
RSVPs"** tab, with its own invite key, its own rate-limit counters and its own
private Drive folder, **Bride side RSVP ID proofs**.

To change the backend, edit it in that repo and redeploy from the Apps Script
editor (**Deploy → Manage deployments → Edit → Version: New version**); the
`RSVP_ENDPOINT` URL in `index.html` stays the same. Both sites share that
deployment, so always deploy a new *version*, never a new deployment.

### Phone numbers

Names and roles (Tarun, Sonia, Vedika) are in `index.html`; the numbers are not.
They live in the Apps Script project's Script Properties under **`CONTACTS_BRIDE`**
as JSON, and are served only to visitors holding a valid invite key:

```json
[
  {"name": "Tarun Sharma", "role": "Father", "phone": "+91 98xxx xxxxx"},
  {"name": "Sonia Sharma", "role": "Mother", "phone": "+91 98xxx xxxxx"},
  {"name": "Vedika", "role": "RSVP co-ordinator", "phone": "+91 98xxx xxxxx"}
]
```

## Before sharing with guests

- [x] Invite key created; share the `?k=…` link, not the bare domain
- [x] Backend deployed and test replies confirmed in **Bride side RSVPs**
- [ ] Fill `CONTACTS_BRIDE` with the three numbers
- [ ] Listen to the page once on a phone, to check the music starts and mutes
- [x] Gallery photographs added (four, in `assets/gallery/`)
- [ ] Confirm the Google Maps link points at the right Evara property
- [ ] Push the site so GitHub Pages serves the new page
