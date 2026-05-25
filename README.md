# Butterfly Dashboard

Interactive Streamlit dashboard for exploring Australian butterfly occurrence records derived from Atlas of Living Australia data.

Public dashboard URL: https://butterfly-dashboard.streamlit.app/

## Data

The hosted dashboard uses pre-aggregated spatial bins, not raw occurrence rows. Locations are rounded and aggregated for exploratory visualization. Observer/user fields, comments, raw media, and profile links are not included.

Current packaged dashboard data:

- `data/butterfly_grid_bins.parquet`
- `data/dashboard_dimensions.json`

## Map Coloring

Map dots change color by the most specific active taxonomy selection:

| Active selection level | Dot colors represent |
| --- | --- |
| No family, genus, or species filter | `family` |
| Family filter active | `genus` values within the selected family/families |
| Genus filter active | `species` values within the selected genus/genera |
| Species filter active | `scientificName` values within the selected species |

Both Include and Exclude modes count as active filters. Family-level colors use
a fixed palette so the six butterfly families are easy to distinguish. Deeper
levels use stable generated colors based on the active genus, species, or
scientific name value.

The default `Dominant category map` mode displays one large map. Each point is
colored by the category with the most observations at that rounded coordinate.
On the landing page, the dominant category is one of the six families. After
drilling into a family, the dominant category becomes the leading genus at each
point. Drilling into genus changes the color level to species, and drilling into
species changes it to scientific name. Point size is based on the total number
of filtered observations at that coordinate. Point opacity increases as the
dominant category's share increases. Tooltips show the dominant category,
dominant share, dominant record count, total record count, and full composition
for that map point.

Dominant map point sizing uses this radius scale. Values between table rows are
interpolated, and values above `50,000` records are capped at `26 px` radius.

| Records at map point | Radius | Diameter |
| ---: | ---: | ---: |
| 1 | 3 px | 6 px |
| 10 | 4 px | 8 px |
| 25 | 5 px | 10 px |
| 50 | 6 px | 12 px |
| 100 | 7 px | 14 px |
| 250 | 8 px | 16 px |
| 500 | 9 px | 18 px |
| 750 | 10 px | 20 px |
| 1,000 | 11 px | 22 px |
| 2,500 | 12 px | 24 px |
| 5,000 | 14 px | 28 px |
| 7,500 | 16 px | 32 px |
| 10,000 | 18 px | 36 px |
| 15,000 | 20 px | 40 px |
| 20,000 | 22 px | 44 px |
| 25,000 | 24 px | 48 px |
| 50,000 | 26 px | 52 px |

The optional `Compare category heatmaps` mode displays one heatmap per active
category value. On the landing page this means six family heatmaps. Selecting a
family changes the panels to genus heatmaps for that family, selecting a genus
changes the panels to species heatmaps, and selecting a species changes the
panels to scientific name heatmaps. State and year filters only subset records;
they do not change the active taxonomy level. Each panel sizes points by the
total observations at the coordinate and uses opacity to show that category's
share of the coordinate total.

The `Piechart composition markers` map mode draws one piechart per rounded
coordinate. Piechart size is based on the total number of filtered observations
at that map point. Pie slices show the proportions of the active family, genus,
species, or scientific name values registered at that same map point. Family
level pies show all family slices. Deeper levels show the top eight values and
combine the rest as `Other` so the marker stays readable.

The `Selected-category share heatmap` map mode keeps the same active color
level rules, then lets you pick one focus value from that level. Each bubble
shows where that focus value occurs after the current family/genus/species,
state, and year filters are applied. Bubble size is the focus value's
observation count at that rounded coordinate. Bubble opacity is the focus
value's share of all filtered observations at that coordinate. Tooltips show
the focus count, total coordinate count, share percentage, and top competing
values at the active color level.

State and year filters only subset the records shown. They do not change
whether colors represent family, genus, species, or scientific name.

| Family | Color | Mapped record count |
| --- | --- | ---: |
| Hesperiidae | Red | 78,204 |
| Lycaenidae | Green | 120,663 |
| Nymphalidae | Blue | 206,261 |
| Papilionidae | Yellow | 41,328 |
| Pieridae | Purple | 68,244 |
| Riodinidae | Orange | 68 |

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

Occurrence data are derived from Atlas of Living Australia records. This dashboard is intended for exploratory analysis and should not be treated as precise locality disclosure.

## Deployment

Deploy on Streamlit Community Cloud with:

- Repository: `karikris/butterfly-dashboard`
- Branch: `main`
- Main file path: `streamlit_app.py`
