# SATashkent — APPDATE landing

Mobile-first pre-launch landing for the August APPDATE campaign.
Funnel: **Meta / Telegram ads → this landing → thanks popup → Telegram channel**.

Single page. Clicking any CTA opens a thank-you **popup** that fires the conversion,
then forwards to the Telegram channel. Nothing else to load.

## ✅ Already configured — just deploy
| Setting | Value (live in `index.html`) |
|---|---|
| GTM container | `GTM-WL46HXLT` (loader + noscript) |
| Meta Pixel | `955344730737867` (safety-net fallback) |
| Telegram | `https://t.me/+3ND_eUdCydU2ZDAy` |
| Countdown target | `2026-08-15 10:00` (UZ time) — change if the date differs |

Nothing needs editing to go live. To tweak later, everything sits in the
**CHANGEABLE CONFIG** block at the top of `index.html`.

## Files
| File | What it is |
|---|---|
| `index.html` | The whole thing — page, popup, config, tracking |
| `logo.svg` | SATashkent crest (logo + favicon) |
| `fonts/` | Self-hosted Inter + Plus Jakarta Sans (variable woff2, latin subset) |
| `_headers` / `vercel.json` | Long-lived cache headers for fonts (Netlify / Vercel) |
| `og-image.png` | **Optional** — add a 1200×630 image for Telegram/Meta link previews |

## Deploy (GitHub → Netlify or Vercel)
1. Push `index.html` + `logo.svg` to a GitHub repo (root is fine).
2. **Netlify:** New site from Git → pick repo → no build command, publish dir = `/`.
   **Vercel:** New Project → import repo → Framework preset = **Other** → deploy.
3. Every push to `main` auto-redeploys. Pure static, no build step.

## Ad links (channel attribution)
Send each channel to the landing with a `utm_source`:

- Meta ads → `https://yoursite.com/?utm_source=meta`
- Telegram ads → `https://yoursite.com/?utm_source=telegram`

That source flows into every dataLayer event as a `source` property, so you can split
reporting by channel in GTM / Meta. The Telegram link is a **private channel invite**,
so it's used as-is (the `?start=` bot trick doesn't apply to channels — switch the CTA
to a bot link and the code will auto-tag `?start=<source>`).

### Events pushed to the dataLayer
| Event | When |
|---|---|
| `lp_view` | Page loads |
| `cta_click` | Any CTA clicked |
| `lead` | Thanks popup opens — **this is the conversion** |

### In GTM (`GTM-WL46HXLT`)
1. Add your **Meta Pixel** tag (or keep the built-in fallback and set `metaPixelId:""`).
2. Trigger it on the custom event **`lead`**.
3. `source` is in the dataLayer if you want per-channel breakdowns.

### In Meta Ads Manager
The popup doesn't change the URL, so use an **event-based** Custom Conversion pointing
at the **`Lead`** event (not a `/thanks` URL rule), then optimize campaigns for it.

## Performance
Nothing render-blocking leaves the origin: fonts are self-hosted in `fonts/`
(the same variable woff2 files Google Fonts serves, preloaded, `font-display: swap`),
CSS/JS are inline, and GTM + Meta Pixel load async. `_headers` (Netlify) and
`vercel.json` (Vercel) cache the fonts for a year, so repeat visits skip them entirely.
