# D1: Exploratory re-analysis of round-3 data with the round-4 measures (dry run of R1)

**Status: exploratory.**
- The data are round-3 NeurIPS22 Public-Test (PT, a held-out *test split*, 50 images, 6,040 GT cells) and LIVECell LC200 (194 images, 53,034 GT cells). Both were already used in round 3.
- No new inference was run for D1.
- N2 (200 fresh LIVECell test images, round 4) is shown alongside as an in-distribution check. It is not confirmatory for R1; N1 is.
- Code: `~/vis4ml_spikes/work/r4/r4_stats.py`. Tables: `spike_results/r4_tables/{pt,lc200}_{pairs,directed}.csv` and `*_r4.json` (aggregate statistics only). Figure: `fig_reference_tradeoff_round3.png`.
- Throughout, "Cellpose-SAM" / `cpsam` means **v2** (cellpose 4.2.1.1 default; see PREREG Amendment 5).

## 1. log-OR and Yule's Q for all pairs (FACT)
Full tables with 95% image-bootstrap CIs are in `r4_tables/*_pairs.csv`. Key pairs:

| Pair | PT κ | PT log-OR [CI] | PT OR | PT Q | LC200 κ | LC200 log-OR [CI] | LC200 OR | LC200 Q |
|---|---|---|---|---|---|---|---|---|
| cpsam vs FT seeds (mean of 3) | 0.84 | 6.57 | ~720 | 0.997 | 0.87 | 5.61 | ~270 | 0.993 |
| FT seed vs FT seed (mean of 3) | 0.92 | 8.04 | ~3,100 | 0.999 | 0.91 | 6.50 | ~670 | 0.997 |
| **cpsam vs cyto3** | 0.333 | **2.82** [2.44, 3.28] | **16.7** | 0.887 | 0.782 | **4.52** [4.37, 4.68] | **91.6** | 0.978 |
| **cpsam vs micro-SAM `vit_b_lm`** | 0.169 | **1.72** [1.29, 2.14] | **5.6** | 0.697 | 0.519 | **3.12** [2.93, 3.35] | **22.7** | 0.916 |
| cpsam vs CellSAM | 0.127 | 1.73 [1.30, 2.18] | 5.7 | 0.700 | excl. | – | – | – |
| micro-SAM vs CellSAM | 0.390 | 2.39 [2.04, 2.83] | 10.9 | 0.832 | excl. | – | – | – |
| cyto3 vs micro-SAM | 0.158 | 1.13 [0.77, 1.59] | 3.1 | 0.511 | 0.570 | 3.32 [3.14, 3.53] | 27.7 | 0.930 |
| cpsam vs livecell_cp3 | 0.024 | 0.56 [−0.07, 1.08] | 1.7 | 0.271 | 0.793 | 4.61 [4.43, 4.81] | 100.5 | 0.980 |

- **The reviewer's odds-ratio estimates check out.** They estimated "about 17 vs 6 (PT), 95 vs 23 (LC200)" from rounded rates. The exact values are **16.7 vs 5.6 (PT)** and **91.6 vs 22.7 (LC200)**.
- **H1 under log-OR** (log-OR(cpsam, cyto3) − log-OR(cpsam, micro-SAM)):
  - PT: **+1.10 [0.56, 1.70]**, one-sided p = 0.001;
  - LC200: **+1.40 [1.27, 1.51]**.

  The family effect survives the change to a margin-free measure on both datasets. This is the R2 contrast, and it holds on N2 as well: +1.36 [1.25, 1.44].
- **But the two robustness checks split by dataset:**
  - **Accuracy-matched subset** (K4 definition):
    - PT: −0.18 [−0.83, 2.93] on only 20 images. The PT effect disappears when accuracy is matched, as in round 3.
    - LC200 and N2: it survives, +1.18 [0.92, 1.45] and +0.87 [0.68, 1.08].
  - **R8 external-consensus strata** (image × number of other models missing the cell):
    - PT: Mantel–Haenszel log-OR difference **+0.06 [−0.40, 0.64]**. The effect **vanishes**, although the stratified κ difference stays positive (+0.065 [0.002, 0.136]).
    - LC200: +0.92 [0.74, 1.09].

  SYNTHESIS: on held-out data the "family" gap is largely a shared-difficulty effect. In-distribution it is not. **This is the main risk for R2 on N1.**

## 2. Reference descriptors for every directed pair (FACT)
All 56 (PT) and 42 (LC200) directed pairs are in `*_directed.csv`, with columns e_r, o, f, κ, log-OR, within-image AUROC and recall@5%, each with a CI. Target = Cellpose-SAM:

| Reference | PT e_r | PT o | PT f | PT AUROC | PT recall@5% [CI] | LC200 e_r | LC200 o | LC200 f | LC200 AUROC | LC200 recall@5% [CI] |
|---|---|---|---|---|---|---|---|---|---|---|
| FT seed 1 | 0.048 | 0.82 | 0.007 | 0.79 | 0.55 [0.49, 0.60] | 0.254 | 0.90 | 0.033 | 0.72 | **0.176** [0.158, 0.205] |
| cyto3 | 0.108 | 0.60 | 0.082 | 0.87 | **0.61** [0.51, 0.67] | 0.276 | 0.88 | 0.072 | 0.74 | 0.165 [0.148, 0.191] |
| livecell_cp3 | 0.530 | 0.66 | 0.523 | 0.75 | 0.05 [0.03, 0.09] | 0.268 | 0.87 | 0.064 | 0.74 | 0.166 [0.149, 0.192] |
| micro-SAM ViT-B | 0.148 | **0.46** | 0.132 | **0.90** | 0.55 [0.47, 0.63] | 0.417 | 0.89 | 0.257 | 0.74 | 0.135 [0.118, 0.162] |
| CellSAM | 0.279 | 0.66 | 0.258 | 0.83 | 0.45 [0.31, 0.58] | excl. | | | | |

- The ceiling on recall@5% is min(1, 0.05 / e_t): **1.00 on PT, 0.197 on LC200.**
- On LC200 every reference sits between 0.135 and 0.176, close to that ceiling.

## 3. Dry run of R1 (FACT unless labelled)
Confirmatory R1 compares κ alone against {f, o}. Both are linear predictors of recall@5% across all directed pairs, scored by leave-one-target-out cross-validated Spearman ρ.

| Variant | PT: ρ(κ) → ρ(f,o); Δ [CI] | LC200 | N2 (in-distribution) |
|---|---|---|---|
| **R1 confirmatory (raw recall@5%)** | 0.58 → 0.80; **+0.22 [0.05, 0.31] "supported"** | 0.91 → 0.89; **−0.02 [−0.04, 0.04] "falsified"** | 0.83 → 0.89; **+0.05 [0.04, 0.07] "inconclusive"** |
| AUROC (secondary) | 0.09 → 0.92; +0.84 [0.71, 0.95] | 0.59 → 0.95; +0.36 [0.27, 0.52] | 0.65 → 0.94; +0.29 [0.20, 0.35] |
| R1-norm: recall / ceiling (Amendment 6) | −0.55 → 0.87; +1.42 [1.09, 1.56] | −0.07 → 0.92; +0.99 [0.39, 1.07] | −0.18 → 0.88; +1.07 [1.01, 1.11] |
| R1-within, recall (Amendment 6) | 0.29 → 0.21; −0.09 [−0.13, 0.12] | 0.68 → 0.62; −0.06 [−0.11, 0.00] | 0.59 → 0.57; −0.02 [−0.05, −0.01] |
| R1-within, AUROC (Amendment 6) | 0.04 → 0.77; +0.73 [0.51, 0.88] | 0.52 → 0.80; +0.29 [0.09, 0.36] | 0.64 → 0.81; +0.17 [0.06, 0.24] |

Full-sample fits: coefficients on (f, o).

| Outcome | PT (f, o) | LC200 (f, o) |
|---|---|---|
| recall@5% | (−0.67, +0.42) | (−0.15, +0.16) |
| AUROC | (−0.21, −0.20) | (+0.05, −0.35) |

**Reading (SYNTHESIS):**
1. **Confirmatory R1 is fragile.** On the three datasets it comes out supported, falsified and inconclusive. Pooled across targets, raw recall@5% mostly tracks each **target's** error rate through the 0.05 / e_t ceiling, and κ tracks it too. This is why Amendment 6 was added *before* N1.
2. **{f, o} predicts AUROC far better than κ, everywhere, including within a target.** For "how well does agreement rank a target's errors", accuracy and independence together explain much more than κ. In the AUROC fits the shared-error term o carries a negative weight, as the hypothesis predicts.
3. **For top-5% triage within a target, κ ranks references as well as {f, o}.** R1-within for recall is ≤ 0 on all three datasets.
   - The recall fit puts a negative weight on the false-alarm source f and, confusingly, a *positive* weight on o.
   - Mechanism (SYNTHESIS): the top 5% of cells ranked by disagreement fill up with the reference's false alarms whenever f · (1 − e_t) is large compared with 0.05. When f is small (the FT seeds), shared errors cost little at a 5% budget, because the target's errors that the reference also misses are simply not found, and there are plenty of others.
   - So **at a fixed inspection budget, reference accuracy (low f) matters more than independence (low o).** For threshold-free ranking (AUROC), independence matters.
4. **Prediction for N1 (SPECULATION, not a change to Part B):**
   - Confirmatory R1 will be decided by how far target error rates spread on N1, so a positive result is plausible but not robust.
   - AUROC-R1 will very likely be supported. R1-within for recall will likely be ≤ 0.
   - If so, the honest headline is: "accuracy and independence explain ranking quality; for budgeted triage, pick the most accurate reference".

## 4. Notes
- `livecell_cp3` on PT (e_r = 0.53) is a near-useless reference: recall 0.05, which is random. It remains in the roster because its error rate is below the 0.6 exclusion rule.
- CellSAM is excluded on LIVECell (error > 0.6 on LC200 and on N2, where its accuracy is 0.28). LIVECell was held out from its training.
- Nothing in Part B was changed because of D1. The secondary R1 variants were added in dated Amendment 6, before N1 was chosen, and the confirmatory R1 is untouched.
