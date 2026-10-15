# SATashkent — free SAT course landing

One page. Funnel: **Meta / Telegram ads → this landing → thanks popup → YouTube playlist**.

Clicking the CTA opens a thank-you **popup** that fires the conversion, then forwards to the
playlist. Nothing else to load.

## ✅ Already configured — just deploy
| Setting | Value (live in `index.html`) |
|---|---|
| GTM container | `GTM-WL46HXLT` (loader + noscript) |
| Meta Pixel | `955344730737867` (safety-net fallback) |
| YouTube playlist | `SAT Full Course \| @satashkent` — 26 lessons |
| Popup floor / ceiling | `200ms` / `900ms` (see **Redirect speed**) |

Everything sits in the **CHANGEABLE CONFIG** block at the top of `index.html`.

## Files
| File | What it is |
|---|---|
| `index.html` | The whole site — page, popup, config, tracking, and the nav lockup (inlined SVG) |
| `mark.svg` | Shield only — favicon and the popup logo |
| `img/` | `lesson-math-12.jpg` — the hero thumbnail, phones only |
| `logo.svg` | Full crest with the circular wordmark (kept, currently unused) |
| `fonts/` | Self-hosted Source Serif 4 + Outfit, latin subset |
| `_headers` / `vercel.json` | Long-lived cache headers (Netlify / Vercel) |
| `og-image.png` | **Optional** — add a 1200×630 image for link previews |

## Type system
Y Combinator's pairing, self-hosted:

| Role | Face |
|---|---|
| Headline, footnote, section titles, popup title, **buttons** | Source Serif 4 — roman for the headline, *italic* for emphasis, headings and buttons |
| Wordmark, eyebrows, card titles, lists | Outfit |

Buttons are serif *italic* on purpose — that is how YC sets its "Apply" pill.

The serif roman is the **single 400 instance, 20KB**. Google's CSS API hands you a variable
file unless you ask for one weight: the weight-axis cut is 51KB, and the full cut with the
optical-size axis is 127KB. The page sets no serif weight but 400, so the instance is enough.
Ask the API for `wght@400`, not a range.

## Deploy
Live on Netlify at **https://satashkent.netlify.app/**. Every push to `main` auto-redeploys.
Pure static, no build step.

## Ad links (channel attribution)
Send each channel to the landing with a `utm_source`:

- Meta ads → `https://satashkent.netlify.app/?utm_source=meta`
- Telegram ads → `https://satashkent.netlify.app/?utm_source=telegram`

That source flows into every dataLayer event as a `source` property. The YouTube link is used
verbatim — YouTube ignores unknown query params, so attribution rides the dataLayer, not the URL.

### Events pushed to the dataLayer
| Event | When |
|---|---|
| `lp_view` | Page loads |
| `cta_click` | CTA clicked |
| `lead` | Thanks popup opens — **this is the conversion** |

### In GTM (`GTM-WL46HXLT`)
**Your `ViewContent` custom conversion goes on the `lead` event.** It fires the moment the popup
opens, and the redirect waits for GTM to confirm every tag bound to it has run — so the
conversion cannot be lost to the navigation.

> ⚠️ **Three things live in the container that this code doesn't know about**, found by watching
> the network on a real page load:
> - A **second Meta Pixel** (`2569268646837340`) fires its own `PageView` alongside the
>   `955344730737867` in this file. Every visit is counted by two pixels.
> - A **Telegram Ads pixel** (`telegram.org/pxl`) still fires — left over from the old funnel.
> - A Meta **`Subscribe`** event fires on `cta_click`, also from the Telegram days.
>
> None are fixable from this repo. Worth a cleanup pass in GTM before optimizing against it.

## Redirect speed
The `lead` push carries a GTM **`eventCallback`**, which fires once every tag bound to that event
has actually run — measured at **17ms** against the live container.

| Config | Default | What it does |
|---|---|---|
| `redirectMinMs` | `200` | Floor, so the popup reads as a confirmation rather than a flash |
| `redirectMaxWaitMs` | `900` | Ceiling, so a stalled GTM can't strand anyone |

So the redirect lands at **200ms**, and the conversion is confirmed rather than assumed. If the
tag scripts are blocked outright the ceiling drops to 150ms. The CTA points at the
`www.youtube.com` URL: the bare host 301s to it, and that hop is a whole round trip on a phone.

## Performance
**Nothing third-party touches the critical path.** The `<head>` only sets up the queues —
`dataLayer.push()` and `fbq()` are safe from the first millisecond — and the scripts load
**150ms after first paint** (`tagDelayMs`). The queues replay in order, so `lp_view` still
reports. Three guards: a click before the timer force-loads the tags, a background tab falls back
at 2.5s, and a blocked request stops the redirect waiting on tags that will never fire.

Fonts are self-hosted and preloaded (the two serif faces, since the headline paints first),
CSS/JS are inline, and `_headers` / `vercel.json` cache fonts for a year. The page also
preconnects to `www.youtube.com` so the redirect's handshake is already warm.
