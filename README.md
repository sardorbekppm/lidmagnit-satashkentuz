# SATashkent — free SAT course landing

Mobile-first landing for the free **SAT English + SAT Math** video course.
Funnel: **Meta / Telegram ads → this landing → thanks popup → YouTube playlist**.

Single page. Clicking any CTA opens a thank-you **popup** that fires the conversion,
then forwards to the YouTube playlist. Nothing else to load.

## ✅ Already configured — just deploy
| Setting | Value (live in `index.html`) |
|---|---|
| GTM container | `GTM-WL46HXLT` (loader + noscript) |
| Meta Pixel | `955344730737867` (safety-net fallback) |
| YouTube playlist | `SAT Full Course \| @satashkent` — 26 lessons |
| Popup floor / ceiling | `200ms` / `900ms` (see **Redirect speed**) |

Nothing needs editing to go live. To tweak later, everything sits in the
**CHANGEABLE CONFIG** block at the top of `index.html`.

## What the page says
Two courses, one playlist:

| Course | Topics | Proof point |
|---|---|---|
| SAT English | 11 | mentors whose students graduated with **1500+** scores |
| SAT Math | 15 | mentors whose students scored **780+** on SAT Math |

Both promise the same explanations, the same presentations, and (for Math) the same
Desmos tricks used in SATashkent's real classes — for free.

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
reporting by channel in GTM / Meta. The YouTube link is used verbatim — YouTube ignores
unknown query params, so attribution rides the dataLayer instead of the URL.

### Events pushed to the dataLayer
| Event | When |
|---|---|
| `lp_view` | Page loads |
| `cta_click` | Any CTA clicked |
| `lead` | Thanks popup opens — **this is the conversion** |

### In GTM (`GTM-WL46HXLT`)
1. Add your **Meta Pixel** tag (or keep the built-in fallback and set `metaPixelId:""`).
2. Trigger it on the custom event **`lead`** — that is the popup opening.
3. `source` is in the dataLayer if you want per-channel breakdowns.

**Your `ViewContent` custom conversion goes on the `lead` event.** It fires the moment the
popup opens, and the redirect deliberately waits for GTM to confirm every tag bound to it
has run before leaving the page — so the conversion cannot be lost to the navigation.
Nothing else needs wiring.

> ⚠️ **Three things already live in the container that this code doesn't know about**,
> found by watching the network on a real page load:
> - A **second Meta Pixel** (`2569268646837340`) fires its own `PageView`, alongside the
>   `955344730737867` in this file. Every visit is counted by two pixels.
> - A **Telegram Ads pixel** (`telegram.org/pxl`) still fires — left over from the old
>   Telegram funnel this page no longer uses.
> - A Meta **`Subscribe`** event fires on `cta_click`, also from the Telegram days.
>
> None of these are fixable from this repo. If you add `ViewContent` on `lead` without
> clearing them out, one click can report `Subscribe` + `Lead` + `ViewContent` across two
> pixels. Worth a cleanup pass in GTM before optimizing campaigns against it.

### In Meta Ads Manager
The popup doesn't change the URL, so use an **event-based** Custom Conversion pointing
at the **`Lead`** event (not a `/thanks` URL rule), then optimize campaigns for it.

## Redirect speed
Clicking a CTA opens the popup, fires the conversion, then forwards to YouTube. The old
build waited a flat `350ms` and hoped that was long enough for the beacons to leave.

It now waits on the conversion itself. The `lead` push carries a GTM **`eventCallback`**,
which GTM invokes once every tag bound to that event has actually run — measured at
**17ms** against the live container. Two bounds keep that honest:

| Config | Default | What it does |
|---|---|---|
| `redirectMinMs` | `200` | Floor. The popup stays up at least this long, so it reads as a confirmation rather than a flash. |
| `redirectMaxWaitMs` | `900` | Ceiling. If GTM stalls, leave anyway — but the conversion gets first claim on the time. |

So the redirect lands at **200ms** (the floor, since tags confirm in 17ms) instead of
350ms — faster *and* now provably fired rather than assumed. Ad blockers are handled too:
if the tag scripts fail to load there is nothing to wait for, and the ceiling drops to
150ms.

The destination is the `www.youtube.com` URL, not the bare `youtube.com` one. The short
form 301s to it, and skipping that hop saves a full round trip on a phone connection.

## Performance
**Nothing third-party touches the critical path.** GTM and the Meta Pixel used to load in
`<head>` on first byte; between them they pulled ~1.7s of requests (including two
`.on.aws` calls over 1.1s each) while the page was still settling. Now the `<head>` only
sets up the queues — `dataLayer.push()` and `fbq()` are safe from the first millisecond —
and the scripts themselves load **150ms after first paint** (`tagDelayMs`).

Nothing is lost by waiting: the queues replay in order once the scripts arrive, so
`lp_view` still reports. Three guards cover the edges:

- **A click before the timer** force-loads the tags immediately, so an impatient visitor
  still converts.
- **A background tab** throttles `requestAnimationFrame`, so a 2.5s fallback loads them
  regardless.
- **A blocked request** sets a flag, and the redirect stops waiting on tags that will
  never fire.

Everything else was already lean and stayed that way: fonts self-hosted in `fonts/`
(the same variable woff2 files Google Fonts serves, preloaded, `font-display: swap`),
CSS/JS inline, and `_headers` / `vercel.json` caching the fonts for a year. The page also
preconnects to `www.youtube.com` so the redirect's TLS handshake is already warm.

> The two font files are **47KB (Inter)** and **27KB (Plus Jakarta Sans)**. They don't
> block paint — `font-display: swap` renders in a system font first — but they are the
> largest thing on the wire. If you ever want the page leaner on a slow connection,
> subsetting Inter to the glyphs actually used is the biggest remaining win.
