# Butterfly Spatial Heatmap Dashboard

Interactive Streamlit dashboard for exploring Australian butterfly occurrence records derived from Atlas of Living Australia data.

Public dashboard URL: https://butterfly-dashboard.streamlit.app/

## Data

The hosted dashboard uses pre-aggregated spatial bins, not raw occurrence rows. Locations are rounded and aggregated for exploratory visualization. Observer/user fields, comments, raw media, and profile links are not included.

Current packaged dashboard data:

- `data/butterfly_grid_bins.parquet`
- `data/dashboard_dimensions.json`

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
