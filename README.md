# India Red Zone Atlas

Multi-hazard (flood, landslide, earthquake) red-zone mapping for India, with historical-event weighting and a relocation planner that ranks vulnerable habitations and assigns them to safe candidate sites within capacity. A static MapLibre web app shows the result.

**Status:** the pipeline and web app work end to end. Out of the box it uses hand-curated seed regions and 20 documented disasters. Replace them with official GSI, NDMA, CWC, BMTPC and Census data (see `docs/DATA_SOURCES.md`) before using it for real decisions.

## Quick start
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
make fetch       # USGS quakes, NASA landslides, boundaries (needs internet)
make build       # hazard grid, heatmaps, events
make relocate    # ranks habitations, assigns sites (uses sample inputs until you add yours)
make serve       # open http://localhost:8000
```
Without `make`: run `python scripts/<name>.py` in the same order.

## How it works
1. **Layers** (`config.yaml`): each hazard type takes the maximum score over enabled layers (Gaussian seed regions, polygons such as seismic zones, rasters such as landslide susceptibility) on a 0.1° grid.
2. **History:** every past event adds a Gaussian bump (0.7° reach) to its hazard type. Casualty-heavy, repeated events therefore push nearby cells into the red.
3. **Red zone:** score of 0.7 or more. `scores.bin` and four PNG heatmaps go to `web/data/`.
4. **Habitation risk:** `100 x (0.7 x max(flood, landslide) + 0.3 x vulnerable share)`. Phases: Immediate 60+, Short-term 45+, Medium-term 30+, else Monitor (editable).
5. **Sites:** candidate land must be in low-hazard cells and gentle slope. Capacity is `area_ha x persons_per_ha x suitability`, and habitations are assigned greedily by risk, nearest site first.
6. **Web app:** click anywhere for scores, nearby events and advice. Click a habitation for its plan.

## Rakshak ResQ (linked, not merged)
This repo is the standalone PS deliverable — mapping, red zones, relocation
planning. It intentionally does **not** contain Rakshak ResQ's source code
(the separate SOS/evacuation/chat app). The web app has an "Open Rakshak
ResQ" button (in `web/index.html`) that links out to it instead. To point it
at your Rakshak ResQ deployment, edit `web/partners.config.js`:
```js
window.PARTNERS = { RAKSHAK_RESQ_URL: "https://your-rakshak-resq-url" };
```
Use its live deployed URL once you have one; until then, point it at the
Rakshak ResQ GitHub repo or a release `.zip` link. Leaving it blank disables
the button (with a tooltip explaining why) rather than linking to a dead URL.

## Layout
```
config.yaml   scripts/ (fetch_open_data, build_hazard, plan_relocation, backtest)
data/seed/    data/raw/ (official downloads)   data/processed/
web/          web/partners.config.js (Rakshak ResQ link, no code copied in)
docs/DATA_SOURCES.md   .github/workflows/pages.yml
```

## Publish on GitHub Pages
```bash
git init && git add . && git commit -m "Initial commit"
git branch -M main && git remote add origin https://github.com/<you>/india-redzone-atlas.git
git push -u origin main
```
Then in the repo: Settings, Pages, Source: GitHub Actions. Commit the generated `web/data/` files, since the workflow publishes `web/` as is.

## Limits
- Seed regions are simplified approximations. Fatality numbers are rounded from widely reported figures.
- The grid is coarse (about 11 km), so this is regional screening, not parcel-level assessment.
- Backtesting on a small event set is a sanity check, not validation.
- OpenStreetMap tiles are for light use. For heavy traffic use a tile provider.
- Boundaries from geoBoundaries may differ from the official Survey of India outline.

## Roadmap
ML landslide model (XGBoost on DEM, rainfall, lithology), live IMD rainfall multiplier, 30 m district rasters, PostGIS and FastAPI backend, SDMA login and alerts.
