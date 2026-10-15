# DDOT Traffic Data Portal (prototype)

A single-page web prototype that helps DDOT staff **compare the volume counts that exist for the same roadway segment** and judge their quality and divergence. Pick a roadway block on the map, then see every observed count type for it side by side, by travel mode, on a common time scale.

> **Prototype status.** The app runs entirely in the browser, with no backend. Most volumes are **synthetic test data**; see [What is real and what is synthetic](#what-is-real-and-what-is-synthetic) below.

## Scope: observed counts only

The portal shows **observed (measured) counts only**. It does **not** include DDOT's modeled estimates, such as AADT. Future iterations may add modeled data alongside the observed counts. The Count types menu lists AADT, a modeled estimate, as "Coming later" (disabled).

## Versions

Each version lives on its own branch so earlier versions stay intact.

| Branch | What it is |
|---|---|
| `main` | The current version of the prototype |
| `version_2` | Replaces modeled AADT with the observed count stations (PCS, WIM, FTMS). |
| `version_3` | Built for comparing count methods on the same segment (see below). |

## Running it

There is nothing to build. Open `index.html` in a browser, or serve the folder to avoid any `file://` limits:

```bash
python3 -m http.server 8765
# then open http://localhost:8765/index.html
```

The page loads Leaflet and SheetJS from cdnjs, map tiles from Esri, and address search from Nominatim (OpenStreetMap), so it needs internet access.

## How to use it

1. **Select a roadway block** by clicking it, or draw a box or circle to select several (then pick one from the list in the details pane).
2. In the **details pane**, choose a **direction** and a **View by** setting (Year, Month, Day, Hour). For Day and Hour, pick the month or day from the calendar. Days with a limited-duration count are marked with a dot.
3. Read the three charts (**Micromobility**, **Pedestrian**, **Vehicle**). Each bar is the **total** counted in that period for one count type. Hover a bar for its value, count type, and date.
4. Use the **Count types** menu and the **Timeframe** slider in the header to filter what the map and charts include.
5. **Export segment data** downloads a CSV of monthly totals. Drawing an area also offers **Download all** (an Excel workbook).

### Map layers

Use the layers button under the zoom buttons to switch the map layer:

- **All counts** (default): each road block is colored by how many count types have data for it (1 to 3 steps of DDOT blue, light to dark).
- **Micromobility, Pedestrian, and Vehicle counts**: the same blue steps, counting only the count types that cover that travel mode. Two or more steps means the same thing is measured by different methods, so those blocks can be compared.

Each legend lists one swatch per step, plus a dashed gray line for "No counts" when the layer has any blocks with none (on All counts that happens only after a Count types or Timeframe choice leaves some blocks with no data).

Every layer follows the Count types menu and the Timeframe slider, and the hover label lists the count types the block has (for the layer's mode, on the mode layers).

## Count types

| Count type | Counts | Collected | History | Direction |
|---|---|---|---|---|
| Automated safety cameras (ASC) | vehicles | continuously, individual vehicles | Dec 2024 | one direction per camera |
| Automated bike and pedestrian counters | bikes, pedestrians | continuously, hourly | 2014 | both combined |
| Permanent count stations (PCS) | vehicles | continuously, hourly | 2018 | both combined |
| Weigh-in-motion stations (WIM) | trucks | continuously, hourly | 2018 | both combined |
| Freeway traffic management sensors (FTMS) | vehicles | continuously, hourly | 2018 | both combined |
| Dockless micromobility | bikes and scooters | continuously, individual vehicles | 2019 | split by direction |
| Manual bike and scooter counts | bikes and scooters | one day per location every 3 years, spring or early summer, 6 to 10 am and 3 to 7 pm | 2004 | split by direction |
| Mid-block multimodal counts | vehicles, pedestrians, bikes | a Before and an After count around a corridor redesign | 2019 | cars by direction |

**Continuous** counts are drawn as solid bars. **Limited-duration** counts (manual and mid-block) cover only a few hours on one day; they are drawn as hatched bars and shown as the total counted in that period, not scaled up to a day.

## What is real and what is synthetic

| Data | Source | Real or synthetic |
|---|---|---|
| Road blocks, names, cross streets, direction | DCGIS Roadway Block layer (Transportation_WebMercator, layer 163) | Real (a sample of about 3,900 blocks, plus blocks added so every station, camera, and mid-block corridor has one; blocks with no count of any type are left off the map) |
| Camera locations | DCGIS Public Safety layer 43 (327 cameras) | Real locations; counts synthetic |
| PCS and WIM stations | DCGIS Transportation Sensors layer 92 | Real locations; counts synthetic |
| FTMS sensors | DDOT Traffic Monitoring System Documentation 2023 | Approximate locations; counts synthetic |
| Dockless counts, Jul to Sep 2026 | `dc-routes-data-for-all-vehicles-in-2026-Q3` (trips per street piece, rounded to 100, under 100 suppressed) | **Real**. Earlier months (back to 2019) and the split by direction are synthetic |
| Mid-block before/after counts | DDOT's Before/After table (16 corridors) | **Real** values; count dates and hours are assumptions |
| Everything else | Generated by the app | Synthetic |

### How the synthetic data works

- Each block has one shared underlying volume per **travel mode** (micromobility, pedestrian, vehicle). Each count type reports the modes it covers from that, with its own method bias and noise, so the same mode is comparable across methods.
- Micromobility is anchored to the real dockless counts. Where a block has a mid-block corridor or a station, the vehicle and pedestrian volumes are anchored to those values.
- About 18% of blocks that have two or more sources get one deliberately divergent source (a constant offset or a recent step change), so quality problems are visible in the demo.
- The data is deterministic (seeded), so it looks the same on every load.
- The charts store one monthly average per day per source. Daily and hourly detail (from 2020) is generated on demand from those monthly values and always adds back up to them. Year and month bars show totals.

## Design

The look follows the [Esri Calcite Design System](https://github.com/Esri/calcite-design-system) (light theme: gray and white surfaces, square corners, 12 and 14 px type, Calcite shadows and tooltip style), with **DDOT blue** (`#293e6b`) as the brand color. The theme is implemented as CSS variables plus one block of Calcite component styles in `index.html`; the Calcite web components themselves are not used.

## Code layout

Everything is in `index.html` (about 2.8 MB, mostly inline data). In order:

1. Styles (the app's own styles, then the Calcite layer near the end).
2. The welcome dialog and page markup.
3. Inline data: road blocks, stations, cameras, dockless Q3, mid-block corridors.
4. `CATEGORIES`: the one place each count type is defined (unit, colors, history, collection method).
5. The synthetic data engine (`buildDataset`) and the on-demand drill-down data (`DRILL`).
6. Map layer, details pane (`renderComparePanel`), time scale (`viewBars`, `buildSlots`), exports, filters, address search, and the welcome dialog.

To add a count type, add an entry to `CATEGORIES` and generate its series in `buildDataset`. The map, key, filter menu, tooltips, and exports pick it up from there.

## Known limitations

- Only one series per count type per block is shown. About 40 cameras and 3 FTMS sensors share a block with another of the same kind and are not shown separately.
- Day and Hour views start in January 2020, so mid-block counts taken in 2019 appear in the Year and Month views only.
- "Now" is fixed at September 2026 (the end of the quarter the real dockless data covers).
- The current year is labeled **YTD** in the Year view; the first year of a source that starts mid-year (for example cameras in 2024) shows a note in its tooltip.
- Tooltips appear on hover. Bars and charts are not yet keyboard or screen-reader accessible.
- The documentation link in the details pane is a placeholder (the DDOT wiki home page).

## Contact

Questions or feedback: Marshall Daly, marshall.daly@dc.gov
