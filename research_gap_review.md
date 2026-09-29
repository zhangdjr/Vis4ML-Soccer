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

- **The goal:** a project that gets a **good grade first** and could become a **workshop or short paper** second. It has to fit about 40 hours of solo work, use open data only, and follow the course's reproduce-then-extend format.
- **What we did:** three literature rounds with 19 agents, covering soccer, about 20 open-topic areas, and 5 science domains. Seven areas got deep dives. Then feasibility spikes ran on the HPC cluster, and all four passed.
- **The main lesson:** Silva's lab has already published the obvious idea in most Vis-for-ML areas. Examples include explanation-disagreement tools (Visagreement, MOUNTAINEER), calibration (Calibrate), forecasting visualization (mTSeer), and the default Tile2Net project. The winning move is to **extend one of their tools on a question their papers leave open.**
- **The gaps that are left are narrow and measurable:** an assumption nobody has tested, or a failure that leaderboards average away. That is why realistic upside tops out at a **workshop or VIS short paper**, which fits a course project.

## 2. Course facts that matter

- **Deadlines:** 4-page proposal **Oct 20**; 1-page update Nov 3; presentations Dec 1 and 8; 8-page report (IEEE VIS format) **Dec 14**. The project is 45% of the grade.
- **Format:** you must reproduce prior work and extend it, and demo both. Solo is allowed.
- **Front end:** **Silva confirmed JS/D3 is not required.** Streamlit/Plotly is fine, as long as the topic fits the class.
- **Oct 6** lecture is the project discussion. Pitch there.

## 3. Top 5 ranking

**Weights:** (1) grade safety, (2) publication path, (3) your interests as the tie-breaker. Details are in `research_gap_details.md` §10.

| # | Candidate | Why it is here | Main risk |
|---|---|---|---|
| **1** | **D5: Do cell-segmentation models make the same mistakes, and does their agreement flag bad cells without ground truth?** (microscopy) | Objective (ground truth exists); answers an **unanswered peer-reviewer question**; no lab overlap; uses your segmentation skills; the spike shows the question is live | The result could be trivial: agreement might just track image difficulty. Needs one non-SAM model |
| **2** | **D2: Where do time-series foundation models fail, window by window?** | **Lowest execution risk:** the benchmark already publishes all model outputs, so no model runs are needed; clear pass/fail tests | Novelty got thinner after the spikes; a null result is plausible |
| **3** | **D4: Does Zoobot's uncertainty (and its explanations) match where Galaxy Zoo volunteers disagree?** (astronomy) | Your interest; the most distinctive story; the spike was easy | Brightness confound; Silva's lab plans an image extension of Visagreement |
| **4** | **D3: Why do LLM judges disagree with each other and with humans?** (LLM; education optional) | Exact reproduction; objective planted-bias test; the run takes only about 4.7 h | Modest novelty; the lab plans a text extension; big labs are close |
| **5** | **Soccer C1: Do xG explanation methods disagree per shot, and does disagreement flag physics-violating explanations?** | Soccer motivation; easy open data; the gap survived the check against Cefis & Carpita | Every method might simply agree on distance/angle |

**Honorable mentions:**
- **D1 Tile2Net** (the default project): the best instructor fit, but crowded, and its install was untested.
- **Soccer C2** (split-regime explanations): the safest soccer option, but the visualization plays only a supporting role.
- **EO spatial-shift diagnosis** and **tabular-FM context attribution:** promising, but scan-level only.

**For Oct 6:** pitch **#1 plus one or two of #2–#4**. Let Silva's answers break the tie (§6).

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
- **Still needed:** a non-SAM model (Cellpose3 `cyto3`), and a run on **NeurIPS22 Public-Test**, the clean held-out set. LIVECell test is in-distribution for these models.
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
