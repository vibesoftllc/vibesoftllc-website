# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Marketing/landing site for Vibesoft LLC, an app development company. Static HTML deployed via GitHub Pages at `vibesoftllc.com`. No build step, no dependencies, no JavaScript.

## Structure

- `index.html` — home/landing page (hero, services, products, contact)
- `nag.html` — Nag Reminders product page: hero + App Store badge + QR + features.
  Carries the Smart App Banner tag (`<meta name="apple-itunes-app" content="app-id=6792002234">`)
  and OG/Twitter card tags. Store links carry the ASC campaign token
  (`?itscg=30200&itsct=…`) so ASC Analytics → Sources attributes web traffic.
- `assets/` — self-hosted images only, per the privacy policy's "no third-party
  servers" promise: `app-store-badge.svg` (official Apple badge, downloaded not
  hotlinked), `nag-qr.svg` (QR to the listing; rendered on a white plate in CSS
  so it scans on the dark theme), `nag-icon.png`, `nag-card.png` (OG image)
- `about.html` — company info + support section (`#support`); used as the Apple App Store **Support URL**
- `privacy.html` — privacy policy; used as the Apple / Google Play **privacy policy URL**
- `terms.html` — terms of service
- `affiliate-disclosure.html` — FTC affiliate disclosure, linked from every page footer
- `CNAME` — GitHub Pages custom domain

## The one thing to know: there IS a shared stylesheet now

**Superseded 2026-11-21.** Through 2026-11-21, every page carried its own complete copy of the design system in its own `<style>` block — no stylesheet, no partials, no includes — and that duplication caused real bugs (see the "complete redesign" section below: a positional-logic script corrupted CruiseSort's card and dropped a store-badge `src` attribute because the same markup pattern had to be hand-replicated 15 times).

As of that redesign, `assets/style.css` is the single shared stylesheet. Every real page (`index.html`, `about.html`, `nag.html`, `more.html`, `content-owners.html`, `terms.html`, `affiliate-disclosure.html`, and all `privacy-*.html` pages) links it with `<link rel="stylesheet" href="assets/style.css" />` in `<head>`, placed *before* that page's own `<style>` block so page-specific rules win the cascade by source order alone (same selector, same specificity — later wins).

`assets/style.css` holds: the light/dark tokens, base reset, body/mesh/grid background (now with a slow `meshDrift` ambient animation), the sticky blurred nav + nav-link underline, the theme toggle, the `.service-card`/`.app-card` visual system (tint wash, colored top edge, hover lift reserved for whatever variant IS a link), `.reveal`/scroll-motion, the footer (including `.footer-more`), the plain legal-page `.page-header`/`.prose` system, `prefers-reduced-motion` support, and the shared `@keyframes`.

**What stays page-specific, in each page's own (now much smaller) `<style>` block**, and why:
- `index.html`: `.hero`/`.hero-eyebrow`, `.about`, `.contact*` — this page's own hero copy sizing and the two content blocks nothing else on the site has.
- `about.html`: `.page-header`/`.page-eyebrow` (richer than the plain legal version — has an eyebrow and em-emphasis), `.company-block`, `.support-block`, `.contact-methods`, `.support-note`, `.contact*`.
- `nag.html`: `.app-hero` and its own `.app-icon` (128px) and `.store-badge` (inline-block, 52px, next to a QR code) — both **intentionally override** the shared `.app-icon`/`.store-badge` defaults built for the product grid, and reset every box-model property the override needs rather than relying on partial cascade. `.download-row`, `.qr-*`, `.app-note`. Also overrides `--tint` to this page's coral.
- `more.html`: `.page-header` copy, `.app-grid`/`.app-card` **layout only** (`display:flex; gap:1rem; align-items:flex-start; padding:1.4rem` — a horizontal icon-beside-text row, genuinely different from `.service-card`'s column layout) — the tint/border/hover-color part of `.app-card` lives in the shared file.
- Legal/privacy pages: only what's genuinely unique to that one page — `affiliate-disclosure.html`'s `.disclosure-callout`, `privacy-huntadvisor.html`/`privacy-manila.html`'s `.prose code`/`.prose pre` for their technical appendices. Nothing else; the plain `.page-header`/`.prose` system is shared.

**When adding a new page**, copy an existing page's `<head>` (link + minimal page-specific `<style>`) rather than re-embedding the whole design system — that's the entire point of this change. **When editing shared chrome** (nav, footer, tokens, card hover, motion), edit `assets/style.css` once; it reaches every page on next load, no propagation script required.

## Products

The "What we've built" grid on `index.html` uses `.badge` to mark status: `.badge-flagship` for CruiseSort, `.badge-soon` for anything still in App Store review. When an app is approved, swap its badge and wrap the `<h3>` in a link to its store page.

Links to `cruisesort.com` are deliberate SEO backlinks — keep them as plain followed links (no `rel="nofollow"`, no `target="_blank"`).

Nag went live 2026-08-05: its card links to `nag.html` (which carries the store link), badge `.badge-flagship` with the text "On the App Store". New product pages should copy `nag.html`'s pattern: badge + QR download row, campaign-tokened store links, Smart App Banner, self-hosted assets.

Every page's footer carries a **More from Vibesoft** list: every live app as a
link (a per-app privacy page omits its own app), then apps not yet released as
plain text — `<li class="soon">Name &mdash; tagline (coming soon)</li>`, styled
by `.footer-more .soon`, never a link, since an unreleased store page reads "not
available". When an app goes live, turn its `soon` item into a link on every
page and swap its home-page badge, in the same commit.

## Theming

The site is **light by default** and supports dark mode three ways, in priority order:

1. `:root` holds the light palette — the no-JS, no-preference default.
2. `@media (prefers-color-scheme: dark) { :root:not([data-theme]) { … } }` — follows the OS when the visitor hasn't chosen.
3. `:root[data-theme="dark"]` — set by the nav toggle, persisted in `localStorage` under `vs-theme`.

The dark token list is **written out twice** (once in the media query, once in the attribute selector) because CSS cannot share one declaration list between a media query and a selector. Change one, change both — but as of the 2026-11-21 redesign below, only once, in `assets/style.css`, not per page.

An inline `<script>` at the end of each `<head>` resolves the theme before first paint (avoiding a flash), sets `data-js="ready"`, and wires the toggle via delegated click on `document`. The toggle button is `display: none` until `data-js="ready"` appears, since without JS the theme can't be changed and a dead button would be worse than none. This is the site's only JavaScript.

**Every colour must go through a token** — the two palettes only stay in sync if nothing is hardcoded. Notably `--grid` is separate from `--border` (the 60px overlay needs to be far fainter than card borders) and `--body-weight` is a token because 300 reads fine on dark but washes out on white, so light uses 400.

| Variable | Light | Dark | Purpose |
|---|---|---|---|
| `--bg` | `#ffffff` | `#0a0e17` | Page background |
| `--surface` | `#f8fafc` | `#111827` | Card/section backgrounds |
| `--accent` | `#2563eb` | `#3b82f6` | Blue highlight |
| `--accent-glow` | `rgba(37,99,235,.10)` | `rgba(59,130,246,.18)` | Eyebrow pill background |
| `--accent-border` | `rgba(37,99,235,.28)` | `rgba(59,130,246,.3)` | Link underlines, pill borders |
| `--accent-hover` | `rgba(37,99,235,.45)` | `rgba(59,130,246,.4)` | Card hover border |
| `--accent-soft` | `rgba(37,99,235,.22)` | `rgba(59,130,246,.25)` | Legal-page accent |
| `--accent-2` | `#ea580c` | `#fb923c` | Warm secondary accent — added 2026-11-21 for the hero underline sweep and the mesh's third gradient layer; not yet used anywhere contrast-critical, so it hasn't been WCAG-checked against text |
| `--accent-2-glow` / `--accent-2-border` | `rgba(234,88,12,.10)` / `.28` | `rgba(251,146,60,.18)` / `.3` | Same shape as the blue accent's glow/border pair, for `--accent-2` |
| `--cta-shadow` | soft drop | blue glow | Hero CTA — glow only works on dark |
| `--cta-text` | `#ffffff` | `#ffffff` | Text on accent fill |
| `--text` | `#0f172a` | `#f1f5f9` | Primary text |
| `--muted` | `#475569` | `#94a3b8` | Secondary text |
| `--faint` / `--faint-strong` | `#64748b` / `#475569` | `rgba(148,163,184,.5)` / `.65` | Footer affiliate microcopy |
| `--border` | `rgba(15,23,42,.10)` | `rgba(255,255,255,.07)` | Card/section borders |
| `--grid` / `--grid-opacity` | `rgba(15,23,42,.04)` / `.7` | `rgba(255,255,255,.07)` / `.5` | Background grid overlay |
| `--mesh-1` / `--mesh-2` | blue / indigo, ~.07 | blue / indigo, ~.10 | Fixed gradient mesh, first two layers |
| `--mesh-3` | orange, `.05` | orange, `.08` | Fixed gradient mesh, third layer — added 2026-11-21 alongside `--accent-2` for a richer, less single-note background |
| `--nav-bg` | `rgba(255,255,255,.72)` | `rgba(10,14,23,.68)` | Translucent fill behind the sticky nav's backdrop-filter blur — added in the 2026-11-21 redesign |
| `--body-weight` | `400` | `300` | Body font weight |

Light-mode contrast is verified against WCAG AA: text 17.9:1, muted 7.6:1, accent 5.2:1, faint 4.8:1 on white. If you darken the background or lighten any of these, recheck — `--faint` has the least headroom. `--accent-2` is not part of that check — keep it off body text and confine it to decorative uses (glows, underlines, badge accents) until it's verified.

### 2026-11-21 visual refresh

A second accent (`--accent-2`, warm orange) and a third mesh layer were added everywhere per the token table above — propagated by exact string match across all pages, since the blocks were confirmed byte-identical first (the approach `sed -i ''` comment above describes).

`index.html` got the full page-specific treatment first, then the same pattern was extended to **`more.html`, `nag.html`, and `about.html`** the same day:

- Emoji icons are inline SVGs (`.service-icon svg` for the three "What We Do" cards on `index.html`/`about.html`, and for `nag.html`'s four "How it works" cards) — emoji as icons was the dated-looking part of the old design, more than the color palette was.
- Every card (`.service-card` on the three pages above, `.app-card` on `more.html`) carries a `--tint` custom property (an R,G,B triple, inline `style="--tint:…"`) sampled from that app's actual icon artwork via PIL, used for a colored hover glow instead of a fixed `--accent-hover`. Cards with no per-app identity (the "What We Do"/"How it works" cards) fall back to the blue accent's RGB, set as the card class's own default.
- `.reveal` + a `data-js="ready"`-gated IntersectionObserver (a `<script>` at the end of `<body>`, same snippet on all four pages) gives cards a staggered scroll-in instead of the old load-time-only `fadeUp`. Follows the same no-JS-safe pattern as the theme toggle: nothing is hidden unless `data-js="ready"` is already set, so a visitor with JS disabled sees everything, just without the motion.
- Each page's hero italic word gets an animated gradient-sweep underline (`h1 em::after`, `--accent` to `--accent-2`) on load — `.hero h1 em` on `index.html`, `.app-hero h1 em` on `nag.html`, `.page-header h1 em` on `about.html` (three different selectors, same rule, since each page names its hero section differently).

Also on `more.html` specifically: Dogged and Ship O'Clock were moved from the "Coming soon" grid to "Available now" with real store links — both had gone live since that page was last touched, the same staleness `index.html`'s content fix caught a day earlier.

**Not yet extended**: the three legal pages (`privacy.html`, `terms.html`, `affiliate-disclosure.html`) and `content-owners.html` deliberately weren't touched beyond the shared token change — legal text fading in on scroll is bad UX for content that should read as immediately stable, and none of those pages have cards or emoji icons to begin with. If a future pass wants *any* motion on those pages, keep it off the `.prose` content itself.

### Fixed the same day: cards that looked clickable but weren't

The lift + shadow hover (above) was originally applied to every `.service-card`/`.app-card`, including ones with no link at all — the three "What We Do" cards on `index.html`/`about.html`, and all four "How it works" cards on `nag.html`. The operator caught this: hovering anywhere on those made the card look like a button, but only the text links inside it actually went anywhere. The fix, in two parts:

- On `index.html`, every product card that genuinely has one real destination (an app's store page, or a pre-release app's privacy notice) is now the `<a>` itself — `<a class="service-card reveal" href="…">` wrapping the whole card — with the redundant inner links (the `<h3>` link, the inline "View on the App Store"/"Learn more" link, and the store badge's own `<a>` wrapper) removed rather than left as dead weight alongside a now-duplicate outer link. The store badge image lost its wrapping anchor and just carries the `store-badge` class directly; `.store-badge`'s CSS moved from styling an `<a>` + nested `img` to styling the `img` alone.
- The hover rule split in two: the base `.service-card`/`.app-card:hover` (or `.app-card.soon:hover`, which `more.html` already had) gives a plain `<div>` only a faint border tint, while `a.service-card:hover` (or plain `.app-card:hover`, since `more.html`'s live cards are already always `<a>`) gets the lift and glow. `nag.html` and `about.html` have no linked cards at all, so they keep only the bare, un-split rule — a subtle tint, nothing more.

The rule going forward: a card only gets the lift-and-glow hover if it is (or is wrapped by) the single link it represents. A descriptive card with no destination gets, at most, a faint border change — never anything that reads as a button.

`more.html`'s "Coming soon" cards got the same whole-card treatment the same day: Clipjar, Hunt Advisor, and Anchorlight are now each a single `<a>` to their privacy notice (all three pages already existed), rather than an unlinked `<div>` whose `cursor: default` and no-op hover already hinted they weren't clickable — if there's somewhere real to send the click, send it there instead of just suppressing the affordance.

### Also 2026-11-21: three site-wide additions, plus a missed Florida fix

Propagated to all 23 real pages by the same verified-identical-block method as the token changes above:

- **A favicon** — none existed before. It's an inline data-URI SVG (`<link rel="icon" href="data:image/svg+xml,...">`), a blue rounded square with a white "V," so there's still no external request even for this — consistent with the self-hosted-assets rule.
- **`prefers-reduced-motion: reduce` support** — a blanket rule collapsing every animation and transition to near-zero duration for anyone whose OS asks for it. There's real motion to respect that preference for now (the scroll-reveal, the hero underline sweep, every hover transition) where there wasn't before this week's passes.
- **An underline sweep on `.nav-link:hover`** — matches the hero's own underline treatment in miniature, for a small piece of cross-page cohesion.

Also caught on this pass: the "(FLA)" prefix fix from earlier in the week was only ever applied to `index.html`'s footer. `about.html` (both its meta-item and its vanity line), `privacy.html`, `terms.html`, and `affiliate-disclosure.html` were all still showing the full "(FLA) FLA-SHIP" — quietly contradicting the jurisdiction-agnostic rule on every page that matters most for it (the legal pages). Fixed everywhere now; see the Contact section below for the corrected reference.

Use this entry as the reference for the pattern rather than re-deriving it, on `content-owners.html` or anywhere else that later gets its own icons/cards.

### The same day, later: the complete redesign and the move to a shared stylesheet

The operator asked for a complete redesign, not another polish pass, and said to push it to main without a further round of confirmation first. This did two things at once:

1. **Replaced per-page duplicated CSS with `assets/style.css`.** See "The one thing to know" above — this is the architectural half of the change, and it directly addresses the root cause of the CruiseSort/store-badge bug from earlier the same day: that bug happened because the same markup pattern had to be hand-replicated across every page's own copy of the CSS, and a positional-logic script got one page's structural difference wrong. A shared stylesheet doesn't remove the need for careful markup edits, but it removes the need to *repeat* a CSS change 23 times, which is where repetition-driven mistakes come from.
2. **A bolder visual language**, all living in the shared file now: cards show their color at rest (a 3px tinted top edge plus a faint tinted background wash, using each app's already-sampled `--tint`), not only on hover; the nav is `position: sticky` with a `backdrop-filter: blur(14px) saturate(160%)` frosted-glass treatment over `--nav-bg`; the hero/page-header emphasis word is a gradient-fill (`background-clip: text`) rather than a flat accent color, still paired with the animated underline sweep; the background mesh drifts slowly (`meshDrift`, 26s loop) instead of sitting static; `.section-label` gets a small two-color gradient tick before the uppercase text.

**Migration method, since this is the template for any future redesign pass**: for each page, confirm (via `md5` of the exact substring between fixed anchors — the opening `<style>` tag through a known marker, not eyeballing) which CSS is byte-identical to every other page versus genuinely unique to that one, move only the identical part into the shared file, and verify structural balance (`<div>`/`<a>`/`<style>` open-close counts, grepped per file) after every edit before moving to the next page. Two real mismatches surfaced doing this and are worth remembering:
- `.service-card` and `.app-card` (`more.html`) look like the same component but are **not** — `.app-card` is a horizontal icon-beside-text row at `padding: 1.4rem`, `.service-card` is a vertical column at `padding: 1.8rem`. A first pass combined them into one shared selector and would have broken `more.html`'s layout had it shipped; the fix was to share only the tint/border/hover-color *language* between them and leave layout (`display`/`flex-direction`/`gap`/`padding`) page-specific.
- `nag.html`'s hero `.app-icon` (128px) and `.store-badge` (inline-block, 52px, next to a QR code) reuse class names the shared file also defines for the product-grid context (56px icon; unwrapped 40px badge image). The page-specific override resets every box-model property the shared rule set (`height`/`width`/`margin-top`/`padding-top`/`align-self`) rather than assuming the two non-overlapping declarations would just add up cleanly — partial overrides of a shared rule are exactly how a shared stylesheet reintroduces the bugs it's meant to prevent.

A scripted *first* attempt at this same migration had a bug of its own: a replace call matched a block starting with the `<style>` opening tag itself, so removing that block deleted the opener and left a dangling `</style>` with no matching tag and no `<link>` in its place, across 15 files. Caught immediately via direct inspection (not by trusting the script's own success report) before committing anything, reverted cleanly with `git checkout --` since nothing had been committed yet, and re-done with index-based slicing against confirmed-exact anchors instead of hand-typed replacement strings — the same "verify before trusting a bulk script" discipline as the CruiseSort incident, applied one level earlier this time.

Fonts are **system stacks, not webfonts** — Google Fonts was deliberately removed (commit `61d4fe1`) so pages have zero external requests. Headings use `Georgia, 'Book Antiqua', Palatino, serif`; body uses `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif` at `font-weight: 300`. Don't reintroduce a font CDN.

Shared visual scaffolding: `body::before` is a radial-gradient mesh, `body::after` is a 60px grid overlay, both `position: fixed` at `z-index: 0` — so `.container`, `nav`, and `footer` all need `position: relative; z-index: 1` (or higher) to sit above them. Entrance animations use the `fadeUp` keyframe with staggered `animation-delay`.

## Legal pages

`privacy.html`, `terms.html`, and `affiliate-disclosure.html` share a layout: `.page-header` with an `<h1>` and a `<p class="meta">Effective &lt;date&gt; · Vibesoft LLC</p>` line, then a `.prose` block of `<section>` elements each containing an `<h2>` (sections are separated by a top border; `:first-child` has it removed).

Two content conventions are deliberate and should be preserved:

- **No jurisdiction is named.** Terms say "the laws of the state in which Vibesoft LLC is organized" rather than naming a state; the site was intentionally scrubbed of Florida references (commit `1bab5e7`). Keep new legal copy state-agnostic.
- When changing legal copy, bump the `Effective` date in the `.meta` line — both privacy and terms tell readers that's how updates are signaled.

## Development

Open any `.html` file directly in a browser. No server, build tool, or install.

## Deployment

Pushes to `main` deploy automatically via GitHub Pages. No CI config.

## Contact details

Two addresses, both landing on the `@vibesoftllc.com` catchall:

- `admin@vibesoftllc.com` — business, press, partnership. Used by the nav "Get in touch" button and the legal-page contact blocks.
- `dev@vibesoftllc.com` — app support, bug reports, feature requests. Used by the `#support` block on `about.html`.
- Phone `(352) 352-7447`, shown alongside the vanity spelling `FLA-SHIP` (the `(FLA)` prefix was dropped site-wide 2026-11-21 — it was quietly contradicting the jurisdiction-agnostic rule above on every page that showed it). Always link it as `tel:+13523527447` — the vanity string is display-only.

`about.html#support` is the **Apple App Store Support URL**. App Review Guideline 1.5 requires that page to give users a working way to make contact, so the email and phone in `.contact-methods` must stay visible and correct there. Don't reduce it to a form or a bare address.
