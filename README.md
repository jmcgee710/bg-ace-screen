# Screen Pricing Calculator — Ace Hardware, Barnegat

Single-file HTML calculator for window/door screen rebuild quotes. No build step, no dependencies, no framework. One `index.html` that runs anywhere a browser does.

## Pricing model

```
base       = (length + width) × RATE
perScreen  = base + materialUpcharge
subtotal   = perScreen × quantity
tax        = subtotal × TAX
total      = subtotal + tax
```

Dimensions are in inches. `RATE` is dollars per inch of combined length + width.

## Configuration

All pricing lives in three constants at the top of the `<script>` block in `index.html`. Change these, save, redeploy.

| Constant | Current | Meaning |
|---|---|---|
| `RATE` | `0.37` | $ per inch (length + width). Applies to aluminum and fiberglass alike. |
| `PET_UPCHARGE` | `8` | Flat $ added **per screen** for pet-resistant fiberglass. **Placeholder — replace with the real number.** |
| `TAX` | `0.06625` | NJ sales tax, 6.625%. |

```js
// ——— EDIT THESE to match your store ———
const RATE = 0.37;        // $ per inch (length + width)
const PET_UPCHARGE = 8;   // $ per screen extra for pet mesh
const TAX = 0.06625;      // NJ sales tax 6.625%
// ———————————————————————————————
```

### Materials

- **Aluminum** — base price
- **Fiberglass** — base price (same as aluminum)
- **Pet Fiberglass** — base + `PET_UPCHARGE` per screen

To add a material later: add a `<button class="opt" data-mat="xyz" onclick="pickMat('xyz')">` to the `.opt-grid`, then add a case to `upchargeFor()`.

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

- `PET_UPCHARGE` is still a guess. Set it to the real number before anyone quotes off this.
- Prices are per screen, not per square foot. That's the store's existing formula, kept as-is on purpose.
- Tax is applied to the full subtotal. If screen labor is ever treated differently for NJ sales tax purposes, that logic would need to split out.
