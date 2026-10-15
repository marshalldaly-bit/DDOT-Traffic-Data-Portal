# DDOT Traffic Data Portal (prototype)

A single-page web prototype that helps DDOT staff **compare the volume counts that exist for the same roadway segment** and judge their quality and divergence. Pick a roadway block on the map, then see every observed count type for it side by side, by travel mode, on a common time scale.

> **Prototype status.** The app runs entirely in the browser and all volumes are **synthetic test data**

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
