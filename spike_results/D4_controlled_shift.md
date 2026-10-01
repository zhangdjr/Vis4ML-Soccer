# D4: Controlled distribution shift (R4, N3): own models on LIVECell leave-one-cell-type-out folds

**Verdict: R4 FALSIFIED (confirmatory, before Holm).**
- R4 statistic: mean over folds of [mean Δ over cross-architecture pairs − mean Δ over seed pairs].
- Result: **−0.57 [−0.72, −0.43]** (log-OR), one-sided p = 1.0.
- So under shift, seed pairs lose *more* error dependence than cross-architecture pairs. The prediction was the opposite.
- κ-based (secondary): +0.012 [−0.009, 0.036], null.

Design: PREREG Part B §B3 R4 + Amendment 5 item 3, fixed before any R4 statistic. Folds: `r4_frozen/N3_folds.json`. Code: `work/r4/r4_unet.py`, `r4_msam_train.py`, `r4_R4.py`. Output: `work/r4/out/n2/R4.json`.

## 1. Setup (FACT)
- **Folds.** The held-out type is SH-SY5Y, BV2 or SKOV3 (chosen to span morphology; frozen in commit 92a3130). Each fold trains on 200 LIVECell *train* images of the other 7 types.
- **Architecture A: Cellpose-3 U-Net from scratch.**
  - Recipe as in C4: 600 epochs, lr 0.005, batch 8.
  - **3 seeds per fold**, about 16–38 min each.
- **Architecture B: micro-SAM UNETR/AIS initialised from vanilla SAM ViT-B.**
  - SA-1B only, `sam_vit_b_01ec64.pth`, so no LIVECell exposure (D2 §2).
  - 8,000 iterations, lr 5e-5, 512² patches, best checkpoint on 14 validation images from the training types.
  - **3 seeds per fold**: 57 min on an L40S, 101 min on an A10.
- **Seed count.** The brief said 2 U-Net + 1–2 micro-SAM seeds. Amendment 5 raised this to 3 + 3 before any R4 statistic.
- **Retrained runs.** Three micro-SAM runs from last night hit the time limit on L4 GPUs. They were retrained from scratch on L40S with identical settings; the partial checkpoints are kept aside (`msam_ckpt/checkpoints/*_timeout_L4`) and were not used.
- **Evaluation.** 200 N2 images (25 per type, LIVECell test split).
  - In-type: the 175 images of the 7 training types.
  - Held-out: the 25 images of the held-out type.
  - U-Nets use the GT-median diameter, as in C4; micro-SAM uses AIS.
- **Pairs per fold.** 6 seed pairs (3 U-Net × U-Net, 3 µSAM × µSAM) and 9 cross pairs (U-Net × µSAM).
- **Bootstrap.** Images resampled within cell type, B = 1000.
- **No exclusions.** Every model's N2 error rate is ≤ 0.6.
- **GPU cost.**
  - micro-SAM: 17.6 GPU-h, including about 7.5 GPU-h lost to last night's three L4 timeouts.
  - U-Nets: 4.1 GPU-h.
  - Inference: about 0.5 GPU-h.

## 2. Results (FACT)
**Accuracy (share of GT cells matched at IoU > 0.5), mean of 3 seeds:**

| Fold | U-Net in-type | U-Net held-out | µSAM-vanilla in-type | µSAM-vanilla held-out |
|---|---|---|---|---|
| SH-SY5Y | 0.70 | **0.37** | 0.53 | **0.19** |
| BV2 | 0.67 | 0.79 | 0.51 | 0.37 |
| SKOV3 | 0.69 | 0.85 | 0.44 | 0.79 |

**Error dependence, mean over pairs** (in-type → held-out):

| Fold | Seed log-OR | Δ seed | Cross log-OR | Δ cross | Cross − seed [CI] | Seed κ | Cross κ |
|---|---|---|---|---|---|---|---|
| SH-SY5Y | 5.00 → 4.19 | 0.81 | 3.11 → 2.95 | 0.16 | **−0.65** [−0.76, −0.51] | 0.83 → 0.74 | 0.53 → 0.45 |
| BV2 | 4.74 → 4.60 | 0.14 | 2.94 → 3.48 | −0.53 | **−0.68** [−1.01, −0.33] | 0.82 → 0.79 | **0.52 → 0.25** |
| SKOV3 | 4.89 → 4.39 | 0.50 | 3.17 → 3.06 | 0.11 | **−0.39** [−0.64, −0.10] | 0.83 → 0.73 | 0.45 → 0.54 |
| **Pooled (R4)** | | | | | **−0.57 [−0.72, −0.43]** | | κ version: +0.012 [−0.009, 0.036] |

- Cross-architecture pairs are less dependent than seed pairs throughout (log-OR about 3 vs about 5), as in round 3.
- **The negative sign holds in every fold**, including SH-SY5Y, the only fold where the held-out type is genuinely harder.

## 3. Reading
1. **The "shift" is real only for SH-SY5Y (SYNTHESIS).**
   - BV2 and SKOV3 are *easier* than the training mix for the U-Nets: they are round, well separated, or large and flat.
   - So two of three folds test "a new cell type", not "a harder distribution".
   - The fold choice was frozen before training, on morphology. We report it as pre-registered and do not swap folds.
   - Because the SH-SY5Y fold alone gives −0.65 [−0.76, −0.51], the falsification does not depend on the two weak folds.
2. **Why seed dependence drops more (SYNTHESIS, testable).**
   - Under shift, the cells a model misses split into two kinds:
     - cells that are hard for *any* model trained on the 7 types (shared by both architectures);
     - cells a given seed happens to miss.
   - The seed-specific part grows, which hits seed pairs, whose baseline log-OR is near 5. Cross-architecture pairs were already sharing mainly the "hard for anyone" cells, which stay shared.
   - In other words, shift moves seed pairs *toward* cross pairs. It does not pull cross pairs apart.
3. **κ and log-OR disagree on BV2, and log-OR is the honest one here (FACT + SYNTHESIS).**
   - On held-out BV2 the U-Net reaches 0.79 accuracy but µSAM-vanilla only 0.37. Mismatched margins cap κ: κ falls from 0.52 to 0.25 while log-OR rises from 2.94 to 3.48.
   - A κ-only analysis would have reported "shift decorrelates cross-architecture errors on BV2", which is a margin artefact.
   - This is the clearest case in the project for making log-OR co-primary.
4. **What this does to the narrative.**
   - The round-3 pattern ("error consistency collapses out of distribution", fig_kappa_by_level) was a κ statement about **pretrained generalists** on Public-Test.
   - With own models and a controlled shift, the margin-free dependence of different architectures barely moves (Δ cross ≈ 0.1–0.2 where the shift is real).
   - SYNTHESIS: much of the round-3 "collapse" is probably about error rates (margins) and about what each generalist was trained on. A shift alone does not make independent models more independent.

## 4. Limitations
- There are only 3 folds, one of them a genuine shift. The 3 seeds per architecture give 6 seed pairs and 9 cross pairs per fold, and those pairs share models, so the pair-level variation is not independent. The CI comes from the image bootstrap only.
- µSAM-vanilla is a weak model: 0.43–0.55 in-type accuracy, from 200 training images and a short schedule. Its error rate differs a lot from the U-Net's, which is exactly where κ misleads. log-OR is robust to this, but not to every margin effect.
- Seed effects in micro-SAM come from both initialisation of the new decoder and data order. The encoder starts from the same SA-1B checkpoint in every seed, so "seed" here means less diversity than for the from-scratch U-Nets.
