# Forecasting the stock-bond correlation: BUSN 35137 Group 2

This repo runs an out-of-sample horse race between three groups of predictors for next quarter's US stock-bond correlation: macro, liquidity/stress and news. Every group is benchmarked against the correlation's own history.

## Quick start
```
pip install -r requirements.txt
jupyter notebook final_project.ipynb     # or open it in VS Code and Run All
```
Run All takes about 9 minutes on a multi-core machine. Tables and figures are written to `output/`.

## Layout
| Path | What it is |
|---|---|
| `final_project.ipynb` | The full pipeline: data checks, target rebuild, framework, benchmarks, method grid (Table 2), nested models, combination, robustness, test (Table 1) |
| `Zip File All Data/` | All data. `panel_monthly.csv` is the combined, already-lagged monthly dataset |
| `output/` | Paper and slide tables (`.csv` + `.md`), figures (`.png`) and `frozen_design.json` |
| `CLAUDE.md` | Full design, decisions and results so far. Claude Code reads it automatically, and it is the best single summary for humans too |
| `*.docx` | The proposal and the original team plan |

## The one rule that matters
`RUN_TEST = False` in the notebook's setup cell keeps the 2018–2025 test window untouched. Flip it to True **once**, when the whole team agrees the design is final. See `CLAUDE.md` for the agreed design and the pre-registered decision rule.

## Data note
The repo includes CRSP data obtained through WRDS, which may not be redistributed. **Keep the repository private** and share it only with the group.
