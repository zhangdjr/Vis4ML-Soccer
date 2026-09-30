# Research-Gap Review (short version)

**Project:** solo final project for NYU DS-GA 3001 *Visualization for ML* (Prof. Claudio Silva), Fall 2026.
**Updated:** 2026-09-29, after three literature rounds and the feasibility spikes.
**Full evidence:** `research_gap_details.md`. The §-numbers below point there.

> **Labels.**
> - **FACT:** verified against a source.
> - **SYNTHESIS:** my inference.
> - **SPECULATION:** a guess.
>
> "No paper found" always means *as far as our searches reached*.

---

## 1. The short story

- **The goal:** a project that gets a **good grade first** and could become a **workshop or short paper** second. It has to fit about 40 hours of solo work and use open data only.
- **What we did:** three literature rounds with 19 agents, covering soccer, about 20 open-topic areas, and 5 science domains. Seven areas got deep dives. Then feasibility spikes ran on the HPC cluster, and all four passed.
- **The main lesson:** Silva's lab has already published the obvious idea in most Vis-for-ML areas. Examples include explanation-disagreement tools (Visagreement, MOUNTAINEER), calibration (Calibrate), forecasting visualization (mTSeer), and the default Tile2Net project. The winning move is to **extend one of their tools on a question their papers leave open.**
- **The gaps that are left are narrow and measurable:** an assumption nobody has tested, or a failure that leaderboards average away. That is why realistic upside tops out at a **workshop or VIS short paper**, which fits a course project.

## 2. Course facts that matter

- **Deadlines:** 4-page proposal **Oct 20**; 1-page update Nov 3; presentations Dec 1 and 8; 8-page report (IEEE VIS format) **Dec 14**. The project is 45% of the grade.
- **Format (syllabus, verbatim):** "reproduce prior work **or** implement a proposed research idea of your choosing". It also says: "demonstrate both the prior work, and your final research project, to the class". Solo is allowed.
  - **Correction (2026-09-29):** earlier versions of this review read that as "reproduce **and** extend". That reading was stronger than the text. A reproduction is **not strictly required**. Every candidate below still includes a small one, because it validates your pipeline and gives you "prior work" to demonstrate. Treat it as optional polish. Your reading of the "demonstrate" sentence: you present whichever path you chose (reproduction or your own idea).
- **Front end:** **Silva confirmed JS/D3 is not required.** Streamlit/Plotly is fine, as long as the topic fits the class.
- **Oct 6** lecture is the project discussion. Pitch there.

## 3. Rankings: two views

There are **two rankings** over the same evidence:
- **3a (grade-first):** safest path to a strong course project.
- **3b (ambitious):** best shot at a paper with Silva continuing as advisor after Dec 14.

Ratings are qualitative (SYNTHESIS from the deep dives and spikes). Reasons are in §4 and in `research_gap_details.md` §5 and §9.

> **Compute is not a constraint (updated 2026-09-29).** You have a SLURM cluster with many L40S (46 GB), A10 (23 GB) and **A100** GPUs. So GPU cost is rated as *your hours to set up and debug*, not GPU time. **The binding constraint is your 3–4 h/week.** Unattended overnight jobs are nearly free.

**Field key:**
- **Upside:** research and publication potential if things go reasonably well.
- **Grade safety:** probability of a solid course deliverable.
- **Difficulty:** technical and implementation difficulty for you.
- **Compute:** GPU and disk burden.
- **Viz burden:** front-end effort.
- **Scoop risk:** chance someone publishes it first.
- **Silva fit:** how naturally it connects to the instructor's work.
- **Interest:** your stated interests.

### 3a. Grade-first top 5 (safest path to a strong course project)

| # | Candidate | Upside | Grade safety | Difficulty | Compute (your setup effort) | Viz burden | Data risk | Scoop risk | Silva fit | Interest | Main risk |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **1** | **D5:** do cell-segmentation models share mistakes, and does agreement flag bad cells without GT? (microscopy) | Med (High if shared-error result holds) | **High** | Med | Low (<2 s/img) | Low–Med | Low–Med (NC licenses) | **Low** (0 citers; RBQE partial) | High (tests Visagreement's conjecture) | High (your segmentation skills) | Agreement may just track difficulty |
| **2** | **D2:** where do time-series foundation models fail, window by window? | Med | **High** | Low–Med | **None** (released outputs) | Low–Med | Low | Med–High (fast field; Wang et al.) | Med–High (mTSeer) | Low | Thin novelty; a null result is plausible |
| **3** | **D4:** Zoobot uncertainty and explanations vs. volunteer disagreement (astronomy) | Med (High if RQ-B works) | Med–High | Med | Low (3 s/epoch) | Low–Med | Low (NC-SA + code-release clause) | Med (data owners; lab image extension) | High (Calibrate, Visagreement) | **High** (astronomy) | Brightness confound |
| **4** | **D3:** why LLM judges disagree with each other and with humans | Med− | **High** | Low–Med | Med (~4.7 h, 32 GB of models) | Med | Low | **High** (PAIR/KAIST/IBM; lab text extension) | Med–High | Med | "Not novel enough" |
| **5** | **Soccer C1:** per-shot xG explanation disagreement vs. physics | Med | Med–High | Low–Med | Low | Med | Low | Low–Med | High (Visagreement) | **High** (soccer) | Methods may simply agree on distance/angle; not spiked |

**Honorable mentions:**
- **D1 Tile2Net** (the default project): best Silva fit, but crowded and install untested.
- **Soccer C2**: the safest soccer option, but the visualization plays a supporting role.
- **EO spatial-shift diagnosis** and **tabular-FM context attribution**: scan-level only.

### 3b. Ambitious top 5 (best shot at a paper; Fall 2028 PhD cycle)

**Target timeline:**
1. A strong course deliverable by **Dec 14, 2026**.
2. Silva advises the extension, **Jan–spring 2027**.
3. Submit in **spring or summer 2027**.
4. A decision by about **fall 2027**, before your **Dec 2027** applications.

**Strategy:** every option keeps a **grade-safe floor**, finished by the Nov 3 update, so a failed ambitious half still yields a strong project and a reportable negative result.

| # | Ambitious version | Upside | Grade safety (with floor) | Difficulty | Compute | Extra hours vs. 40 | Scoop risk | Floor if it fails | Paper needs (post-course) | Plausible venues (verify deadlines) |
|---|---|---|---|---|---|---|---|---|---|---|
| **1** | **D5 full:** a correlated-error audit across 4+ models (incl. non-SAM) and 2–3 held-out datasets, plus GT-free triage evaluated by simulated inspection. **With the cluster, it can add seed-level ensembles** (see the D5 + Rashomon note below) | **Med–High** (High with the combined version) | Med–High | Med–High | Low | +25–35 h | Low | Error taxonomy + consistency matrix on LIVECell | A third dataset; κ with CIs across model lineages; optional biologist feedback | CVPR/MICCAI microscopy workshops; IEEE VIS short; ISBI |
| **2** | **D4 RQ-B:** does *human* ambiguity predict *explanation* unreliability? (GZ3D masks, beating a brightness baseline) | **Med–High** (the most distinctive story) | Med | Med–High | Low | +25–35 h | Med (data owners; lab image extension) | Small-N calibration audit (RQ-A) | Several morphology questions; comparison vs. ZooBot:3D | NeurIPS ML4PS; RAS Techniques & Instruments (rolling); VIS short |
| **3** | **Rashomon/multiplicity VA for segmentation:** where seed-varied models disagree, and whether that tracks annotation ambiguity (pairs naturally with D5) | Med–High | Med | High (training pipeline) | **Low** (10–20 seeds = one overnight run on the cluster) | +30–40 h | Low–Med | Seed-variance maps on one dataset | A link to annotation ambiguity; a second dataset | VIS short; CVPR/ICCV workshops; the VIS Uncertainty workshop |
| **4** | **D2-max:** the in-vivo test of Jander's TSFM failure modes, plus failure *prediction* on held-out datasets | Med | High | Med | None–Low | +20–30 h | **Med–High** | Variance decomposition + a failure predictor | A second benchmark (GIFT-Eval); a dose-response analysis | ICLR TSFM workshops; IEEE VIS short; IJF |
| **5** | **Soccer C1 full:** per-shot disagreement across methods and models on several tournaments, checked against physics ground truth | Med (smaller venues, higher acceptance odds) | Med–High | Med | Low | +20–25 h | Low–Med | Global + semantic-perturbation analysis on one tournament | Several tournaments; a Rashomon set of xG models | MLSA @ ECML-PKDD; JQAS; J. Sports Analytics |

**Novelty checks on the ambitious versions (2026-09-29, cross-checked 2026-09-30 by 3 external LLM runs, all agreeing; details in `lit_notes_open/check_d5_error_hierarchy.md` and `check_d4_ambiguity_explanations.md`):**
- **D5 ambitious: partly falsified, and the core survives.**
  - *Done for classification:* Gontijo-Lopes et al. (ICLR 2022, full text) show that errors decorrelate from seeds → hyperparameters → architecture → objective → data.
  - *Partly done for segmentation*, but only per image and with at most 2–3 levels: RBQE; Kirscher et al. 2026 (read in full: seed vs CV-fold ensembles, image-level, semantic segmentation, no error-correlation metric, no microscopy); Zenk 2024.
  - **Open:** *per-cell* error consistency (κ with bootstrap CIs) across seed → fine-tuned → shared-SAM-backbone → non-SAM → different-data pairs, linked to agreement-QC AUROC and silent failures.
  - The paper hook is where segmentation **departs** from the classification ordering, e.g. SAM-sharing pairs behaving like seed copies.
  - Must-cite: Gontijo-Lopes 2022, Geirhos 2020 (error consistency), Klein 2025, Saxena 2024, RBQE, Kirscher 2026, Zenk 2024, **arXiv 2512.15921** (GT-free concordance of 6 CT segmenters), and **Comput. Biol. Med. 2023** (ensembles that "largely agree on mistakes"). So frame silent failures as *quantified across the hierarchy*, not as newly discovered.
- **D4 ambitious: partly falsified, and the core survives.**
  - *Done:* the **model's own** uncertainty predicts explanation unreliability (Mikriukov et al. 2026, Saporta 2022 / CheXlocalize, Bhatt et al.). Jukić 2023 finds saliency agreement is *higher* on model-ambiguous inputs, so **test two-sided**.
  - **Open (moderate evidence after a 2026-09-30 retry:** citation chaining over about 850 citers of 6 key papers, 6 targeted searches, and 2 new Sept-2026 preprints checked, none of which tests explanations**):** *human* vote entropy as the predictor, **beyond** model uncertainty. Cite Singh & Pakrashi 2026 (model uncertainty vs human ambiguity, no explanations). A Google Scholar title check of CIFAR-10H's ~500 citers, done by you on 2026-09-30, found no threat. Evidence is now **moderate-to-good**. The galaxy angle is strong for three reasons:
    - GZ3D gives label *and* spatial disagreement. That is rare but not unique; LIDC-IDRI and Gleason19 also have both.
    - Zoobot is trained on vote counts, which makes this the strictest version of the test.
    - Brightness gives a built-in baseline that attributions must beat.
- **Next:** two small cluster spikes (B1 for D5, B2 for D4) in `NEXT_SESSION_TASKS_2.md`. The full D4 version will need approval for a **17.5 GB** GZ DESI download; the spike does not.

**The combined D5 + Rashomon idea (SPECULATION, strongest ambitious framing; cheap with the cluster):**
- Measure *where* segmentation errors stop being shared, along a hierarchy of disagreement: same model with different seeds → fine-tuned variants → different architectures (SAM vs non-SAM) → different training data.
- This turns D5's "are errors correlated?" into **"at what level do errors become independent, and so when is agreement-based QC trustworthy?"**
- The cost is your hours for one training pipeline (fine-tuning Cellpose/micro-SAM with seeds). GPU time is overnight jobs.
- **Floor:** D5's cross-model audit alone.

**Dropped from the ambitious view:** D3, because of crowded scoop exposure from big labs, and D1, because it is the default project and classmates overlap.

**The biggest lever for a paper is not the topic.** It is asking Silva on Oct 6 whether he would advise an extension if the course result is strong. #1, #2 and #5 can each be pitched as "a rigorous, GT-based test of your lab's Visagreement conjecture", which makes his involvement natural.

**For Oct 6:**
- **Grade-first:** pitch #1 plus one or two of 3a's #2–#4.
- **Ambitious:** lead with **D5**, and offer **D4 RQ-B** as the higher-risk, more distinctive alternative.

## 4. The candidates in one screen each

### #1 D5: Ground-truth-free QC for generalist cell segmentation
- **Question:**
  - Does per-cell agreement between models (Cellpose-SAM, micro-SAM, CellSAM, and a non-SAM model) rank segmentation quality without ground truth?
  - Do models that share a backbone share their mistakes?
  - Where are the "silent failures" (all models wrong in the same way)?
- **Why it is open:**
  - BISCUIT, an agreement-based model-picking tool, *assumes* errors are uncorrelated. A reviewer (Bankhead) asked whether that holds for models with overlapping training data, and got no answer (FACT).
  - The closest precedent, **RBQE**, works per image on polyps, not per cell in microscopy (must cite).
- **Lab hook:** Visagreement's Case Study 2 only *conjectures* that disagreement goes with errors. D5 tests an analogous claim rigorously against ground truth.
- **Reproduce:** a NeurIPS22 CellSeg F1 number and a Cellpose-SAM number on LIVECell.
- **Minimum project:** per-cell error types (merge / split / miss / false positive), an error-overlap matrix across models, and attribute plots on LIVECell.
- **Stronger version:** QC ranking scored with within-image AUROC, plus a triage view scored by "errors found per K cells inspected".
- **Spike (PASS):**
  - Both models run in under 2 s per image.
  - Early signal from 8 images: error κ = 0.52 (κ is chance-corrected agreement), 80% of errors shared, 113 silent failures.
- **Spike round 2 (B1, PASS):**
  - 3 fine-tune seeds, plus cyto3 and livecell_cp3, on 40 images (10,292 cells).
  - Per-cell κ: seeds **0.92** > checkpoint variant 0.88 > **SAM vs non-SAM (both Cellpose lineage) 0.79** > **SAM vs SAM (Cellpose-SAM vs micro-SAM) 0.58** [0.52, 0.62].
  - So **shared lineage, not the shared SAM encoder, predicts shared errors**; H-mono is not supported. Agreement-QC AUROC rises as κ falls.
- **Still needed:** a run on **NeurIPS22 Public-Test**, the clean held-out set. LIVECell test is in-distribution for all models except CellSAM.
- **Details:** `research_gap_details.md` §5.1; `lit_notes_open/deep_5_microscopy_qc.md`; `spike_results/A1_microscopy_seg.md`.

### #2 D2: Window-level failure regimes of time-series foundation models
- **Question:** where do zero-shot forecasters (Chronos-2, TimesFM-2.5, and others) fail per forecast window, compared with classical baselines and with each other? Can context features *predict* those failures on held-out datasets? Do the synthetic failure modes reported by Jander et al. show up in real data?
- **Why it is open:**
  - The TIME benchmark (ICML 2026) only stratifies statically.
  - Jander et al. ask for real-data validation.
  - mTSeer (Silva lab) lists this as future work.
  - Wang et al. 2026 already show that aggregates hide failures (traffic data), so **lead with prediction plus the in-vivo test**.
- **Reproduce:** TIME's tables. Already done in the spike, within 0.1%.
- **Minimum project:** classical baselines, context features, a leave-dataset-out failure predictor, and 5 linked views. No model inference needed.
- **Spike (PASS, with a correction):** the old pilot joined rows in the wrong order. The fix is to join by name. Within-series variance share is 0.66 (MASE) vs 0.45 (CRPS).
- **Details:** `research_gap_details.md` §5.2; `lit_notes_open/deep_2_tsfm_va.md`; `spike_results/A2_time_tsfm.md`.

### #3 D4: Zoobot vs. Galaxy Zoo volunteer disagreement
- **Question:**
  - **Safe:** is Zoobot calibrated to volunteer vote splits, once small vote counts are accounted for? Which kind of model uncertainty tracks human disagreement?
  - **Ambitious:** do explanation maps land on volunteer-drawn GZ3D bar and spiral masks less often when volunteers disagree?
- **Why it is open:** the Zoobot papers report only an aggregate "noise floor", and nobody has compared galaxy-CNN explanations against GZ3D masks (FACT, verified).
- **Reproduce:** Galaxy Zoo DESI Fig. 5 and Galaxy Zoo DECaLS Fig. 19.
- **Spike (PASS):** 3 s per epoch; masks align to about 1 px. Bar pixels are 30× brighter than the image mean, so any explanation test **must beat a brightness baseline**.
- **Validity risk (2026-09-30):** GZ3D masks are volunteer-drawn too, so vote ambiguity may go with noisier masks. Control for **mask consensus** (number of drawers, overlap), and lean on mask-free outcomes (cross-method and cross-seed agreement, deletion faithfulness).
- **Spike round 2 (B2, PASS with a red flag):**
  - 100 GZ3D galaxies with independent votes, 3 seeds, 4 attribution methods.
  - **Brightness alone localizes bar masks better** (median AUPRC 0.83 vs 0.29–0.44). Attributions beat it on only 10–17% of galaxies and add just +0.005 to +0.02 over a light-profile model.
  - Human vote entropy *raises* method agreement (as in Jukić), but this vanishes after controlling for model uncertainty. There is one uncorrected hint that ambiguity lowers the gain over the light profile (partial ρ −0.27).
- **Details:** `research_gap_details.md` §5.3; `lit_notes_open/deep_4_astro_zoobot.md`; `spike_results/A4_zoobot_gz.md`.

### #4 D3: Decomposing LLM-judge disagreement
- **Question:** with local judges plus GPT-4 on MT-Bench items that have several human votes, split judge–human disagreement into four sources: human ambiguity, judge instability, feature-linked bias, and rationale divergence. Show that the tool recovers biases you planted on purpose.
- **Why it is open:** LLM Comparator handles a single judge. The 2026 statistics papers each cover one layer.
- **Spike (PASS):**
  - MT-Bench agreement reproduced exactly (66.0 / 84.6 / 63.0 / 81.0%).
  - The minimum run takes about 4.7 h on one L40S.
- **Details:** `research_gap_details.md` §5.4; `lit_notes_open/deep_3_llm_judges.md`; `spike_results/A3_llm_judges.md`.

### #5 Soccer C1: Local, physics-anchored explanation disagreement for xG
- **Question:** do explanation methods and models disagree *per shot*, and does that disagreement flag explanations that break shot geometry (mirror symmetry, distance monotonicity)?
- **Why it is open:**
  - Cefis & Carpita 2024 compared xG explanations only *globally*.
  - 0 of 25 soccer explanation papers test per-shot agreement.
  - Visagreement measured explanation quality only through proxy scores.
- **Details:** `research_gap_details.md` §5.6; `lit_notes/round1_soccer_review.md` §11.

## 5. What is ruled out (and why, in one line each)

| Area | Why not |
|---|---|
| SHAP vs LIME vs IG dashboards (any domain) | Visagreement, MOUNTAINEER and Explainalytics (Silva's lab) |
| LLM explains a model, then check faithfulness | Already done (2024–26 papers) |
| Embeddings → UMAP → "discover clusters" | Done repeatedly; a known misuse of UMAP |
| Tile2Net pixel↔network debugging tool | Last year's cohort built it |
| SAE dashboards; Mapper/TDA for explanations | Scooped in 2026; Silva's own line of work |
| Student-success explainer disagreement | Silva's lab published it 4 times |
| "Is LLM grading reliable?" | Several papers per month |
| "Stratify forecasting errors by seasonality" | TIME (ICML 2026) did it |
| "Foundation models degrade under geographic shift" | Crowded 2026 preprints |
| NFL Big Data Bowl | Data terms forbid non-contest use |
| Full list, about 35 areas | `research_gap_details.md` §4 and §8 |

## 6. Your open tasks

**This week:**
1. Read this file. For your top picks, skim the matching `research_gap_details.md` §5 sections.
2. Choose 2–3 candidates to pitch.
3. Decide whether to make the GitHub repo **private**. It is public now. I recommend private.

**Oct 6: ask Silva:**
- Which pitched topic he prefers.
- Whether extending his lab's tools is welcome (Calibrate, Visagreement, mTSeer).
- **Whether Visagreement's planned image/text extension is already in progress.** This matters for D4 and D3.
- Confirm your solo status.
- **Whether he would advise a paper extension after Dec 14** if the course result is strong (you are targeting Fall 2028 PhD applications).
- *(Optional, quick confirm)* **"Demonstrate both the prior work, and your final research project":** your reading (2026-09-29) is that you demo whichever path you chose. Nothing in the plan depends on this.
- *Only if D1:* which NYC areas Tile2Net was trained on.

**Optional:** ask the TA for last year's project list; skim BISCUIT's open reviews, which are the D5 hook.

**After Oct 6:**
- Write the proposal by Oct 20. I can draft it from the deep-dive notes and spike results.
- Remaining technical steps on the HPC, depending on your pick:
  - D5: cyto3 plus a Public-Test run.
  - D3: the full run, which needs approval.
  - D2: a weekly scoop check.
- Disk: the cluster folder is 49 GB, 32 GB of it Ollama models. Delete those if you drop D3.

## 7. Where everything lives

| File | What's in it |
|---|---|
| `research_gap_details.md` | The full evidence: method, limits, lab-overlap map, all ~35 areas, candidate details, higher-upside tier, literature table, comparison tables, verification results, spike results, corrections, Visagreement full-text notes |
| `spike_results/*.md` | Per-spike commands, numbers and timings (A1–A4), dblp and license checks, scoop checks |
| `lit_notes_open/deep_1…5_*.md` | Deep dives: full paper tables, search logs, project designs, hour budgets |
| `lit_notes_open/scan_1…10_*.md` | Broad scans per domain |
| `lit_notes/` | Round 1 (soccer) notes, plus the archived soccer review |
| `NEXT_SESSION_TASKS.md` | The brief used by the HPC spike session |
| Cluster: `~/vis4ml_spikes/` | Code, conda envs, data, weights and job scripts (not in the repo) |

*AI-use disclosure: produced with Claude (Opus 5.5) and sub-agents. Load-bearing claims were verified against primary sources (see `research_gap_details.md` §0.2 and §13.4). The research choices are yours to make and defend.*
