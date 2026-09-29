# Spike A1: D5 microscopy, two generalist segmenters and per-cell error scoring

**Date:** 2026-09-29 · **Machine:** BC Andromeda, `gtml` partition, grs001, **NVIDIA L40S (46 GB)**, 8 CPUs · **Wall time:** final job about 2 min (the first run also downloaded weights: 101 s); about 1.5 h total including env debugging.
**Verdict: PASS.** Both models run, matching works, the per-cell table exists, and every image takes ≤ 1.8 s, far under the 60 s target. Three caveats are listed at the end.

## Environment
- `d5` at `~/vis4ml_spikes/envs/d5`: python 3.11, **conda-forge `micro_sam 1.8.14`**, pytorch 2.7.1 (CUDA 12.6 build), torch_em 0.10.7, plus pip **`cellpose 4.2.1.1`**.
- The env is **11–12 GB** because the CUDA libraries plus napari come along with micro_sam. This is bigger than planned; see the disk section in `research_gap_review.md` §12.
- Gotcha 1: on a CPU node, conda cannot see `__cuda`, so the first solve failed. The fix: `CONDA_OVERRIDE_CUDA=12.6` plus `"pytorch=*=cuda*" cuda-version=12.6`.
- Gotcha 2: `pip check` reports `kornia 0.8.2` wants `kornia_rs>=0.1.9`, but 0.1.8 is installed. It is harmless here.
- Weights: Cellpose-SAM `cpsam` comes from HF on first use. micro-SAM `vit_b_lm` and its decoder come from the BioImage.IO S3 bucket, about 375 MB. Both are cached under `~/vis4ml_spikes/hf_cache/`, not `~/.cellpose`.

## Data
- LIVECell test split (CC BY-NC 4.0). `livecell_coco_test.json` (261 MB) was downloaded, subset, then deleted.
- **8 images, one per cell type**: the median-density test image of each type.
- Images were pulled out of the 1.24 GB `images.zip` via **HTTP Range requests**: 11 requests, 1.9 MB total. There was no need to fetch the archive.
- **2,006 GT cells** in total. Script: `work/a1/a1_data.py`; subset saved at `data/livecell/subset_test_8.json`.

## Commands
- `sbatch jobs/a1_data.sbatch` (CPU)
- `sbatch jobs/a1_run.sbatch` (GPU; final run SLURM 3073261)
- `sbatch jobs/amg_debug.sbatch` (3073252)
- Code is in `work/a1/`: `a1_run.py` and `match.py`. Outputs are in `work/a1/out/`: `per_cell.csv`, `per_image_counts.csv`, `predictions.npz`, `report.json`.

## Speed and memory (8 images, 520×704)

| Model | Per image (s) | Load (s, cached) | Peak torch alloc / nvidia-smi |
|---|---|---|---|
| Cellpose-SAM (`CellposeModel(gpu=True)`, defaults) | 0.26 (first 0.51) | 1.4 | 1.9 GB / 4.1 GB |
| micro-SAM AIS (`vit_b_lm` + decoder, `segmentation_mode="ais"`) | 0.13 | 0.5 | 2.8 GB / 3.7 GB |
| micro-SAM AMG (`segmentation_mode="amg"`) | 1.6 | 0.5 | 2.8 GB / 4.4 GB |

All three would fit comfortably on an 11–12 GB card as well.

## Matching and taxonomy (`match.py`)
- GT is kept as a list of possibly overlapping COCO masks.
- Matching is Hungarian on IoU, keeping pairs with IoU ≥ 0.5.
- **merge:** an unmatched GT cell that is ≥50% covered by a prediction which covers ≥50% of ≥2 GT cells.
- **split:** an unmatched GT cell that contains ≥2 predictions, each lying ≥50% inside it.
- **miss:** any other unmatched GT cell.
- **FP:** an unmatched prediction that is not a merge and lies <50% inside any GT cell.

| Model | GT | Pred | TP | Miss | GT in merges | Split | FP | Pooled F1@0.5 |
|---|---|---|---|---|---|---|---|---|
| Cellpose-SAM | 2006 | 1816 | 1650 | 255 | 87 | 14 | 52 | 0.863 |
| micro-SAM AIS | 2006 | 1748 | 1451 | 131 | **407** | 17 | 64 | 0.775 |
| micro-SAM AMG (thr 0.7 / 0.8) | 2006 | 745 | 686 | 1240 | 80 | 0 | 27 | 0.500 |

- Per-image F1@0.5: Cellpose-SAM 0.68 (SHSY5Y) to 0.97 (SkBr3); micro-SAM AIS 0.56 (SHSY5Y) to 0.95 (SkBr3). See `per_image_counts.csv`.
- SYNTHESIS: the two models fail differently. Cellpose-SAM mostly *misses* cells (SHSY5Y: 128 misses). micro-SAM AIS mostly *merges* them (MCF7: 156 GT cells in merges). That is exactly the kind of structure a per-instance error taxonomy view would show.

## Cross-model agreement (smoke test, 8 images, not a result)
Agreement for a GT cell = IoU between the two models' best-overlapping instances for that cell.

| Spearman ρ | All cells | Median within-image | Cells detected by both |
|---|---|---|---|
| agree(Cellpose-SAM, AIS) vs Cellpose-SAM IoU | 0.643 | 0.531 | 0.624 (n = 1,945) |
| agree(Cellpose-SAM, AIS) vs AIS IoU | 0.772 | 0.645 | 0.759 |
| agree(Cellpose-SAM, AMG) vs Cellpose-SAM IoU | 0.543 | 0.442 | 0.507 (n = 1,117) |

**Model-internal signals** (Spearman ρ with the same model's per-cell IoU):

| Signal | ρ | n |
|---|---|---|
| Cellpose flow error | −0.625 | 1,968 |
| Cellpose mean cell probability | +0.576 | 1,968 |
| micro-SAM AIS mean foreground | +0.442 | 1,980 |
| micro-SAM AMG predicted IoU | +0.405 | 1,120 |
| micro-SAM AMG stability score | +0.358 | 1,120 |

**Error consistency (first look at RQ2).** An "error" here is any non-TP status.
- Cellpose-SAM vs AIS: **Cohen's κ = 0.52**, and **80% of Cellpose-SAM's error cells are also AIS errors**. There are **113 "silent failures"**: cells where both models are wrong yet agree with each other at IoU ≥ 0.5.
- Status cross-table: of 255 Cellpose-SAM misses, AIS merged 130, missed 62, got 57 right, and split 6.

SYNTHESIS, 8 images: agreement looks about as informative as Cellpose's own flow error, and more informative than micro-SAM's internal scores. Errors are substantially correlated across the two SAM-backbone models. RQ2 needs a non-SAM model to test whether that correlation is about lineage. The median within-image ρ (0.53–0.65) is lower than pooled, which is the "agreement partly encodes image difficulty" effect the design anticipates.

## Answers to the "also note" questions (FACT, from installed source + run)
- **Does Cellpose expose flow error or cell probability per instance?** Not as a per-instance return value. `eval()` returns `flows = [HSV, dP, cellprob, p]`, and `cellpose.dynamics.flow_error(masks, dP)` returns **per-mask flow errors**. It is the same quantity `flow_threshold` filters on. Per-instance mean cellprob is a one-line `ndimage.mean`.
- **Does micro-SAM expose a predicted-IoU score per instance?**
  - **Only in AMG mode.** `AutomaticMaskGenerator.generate(output_mode="binary_mask")` returns `predicted_iou` and `stability_score` for each mask.
  - **AIS (the recommended mode for `*_lm` models) has no IoU score.** It is a watershed on decoder foreground / centre / boundary distance maps. The closest per-instance signal is the mean foreground probability.
  - **Gotcha:** with micro-SAM's default AMG thresholds (`pred_iou_thresh=0.88`, `stability_score_thresh=0.95`), `vit_b_lm` returns **0 masks** on LIVECell, including through micro-SAM's own `automatic_instance_segmentation` (debug job 3073252). The max predicted IoU was 0.91–0.93.
  - The run above used 0.7 / 0.8. They were picked by eye on one test image, so AMG numbers are **not** held-out. This itself is a calibration finding worth a sentence in D5.
- **Is CellSAM usable?** **Yes, as of 2026-09-29.** The user supplied a DeepCell key; it is read from a private, git-excluded file and redacted from job logs.
  - `pip install git+https://github.com/vanvalenlab/cellSAM` into `d5` with torch pinned. Package 0.x, code Apache-2.0.
  - Weights: `cellsam_general` v1.2, 290M parameters, a 1.7 GB tarball, cached via the symlink `~/.deepcell → ~/vis4ml_spikes/hf_cache/deepcell`.
  - **Correction:** CellSAM is **SAM-based** (a SAM ViT image encoder plus an AnchorDETR "CellFinder" box prompter), so it is *not* a non-SAM control. Our deep-dive already listed micro-SAM ↔ CellSAM as a same-lineage pair.
  - The true different-architecture control is still **Cellpose3 `cyto3`**, a U-Net that needs its own env.

## CellSAM added (final run SLURM 3073309; zero-shot on LIVECell, default settings)
- Speed: 0.32–2.06 s per image; peak 3.5 GB torch / 4.4 GB nvidia-smi.
- Pooled over the 8 images: 896 predictions, **707 TP, 956 misses, 339 GT cells in merges, 25 FP (F1 about 0.49).**
- **Bimodal by cell type:**
  - F1 **0.92** on BV2 and SkBr3.
  - F1 **0.02–0.31** on A172, BT474, Huh7, MCF7, SHSY5Y and SKOV3. There the box detector returns only 13–40 cells per image, and it is not a query cap: `num_query_position` is 3500.
  - Not tuned: `bbox_threshold` 0.4 and `cellsam_extra` weights untested. Consistent with MIDL 2026 (Archit & Pape), which ranks CellSAM below the others.

**Error consistency, all pairs** (`work/a1/out/error_consistency.csv`; error = any non-TP status):

| Pair | Error rates | Cohen's κ | P(b wrong \| a wrong) | Silent failures | ρ(agree, a IoU), both detected |
|---|---|---|---|---|---|
| Cellpose-SAM vs micro-SAM AIS | 0.18 / 0.28 | **0.52** | 0.80 | 113 | 0.62 |
| Cellpose-SAM vs CellSAM | 0.18 / 0.65 | 0.19 | 0.96 | 25 | 0.36 |
| micro-SAM AIS vs CellSAM | 0.28 / 0.65 | 0.27 | 0.92 | 48 | 0.44 |
| Cellpose-SAM vs micro-SAM AMG | 0.18 / 0.66 | 0.19 | 0.97 | 42 | 0.51 |
| micro-SAM AIS vs AMG (same weights) | 0.28 / 0.66 | 0.31 | 0.98 | 81 | 0.47 |
| micro-SAM AMG vs CellSAM | 0.66 / 0.65 | 0.40 | 0.79 | 36 | 0.65 |

SYNTHESIS, 8 images:
- κ is chance-corrected for each model's error rate, but pairs with very unequal accuracy still have limited room for κ. The weak models (CellSAM, AMG) err on about 65% of cells, so their conditional-error rates near 0.96 are mostly base rate.
- The informative comparison is between **accuracy-matched pairs**: Cellpose-SAM vs AIS (κ 0.52) and AMG vs CellSAM (κ 0.40).
- For RQ2, report κ alongside each pair's accuracy gap, as in Geirhos et al.'s "error consistency", or compare on cell types where both models are competent (e.g. BV2 and SkBr3 for CellSAM).

## Caveats
1. Eight images is a smoke test. The numbers above are not results.
2. Leakage: micro-SAM `vit_b_lm` and Cellpose-SAM both trained on LIVECell *train*; test is held out.
3. The merge/split rules are a first definition. At IoU exactly 0.5 a half-cell fragment still counts as TP; use `>` 0.5 or a split-aware matching before any real analysis.
