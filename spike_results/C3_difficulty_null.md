# Spike C3: difficulty-controlled null (K2), i.e. are shared errors just shared hard cells?

**Verdict: K2 passed.** The H1 difference survives every difficulty control we tried on both datasets. Caveat: on Public-Test the *attribute-based* difficulty model is weak, so the stronger evidence there comes from the image-level and external-model controls.

## Method (PREREG §4 K2, Amendment 1 #7–8)
**Per-cell attributes** (from GT and the preprocessed grayscale image):
- log area;
- local density: other GT centroids within 2 equivalent diameters;
- touching fraction: the share of boundary pixels 4-adjacent to another GT cell;
- contrast: |inside − 3-px ring| / (ring std + 1).

**Difficulty model:** per model, an attribute-only logistic error model, **cross-fitted with 5-fold image-grouped folds** on each test set (the brief's second option, so there is no in-sample leakage). A pair's difficulty is the mean of the two models' predicted error probabilities.

**How well the attributes predict errors** (within-image AUROC of the cross-fitted model):

| | LC200 | Public-Test |
|---|---|---|
| All models | 0.74–0.79 | |
| Cellpose models | | **0.50–0.57** |
| micro-SAM | | 0.72 |
| CellSAM | | 0.67 |

So on held-out data, cell size, density, touching and contrast barely explain *which* cells Cellpose models miss (FACT).

**Controls:**
- (a) **Pooled stratified κ** over difficulty deciles: Σ_s n_s(p_o − p_e) / Σ_s n_s(1 − p_e). Chance agreement is computed within each difficulty bin.
- (b) Stratified by **image**, and by **image × difficulty tercile**.
- (c) A **permutation null:** one model's errors shuffled within difficulty deciles, 1,000×.
- (d) *Post hoc, exploratory:* strata = **image × CellSAM error**. CellSAM is a third, independently built model and serves as an external proxy for "hard for everyone".

## Results: the H1 pairs

| Control | cpsam vs cyto3 (PT) | cpsam vs micro-SAM (PT) | **H1 diff, PT [95% CI]** | **H1 diff, LC200** |
|---|---|---|---|---|
| none (raw κ) | 0.333 | 0.169 | +0.164 [0.086, 0.245] | +0.262 |
| difficulty deciles (primary K2) | 0.317 | 0.151 | **+0.166 [0.088, 0.247]** | +0.297 [0.265, 0.328] |
| image | 0.296 | 0.161 | +0.135 [0.061, 0.211] | +0.316 [0.286, 0.344] |
| image × difficulty tercile | 0.295 | 0.124 | +0.172 [0.112, 0.238] | +0.344 [0.313, 0.371] |
| image × CellSAM error (exploratory) | 0.207 | 0.107 | +0.099 [0.020, 0.188] | +0.339 [0.307, 0.365] |

**Permutation null** (errors shuffled within difficulty deciles):

| | Public-Test | LC200 |
|---|---|---|
| Null mean κ, all pairs | 0.02 | 0.06–0.11 |
| Observed κ (cpsam vs cyto3 / vs micro-SAM) | 0.33 / 0.17 | 0.78 / 0.52 |
| p | 0.001 (all pairs) | 0.001 |

The attributes explain almost none of the observed consistency.

## Interpretation (SYNTHESIS)
- **"Shared errors = shared hard cells" is not what drives the H1 ordering.** Within the same image, the same difficulty band, and even among cells that an unrelated model gets right, Cellpose-SAM still shares more errors with cyto3 than with micro-SAM. On Public-Test, among the 4,358 cells CellSAM gets right, κ is 0.18 vs 0.06.
- **But hard cells exist, and they matter more in-distribution.**
  - On LIVECell, **97%** of Cellpose-SAM's errors are also CellSAM errors; on Public-Test, 66%.
  - Stratifying by image lowers the LIVECell κ values somewhat, for example cpsam–micro-SAM from 0.52 to 0.43.
  - The large in-distribution κ values are partly "every model fails on the same crowded LIVECell images".
- **The remaining threat is accuracy (C2, K4), not difficulty.** Stratified κ still depends on the two models' marginal error rates within each stratum, so K2 and K4 are complementary, not redundant.
- **Hidden oversight to fix in the full study.** The attribute model is weak on Public-Test. "Difficulty" there needs richer features (image-level SNR, modality, cell morphology), or a leave-pair-out consensus of *several* external models. The CellSAM control is one step in that direction.
