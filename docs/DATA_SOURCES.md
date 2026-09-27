# Getting the official data

Portals change and several need a free registration or an official request, so verify each link and licence. Save files under `data/raw/`, then edit the matching layer in `config.yaml` (`enabled: true`, correct `path`, `attr`, `mapping`) and run `make build`.

| Need | Official source | What to get | How it plugs in |
|---|---|---|---|
| Landslide susceptibility | GSI **Bhukosh** (bhukosh.gsi.gov.in), National Landslide Susceptibility Mapping | NLSM raster or shapefile, landslide inventory | Raster: set `kind: raster`, `mapping` from class value to score. Shapefile: `kind: polygon`, set `attr` |
| Landslide history | GSI Landslide inventory (Bhukosh); NASA Global Landslide Catalog | Point events with date and fatalities | NASA one is fetched by `make fetch`. Convert GSI points to `name,type,year,lon,lat,deaths` and list in `events:` |
| Earthquake zones | BIS **IS 1893:2016** seismic zone map; BMTPC Vulnerability Atlas of India | Zone II to V polygons (digitise the map if no GIS file is published) | `kind: polygon`, `attr: ZONE`, `mapping: {II: .2, III: .4, IV: .7, V: 1}` |
| Earthquake history | National Center for Seismology (seismo.gov.in) catalogue; USGS ComCat | Magnitude, location, date | USGS is fetched by `make fetch`. Add NCS rows in the same CSV format |
| Flood hazard | NRSC/ISRO **Bhuvan** flood hazard atlases and NDEM; Rashtriya Barh Ayog flood-prone area maps; state flood-hazard zonation | Flood-prone or hazard-class polygons | `kind: polygon`, set `attr` and `mapping` (Low/Moderate/High to 0.3/0.6/0.9) |
| Flood history and river levels | **CWC** flood forecasting and **India-WRIS** (indiawris.gov.in): gauge stations, danger levels, historical peaks | Station tables, Flood Hazard/Danger-level exceedances | Convert flood events to the events CSV. Danger-level exceedance years are good event records |
| Cloudburst and rainfall | **IMD** gridded rainfall (imdpune.gov.in; Python package `imdlib`), CHIRPS (UCSB CHC) | Daily rainfall grids | Next step: use extreme-rainfall percentiles as a live multiplier on landslide and flood scores |
| Elevation and slope | SRTM (NASA Earthdata, free login), Cartosat DEM via Bhuvan | DEM tiles | Derive slope with `gdaldem slope`. Use it for candidate-site screening (`slope_deg`) |
| Habitations and population | **Census of India** Primary Census Abstract (village level), SECC, state village directories, Survey of India boundaries | Village name, population, coordinates | Build `data/raw/habitations.csv`: `name,lon,lat,population,vulnerable_share` |
| Candidate relocation land | State revenue department land records, panchayat and forest-department parcels | Parcel centroid, area, slope, water, road distance | Build `data/raw/candidate_sites.csv` |
| Past relocation outcomes | State DMA and NDMA reports, World Bank project documents (Latur/Killari, Bhuj, Kosi) | Households moved, cost, return rates | Use to tune `persons_per_ha` and site rules |

## Steps
1. Run `make fetch`. Open sources (USGS, NASA GLC, boundaries) download automatically. If one fails, its message tells you to fetch it by hand.
2. Download each official layer you can access. If a portal offers only a PDF map, use QGIS to georeference and digitise it.
3. Open each file in QGIS: check the CRS, the attribute column and the class values, and write them into `config.yaml`.
4. Set `seed_regions` to `enabled: false` once official hazard layers cover all three hazards. The seed regions are hand-drawn approximations.
5. Run `make build`, then `make backtest` to see how held-out events fall on the map.
6. Prepare habitations and candidate sites, then run `make relocate`.
7. Have geologists and district officials verify the top-ranked habitations on site before any relocation decision.

## Notes
- Earthquake hazard is not a relocation trigger in this tool. It reports retrofit advice, since shaking cannot be avoided by moving a few km.
- The grid is about 11 km. For village-level decisions, rasterise at 30 m in a smaller area and keep `resolution_deg` small.
- Event weights and the risk formula are transparent starting points. Calibrate them with local experts.
