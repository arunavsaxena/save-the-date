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

## Collecting real RSVPs in a private Google Sheet

Guests only ever see the form on this page. Behind it, the page POSTs to a
Google Form, and the Form drops each reply into its linked Sheet — which stays
private to whoever owns it. Nothing on the page links out to Google.

The Form and its field ids are already wired into `index.html`. This section
records how, so it can be rebuilt or repointed later.

**Form:** `1FAIpQLSewM_dhebvTa_yj5_W1KRt9gX3bUjafmIamG2t1-gl1lzRrcg`

| Question | Type | Field id | Sent from |
|----------|------|----------|-----------|
| Full Name | Short answer | `entry.1224194294` | `name` |
| Phone number | Short answer | `entry.883360929` | `phone` |
| Email ID | Short answer | `entry.857968569` | `email` |
| Functions attending | Short answer | `entry.771900964` | `functionsText` |
| Number of guests | Short answer | `entry.1672330680` | `guests` |
| Special requests | Paragraph | `entry.355334609` | `requests` |
| Message for us | Paragraph | `entry.1674024418` | `message` |

Every question is deliberately **not required** — the page validates first, and
a required question Google cannot see filled rejects the whole reply.
"Functions attending" is deliberately **Short answer**: a Checkboxes question
rejects any value that does not match one of its options exactly.

### Rebuilding this mapping

Field ids live in the `FB_PUBLIC_LOAD_DATA_` blob on the form's public page:

```sh
curl -sL "https://docs.google.com/forms/d/e/<FORM_ID>/viewform" \
  | grep -o 'FB_PUBLIC_LOAD_DATA_ = .*'
```

Each question appears as `[[<entry id>, ...]]` in question order. The form id
is the long token in the share link.

### What to expect

Google Forms serves no CORS headers, so the page posts `mode: "no-cors"` and
gets an **opaque response** back — it cannot tell a saved reply from a rejected
one, and always shows the thank-you screen. Two consequences:

- **Send one test reply and confirm it lands in the Sheet** before sharing the
  link with guests. A wrong `entry.` id fails silently.
- Every reply is also written to `localStorage` under `sa-wedding-rsvps`, so a
  guest's own browser keeps a copy even if the send failed.

### Alternative

`RSVP_ENDPOINT` takes any URL that accepts a JSON `POST` (Formspree, an Apps
Script web app). It runs *alongside* the Google Form, so you can use both.
Payload:

```json
{
  "name": "", "phone": "", "email": "",
  "functions": ["Welcome Lunch", "Wedding"],
  "functionsText": "Welcome Lunch, Wedding",
  "guests": "2", "requests": "", "message": "",
  "submittedAt": "2026-09-05T00:00:00.000Z"
}
```

## Before sharing with guests

- [ ] Replace the two `+91 00000 00000` placeholders in the footer
- [ ] Delete the two `ZZ ... TEST` rows from the responses Sheet
- [ ] Confirm the Google Maps link points at the right Evara property
