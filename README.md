# Santa Rosa Beach, FL (30A) — Underwriting map

Static site (no build step for the page), the Gulf Shores **Underwritten Properties** page (`underwritten.html`) ported to this market. It is only the map so far: `index.html` forwards to `underwritten.html`, keeping any `#p=` link. There are no UW sheets yet, so it starts empty: **+ Add listing** puts a Zillow listing on the map to rank its comps, build a comp set and download a UW CSV.

## Data

- Workbook `../Santa Rose Beach, FL.xlsx` (snapshot 2026-11-02), sheet `Cleaned_Data`: 133 entire homes, Santa Rosa Beach ZIP 32459, Dune Allen to Seagrove Beach along Scenic Hwy 30A. Every home earns $90k or more, so the default $90k floor for Match by location filters nothing here.
- `../notebooks/market_common.py` (same API as the Gulf Shores module) gives each home:
  - its **area**, from Ward clusters (k=3) of the coordinates, named from the place names in the listings' own titles and descriptions: **Dune Allen & Gulf Place** (west 30A, 33 homes), **Blue Mountain Beach** (to Grayton Beach, 41) and **Seaside & Seagrove** (east of Grayton Beach State Park, the widest gap in the homes, 59). A new target takes the nearest listing's area (right for all 133 in a leave-one-out test);
  - its **position on the water**: Gulf-front / Bay / lake / Off the water. Gulf-front = within 100 m of the Gulf shoreline, or within 600 m when the title or description says beachfront, unless it says across the street / across 30A or Scenic Gulf Drive. Bay / lake = lakefront, sits on Choctawhatchee Bay or a (coastal dune) lake, or has its own or a community dock or pier ("lagoon pool", "lakefront views" and "kayaking on the lake" don't count). Result: Gulf-front 7, Bay / lake 3, Off the water 123. The Airbnb waterfront flag is not used on its own. Listings can be fixed one by one in `../notebooks/position_overrides.csv` (Property ID, position, evidence; none yet);
  - its size bucket 1-2BR / 3BR / 4BR / 5BR+ (1BRs and 2BRs measured against their own bedroom count, as Gulf Shores) and tiers.
- The Gulf shoreline is the OSM coastline from Destin to Inlet Beach (`../notebooks/coastline_osm.json`, Overpass 2026-11-02), densified to 20 m, keeping the southern-most points per 100 m east–west slice.

## Pipeline

1. `python scripts/build_underwritten.py` (the `str` venv; add `--offline` to skip geocoding) writes `data/*.json`, `tests/fixtures/place_check.json` and `reports/comp_audit.csv`, then runs `node tests/test_underwritten.mjs`.
2. Browser test: `NODE_PATH=<playwright-core node_modules> node tests/test_underwritten_browser.mjs`.
3. Drop "UW'ing - NN.csv" exports into `underwriting/source_csv/` and rebuild to add underwritten targets. New sheets start from `underwriting/template/UW'ing - template (from Island Park 112).csv` with the five standard setup lines and fifteen OPEX lines (Galveston file 101's amounts).

## Differences from the Gulf Shores page

- Water class "Bay / lake" (key `Bay-lake`) instead of "Bay / lagoon / canal"; the target notes rule reads lakefront / dune lake / bay / dock words.
- Map projection centred on this market (`LAT0`, `LON0` in `market_common.py`, the same numbers in `js/underwritten/uw-geo.js`).
- Zillow city names for the address parser (`CITIES` in the build and in `uw-geo.js`), Florida as the default state.
- Popup label column capped at 45% of the popup (`.uw-pop__dl`), so long 30A street names in "Distance to …" wrap instead of making the popup taller than the map.
- No main-page Underwriting strip (`uw-strip.js`): there is no main page yet.
- The targets sort label is shortened to "By area (Dune Allen → Seaside)" so it fits a phone screen.
