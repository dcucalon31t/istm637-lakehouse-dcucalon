# ISTM 637 Governed Lakehouse Project

**Name:** Diego Cucalon  
**NetID:** dcucalon31  
**Catalog:** `istm637_dcucalon31`  
**Schema:** `oilgas`

This project builds a governed oil-and-gas lakehouse in Databricks using Unity Catalog, Lakeflow, Genie, AI/BI dashboards, MLflow, predictive modeling, Databricks Apps, and OpenSharing.

## Repository contents

- `ISTM637_Databricks_Project_Starter.ipynb` — catalog, schema, volume, governance, and validation steps
- `ISTM637_Lakeflow_Ingest_Pipeline.sql` — declarative ingestion pipeline for the three source CSV files
- `ISTM637_Predictive_Model_Notebook.ipynb` — model training, evaluation, registration, and 180-day forecasts
- `REPORT.md` — written project report covering Parts 1–8
- `screenshots/` — submission evidence organized by project part

## Key results

- `dim_well`: 50 rows
- `dim_date`: 547 rows
- `fact_production`: 22,806 rows
- Model: `istm637_dcucalon31.oilgas.oil_rate_predictor`, version 1, alias `champion`
- MAE: 33.5 bbl/day
- RMSE: 70.0 bbl/day
- R²: 0.933
- `well_forecast`: 6,660 rows covering 37 wells and 180 forecast days

Source CSV data and credential files are intentionally excluded from this repository.
