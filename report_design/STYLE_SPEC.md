# TOU reporting visual design system: style spec (v1)

This spec is the basis for **all** reporting output of the TOU / rate-fit demo: PDF one-pagers, PNG slides, and the later
Cloudflare Worker web UI. The primary reference is `ref/style_ref.jpeg` (a light analytics dashboard). The secondary
references are `ref/video_notes.md` (header stat strip, pipeline stage cards, mono MAIN LOG with ASCII bars, dense
bipartite network, and the log-scale growth chart), all translated into the light card style.
The tokens live in `template/tokens.css`. Components live in `template/report.css` and `template/lib/tou-viz.js`.
**Never put a literal colour or number in a page. Take it from a token or from the data JSON.**

## 1. Principles
1. **Minimal ink.** Use an off-white page, white cards and 1 px hairlines. Leave no chart junk: no axis lines except the
   baseline, no boxes around legends, and gridlines at most at `--hair-soft`.
2. **Two voices.** A bold sans (Inter) carries meaning: titles and hero numbers. Uppercase mono (JetBrains Mono) carries
   labels, ticks, logs and provenance.
3. **Colour means something.** Green is good or money saved, crimson is bad, at risk or on-peak, and blue is
   informational. The rate-fit category colours are fixed across every visual.
4. **Data in, visuals out.** Components are pure functions `(container, spec)`. Text is formatted by `TV.fmt`, and the
   page composer contains copy but no numbers.
5. **Provenance on every card.** A mono source line sits bottom-right. Every page carries the build label
   (e.g. `PROTOTYPE, Phase 2 data`) as a black chip, plus the data class (`SYNTHETIC DATA`).

## 2. Colour tokens
| Token | Hex | Use |
|---|---|---|
| `--page` | `#f3f3f1` | page background |
| `--card` | `#ffffff` | card fill |
| `--card-tint` | `#faf9fd` | highlighted (latest) log row |
| `--hair` / `--hair-soft` | `#e7e7e4` / `#f0f0ee` | card borders, dividers / in-chart rules |
| `--ink` | `#141414` | titles, values, primary lines |
| `--ink-2` | `#3a3a3a` | body |
| `--ink-3` | `#6f6f6f` | mono labels |
| `--ink-4` | `#a3a3a0` | ticks, footnotes, timestamps |
| `--ink-5` | `#cfcfcb` | back ridge rows, faint links |
| `--crimson` / soft | `#d6264f` / `#f7d9e0` | bad, on-peak window, callouts, the "hot" highlight |
| `--pink` | `#de5b8c` | card dot, lattice D5 |
| `--orange` | `#e8772e` | card dot, *On TOU, shouldn't be* |
| `--amber` | `#d99a1e` | warnings (WARN pill), lattice D1 |
| `--teal` | `#1f9d84` | *Should add SDTR*, TOU family, C&I links |
| `--green` | `#2e8b57` | good values, PASSED |
| `--blue` | `#2f5bd8` | informational values, *Should be on TOU*, flow chip |
| `--purple` | `#7c4dcc` | *Other wrong rate*, insight-library events |
| `--chip` | `#111111` | black chips (white mono text) |

**Rate-fit categories:** Right rate `#b9bdc3` (neutral), On TOU/shouldn't be = orange, Should be on TOU = blue, Should add
SDTR = teal, Other wrong rate = purple.
**Tariff families (chord):** Flat/demand = blue, TOU = teal, SDTR rider = crimson, HLFT = purple.
**Value tones:** `tone-good` (green), `tone-bad` (crimson), `tone-info` (blue), `tone-warn` (amber), `tone-mute` (ink-4).

## 3. Typography (fonts are bundled locally and OFL-licensed; see `template/fonts/LICENSE-*`)
| Role | Font | Size / weight | Notes |
|---|---|---|---|
| Page title (H1) | Inter | 28 px / 700 | letter-spacing -0.01em |
| Card title | Inter | 17.5 px / 700 | preceded by a 7 px coloured dot |
| Hero value (strip) | Inter | 22 px / 600 | tabular |
| KPI value | JetBrains Mono | 14 px / 500 | right-aligned, tone colour |
| Micro-label / KPI label | JetBrains Mono | 9.5 px / 400, UPPERCASE, +0.07em | `--ink-3` |
| Tick labels | JetBrains Mono | 8.5 px / 400 | `--ink-4` |
| Log / console | JetBrains Mono | 10 px (log) / 9.5 px (console) | timestamps `--ink-4`, codes bold in accent |
| Source line | JetBrains Mono | 8 px | `--ink-4`, bottom-right |
Fallbacks: IBM Plex Sans / IBM Plex Mono, then system. The full (not latin-subset) font files are bundled so that
`→ █ ░ ↔ ≈ ≤` render in the brand fonts.

## 4. Geometry
- Canvas: **1440 px** wide, 26 px page padding, 14 px grid gap, 12-column grid (`span-3 … span-12`).
- Cards: radius 10 px, 1 px `--hair` border, padding 16/18/12 px, fixed heights per row so rows align.
- Chips: radius 3 px, 5×8 px padding. Pills: fully rounded, 8 px mono uppercase.
- Strokes: hairline 1 px; primary series 1.4–1.5 px; ridge rows 0.85 px (the annotated row 1.3 px); network links 0.4–2 px.

## 5. Card anatomy
```
┌───────────────────────────────────────────────────────────────────────────┐
│ ● Card Title        MICRO-LABEL (centre-left)             RIGHT MICRO-LABEL │  header: grid auto | 1fr | auto
│ ┌ KPI list ┐ │ ┌──────────────── chart ─────────────────┐ ┌ KPI list ┐    │  body: flex row, 18 px gap,
│ │LABEL  val│ │ │                                         │ │          │    │  1 px vertical rule between
│ └──────────┘ │ └─────────────────────────────────────────┘ └──────────┘    │
│ MONO TAGLINE (bottom-left)                          source / provenance (r) │  footer
└───────────────────────────────────────────────────────────────────────────┘
```
- Dot colours rotate crimson / orange / blue / pink / teal / purple. The ridge and log cards use a black dot.
- The micro-label says *what the lens is* ("TOU WINDOW LANDSCAPE"). The right micro-label says *scope* ("ONE DAY / 17
  SEGMENTS / JULY"). The tagline is a short imperative ("PRICE THE PEAK. CHECK THE RATE."). The source line cites the
  data.

## 6. KPI stat list anatomy
- An optional mono heading ("SHAPE SCANNER / ALL CUSTOMERS"), then an optional legend (dot + mono label), then rows.
- Row: UPPERCASE mono label on the left (`--ink-3`, 9.5 px) and a value on the right (mono 14 px 500), with 5.5 px vertical
  padding and no dividers (`lined` adds hairline dividers for dense lists).
- Value tone encodes meaning: money saved and good states are green, mis-rated and risk are crimson, and neutral facts
  are ink. A tone is never used as decoration.
- The width is 170–220 px. A 1 px vertical rule separates the list from the chart.

## 7. Chart conventions
| Visual | Convention |
|---|---|
| Time series / curve | 1.5 px ink line with a monotone curve; soft vertical gradient fill (accent at 16% to 1.5%); three y ticks (0, mid, max) at the left edge in mono; three x ticks (start/mid/end); end point = crimson dot with a 14% halo, a vertical hairline to the baseline, and a **boxed callout** (white, 1 px crimson border, crimson mono). Secondary marks use a dashed guide, a hollow dot and a two-line mono label. |
| Ridgeline | Rows are stacked back (light `--ink-5`) to front (`--ink`), each occluding the rows behind it with a white fill. The highlighted window is crimson strokes plus a 4.5% crimson tint, bounded by dashed guides, with a black chip naming the window. Row labels are mono on the left, the row metric on the right, and the annotated row is bold, with a tooltip box. |
| Chord | Directed chord; outer arcs are 3.2 px, coloured by family; ribbons are 0.7 px strokes at 55% with a 7% fill; arcs under 1.2% get a dot but no label; the largest arcs get bold coloured labels. |
| Lattice | 5-cube projected onto a skewed 5-point star; edge colour = dimension, opacity and width = depth; vertex size = √population, vertex colour = ink-5 → ink → crimson by metric; faint orbit ellipses; a dimension legend `D1 … D5` in colour at the bottom. |
| Relationship graph | Deterministic d3-force (seeded). Hubs: category fill, 2.5 px white ring, a grey halo ring, and a crimson outer arc = mis-rated share. Satellites are small dots with thin spokes in category colour. Cross-links are faint grey, with width ∝ weight. The flow path is a dashed 1.6 px ink wave with an arrow and a **blue chip** label; focus hubs get black-bordered label boxes. |
| Event log | Latest event in a lavender "hot" box; then rows `time │ CODE │ text │ •`, with the code in accent bold and a fade-out mask at the bottom. Rows auto-fit the card. |
| Stage cards | A mono file-like name, a status pill (PASSED green, WARN amber, FAILED crimson, PENDING dashed grey), a title, a 2×2 micro-metric grid, and a check sentence. A dashed connector joins the stages. |
| Console | Mono pre block; `[hh:mm:ss]` in ink-4; OK green / WARN amber / MISMATCH crimson; ASCII bars `[████░░░░]` 30 characters wide. |
| Bipartite | Two point scales, cubic Bézier links, width ∝ √weight, opacity 18–73%, coloured by source class. |
| Log growth | Log-log; the dashed ink line is the linear quantity (nodes); the blue line with hollow markers is the super-linear quantity; a boxed blue callout at the end. |
Always: mono tick labels, no chart borders, no 3-D effects, no drop shadows (except a 1 px, 6% shadow on tooltips),
deterministic layouts (seeded randomness) so that re-renders are identical.

## 8. Number formatting (`TV.fmt`)
`int` 470,489 · `compact` 6.00M / 470.5K · `compact1` 94K · `usd` $367.1M / $2.90K / $812 · `usd2` $0.58 · `pct` 7.8% ·
`pct0` 87% · `bytes` 433.0 GB · `hours` 33.0 h · `sci` 1.0E8. Missing values render as `n/a`, never as 0.

## 9. Page types
- **Executive view:** header, headline strip, then capture curve + activity log / ridge / chord + lattice / relationship graph.
- **System view:** header, run stat strip, pipeline stage cards, then bipartite network + graph compounding / main log +
  shard completion with reconciliation table.
- The PDF is one page per view at 1440 px wide (page height = tallest view). PNGs are rendered at 2× device scale.
