# Shambhavi & Arunav — Save the Date

Static site for **save-the-date.arunav.work**, coded from the Claude Design canvas
"Save The Date" (project `a236be5b-1043-4836-acc4-b97b9f752d55`).

## Structure

```
index.html          Whole site — markup, styles and behaviour in one file
assets/             Photography and the Evara logo
CNAME               Custom domain for GitHub Pages
```

## Behaviour

- **Intro envelope** — full-screen overlay, dismissed by the *Open* button. Scroll is
  locked until it is dismissed.
- **Countdown** — ticks to 27 Jan 2027, 12:30 PM IST (celebrations begin). Change
  `TARGET` at the top of the script block to retarget it at the wedding ceremony
  (`2027-01-28T11:00:00+05:30`).
- **RSVP** — replies are written to `localStorage` under `sa-wedding-rsvps`. This
  keeps them *in the guest's own browser only*; nothing reaches you.

## Collecting real RSVPs

Set `RSVP_ENDPOINT` at the top of the script block in `index.html` to a URL that
accepts a JSON `POST`. Options that work from a static page:

- **Formspree** — `https://formspree.io/f/<id>`
- **Google Apps Script** — a Sheet-backed web app deployed with "Anyone" access
- **Airtable / Basin / Getform** — any of their JSON endpoints

Payload shape:

```json
{
  "name": "", "phone": "", "email": "",
  "functions": "Welcome Lunch, Wedding",
  "guests": "2", "requests": "", "message": "",
  "submittedAt": "2026-09-05T00:00:00.000Z"
}
```

## Before sharing with guests

- [ ] Replace the two `+91 00000 00000` placeholders in the footer
- [ ] Wire up `RSVP_ENDPOINT`
- [ ] Confirm the Google Maps link points at the right Evara property
