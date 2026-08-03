# ISTM 637 — Databricks Governed Lakehouse Project Report

**Student:** Diego Cucalon  
**NetID:** dcucalon31  
**Unity Catalog catalog:** `istm637_dcucalon31`  
**Schema:** `oilgas`  
**GitHub repository:** https://github.com/dcucalon31t/istm637-lakehouse-dcucalon

## Part 1 — Git integration and reproducibility

I created a public GitHub repository and connected it to a Databricks Git folder. The starter notebook, Lakeflow ingestion pipeline, predictive-model notebook, README, and supporting documentation were committed to the `main` branch. The repository contains multiple meaningful commits, including the initial project files, governance work, and the completed predictive model and forecasts.

## Part 2 — Lakehouse setup and Lakeflow ingestion

The three supplied CSV files were uploaded to the managed Unity Catalog volume at:

`/Volumes/istm637_dcucalon31/oilgas/raw`

The Lakeflow Declarative Pipeline created three materialized views in `istm637_dcucalon31.oilgas`. The completed pipeline produced the following validated row counts:

| Table | Rows |
|---|---:|
| `dim_well` | 50 |
| `dim_date` | 547 |
| `fact_production` | 22,806 |

The pipeline completed successfully, and the `fact_production` expectations were met.

## Part 3 — Governance, tags, and AI-generated comments

I applied governance tags to all three tables:

| Table | Domain | Layer | PII |
|---|---|---|---|
| `dim_well` | `well_master` | `silver` | `none` |
| `dim_date` | `calendar` | `silver` | `none` |
| `fact_production` | `production` | `silver` | `none` |

I used Databricks AI to generate column comments and then reviewed every suggestion before saving it. I corrected generic descriptions so that the comments included the proper business meaning and units—for example, barrels for `oil_bbl`, thousand cubic feet for `gas_mcf`, psi for pressure fields, 64ths of an inch for choke size, and feet for lateral length. I also clarified the table grain, the `well_id` and `date_id` join keys, operational status values, and the purpose of the `_rescued_data` ingestion column. A final metadata check confirmed that every column in all three tables had a comment.

## Part 4 — Genie Agent and validation tests

I created the **Oil and Gas Production Analysis** Genie Agent using the three governed tables as sources. I added a domain description, reviewed the generated instructions, and supplied trusted SQL examples covering basin production, time trends, and producing-well counts.

| # | Test question | Validated result | Correct? | Fix or validation performed |
|---:|---|---|:---:|---|
| 1 | Which basin produced the most oil overall? | Permian Basin, approximately 2.2 million barrels | Yes | Reviewed the `well_id` join and descending oil-total sort. |
| 2 | Show the monthly oil production trend for 2024. | Production peaked in May at 731,000 barrels and ended December at 429,000 barrels. | Yes | Verified the `date_id` join, 2024 filter, and month ordering. |
| 3 | Which operator has the most producing wells? | Coterra and Marathon Oil tied at 7 producing wells each. | Yes | Verified the `Producing` status filter and distinct well count. |
| 4 | List the top five wells by total gas production. | Weld 35-34, Lea 20-2, Martin 28-2H, La Salle 5-11H, and Eddy 25-15H | Yes | Checked the aggregation, well join, descending gas sort, and five-row limit. |
| 5 | What is the average water cut for Permian Basin wells? | 51.79% | Yes | Confirmed the formula `water_bbl / (water_bbl + oil_bbl)`, null handling, and division-by-zero protection. |

No final numerical answer required correction. To improve reliability, I strengthened the metadata with units and join-key descriptions and retained trusted SQL examples for the main analytical patterns.

## Part 5 — AI/BI dashboard

I used Genie Code to generate the dashboard datasets and visualizations rather than supplying a separate manually written dashboard query. The published **Oil and Gas Production Dashboard** contains:

- A calendar-date range filter
- A basin multi-select filter
- Total oil and total gas KPI counters
- A monthly oil-production line chart
- An oil-production-by-basin bar chart
- A producing-wells-by-operator bar chart

I tested both filters and confirmed that the KPI values and visualizations changed with the selected basin and date range. The dashboard was published using shared data permissions. I attempted to share it with my partner, but Databricks Free Edition did not return the external user in the recipient search, consistent with the instructor’s Free Edition announcement.

## Part 6 — Predictive model and 180-day forecast

The predictive notebook built a feature table by joining `fact_production`, `dim_well`, and `dim_date`. The model predicts a well’s daily oil rate using well age, well attributes, operating measurements, and categorical geological fields. I trained a `HistGradientBoostingRegressor` inside a scikit-learn preprocessing pipeline.

### Evaluation metrics

| Metric | Result |
|---|---:|
| MAE | 33.5 bbl/day |
| RMSE | 70.0 bbl/day |
| R² | 0.933 |

The model was registered in Unity Catalog as `istm637_dcucalon31.oilgas.oil_rate_predictor`, version 1, with the alias `champion`.

### Sample 180-day forecast

The sample forecast used `WELL-0001`. It generated 180 daily prediction rows with a total forecast of approximately **51,855 barrels** over the 180-day horizon. Forecasts were then generated for all 37 producing wells and saved to `istm637_dcucalon31.oilgas.well_forecast`, producing **6,660 rows** (`37 wells × 180 days`).

## Part 7 — Databricks App

The application reads its well selector and well attributes from `dim_well`. Historical monthly production comes from `fact_production` joined to `dim_date` through `date_id`, while the forward 180-day curve comes from the precomputed `well_forecast` table through `well_id`. The app therefore uses governed Unity Catalog tables directly and does not contain hard-coded production data or need to load the MLflow model at runtime. Databricks reported the app as **Running**, but opening its URL returned **App Not Available**, which the instructor identified as an acceptable Databricks Free Edition limitation.

## Part 8 — OpenSharing attempt and open-protocol fallback

I configured an OpenSharing share and selected a governed oil-and-gas table. I also entered my partner’s recipient name and Databricks sharing identifier. Databricks Free Edition blocked recipient creation with the message that External Delta Sharing was not enabled on the metastore, so I documented the complete setup and error state as instructed.

### How a non-Databricks recipient would consume the share

With external Delta Sharing enabled, the provider creates a share, adds the desired tables, creates an open-protocol recipient, and securely sends the generated credential file to that recipient. The credential file contains the endpoint and short-lived access information needed by an open-source Delta Sharing client. It must be transferred securely and must never be committed to GitHub.

The recipient could use the open-source Python client as follows:

```python
# pip install delta-sharing
import delta_sharing

profile = "/secure/path/recipient.share"
client = delta_sharing.SharingClient(profile)

# Inspect the shares available through the credential profile.
for share in client.list_shares():
    print(share)

# Replace the placeholders with the names supplied by the provider.
table_url = f"{profile}#<share_name>.<schema_name>.dim_well"
dim_well = delta_sharing.load_as_pandas(table_url)
print(dim_well.head())
```

A Spark recipient with the Delta Sharing connector could load the same table with:

```python
profile = "/secure/path/recipient.share"
table_url = f"{profile}#<share_name>.<schema_name>.dim_well"

shared_df = spark.read.format("deltaSharing").load(table_url)
shared_df.createOrReplaceTempView("shared_dim_well")
```

The recipient could then query the temporary view with SQL:

```sql
SELECT basin, COUNT(*) AS wells
FROM shared_dim_well
GROUP BY basin
ORDER BY wells DESC;
```

For a Databricks-to-Databricks recipient, the recipient would create a catalog from the received share and query it using standard SQL:

```sql
CREATE CATALOG partner_oilgas
USING SHARE `<provider_name>`.`<share_name>`;

SELECT *
FROM partner_oilgas.oilgas.dim_well
LIMIT 20;
```

## Conclusion

The project produced a governed Unity Catalog lakehouse, a successful Lakeflow ingestion pipeline, complete metadata and tags, a validated Genie Agent, a published interactive dashboard, a registered predictive model, a 180-day forecast table, and a deployed Databricks App attempt. GitHub provides the reproducible source artifacts, while the submitted screenshots document Databricks outputs and the Free Edition sharing and application limitations.
