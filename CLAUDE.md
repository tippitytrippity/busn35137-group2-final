# BUSN 35137 Final Project (Group 2): Forecasting the stock-bond correlation

Context for Claude Code sessions. Everything here was agreed by the team; follow it rather than re-deriving or re-litigating it.

## The project
- **Team:** Abhi Ballal, Phillip Seo, John Walls, Arthur Zaharko.
- **Due 2026-11-22:** paper (5–10 pages of text), slide deck, 20-minute talk.
- **Question:** which predictor group best forecasts next quarter's US stock-bond correlation **out of sample**, beyond the correlation's own history?
  - A: macro
  - B: liquidity/stress
  - C: news
  - D: WSJ topics, robustness only; data end 2017-06
- **Source documents:** `Business 35137 – Group 2 Final Project Proposal.docx` (proposal) and `Project Instructions.docx` (Arthur's team plan, Oct 30). The design below **supersedes** both where they differ.

## Files
- `final_project.ipynb` is the whole pipeline. It runs top to bottom in about 9 minutes, with refits parallelized over years via joblib. Edit it directly.
- `Zip File All Data/panel_monthly.csv`: one row per month t. It holds the target `target_next3m_corr`, the trailing benchmark `bench_trailing3m_corr` and every predictor, **already lagged** to the end of month t. `coverage.csv` lists each column's first and last month.
- The raw daily files (`T1_*` Ken French, `T2_*` CRSP fixed-term Treasuries) are used to rebuild the target. The notebook asserts that the rebuilt 10Y series matches the panel exactly, which makes the 5Y/30Y targets for robustness trustworthy.
- `output/` holds the tables (CSV + markdown) and figures (PNG) for the paper, plus `frozen_design.json`.

## Target and timing
- **Target:** for row t, the correlation of daily (Mkt-RF + RF) with daily CRSP 10Y `TDRETADJ` over months t+1..t+3.
- **Trailing correlation:** for row t, the same over t-2..t, which equals the target of row t-3.
- **Forecast window:** 1987-01 to 2025-09, 465 rows. A, B and C are complete there.

## Agreed design
| Window | Rows | Role |
|---|---|---|
| Burn-in | 1987-01..1999-12 (156) | Starting history only. Never scored. |
| Validation | 2000-01..2017-12 (216; 214 scored) | Real-time OOS forecasts. Used for team design choices: best method per predictor set. |
| Test | 2018-01..2025-09 (93) | Run **once**, with everything frozen, to judge the winner. |

- **Yearly expanding refits**, the same procedure in validation and test:
  - For forecast year Y, fit on rows 1987-01..(Y-1)-10 and forecast Y-01..Y-12. Dropping (Y-1)-11/12 is the 2-row overlap rule: row t's target is realized only in month t+3.
  - **Hyperparameters are re-tuned at every refit** by purged, blocked 5-fold CV: consecutive blocks, no shuffling, 2 rows dropped either side of each held-out block.
- **Validation scoring** stops at 2017-10, because the targets of 2017-11/12 overlap test targets.
- **Every model includes the trailing correlation.** In the linear-family models it is unpenalized (Frisch–Waugh partialling-out). Penalized models, PCR and PLS can therefore all shrink exactly back to own history (ridge max penalty, lasso rel = 1, k = 0).
- **Methods:** OLS, ridge, lasso, elastic net, PCR, PLS, random forest, gradient boosting. The **headline** is a pre-specified equal-weight ridge + PCR average and is not a selection candidate.
- **Benchmarks:**
  - expanding historical mean
  - **random walk** (forecast = trailing correlation)
  - **own-history regression** (target on trailing correlation)
- **Pre-registered decision rule:**
  - The primary metric is test R² vs. own history.
  - A group adds value only if one-sided Clark–West p < 0.05 (Newey–West, 2 lags).
  - The winner is the top-ranked adder. It is a *clear* winner only if Diebold–Mariano also rejects vs. the runner-up.
  - Sign accuracy and the combination weights are secondary evidence.
  - The headline method is run on every group as a method check.
  - R² and Clark–West vs. the random walk are reported as a second bar, but are not part of the rule.
- **Combination:** inverse-MSE weights over the last 36 *realized* errors, with equal weights until 12 exist. Fixed in advance and never tuned.
- **Robustness**, validation window only:
  - WSJ topics in place of news
  - 5Y and 30Y Treasuries
  - a rolling 15-year window
  - an appendix of univariate models

## Test-window discipline (important)
- `RUN_TEST = False` in the setup cell. While it is False, **no 2018–2025 forecast is ever computed**.
- Set it to True only **once**, after the whole team agrees the design is final, and rerun the notebook.
- That run checks the selections against `output/frozen_design.json`, then writes timestamped `table1_test_*` files. Nothing may change after that.
- Do not add predictors, change grids or alter the decision rule after seeing results. The proposal fixed the variables, and we report everything we run.

## Validation results so far (2000-01..2017-10)
- The own-history regression loses to the random walk (R² −7%). Its intercept is learned partly from the positive-correlation 1990s, and the expanding window never forgets them.
- **News (C) is the only group that clears both bars:**
  - selected method OLS: +9.0% vs. own history, +2.7% vs. the random walk;
  - A+C with lasso: +9.9% and +3.6%.
- **Liquidity (B) adds nothing.** Cross-validation shrinks it fully to own history at every refit, contrary to Baele, Bekaert and Inghelbrecht (2010).
- **Macro (A) alone hurts** (−10% to −26%).
- **Random forests and gradient boosting fail everywhere**, with too little data.
- **News beats macro** (Diebold–Mariano p ≈ 0.015).
- The news result **holds** for 5Y and 30Y Treasuries and under a rolling window. The WSJ topics do not replace it.
- **Caveats:**
  - Clark–West vs. the random walk rejects even for some negative-R² models. It tests whether the true coefficients help, not whether the realized forecasts beat the random walk. Lead with R².
  - Validation R² for the *selected* methods is optimistic. Only the test and headline results are clean.
  - The test window has about 31 independent quarters, so it has low power.

## Remaining work
1. The team agrees the design is final and sets `RUN_TEST = True`, once.
2. Paper sections: intro, background/literature (from the proposal), data, methods, results, robustness, conclusion and allocator implications.
3. Slides: built around the correlation chart, Tables 1 and 2, and the combination-weights chart.
4. Check every number in the paper and slides against `output/`.

## Conventions
- Python 3.13, with numpy, pandas, scikit-learn, scipy, matplotlib and joblib (see `requirements.txt`). Run the notebook from this folder.
- Window dates, grids and seeds live in the notebook's setup cell. `SEED = 0`.
- Plot style matches the HW2 notebook.
