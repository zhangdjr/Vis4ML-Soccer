# Next-session tasks (round 4): "What makes a good reference for agreement-based QC?"

**For:** a fresh Claude Code session **on the BC Andromeda cluster**. Round 3 retired the "family, not encoder" headline. The new question is which reference model makes agreement-based QC work, and why: reference accuracy vs independence.

**Read first, in this order:**
1. **`PREREG_D5.md` Part B.** R1–R8, the confirmatory data N1–N3, and the kill rules. **Follow it exactly.** Changes go in dated amendments, labelled "before" or "after" seeing N1 results.
2. `spike_results/C1_heldout_replication.md`, including the **"Lead-reviewer notes"** at the end. Then C2–C4.
3. `spike_results/K7_scoop_check.md`.

Reuse round-3 code in `~/vis4ml_spikes/work/c1/`: `c1_match.py`, `c1_cells.py`, `c1_stats.py` and `c1_infer.py`.

## Operational rules (unchanged)
- **Nothing heavy on the login node.** Every SLURM job includes `--mail-type=BEGIN,END,FAIL --mail-user=zhangdjr@bc.edu`.
- **Ask the user before:** jobs longer than 1 h, downloads larger than 2 GB, or system installs.
- **Partitions:** `-p gtml -A gtml` (4× L40S) or `-p weilab -A weilab` (4× A10).
- **Environments:**
  - `envs/d5`. `pip install` into it must use `-c jobs/d5_constraints.txt`.
  - Cellpose 3: set `PYTHONPATH=envs/d5c3_overlay`.
  - `jobs/common.sh` redirects the caches.
- **DeepCell token:** in `<repo root>/private`. Never print it or commit it.
- **Licenses:** commit no pixels, masks or overlays derived from any ND-licensed data. Check N1's license too.
- **Outputs:**
  - one file per task: `spike_results/D1_…md` to `D6_…md`;
  - a "Spike round 4" table in `research_gap_details.md` §12;
  - a short "Round 4" subsection in `D5_onboarding.md` §4.
- **Label claims** FACT / SYNTHESIS / SPECULATION.
- **Commit and push when done.**
- **Language:** call Public-Test a "held-out test split", not "OOD". Cellpose-SAM and micro-SAM trained on NeurIPS22 Training.

---

## D0. Pick and freeze N1 (do this first; commit before running any model on it)
- Search for a dataset meeting the PREREG Part B §B1 rules. Check each candidate against the training-data lists in the papers of:
  - Cellpose-SAM v1 and v2;
  - cyto3;
  - micro-SAM (`vit_b_lm`, `vit_l_lm`);
  - CellSAM.
- Write `spike_results/D0_n1_choice.md`:
  - the candidates;
  - the per-model leakage evidence, with quotes and page numbers;
  - the license;
  - size;
  - the choice.
- **Commit and push it before any inference on N1.** That commit is the pre-registration record.
- Also freeze N2 (200 fresh LIVECell test images, disjoint from LC200 and E40): write the file list, then commit.

## D1. Exploratory re-analysis of round-3 data with the new measures (fast; no new inference)
- log-OR and Yule's Q with image-bootstrap CIs for all pairs, on Public-Test and LC200. Recompute H1 under log-OR.
  - The reviewer's back-of-envelope from rounded rates gave OR about 17 vs 6 (PT) and 95 vs 23 (LC200). Check it.
- For every directed pair, compute e_r, o and f (Part B §B2) with AUROC and recall@5%. Then fit R1's two models on round-3 data (exploratory).
- Deliverable: **a scatter of recall@5% against f (x) and o (colour)** for Cellpose-SAM as the target, both datasets.
- This is a dry run of R1. Say whether the pattern looks like it will hold, but **do not change Part B** based on it except via a dated amendment.

## D2. New models (the R3 contrast and Cellpose-SAM v2)
- Add micro-SAM **`vit_l_lm`** (AIS).
- Add **Cellpose-SAM v2** (the June 2026 release), if it can be installed without breaking `envs/d5`. If not, make a separate env and record it.
- **FACT-check each model's encoder from the installed code**, as round 3 did: architecture (ViT-B/L/H) and pretrained checkpoint source. Put a table in D2.
- Run all models (round-3 roster + these two) on **N1 and N2**.

## D3. Confirmatory tests on N1 (R1–R4, Holm) and secondary R5–R8
- Follow PREREG Part B exactly.
- **R7 operating-point matching:** sweep Cellpose-SAM's `cellprob_threshold` on N1 until its error rate matches micro-SAM `vit_b_lm`'s within 0.01.
- **R8:** strata = image × the number of other models missing the cell.
- **Holm across R1–R4.** Report supported / falsified, and what each B5 kill rule implies.

## D4. Controlled distribution shift (R4, N3): ask before submitting if > 1 h
- LIVECell leave-one-cell-type-out folds. Use **3 folds** (pick 3 held-out types spanning morphology, e.g. SH-SY5Y, BV2, SKOV3; record the choice before training). For each fold:
  - **own Cellpose-3 U-Nets from scratch:** 2 seeds on 200 train images from the 7 remaining types. Round-3 cost: about 17 min per model on an L40S;
  - **A second architecture that has never seen LIVECell:** 1–2 seeds on the same 200 images.
    - **Do not start from `cpsam`, cyto3 or micro-SAM `*_lm` checkpoints.** They were trained on all 8 LIVECell types, so the "held-out" type would not be held out.
    - Clean options, in order of preference:
      - (i) micro-SAM training from **vanilla SAM `vit_b`** (SA-1B natural images only);
      - (ii) the Cellpose-SAM architecture initialized from vanilla SAM ViT-L, if cellpose 4 supports it;
      - (iii) another from-scratch architecture, e.g. StarDist.
    - FACT-check that the initial weights have no LIVECell exposure, and record it.
- Evaluate on (a) in-type test images and (b) held-out-type test images, both from N2.
- Compute the R4 Δ. Estimate the total GPU-hours first. **Each job must stay under 1 h, or ask.**

## D5. Figures for the proposal
1. **Simplified pitch figure** `spike_results/fig_kappa_by_level_v2.png`:
   - 5 groups: seeds, fine-tunes, Cellpose-SAM vs cyto3, Cellpose-SAM vs micro-SAM ("shared SAM pretraining, ViT-L vs ViT-B"), micro-SAM vs CellSAM;
   - log-OR panel next to the κ panel;
   - datasets: LIVECell and Public-Test (aggregate stats only).
2. **Reference-QC figure** `spike_results/fig_reference_tradeoff.png`: the D1 scatter, and on N1 once D3 is done. This is likely the proposal's key figure.

## D6. Blunt verdict
- R1–R4: supported / falsified.
- R5–R8: one line each.
- Does "what makes a good reference" survive as the headline? If not, what does?
- Surprises, and any new oversight found (like round 3's ViT-L/ViT-B catch).
- Remaining risks for the proposal (due Oct 20).
