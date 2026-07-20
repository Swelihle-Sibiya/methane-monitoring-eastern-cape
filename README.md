# Methane Concentration Monitoring — Eastern Cape, South Africa

A **Google Earth Engine (GEE)** workflow for monitoring atmospheric methane (CH₄) concentrations over the **Eastern Cape Province** using **Sentinel-5P TROPOMI** offline (OFFL) CH₄ data for **September – November 2022**, contextualised with Sentinel-2 surface reflectance (NDVI) and SRTM terrain data.

![Platform](https://img.shields.io/badge/platform-Google%20Earth%20Engine-4285F4?logo=google&logoColor=white)
![Sensor](https://img.shields.io/badge/sensor-Sentinel--5P%20TROPOMI-green)
![Variable](https://img.shields.io/badge/variable-CH%E2%82%84%20column%20mixing%20ratio-orange)
![Language](https://img.shields.io/badge/language-JavaScript%20(GEE%20Code%20Editor)-yellow)

---

## Overview

Methane is the second most important anthropogenic greenhouse gas, with a global warming potential far exceeding CO₂ over a 20-year horizon. This script maps the spatial distribution of column-averaged dry-air CH₄ mixing ratios over the Eastern Cape for spring 2022 (Sep–Nov), flags concentration hotspots, summarises the distribution statistically, and relates the methane field to vegetation condition (NDVI) and topography (SRTM elevation).

### Workflow

1. **Load methane data** — `COPERNICUS/S5P/OFFL/L3_CH4`, band `CH4_column_volume_mixing_ratio_dry_air`, filtered to 1 Sep – 30 Nov 2022 and bounded to the Eastern Cape asset
2. **Compute average concentration** — temporal `mean()` of the CH₄ column mixing ratio, clipped to the province; rendered at 1800–1850 ppb (black → blue → yellow → red)
3. **Identify hotspots** — binary mask of pixels where mean CH₄ > **1835 ppb**, displayed with `selfMask()` in red
4. **Descriptive statistics** — regional mean / min / max / standard deviation via a combined `ee.Reducer` chain in `reduceRegion()` at 1000 m scale
5. **NDVI layer** — Sentinel-2 SR median composite (Sep–Nov 2022, `CLOUDY_PIXEL_PERCENTAGE < 20`), `normalizedDifference(['B8','B4'])`
6. **Elevation layer** — SRTM 30 m DEM clipped to the province (0–2500 m colour ramp)
7. **Time series** — per-image regional mean CH₄ chart over the study period (`ui.Chart.image.series`, 1000 m scale)
8. **Map legend** — custom `ui.Panel` legend for the CH₄ colour ramp (1800–1850 ppb)
9. **Export** — mean CH₄ raster to Google Drive (GeoTIFF, 1000 m, folder `GEE`) and the descriptive statistics as a CSV table export

## Study area

The Eastern Cape provincial boundary is maintained as a GEE **Table asset** (`projects/ee-snothilesbiya19/assets/EC`). The boundary was prepared in **QGIS** — the project file `Eastern_Cape.qgz` in this repo documents that step — then uploaded to Earth Engine as a FeatureCollection and used for filtering, clipping, statistics, and the export region.

## Data

| Dataset | GEE ID | Role | Native resolution |
|---------|--------|------|-------------------|
| Sentinel-5P OFFL CH₄ (TROPOMI) | `COPERNICUS/S5P/OFFL/L3_CH4` | CH₄ column volume mixing ratio (dry air, ppb) | ~1113 m L3 grid (~7 km sensor footprint) |
| Sentinel-2 Surface Reflectance | `COPERNICUS/S2_SR_HARMONIZED` | NDVI vegetation context | 10 m |
| SRTM DEM | `USGS/SRTMGL1_003` | Elevation context | 30 m |
| Eastern Cape boundary | `projects/ee-snothilesbiya19/assets/EC` | Study area / clip mask | Vector |

**Primary band:** `CH4_column_volume_mixing_ratio_dry_air` — column-averaged dry-air mole fraction of methane, in parts per billion (ppb). Regional values over the study area fall in the ~1800–1850 ppb range; the hotspot threshold is set at 1835 ppb.

**Temporal window:** 1 September – 30 November 2022 (austral spring).

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `EasternCape_Methane_2022` | GeoTIFF (1000 m) | Seasonal mean CH₄ column mixing ratio, clipped to the province |
| `EasternCape_CH4_Statistics` | CSV | Regional mean, min, max, and standard deviation of CH₄ |
| Map layers | Interactive | Boundary, mean CH₄, hotspots (>1835 ppb), NDVI, elevation, legend |
| Console chart | Time series | Regional mean CH₄ (ppb) per acquisition, Sep–Nov 2022 |

## Interpretation notes

- **Scale mismatch is intentional:** TROPOMI CH₄ (~km-scale) cannot be meaningfully sharpened to Sentinel-2/SRTM resolution. NDVI and elevation are *context layers* — e.g. wetlands and agricultural areas (biogenic sources) vs terrain channelling — not per-pixel predictors of the CH₄ signal.
- **OFFL vs NRTI:** the offline (OFFL) stream is used deliberately — it applies stricter quality screening than near-real-time and is the appropriate choice for retrospective analysis.
- **Retrieval gaps:** CH₄ retrievals are only produced for cloud-free, quality-assured pixels; coastal and persistently cloudy areas may show data gaps in the seasonal mean. A larger compositing window increases coverage at the cost of temporal specificity.
- **Hotspots ≠ sources:** elevated column values indicate enrichment integrated over the full atmospheric column and are affected by transport, surface albedo, and terrain; attributing a hotspot to a specific emitter requires further analysis (wind data, higher-resolution plume imagery).

## Repository contents

```
methane-monitoring-eastern-cape/
├── ch4_monitoring_ec.js     # GEE Code Editor script (full workflow)
├── Eastern_Cape.qgz         # QGIS project used to prepare the provincial boundary
└── README.md
```

## How to run

1. Sign in to the [GEE Code Editor](https://code.earthengine.google.com/).
2. Upload the Eastern Cape boundary as a Table asset (or request access to the existing asset) and add it as the `table` import.
3. Paste the script and click **Run** (the Sentinel-5P, Sentinel-2, and SRTM collections are referenced by ID in code).
4. Inspect the map layers (boundary, mean CH₄, hotspots, NDVI, elevation, legend) and the Console (statistics, time-series chart).
5. Launch both export tasks (`EasternCape_Methane_2022` GeoTIFF and `EasternCape_CH4_Statistics` CSV) from the **Tasks** tab.

## Requirements

- Google Earth Engine account (Code Editor access)
- QGIS ≥ 3.x (only to reproduce the boundary preparation from `Eastern_Cape.qgz`)
- Google Drive space for exported GeoTIFFs

## Possible improvements

- Monthly/weekly composites and an anomaly map (deviation from a multi-year baseline) instead of a single seasonal mean
- Data-driven hotspot detection (e.g. mean + 2σ from the computed statistics) rather than the fixed 1835 ppb threshold
- Zonal statistics per district municipality for reporting
- Cross-reference hotspots with known source sectors (landfills, livestock density, wastewater treatment, coal/gas infrastructure)
- Incorporate ERA5 wind fields to separate local emission signals from transported plumes
