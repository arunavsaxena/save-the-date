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

### 1. Build the Form

Create a form at [forms.new](https://forms.new) with these seven questions, in
this order. Make every one a **Short answer** except the last two, which are
**Paragraph**:

| # | Question | Type |
|---|----------|------|
| 1 | Name | Short answer |
| 2 | Phone number | Short answer |
| 3 | Email ID | Short answer |
| 4 | Functions attending | Short answer |
| 5 | Number of guests | Short answer |
| 6 | Special requests | Paragraph |
| 7 | Message for us | Paragraph |

Use **Short answer**, not Checkboxes, for "Functions attending". A Google
Forms checkbox question rejects the whole submission if a value does not match
one of its options character-for-character; a text question accepts anything.
Leave every question **not required** — validation already happens on the page,
and a required question Google cannot see filled will reject the reply.

Then, in the form's **Responses** tab, click **Link to Sheets**.

### 2. Read the field ids

Open the form's **Preview** (the eye icon), view source, and search for
`entry.`. Each question has an id like `entry.1234567890`, in question order.
The form's own id is the long token in its share link:

```
https://docs.google.com/forms/d/e/<THIS PART>/viewform
```

### 3. Fill in the config

Near the top of the `<script>` block in `index.html`:

```js
var GOOGLE_FORM = {
  id: "1FAIpQLSc...",            // from the share link
  fields: {
    name:      "entry.1111111111",
    phone:     "entry.2222222222",
    email:     "entry.3333333333",
    functions: "entry.4444444444",
    guests:    "entry.5555555555",
    requests:  "entry.6666666666",
    message:   "entry.7777777777"
  }
};
```

Any field left as `""` is simply not sent. Leaving `id` empty disables the
Google Form path entirely.

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
- [ ] Create the Google Form, fill in `GOOGLE_FORM`, and send one test reply
- [ ] Confirm the Google Maps link points at the right Evara property
