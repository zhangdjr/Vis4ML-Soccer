# Spike B2: D4 ambitious, explanation quality vs GZ3D masks, a brightness baseline, and human ambiguity

**Date:** 2026-09-29 · **Machine:** BC Andromeda. Data prep on `short` (CPU); model + attributions on `weilab` gdw001, **A10 23 GB**.
**Wall time:** about 2 h, including two failed prep runs (a bug, then rate limiting). The final prep took 18 min; the GPU job took 4.6 min.
**Verdict: PASS for feasibility, with a PROMINENT RED FLAG.** Attributions **rarely beat the brightness baseline** at localizing volunteer bar masks. The first-signal correlations are weak and mostly null after controlling for model uncertainty.

## What was run

**Galaxy selection** (`work/b2/b2_data.py`, SLURM 3074477):
- Vote catalogues (Zenodo, CC BY 4.0, no images): GZ DECaLS `gz_decals_volunteers_5` and `_1_and_2` parquet (41 + 19 MB), and GZ DESI `gzd8_volunteer_core` (6 MB). **66k galaxies have ≥10 bar votes.** The 17.5 GB GZ DESI image set was *not* downloaded.
- Pre-match: MaNGA `drpall-v3_1_1.fits` (75 MB, RA/Dec for every MaNGA ID) against the vote catalogues within 2″ gives **1,027 MaNGA galaxies** with a GZ3D file and ≥10 independent bar votes.
- Selection: GZ3D files fetched in parallel from DR17 SAS, in random order, until **80 galaxies with a ≥3-drawer bar mask + 20 without** (352 files scanned). The vote source is gzd5 for 52, gzd12 for 46 and gzd8 for 2; median 30 bar votes.

**Cutouts:**
- Legacy Survey DR10 grz JPEG and r-band FITS, 424 px on the **GZ3D field (51.97″, 0.1226″/px)**.
- GZ3D bar/spiral/centre vote masks resampled through both WCSs (as in A4).
- **Orientation verified:** the flipped FITS grid correlates better with the JPEG in **100/100** galaxies (median r 0.88 vs 0.71).
- The GZ3D HDU0 is not used as input; it has the IFU hexagon drawn on it.

**Bugs fixed on the way** (worth knowing for the full version):
1. GZ3D HDU0 is (ny, nx, **3**), channels last. Taking `shape[-1]` as the width gave a 0.3″ field.
2. legacysurvey.org returns **HTTP 429 at 6 concurrent requests**. 2 concurrent requests with exponential backoff works.

**Model** (`work/b2/b2_run.py`, 3074478):
- Zoobot ConvNeXt-nano encoder + a 2-way Dirichlet-multinomial ("bar vs no bar") head.
- Training data: GZ DESI `tiny`, bar votes **pooled across campaigns** (dr5 / dr8 strong+weak vs no; dr12 yes vs no), galaxies with ≥3 votes → 1,566 train / 379 test.
- **3 seeds**, 5 epochs each: **13–25 s per seed** on the A10.
- Test NLL 1.55–1.57; mean absolute error of the predicted bar fraction 0.144.
- One GZ3D galaxy overlapped `tiny` and was excluded, leaving **n = 99** (79 with a bar mask ≥ 20 px on the 224 grid; median mask 950 px).
- ρ(model p_bar, volunteer p_bar) = **0.68**, so the head learned something real.

**Attributions** (Captum 0.9), target = the logit of the predicted bar fraction, 224 px input (the timm eval geometry reproduced for the masks):
- IG (32 steps, blurred-image baseline);
- SmoothGrad (16 samples, σ 0.15);
- Grad-CAM (last ConvNeXt stage);
- Occlusion (16 px window, stride 8, blurred fill).

That is 3 seeds × 4 methods × 99 galaxies in **177 s**.

**Metrics per galaxy:**
- **Cross-method agreement:** mean pairwise Pearson r over the 6 method pairs on 28×28 pooled maps (seed 1).
- **Cross-seed agreement:** the same metric per method across the 3 seed pairs.
- **Localization vs the ≥3-vote bar mask:** AUPRC, relevance-mass accuracy (RMA) and pointing game, compared against a **brightness map** (smoothed r-band flux) and a **radial map**.
- **ΔAUPRC:** the gain in AUPRC when the seed-averaged attribution is added to a per-galaxy logistic "light-profile" model (log flux + radius) on 56×56 pixels.
- **Human entropy H_h:** the binary entropy of the Beta posterior mean of the independent bar vote fraction. Also GZ3D-internal: bar drawers / classifiers.
- **Model uncertainty:** the entropy of the seed-mean predicted fraction, and the seed standard deviation.

## Result 1: localization, attributions vs brightness (n = 79) — RED FLAG

| Map | Median AUPRC | Beats brightness (share of galaxies) | Pointing game | Median RMA | Median ΔAUPRC over light profile | Share with ΔAUPRC > 0.01 |
|---|---|---|---|---|---|---|
| **Brightness (r-band)** | **0.825** | — | **0.65** | — | — | — |
| Radial (distance to centre) | 0.746 | — | — | — | — | — |
| Integrated Gradients | 0.294 | **10%** | 0.47 | 0.15 | +0.005 | 41% |
| SmoothGrad | 0.420 | **17%** | 0.48 | 0.14 | +0.020 | 63% |
| Grad-CAM | 0.442 | **17%** | 0.49 | 0.14 | +0.005 | 41% |
| Occlusion | 0.288 | **14%** | 0.46 | 0.21 | +0.012 | 56% |

- FACT: on its own, **brightness localizes the volunteer bar far better than any attribution**. Averaged over methods, attributions beat it on **11%** of galaxies; at least one method beats it on 27%.
- Attributions carry only a **little** information beyond the light profile: median ΔAUPRC +0.005 to +0.02; positive (> 0.01) for 41–63% of galaxies.
- SYNTHESIS: bars are the bright central feature, and volunteers draw them there. A "localization vs GZ3D mask" score is therefore dominated by the light profile.
  - The full version must use **ΔAUPRC (or a partial-correlation metric) as the primary outcome, not raw localization.**
  - The effect it can detect is small. That is the key risk the novelty check flagged, and it is now confirmed.
- Visual check (`work/b2/out/examples.png`): masks sit on the bars. For the clearest bar (lowest H_h), IG, SmoothGrad and occlusion trace the bar. For ambiguous galaxies the attributions scatter off it, while brightness still peaks on the centre.

## Result 2: first signal, human entropy vs explanation metrics (two-sided; Spearman ρ, partial = rank-residualized on model entropy + seed std; bootstrap 95% CI)

| Human ambiguity | Explanation metric | n | ρ [CI] | Partial ρ [CI] |
|---|---|---|---|---|
| Independent votes (H_h) | cross-method agreement | 99 | **+0.24** [0.04, 0.43] | +0.07 [−0.11, 0.27] |
| H_h | cross-seed agreement | 99 | +0.15 [−0.05, 0.32] | +0.02 [−0.18, 0.20] |
| H_h | gain over brightness (mean of methods) | 79 | −0.02 [−0.23, 0.19] | −0.05 [−0.26, 0.17] |
| H_h | **ΔAUPRC over light profile** | 79 | **−0.23** [−0.44, −0.01] | **−0.27** [−0.47, −0.04] |
| GZ3D drawers (gz3d_H) | cross-method agreement | 99 | **+0.34** [0.14, 0.52] | +0.17 [−0.03, 0.37] |
| gz3d_H | cross-seed agreement | 99 | **+0.33** [0.14, 0.51] | **+0.22** [0.03, 0.42] |
| gz3d_H | ΔAUPRC | 79 | −0.02 [−0.25, 0.22] | −0.03 [−0.27, 0.20] |
| Model seed std (control) | gain over brightness | 79 | −0.24 [−0.45, −0.01] | −0.27 [−0.47, −0.04] |

- Collinearity: ρ(H_h, model entropy) = 0.38; ρ(H_h, gz3d_H) = 0.30.

SYNTHESIS (n ≈ 100; 13 correlations; **nothing survives a Holm correction**):
- **Direction matches Jukić 2023, not the naive hypothesis.** More ambiguous galaxies have *higher* cross-method and cross-seed agreement (ρ +0.24 to +0.34). Most of that is explained by model uncertainty: the partials shrink toward 0, except GZ3D-internal ambiguity vs cross-seed agreement (+0.22).
  - A plausible mechanism (SPECULATION): on ambiguous galaxies every method falls back to the bright centre, so the methods agree *because* they are all uninformative.
- **The one "beyond model uncertainty" hint is on the right outcome.** Independent-vote ambiguity predicts a *smaller* ΔAUPRC over the light profile (partial ρ −0.27, CI excluding 0). That is the RQ-B hypothesis, but it is a single uncorrected test at n = 79.
- Model seed spread also predicts worse localization (partial −0.27), reproducing the model-side literature (Mikriukov / Bhatt).

## Pass criteria
- Pipeline runs on 100 galaxies ✅ (99 usable after the train-overlap exclusion).
- Attributions beat brightness on some galaxies: ✅ technically, but only 10–17% per method. **RED FLAG, as the brief asked to flag.** Relative to the light-profile model they add only +0.005 to +0.02 AUPRC.
- First-signal table ✅.
- **Is the 17.5 GB download required? Mostly no for the question itself; yes only for a stronger model.**
  - SYNTHESIS: the human-ambiguity and mask inputs come from the small catalogues plus GZ3D and Legacy Survey cutouts. That scales to about 1,000 galaxies (the 1,027 pre-matched) at roughly 20 min per 100 with polite rate limits.
  - The GZ DESI images are only needed to train a *better* model (a 5-seed ensemble on the full split) than the tiny-trained head, whose MAE is 0.14.
  - A middle path: train on a larger **subset** via Hugging Face streaming, or use Legacy Survey cutouts of about 20k GZ DECaLS vote-catalogue galaxies, fetched politely. This needs a decision, not necessarily the 17.5 GB.

**Full-version effort estimate: about 21–24 student hours** (the check note said 21).
- Pipeline, masks, attributions and correlations now exist: roughly −4 h.
- But the brightness dominance means designing a stronger outcome (ΔAUPRC with a Sérsic model; deletion faithfulness; a spiral-arm replication where brightness is less dominant) and a proper power analysis for a small effect: about +4–6 h.
- Scaling to about 1,000 galaxies: data-join time (about 3 h of prep jobs) and cutout politeness.

## Caveats
- Model: trained on `tiny` only (1,566 galaxies), 5 epochs, 3 seeds. Weaker than a real Zoobot fine-tune.
- **Mask quality may fall with ambiguity** (confound #1 in the check note): fewer or less consistent drawers on borderline bars. The ΔAUPRC hint could partly be mask noise. The full version needs the number of drawers and a spatial-ambiguity score as covariates.
- Galaxies without a ≥3-drawer bar mask (20) enter only the agreement analyses.
- The cutout scale is fixed by the GZ3D field (52″), not Zoobot's size-adaptive DESI crops, so there is some domain shift from the training images.
