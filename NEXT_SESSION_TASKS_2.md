# Next-session tasks (round 2): ambitious-version feasibility spikes for D5 and D4

**For:** a fresh Claude Code session **on the BC Andromeda cluster**. The user has narrowed to **D5 (microscopy) and D4 (astronomy)**, including their ambitious versions if things go well.

**Read first:**
- `research_gap_review.md` §3b and §4 (#1 D5, #3 D4)
- `lit_notes_open/check_d5_error_hierarchy.md`
- `lit_notes_open/check_d4_ambiguity_explanations.md`
- `spike_results/A1_microscopy_seg.md` and `spike_results/A4_zoobot_gz.md`: the previous spikes. Reuse their envs and scripts.

## Operational rules (same as the last cluster run)
- **Never run anything heavy on the login node.** Every SLURM job must include `--mail-type=BEGIN,END,FAIL --mail-user=zhangdjr@bc.edu`.
- **Ask the user before:** jobs longer than 1 h, downloads larger than 2 GB, or system installs. The full **GZ DESI (17.5 GB)** download is **not** approved yet. Do not fetch it.
- **Partitions:** `-p gtml -A gtml` (grs001, 4× L40S 46 GB, no time limit) or `-p weilab -A weilab` (gdw001, 4× A10 23 GB). A100 nodes may exist; ask the user for the partition name. Compute nodes have internet.
- **Workspace:** `/projects/weilab/zhangdjr/vis4ml_spikes` (symlinked as `~/vis4ml_spikes`).
  - Reuse `envs/d5` (torch 2.7.1 cu126, cellpose 4.2.1, micro_sam, zoobot 2.9.0, CellSAM).
  - `pip install` into d5 must use `-c jobs/d5_constraints.txt`.
  - `jobs/common.sh` redirects caches.
- **The DeepCell token** is in `<repo root>/private`. Never print it or commit it.
- **Where results go:**
  - Write `spike_results/B1_d5_hierarchy.md` and `spike_results/B2_d4_ambiguity.md`.
  - Append a short "Spike round 2" table to `research_gap_details.md` §12.
  - Update the D5 and D4 cards in `research_gap_review.md` §4 in one or two lines each.
- **Git:** commit and push when done. The user allows it. Push over HTTPS if SSH fails.
- **Label claims** FACT / SYNTHESIS / SPECULATION.

---

## B1. D5 ambitious: does a seed-to-lineage error hierarchy show up per cell? (about 1–2 h of wall time)

**Goal:** check *feasibility* and get a *first signal*. This is not a result. The claim under test (from `check_d5_error_hierarchy.md`): per-cell error consistency (κ) should rise from cross-lineage pairs to seed copies. The paper-worthy question is whether SAM-sharing pairs behave like seed copies.

1. **Set up a non-SAM model** (it is needed anyway):
   - Create env `d5c3` with Cellpose 3.x (cyto3). Cellpose 4 cannot load cyto3.
   - Check whether a LIVECell-specific Cellpose model (e.g. `livecell_cp3`) is downloadable, and record the result.
2. **Seed copies:** fine-tune **3 seeds** of one model on a *small* LIVECell train subset, e.g. 200 images, a few epochs, on one L40S.
   - Preferred: Cellpose-SAM via `cellpose` training, if supported and fast.
   - Otherwise: cyto3 in `d5c3`.
   - Record time per seed.
3. On the same **8 LIVECell test images** as A1 (2,006 GT cells), compute per-cell error indicators. Use A1's matching code at IoU ≥ 0.5, with error types miss / merge / split / FP.
4. **Cohen's κ** on per-cell error indicators, with image-level bootstrap 95% CIs, for:
   - (a) seed–seed pairs
   - (b) Cellpose-SAM vs micro-SAM, both SAM-backbone (A1 found 0.52)
   - (c) SAM-based vs cyto3 (non-SAM)
   - (d) CellSAM vs the others, if cheap

   Report pairwise accuracies alongside κ, because κ depends on accuracy.
5. **Also report** the within-image AUROC of pairwise agreement for detecting a cell's error, for each pair type.
6. **Pass criteria:**
   - 3 seeds trained;
   - κ table with CIs for (a)–(c);
   - total student-facing effort estimate for the full version (A1 estimated about 20–22 h).

## B2. D4 ambitious: can we measure explanation quality against GZ3D masks and a brightness baseline? (about 1–2 h of wall time)

**Goal:** check *feasibility* and get a *first signal* for "does human vote entropy predict explanation unreliability, beyond model uncertainty?" Follow the experiment sketch in `check_d4_ambiguity_explanations.md`, **scaled down**.

1. **Pick about 100 galaxies** in GZ3D (SDSS DR17 SAS) that also have GZ DESI/DECaLS vote counts. Use the Zenodo vote catalogues (CC BY 4.0, no images), not the 17.5 GB image set.
2. For each galaxy, fetch a **Legacy Survey DR10 cutout** by RA/Dec, as in A4. Align the GZ3D bar/spiral masks (A4 showed about 1 px alignment).
3. **Model:** reuse or retrain the A4 Zoobot head on the `tiny` config. Train 2–3 seeds if quick, so there is some model uncertainty to measure.
4. **Attributions:** for the "bar" or "spiral" question, compute Integrated Gradients, Grad-CAM and occlusion (plus SmoothGrad if easy), with Captum.
5. **Metrics per galaxy:**
   - Cross-method rank agreement: Spearman over pixels, or top-k IoU.
   - Mask localization: pointing game, or energy inside the volunteer mask. Compare it against a **light-profile baseline**, i.e. the same metric computed on the image-brightness map. Report the "gain over brightness".
   - Volunteer vote entropy for that question.
   - **Mask consensus**: number of GZ3D drawers for that mask, plus pixel-level agreement of the drawings (the FITS layers are volunteer counts). This is a control for the mask–ambiguity confound: ambiguous galaxies may just have noisier masks.
   - Model uncertainty: predictive entropy, and seed spread if multiple seeds exist.
6. **First signal:**
   - Spearman ρ between human entropy and each explanation metric, with and without controlling for model uncertainty **and mask consensus** (a partial correlation is fine). Also report the mask-free outcomes (cross-method and cross-seed agreement) separately.
   - **Two-sided.** Jukić 2023 found *higher* agreement on ambiguous inputs.
7. **Pass criteria:**
   - The pipeline runs on 100 galaxies.
   - Attributions beat the brightness baseline on at least some fraction of galaxies. If attributions never beat brightness, **flag it prominently**, because that is a key risk.
   - First-signal table produced.
   - Estimate of the full-version effort (about 21 h) and whether the 17.5 GB download is really required.

## At the end
Summarize for the user, in 5 bullets:
- whether each ambitious version still looks feasible;
- any red flags;
- the recommended pitch (D5 vs D4 as the lead) based on the signals.
