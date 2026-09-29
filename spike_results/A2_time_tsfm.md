# Spike A2: D2 time-series foundation models (TIME-Output). Row ordering, reproduction, pilot rerun

**Date:** 2026-09-29 · **Machine:** BC Andromeda, CPU job on `short` (c098), 4 CPUs / 32 GB · **Wall time:** 65 s job (38 s of it download), about 30 min total including code reading.
**Verdict: PASS, with a correction.** The row ordering the previous pilot assumed was **wrong** for most datasets. The correct join is by name. It was verified by content and reproduces TIME's published numbers to 0.1%. The pilot's variance numbers are unaffected, but its feature-correlation claims change.

## What was run
- Env `d2` (py3.11, pandas 3.0.6, numpy 2.4.6, huggingface_hub 1.32, scipy, pyarrow) at `~/vis4ml_spikes/envs/d2`.
- Downloads (all under `~/vis4ml_spikes/data/time/`, about 270 MB total):
  - `Real-TSF/TIME-Output`:
    - `results/{seasonal_naive, Chronos-2, TimesFM-2.5, Moirai2, TiRex, Chronos-bolt}/**/{metrics.npz, config.json}`, with no `predictions.npz`;
    - `features/**`.
  - `Real-TSF/TIME` (the dataset itself, 0.22 GB of arrow files). It is needed for the `item_id` and `variate_names` order.
- Code: `~/vis4ml_spikes/work/a2/a2_time.py` (job `jobs/a2_time.sbatch`, SLURM 3073093). Outputs are in `work/a2/out/`: `ordering_check.csv`, `table5_repro.csv`, `table4_repro.csv`, `report.json`.

## 1. How metrics.npz indices map to features rows (FACT, from code + data)
Sources: `zqiao11/TIME` `src/timebench/evaluation/saver.py`, `feature/features_runner.py`, `evaluation/data.py`, and the HF Space `Real-TSF/TIME-leaderboard` `src/utils.py` / `src/leaderboard.py`.

**How `metrics.npz` is indexed**
- It has shape `(num_series, num_windows, num_variates)`. The series axis follows the order of the HF arrow dataset, i.e. `hf_dataset["item_id"]`. For multivariate data, the variate axis follows `variate_names` from the first row.
- For univariate (UTS) datasets, `V = 1` and the variate name *is* the `item_id`.

**How `features/*/test.csv` (or `full.csv`) rows are ordered**
- `features_runner.py` builds `unique_id = f"{series_name}_{variate}"` from processed CSV filenames. It then computes features via `groupby('unique_id')`, which **sorts the strings lexicographically**.
- It **drops rows with NaN features**.
- Result: the row order is neither the arrow order nor numeric order.

**How TIME's own leaderboard joins them:** by name, on `(dataset_id, series_name, variate_name)`. For UTS datasets it matches the features' `series_name` *or* `variate_name` against `item_id`.

**Result on the data (FACT, this run)**

| Check | Result |
|---|---|
| Datasets / metric series-variate keys | 50 / 6,633 |
| Keys matched to a feature row by name | 6,621. Missing: Crypto/D, 4 keys, where the names differ (`cryptocurrencies_d`/`Bitcoin` vs `item0`/`CoinbaseBitcoin`); SG_Carpark/15T, 8 keys, feature rows dropped as NaN |
| **Content check** of the name join: feature `mean` and `missing_rate` vs the last `length` points of the raw arrow target | **100% match** (6,621/6,621) |
| **Positional assumption** of the previous pilot (feature row k ↔ flat index s·V+v) | correct for all rows in only **16/50 datasets**; wrong for **5,860/6,621 keys (88.5%)** across 34 datasets, incl. azure2019_D (2,967 keys, 11% correct) |
| Row-count mismatch (the pilot's "49/50" check) | only SG_Carpark (354 vs 346). **Equal row counts did not imply equal order.** |

**Conclusion:** always join by name. The previous pilot's feature-level results (its Spearman ρ table) were computed on misaligned rows and are void. The helper in `a2_time.py` (`names_for` + name merge) is the correct join.

## 2. Reproduce TIME numbers (paper arXiv 2602.12147, Tables 4–5)
Method: the same as the leaderboard.
1. Average each variate's metric over windows.
2. Normalize by seasonal naive at the (dataset, series, variate, horizon) level.
3. Take the geometric mean.
4. Pattern = feature > global median.
5. Use all 3 horizons.

| Pattern / model | MASE ours | paper | Δ | CRPS ours | paper | Δ |
|---|---|---|---|---|---|---|
| Seasonal strength = 1, Chronos-2 | 0.5653 | 0.565 | +0.06% | 0.5089 | 0.509 | −0.02% |
| Seasonal strength = 0, Chronos-2 | 0.6544 | 0.654 | +0.06% | 0.5603 | 0.560 | +0.05% |
| Seasonal strength = 1, TimesFM-2.5 | 0.5880 | 0.588 | 0.00% | 0.5317 | 0.532 | −0.06% |
| Seasonal strength = 0, TimesFM-2.5 | 0.6635 | 0.664 | −0.07% | 0.5718 | 0.572 | −0.04% |
| Trend strength = 1 / 0, Chronos-2 | 0.5900 / 0.6301 | 0.590 / 0.630 | ≤0.02% | 0.5174 / 0.5549 | 0.517 / 0.555 | ≤0.08% |

The other models (TiRex, Moirai2, Chronos-bolt) are all within 0.08%; see `table5_repro.csv`.
Table 4 (overall, dataset-level normalization over 98 tasks):

| Model | MASE, ours vs paper | CRPS, ours vs paper |
|---|---|---|
| Chronos-2 | 0.6620 vs 0.662 | 0.5563 vs 0.556 |
| TimesFM-2.5 | 0.6686 vs 0.669 | 0.5674 vs 0.567 |
| TiRex | 0.6831 vs 0.683 | 0.5731 vs 0.573 |
| Moirai2 | 0.7030 vs 0.703 | 0.5885 vs 0.588 |
| Chronos-bolt | 0.7328 vs 0.733 | 0.6200 vs 0.620 |

**PASS:** everything is within 0.1%, against a target of 2%. The Chronos-2 high/low seasonal-strength MASE ratio is 0.5653 / 0.6544 = 0.864; the paper's is 0.565 / 0.654 = 0.864.

## 3. Pilot rerun with MASE and CRPS (50 short-horizon tasks, about 95.6k windows)
Relative error = model / seasonal naive, per window. Variance shares are of log relative error.

| Quantity | Previous pilot (MASE) | MASE now | **CRPS now** |
|---|---|---|---|
| Chronos-2: share of window-level variance **within series** | 0.66 | 0.662 | **0.454** |
| TimesFM-2.5: same | – | 0.624 | 0.422 |
| Chronos-2: share within task | 0.92 | 0.925 | 0.871 |
| Chronos-2: series-level share within dataset | 0.86 | 0.777 (see note) | 0.810 |
| Chronos-2 vs TimesFM-2.5: windows differing by more than 25% | 31.8% | 31.8% | **27.7%** |
| … median / p90 of abs(log ratio) | 0.116 / 0.564 | 0.116 / 0.564 | 0.108 / 0.454 |
| Chronos-2 vs TiRex, >25% | – | 35.1% | 31.0% |
| TimesFM-2.5 vs Moirai2, >25% | – | 33.0% | 29.1% |
| **Noise-floor control, same family:** Chronos-2 vs Chronos-bolt, >25% | – | 40.1% | 37.4% |

Notes (SYNTHESIS):
- The window-level MASE numbers reproduce the old pilot exactly (0.662, 31.8%). Those quantities never used the features, so the ordering bug did not affect them.
- The series-level within-dataset share came out at 0.78 rather than 0.86. My unit is the mean over windows of the ratio of MASEs; the old pilot probably used the ratio of means or the median. Treat it as definition-dependent.
- **CRPS lowers the within-series share from 0.66 to 0.45.** Part of the MASE within-series variance is scale-denominator noise, as the old §5 warned. **H1 (>50% within series) holds for MASE but *fails* for CRPS.** It stays well above the 0.3 falsification line in both. Pre-register H1 per metric.
- The "same-family control" is not a noise floor. Chronos-bolt vs Chronos-2 disagree *more* (40%) than Chronos-2 vs TimesFM-2.5 (32%), because Chronos-bolt is a much weaker model. A proper noise floor needs seed or sample reruns of one model, e.g. Chronos-2 with different sample seeds, or bootstrap over quantile levels. This is open.

**Feature correlations with the correct join** (Spearman, series-level log relative error of Chronos-2, short horizon, n = 6,621):

| Feature | MASE ρ | CRPS ρ | Old (misaligned) pilot |
|---|---|---|---|
| seasonal_strength | **−0.21** | −0.11 | abs(ρ) < 0.08 |
| x_entropy | **+0.21** | +0.15 | abs(ρ) < 0.08 |
| spike | +0.12 | +0.07 | abs(ρ) < 0.08 |
| length | −0.11 | −0.10 | −0.26 |
| missing_rate | −0.10 | −0.06 | +0.15 |
| trend_strength / seasonal_corr | ≈0 | ≈0 | – |

SYNTHESIS: static features still explain little (all abs(ρ) ≤ 0.21), so the §9 risk stands. But *which* features matter changed: seasonality and entropy, not length or missingness. That is also consistent with TIME's own Table 5, which was built with the name join. Any "length is the strongest predictor" statement in the deep dive should be withdrawn.

## Pass criteria
- Row ordering confirmed, and **corrected**: ✅
- One TIME number reproduced within about 2%: ✅, within 0.1%, for 14 pattern cells and 5 overall cells
- Pilot recomputed with MASE and CRPS: ✅
