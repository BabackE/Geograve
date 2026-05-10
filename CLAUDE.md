# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Geograve** — an interactive D3.js visualization of bird migration and distribution for UW CSE 512 HW3. Single-page app with a biome globe, species search, weekly migration animation, and regional distribution bar chart. Canvas target: 1366×768.

Design reference (Canva): https://canva.link/f0pljxttt29hw8d

## Running the App

No build system. Serve files with a local HTTP server (required for `d3.json()` / `d3.csv()` to load local files):

```sh
# Python
python -m http.server 8000

# Node (if http-server is installed)
npx http-server .
```

Then open `http://localhost:8000` in a browser.

## Architecture

The app is a vanilla D3.js single-page app — no framework, no bundler. All data loading is done at runtime via `d3.json()` / `d3.csv()`.

### Data Loading Strategy

Two-phase load:

1. **On page load** (once):
   - `Data/~featured_species_data.json` — species metadata (search, card, taxonomy, IUCN)
   - `Data/biomes_simplified.geojson` — biome polygons for globe background

2. **On species selection** (per species):
   - `Data/Species/{Common Name}/weekly_abundance_27km.json` — 52-week abundance grid for animation
   - `Data/Species/{Common Name}/regional_stats.csv` — regional distribution for bar chart

### Key Data Files

| File | Purpose |
|---|---|
| `Data/~featured_species_data.json` | Species metadata: taxonomy, IUCN status, trend, media URLs |
| `Data/biomes_simplified.geojson` | Globe background (property: `biome_group`) |
| `Data/Species/{Name}/weekly_abundance_27km.json` | Fields: `speciesCode, commonName, week (1–52), lon, lat, abundance` |
| `Data/Species/{Name}/regional_stats.csv` | Regional distribution; season values: `breeding`, `nonbreeding`, `prebreeding_migration`, `postbreeding_migration` |

Season label mapping (eBird → UI):
- `All Year` = all four seasons combined
- `Breeding` = `breeding`
- `Wintering` = `nonbreeding`
- `Migration` = `prebreeding_migration` + `postbreeding_migration`

### Design Tokens

`Design Info/geograve.tokens.js` — source of truth for all colors, typography, panel dimensions, badge styles. Import or copy constants from here; do not hardcode values.

Key tokens:
- `COLOR_BG_APP = "#162130"` (dark navy background)
- `COLOR_ACCENT = "#7ec8a0"` (mint green — active states, progress bars, bars)
- `FONT_FAMILY = "'Playfair Display', Georgia, serif"`
- Panel layout: `PANEL_HEADER`, `PANEL_INFO`, `PANEL_GLOBE` (x/y/w/h from Figma)
- `getBadgeStyle(hex)` — returns `{ stroke, fill, color, strokeWidth, borderRadius }` for IUCN/trend badges

### Icons

`Design Info/Icons/` — 7 SVG/PNG icons: bird, search, play, speaker, sky, worldwide, no-sound.

### Biome Globe Layer

`Data/Biomes/biome_globe_layer.js` — reference snippet showing how to draw biome polygons. The `path` generator must use the same D3 projection as the globe. Biome colors are also defined in `geograve.tokens.js` under `BIOME_COLOR_MAP`.

### Weekly Abundance Animation

Group loaded JSON by week, then use `d3.group(weeklyData, d => +d.week)`. On each animation tick, call `updateWeek(week)` using `.join()` to update circle positions and radii via an `abundanceScale`. Points use `fill: COLOR_ACCENT` at 65% opacity.

### Taxonomy Display Format

```
COMMON       Barn Swallow
SCIENTIFIC   Hirundo rustica
FAMILY       Swallows • Hirundinidae
ORDER        Passeriformes
```

### Bar Chart Titles

Use the exact format: `DISTRIBUTION BY REGION • ALL YEAR` (the second part updates dynamically as the active season tab changes).
