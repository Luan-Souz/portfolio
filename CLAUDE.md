# Luan Ferreira de Souza — Portfolio

Single-file HTML/CSS/JS portfolio. No build step, no dependencies (only Google Fonts CDN).

## Quick Start

```bash
npm run dev
```

Runs `npx serve .` → opens on `http://localhost:3000` (or next available port).

## Project Structure

- **`index.html`** — entire app (embedded styles + scripts)
- **`package.json`** — minimal config, `dev` script only

## Design System

### CSS Custom Properties

```
--blue: #3943b7                   (primary)
--blue-dim: rgba(57,67,183,0.09) (subtle overlay)
--amber: #e08e45                  (accent)
--taupe: #706563                  (muted)
--taupe-light: rgba(...,0.18)     (borders)
--taupe-xlight: rgba(...,0.08)    (grids)
--black: #000000
--white: #ffffff
--off: #f9f8f6                    (alt section bg)
--serif: 'Playfair Display'        (headings)
--sans: 'Josefin Sans'             (body)
--data: 'Libre Baskerville'        (data)
```

### Typography

- Headings: 56px, Playfair Display, 600 weight
- Body: 18px, Josefin Sans, 400 weight
- Data/labels: Libre Baskerville Italic, 10–13px
- Line spacing: 1.1–1.85 depending on context

## Page Sections

### 1. Hero
- Full-viewport height, centered flex
- Manifesto: 3 lines with mixed styling (black + blue italic)
- Stats grid: 2×2 mini-cards with metrics
- Mini chart: skill distribution (black bg, bar chart)
- CTAs + social links

### 2. Work / Projects
**Dynamic from data:** `featuredProjects` array indexes into `projectData`
- Cards injected via JS loop
- Each card: `.reveal` class (scroll animation)
- **Critical:** After injection, must register with IntersectionObserver: `obs.observe(el)`

### 3. Education
Three collapsible modals (Northeastern, UEA, Other)

**NEU Modal:**
- Tab 1: Projects (3 cards with `data-project-index` linking to `projectData`)
- Tab 2: Finance Coursework (15 items, last 3 expanded into sub-topics)
- Tab 3: Activities

**UEA Modal:**
- Tab 1: Algorithmic Trading Research project (linked to `projectData[3]`)
- Tab 2: Coursework (2-column grid: Math & Quant Methods vs Control Systems & ML)
- Tab 3: Activities

**Key:** `openEduProject(el, modalId)` closes modal + calls `openProject(idx)` to show project detail.

**Font sizing:**
- School name: 26px
- Degree: 18px (card) / 16px (modal)
- Department/period: 14px
- Course item: 15px
- Section title: 13px
- Open hint: 13px

### 4. Professional Background — Interactive Milestone Map

**Concept:** 10 experiences on a left-to-right timeline with 3 branching periods (parallel roles).

**Data:**
```js
mmGroups = [
  {top: 0},           // Red Bull (single)
  {top: 1},           // Walt Disney
  {top: 2},           // Amazonas TA
  {top: 3, bottom: 4}, // Dock Intern + R&D Researcher (concurrent)
  {top: 6, bottom: 5}, // Dock Analyst I + UG Researcher
  {top: 7},           // 360 HF Analyst
  {top: 8, bottom: 9}, // 360 HF Sector Manager + NEU Supplier (concurrent)
];
```

**Layout (206px total height):**
- 7 columns (each 1/7 ≈ 14.29% wide)
- Top road: full-width, always visible
- Bottom roads: partial-width segments during branches, offset by FW=60px
- Fork/merge connectors: smooth Y-curve SVG at branch boundaries

**Visual Elements:**
- Pins: 22×22px teardrops, blue border (inactive) or blue fill (active)
- Roads: 12px rounded bars with white dashed center line
- Fills: width-animated, 0.55s cubic-bezier(0.4,0,0.2,1)

**SVG Connectors (Bezier Y-curves):**
- Fork: top road center → bottom road center (smooth curve downward)
- Merge: bottom road center → top road center (smooth curve upward)
- 60px wide, 3 overlaid paths: bg (taupe), dashes (white), fill (blue, opacity animated)

**Navigation:**
- 10 items total (milestones 0–9)
- NavSeq ordering: left-to-right, top then bottom per column
- Prev/Next buttons
- Detail card shows: year, phase, role, description, tags

**Road Fill Logic:**
- **Top road:** fills to `(gi+0.5)/NG*100%` (column center)
- **Bottom road:** pixel-accurate, accounts for 60px offset
  - Formula: `fillPct = (pinX - roadLeft) / (roadRight - roadLeft) * 100`
  - Calculated using `stage.offsetWidth` to handle viewport size

### 5. Projects (Unified Data)
```js
projectData = [
  {
    type: '...',      // category/subtitle
    title: '...',     // headline
    metrics: [...],   // {l: label, v: value}
    tags: [...],      // skill tags
    desc: '...',      // 1–2 sentence summary
    overview: '...',  // full context
    motivation: '...', // problem statement
    methodology: '...', // approach/methods
    results: '...'    // outcomes/metrics
  },
  // 12 total
];
```

**Linking:**
- Work section: dynamic cards from `featuredProjects` indices
- Education modals: project cards with `data-project-index` attribute

## JavaScript Patterns

### Scroll Animations
- IntersectionObserver (`obs`) watches `.reveal` elements
- On view: opacity 0→1, translateY +20px→0 (0.65s ease)
- Stagger with `.d1`, `.d2`, `.d3`, `.d4` (0.1s delay increments)
- **Work cards issue fixed:** After dynamic injection, re-query + observe all `.reveal` elements

### Modal System
- `openEduModal(schoolId, modalId)` → show modal
- `closeEduModal(modalId, tabId, skipDetail)` → hide modal
- Tab switching: click `.edu-tab` to show/hide `.edu-tab-content[data-tab="..."]`
- Project linking: `.edu-project-item[data-project-index]` → `openEduProject(el, modalId)`

### Milestone Road Map (IIFE)
Local scope binds: NG, W, FW, TL_H, TR_Y, etc.
```js
const mmGroups = [...];  // groups array
const milestones = [...]; // 10 milestone objects

(function(){
  // mmGroups length, column width, connector zone width
  var NG = mmGroups.length; // 7
  var W = 100/NG;           // 14.286%
  var FW = 60;              // 60px fork/merge connector width
  
  // Road layout constants
  var TR_Y = 76, TR_H = 12, TR_CY = 82;  // top road
  var BR_Y = 118, BR_H = 12, BR_CY = 124; // bottom road
  var SVG_H = 54, SVG_TR = 6, SVG_BR = 48; // SVG dimensions
  
  // Build bottom road segments, fork/merge connectors, pins, labels
  // mmSel(msIdx) → select milestone, fill roads, show detail, update nav
})();
```

**Bottom Road Fill (Pixel-Accurate):**
```js
var stageW = stage.offsetWidth;
var pinX = (gi + 0.5) * W / 100 * stageW;
var roadL = seg.l / 100 * stageW + FW;
var roadR = seg.term ? stageW : seg.r / 100 * stageW - FW;
var fillPct = (pinX - roadL) / (roadR - roadL) * 100;
```

## Key Rules & Gotchas

1. **Work cards injection:** Must re-register `.reveal` elements with IntersectionObserver after DOM injection
2. **Education linking:** Project cards need `data-project-index` attribute; click triggers `openEduProject()`
3. **Fork width:** FW=60px affects both SVG size and bottom road offset; change carefully
4. **SVG paths:** Bezier formula uses `M 0,y1 C cp,y1 cp,y2 FW,y2` where cp=FW*0.5
5. **Stage width:** Fill calculations use `stage.offsetWidth` (in pixels, not percentages)
6. **Modal tabs:** Data attribute must match: `data-tab="..." ` on content, `.edu-tab[data-tab="..."]` on button

## Testing Checklist

- [ ] All 10 milestones clickable; detail card updates
- [ ] Road fills reach selected pin (top + bottom proportional)
- [ ] Fork/merge connectors turn blue on branch selection
- [ ] Education modal tabs switch smoothly
- [ ] Project links from education → detail view work
- [ ] Scroll animations trigger on all `.reveal` elements
- [ ] Localhost serves without errors
- [ ] Fonts load (Playfair, Josefin, Libre Baskerville)

## Recent Changes (June 2026)

**Professional Background Milestone Map:**
- Replaced diagonal SVG lines with smooth **bezier Y-curves** (FW=60px wider zone)
- Fixed bottom road fill calculation to be **pixel-accurate** (accounts for 60px offset)
- Fork/merge connectors now fade in/out via CSS opacity transition (not stroke color swap)

**Work Section:**
- Refactored to render from `projectData` array using `featuredProjects` indices
- Dynamically injected cards must be re-observed by IntersectionObserver

**Education Linking:**
- Education project cards link to `projectData` via `data-project-index` attribute
- NEU: indices 0, 1, 2 linked; UEA: index 3 linked

---

**Owner:** Luan Ferreira de Souza  
**Contact:** ferreiradesouza.l@northeastern.edu  
**GitHub:** github.com/Luan-Souz  
**Last Updated:** June 2026
