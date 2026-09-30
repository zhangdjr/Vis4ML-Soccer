# Spike C1: held-out replication on NeurIPS22 Public-Test (K1; H1–H4)

**Date:** 2026-09-29/30 · **Machine:** BC Andromeda. Inference on `gtml` (L40S / L4) and `weilab` (A10); analysis on `short`.
**Wall time:** about 1 h 40 min end to end, about 4.7 GPU-hours in total across C1–C4.
**Pre-registration:** `PREREG_D5.md`, plus Amendments 1–3. All three were committed and pushed **before** any Public-Test statistic was computed (commits `4183eec`, `8eb1b38`, `7d3f29f`).

**Verdict (confirmatory, Public-Test):**

| Hypothesis | Result |
|---|---|
| H1 | **supported** |
| H2 | **supported** |
| H3 | **supported** |
| H4 | **inconclusive** (neither supported nor falsified) |

K1 passes. But two kill tests trigger (K4 accuracy-matched, K6 practical triage; see C2) and the H3 sensitivity reverses. The headline therefore needs **reframing** (see "Is D5 still safe?" at the end).

## Data
**NeurIPS22 CellSeg Public-Test** (Zenodo 10719375, `Testing.zip`):
- Only `Public/images` and `Public/labels` were range-read: 50 images + 50 label TIFFs, 253 MB compressed, of the 2.9 GB archive. The two WSIs are excluded.
- `Hidden/` has **no labels**, so it can't be used.
- CC BY-NC-ND 4.0: nothing derived (masks, overlays, thumbnails) is committed. Only aggregate statistics are.
- **6,040 GT cells** after the < 20 px exclusion. Cellpose-SAM makes only 300 errors, so the Public-Test CIs are wider than LIVECell's.

**Modality**, labelled by eye from a thumbnail montage (SYNTHESIS; no metadata ships with the data):

| Group | Images | Cells |
|---|---|---|
| stained brightfield smears | 18 | 2,011 |
| fluorescence | 12 | 1,593 |
| phase-contrast bacteria | 8 | 858 |
| round/yeast-like cell clusters | 7 | 613 |
| LIVECell-like phase-contrast cultured cells | 5 | 965 |

**LC200** (secondary): 25 LIVECell test images per cell type, containing B1's E40.
- **FACT:** the LIVECell test json lists **52 files twice** under different ids, with *disjoint* annotation sets. The 4 affected files are excluded, leaving **194 images and 53,034 cells** (Amendment 2).
- GT is now painted label images. On B1's E40 images this reproduces B1's κ within 0.02: cpsam–cyto3 0.801 vs 0.795; cpsam–micro-SAM 0.553 vs 0.570; seeds 0.917 vs 0.915.

## Models (all run; per PREREG settings)
Shared preprocessing: each channel is scaled to uint8 between its 1st and 99.8th percentiles. The grayscale-default models get the channel mean.

| Model | LIVECell (LC200) acc | Public-Test acc | Notes |
|---|---|---|---|
| Cellpose-SAM (`cpsam`) | 0.747 | **0.950** | defaults |
| cpsam, `flow_threshold=0` | 0.826 | 0.955 | QC-signal run only. Recall rises because default flow filtering drops true cells (FPs are not counted by this metric) |
| B1 fine-tune seeds s1–s3 | 0.737–0.747 | 0.952–0.953 | |
| cyto3 (auto diameter) | 0.724 | **0.892** | GT-median diameter: 0.732 / 0.900 |
| livecell_cp3 (cyto3's diameter) | 0.732 | **0.470** | LIVECell specialist; fails out of distribution, but error 0.53 < 0.6, so it stays in |
| micro-SAM AIS `vit_b_lm` | 0.583 | **0.852** | E40 0.604 vs B1's 0.649; SPECULATION: micro-SAM is sensitive to the input rescaling |
| micro-SAM AMG (thresholds 0.7/0.8, tuned on LIVECell train) | 0.278 (excluded) | 0.700 | H5 only |
| CellSAM `cellsam_general` | 0.317 (excluded) | **0.722** | included on Public-Test (error < 0.6) |
| own U-Nets (C4) | 0.711–0.718 | 0.38–0.46 | LIVECell-trained, so descriptive only on Public-Test |

Accuracy = the share of GT cells matched at IoU > 0.5 (strict, Hungarian).

## Result: per-pair κ on Public-Test (key pairs; all 28 primary pairs in `work/c1/out/pt/pairs.csv`)

| Pair | Level | κ [95% image-bootstrap CI] | κ/κ_max | P(b wrong \| a wrong) | Silent failures /1000 | AUROC flagging a's errors |
|---|---|---|---|---|---|---|
| FT seed–seed (3 pairs) | L0b | **0.91–0.93** | 0.91–0.94 | 0.92 | 28 | 0.77–0.80 |
| cpsam vs its fine-tunes | L1 | 0.83–0.85 | 0.85–0.87 | 0.82 | 24–25 | 0.74–0.79 |
| **cpsam vs cyto3** | same family | **0.333** [0.278, 0.385] | 0.55 | **0.60** | 13.1 | 0.87 |
| **cpsam vs micro-SAM** | shared SAM encoder | **0.169** [0.107, 0.230] | 0.37 | **0.46** | 7.3 | **0.90** |
| cyto3 vs micro-SAM | different family, different encoder | 0.158 | 0.19 | 0.31 | 13.1 | 0.89 |
| micro-SAM vs CellSAM | both SAM-based | 0.390 | 0.63 | 0.73 | 28.5 | 0.77 |
| cpsam vs CellSAM | both SAM-based | 0.127 | 0.53 | 0.66 | 8.8 | 0.83 |
| cpsam vs livecell_cp3 | same family (specialist) | 0.024 [−0.002, 0.067] | 0.27 | 0.66 | 7.9 | 0.75 |
| cyto3 vs livecell_cp3 | same architecture, different data | 0.040 | 0.20 | 0.63 | 16.2 | 0.67 |

LIVECell (LC200) for comparison. It reproduces B1:

| Pair | κ |
|---|---|
| seeds | 0.91 |
| cpsam vs FT | 0.86–0.87 |
| cyto3 vs livecell_cp3 | 0.84 |
| **cpsam vs cyto3** | **0.78** |
| micro-SAM vs cyto3 | 0.57 |
| **cpsam vs micro-SAM** | **0.52** |

## Confirmatory hypotheses (Public-Test; Holm-adjusted p over H1–H4)

| | Pre-registered test | Public-Test result | Holm p | Verdict | LC200 (secondary) |
|---|---|---|---|---|---|
| **H1** family > shared encoder | κ(cpsam, cyto3) − κ(cpsam, micro-SAM) > 0, CI excludes 0, same sign for κ/κ_max | **+0.164 [0.086, 0.245]**; κ/κ_max +0.186 [0.057, 0.316] | 0.004 | **Supported** | +0.262 [0.235, 0.290]; κ/κ_max only **+0.023** [0.002, 0.044] |
| **H2** seeds most consistent | every seed pair > every cross-model pair | seeds 0.91–0.93 vs max cross 0.39; closest gap +0.52 [0.44, 0.59] | 0.004 | **Supported** | supported; closest +0.074 [0.066, 0.082] (vs cyto3–livecell_cp3) |
| **H3** agreement-QC improves as relatedness falls | ρ(pair κ, mean QC AUROC) < 0 | **ρ = −0.30 [−0.41, −0.14]**, 28 pairs | 0.004 | **Supported** | ρ = −0.93 [−0.97, −0.88] |
| **H4** agreement beats baselines | agreement(cpsam, ref) AUROC ≥ best baseline + 0.05 with CI of Δ > 0; falsified if ≤ attribute + 0.02 | ref = micro-SAM, chosen on LC200 as pre-registered. **0.898** vs flow error (`flow_threshold=0`) **0.885**: Δ +0.013 [−0.043, 0.072]. vs attribute model 0.537 | 0.70 | **Inconclusive** | **Falsified:** agreement 0.739 vs flow error 0.842, attribute 0.742 |

### What the sensitivities say (exploratory; Amendment 3, fixed before Public-Test statistics)
**H1: the accuracy gap is the main threat.**
- On LC200, P(b wrong | cpsam wrong) is **0.88 for cyto3 and 0.89 for micro-SAM**. The raw-κ gap there comes almost entirely from micro-SAM's *extra* errors (accuracy 0.58 vs 0.72), which is why κ/κ_max barely differs.
- On Public-Test the conditional overlap does differ: **0.60 vs 0.46**. This holds even though micro-SAM has *more* errors, which by chance alone would inflate its overlap. So the Public-Test effect is more than an accuracy artifact.
- But the accuracy-matched subset (K4, C2) does not reproduce it.

**H3 is mostly driven by the weaker model's direction on Public-Test.**
- With the pair's **more accurate** model as the target, ρ **reverses** to **+0.23 [0.08, 0.35]**. On LC200 it stays negative (−0.56 [−0.78, −0.38]).
- κ/κ_max vs AUROC: −0.29 (Public-Test), −0.84 (LC200).
- Cellpose-SAM as the target, across its 7 references:
  - Public-Test ρ = −0.21 [−0.50, 0.18]: not significant.
  - Its best reference is micro-SAM (0.898), then cyto3 (0.866), CellSAM (0.827), seeds (0.74–0.79) and livecell_cp3 (0.75).
- SYNTHESIS: "pick a less-related reference" helps a *weak* model a lot. For the strongest model it only helps against seed copies.

**H4, other views:**
- Cells where Cellpose-SAM has an overlapping instance (the deployable QC case): agreement 0.868 vs flow error (`flow_threshold=0`) 0.841 vs default flow error 0.759.
- Agreement beats the attribute model by +0.33 [0.24, 0.41] on Public-Test.
- **Attribute-only error models are nearly uninformative on Public-Test**: within-image AUROC 0.50–0.57 for the Cellpose models, against about 0.75 on LIVECell.

## K1 (kill rule: drop the hierarchy headline if H1 **and** H3 are falsified on Public-Test)
**Not triggered.** H1 and H3 both pass their pre-registered tests.

But both passes are weaker than they look:
- **H1** fails its accuracy-matched check (K4).
- **H3** reverses when the target is the stronger model.

See C2 and the verdict.

## Other findings worth a sentence (FACT unless marked)
1. **Out of distribution, cross-model consistency collapses while seeds don't.**
   - cpsam–cyto3 κ goes from 0.78 (LIVECell) to **0.33** (Public-Test).
   - cpsam–micro-SAM goes from 0.52 to **0.17**.
   - Seeds stay at 0.91–0.93.
   - SYNTHESIS: in-distribution, most errors are shared "hard cells" that every LIVECell-trained model misses. Out of distribution, errors become model-specific. That is why the pilot's κ ordering looked so strong.
2. **The strongest held-out consistency among different models comes from shared training data plus architecture.** Our own LIVECell-trained U-Nets vs `livecell_cp3` reach κ 0.73–0.84 on Public-Test, while they are ~0 with everything else. Models trained on the same data fail *together* out of distribution.
3. **Silent failures**, where both models are wrong and agree with each other, per 1,000 cells on Public-Test:
   - 28 for seed pairs;
   - 13 for cpsam–cyto3;
   - 7 for cpsam–micro-SAM.

   On LIVECell: 166, 122 and 68 respectively.
4. **`flow_threshold=0` raises Cellpose-SAM's GT-cell recall on LIVECell from 0.747 to 0.826.** The default flow filter removes many real cells there; this metric ignores the extra FPs.

## An oversight this round exposed: the "shared encoder" pair doesn't share an encoder
- **FACT** (from the installed code: `cellpose/vit.py`, `micro_sam`, `cellSAM/sam_inference.py` and `AnchorDETR/models/anchor_detr.py`):
  - **Cellpose-SAM uses SAM ViT-L.**
  - **micro-SAM `vit_b_lm` and CellSAM both use SAM ViT-B.**
- So H1's "shared SAM encoder" pair shares only the SAM *pretraining family*, not the same encoder or checkpoint. The earlier docs (B1, the onboarding, PREREG) glossed over this.
- The one cross-family pair that shares an encoder architecture, **micro-SAM vs CellSAM**, is the **most consistent cross-family pair** on Public-Test:

  | | Public-Test | LC200 |
  |---|---|---|
  | micro-SAM vs CellSAM | κ 0.39 [0.33, 0.45], κ/κ_max 0.63 | 0.42 |
  | CellSAM vs the Cellpose models | 0.13–0.24, κ/κ_max 0.50–0.53 | 0.25–0.28 |

  (CellSAM is excluded from the LC200 κ comparisons because its accuracy there is 0.32.)
- SPECULATION: a narrower H-mono ("same encoder architecture and pretrained checkpoint → shared errors") is **not** refuted and may even hold.
  - Confounds: both are the weaker models, and both are SAM-prompt-style decoders.
  - The full study should test it directly: micro-SAM `vit_b_lm` vs `vit_l_lm` (same recipe, different encoder), and CellSAM vs micro-SAM.

## Files
- Code: `~/vis4ml_spikes/work/c1/`:
  - `c1_data.py`, `rangefile.py`: range-read data;
  - `c1_infer.py`: all models;
  - `amg_tune.py`;
  - `c1_match.py`: sparse Hungarian matching; unit-tested against A1's `match.py`;
  - `c1_cells.py`: per-cell table;
  - `c1_stats.py`: joint image-bootstrap stats;
  - `c1_extra.py`;
  - `c5_fig.py`.
- Outputs: `out/{pt,lc200}/{cells.csv.gz,pairs.csv,hypotheses.json,k2_difficulty.csv,levels.csv}`, `out/extra.json`.
- Jobs: `~/vis4ml_spikes/jobs/c1_*.sbatch`, `c4_train.sbatch`, `c5_fig.sbatch`. All carry the BEGIN/END/FAIL mail flags.

## Is D5 still safe? (round-3 verdict, blunt)

| | Verdict |
|---|---|
| H1 family > shared SAM | **Supported** as pre-registered, but **accuracy-confounded** (K4), concentrated in 2/5 modalities, and the "shared encoder" label is wrong (ViT-L vs ViT-B) |
| H2 seeds most consistent | **Supported**, robustly, on both datasets |
| H3 less related → better QC | **Supported** as pre-registered. **Fragile:** it reverses when the target is the stronger model |
| H4 agreement beats baselines | **Inconclusive** on Public-Test (≈ flow error, ≫ attributes). **Falsified** on LIVECell |
| K1 out-of-distribution replication | passed |
| K2 hard cells | passed |
| K3 preprocessing | passed |
| **K4 accuracy** | **triggered** |
| K5 threshold / modality | passed, but heterogeneous |
| **K6 practical** | **triggered** |
| K7 scoop | not checked this session; do it manually before Oct 20 |

**As a course project, D5 is safe.** The floor holds on held-out data:
- the per-cell error taxonomy;
- the seed hierarchy;
- agreement-QC far above the attribute baselines;
- silent-failure counts.

Several findings are genuinely reportable:
- the out-of-distribution collapse of consistency;
- LIVECell-only models failing together;
- the ViT-B pair.

**As a paper headline, "family, not encoder" should be retired.** Suggested reframing:
> *"Error consistency between cell segmenters is high in-distribution and collapses out of distribution. What predicts shared failures there is shared training distribution and weights, not a shared foundation-model pretraining. Cross-model agreement is a strong QC signal, but no better than the model's own flow-error signal. A same-family reference triages the best model's errors as well as a cross-family one."*

Pre-register accuracy-matched κ as a primary analysis, and add the same-checkpoint contrast (micro-SAM ViT-B vs ViT-L).
