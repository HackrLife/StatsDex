x# StatsDex — BarcaFutbol Data Stories

**By HackrLife · Sydney, Australia**

---

## What This Is

StatsDex is a scroll-driven interactive data storytelling platform built for football analytics. Each player profile is a self-contained HTML file — no framework, no database dependency, no build step — that loads in a browser and tells a complete analytical story through animated charts, signature numbers and editorial narrative.

The platform lives inside the BarcaFutbol editorial ecosystem, published alongside the BarcaFutbol newsletter on Beehiiv.

---

## File Structure

```
statsdex/
├── index.html          ← Hub page — player card grid + about section
├── yamal.html          ← Lamine Yamal data story
├── saka.html           ← Bukayo Saka data story
├── vitinha.html        ← Vitinha data story
└── README.md           ← This file
```

Each player file is fully standalone. All chart rendering is done via native Canvas API — no Chart.js, no D3, no external dependencies beyond Google Fonts.

---

## Data Architecture

### Source

Each CSV contains:

- One row per season per competition (All, LaLiga, UCL, PL, Copa, etc.)
- 31 columns covering output, creation, possession, defensive and physical metrics
- All figures expressed **per 90 minutes**

### Verified Columns (31 total)

| Column | Description |
|:---|:---|
| `MP` | Matches played |
| `MIN` | Average minutes per game |
| `GLS` | Goals per 90 |
| `AST` | Assists per 90 |
| `GI` | Goal involvements per 90 (GLS + AST) |
| `XG` | Expected goals per 90 |
| `XA` | Expected assists per 90 |
| `XGI` | Expected goal involvements per 90 |
| `KEYP` | Key passes per 90 |
| `BCC` | Big chances created per 90 |
| `SDR` | Successful dribble rate (%) |
| `DRP` | Dribbles attempted per 90 |
| `TOS` | Touches in opposition box per 90 |
| `SOT` | Shots on target per 90 |
| `BCM` | Big chances missed per 90 |
| `APS` | Accurate passes per 90 |
| `APS%` | Pass accuracy (%) |
| `ALB` | Accurate long balls per 90 |
| `LBA%` | Long ball accuracy (%) |
| `ACR` | Accurate crosses per 90 |
| `CA%` | Cross accuracy (%) |
| `TACK` | Tackles per 90 |
| `INT` | Interceptions per 90 |
| `BLS` | Blocks per 90 |
| `CLS` | Clearances per 90 |
| `YC` | Yellow cards per 90 |
| `RC` | Red cards per 90 |
| `ELTG` | Errors leading to goal per 90 |
| `ADW` | Aerial duels won per 90 |
| `ASR` | Attribute score rating (composite) |

### Data Rules

1. **Every number in every chart must trace to a CSV row.** No estimated, interpolated or fabricated values.
2. **Scatter peer groups use only players with CSVs in the dataset.** No peers are added from memory or external sources.
3. **Competition splits use the `All` row** as the primary number for all cross-player comparisons. Competition-specific rows (LaLiga, UCL, PL) are used only for Big Game Index calculations.
4. **Career arc xG is null for early seasons** (Sofascore does not backfill xG for older data). These render as 0 in the chart — not interpolated.
5. **All data is verified before HTML build** via a markdown approval table shown to the editor. HTML is only built after explicit approval.

---

## Proprietary Metrics

### Big Game Index (BGI)
Measures performance elevation in high-stakes competitions vs domestic league.

```
BGI = (UCL metric / PL metric) × 100
```

A BGI of 175 means the player's output in the Champions League is 75% higher than in the league. Currently applied to GI per 90.

### Game Impact Score (GIS)
A composite metric designed to capture contribution that xGI misses — ball progression, chance creation, defensive work and costly errors.

```
GIS = (ChI × 2) + (SprR × 3) + (TchD × 0.1) + (Tkl × 1.5) + (Int × 1.5) − (CstErr × 3)
```

Where:
- `ChI` = Chance Ignition (KEYP)
- `SprR` = Sprint Rate / Dribble Drive (SDR)
- `TchD` = Touch Dominance (APS)
- `Tkl` = Tackles
- `Int` = Interceptions
- `CstErr` = Costly errors (ELTG)

GIS is used as dot size in scatter charts — larger dot = higher composite impact.

---

## Chart Types Per Story

Each player HTML contains six animated canvas charts:

| Chart | Description |
|:---|:---|
| **Career Arc** | Line + fill showing GI per 90 across all seasons, with xG overlay |
| **Season Radar** | 10-metric spider chart normalised to peer group |
| **Dual Radar** | Comparison radar overlaying two players on the same axes |
| **Pizza Chart** | Percentile wedge chart across 10 metrics vs positional peer group |
| **Polar Chart** | Four-family composite: Output / Creativity / Control / Defense |
| **Peer Scatter** | xA vs GLS (or APS vs KEYP for midfielders), dot size = GIS, CSV peers only |

All charts animate on scroll via `IntersectionObserver`. Progress bar tracks scroll position.

---

## Design System

### Palette

| Variable | Hex | Use |
|:---|:---|:---|
| `--bg` | `#0d1117` | Page background |
| `--bg2` | `#0d1b2a` | Chart / section background |
| `--bg3` | `#161b22` | Gate / premium block |
| `--barca` | `#A50044` | Primary accent (Barça crimson) |
| `--gold` | `#EDBB00` | Highlight / hero player |
| `--white` | `#f0f6fc` | Primary text |
| `--gray` | `#8b949e` | Secondary text |
| `--gray-dim` | `#3d444d` | Borders / grid lines |
| `--pass` | `#4CAF50` | Passing / creation metrics |
| `--def` | `#2196F3` | Defensive / control metrics |
| `--atkhi` | `#ffd60a` | Attacking metrics (high) |
| `--atklo` | `#e63946` | Attacking metrics (negative) |

Player colour override: each player file sets `--player` to their personal accent colour (Yamal = `#A50044`, Saka = `#f0f6fc`, Vitinha = `#EDBB00`).

### Typography

| Role | Font |
|:---|:---|
| Display / numbers | Bebas Neue |
| Editorial / body | Playfair Display (italic) |
| Data / labels | DM Mono |

### Layout

Each story follows a 10-act scroll structure:

1. Hero (full viewport — name, position, signature number)
2. Career Arc (chart right)
3. Signature Number (full viewport pull quote)
4. Season Radar (chart right)
5. Comparison Radar (chart left)
6. Pizza Chart (chart right)
7. Polar / Archetype (chart left)
8. Peer Scatter (chart right)
9. Hidden Metric (full viewport — xGI vs proprietary metric)
10. Premium Gate (blurred preview + CTA)

---

## Adding a New Player

### Step 1 — Export CSV 
Export the player's career stats. Ensure the file follows the 31-column schema above. Name it `{player_slug}_stats.csv`.

### Step 2 — Generate approval table
Run the data extraction script against the CSV. Produce a markdown table covering:
- Career arc (all seasons, all columns)
- 25/26 competition splits
- Scatter peer group (CSV-only players, no fabricated values)

### Step 3 — Editorial approval
Table is reviewed and approved before any HTML is written.

### Step 4 — Build HTML
Copy the closest existing player HTML as template. Replace the JS data block and peer array using the approved table. Run the verification script to confirm every value matches the CSV.

### Step 5 — Verification script
Automated check confirms:
- All arc arrays match CSV row by row
- No fabricated peer names present in the PEERS array
- Scatter axis range accommodates all real data points
- Brand strings are consistent

### Step 6 — Add card to index.html
Add player card to the grid with correct headline number, archetype label and player colour.

---

## Current Players

| Player | File | Archetype | Signature Number |
|:---|:---|:---|:---|
| Lamine Yamal | `yamal.html` | Dual-Threat Creator | 1.0 GI/90 |
| Bukayo Saka | `saka.html` | Precision Finisher | BGI 175 |
| Vitinha | `vitinha.html` | Elite Controller | 104.9 APS/90 |

---

## Roadmap

Players queued (CSV required before build):

- Ousmane Dembélé
- Michael Olise
- Khvicha Kvaratskhelia
- Désiré Doué
- Bradley Barcola
- Luis Díaz
- Harry Kane
- Mohamed Salah
- Marcus Rashford
- Leroy Sané

---

## Publishing

Files deploy as static HTML — any static host works (Netlify, GitHub Pages, Cloudflare Pages). No server, no build pipeline, no dependencies to install.

Each file is self-contained. Drop any player HTML into a folder alongside index.html and it works.

---

## Credits

**Editorial & Analytics** — HackrLife Media LLC  
**Data source** — Sofascore  
**Platform** — BarcaFutbol (Beehiiv)  
**Contact** — dev@hackrlife.com

© 2026 HackrLife · Sydney, Australia
