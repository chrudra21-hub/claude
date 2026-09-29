# Lama Fera Healing — Prasiddhe Holistic

A single-file landing page for the Lama Fera Healing service, built to be dropped into a
Wix HTML embed. No build step, no dependencies: `index.html` is the whole thing.

## Using it

Paste the contents of `index.html` into the Wix **Embed HTML → Code** element (or host the
file and point an iframe at it). Everything is inline apart from Google Fonts.

## Integration contracts

The page talks to its parent Wix page via `postMessage`, and those message shapes are kept
exactly as the earlier version used them:

| Message | Direction | Meaning |
|---|---|---|
| `{type:'PH_THEME', dark}` | both ways | theme sync; the parent wins when it sends one |
| `{type:'OPEN_LINK', url}` | to parent | nav link clicked (relative URL), with a `window.top` fallback after 250 ms |
| `{type:'SB_GO_TO_BOOKING', url}` | to parent | any Book button, with a `window.top` fallback after 400 ms |

Booking points at `https://www.prasiddhiholistics.com/service-form` with
`?service=lama_fera_healing&duration=healing45`. Both keys must exist in the `SERVICES`
object of the service-form embed. The legacy globals `phSetTheme(dark)` and `phNavGo(url)`
still exist for embeds that call them directly.

## Photo slots

The stat cards and the benefit cards each have a curved media panel built for
a photograph. None are set, so every panel falls back to its own gradient and
ring texture — nothing renders empty.

- **Stat cards** (`index.html`, the `.stat-item` blocks): add
  `style="--img:url('https://your-image.jpg')"` to the card.
- **Benefit cards**: fill the `img` field in the `benefits` array near the
  bottom of the file — `{img:'https://your-image.jpg', ic:ic.flame, …}`.

Landscape crops work best; the panel covers roughly the right 45% of a
benefit card and the whole of a stat card, both centred.

## Before publishing

- **Pricing** — `₹6,500` appears in the booking card and the mobile sticky bar. Update both.
- **Testimonials** — the four quotes under *Voices From Sessions* are clearly-marked
  placeholders. Replace them with real, consented client reflections, or delete the section
  (`#labelVoices` and `#voices`) rather than publish invented ones.
- **Practitioner experience** — the `10+ yrs` stat should match reality.

## How it behaves

- **Two themes.** Parchment/gold by default, teal "Energy Mode" for the dark side. The
  choice is remembered in `localStorage`; with nothing stored it follows the visitor's
  system setting. The parent page can still override it at any time.
- **Motion budget.** `prefers-reduced-motion`, low core count, low device memory or
  Save-Data all switch the page into `lite` mode: no particle canvas, no drifting orbs, no
  twinkles. Reduced motion never leaves content stuck invisible.
- **Particles.** Canvas is viewport-sized (not document-sized), capped at 2× DPR, paused
  whenever the tab is hidden or the light theme is active.
- **Reveal on scroll** via IntersectionObserver, with a scroll-tick sweep as a safety net so
  a fast fling can never leave a section at `opacity: 0`.
- **Accessibility.** Skip link, real `<a href>` nav, `aria-pressed` on the theme buttons,
  a keyboard-operable carousel, an `aria-expanded` accordion and visible focus rings.

## Sections

Header · hero · stat strip · benefits · session flow · who it's for + what's included ·
voices · FAQ · booking card · disclaimer. Plus a scroll-progress bar, a back-to-top button
and a mobile sticky booking bar that hides itself once the booking card is on screen.
