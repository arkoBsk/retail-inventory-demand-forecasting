# Tableau dashboard build guide

Data: connect Tableau to `tableau_data.xlsx` (sheets below) or to the matching CSV files. Relate sheets on `Category` (and `Date` where noted).

| Dashboard visual (from the report) | Data sheet | Build |
|---|---|---|
| Forecasted demand trends by product category | `forecast_long` | Columns: `Date`. Rows: `Actual` and `Forecast`. Colour: `Model`. Filter: `Category`, and keep `Model` to ARIMA / Prophet (or show the baselines for context). |
| Inventory levels vs. predicted demand | `category_daily` + `forecast_long` | Dual-axis line: `Inventory_On_Hand` vs `Units_Sold` by `Date`; overlay `Forecast` for the test period (`Is_Test_Period = True`). |
| Model comparison (MAE, RMSE, MAPE) | `model_metrics`, `best_model_by_category` | Highlight table: `Category` x `Model`, colour by `MAPE (%)`. Add a text callout from `best_model_by_category`. |
| Store-wise stock movement | `store_category_kpis` | Bar chart of `Avg_Inventory` and `Units_Sold` by `Store ID`; filter by `Category`. |
| Reorder points and safety stock | `inventory_policy` | Table of `Safety stock`, `Reorder point`, `Order-up-to level` by `Category`. Show the assumptions (lead time, service level) in a caption; they are inputs, not data. |
| Filters | all | Store location: `Store ID`. Time period: `Date`. Category: `Category`. |

## KPIs

| KPI | Source | Note |
|---|---|---|
| Stockouts | `store_category_kpis[Stockout_Rows]`, `Stockout_Rate_%` | Proxy: rows where units sold equals inventory level. The dataset has no explicit stockout flag. |
| Forecast error rate | `best_model_by_category[MAPE (%)]` | Hold-out MAPE of the best ARIMA/Prophet model per category. |
| Wastage % | not available | The dataset has no wastage or write-off field. Join your wastage records on Date / Store ID / Product ID, or drop this KPI. |

The workbook itself (.twbx) is built in Tableau Desktop or Tableau Public; this file only maps the exported data to each visual.
