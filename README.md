# Geo-programming-II_Final-project
# Project: Whale–Vessel Disturbance Risk — California Coast

## Datasets
- NOAA WhaleWatch  
  Monthly modeled whale density + presence probability (0.25° grid, 2020–2025)

- Global Fishing Watch AIS  
  Vessel presence hours per grid cell, filtered by vessel type (cargo, fishing) and speed band (2020–2025)

- California Boundary  
  TIGER/Line shapefile for clipping

---

## Research Questions
- Where and when is the probability of whale–vessel co-occurrence highest along the California coast?
- How does vessel traffic volume and speed vary across high whale presence areas by season and year?
- Have risk patterns shifted over 2020–2025?

---

## Analysis Plan
1. Load and clean both datasets  
   - Convert WhaleWatch longitude from 0–360 → -180–180

2. Join datasets  
   - Spatial-temporal join of GFW vessel hours to WhaleWatch grid cells (lat/lon + month)

3. Compute vessel exposure  
   - Weighted vessel exposure score per cell using speed bands (faster vessels = higher weight)

4. Calculate risk score  
   - Composite risk = whale presence probability × weighted vessel exposure

5. Spatial analysis  
   - Run LISA to identify statistically significant risk clusters

6. Temporal analysis  
   - Identify seasonal peaks  
   - Analyze year-over-year trends  
   - Detect shifts in hotspot locations

7. Export results  
   - Generate disturbance probability surface as raster (GeoTIFF)

---

## Final Outputs
- Disturbance probability raster  
  Spatial map of highest risk zones

- Risk cluster map  
  Statistically significant hotspot regions

- Temporal analysis  
  Seasonal and yearly variation in risk

- Interactive map  
  Time slider visualization of risk shifts (2020–2025)
