# Screen Pricing Calculator — Ace Hardware, Barnegat

Single-file HTML calculator for window/door screen rebuild quotes. No build step, no dependencies, no framework. One `index.html` that runs anywhere a browser does.

## Pricing model

```
perScreen  = (length + width) × RATE + PET    — per line (PET only for pet screen)
lineTotal  = perScreen × quantity
subtotal   = Σ lineTotal                       — all lines in the quote
tax        = subtotal × TAX
total      = subtotal + tax
```

Dimensions are in inches. `RATE` is dollars per inch of combined length + width.

### Quotes

Orders with mixed sizes are quoted as multiple lines. The entry form (dimensions, material, quantity) is the current line; **+ Add screen to quote** commits it to the quote list and clears the form for the next size. The screen being typed always counts toward the totals, so a single-screen quote never needs the Add button. Lines can be removed individually (✕) or all at once (**Clear quote**). Quotes live in memory only — a page reload starts fresh.

### Desktop layout

On screens 1024px and wider the app switches to a desktop layout: a dark sidebar (title, ⚙ settings, staff **Instructions**) and a wide entry form. From 1280px up, the quote/total/print panel is pinned in a right-hand column; between 1024–1279px it sits below the form. Phones and tablets keep the single-column layout.

The Instructions list is plain HTML in the `<aside class="side">` block of `index.html` — edit it freely. The highlighted rule at the top: **Are the splines rubber? If no, we can't do the screen.**

### Rules enforced by the app

- **Splines are rubber** — a checkbox that must be ticked before each screen can be added (it clears after every add). A typed screen that isn't checked blocks printing until it's checked or cleared. The printout notes "splines checked: rubber ✓".
- **Phone number is required to print** (at least 10 digits, i.e. with area code). Quoting a price without a phone still works; only the Print button is blocked, with a red hint explaining why. Customer name stays optional.

### Printing (desktop)

Open the app in any desktop browser. Enter the customer phone (required), name (optional), and drop-off date (defaults to today), build the quote, then click **🖨 Print quote** (or press Ctrl+P). One print job produces two pages in the same card layout as the phone view:

1. **Customer Copy** — keep for pickup
2. **Store Copy** — attach to the screen; has blank lines for Ready / Customer called / Picked up

A fully-typed, splines-checked screen that hasn't been added yet is included on the printout. On the desktop, Enter in Length jumps to Width, and Enter in Width adds the screen. **Clear quote** also clears the customer name/phone and resets the drop-off date to today for the next customer. Want both copies on one sheet? Choose "Pages per sheet: 2" in the print dialog.

## Configuration

Rate, tax, and the pet screen upcharge are editable **in the app**: tap the ⚙ button in the header, change the values, done — they apply instantly and are saved on the device (`localStorage`), so no redeploy is needed for a price change. **Reset to defaults** in the same panel returns to the shipped values.

The shipped defaults live at the top of the `<script>` block in `index.html`:

| Constant | Default | Meaning |
|---|---|---|
| `DEFAULT_RATE` | `0.37` | $ per inch (length + width). Applies to aluminum and fiberglass alike. |
| `DEFAULT_TAX` | `0.06625` | NJ sales tax, 6.625%. |
| `DEFAULT_PET` | `8` | Flat $ added **per screen** for pet screen mesh. **Placeholder — set the real number.** |

Note: once a device has saved its own settings, redeploying with new defaults won't change that device — its stored values win until someone taps **Reset to defaults**.

### Materials

- **Aluminum** — base price
- **Fiberglass** — base price (same as aluminum)
- **Pet Screen** — base price + pet upcharge per screen

To add another material: add a `<button class="opt" data-mat="xyz" onclick="pickMat('xyz')">` to the `.opt-grid`, give it a name in `matLabel`, and any upcharge in `eachFor`.

## Local use

No server needed. Open the file:

```bash
open index.html          # macOS
start index.html         # Windows
```

Or serve it if you prefer:

```bash
npx serve .
```

## Repo setup

```bash
mkdir screen-pricing && cd screen-pricing
# drop index.html in here, plus this README.md
git init
git add .
git commit -m "Screen pricing calculator v1"
git branch -M main
git remote add origin https://github.com/YOURNAME/screen-pricing.git
git push -u origin main
```

Suggested structure — that's genuinely the whole thing:

```
screen-pricing/
├── index.html
├── manifest.json
├── sw.js
├── icon-192.png
├── icon-512.png
├── icon-maskable-512.png
└── README.md
```

## Vercel deployment

### Option A — dashboard

1. Push to GitHub (above).
2. [vercel.com/new](https://vercel.com/new) → **Import** the repo.
3. Framework Preset: **Other**
4. Build Command: leave **empty**
5. Output Directory: leave **empty** (or `.`)
6. Install Command: leave **empty**
7. **Deploy.**

Vercel serves `index.html` from the repo root as a static site. Every push to `main` redeploys automatically.

### Option B — CLI

```bash
npm i -g vercel
vercel          # preview deploy
vercel --prod   # production
```

Accept the defaults; when it asks about build settings, skip them.

### Optional `vercel.json`

Not required, but harmless if you want it explicit:

```json
{
  "cleanUrls": true
}
```

## Phone / counter use

Once deployed, add it to the home screen on the counter tablet or your phone:

- **iOS Safari** — Share → Add to Home Screen
- **Android Chrome** — ⋮ → Add to Home screen

It's mobile-first (560px max width, big tap targets on the qty stepper) so it works fine one-handed at the bench.

### Installable PWA (offline-capable)

The app ships as an installable Progressive Web App, so "Add to Home Screen" gives it a real app icon and it keeps working with no signal:

- `manifest.json` — app name, Ace-red theme, icons
- `sw.js` — service worker caching the app shell (network-first for the page so a redeploy is picked up when online, cache fallback when offline)
- `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` — home-screen icons

Requirements: the service worker only registers over **HTTPS** (or `localhost`) — it will not activate from a `file://` open, but Vercel serves HTTPS so it just works once deployed. Bump the `CACHE` version string in `sw.js` whenever you change a cached file so tablets pull the new version.

## Notes / open items

- The pet screen upcharge default ($8) is still a guess. Set the real number in ⚙ (or `DEFAULT_PET`) before anyone quotes off this.
- Prices are per screen, not per square foot. That's the store's existing formula, kept as-is on purpose.
- Tax is applied to the full subtotal. If screen labor is ever treated differently for NJ sales tax purposes, that logic would need to split out.
