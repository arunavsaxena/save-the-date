# Shambhavi & Arunav — invitation from the bride's family

Static site for **save-the-date.arunav.work**. The page is written in the voice of
the bride's parents, **Sonia & Tarun Sharma**, and collects RSVPs for both days.

Handing this to another developer? `HANDOFF-PROMPT.md` is written to be pasted into a
fresh Claude session: it covers both sites, the shared backend and the traps.

```
index.html          Whole site — markup, styles and behaviour in one file
HANDOFF-PROMPT.md   Briefing for whoever picks this up next
assets/             Photography and the Evara logo
CNAME               Custom domain for GitHub Pages
```

## Sections

Landing → hero → invitation (Ganesh shloka and the parents' words) → venue →
countdown → events (Day One and Day Two) → RSVP → who to call → footer.

## Behaviour

- **Landing screen** — the same scene as the groom's site: a watercolour of the
  resort at golden hour behind, the garden and its brass lanterns in front, and
  the invitation on frosted glass between them, drifting a little with the
  pointer. The arch here is domed rather than cusped. Artwork lives in
  `assets/landing/` and is shared with `arunav-rsvp-invitation`. A tap anywhere
  lifts it away; Enter, Space or Escape do too, and scroll is locked until then.

- **Music** — `assets/shloka.mp3` (Om Mangalam Bhagwan Vishnu, 42 seconds,
  looped), started by the tap that opens the envelope, with a mute button at the
  bottom left. `AUDIO_SRC` also accepts a YouTube link, but phones refuse to start
  a YouTube embed from a tap on anything other than the player itself, so a file
  is the only thing that plays reliably on a phone. Leave `AUDIO_SRC` empty to
  keep the page silent and hide the button.
- **Motion** — each section settles in as it arrives, and a few layers drift
  against the scroll: the hero photograph, Ganesh above the shloka, the washes
  behind the venue card, the glow on the countdown and footer, and the gallery
  photographs inside their frames. Both run off one rAF-throttled scroll pass
  rather than an IntersectionObserver, so nothing can stay invisible because a
  callback never arrived, and both are skipped entirely for
  `prefers-reduced-motion`.

- **Countdown** — ticks to 27 Jan 2027, 12:00 PM IST (Welcome Lunch). Change
  `TARGET` at the top of the script block to retarget it at the Vivah Sanskar
  (`2027-01-28T11:00:00+05:30`).
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
| Arrival date and time | 26–28 Jan, time in quarter hours. Both are selects: iOS Safari draws `input[type=time]` in its own box, ignoring the surrounding type |
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

### Editing and deploying the script

The repo copy is the source of truth, but it is **not** what is running: the
deployed copy has `DRIVE_FOLDER_ID` filled in, and the repo keeps it blank. So
every change goes out like this.

1. **Open the editor.** The Sheet → **Extensions → Apps Script**. Both sides of
   the wedding live in one `Code.gs`; the bride's half is everything named
   `BRIDE_*`, plus `saveBrideRsvp_`, `brideSheet_` and `setupBrideInviteKey`.
2. **Check the editor against the repo first.** Click in the code, select all,
   copy, and diff it against `apps-script/Code.gs` in `arunav-rsvp-invitation`.
   If they differ, someone edited in the browser without committing: work out
   which is newer before you overwrite anything.
3. **Make the change in the repo file**, not in the browser, and keep
   `DRIVE_FOLDER_ID` and `BRIDE_DRIVE_FOLDER_ID` blank there.
4. **Keep every validated value ASCII.** Anything the backend checks against a
   fixed list (the days, the modes, the dates, the assistance options) must be
   plain ASCII on both sides. A middle dot does not survive the POST into Apps
   Script and every reply is rejected.
5. **Paste it in.** In the editor: select all, paste, then restore the
   `DRIVE_FOLDER_ID` line the deployed copy had. Save (⌘S / Ctrl+S) and wait for
   "Saved to Drive". A syntax error blocks the save, which is the first check.
6. **Verify what actually landed.** Select all, copy, and diff against the file
   you meant to deploy. The editor is easy to paste into by accident, and a
   clipboard that changed under you is hard to spot by eye.
7. **Deploy.** **Deploy → Manage deployments → pencil → Version: New version →
   Deploy.** The deployment id, and so the `/exec` URL, stays the same.
8. **Send one real reply** from the site and confirm the row lands in the right
   tab, then delete that row. Apps Script errors are invisible to the guest
   beyond a short message, so a live reply is the only real test.
9. **Commit and push the repo copy**, so the next person does not start from a
   version that is behind what is running.

To undo a bad paste before saving: ⌘Z / Ctrl+Z in the editor restores the
previous content. If a bad version is already deployed, redeploy from the repo
file rather than editing the live one by hand.

### If someone else is deploying

Script access is granted by sharing the **Sheet** as Editor: the script is bound
to it and has no sharing of its own. That also hands over every RSVP, so decide
on the data access first. Two rules for whoever has it:

- **Deploy a new version of the existing deployment, never a new deployment.** A
  new one gets a new URL and silently breaks both sites.
- **Don't run `setupInviteKey`, `setupBrideInviteKey` or the rotate functions
  from the editor.** Those run as them, and the Drive folders aren't shared with
  them, so they'll fail or create stray folders in their own Drive. Keep those
  for the owner.

One person in the editor at a time, working from the repo file: the Apps Script
editor has no merge, so whoever saves last wins.

### Phone numbers

Names and roles (Tarun, Sonia, Vedika) are in `index.html`; the numbers are not.
They live in the Apps Script project's Script Properties under **`CONTACTS_BRIDE`**
as JSON, and are served only to visitors holding a valid invite key. The
parents carry no role label, so leave those blank:

```json
[
  {"name": "Tarun Sharma", "role": "", "phone": "+91 98xxx xxxxx"},
  {"name": "Sonia Sharma", "role": "", "phone": "+91 98xxx xxxxx"},
  {"name": "Vedika", "role": "RSVP co-ordinator", "phone": "+91 98xxx xxxxx"}
]
```

## Before sharing with guests

- [x] Invite key created; share the `?k=…` link, not the bare domain
- [x] Backend deployed and test replies confirmed in **Bride side RSVPs**
- [ ] Fill `CONTACTS_BRIDE` with the three numbers
- [ ] Listen to the page once on a phone, to check the music starts and mutes
- [x] Music file added (`assets/shloka.mp3`)
- [ ] Confirm the Google Maps link points at the right Evara property
- [ ] Push the site so GitHub Pages serves the new page
