# Phase 4 Checklist: Modeling & Analysis

Notebook: `../notebooks/modeling.ipynb` (new; the teammate's `modeling_analysis.ipynb` is left as written).
Rubric: `spec/phase4_details.pdf`. Framing unchanged from Phase 1: on day `d`, project the month-end bill
(kWh x 14.228 p/kWh, a computed proxy) per standard-tariff household, with a 90% prediction interval, then alert
if it exceeds the household's baseline by more than X.

## 1. Model selection (rubric task 1)
- [x] Regression; benchmarks from Phase 1 (run-rate, last month, same month last year) plus a fallback chain
- [x] Ridge (linear reference) and histogram gradient boosting, with the reason for each
- [x] Target transformation: ratio to the run-rate's remaining kWh vs raw remaining kWh

## 2. Training and splitting (rubric task 2)
- [x] Time-based split by month: 3 expanding-window folds (Jan-Mar, Apr-Jun, Jul-Sep 2013), test Oct 2013-Jan 2014
- [x] Preprocessing (imputation, scaling) fitted inside each fold's training months only
- [x] Built-in random early stopping switched off (it would mix months)

## 3. Evaluation (rubric task 3)
- [x] MAE, RMSE, MdAPE and bias on the bill in GBP; by `d`, by month
- [x] Household-block bootstrap CI for model minus run-rate MAE
- [x] Interval coverage and width, by `d`, month and household size
- [x] Figures in `reports/figures/p4_*.png`

## 4. Optimization (rubric task 4)
- [x] Feature-group ablation with a stated drop rule
- [x] Ridge alpha and gradient boosting grid on the 3 folds
- [x] Permutation importance

## 5. Project-specific items (CLAUDE.md)
- [x] Pooled vs per-household (secondary question)
- [x] Prediction intervals that hold near 90% (absolute, scaled, CQR compared)
- [x] Decision layer: baseline, X, rule and final `d` chosen and justified
- [x] Config cell, no magic numbers, final checks cell, Restart & Run All passes

## 6. Report
- [x] `reports/phase4/main.tex` on the shared template (drafted by Claude on request; `% CHECK:` notes for review)
- [x] Check every number in the report against the notebook outputs (automated match, 152 numbers)
