# Phase 3 Checklist: Changes to Realign with Phase 1

The first Phase 3 pass (`../notebooks/data_exploration.ipynb`, `../reports/phase3/phase3_report.tex`) framed
the task as forecasting **next month's** bill from previous months. We are keeping the Phase 1 framing:

> On day `d` of a billing month, project that month's end-of-month bill for each household, with a
> prediction interval. The kWh recorded on days 1..d is known; the model predicts the kWh for days
> d+1..end and adds it to the known part.

The bill is a computed proxy (kWh x 14.228 p/kWh), not a real invoice.
Rubric: `spec/phase3_details.pdf`. Tick items off as they are completed.

## 1. Notebook hygiene
- [x] Add a config cell at the top: paths, the tariff rate, candidate `d` values (e.g. 7, 14, 21), thresholds
- [x] Replace hardcoded `../data/...` and `../reports/...` paths with the config values
- [x] Fix the histogram cell: it saves to `reports/` instead of the figures folder
- [x] Save all figures to `reports/figures/` (per `reports/CLAUDE.md`)
- [x] Use `orientation="horizontal"` instead of the deprecated `vert=False` in the box plot
- [x] Restart & Run All passes cleanly, with execution counts in order

## 2. Scope
- [x] Decide the scope: main analysis is **standard-tariff households only** (Phase 1 scope; decided 2026-11-10).
      ToU households are kept for the stretch goal only.
- [x] Apply the filter (`tariff = 'Std'`) at the top of the notebook and log the counts: 106,225 -> 84,807
      household-months, 5,548 -> 4,428 households (re-checked in `feature_engineering.ipynb` section 2: confirmed)
- [x] Drop `tariff_tou` from the feature set: it is constant once filtered

## 3. EDA: keep, fix, add
Keep: descriptive statistics, distributions, seasonal trend, temperature relationship, tariff and ACORN
comparisons, monthly gap check, duplicate check.
- [x] Reword "correlation with bills": since bill = kWh x a fixed rate, these are correlations with kWh
- [x] Reframe the IQR outliers: a pooled IQR mostly flags high-consuming households, not unusual months
- [x] Add a within-household outlier check (each month against that household's own history)
- [x] Add daily-level EDA from `data/processed/daily.parquet`, which Phase 3 never used:
  - [x] Distribution of daily kWh; day-of-week and holiday effects
  - [x] How much of a month's kWh is already recorded by day d (share of month total at each d)
  - [x] Error of the naive run-rate projection at each candidate d (how fast it converges)
- [x] Check same-month-last-year availability (most households start Apr-Jul 2012)

## 4. Feature engineering: build the day-d dataset
One row per household x month x `d` (for each candidate `d`), from `daily.parquet`.
Every feature may use only days 1..d of the target month, or earlier months.
- [x] Month-to-date features:
  - [x] `kwh_to_date` (days 1..d)
  - [x] `days_elapsed` (= d) and `days_remaining`
  - [x] `daily_mean_to_date` and the mean of the last 7 days
  - [x] `run_rate_kwh` = `kwh_to_date` / d x days in month
- [x] Earlier-month features (keep the consecutive-month checks already written):
  - [x] `previous_month_kwh`
  - [x] `rolling_3month_kwh`
  - [x] `same_month_last_year_kwh` (12-month lag)
- [x] Calendar features known in advance:
  - [x] Month as a sine/cosine pair (so December sits next to January)
  - [x] Weekend days and bank holidays remaining in the month
- [x] Weather: temperature to date only (the full-month `temp_mean` includes future days, so it leaks)
- [x] Household features: tariff and ACORN group
- [x] Targets:
  - [x] `remaining_kwh` (days d+1..end), the model's target
  - [x] `final_bill_gbp` = (`kwh_to_date` + `remaining_kwh`) x tariff
- [x] Benchmark columns, so Phase 4 can score them directly:
  - [x] Run-rate bill
  - [x] Last month's bill
  - [x] Same month last year's bill
- [x] Measure availability of each lag feature (rows with / without a value)
- [x] Save as a new file (e.g. `data/processed/features_day_d.parquet`); leave Phase 2 outputs untouched

## 5. Encoding and scaling
- [x] Drop one category from each one-hot set (season, ACORN) for linear models: all four always sum to 1
- [x] Drop redundant calendar columns (`month_number`, `quarter`, season dummies overlap); keep one representation
- [x] Keep scaling deferred to Phase 4, fitted on the training period only; say so in the report

## 6. Validation checks (cell that asserts, like Phase 2 section 13)
- [x] No duplicate household x month x d rows
- [x] `kwh_to_date` + `remaining_kwh` = the month's total kWh for every row
- [x] No feature uses a day after d (spot-check one household by hand)
- [x] Row count = kept household-months x number of candidate d values

## 7. Report
- [x] Restore the Phase 1 research question and state the day-d framing
- [x] Rebuild on the shared template: `reports/phase3/main.tex` with `../_template/preamble`, team names, course
- [x] State that the bill is a computed proxy
- [x] Update the statistics, figures and insights for the day-d framing (add the daily EDA figures)
- [x] Features table: description and justification for each day-d feature (rubric deliverable)
- [x] Insights list: tie each insight to a modeling decision for Phase 4
- [x] Check every number in the report against the notebook outputs
