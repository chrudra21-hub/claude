# Prasiddhe Holistic — Wix embeds

Single-file pages built to drop into Wix HTML embeds. No build step, no dependencies.

| File | What it is |
|---|---|
| `index.html` | Lama Fera Healing service page |
| `catalog.html` | Diwali sale product catalogue |

Both share one design system, one theme contract and one nav, so they read as the same site.

---

## Putting these in Wix

**1 · Use the right element.** Editor → **Add → Embed Code → Embed HTML**, then *Enter
code* and paste the whole file. Do **not** use Settings → Custom Code: that injects into
the site `<head>` and will not render a full HTML document.

**2 · Set the element height.** A Wix embed is a fixed-size iframe. If its height is
smaller than the page, the page scrolls inside a little box — which is what "not working"
usually looks like. Approximate heights at the moment:

| | desktop | mobile |
|---|---|---|
| `index.html` | ~5300px | ~9100px |
| `catalog.html` | ~2800px | ~3100px |

Set them separately for desktop and mobile in the editor.

**3 · Or size it automatically (Velo).** Both pages post their real height on load and on
every resize, so a few lines size the element for you:

```js
$w.onReady(() => {
  $w('#html1').onMessage(event => {
    if (event.data && event.data.type === 'PH_HEIGHT') {
      $w('#html1').height = event.data.height;
    }
  });
});
```

**4 · Links out of the embed.** Wix runs the embed in a sandboxed cross-origin iframe,
where `window.top.location` from JavaScript is blocked — that is why buttons can look
dead. Every outward control is therefore a real `<a target="_top">`, which browsers allow
on a user click. If your embed blocks top navigation too, change one line near the top of
the script:

```js
var LINK_TARGET = '_top';   // '_blank' opens links in a new tab instead
```

Also note `localStorage` throws inside that iframe. It is wrapped, so the pages work
either way — the visitor's theme choice just isn't remembered between visits.

### Messages the parent page can listen for

| Message | From | Meaning |
|---|---|---|
| `PH_HEIGHT` | both | the page's current height in px |
| `PH_THEME` | both ways | light/night theme; the parent wins when it sends one |
| `OPEN_LINK` | both | a nav link was used (relative URL) |
| `SB_GO_TO_BOOKING` | `index.html` | a Book button was used |
| `PH_PRODUCT_ENQUIRY` | `catalog.html` | an Enquire button was used, with `id` and `name` |

---

# Lama Fera Healing (`index.html`)

A single-file landing page for the Lama Fera Healing service.

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


---

# Diwali Sale Catalogue (`catalog.html`)

A filterable product catalogue for the festival collection: ten products, each with a
photo, a one-line summary and a detail view.

## Editing the catalogue

Everything you change day to day sits in the `PRODUCTS` array near the bottom of the file.

```js
{ id:'kuber-kit', name:'Kuber Kit', cat:'Kuber Kits', sale:true, price:null, img:PHOTO,
  blurb:'A five-piece Kuber set, packed together for the festival.',
  detail:'…the longer text shown in the detail view…',
  kit:['Mini pyrite frame','Kuber Kunji','Kuber Kumkum','Kuber Potli','Kuber Stone'] }
```

- **`img`** — every item currently points at the same `PHOTO` constant at the top of the
  array. Give an item its own URL to replace just that one. A dead URL falls through to
  the card's gradient rather than showing a broken image.
- **`price`** — a number renders as `₹1,234`; `null` renders "Price on request". Items
  without a price sort to the end of a price sort, so a half-priced catalogue still works.
- **`cat`** — must match one of the strings in `CATS`; the filter bar and its per-category
  counts build themselves from that list.
- **`kit`** — optional. The card shows the first three with a "+n more", the detail view
  shows all of them.
- **`sale`** — `true` puts the red Diwali flag on the card.

## Where an enquiry goes

```js
var ENQUIRY = { page:'https://www.prasiddhiholistics.com/contact', whatsapp:'' };
```

Put a number with country code in `whatsapp` (e.g. `'919812345678'`) and every Enquire
button switches to WhatsApp with the product name pre-filled. Left empty, buttons go to
the contact page with `?product=<id>`. Either way the page also posts
`{type:'PH_PRODUCT_ENQUIRY', id, name, url}` to the parent Wix page first, so you can
intercept it there instead.

## What it does

Two cards per row on phones (one scrolling filter strip, short category pills, kit chips
deferred to the detail view) · live search across name, category and kit contents · category filters with counts ·
sort by name or price · a detail dialog with keyboard arrows, Escape to close and focus
returned to the card you opened · empty state with a reset · the same light/energy theme,
remembered and shared with the rest of the site.
