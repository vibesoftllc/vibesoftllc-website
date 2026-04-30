# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the marketing/landing page for Vibesoft LLC, a Florida-based app development company. It is a single-file static site deployed via GitHub Pages at `vibesoftllc.com`.

## Structure

- `index.html` — home/landing page
- `about.html` — company info and support contact (used as the Apple App Store Support URL)
- `privacy.html` — privacy policy (used as the Apple/Google Play privacy policy URL)
- `terms.html` — terms of service
- `affiliate-disclosure.html` — FTC-compliant affiliate disclosure for Vibesoft apps
- `CNAME` — GitHub Pages custom domain (`vibesoftllc.com`)

All pages are self-contained (styles inline, no build step). They share the same nav, footer, and CSS design tokens.

## Development

Open any `.html` file directly in a browser — no server, build tool, or dependency installation required.

## Design System

All CSS lives in the `<style>` block of `index.html`. Key CSS custom properties (defined in `:root`):

| Variable | Purpose |
|---|---|
| `--bg` | Page background (`#0a0e17`) |
| `--surface` | Card/section backgrounds |
| `--accent` | Blue highlight color (`#3b82f6`) |
| `--text` | Primary text |
| `--muted` | Secondary/subdued text |
| `--border` | Subtle borders |

Fonts: **DM Serif Display** (headings) and **DM Sans** (body), loaded from Google Fonts.

## Deployment

Pushes to `main` deploy automatically via GitHub Pages. No CI configuration needed.

## Contact

Business email referenced throughout: `admin@vibesoftllc.com`
