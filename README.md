# Butterfly Dashboard

Interactive Streamlit dashboard for exploring Australian butterfly occurrence records derived from Atlas of Living Australia data.

Public dashboard URL: https://butterfly-dashboard.streamlit.app/

## Data

The hosted dashboard uses pre-aggregated ABS Statistical Area polygons, not raw
occurrence rows and not rounded coordinate point bins. The default map view is
SA2, with a sidebar toggle for SA1, SA2, and SA3. Observer/user fields,
comments, raw media, and profile links are not included.

Current packaged dashboard data:

- `data/butterfly_sa1_bins.parquet`
- `data/butterfly_sa2_bins.parquet`
- `data/butterfly_sa3_bins.parquet`
- `data/sa1_boundaries_2021.parquet`
- `data/sa2_boundaries_2021.parquet`
- `data/sa3_boundaries_2021.parquet`
- `data/sa1_dimensions.json`
- `data/sa2_dimensions.json`
- `data/sa3_dimensions.json`
- `data/reference/butterfly_conservation_status.csv`

The SA1, SA2, and SA3 geographies come from the Australian Bureau of
Statistics ASGS Edition 3 `Statistical Areas - 2021 - Shapefile` boundary files
in GDA2020. The offline build uses the full ABS geometry for the spatial join,
then stores simplified display geometry as both WKB and GeoJSON in Parquet so
Streamlit can render polygons quickly. Runtime reads only these pre-aggregated
Parquet files.

Only records with `year >= 1950` are included, which keeps observations from
January 1950 onward when using the dataset's year-level date field. Records
before 1950 and records without usable coordinates are excluded from the map
artifacts.

Current packaged build summary:

| Metric | Count |
| --- | ---: |
| Source occurrence rows | 521,911 |
| Rows from 1950 onward | 431,012 |
| Rows from 1950 onward with coordinates | 426,421 |
| Records matched to ABS polygons | 422,406 |
| Coordinate rows not matched to an ABS polygon | 4,015 |
| Occupied SA1 polygons in artifact | 24,095 |
| Occupied SA2 polygons in artifact | 2,419 |
| Occupied SA3 polygons in artifact | 340 |

## Conservation Status

The dashboard data includes a curated threatened-species enrichment table at
`data/reference/butterfly_conservation_status.csv`. The table records EPBC Act
and state/territory listing evidence for Australian butterflies identified from
official Commonwealth and state sources, including SPRAT profiles, conservation
advice or recovery-plan links where available, protected-matters evidence, and
state threatened-species profiles or lists.

The packaged area-level parquet files carry these enrichment fields:

| Column | Meaning |
| --- | --- |
| `Status` | EPBC Act threatened status used as the main Commonwealth status field |
| `state_status` | State or territory listing status, usually prefixed by jurisdiction |
| `state_status_level` | Normalised state/territory level, such as `Critically Endangered`, `Endangered`, or `Vulnerable` |
| `state_status_for_occurrence` | Jurisdiction-specific state listing matched to the record's `stateProvince` |
| `epbc_listed_taxon` | Accepted EPBC-listed taxon matched to the occurrence |
| `state_listed_taxon` | Accepted state-listed taxon matched to the occurrence |
| `epbc_sprat_url` | SPRAT profile URL where available |
| `epbc_conservation_advice_url` | Commonwealth conservation advice URL where available |
| `epbc_recovery_plan_url` | Commonwealth recovery-plan URL where available |
| `epbc_protected_matters_url` | Protected Matters report or evidence URL where available |

Matching is intentionally conservative. Occurrences are matched first on exact
`scientificName`, then exact `species`, then by synonyms listed in the reference
CSV. Blank conservation fields mean the occurrence did not match the curated
reference table; they should not be interpreted as proof that the taxon has no
legal or conservation status.

The sidebar conservation filter has two controls. `Listing authority` selects
between `National EPBC` and `State / territory`. `Conservation status` then
filters to one or more status levels from the selected authority. When a
conservation status is active, polygon colors are forced to the `species` level
so the listed butterfly species under that status remain visually distinct.
State, year, family, genus, and species filters continue to subset the same
view.

## Map Coloring

The dashboard has one map view: the dominant category polygon map. The left
menu starts with an `ABS area level` toggle ordered SA1, SA2, SA3; SA2 is the
default because it gives a local map without the visual density of SA1. Each
polygon is colored by the category with the most observations in that area
after the current filters are applied.

| Active selection level | Polygon colors represent |
| --- | --- |
| No family, genus, or species filter | Dominant `family` in the active ABS area |
| Family filter active | Dominant `genus` within the selected family/families |
| Genus filter active | Dominant `species` within the selected genus/genera |
| Species filter active | Dominant `scientificName` within the selected species |
| Conservation status filter active | Dominant `species` under the selected national or state status |

Both Include and Exclude modes count as active taxonomy filters. Family-level
colors use a fixed palette so the six butterfly families are easy to
distinguish. Deeper levels use stable generated colors based on the active
genus, species, or scientific name value.

Polygon opacity represents the total number of filtered occurrence records in
that ABS area. The lowest opacity is used for one occurrence record. The highest
opacity is assigned to the polygon with the largest filtered record count currently
on the map. This means hue answers "which category dominates here?", while
opacity answers "how many records are here relative to the current view?"

Tooltips show the ABS area level/name, dominant category, dominant share, dominant record
count, total record count, exact composition text, and the same SVG pie chart
used by the previous dominant point tooltip.

State and year filters only subset the records shown. They do not change
whether colors represent family, genus, species, or scientific name.

| Family | Color | Area-matched record count |
| --- | --- | ---: |
| Hesperiidae | Red | 56,018 |
| Lycaenidae | Green | 86,471 |
| Nymphalidae | Blue | 182,424 |
| Papilionidae | Yellow | 37,123 |
| Pieridae | Purple | 60,346 |
| Riodinidae | Orange | 24 |

## Build Pattern

The reproducible source-side build lives in the parent project:

```bash
PYTHONPATH=. .venv/bin/python scripts/visuals/spatial_heatmap_dashboard/build_sa3_bins.py --area-level all
```

The build pattern is:

1. Download/extract the official ABS SA1, SA2, and SA3 2021 GDA2020 shapefiles.
2. Use DuckDB Spatial offline to join original occurrence coordinates to the selected ABS geography.
3. Filter to records with `year >= 1950`.
4. Store pre-aggregated Parquet artifacts.
5. Let Streamlit read only aggregates at runtime.

This keeps the public app small and fast, and gives a path for much larger
groups such as Aves: run the expensive spatial join once offline, then deploy
only pre-aggregated regional geometry artifacts.

## Local Run

```bash
python -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/streamlit run streamlit_app.py
```

Windows PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\pip install -r requirements.txt
.venv\Scripts\streamlit run streamlit_app.py
```

## Attribution

Occurrence data are derived from Atlas of Living Australia records. ABS
boundaries are derived from Australian Bureau of Statistics ASGS Edition 3
digital boundary files. This dashboard is intended for exploratory analysis and
should not be treated as precise locality disclosure.

## Deployment

Deploy on Streamlit Community Cloud with:

- Repository: `karikris/butterfly-dashboard`
- Branch: `main`
- Main file path: `streamlit_app.py`
