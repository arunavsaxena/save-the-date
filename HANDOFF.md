# Handing this over

Paste everything below the line into a fresh Claude session. It assumes no
knowledge of the work so far, and it deliberately carries no keys or ids: ask
Arunav for the invite link and the Sheet link.

---

I'm taking over work on two wedding websites. Read this, then read both
READMEs before changing anything.

## The two sites

Both are static sites served by GitHub Pages from `main`, in sibling folders:

- `~/Documents/arunav_work/save-the-date` → save-the-date.arunav.work
  The bride's side, from Sonia & Tarun Sharma. Everything is in one
  `index.html`: markup, inline styles, a `<style>` block and one `<script>`
  block. No build step, no framework.
- `~/Documents/arunav_work/arunav-rsvp-invitation` → arunav-rsvp-invitation.arunav.work
  The groom's side. `index.html` + `app.js`, same idea.

Read `save-the-date/README.md` and `arunav-rsvp-invitation/README.md` first.
They are current and describe the sections, the RSVP fields, the invite-key
gate and the backend.

## The shared backend (read this twice)

`arunav-rsvp-invitation/apps-script/Code.gs` is ONE Google Apps Script that
serves BOTH sites, bound to the "Wedding RSVPs · Arunav & Shambhavi" Sheet and
published as ONE deployment. Both sites hardcode the same `/exec` URL.

- Groom's side: `kind: 'rsvp'` → "Invitation RSVPs" tab, `INVITE_KEY`, `CONTACTS`
- Bride's side: `kind: 'rsvp-bride'` → "Bride side RSVPs" tab,
  `BRIDE_INVITE_KEY`, `CONTACTS_BRIDE`, its own Drive folder, its own
  rate-limit counter
- The repo copy keeps `DRIVE_FOLDER_ID` blank; the DEPLOYED copy has it set.
  The repo and the deployment drift apart easily, so always diff before you
  paste.

Deploying: Sheet → **Extensions → Apps Script** → paste → save → **Deploy →
Manage deployments → pencil → Version: NEW VERSION → Deploy**. Never create a
new deployment: that mints a new URL and breaks both sites silently.

## Rules that were learned the hard way, not guesses

1. Before pasting into the Apps Script editor, copy the editor's current
   content out and diff it against the repo file. Someone editing in the
   browser without committing is normal here, and a blind paste has already
   wiped one side's code once.
2. After saving, copy it back out and diff again. The clipboard can change
   under you; a wrong paste looks fine on screen.
3. ⌘Z in the editor undoes a bad paste if you catch it before saving.
4. Any value the backend validates against a fixed list must be plain ASCII on
   BOTH sides. A middle dot in a value does not survive the POST into Apps
   Script and silently rejects every reply. Labels may use any character; only
   the `value=` attribute matters.
5. Don't run `setupInviteKey` / `setupBrideInviteKey` / the rotate functions
   unless you own the Drive folders. They run as you and will fail or create
   stray folders in your own Drive.
6. One person in the Apps Script editor at a time. It has no merge.
7. After any backend change, send one real reply from the site, confirm the row
   lands in the right tab, then delete it. Apps Script failures are nearly
   invisible to the guest.

## Front-end gotchas already hit

- Test anything form-related on a REAL iPhone. `input[type=time]` renders as an
  empty, oversized box in iOS Safari and ignores your styling; it is now a
  `<select>`. A local Chromium preview will not show you this.
- Don't build scroll reveals on IntersectionObserver alone. Elements created by
  script after the observer is set up never get observed, and a missed reveal
  means invisible content, not just a missing animation. The page drives both
  reveals and parallax from one rAF-throttled scroll handler. Keep it that way.
- An `<svg>` with a viewBox and `left:0/right:0` sizes itself from its own ratio
  and overflows. Set `width` explicitly.
- Respect `prefers-reduced-motion`; the page already does.

## House style

- Page copy has no em dashes anywhere. Keep it that way.
- Comments explain why a thing is done, not what the line does.
- Verify in a browser before saying something works, and check both phone and
  desktop width.

## Running it locally

```sh
cd ~/Documents/arunav_work/save-the-date && python3 -m http.server 8777
```

Then open `http://localhost:8777/?k=<invite key>`. Without the key the RSVP form
is replaced by a "use the link from your invitation" notice, by design. Ask
Arunav for the invite link and the Sheet link.

## Outstanding

- `CONTACTS_BRIDE` in Script Properties is still empty, so the bride's "For any
  assistance please contact" panel shows three names with no numbers and no
  call or WhatsApp buttons. The format is in `save-the-date/README.md`.
- The repo copy of `Code.gs` may be ahead of what is deployed. Diff it before
  assuming either way.

Start by reading both READMEs and diffing `apps-script/Code.gs` against the live
editor, and tell me what you find before changing anything.
