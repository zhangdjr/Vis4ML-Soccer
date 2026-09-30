# D5 Pre-registration: hypotheses, analysis plan, and kill criteria

**Status:** written and committed **2026-09-29 23:05 EDT (2026-09-30 UTC), before** any held-out (NeurIPS22 Public-Test) data was analysed. The git commit timestamp is the record. Changes after that date go in the "Amendments" section at the bottom, dated, with a reason. Earlier text is never silently edited.

**Why pre-register?** Writing down in advance what counts as support or failure makes a positive result credible and a negative result publishable. It also stops us from choosing, after the fact, the analysis that looks best.

**What is already known (exploratory; this does NOT count as confirmation):** spikes A1 and B1 on LIVECell test (in-distribution; 40 images, 10,292 cells). Per-cell error κ:

| Pair | κ |
|---|---|
| seeds | 0.92 |
| checkpoint variant | 0.88 |
| cyto3 vs `livecell_cp3` | 0.84 |
| Cellpose-SAM vs cyto3 | 0.79 |
| micro-SAM vs cyto3 | 0.61 |
| Cellpose-SAM vs micro-SAM | 0.58 |

The hypotheses below were **generated** from that data. The **confirmatory** tests are on held-out data.

---

## 1. Data, units, and exclusions (fixed now)

- **Confirmatory set:** NeurIPS22 CellSeg **Public-Test**, the 50 labeled images, excluding the 2 whole-slide images. Held out for all models (see `D5_onboarding.md` §6.2).
- **Secondary set:** LIVECell test, 100–200 images stratified by cell type. In-distribution; reported but not confirmatory.
- **Unit of analysis:** a GT cell. **Unit of resampling:** an image, via a 1,000× cluster bootstrap for all CIs.
- **Matching:**
  - Hungarian matching, IoU **> 0.5** as primary.
  - Sensitivity analyses at IoU > 0.3 and > 0.7.
  - Error indicator e_m(c) = 1 if model m has no match for GT cell c.
- **Exclusions, fixed now:**
  - GT cells < 20 px area (annotation noise).
  - Images where a model predicts 0 cells: kept, and flagged. Never silently dropped.
  - Any model with error rate > 0.6 on a dataset is **excluded from κ comparisons** for that dataset. Its κ_max is too small to be interpretable. It is reported descriptively only.
- **Model settings, fixed now:**
  - Defaults, except where a model requires a diameter. For those, run **both** the auto-estimated diameter and a **GT-median diameter per image** (control for shared preprocessing, §4 K3).
  - micro-SAM **AIS** is the primary mode. AMG thresholds are tuned on LIVECell **train** only, never on test data.

## 2. Primary hypotheses (confirmatory, on Public-Test)

**H1: family > shared encoder.**
- Claim: κ(Cellpose-SAM, cyto3) − κ(Cellpose-SAM, micro-SAM) > 0.
- **Supported** if the 95% bootstrap CI of the difference excludes 0 **and** the same sign holds for κ/κ_max.
- **Falsified** if the CI includes 0, or the sign reverses.

**H2: seeds are most consistent (hierarchy top).**
- Claim: κ(seed pairs) > κ(every cross-model pair).
- Supported if the CI of each difference excludes 0.

**H3: agreement-QC improves as relatedness falls.**
- Claim: across all model pairs that pass the exclusions, the Spearman correlation between pair κ and within-image QC AUROC is **negative**.
- Supported if the bootstrap CI of ρ excludes 0.

**H4: agreement beats the baselines as a QC signal.**
- Target model: Cellpose-SAM. Signal: its agreement with the best **cross-family** reference.
- Compare its within-image AUROC for detecting Cellpose-SAM errors against:
  - (a) Cellpose-SAM flow error (run with `flow_threshold=0`);
  - (b) an **attribute-only** logistic model (area, local density, touching fraction, contrast), fit on LIVECell train.
- **Supported** if agreement's AUROC ≥ best baseline + 0.05, with the CI of the difference excluding 0.
- **Falsified** if agreement's AUROC ≤ attribute-only + 0.02.

## 3. Secondary hypotheses (reported, lower evidential weight)

- **H5: decoding drives merge/split sharing.**
  - Claim: for merge/split errors only, κ(same-decoding pairs) > κ(different-decoding pairs).
  - Same-decoding pairs: Cellpose-SAM vs cyto3 (flows).
  - Different-decoding pairs: micro-SAM AIS vs AMG (same encoder), and Cellpose-SAM vs micro-SAM.
- **H6: data vs architecture.** On own-trained U-Nets: κ(same data, different seed) > κ(different data subset, same architecture). Also report where the "different data" level falls relative to the cross-architecture levels.
- **H7: silent failures.** Silent-failure rate (both wrong, and agreeing at IoU > 0.5) falls as relatedness falls. Report the rate per 1,000 cells, with CIs.
- **H8: label errors (exploratory).** Of about 50–100 adjudicated silent failures, report the share judged to be GT label errors, with a Wilson CI.

## 4. Kill tests: what could make D5 fail later (pre-mortem), and how we check now

| # | Failure mode (oversight) | Test | Kill / reframe rule |
|---|---|---|---|
| **K1** | Result doesn't replicate out of distribution | H1–H3 on Public-Test | If H1 **and** H3 are both falsified on Public-Test: **drop the hierarchy headline**. Fall back to the floor (error taxonomy + QC comparison, still gradeable), and report "in-distribution-only" as a finding |
| **K2** | **Shared errors reflect cell difficulty, not relatedness.** Hard or mislabeled cells defeat every model | **Difficulty-controlled κ**: fit an attribute-only error model per model, bin cells into deciles of predicted difficulty, and compute κ *within* bins (a stratified Cohen's κ). Also run a permutation null that shuffles errors within difficulty bins | If the H1 difference **disappears** (CI includes 0) after difficulty control: the story becomes "shared errors = shared hard cells" (still reportable, but reframed). Relatedness then no longer headlines |
| **K3** | **Shared preprocessing** (the auto-diameter estimate) creates the family effect | Rerun cyto3 / `livecell_cp3` with GT-median diameters | If κ(Cellpose-SAM, cyto3) drops to within the CI of κ(Cellpose-SAM, micro-SAM): the "family" effect was partly a preprocessing artifact. Reframe, and note it |
| **K4** | The **accuracy gap** (micro-SAM is less accurate) produces the κ ordering | κ/κ_max; plus an **accuracy-matched** analysis restricted to images or cell types where both models' accuracies are within 0.05 | If H1 fails on the accuracy-matched subset: report H1 as accuracy-confounded |
| **K5** | Threshold / single-cell-type artifact | IoU sweep (0.3 / 0.5 / 0.7); per-cell-type (LIVECell) or per-modality (NeurIPS22) κ; leave-one-type-out | If the H1 sign flips at another threshold, or is driven by a single type or modality: report it as fragile |
| **K6** | The QC gain is statistically real but **practically tiny** | Simulated inspection: errors found in the top 5% of cells, with CIs, per reference choice | If the cross-family advantage is < 2 percentage points of recall at 5% inspected: downplay the practical claim |
| **K7** | **Scoop** | Manual arXiv / Scholar check before Oct 20 and Nov 3 ("error consistency" + segmentation; citers of RBQE, Gontijo-Lopes, BISCUIT) | If a paper does per-object κ across a relatedness hierarchy for cell segmentation: pivot to the QC/triage + adjudication angle |

**The honest floor.** Even if K1 and K2 both go against the hypotheses, the project still delivers:
- a per-cell error taxonomy of 4–6 generalist models;
- a validated comparison of GT-free QC signals on held-out data;
- a triage and adjudication tool.

That is an A-gradeable course project and a reportable negative result about agreement-based QC. What changes is the paper story, not the course grade.

## 5. Multiple comparisons
- H1–H4 are the confirmatory family: **Holm correction** across these four.
- H5–H8 and K-tests are reported with uncorrected CIs and labelled exploratory.

## 6. What will not be changed after seeing held-out data
The confirmatory dataset, the IoU > 0.5 primary threshold, the exclusion rules, the H1–H4 definitions and thresholds, the target model (Cellpose-SAM), and the bootstrap unit (the image).

---

## Amendments
*(Dated entries only. Each gives what changed, why, and whether it was made before or after seeing held-out results.)*

### Amendment 1: operational definitions (2026-09-29, **before** any Public-Test model output existed)
Written by the round-3 cluster session while the Public-Test images were still downloading. No model had been run on Public-Test at commit time. These entries fill gaps in §1–§4 and do not change any confirmatory threshold.
1. **GT format.**
   - LIVECell GT is painted into label images, large cells first, the same format as Public-Test.
   - B1 used overlapping polygon masks; the E40 overlap is used to check that this change makes no material difference.
   - The < 20 px exclusion applies to both datasets.
2. **Input preprocessing** (identical for all models):
   - Each channel is scaled to uint8 between its 1st and 99.8th percentiles. A 3-channel image with identical channels is treated as 2-D.
   - Cellpose-SAM and micro-SAM receive RGB when present. cyto3, `livecell_cp3`, CellSAM and our own U-Nets receive the channel mean, since they are grayscale-default (`channels=[0,0]`).
   - micro-SAM runs tiled (tile 1024, halo 256) on images with a side over 1536 px.
3. **Model roster.**
   - Primary models for κ: `cpsam`, `cpsam_ft_s1–3`, `cyto3` (auto diameter), `livecell_cp3` (given cyto3's auto diameter), `msam_ais`, `cellsam`.
   - Used only in their named tests: GT-median-diameter variants (K3), `msam_amg` (H5), and `cpsam` with `flow_threshold=0` (QC signals).
   - Error rate > 0.6 on a dataset excludes a model from that dataset's κ comparisons (§1).
   - **If a model needed for H1 is excluded, H1 counts as "not testable", which K1 treats as falsified.**
4. **H2.**
   - Cross-model pairs are pairs of models from different releases. Seed–seed pairs, `cpsam`-vs-its-fine-tune pairs, and setting variants of the same model are not cross-model pairs.
   - Supported if, for **every** seed pair, the CI of κ(seed pair) − κ(cross pair) excludes 0 for every cross pair.
   - p for Holm is the largest of the individual p (intersection–union).
5. **H3.** Pair QC AUROC is the mean of the two directional within-image AUROCs, meaning agreement ranks model a's errors and model b's errors. Pairs are all pairs of primary models that pass the exclusion. The statistic is Spearman ρ with the image bootstrap. The directional version is reported as secondary.
6. **H4.**
   - The cross-family reference for Cellpose-SAM is chosen by its AUROC on **LIVECell (LC200)**, never on Public-Test. Cross-family means a non-Cellpose model that passes the exclusion.
   - Flow-error baseline: whichever of two variants has the higher Public-Test AUROC. This is deliberately generous to the baseline. The variants are:
     - (i) the flow error of the default-output instance;
     - (ii) the flow error of the best-overlapping instance in the `flow_threshold=0` output.
   - A GT cell where the signal's model has no overlapping instance gets that signal's most-suspicious value. This applies to agreement and flow error alike.
   - Attribute-only baseline: logistic regression on log area, local density, touching fraction and contrast, fit on Cellpose-SAM's errors on LIVECell train subset A (200 images).
   - Also reported, as sensitivities:
     - (a) an attribute model cross-fitted on Public-Test (5-fold, grouped by image);
     - (b) AUROC restricted to cells where Cellpose-SAM has an overlapping instance. This is the deployable QC case.
   - Holm p-values come from the bootstrap: p = 2·min(P(Δ* ≤ 0), P(Δ* ≥ 0)), with a floor of 1/B. For H4, Δ = agreement − best baseline, and the +0.05 point-estimate requirement still applies.
7. **Cell attributes** (from GT and the preprocessed grayscale image):
   - area and equivalent diameter d;
   - local density: the number of other GT centroids within 2d;
   - touching fraction: the share of boundary pixels 4-adjacent to another GT cell;
   - contrast: |mean inside − mean of a 3-px outer ring| / (ring std + 1).
8. **C3 / K2.**
   - Per model, an attribute-only logistic error model, cross-fitted with 5-fold image-grouped folds on each test set. The pair's difficulty is the mean of the two models' predictions, binned into deciles.
   - **Primary: the pooled stratified κ** = Σ_s n_s(p_o,s − p_e,s) / Σ_s n_s(1 − p_e,s), where p_o,s and p_e,s are the observed and chance agreement within bin s. Also reported: the size-weighted mean of within-bin κ.
   - Null: each model's error vector permuted within bins, 1,000×.
   - Exploratory: strata = image, and strata = image × difficulty tercile.
9. **K4 accuracy-matched subset:** images where |acc(cpsam) − acc(msam_ais)| ≤ 0.05 **and** |acc(cpsam) − acc(cyto3)| ≤ 0.05.
10. **H5 error types.** From the IoU > 0.5 taxonomy:
    - miss indicator: status = miss;
    - merge/split indicator: status ∈ {merge, split};
    - κ is computed over all cells.
11. **K6 triage.** Rank all GT cells in a dataset by −agreement with the chosen reference, breaking ties at random. Report the recall of Cellpose-SAM errors in the top 5%, with an image-bootstrap CI.
