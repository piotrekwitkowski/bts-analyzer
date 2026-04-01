# Radar 5G

Interactive BTS (Base Transceiver Station) analyzer with terrain-aware speed estimation. Built for evaluating 5G/LTE coverage and selecting optimal outdoor CPE placement.

**[Live Demo →](https://piotrekwitkowski.github.io/bts-analyzer/)**

## Features

- **86 cell towers** from 4 Polish operators (Orange, Play, Plus, T-Mobile) within 10 km radius
- **Interactive Leaflet map** with pie-chart markers showing operator mix per tower
- **Terrain profile analysis** using GUGiK NMT (1m resolution) with SRTM fallback
- **Line-of-Sight (LOS)** calculation with Fresnel zone clearance and knife-edge diffraction loss
- **Two speed columns**: theoretical (at tower) vs estimated (accounting for distance + terrain)
- **Lazy LOS loading** — terrain data fetched on scroll via IntersectionObserver, cached in sessionStorage
- **Filters**: operator checkboxes, technology (n78 / 5G / LTE), distance slider
- **Dark/light theme** with map tile switching (CARTO dark/light)
- **Single HTML file** — no build step, no npm, CDN-only dependencies

## Data Sources

| Source | What | Resolution |
|---|---|---|
| [UKE Wykaz pozwoleń](https://bip.uke.gov.pl/) | Station locations, bands, operators | Per-station |
| [UKE Rejestr urządzeń](https://bip.uke.gov.pl/pozwolenia-radiowe/rejestr-urzadzen/) | Device registry (fills gaps for Plus/Polkomtel) | Per-device |
| [GUGiK NMT](https://services.gugik.gov.pl/nmt/) | Terrain elevation (Digital Terrain Model) | 1 m |
| [OpenTopoData SRTM](https://www.opentopodata.org/) | Elevation fallback | 30 m |
| [CARTO](https://carto.com/) | Map tiles (dark + light) | — |

## Tech Stack

- **Leaflet.js** — map
- **Chart.js** — terrain profile charts
- **Bootstrap 5** — layout, dark/light theme
- **Vanilla JS** — no framework

## How Speed Estimation Works

### Theoretical (at tower)
Based on available bands only — no distance or terrain factors:
- n78 (3600 MHz): 700 Mbps
- n28 (700 MHz) + LTE aggregation: up to 300 Mbps
- 5G DSS: 200 Mbps
- LTE: 40 Mbps × band count

### Estimated (real-world)
Factors in:
1. **Distance** — signal attenuation over distance
2. **Line of Sight** — terrain profile from GUGiK NMT
3. **Obstruction penalty** — proportional to obstacle height above LOS line
4. **Frequency sensitivity** — n78 (3.5 GHz) severely degraded by NLOS, n28 (700 MHz) more resilient

## Project Structure

```
index.html          — Single-file app (HTML + CSS + JS)
data.json           — Tower data (86 towers, 192 operator-stations)
report.html         — Detailed 5G analysis report (Polish)
TP-Link_5G_*.pdf    — Router comparison PDF
teren-*.png         — Terrain profile screenshots
```

## Origin

Built as part of a 5G router selection analysis for a location near Kielce, Poland (Góry Świętokrzyskie). The terrain analysis revealed that the closest n78 tower (Cedzyna, 2.8 km) is blocked by a 317m ridge, while a farther LTE tower (Radlin, 1.7 km) has clear line of sight — making it the better choice despite lower theoretical speeds.
