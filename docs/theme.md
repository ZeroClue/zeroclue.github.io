---
title: zeroclue.dev — theme reference
---

# Theme Reference

## Design concept

**Warm editorial dark.** A writing space that reads as deliberate, not default. The literary serif in titles creates contrast with the technical sans in body copy — suits a software architect who actually writes.

Layout stays minimal: single column, 680px, no decoration. The character comes from typography and color temperature, not from added elements.

## Typography

| Role | Font | Weight | Notes |
|---|---|---|---|
| Site name, post titles, h2 | Cormorant Garamond | 500/600 italic | Elegant, literary, unexpected for a developer |
| Body, nav, footer, labels | IBM Plex Sans | 400/500 | Clean, technical, not overused |
| Code | IBM Plex Mono | 400 | Matches the Plex family |

All three loaded from Google Fonts. The serif/sans contrast is the defining typographic choice — do not swap to a single-family system.

## Colors

| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#0c0b09` | Page background — warm near-black, not cool dark |
| `--surface` | `#161410` | Code blocks, inline code backgrounds |
| `--text` | `#e5e0d5` | Body text — warm off-white |
| `--text-muted` | `#78705f` | Dates, nav links, footer, secondary labels |
| `--border` | `#252118` | Dividers, code block borders, blockquote rule |
| `--accent` | `#c88b5a` | Links — amber/terracotta, warm and readable |

The warmth of `--bg` is intentional: `#0c0b09` has a brown-black tint vs the cool blue-black of Kern (`#050506`) or CoolMinds (`#08080a`). All three sites are dark but each reads differently.

## Animation

Subtle page entrance only — `.container` fades up 6px over 350ms on load. CSS-only, no JS. No other animations.

## Compared to other sites

| | zeroclue.dev | kern.web.za | coolminds.co.za |
|---|---|---|---|
| Background | warm `#0c0b09` | cool `#050506` | cool `#08080a` |
| Accent | amber `#c88b5a` | lime `#32CD32` | gold `#f0c05a` |
| Display font | Cormorant Garamond italic | Satoshi | Syne |
| Body font | IBM Plex Sans | Inter | DM Mono |
| Personality | editorial/writer | product/agency | engineering consultancy |

## Files

- `src/styles/global.css` — all base styles and CSS custom properties
- `src/layouts/Base.astro` — HTML shell, Google Fonts, nav, footer
- `src/layouts/Post.astro` — post-specific typography (title, meta, body prose)
