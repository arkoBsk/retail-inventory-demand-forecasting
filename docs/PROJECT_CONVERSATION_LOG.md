# Project conversation log (handoff for another agent)

Reconstructed summary of the working session, not a verbatim transcript (the raw transcript was lost when the sandbox was replaced). User: Arko. Facts and numbers below come from the final v8 run in this repo.

## 1. What the user asked, in order
1. Draft the capstone project overview (5 page images in `docs/project_overview_pages/`) as a Word file. Result: `docs/Capstone_Project_Report_draft.docx`.
2. Convert `sales_data.csv` to Excel. Result: `data/sales_data.xlsx`.
3. Check whether the supplied notebook (`archive/original_notebook_...ipynb`) covers the whole project.
4. Build the missing pieces (ARIMA/Prophet forecasting, inventory policy, Tableau exports).
5. Audit the datasets against each other, fix inconsistencies, rebuild everything, verify with a different model/agent, and report findings. If gaps remain, loop; if inconsistencies return a third time, highlight them. Use real-world data for gaps, random data where acceptable, and ask the user for what is still missing.
6. User's answers to the three open inputs:
   - Lead time: no real data; research online, make a judgement and cite sources or reasoning.
   - Service level: analyse the three candidates and pick the one that makes most sense.
   - Wastage: use inventory level minus units sold as leftover stock; assume Groceries 2 months, Clothes 6 months, Toys and Electronics 12 months, Furniture 2 years; use real-world data if it can be found and cite it.
   Closing instruction: rebuild, re-run with another agent, check consistency, iterate until consistent.
7. Summaries for a one-page report: 3-4 bullets, then 3-4 sentences, then 50 words, then a resume-style template (title, dates, bullets starting with the triangle marker), then reworded to own the work and read clearly to a first-time reader, no arrows.
8. Clarified that the MAPE range 12.0-40.4% is ARIMA's spread across the five categories; average is 22.6% (ARIMA), 23.2% (Prophet), 21.9% (training-mean baseline).
9. Asked for LinkedIn skills for the project.
10. Asked to upload the project to GitHub like the UAE invoicing project. GitHub was not linked in that session, so the work was packaged as a repo-ready zip instead.

## 2. What was built, by iteration
- v1: first build. Used the notebook's category majority-vote fix, which corrupted category totals (Toys -64.7%). Removed in v2.
- v2: the "leakage-free" LightGBM still used same-day Price and Competitor Pricing; assertions were vacuous. Fixed with information tiers, recursive forecasting, checks that read exports from disk.
- v3: model selection excluded the baselines, summary used a test-set argmin, attribution mislabelled. Fixed in v4.
- v4: safety stock used demand SD instead of forecast-error SD; early stopping misreported. Fixed in v4-v5.
- v5: "95% reaches 100% fill" was really 99.9967% rounded; safety stock used sigma*sqrt(L) though errors are autocorrelated; service level was chosen on the test window; result depended on order-arrival convention. Fixed in v6.
- v6: order lag wrongly declared inconclusive; "99% buys nothing" was wrong on test. Fixed in v7.
- v7: order-weighted lag (1.79) was biased low, flat lag of 2 distorted wastage, review period not simulated. Fixed in v8 with per-series lags, demand weighting and a review gate.
- v8 (final): 28/28 verification checks pass. 8 iterations, 5 independent audits by a separate model.

## 3. Final results (v8)
- Test MAPE by category (ARIMA / Prophet / training-mean baseline): Clothing 12.0 / 14.7 / 11.2; Groceries 13.3 / 15.5 / 13.3; Electronics 21.1 / 21.0 / 20.7; Furniture 26.1 / 24.5 / 23.6; Toys 40.4 / 40.5 / 40.5. Averages 22.6 / 23.2 / 21.9.
- Models beat the baseline in only 2 of 5 categories. Best leakage-free LightGBM R2 = 0.122 (per-series mean 0.132).
- Leakage: same-day Price lifts R2 0.122 -> 0.923 (0.938 with competitor, weather, epidemic columns). Price correlates +0.797 with Demand same day, 0.225 one day apart; Price was generated from Demand. The original report's R2 0.7033 is circular (corr(Demand, Units Sold)^2 = 0.6946).
- Lead time: exact fixed lag of 1-4 days per Store-Product series, ledger reconstructed with zero error over 75,755 day-pairs, demand-weighted mean 2.45 days. Used 3 days for four categories, 2 for Toys.
- Service level: 95% gives 96.14% fill on validation; 99% adds 0.90 pp for 41% more safety stock. Chose 95%.
- Wastage: zero under the user's shelf lives (median days of cover 3.4). Calibrated to published spoilage rates: 0.859% of units sold for Groceries at an 11-day shelf life.
- Takeaway: the dataset has almost no forecastable signal; forecast-driven replenishment showed no measurable benefit in the backtest.

## 4. Open items and caveats
- The 11-day Groceries shelf life is tuned to a benchmark band.
- Safety stock rests on one 92-day validation window; service-level economics change roughly 5x with the order-arrival convention.
- The Word report has not been rewritten to the final honest framing; the user must confirm that direction first. The Tableau guide still reflects v1.
- Build scripts (`pipeline.py`, `build_nb.py`) were lost with the sandbox; the v8 notebook is the only code.

## 5. User preferences to honour
Output-first bullet structure, real numbers only (never invent figures), human-sounding writing, honest framing of gaps rather than overclaiming, concise outputs with clear word-count targets.

## 6. Asked of the receiving agent
Unzip, create a repo named `retail-inventory-demand-forecasting` under the user's account (match the naming and visibility used for the UAE invoicing project), and push this folder as the first commit.
