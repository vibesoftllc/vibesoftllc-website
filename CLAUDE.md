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

## The one thing to know: there is no shared CSS

**Every page carries its own complete copy of the design system in its own `<style>` block.** There is no stylesheet, no partials, no includes. A change to a token, the nav, or the footer must be applied to **all five HTML files** or the site drifts out of sync.

The same applies to the markup for `nav`, `footer` (footer links + copyright + affiliate blurb), and the `:root` token block — these are duplicated verbatim across pages. When editing one, edit them all. `sed -i ''` across the five filenames is the usual approach for token-level changes.

Page-specific CSS does exist on top of the shared base: `index.html` has `.hero`/`.about`, `about.html` has `.company-block`/`.support-block`, and the three legal pages have `.prose`.

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

The dark token list is **written out twice** (once in the media query, once in the attribute selector) because CSS cannot share one declaration list between a media query and a selector. Change one, change both — in all five files.

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

Use this entry as the reference for the pattern rather than re-deriving it, on `content-owners.html` or anywhere else that later gets its own icons/cards.

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
- Phone `(352) 352-7447`, shown alongside the vanity spelling `(FLA) FLA-SHIP`. Always link it as `tel:+13523527447` — the vanity string is display-only.

`about.html#support` is the **Apple App Store Support URL**. App Review Guideline 1.5 requires that page to give users a working way to make contact, so the email and phone in `.contact-methods` must stay visible and correct there. Don't reduce it to a form or a bare address.
