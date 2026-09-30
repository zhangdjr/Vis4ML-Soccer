# Spike C2: robustness of H1 (K3, K4, K5) and the practical-size check (K6)

**Data and code:** as in C1 (`work/c1/c1_stats.py`, `out/{pt,lc200}/hypotheses.json`). All analyses follow PREREG §4 and Amendment 1. Public-Test is confirmatory; LC200 is secondary.
**Verdict:**

| Test | Result | Consequence |
|---|---|---|
| K3 | **passed** | |
| K5 | **passed** by its rule, but heterogeneous | |
| **K4** | **triggered** on Public-Test | report H1 as **accuracy-confounded** |
| **K6** | **triggered** on both datasets | downplay the practical "use a cross-family reference" claim |

## K3: shared preprocessing (the auto-diameter estimate)

| | Public-Test | LC200 |
|---|---|---|
| κ(cpsam, cyto3), auto diameter | 0.333 | 0.782 |
| κ(cpsam, cyto3), **GT-median diameter** | **0.349** [0.294, 0.396] | 0.785 |
| κ(cpsam, micro-SAM), its upper CI | 0.230 | 0.556 |
| H1 difference with GT diameter | +0.180 [0.103, 0.253] | +0.266 [0.238, 0.293] |

**Passed.** Removing the shared size estimate *raises* κ slightly. So the family effect is not a preprocessing artifact (FACT).

A side note: κ(cyto3 auto, cyto3 GT-diameter) is 0.81 on Public-Test and 0.88 on LC200. The diameter setting alone changes about 1 in 8 errors on held-out data.

## K4: the accuracy gap

**κ/κ_max** (κ divided by the maximum κ possible given the two error rates):

| Pair | Public-Test | LC200 |
|---|---|---|
| cpsam vs cyto3 | 0.55 | 0.83 |
| cpsam vs micro-SAM | 0.37 | 0.81 |
| **Difference** | **+0.19 [0.06, 0.32]** | **+0.02 [0.002, 0.044]** |

- On LIVECell, the whole raw-κ gap (0.26) is essentially micro-SAM's extra errors (accuracy 0.58 vs 0.72).
- On Public-Test, the normalized gap survives.

**Accuracy-matched subset** (images where |acc(cpsam) − acc(micro-SAM)| ≤ 0.05 **and** |acc(cpsam) − acc(cyto3)| ≤ 0.05):

| | Public-Test | LC200 |
|---|---|---|
| Images / cells | 20 / 2,279 | 62 / 12,598 |
| κ(cpsam, cyto3) | 0.269 | 0.815 |
| κ(cpsam, micro-SAM) | 0.300 | 0.691 |
| **H1 difference** | **−0.031 [−0.133, 0.213]** | **+0.124 [0.097, 0.151]** |
| κ/κ_max difference | +0.092 [−0.079, 0.708] | +0.092 [0.046, 0.132] |

**K4 is triggered on Public-Test:** H1 fails on the accuracy-matched subset. The pre-registered rule is to **report H1 as accuracy-confounded.**
- Caveat: only 20 images, which are the easy ones where all three models are near ceiling. The CI is very wide, so this is "no evidence for H1", not "evidence against".
- LC200 passes.

## K5: threshold, modality/cell-type and leave-one-group-out

**IoU sweep, H1 difference:**

| Threshold | Public-Test | LC200 |
|---|---|---|
| > 0.3 | +0.095 [0.029, 0.156] | +0.237 |
| > 0.5 (primary) | +0.164 | +0.262 |
| > 0.7 | +0.147 [0.051, 0.247] | +0.244 |

**No sign flip.**

**Per modality on Public-Test** (H1 diff [CI]; κ/κ_max diff; micro-SAM accuracy in that group):

| Group | Images | H1 diff | κ/κ_max diff | micro-SAM acc |
|---|---|---|---|---|
| phase-contrast cultured (LIVECell-like) | 5 | **+0.39 [0.30, 0.45]** | −0.16 [−0.23, −0.01] | 0.65 |
| phase-contrast bacteria | 8 | **+0.20 [0.04, 0.41]** | +0.47 [0.09, 0.82] | 0.64 |
| fluorescence | 12 | +0.04 [−0.18, 0.27] | +0.45 [−0.01, 0.90] | 0.99 |
| stained brightfield | 18 | +0.02 [−0.08, 0.11] | +0.17 [0.004, 0.29] | 0.89 |
| round/yeast-like | 7 | −0.05 [−0.30, 0.10] | 0.00 [−0.50, 0.00] | 0.99 |

- **Leave-one-modality-out:** every H1 difference stays positive with the CI above 0 (minimum +0.114 [0.023, 0.199], when bacteria are left out).
- **Per LIVECell cell type:** raw diff positive in 8/8 types (all CIs above 0). κ/κ_max diff is mixed: 3 clearly positive (A172, SH-SY5Y, SKOV3), 1 clearly negative (BV2 −0.05), 4 ≈ 0.
- LC200 leave-one-type-out: all CIs above 0.

**K5 is not triggered by its rule:** no sign flip, and no single group drives it (the LOGO check).

SYNTHESIS: the raw effect is **concentrated in the two modalities where micro-SAM is much weaker** (accuracy 0.64–0.65). Where micro-SAM and Cellpose-SAM are about equally accurate (brightfield), the raw-κ difference is ~0. That is the same accuracy story as K4.

## K6: is the QC gain practically meaningful? (simulated inspection)

Setup: rank all GT cells by disagreement between Cellpose-SAM and a reference, inspect the top 5% of cells, and count the share of Cellpose-SAM's errors found.

| Reference | Public-Test recall@5% [CI] | LC200 recall@5% |
|---|---|---|
| cyto3 (same family) | **0.61** [0.53, 0.68] | 0.165 |
| livecell_cp3 (same family) | 0.05 | 0.165 |
| FT seeds | 0.53–0.55 | 0.17–0.18 |
| **micro-SAM (cross-family)** | **0.55** [0.47, 0.64] | 0.138 |
| CellSAM (cross-family) | 0.45 | excluded |
| Cellpose-SAM's own flow error | 0.48 | **0.19** |
| random / maximum possible | 0.05 / 1.00 | 0.05 / 0.20 |

- **Cross-family advantage: −5.7 pp on Public-Test and −2.8 pp on LC200.** The rule threshold is < 2 pp, so **K6 is triggered.**
- For triage of the strongest model's errors, a same-family reference (cyto3) is at least as good as a cross-family one.
- Every agreement signal is 9–12× better than random on Public-Test, which is itself a useful floor result.
- SYNTHESIS: AUROC (C1, H4) and top-5% recall disagree about the best reference. AUROC favors micro-SAM (0.90 vs 0.87); the top of the ranking favors cyto3. For a triage *tool*, the top of the ranking is what matters. That argues for showing several references side by side in the VA tool, not for a single "best" one.
