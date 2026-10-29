# MitraaPay — coming soon site

A single-file, zero-dependency launch page for MitraaPay. No build step, no framework,
no npm install. `index.html` is the whole site.

---

## Deploy on GitHub Pages (5 minutes)

1. Create a new repo on GitHub, e.g. `mitraapay-site` (public, no README).
2. On your machine:

   ```bash
   mkdir mitraapay-site && cd mitraapay-site
   # copy index.html into this folder
   git init
   git add index.html README.md
   git commit -m "MitraaPay launch page"
   git branch -M main
   git remote add origin https://github.com/<your-username>/mitraapay-site.git
   git push -u origin main
   ```

3. On GitHub: **Settings → Pages → Build and deployment**
   - Source: **Deploy from a branch**
   - Branch: **main**, folder: **/ (root)** → Save
4. Wait ~60 seconds. The site is live at
   `https://<your-username>.github.io/mitraapay-site/`

### Custom domain (mitraapay.com)

1. Add a file named `CNAME` in the repo root containing exactly one line:

   ```
   mitraapay.com
   ```

2. At your DNS provider, point the domain at GitHub:

   | Type  | Name  | Value |
   |-------|-------|-------|
   | A     | `@`   | `185.199.108.153` |
   | A     | `@`   | `185.199.109.153` |
   | A     | `@`   | `185.199.110.153` |
   | A     | `@`   | `185.199.111.153` |
   | CNAME | `www` | `<your-username>.github.io` |

3. GitHub → Settings → Pages → Custom domain → enter `mitraapay.com`, then tick
   **Enforce HTTPS** once the certificate is issued (can take up to an hour).

Nginx / Vercel / S3 work the same way — it is one static file, drop it at the web root.

---

## Things to change before you go public

Everything below is a placeholder. Search `index.html` for the string in the left column.

| Search for | What it is |
|---|---|
| `LAUNCH_ISO` | Launch date and time, IST. Drives the countdown. |
| `LAUNCH_TEXT` | The line under the countdown. |
| `hello@mitraapay.com` | General enquiry address (appears twice — text and copy button). |
| `partners@mitraapay.com` | Partnership address (appears twice). |
| `Gurugram, Haryana, India` | Office address block. |
| `Q4 2026` … `Q3 2027` | The four roadmap cards. |
| `19,000+`, `63M+`, `<10 min`, `100%` | The "Why now" figures. These are market context and design targets, and the page says so — swap for audited numbers at launch. |
| `© 2026 MitraaPay` | Footer line; add the legal entity name once incorporated. |
| Legal paragraph in `<footer>` | Written for a tech/credit-facilitation platform lending through RBI-registered NBFC partners. Have it reviewed before launch, and replace it entirely if MitraaPay itself holds the licence. |

Logo: the sun mark is inline SVG at the top of `<body>` (`<svg class="mark">`, used in
the nav and the footer). Replace both copies, or point them at a `logo.svg` you commit
alongside `index.html`.

---

## Wiring up the waitlist form

Right now the form validates in the browser and shows a confirmation — nothing is sent
anywhere. Pick one:

**Formspree (fastest, no server).** Sign up, create a form, then edit the `<form>` tag:

```html
<form class="form" id="waitlist" action="https://formspree.io/f/XXXXXXX" method="POST">
```

…and delete the `e.preventDefault();` line plus the custom `submit` handler in the script.

**Google Form (free).** Create a Google Form, open it, view source, and copy the
`entry.XXXXXXX` name for each field onto the matching `name=` attribute in the HTML,
then post to `https://docs.google.com/forms/d/e/<form-id>/formResponse`.

**Your own API (recommended, since you already run FastAPI).** Keep the handler and
replace the confirmation with:

```js
fetch("https://api.mitraapay.com/waitlist", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: name, mobile: mobile, email: ..., city: ..., role: ... })
}).then(function () { show("<b>You're on the list.</b> …", true); });
```

GitHub Pages is static, so the endpoint has to live elsewhere (your EC2 box is fine) and
must send `Access-Control-Allow-Origin` for the site's domain.

Store mobile numbers as personal data: consent text is already on the form, keep the
record of it, and don't log numbers in plain text where they aren't needed.

---

## Notes on the build

- One file, ~39 KB, no external JavaScript. The only network request is Google Fonts
  (Unbounded, IBM Plex Sans / Mono / Sans Devanagari). If you want fully offline, host
  the woff2 files yourself and swap the `<link>` for a `@font-face` block.
- The hero map is a `<canvas>`: ~1,500 points sampled inside a simplified outline of
  India, with twelve pulsing hubs. It redraws on resize and respects
  `prefers-reduced-motion`.
- Dark single-theme by design — colours are declared as tokens in the `:root` block at
  the top of `<style>`, so a palette change is six lines.
- Responsive down to ~360 px, no horizontal scroll, keyboard focus states throughout.
