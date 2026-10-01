# D3: Confirmatory tests on N1 (R1–R4 with Holm) and secondary R5–R8

**N1 = mCellSeg**, frozen in D0 (commit 2a758ce): 198 images, 15,975 GT cells, label-free DIC/bright-field, HEK-293T and HUVEC.
- Code: `work/r4/r4_stats.py`, `r4_R4.py`, `r4_holm.py`.
- Outputs: `work/r4/out/{n1,n1s,n2}/`, copied to `spike_results/r4_tables/`.
- "Cellpose-SAM" = v2 (Amendment 5).
- Labels: FACT = computed here; SYNTHESIS = interpretation; SPECULATION = unverified.

## 0. Headline (FACT)

| | Pre-registered N1 (native scale) | N1-rescaled (Amendment 7: post-hoc design, outcome-blind) | After Holm (m = 4) |
|---|---|---|---|
| **R1** {f, o} beats κ for recall@5% | **not testable**: only 2 targets pass the > 0.6 error rule | Δρ = **+0.05 [−0.02, 0.17]**, below the 0.10 bar, so **inconclusive** | not supported |
| **R2** family effect is margin-free | **not testable**: Cellpose-SAM excluded (error 0.652) | **−0.06 [−0.28, 0.15]**, so **falsified** | falsified |
| **R3** shared SAM ViT-L raises dependence | **not testable** | **not testable**: µSAM ViT-L excluded (error 0.604) | not testable |
| **R4** shift decorrelates cross-architecture > seeds (N3) | **−0.57 [−0.72, −0.43]**, so **falsified** (D4) | (same; R4 does not use N1) | falsified |

**No R1–R4 hypothesis is supported.** The B5 kill rules apply as follows:
- **R1:** not falsified, but not supported either. On N1-rescaled it is inconclusive, and on round-3 data and N2 it was mixed (D1).
  - The rule "R1 falsified → drop the headline" is not triggered.
  - But the evidence does not carry "what makes a good reference" as a confirmed headline (see D6).
- **R2 falsified on N1** → "the family effect was test-set-specific". It held on LIVECell (LC200, N2) and NeurIPS22 Public-Test (κ, not accuracy-matched). It does not hold on a new dataset that neither model trained on.
- **R3 and R4** are mechanism claims. R3 is untestable and R4 is falsified, so both mechanism claims are removed.

## 1. What happened on native-scale N1 (FACT)
Accuracy (share of GT cells matched at IoU > 0.5):

| Model | Accuracy | Excluded (error > 0.6)? |
|---|---|---|
| Cellpose-SAM v2 | 0.349 | yes |
| Cellpose-SAM v1 | 0.357 | yes |
| FT seeds | 0.351–0.388 | yes |
| cyto3 | 0.239 | yes |
| livecell_cp3 | 0.104 | yes |
| CellSAM | 0.333 | yes |
| CellposeDINO-L | 0.412 | yes (exploratory) |
| µSAM ViT-B | 0.482 | no |
| µSAM ViT-L | 0.527 | no |

- **Cause: cell scale.** The median GT diameter is 132 px (range 50–327).
  - Cellpose-SAM accuracy is 0.53 in images with median diameter ≤ 120 px, but 0.17 above it.
  - On the 40× DIC images it is 0.02.
  - It finds 2–9 objects where there are 30–100 cells.
- D0 §7 named exactly this risk. Following D0, no rescaling rule was added before the run.
- **The pre-registered confirmatory test therefore did not happen.** That is a design failure, recorded as such: the dataset choice put the main model outside its operating range.

## 2. N1-rescaled (Amendment 7)
- **Rule.** Each image is resized by 30 / (GT-median diameter), using area-averaging and the same rule for every model. Predictions are upsampled back to the original grid, and matching uses the original GT.
- **Timing.** The rule was registered after the native-scale accuracies were seen, but before any dependence or QC statistic involving the excluded models. Results are **post-hoc in design, outcome-blind**, and weaker evidence than a pre-registered test.
- **Accuracy after rescaling:**
  - Cellpose-SAM 0.442, v1 0.471, FT seeds 0.50–0.53, cyto3 0.409, µSAM ViT-B 0.405;
  - excluded: µSAM ViT-L 0.396, CellSAM 0.337, livecell_cp3 0.160.
  - Rescaling helps the Cellpose family and **hurts micro-SAM** (0.48 → 0.40). micro-SAM is the one model whose native-scale accuracy did not depend on cell size.
- **Confirmatory roster** (7 models): Cellpose-SAM, v1, FT seeds 1–3, cyto3, µSAM ViT-B. CellposeDINO-L/B are exploratory.

### R1 (Amendments 5–6 variants)

| Variant | ρ(κ) | ρ(f, o) | Δ [95% CI] | Verdict |
|---|---|---|---|---|
| **Confirmatory: recall@5%, pooled LOTO** | 0.69 | 0.74 | **+0.05 [−0.02, 0.17]**, p = 0.095 | inconclusive |
| AUROC (secondary) | −0.22 | 0.96 | +1.18 [0.94, 1.65] | supported |
| R1-norm: recall / ceiling | 0.15 | 0.94 | +0.79 [0.66, 0.86] | supported |
| R1-within: recall | 0.33 | 0.28 | −0.05 [−0.17, 0.29] | ≤ 0 |
| R1-within: AUROC | −0.18 | 0.84 | +1.03 [0.72, 1.27] | supported |
| With exploratory models (DINO), recall | 0.70 | 0.74 | +0.03 [−0.01, 0.10] | inconclusive |

- **The pattern matches the D1 prediction exactly** (SYNTHESIS).
  - {f, o} explains threshold-free ranking quality (AUROC) far better than κ, both across and within targets.
  - For budgeted top-5% triage it adds nothing over κ.
- The reason is mechanical. Cellpose-SAM's error rate is 0.56, so the recall@5% ceiling is 0.09, and **every reference reaches 0.079–0.089 of a possible 0.090** (88–99% of the ceiling). When the target is this bad, almost any disagreement points at a real error, and reference choice cannot matter at a 5% budget.

### R2 and its robustness (FACT)

| Measure | Value |
|---|---|
| log-OR(Cellpose-SAM, cyto3) | 2.65 [2.39, 2.91] |
| log-OR(Cellpose-SAM, µSAM ViT-B) | 2.70 [2.50, 2.92] |
| **R2 difference** | **−0.06 [−0.28, 0.15]** |
| κ difference | −0.008 [−0.046, 0.025] |
| v1 sensitivity | −0.13 [−0.36, 0.12] |
| Accuracy-matched (17 images) | −0.08 [−0.79, 0.43] |
| **R7 matched operating point** (`cellprob_threshold` 0.5: error 0.593 vs µSAM 0.595, within 0.01) | −0.08 [−0.32, 0.13] |
| R8 external-consensus strata (MH log-OR) | +0.14 [−0.09, 0.37] |

- **Every version is null.** On N1, Cellpose-SAM's errors are no more like cyto3's (same lab, CNN, 9 datasets) than like micro-SAM's (different lab, SAM ViT-B).
- SYNTHESIS: the earlier family effect tracked shared **training data**. Cellpose-SAM and cyto3 both trained on LIVECell, where the gap was largest (LC200 OR 92 vs 23), and both trained on the Cellpose cyto2 images. On data that neither model saw, the gap disappears.

### R3: not testable, plus exploratory indications (FACT; exploratory because µSAM ViT-L is excluded)
- Accuracy-matched subset (24 images): log-OR(Cellpose-SAM, ViT-L) − log-OR(Cellpose-SAM, ViT-B) = **−0.39 [−0.72, −0.10]**.
- At matched operating points (R7): **−0.26 [−0.40, −0.11]**.
- So, if anything, the pair that shares the SAM ViT-L checkpoint is *less* dependent.
- On N2 (in-distribution, both included): +0.10 [0.04, 0.16], but accuracy-matched +0.10 [−0.04, 0.25], and κ/κ_max −0.005.
- **No evidence that a shared encoder checkpoint raises error dependence.**

### R5–R8 (secondary, uncorrected; FACT)
- **R5/R6 as pre-registered: not computable.** The N2-chosen best reference (µSAM ViT-L) is excluded on N1-rescaled.
- **Exploratory fallback:** the next N2-ranked reference that passes on N1 is cyto3, with top-2 = cyto3 + µSAM ViT-B.
  - **R5:** the cross-fitted logistic combination of agreement and flow error reaches AUROC 0.803. That is +0.065 [0.054, 0.079] over agreement but only **+0.007 [−0.001, 0.015] over flow error**, which fails the ≥ 0.02 bar. **Not supported.**
  - Cellpose-SAM's own flow error (`flow_threshold=0` run) alone gives AUROC 0.796. On N2 the picture is the same: combination 0.853 vs flow 0.850.
- **R6:** the mean agreement with cyto3 and µSAM beats the best single reference (cyto3) by +0.048 AUROC [0.038, 0.058] and +0.006 recall@5% [0.003, 0.008]. **Supported** (exploratory fallback). On N2 it is supported as pre-registered: +0.028 AUROC [0.022, 0.033], +0.033 recall@5% [0.026, 0.042].
- **R7:** done (above). The threshold 0.5 matches within 0.01, and R2 stays null.
- **R8:** above; null for R2 and for the exploratory R3.

## 3. Exploratory: what *does* drive error dependence on N1 and N2 (FACT values; SYNTHESIS reading)
log-OR with Cellpose-SAM v2:

| Partner | What changes vs Cellpose-SAM v2 | N2 | N1-rescaled |
|---|---|---|---|
| FT seed (fine-tuned on LIVECell) | weights only (+ LIVECell fine-tune) | 5.42 | 5.13 |
| Cellpose-SAM v1 | retrained version, same recipe | 5.87 | 5.04 |
| **CellposeDINO-L** | **foundation backbone (SAM ViT-L → DINOv3 ViT-L)**, same Cellpose data and objective | **4.95** | **4.55** |
| CellposeDINO-B | backbone and size | 4.51 | 4.14 |
| cyto3 | architecture (CNN), smaller data (9 vs 18+ sets), same objective | 4.53 | 2.65 |
| µSAM ViT-B | lab, objective/decoder, data, backbone size | 3.18 | 2.70 |

- Also: µSAM ViT-B vs ViT-L (same recipe, different encoder size): 4.61 on N2.
- **Swapping the foundation backbone (SAM → DINOv3) inside the same Cellpose recipe keeps errors nearly as dependent as a retrained copy.**
- Changing the **training recipe and data** (cyto3, micro-SAM) lowers dependence much more. The effect is largest on N1, which is new to every model.
- SYNTHESIS: on this evidence the right causal unit is *training data + objective*, not the foundation-model checkpoint. That matches R2's failure, R3's negative indications and R4's falsification.
- This is **exploratory** (CellposeDINO was added after Part B). It is a candidate confirmatory hypothesis for the proposal.
