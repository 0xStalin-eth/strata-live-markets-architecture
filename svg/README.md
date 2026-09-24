# Diagram sources

The eight diagrams from `index.html`, extracted as standalone SVG files.

**These are not referenced by the page.** `index.html` still carries its own inline copies and
renders independently of this directory. These exist so the diagrams can be edited, embedded in
a report, or converted without touching the HTML.

| File | Market |
|---|---|
| `01-ethena.svg` | ethena — USDe / `sUSDeStrategy` / sUSDe |
| `02-neutrl.svg` | neutrl — NUSD / `sNUSDStrategy` / sNUSD |
| `03-saturn.svg` | saturn — USDat / `SaturnStrategy` / sUSDat |
| `04-figure.svg` | figure — USDC / `FigureStrategy` / wYLDS → PRIME |
| `05-mhyper.svg` | mhyper — USDC / `MidasStrategy` / mHYPER |
| `06-mm1usd.svg` | mm1usd — USDC / `MidasStrategy` / mM1-USD |
| `07-nestopal.svg` | nestopal — USDC / `NestOpalStrategy` / nOPAL |
| `08-spkmhyperiso-multistrategy.svg` | spkMhyperIso — **not deployed**, multi-strategy contrast |

## Self-contained

Each file carries its own `<style>` block and its own `<defs>` arrowhead markers, so it renders
correctly on its own — open one directly in a browser, or drop it into Figma, Inkscape,
Illustrator or a LaTeX/Markdown report.

Two differences from the inline versions in `index.html`:

- **Colours are literal hex, not CSS custom properties.** The page defines its palette as
  `var(--jr)` etc. on `:root`; that would resolve to nothing outside the page, so the values are
  baked in here for portability.
- **Light theme only.** The page adapts to the viewer's system theme; these are fixed to the
  light palette, which is what you want for a printed or embedded figure. The dark values are in
  `index.html` if a dark variant is ever needed.

## Palette

| Role | Hex |
|---|---|
| Junior | `#A9571A` on `#F6ECE2` |
| Senior | `#0F6959` on `#E5F1EE` |
| Core (CDO, accounting) | `#3A41A0` on `#E9EAF6` |
| External protocol | `#8A3F63` on `#F6E9EF` |
| Cooldowns | `#1F6390` on `#E4EFF6` |

Fonts are referenced by name (Archivo, IBM Plex Sans, IBM Plex Mono) with system fallbacks. They
are not embedded — install them locally for an exact match, or substitute freely.

## Regenerating

These were extracted from the inline `<svg class="dia">` blocks in `index.html`. If the page
changes, re-extract rather than editing both copies by hand.
