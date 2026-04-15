# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file quote calculator for **Houston Centerless Grinding Service LLC**. Everything (HTML, CSS, JS) lives in `index.html` — no build system, bundler, or framework. Open the file directly in a browser to run it.

## Architecture

### Data Store (`store` object)
All rate tables and lookup data live in the `store` global object at the top of the `<script>` block. These are editable at runtime via the Rate Tables UI. Key tables:

- `shopRates[]` — $/hr rate tiers by raw OD size
- `materials[]` — material grades with `removalRate` (in³/min), `grindFactor` (cost multiplier, currently all 1.00), and `density` (lb/in³)
- `materialConditions[]` — condition multipliers applied to the grind/setup/finish cost base
- `setupSizes[]` — setup time (hrs) by OD range, auto-detected from raw OD; displayed in the Setup Range field as `hrs`
- `tolerances[]`, `surfaceFinish[]`, `straightness[]` — time-based adders in min/ft
- `qtyDiscounts[]` — percentage discounts applied to setup, surface finish, straightness, and tolerance
- `packaging[]` — cost per foot; box types enforce a $75 minimum charge; box count is capped at qty (no more boxes than pieces)
- `boxWeightLimit` — max lbs per box (default 1500 lb); used to calculate number of boxes needed
- `inspection[]` — cost per piece
- `delivery[]` — rush multiplier applied to grinding/processing costs only (grind, setup, surface finish, straightness, tolerance) — not applied to packaging, inspection, or add-ons
- `overstockAllowances[]` — added to raw OD when "oversize stock" is checked

### Quote Calculation Flow (`calculate()`)
1. Parse inputs: raw OD, finish OD, length, quantity, all select values
2. Apply overstock allowance to get effective raw OD (if stock toggle is on)
3. Compute volume removed: `(rawArea - finArea) × length`
4. Compute weights using material density
5. Grind time: `volumeRemoved / removalRate × grindFactor`
6. Grind cost: `grindHours × shopRate`, enforcing minimum
7. Setup cost: `setupTime × shopRate × passes` (1 pass per 0.250" OD reduction), with qty discount
8. Time-based adders (surface finish, straightness, tolerance): `min/ft ÷ 60 × lengthFt × rate × qty`, with qty discount and minimums
9. Box count: `Math.min(Math.ceil(totalWeight / boxWeightLimit), qty)` — capped at qty so boxes never exceed pieces
10. Packaging cost: box types use box count × cost/ft with $75 minimum; non-box types use qty × cost/ft
11. Packaging add-ons: straps ($10/ea), cradles ($1.25/ft × length × qty), export stamp ($30 flat)
12. Material condition multiplier on grind + setup + finish costs
13. Delivery/rush multiplier on processing costs only (grind + setup + surface finish + straightness + tolerance)
14. Price adjustment slider (±15%) on final total
15. Minimum charge floor: $150

### UI Structure
- **Two views** toggled by nav buttons: Quote Builder and Rate Tables
- **Quote Builder**: left panel (inputs) + right panel (results with cost breakdown, metrics, weight summary)
- **Rate Tables**: editable grid of all `store` tables with add/delete row support
- Packaging section includes 3 toggle checkboxes: Straps (with qty input), Cradles, Export Stamp

### Key Functions
- `populateSelects()` — fills all dropdowns from `store` data, sets defaults
- `calculate()` — main calculation and render entry point
- `renderTables()` / `renderTableBody()` — builds the Rate Tables UI
- `updateCell()` / `addRow()` / `deleteRow()` — inline editing of rate tables
- `updateSetupDisplay()` — auto-fills the Setup Range field with the matched range label and setup time in hours
- `resetForm()` — clears inputs and resets all toggles
- `copyQuote()` — copies formatted quote text to clipboard

### Defaults
- Material: 17-4 PH SS (index 6)
- Condition: Annealed (index 4)
- Tolerance: ±.002" (index 2)
- Surface Finish: 32 Ra PSQ (index 1)
- Straightness: 0.0015"/ft PSQ (index 3)
- Delivery: Standard (index 5)
- Packaging: Box (index 4)
- Stock toggle: ON by default

### Styling
- Dark theme with CSS custom properties (`:root` vars)
- Fonts: Bebas Neue (display), IBM Plex Sans (UI), IBM Plex Mono (data/labels) — all via Google Fonts CDN
- No external CSS framework
