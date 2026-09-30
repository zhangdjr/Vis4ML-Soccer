# Spike B1: D5 ambitious, a per-cell error-consistency hierarchy (seed → lineage)

**Date:** 2026-09-29 · **Machine:** BC Andromeda. Seeds trained on `gtml` grs001 (3× L40S in parallel); inference on L40S; analysis on CPU (`short`).
**Wall time:** about 1 h 45 min total. Of that, training took 30 min (3 seeds in parallel) and inference 5 min.
**Verdict: PASS (feasible), with a first signal that goes against H-mono.** All three models whose training data we know saw LIVECell train, so this is an in-distribution smoke test. See the caveats.

## What was run

**Env `d5c3`:** a cellpose **3.1.1.3** `--no-deps` overlay (1.9 MB) on top of `d5`, via `PYTHONPATH=envs/d5c3_overlay`.
- It reuses d5's torch 2.7.1. `cyto3` and `livecell_cp3` both load and run on GPU.
- FACT: **`livecell_cp3` is still downloadable** (cellpose.org model URL returns 200; 25 MB).
- Cellpose 4 remains the default in d5.

**Data** (`work/b1/b1_data.py`): LIVECell train, **25 images per cell type → 200 images**, labels painted from the COCO polygons.
- Test set **E40** = A1's 8 images (**E8**, 2,006 GT cells) plus 4 random test images per cell type, **10,292 GT cells** in total.
- Images were range-read from `images.zip`; about 230 MB on disk.

**Seed copies** (`work/b1/b1_train.py`, SLURM array 3074326): Cellpose-SAM fine-tuned with cellpose 4 `train_seg`.
- Settings: 100 epochs, batch 8, lr 1e-5, wd 0.1, 200 images.
- **FACT, a code gotcha:** `train_seg` calls `np.random.seed(iepoch)` every epoch. Without intervention, every "seed" would get identical data order and augmentation. The script offsets it by 1000·seed and sets `torch.manual_seed`.
- So seeds differ in **data order + augmentation** only. All start from the same pretrained `cpsam`, so this is Gontijo-Lopes' L0b / Saxena's "data ordering" randomness.
- **Time per seed: 30.4 min (18.2 s/epoch) on one L40S, 31 GB peak.**
- Train loss barely moved (0.77 → 0.74–0.77), as expected: cpsam already saw LIVECell.

**Inference** (`work/b1/b1_infer.py`, 3074331): 8 models on E40.

| Model | Median s/img | Share of GT cells matched at IoU > 0.5 (E40 / E8) |
|---|---|---|
| cpsam (off-the-shelf) | 0.26 | 0.758 / 0.824 |
| cpsam_ft_s1 / s2 / s3 | 0.26 | 0.755 / 0.750 / 0.752 (E8: 0.826 / 0.817 / 0.821) |
| livecell_cp3 (cellpose 3, dataset-specific) | 0.18 | 0.750 / 0.812 |
| cyto3 (cellpose 3 generalist; diameter auto-estimated) | 0.52 | 0.741 / 0.812 |
| micro-SAM AIS `vit_b_lm` | 0.13 | 0.649 / 0.724 |
| CellSAM `cellsam_general` (zero-shot) | 0.47 | 0.275 / 0.352 |

- `livecell_cp3` was given cyto3's per-image diameter estimate, because it has no size model.

**Analysis** (`work/b1/b1_analyze.py`, 3074335). Outputs: `work/b1/out/{kappa_table,kappa_by_level,per_cell_b1}.csv`.
- **Error:** e_m(c) = 1 if no instance of model m overlaps GT cell c with IoU **> 0.5**. This is strict, which fixes A1's ≥ caveat.
- **κ** is Cohen's (Geirhos error consistency), with **95% CIs from a 1,000× image-level cluster bootstrap**.
- **κ/κ_max** is κ divided by its maximum given the two error rates.
- **Within-image AUROC** measures how well pairwise agreement (IoU between the two models' instances at the cell) ranks model a's errors. Its CI is a bootstrap over images.

## Result: κ by level (E40; E8 in brackets)

| Level | Pairs | Accuracy a / b | κ [95% CI] | κ/κ_max | P(b wrong \| a wrong) | Silent failures /1000 cells | AUROC (err of a) |
|---|---|---|---|---|---|---|---|
| **L0b seed–seed** (cpsam FT) | 3 | 0.75 / 0.75 | **0.92** [0.90–0.93] (E8 0.89) | 0.93 | 0.94 | **166** | 0.71 |
| **L1 checkpoint variant** (cpsam vs its FT) | 3 | 0.76 / 0.75 | **0.88** [0.86–0.90] (0.85) | 0.90 | 0.92 | 150 | 0.72 |
| **L4 same arch, different data** (cyto3 vs livecell_cp3) | 1 | 0.74 / 0.75 | **0.84** [0.82–0.86] (0.83) | 0.86 | 0.87 | 138 | 0.74 |
| **L3 SAM vs non-SAM, same Cellpose lineage** (cpsam or FT vs cyto3 / livecell_cp3) | 8 | 0.75 / 0.74–0.75 | **0.79** [0.75–0.83] (0.75) | 0.79–0.83 | 0.84–0.88 | 117–120 | 0.72–0.77 |
| **L3 SAM vs non-SAM, cross lineage** (micro-SAM vs cyto3 / livecell_cp3) | 2 | 0.65 / 0.74–0.75 | **0.61** [0.56–0.64] (0.56) | 0.77–0.80 | 0.85–0.87 | 74–75 | 0.76–0.77 |
| **L2 shared SAM backbone, cross lineage** (cpsam or FT vs micro-SAM) | 4 | 0.75 / 0.65 | **0.58** [0.52–0.62] (0.52) | 0.76–0.78 | 0.84–0.85 | 67–70 | 0.75–0.78 (err of micro-SAM: 0.83–0.85) |
| CellSAM vs anything | 7 | 0.28 / 0.65–0.76 | 0.20–0.29 | (0.86–0.93, not meaningful: κ_max is tiny) | 0.33–0.47 | 10–22 | 0.58–0.60 (err of CellSAM: 0.91–0.94) |

The code's automatic level labels put "cpsam vs cyto3" and "micro-SAM vs cyto3" in one L3 bin. The table splits them by lineage. Per-pair rows are in `kappa_table.csv`.

## First signal (SYNTHESIS; 40 images from one in-distribution dataset)

1. **The hierarchy's top end behaves as expected.** Seed copies are the most consistent (κ 0.92), then checkpoint variants (0.88). **Seed diversity is small but not zero**, matching Saxena's warning about data-order-only randomness. Even so, about 1 in 6 cells is a *shared, agreed-upon* error between two seeds (166 per 1,000).
2. **H-mono is not supported. The shared SAM encoder does not make errors more alike.**
   - Cellpose-SAM vs micro-SAM (both SAM ViT encoders): κ **0.58**.
   - Cellpose-SAM vs cyto3 (SAM vs U-Net): κ **0.79**. The CIs do not overlap.
   - micro-SAM is equally (un)correlated with the SAM-based Cellpose (0.58) and the non-SAM cyto3 (0.61).
   - After normalizing for accuracy (κ/κ_max), the SAM–SAM pair is still the *least* consistent (0.77 vs 0.83).
   - **What predicts shared errors here is lineage: same group, objective (flow fields), training recipe and data (Cellpose models all saw LIVECell).** The encoder does not predict it.
   - This is a real, reportable deviation from the "component-sharing" intuition. It answers Bankhead's question the other way: models with *different* architectures but overlapping training sets and objectives are the ones that fail together.
3. **Data vs architecture (H-data > H-arch): not supported.** Same-arch/different-data (cyto3 vs livecell_cp3, 0.84) is *more* consistent than cross-arch pairs. Caveat: `livecell_cp3` and `cyto3` both saw LIVECell (cyto3's training set is SECONDHAND; verify), so this is not a clean "different data" pair.
4. **QC consequence.** Agreement-as-QC works better the less correlated the pair is:
   - within-image AUROC of 0.71 for seed pairs, against 0.75–0.78 for cpsam vs micro-SAM;
   - and 0.83–0.85 when cpsam's agreement is used to flag micro-SAM's errors.
   - The silent-failure mass falls from 166 to 67 per 1,000 cells.
   - This is the negative κ–AUROC relation the proposal predicts (Fig. 2).
5. **CellSAM rows are uninformative** at default settings. Its error rate is 72%, so κ_max is tiny, and its "high AUROC" only means "it missed the cell, so agreement = 0".

## Caveats (state these up front)
- **In-distribution:** cpsam, micro-SAM `vit_b_lm`, livecell_cp3 (and probably cyto3) all saw LIVECell *train*; only CellSAM is zero-shot. The full version needs a **held-out dataset**, e.g. NeurIPS22 CellSeg Public-Test. Its images can be range-read from the 2.9 GB `Testing.zip`, just as LIVECell was pulled from `images.zip`, so the full archive need not be downloaded.
- **40 images** and bootstrap over 40 clusters: the CIs are honest but come from one dataset, with 5 images per cell type.
- Seeds vary data order and augmentation only. There is no from-scratch reinit (L0a) yet; it needs a cellpose 3 `train_seg` from scratch, about 1–2 GPU-h per seed.
- The off-the-shelf levels (L2, L3) change decoder, objective and data at once, as the novelty-check notes already say.

## Pass criteria
- 3 seeds trained ✅ (30 min each, one L40S each)
- κ table with CIs for (a) seed–seed, (b) SAM vs SAM, (c) SAM vs non-SAM ✅, and (d) CellSAM ✅
- Effort estimate ✅. **About 18–20 student hours** for the full version, down from 20–22, since the env, the seed-training pipeline, matching, κ + bootstrap and AUROC now exist:

| Task | Hours |
|---|---|
| L0a from-scratch seeds (cellpose 3) and sbatch array | 2 |
| Held-out dataset: range-read NeurIPS22 Public-Test (phase-contrast/brightfield subset), GT conversion | 3 |
| Scale the test set to 100–200 images; held-out-cell-type probe (retrain seeds on 7 types) | 2 |
| Accuracy-matched κ analysis; stratify by error type (miss / merge / split) | 3 |
| Three figures (κ by level; AUROC vs κ; silent failures by level) plus a triage view | 4 |
| Write-up | 3 |
| Buffer | 2–3 |
