---
title: zeroclue.dev — theme reference
---

# Theme Reference

## Design principle

Dark, editorial, personal. The site should feel like a thinking space, not a product. No decorative elements, no gradients, no animations. The writing is the design.

## Colors

| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#0e0e11` | Page background |
| `--text` | `#dddad3` | Body text, headings |
| `--text-muted` | `#78756f` | Dates, secondary labels, footer |
| `--border` | `#242428` | Dividers, code block borders, blockquote rule |
| `--accent` | `#7a9cb8` | Links |

The accent is a desaturated steel blue — calm and readable, distinct from Kern (lime green) and CoolMinds (gold).

## Typography

- **Font:** Inter (Google Fonts, weights 400/500/600) + system-ui fallback
- **Base size:** 1.0625rem (17px)
- **Line height:** 1.75 — optimised for long-form reading
- **Heading tracking:** `-0.02em` to `-0.03em` — tighter than default

## Layout

- **Max-width:** 680px, centered
- **Padding:** `0 1.5rem` on the container (mobile-safe)

## Files

- `src/styles/global.css` — all base styles and CSS variables
- `src/layouts/Base.astro` — HTML shell, Inter font import, nav, footer
- `src/layouts/Post.astro` — post-specific typography, blockquote, back link

## Compared to other sites

| | zeroclue.dev | kern.web.za | coolminds.co.za |
|---|---|---|---|
| Background | `#0e0e11` | `#050506` | `#08080a` |
| Accent | steel blue `#7a9cb8` | lime `#32CD32` | gold `#f0c05a` |
| Fonts | Inter | Satoshi + Inter | Syne + DM Mono |
| Personality | editorial | product/agency | engineering |
