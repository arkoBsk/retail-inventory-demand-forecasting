# Retail Smart Inventory Optimization

Category-wise demand forecasting and inventory policy audit on a 76,000-row retail panel (5 stores x 20 products x 760 days, 2022-01-01 to 2024-01-30).

## What was done
- Built the forecasting and inventory-optimization pipeline end to end from the raw sales data, with a chronological 576-day train / 92-day validation / 92-day test split.
- Corrected a category-mapping error (a majority-vote relabelling) that had shifted category totals by -64% to +56%.
- Joined the US federal holiday calendar (28 dates) and used the uncensored `Demand` as the target instead of the stock-capped `Units Sold`.
- Final run passes 28/28 verification checks, including a poison test (corrupting test-window demand must leave forecasts bit-identical).

## Models and metrics
- Category-wise ARIMA (order chosen by AIC, p,q in 0-3, d in {0,1}), Prophet (weekly seasonality), recursive 92-day LightGBM (L1 objective, 3 seeds), two flat baselines, and ARIMAX/Prophet "scenario oracle" variants.
- Metrics: MAE, RMSE, MAPE, R2. Models selected on validation, scored once on test.

## Results
| Test MAPE | Clothing | Groceries | Electronics | Furniture | Toys | Average |
|---|---|---|---|---|---|---|
| ARIMA | 12.0% | 13.3% | 21.1% | 26.1% | 40.4% | 22.6% |
| Prophet | 14.7% | 15.5% | 21.0% | 24.5% | 40.5% | 23.2% |
| Baseline (training mean) | 11.2% | 13.3% | 20.7% | 23.6% | 40.5% | 21.9% |

- The models beat the flat baseline in only 2 of 5 categories. The best leakage-free LightGBM reached R2 = 0.122.
- **Leakage finding:** same-day Price lifts R2 from 0.122 to 0.923 (0.938 with all extra columns). Price correlates +0.797 with Demand on the same day but 0.225 one day apart, so Price was generated from Demand. The original report's R2 of 0.7033 is circular.

## Inventory policy
- Each Store-Product series has an exact fixed replenishment lag of 1-4 days (demand-weighted mean 2.45 days); lead time is 3 days for four categories and 2 for Toys.
- Safety stock = z x empirical L-day forecast-error SD. At 95% service, validation fill rate is 96.14%; 99% adds 0.90 pp of fill for 41% more safety stock.
- Wastage: zero under the stated shelf lives (median days of cover 3.4). Calibrating to published spoilage rates gives 0.859% of units sold for Groceries at an 11-day shelf life.
- Takeaway: the data has very little forecastable signal, and forecast-driven replenishment showed no measurable benefit in the backtest.

## Caveats
- The 11-day Groceries shelf life is tuned to a published benchmark band.
- Safety stock rests on a single 92-day validation window.
- `docs/Capstone_Project_Report_draft.docx` and `docs/tableau_dashboard_guide_v1.md` predate the final v8 run and are not updated to match it.

## Layout
- `data/` raw sales data (CSV and XLSX)
- `notebooks/` final executed notebook (v8)
- `outputs/` v8 CSV exports and `tableau_data.xlsx` (22 sheets)
- `docs/` report draft, Tableau guide, project overview pages
- `archive/` earlier builds (v1, v4) and the original notebook, kept for the audit trail

## Reproduce
```bash
pip install -r requirements.txt
cp data/sales_data.csv notebooks/
cd notebooks && jupyter notebook retail_forecasting_v8_executed.ipynb
```
The notebook looks for `sales_data.csv` in its own folder, so copy it next to the notebook as above. It writes its exports to `notebooks/outputs/` (the committed copies are in `outputs/`). Package versions are not pinned, so results may differ slightly on newer library versions.
