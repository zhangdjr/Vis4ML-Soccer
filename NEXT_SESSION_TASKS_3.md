# Next-session tasks (round 3): D5 kill tests and the held-out replication

**For:** a fresh Claude Code session **on the BC Andromeda cluster**. The user has chosen **D5** and wants to be confident it won't fail later from an oversight. This round runs the **pre-registered kill tests**.

**Read first, in this order:**
- **`PREREG_D5.md`**: the hypotheses, thresholds, exclusions and kill tests. **Follow it exactly.** Do not change a confirmatory definition. If something must change, add a dated entry under "Amendments" with the reason, and say whether it was made before or after seeing the Public-Test results.
- `D5_onboarding.md` §2 and §4–6 (concepts, pilot results, design).
- `spike_results/B1_d5_hierarchy.md`, including its "Lead-reviewer notes". Reuse B1's code: `~/vis4ml_spikes/work/b1/{b1_data,b1_train,b1_infer,b1_analyze}.py`.

## Operational rules (unchanged)
- **Nothing heavy on the login node.** Every SLURM job includes `--mail-type=BEGIN,END,FAIL --mail-user=zhangdjr@bc.edu`.
- **Ask the user before:** jobs longer than 1 h, downloads larger than 2 GB, or system installs. The L0a/L4 training jobs in C4 will likely need approval; ask before submitting them.
- **Partitions:** `-p gtml -A gtml` (grs001, 4× L40S) or `-p weilab -A weilab` (gdw001, 4× A10). Compute nodes have internet.
- **Environments:**
  - `envs/d5` (cellpose 4.2.1, micro_sam, CellSAM). `pip install` into it must use `-c jobs/d5_constraints.txt`.
  - Cellpose 3: set `PYTHONPATH=envs/d5c3_overlay`.
  - `jobs/common.sh` redirects the caches.
- **The DeepCell token** is in `<repo root>/private`. Never print it or commit it.
- **Results:** one file per task, `spike_results/C1_…md` to `C5_…md`. Append a "Spike round 3" table to `research_gap_details.md` §12. Update `D5_onboarding.md` §4 with a short "Round 3" subsection.
- **Commit and push when done** (HTTPS if SSH fails).
- **Label claims** FACT / SYNTHESIS / SPECULATION.

---

## C1. Held-out replication on NeurIPS22 Public-Test (K1; H1–H4). Highest priority
1. **Range-read** only the `Public/` images and labels (50 labeled images, excluding the 2 WSIs) from Zenodo 10719375 `Testing.zip`. Do not download the full 2.9 GB. Record modality per image if a metadata file exists; otherwise infer it (brightfield / fluorescence / phase / DIC) and label it SYNTHESIS.
2. **Run all models** with the PREREG settings:
   - Cellpose-SAM;
   - the 3 B1 fine-tune seeds (`work/b1/models/models/cpsam_ft_s{1,2,3}`);
   - cyto3 and `livecell_cp3`, each with **both** auto and GT-median diameters;
   - micro-SAM AIS;
   - CellSAM (descriptive only if its error rate is > 0.6).
   Save per-cell error indicators, and flows/probabilities for Cellpose (`flow_threshold=0` for QC signals).
3. Compute everything in PREREG §2 on Public-Test:
   - per-pair κ, κ/κ_max, accuracies, silent failures per 1,000 cells, within-image AUROC, all with **image-bootstrap CIs**;
   - the H1 difference CI;
   - the H3 ρ(κ, AUROC) CI;
   - H4 (agreement vs flow error vs an attribute-only model fit on LIVECell train);
   - a Holm correction across H1–H4.
4. Also rerun on **LIVECell test with 100–200 stratified images** (secondary).
5. Report clearly: **for each of H1–H4, supported or falsified, and what the kill rule K1 says.**

## C2. Robustness (K3, K4, K5)
- **K3:** κ(Cellpose-SAM, cyto3) with auto vs GT-median diameters.
- **K4:** κ/κ_max for all pairs; an **accuracy-matched** subset (images or cell types where both models' accuracies are within 0.05); H1 on that subset.
- **K5:**
  - an IoU threshold sweep (0.3 / 0.5 / 0.7) for the H1 pairs;
  - per-cell-type (LIVECell) and per-modality (Public-Test) κ;
  - leave-one-type-out for H1.

## C3. Difficulty-controlled null (K2). **Critical: the most likely hidden oversight**
1. Compute per-cell attributes: area, equivalent diameter, local density (neighbours within 2 diameters), touching fraction, local contrast.
2. Per model, fit an **attribute-only logistic model** of error probability. Fit it on LIVECell *train*-like data, or cross-fitted on the test set with image-grouped folds, so that there is no in-sample leakage. Record which option was used.
3. Bin cells into deciles of predicted difficulty (use the mean of the two models' predictions for a pair), then compute:
   - (a) **stratified κ** (the Mantel–Haenszel-style pooled κ across bins, or the mean of within-bin κ, stating which);
   - (b) a **permutation null**: shuffle each model's errors *within* difficulty bins, 1,000×. Report observed κ vs the null distribution.
4. Report whether the **H1 difference survives difficulty control** (CI excludes 0). Apply kill rule K2.

## C4. Mechanism: untangle "family" (H5, H6). Ask before long jobs
- **H5, decoding:**
  - micro-SAM **AMG** with thresholds tuned on LIVECell **train** (never test), vs **AIS** (same encoder and weights).
  - κ **by error type** (miss vs merge/split) for all main pairs.
- **H6, data vs architecture:** train Cellpose-3-architecture U-Nets **from scratch** in the `d5c3` overlay:
  - **L0a:** 3 seeds on the same LIVECell train subset.
  - **Clean L4:** 2 models on **disjoint** LIVECell train subsets (e.g. split by image), same seed schedule.
  - Estimate the GPU hours first, and **ask the user before submitting** if a job is over 1 h.
  - Then compute κ for L0a pairs, for L4 pairs, and between these U-Nets and Cellpose-SAM / micro-SAM.

## C5. Pitch figure (for Oct 6)
- One clean figure: **κ by relatedness level** with bootstrap CIs, LIVECell and Public-Test side by side. Save it as `spike_results/fig_kappa_by_level.png`, with a two-sentence caption.
- If there's time, a second one: **κ vs QC AUROC** per pair.

---

## At the end
Give the user a short **"Is D5 still safe?"** verdict:
- H1–H4 each: supported / falsified;
- K1–K6 each: passed / triggered, with what the kill rule implies;
- whether the headline claim should stay as "family, not encoder", or be reframed (e.g. "shared hard cells" or "decoding objective");
- anything that surprised you.

Be blunt. If a kill rule triggers, say so plainly.
