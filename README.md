# Whale–Vessel Collision Risk Analysis — California Coast (2020–2025)

## Overview

This project analyzes the spatial and temporal risk of whale–vessel collisions along the California coast. By combining modeled whale presence with AIS vessel traffic, it identifies where and when whales and vessels overlap most, detects statistically significant risk hotspots, and examines how those hotspots shift across seasons and years.

The final outputs include a composite risk raster, hotspot cluster maps, and an interactive time-slider map for 2020–2025.

---

## Research Questions

- Where and when is the probability of whale–vessel co-occurrence highest along the California coast?
- How does vessel traffic volume and speed vary across high whale-presence areas by season and year?
- Have risk patterns shifted over 2020–2025?

---

## Objectives

- Identify high-risk areas for whale–vessel interactions
- Quantify overlap between whale presence and vessel traffic
- Detect statistically significant risk hotspots (LISA)
- Examine seasonal and year-over-year changes in collision risk

---

## Data Sources

- **NOAA WhaleWatch** — Monthly modeled whale presence probability and density, 0.25° grid, 2020–2025.
- **Global Fishing Watch AIS** — Vessel presence hours per grid cell (0.1°), filtered by vessel type and speed band, 2020–2025.
- **TIGER/Line 2024** — U.S. Census state boundary shapefile used to delineate California.

---

## Study Area

California coastal waters out to a 200 km buffer from the state boundary. The buffer is computed in California Albers (EPSG:3310) for accurate distance and then reprojected to WGS84 (EPSG:4326) for joining.

---

## Methodology

### 1. Data Preparation
- Clean datasets: drop missing/invalid values, standardize column names.
- Convert WhaleWatch longitudes from 0–360 to –180/180.
- Reproject the California polygon from NAD83 (EPSG:4269) to California Albers (EPSG:3310) and back to WGS84 for joining.

### 2. Spatial Framework (Grid)
- Build a 0.25° fishnet aligned to the WhaleWatch lattice.
- Clip the fishnet to the 200 km coastal buffer.
- Every downstream metric is computed on this cell–month unit.

### 3. Spatial Aggregation & Join
- Aggregate WhaleWatch → one whale presence probability per (cell, year, month).
- Snap each 0.1° GFW cell to the nearest 0.25° grid point and sum vessel hours across flags by vessel type and speed band.
- Join the aggregated vessel table to the cleaned WhaleWatch table on `(cell_lat, cell_lon, year, month)`.

### 4. Vessel Exposure (Speed-Weighted)

A literature-based step function applies higher weights to fast vessels, reflecting the lethal-strike threshold near 10 knots (Conn & Silber 2013; van der Hoop et al. 2015):

| Speed band (kn) | Weight |
|-----------------|--------|
| < 2             | 0.2    |
| 2–4             | 0.2    |
| 4–6             | 0.2    |
| 6–10            | 0.2    |
| 10–15           | 0.6    |
| 15–25           | 1.0    |
| > 25            | 1.2    |

`exposure = Σ (hours_in_band × weight_for_band)`

### 5. Risk Calculation

`risk = whale_presence_probability × exposure`

A log-dampened variant is also computed to control heavy-tailed exposure:

`risk_log = whale_presence_probability × log1p(exposure)`

### 6. Spatial Analysis (LISA)
- Queen contiguity weights via `libpysal`.
- Global Moran's I and Local Moran (999 permutations) via `esda`.
- Cluster labels at p ≤ 0.05: HH, LL, HL, LH, ns.
- Run monthly for 2020–2025.

### 7. Temporal Analysis
- Monthly, seasonal, and year-over-year risk summaries.
- Hotspot-trajectory classification per cell: persistent, emerging, fading, intermittent, never.
- Detect shifts in high-risk locations across years.

---

## Outputs

- **Multi-band GeoTIFF** — monthly composite risk surface, 2020–2025.
- **Whale presence map**, **vessel exposure map**, **composite risk map**.
- **LISA cluster maps** — statistically significant HH/LL/HL/LH hotspots.
- **Temporal summaries** — monthly, seasonal, and yearly risk.
- **Interactive Folium map** — `TimeSliderChoropleth` of monthly risk over 2020–2025.

---


## How to Run

1. Create a Python environment with the dependencies listed below.
2. Open `whale_vessel_risk_analysis.ipynb`.
3. Run the notebook top to bottom; it follows a pipeline structure (load → filter → transform → compute → save) with each section producing an artifact in `processed/`.

### Dependencies

`pandas`, `numpy`, `geopandas`, `shapely`, `pyproj`, `rasterio`, `rioxarray`, `xarray`, `libpysal`, `esda`, `matplotlib`, `folium`.

---

## References

- Conn, P. B., & Silber, G. K. (2013). Vessel speed restrictions reduce risk of collision-related mortality for North Atlantic right whales. *Ecosphere*, 4(4).
- van der Hoop, J. M., Vanderlaan, A. S. M., & Taggart, C. T. (2015). Absolute probability estimates of lethal vessel strikes to North Atlantic right whales in Roseway Basin, Scotian Shelf. *Ecological Applications*, 25(6).
- NOAA WhaleWatch — https://oceanview.pfeg.noaa.gov/whalewatch/
- Global Fishing Watch — https://globalfishingwatch.org/
- U.S. Census TIGER/Line Shapefiles — https://www.census.gov/geographies/mapping-files.html

---

## Team

Charlotte · JinHyuk Son · Jeongmin An · Yeobi Hobson
Consultant: Prof. Michael Mann
Geo-programming II Final Project — The George Washington University
