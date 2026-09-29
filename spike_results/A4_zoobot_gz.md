# Spike A4: D4 astronomy (Zoobot fine-tune + GZ3D mask alignment)

**Date:** 2026-09-29 · **Machine:** BC Andromeda, `weilab` partition, gdw001, **NVIDIA A10 (23 GB)**, 8 CPUs · **Wall time:** job 37 s (5.7 s alignment + 30.7 s fine-tune incl. downloads), about 25 min total including setup.
**Verdict: PASS.** The fine-tune runs and learns, and the GZ3D masks align with the Legacy Survey cutout.

## Setup
- **No separate `d4` env,** to stay near the disk budget. `zoobot[pytorch]==2.9.0` and `astropy 8.0.1` were pip-installed into `d5`, with torch pinned via constraints to the existing **conda-forge torch 2.7.1 (CUDA 12.6)**; timm is 1.0.30. Cellpose and micro-SAM still import afterwards (job 3073220). Zoobot 2.9.0 requires `torch>=2.7.0` and `timm>=1.0.15`.
- Code: `~/vis4ml_spikes/work/a4/a4_finetune.py` and `a4_gz3d_align.py` (job `jobs/a4_run.sbatch`, SLURM 3073245). Outputs are in `work/a4/out/`.
- The fine-tune uses the Zoobot encoder through `timm` (`hf_hub:mwalmsley/zoobot-encoder-convnext_nano`, Apache-2.0) plus a custom 3-answer **Dirichlet-multinomial head and loss** (the same likelihood Zoobot uses). Zoobot's own `FinetuneableZoobotTree` would be the next step.

## 1–3. Fine-tune one question on GZ DESI `tiny`
- Data: HF `mwalmsley/gz_desi`, **`tiny` config only** (0.175 GB; no `small` config exists, and the full dataset is 17.5 GB). License CC-BY-NC-SA-4.0.
- The dataset mixes question versions per campaign (`-dr5`, `-dr8`, `-dr12`). The script picked the one with the most votes, `smooth-or-featured-dr5`, with answers smooth / featured-or-disk / artifact.
- Galaxies with votes: 1,971 train / 492 test; median of 5 votes per galaxy for this question.
- Training: 2 epochs, batch 64, AdamW (encoder lr 1e-5, head lr 1e-3), bf16 autocast, flips, 224 px.

| | Test NLL (Dirichlet-multinomial) | Mean abs error of predicted vs. vote smooth fraction | Epoch time |
|---|---|---|---|
| Before fine-tune (random head) | 4.317 | 0.362 | – |
| Epoch 1 | 2.775 | 0.162 | 11.7 s (includes warm-up) |
| Epoch 2 | 2.730 | 0.158 | **3.0 s** |

- Train loss every 10 steps: 3.53, 2.90, 3.24, 2.95, 2.84, 3.18. It is noisy at 30 steps per epoch.
- Peak torch memory: 2.8 GB.
- SPECULATION: at about 1.5 ms per image, a fine-tune on the full GZ DESI train split would be minutes per epoch on one A10, probably limited by data loading rather than GPU. The full split would mean the 17.5 GB download, which needs your approval first.
- Caveat: `tiny` is too small for the stratified calibration analysis, as the deep dive noted. This only shows that the pipeline works.

## 4. GZ3D mask vs. Legacy Survey cutout
- The script scanned the DR17 GZ3D listing (29,813 files) and picked the first file with ≥3 bar drawers. It had to open only 2 files; the one chosen is **`gz3d_1-100017_61_14715415.fits.gz`** (MaNGA 1-100017, RA 357.5801, Dec −11.0665). Max votes per pixel: 13 for bar, 5 for spiral.
- GZ3D layout (FACT, this file):
  - HDU0: the 525×525 image shown to volunteers, with WCS, at **0.099″/px**.
  - HDU1–4: centre / star / spiral / bar vote-count masks.
  - HDU5: metadata (ra, dec, MANGAID, GZ_bar_votes, …).
- Cutout: Legacy Survey DR10, r band, 0.262″/px, 199×199 px, which is the same 52″ field. URL in `gz3d_alignment.json`.
- Method: resample the GZ3D image and masks onto the Legacy Survey pixel grid through both WCSs, then compare.

| Check | Result |
|---|---|
| FFT cross-correlation shift, GZ3D image vs. LS r | **1 px = 0.26″** |
| Pearson r at zero shift | 0.81 |
| LS light peak vs. catalogue RA/Dec | 0.0″ |
| Volunteer centre-mask centroid vs. LS light peak | 0.39″ |
| Bar-mask (≥3 votes) centroid vs. LS light peak | 0.75″ |
| Pixel-scale ratio GZ3D / LS | 0.378 (0.099 / 0.262), handled by the WCS |

- Visual check (`work/a4/out/gz3d_alignment.png`): the ≥3-vote bar contour lies along the bar, and the spiral contour follows the upper arm. **PASS.**
- FACT, and important for RQ-B: the **GZ3D HDU0 image has the MaNGA IFU hexagon drawn on it.** Attribution experiments must use the Legacy Survey cutout as model input, with the GZ3D masks resampled onto it as done here.
- FACT, and it supports the "brightness confound" risk: the mean LS flux inside the ≥3-vote bar mask is **30×** the mean flux of the cutout. Any attribution-vs-mask score must beat a light-profile baseline.
