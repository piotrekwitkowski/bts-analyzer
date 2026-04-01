# Building Radar 5G: From Research to Product

This document traces how Radar 5G evolved from a conversational research question into a polished interactive tool — and the design principles that emerged along the way.

## The Journey

### Phase 1: Research (conversation, no code)

It started with a simple question: *"Is this a good device?"* — a spec sheet for a TP-Link ER701-5G outdoor router.

What followed was deep research:
- 5G Release 16 standard analysis
- ER701 vs NE200 router comparison
- Polish LTE/5G band allocation mapping
- BTS station identification near Kielce
- Terrain profile analysis revealing that the "best" tower (n78 in Cedzyna) is blocked by a 317m ridge

**Key insight**: The closest 5G n78 tower was useless due to terrain. A farther LTE tower with clear line-of-sight was actually the better choice. This couldn't be seen from specs alone — you need terrain data.

### Phase 2: Static Report (`report.html`, PDF)

The research materialized into a structured HTML report with:
- Router comparison tables
- Band compatibility matrices
- BTS station analysis per location
- Terrain profiles with ASCII LOS diagrams
- Final recommendation

This was useful but static — a snapshot frozen in time.

### Phase 3: Interactive MVP (`index.html` + `data.json`)

The leap from report to tool. First version had:
- Leaflet map with colored markers
- Sortable data table
- Operator/technology filters
- Distance slider
- 73 stations from UKE "Wykaz pozwoleń" data

**What was missing**: Plus/Polkomtel had only 2 stations (vs 25 for Play). The UKE permit files were incomplete for one operator.

### Phase 4: Data Completeness

Discovered that UKE has two data sources:
1. *Wykaz pozwoleń* — station permits (our original source)
2. *Rejestr urządzeń* — device registry (fills gaps for Plus)

Downloaded all 4 operator registries (~400MB), parsed them, and rebuilt the data model:
- **Flat list → Tower-centric model**: stations at the same physical location grouped into towers
- **86 towers, 192 operator-stations** across 4 operators
- **Pie-chart markers** showing operator mix per tower

### Phase 5: Terrain Intelligence

Added real terrain analysis using Polish GUGiK NMT API (1m resolution):
- Line-of-sight calculation
- Fresnel zone clearance
- Knife-edge diffraction loss estimation
- **Two speed columns**: theoretical (at tower) vs estimated (distance + terrain)

### Phase 6: Iterative UI Refinement (30+ commits)

This is where most of the time went — and where the interesting design principles emerged.

---

## Design Principles (Discovered Through Iteration)

These weren't planned upfront. They emerged from the conversation — from trying things, seeing what felt wrong, and fixing it.

### 1. Show the answer, not the data

**Before**: Table showed raw band lists (`5G3600, 5G700, LTE1800, LTE2100, LTE2600...`).
**After**: Table shows estimated speed (`260 Mbps ✅ Play n28`). Bands are hidden behind a click.

The user doesn't care about `LTE2600`. They care about "how fast will this be for me?"

### 2. Two numbers are better than one

Speed estimation went through three versions:
1. **v1**: Single speed (theoretical, no terrain) — misleading
2. **v2**: Single speed (with terrain) — updates dynamically, confusing when values change
3. **v3**: Two columns — "Na nadajniku" (theoretical) + "Szacowana" (real-world with ✅/⛔)

Showing both lets the user understand *why* — "this tower has 700 Mbps capacity but I'll only get 75 because of the hill."

### 3. Color must mean the same thing everywhere

Early versions had green `★ n78` in the 5G column but yellow `150 Mbps` in the speed column for the same tower at 6km. Green said "great" while yellow said "meh" — contradictory.

Resolution: 5G column shows *what's available* (fixed colors). Speed column shows *what you'll get* (dynamic colors). Different questions, different color scales. Trying to unify them made both worse.

### 4. Every pixel earns its space

Progressive removal of elements that didn't earn their keep:
- ~~Loading spinner~~ → data loads in milliseconds from local JSON
- ~~Separate filter bar~~ → merged into header
- ~~Separate "Profil" column~~ → click row to expand
- ~~Map popups~~ → click marker scrolls to table row
- ~~"Szczegóły" button column~~ → click anywhere on row
- ~~GPS coordinates in header~~ → redundant with map
- ~~"wieże w promieniu 10km"~~ → redundant with distance slider

Each removal made the interface feel more direct.

### 5. Don't load what you won't see

LOS calculation went through three strategies:
1. **v1**: Load all 86 profiles at startup (1-2 min wait)
2. **v2**: Auto-fetch in background with 1s delay, closest first
3. **v3**: `IntersectionObserver` + `sessionStorage` cache

v3 loads terrain data only when the row scrolls into view (with 200px margin). Second visit is instant from cache. Zero wasted API calls.

### 6. Consistency > correctness in naming

"Odległość" (distance) is technically correct Polish. But the header filter said "Max odległość", the table column said "Odległość", and both competed for space. Changed both to "Dystans" — shorter, consistent, still clear.

Similarly: "DSS" (Dynamic Spectrum Sharing) is technically correct for 5G-on-shared-spectrum. But nobody outside telecom knows what DSS means. Changed to just "5G" — it's what the user will see on their phone.

### 7. The theme must be set before the first paint

Dark→light flash on page load was caused by: HTML defaults to dark → JS reads localStorage → switches to light. Fix: inline `<script>` in `<head>` before any `<body>` rendering. Six bytes of prevention vs 200ms of flicker.

### 8. Separate facts from estimates

The table now has clear separation:
- **Left columns** (Dystans, Azymut, Operatorzy, 5G) — facts from UKE data
- **"Na nadajniku"** — theoretical maximum (fact about the tower)
- **"Szacowana"** — our calculation (estimate, may be wrong)
- **✅/⛔ icons** — LOS status (calculated from terrain data)

The user can trust the left side absolutely. The right side is "our best guess."

---

## What Would Come Next

If this were to become a real product:

1. **User location input** — enter any address, not just hardcoded coordinates
2. **Dynamic UKE data** — fetch from API instead of static JSON
3. **Signal measurement overlay** — crowdsourced actual speed data
4. **Router recommendation engine** — given your location and budget, which router + operator
5. **Mobile-first table** — card layout instead of horizontal scroll
6. **Antenna aiming tool** — AR overlay showing which direction to point your CPE

## Commit History

32 commits across ~8 hours of iterative development:

```
dc3da3d  BTS analyzer: 5G report + interactive station map
5411f3d  Add terrain profile with LOS analysis (GUGiK NMT 1m + SRTM fallback)
9070915  Tower-centric data model with full UKE device registry
b1f3e8f  Per-operator speed estimates with top-2 in main row
9c28e38  UI polish: inline terrain profiles, per-operator speeds with LOS
ac3f5c3  Split speed into theoretical/estimated columns
8670d51  Round azimuth to whole degrees
0ecded9  Muted units: km on distance, cardinal direction on azimuth
c1b035c  Light theme toggle, filters above map
2f0cd76  Light/dark theme, compact filters, responsive mobile
47c700b  Full-width table, equal-height filter groups
9d5fef0  Unified header: title + filters + theme toggle in one line
cf67bb7  Compact header, hide filters on mobile
38fd275  Technology filter: multi-select checkboxes
6cf0e14  100vh layout, compact distance label, alphabetical operators
1d1ae19  Simplified theme toggle
28ed56f  Remove terrain panel, unify to inline details, rename to Radar 5G
76c0120  Remove popups, click row to toggle details
017731d  Non-sortable # column
ac20d50  100vh layout: map 60vh, table fills rest with scroll
492a693  Align header controls to right
a9e2535  Narrower distance slider
e2b0e49  Azimuth: rotated arrow indicator
cab57f5  Lazy LOS: IntersectionObserver + sessionStorage cache
ccaa460  Remove loading spinner
665ffbe  Rename to Dystans
e07322b  Add README
059eacd  Add GitHub link
1a8ad14  Fix theme flash: apply saved theme before first render
0b3488f  Smooth wheel zoom
36a6e6d  Sort bands: n78 → n28 → 5G → LTE → UMTS → GSM
8bfbb2c  Rich inline terrain chart: red/green fill, Fresnel zone, tooltips
```
