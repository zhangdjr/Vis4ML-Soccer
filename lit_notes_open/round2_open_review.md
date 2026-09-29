# Open-Topic Research-Gap Review for DS-GA 3001 (Visualization for ML)

**Prepared for:** solo project, NYU DS-GA 3001, Fall 2026 (Prof. Claudio Silva)
**Compiled:** 2026-09-28 by Claude (Opus 5.5) as lead reviewer. Five broad-scan sub-agents (Sonnet) and three deep-dive sub-agents (Opus) did the searching. The lead reviewer verified the load-bearing claims.
**Companion documents:**
- `literature_review.md`: the soccer-only review. Its candidates are carried here as a baseline.
- `lit_notes_open/`:
  - `PLAN.md`, `SCAN_BRIEF.md`, `DEEP_BRIEF.md`
  - `scan_1…5_*.md`: broad scan
  - `deep_1_tile2net_segrel.md`, `deep_2_tsfm_va.md`, `deep_3_llm_judges.md`: deep dives with full paper tables and search logs
  - `pdfs/`

> **Labels, same convention as the soccer review.**
> - **FACT:** checked against a primary source (paper, repo, API, dataset).
> - **AUTHOR CLAIM:** a paper's own statement.
> - **SYNTHESIS:** my inference across sources.
> - **SPECULATION:** a guess.
>
> "No paper found" is always SYNTHESIS, with a coverage caveat (§0.3).

---

## Contents
0. [Scope, method, limits](#0-scope-method-limits)
1. [Executive summary: the uncomfortable answers](#1-executive-summary-the-uncomfortable-answers)
2. [Broad-scan map (≈20 sub-areas)](#2-broad-scan-map-20-sub-areas)
3. [Instructor-lab overlap map](#3-instructor-lab-overlap-map)
4. [Load-bearing literature table](#4-load-bearing-literature-table)
5. [Deep-dive findings](#5-deep-dive-findings)
6. [Gap synthesis table](#6-gap-synthesis-table)
7. [Tempting but bad choices (open-topic edition)](#7-tempting-but-bad-choices-open-topic-edition)
8. [Final shortlist](#8-final-shortlist)
9. [Final comparison tables (12a / 12b), soccer included](#9-final-comparison-tables-12a--12b-soccer-included)
10. [If I were independently doing this review, here is what I should verify](#10-if-i-were-independently-doing-this-review-here-is-what-i-should-verify)
11. [Suggested next steps before Oct 20](#11-suggested-next-steps-before-oct-20)

---

## 0. Scope, method, limits

### 0.1 Constraints used throughout
These are from the user; see `lit_notes_open/PLAN.md`.
- **Time:** solo, about 40 h total. Proposal Oct 20, update Nov 3, final report Dec 14.
- **Course requirement:** reproduce and extend.
- **Data:** open-downloadable only.
- **Compute:** M1 Pro 32 GB; SLURM cluster; a 4 × 11 GB RTX box running Ollama (`qwen3.8:27b`, `gpt-oss-20b`, others).
- **Priorities:** a good grade first, then a plausible publication path for PhD applications.
- **Nice-to-have domains:** soccer, maps, education.

### 0.2 Method
1. **Broad scan (5 slices, about 20 sub-areas).** For each sub-area: instructor-lab overlap, key 2023–26 papers, saturation, verified open data, candidate RQs with falsification queries, and a triage verdict.
   - S1: maps and urban
   - S2: education
   - S3: LLM-side visualization
   - S4: CV / data-centric evaluation
   - S5: time series, TDA, scientific ML
2. **Deep dive (3 areas)**, chosen on course fit, feasibility and open-gap evidence. Each deep dive did full-text reads of the closest papers, a citing-paper sweep, repo and dependency checks, data-size checks, a concrete project design and an hour budget.
   - D1: Tile2Net segmentation reliability (maps)
   - D2: failure regimes of time-series foundation models
   - D3: disagreement among LLM judges, with an education option
3. **Lead-reviewer verification**, done independently this session:
   - existence and status of 12 repos (`gh api`);
   - the course default-project page;
   - SAEfarer (arXiv 2609.21142): accepted VIS 2026 short paper, authors Kerrigan, **Barr**, Bertini;
   - the prior-cohort repo **SegNetVis**;
   - LLM Comparator (archived);
   - the MT-Bench human-judgment license (CC-BY-4.0);
   - Coin Flip Judge (single-author preprint);
   - TIME (ICML 2026);
   - **mTSeer** (CHI 2021, Xu/Yuan/Wang/**C. Silva**/Bertini).
4. **Corrections to sub-agent claims** (FACT):
   - (a) S5 said Priscylla Silva is "unrelated" to the instructor's lab. **Wrong.** She is first author of Visagreement with C. Silva and Nonato.
   - (b) S5 said time-series VA has "no VIDA overlap". **Wrong.** mTSeer, found by D2, is Silva/Bertini forecasting-model VA.
   - (c) D3 could not find P. Silva's AAAI AI4Ed 2024 paper. It exists (arXiv 2405.13957). This was verified in the soccer review.

### 0.3 Known limits (read before trusting any "open gap")
- **Search services were throttled.** dblp was blocked (bot-check) for every agent. Semantic Scholar returned HTTP 429 almost immediately. OpenAlex's free daily budget ran out partway through the deep dives.
  - Lab-overlap checks rely on OpenAlex author sweeps (done in D1 and D3), arXiv author queries, ctsilva.github.io, and web search.
  - **So the lab overlap was missed twice in the broad scan (mTSeer, and last year's course project), and caught only in the deep dives.** Assume something similar could still be missing.
- **Not swept item by item:** CHI 2026 full program, UIST 2025, IUI 2026, IEEE TGRS/JSTARS (for remote-sensing calibration), MLSA/ECML workshops.
- **D2's pilot numbers** (§5.2) assume a row ordering between TIME's metric files and feature files that has not been verified against TIME's code.
- **Compute estimates** for Ollama throughput on 11 GB cards (D3) come from public benchmarks. They were not measured on your machine.

---

## 1. Executive summary: the uncomfortable answers

1. **The instructor's group is everywhere in Vis-for-ML, and that shapes every choice.**
   - FACT: overlapping Silva/Nonato/Barr/Bertini work exists for these candidate areas:
     - explanation disagreement (Visagreement, MOUNTAINEER, Explainalytics);
     - student-success XAI (the AAAI AI4Ed 2024 paper);
     - Mapper/TDA and DR explanation (GALE → MOUNTAINEER → FADEx, VIS 2026);
     - classifier calibration VA (Calibrate);
     - forecasting-model VA (mTSeer);
     - urban segmentation (Tile2Net);
     - SAE-based model VA (SAEfarer, VIS 2026, with Barr).
   - SYNTHESIS: you cannot avoid their work. **The winning move is to extend one of their tools on a question their papers leave open, and say so explicitly.** All three deep-dive candidates do this.
2. **The course default project (Tile2Net) is less open than it looks.**
   - FACT: a student in last year's cohort already built **SegNetVis** (Dec 2025). It is a linked "pixel ↔ network" debugging tool with a confidence filter. Two other Tile2Net student repos exist from the same week.
   - FACT: the course page lists "relationship between prediction confidence and accuracy" as a Track A key question.
   - So a **tool** will not differentiate you. A **measured finding** can. Nobody has measured whether Tile2Net's confidence is calibrated, or what share of its network breaks come from segmentation vs. the vectorizer (§5.1).
3. **Several attractive areas are already saturated or scooped as of 2026. Drop them.** SAE-feature dashboards and reliability, Mapper-for-explanations, student-success explainer disagreement, "is LLM grading reliable?", and generic "DR reliability" (§7).
4. **The three strongest open-topic candidates are all "extend-a-lab-tool + a quantitative, falsifiable test".**
   - **D1:** Tile2Net calibration plus error attribution. It extends Tile2Net + Calibrate, answers stated future work in Tile2Net/PathwayBench, and sits in maps (your interest).
   - **D2:** window-level failure regimes of time-series foundation models. It extends mTSeer's stated future work and TIME. The benchmark's released outputs mean **the MVP needs no model inference**.
   - **D3:** decomposing LLM-judge disagreement. It is structurally "Visagreement for LLM judges", with an exact reproduction (MT-Bench agreement numbers) and a planted-bias evaluation. Education grading is a stretch option.
5. **Novelty is moderate at best everywhere. Say it plainly in the proposal.**
   - D1: the tool half is done; the findings half is open.
   - D2: the stratification half is done (TIME, EXPRTS, Kang 2017); the window-level, in-vivo half is open.
   - D3: every ingredient is published; the integration plus validation is open.
   - None of these is a "big new method" paper. Each is a **workshop or VIS-short-shaped** contribution if the results are clean. **That matches the course and your risk tolerance.**
6. **Your best soccer candidate is still competitive.** Soccer Candidate 2 (split-regime dependence of explanations) has the highest evaluation clarity and lowest risk of anything reviewed, but a thinner visualization story. §9 puts all candidates side by side.

---

## 2. Broad-scan map (≈20 sub-areas)

Source: `scan_1…5`. Verdicts are those of the scan agents, adjusted by the lead reviewer where the deep dives or verification contradicted them (marked ✎).

| Slice | Sub-area | Saturation | Instructor-lab overlap | Best idea from scan | Verdict |
|---|---|---|---|---|---|
| S1 maps | (a) Tile2Net pipeline-error VA (default project) | VA side thin; CV side crowded (52 citers) | **High** (Tile2Net is theirs) | Pixel-confidence ↔ network-error VA | **Deep-dived (D1)** ✎ narrowed after the SegNetVis finding |
| S1 | (b) Road/building topology-error VA (SpaceNet, DeepGlobe) | Metrics saturated (APLS, clDice); VA thin | None found | Interactive topology-error diagnosis | Backup to D1 (same gap, larger benchmarks, weaker course hook) |
| S1 | (c) Spatial disparity of AI-map quality vs. census | Thin, probably because it is hard to do rigorously | Adjacent (Miranda's environmental-justice work) | Disparity layer on D1/S1b | Add-on only; confound-heavy |
| S2 edu | (a) Knowledge-tracing explanation reliability (pyKT) | Architectures crowded; explanation *evaluation* thin (a 2024 survey says so) | None found (dblp unchecked) | Faithfulness + cross-explainer disagreement on DKT/SAKT/AKT | Scan-level candidate (§8.5). Reads as "Visagreement on KT" |
| S2 | (b) LLM essay/short-answer grading reliability | **Saturated** (several preprints/month in 2026) | None | Grader-disagreement VA | Folded into D3 as its education instantiation |
| S2 | (c) Student-success XAI / fairness (OULAD) | **Saturated at the instructor lab** (4 papers, 2024–25) | **Very high** | Subgroup-conditioned explainer disagreement | **Avoid**, unless Silva explicitly invites it |
| S2 | (d) Learning-analytics dashboards | HCI-soft; weak effects in the systematic review | None | – | No |
| S3 LLM | (a) SAE / interpretability dashboards (small models) | **Scooped:** seed instability (Paulo & Belrose 2025; Gerasimov 2026); auto-interp labels (Karne 2026); **SAEfarer, VIS 2026 (Kerrigan, Barr, Bertini)** | Adjacent (Barr, Bertini) | – | **Avoid** |
| S3 | (b) LLM-as-judge disagreement VA | ML findings saturated; multi-judge VA open | None (Visagreement is a structural analogue) | Multi-judge × human disagreement VA | **Deep-dived (D3)** |
| S3 | (c) Text/CLIP embedding DR reliability | General DR reliability saturated (Jeon/Seo CHI 2025 + VIS 2026; FADEx VIS 2026) | **High** (FADEx) | Atzberger et al. 2024 pipeline extended to CLIP | Scan-level candidate (§8.5) |
| S4 CV | (a) Slice discovery for segmentation | VA active for classification and detection; segmentation thin | None | Domino adapted to per-image IoU | Maybe (check VibE's scope first) |
| S4 | (b) Calibration/uncertainty VA for segmentation | Stats side active (conformal segmentation); VA thin | **Calibrate is theirs** (classification only) | Calibrate → per-pixel | Merged into D1 |
| S4 | (c) Label-error VA for segmentation (cleanlab) | Algorithm done (Lad & Mueller 2023); academic VA absent; a Cleanlab product demo exists | None | Triage UI over cleanlab scores, evaluated on injected noise | Scan-level candidate (§8.5) |
| S4 | (d) Rashomon / multiplicity in vision | ML side active; no clean reproduction target | None | Seed-ensemble disagreement VA | No (fails reproduce-first) |
| S4 | (e) Concept-based explanations | **Saturated** | – | – | No |
| S5 | (a) Time-series foundation-model failure VA | ML side weekly; VA thin | ✎ **mTSeer (CHI 2021, C. Silva)**, found in D2 | Window-level failure regimes | **Deep-dived (D2)** |
| S5 | (b) TDA / Mapper for ML | **Saturated at the instructor lab** (GALE → MOUNTAINEER → FADEx); no public code found | **Very high** | Mapper parameter stability | **Avoid** (reimplementation risk) |
| S5 | (c) Climate / remote-sensing XAI vs. physics | Methods active (Mamalakis, Bommer); VA thin | Partial (Miranda climate vis) | EuroSAT attribution vs. spectral ground truth | Scan-level fallback; low ceiling |

---

## 3. Instructor-lab overlap map

All FACT except where marked. Barr (Capital One) and Bertini (ex-NYU, now Northeastern) co-author frequently with the lab.

| Lab work | Venue / year | Overlaps with | How to use it |
|---|---|---|---|
| Tile2Net (Hosseini, Sevtsuk, Miranda, Cesar, Silva) | CEUS 2023 | D1, S1 | **Reproduction target** (default project) |
| Calibrate (Xenopoulos, Rulff, Nonato, Barr, Silva) | TVCG 2023 | D1, S4b | **Reproduction target**. Classification only; "no multiclass" in its limitations. |
| mTSeer (Xu, Yuan, Wang, C. Silva, Bertini) | CHI 2021 | D2 | **Framing hook.** Its future work asks for "more features like the trend", "a larger number of candidate models" and expanded instance-level evaluation. |
| TiVy (Chan, Nonato, Palpanas, Silva, Freire) | VIS 2025 | D2 (a component) | Possible window-summary component |
| Visagreement (P. Silva, Guardieiro, Barr, C. Silva, Nonato) | TVCG 2025 | D3 (structural analogue), soccer C1, S2c | **Design inspiration** for D3. Also a novelty risk for a paper. |
| MOUNTAINEER / GALE | TVCG 2024 / ICML-W 2022 | Soccer C1, S5b | Avoid re-doing |
| Explainalytics; AAAI AI4Ed 2024 (P. Silva) | Inf. Sys. 2025 / AAAI-W 2024 | S2c | Avoid |
| FADEx (Meneses, Ortigossa, Silva, Nonato) | VIS 2026 (accepted, per program listing) | S3c | Avoid general DR reliability |
| SAEfarer (Kerrigan, Barr, Bertini) | VIS 2026 short (arXiv 2609.21142) | S3a | Avoid |
| Urbanite, Curio, StreetWeave (Miranda lab) | TVCG 2024–25 | D1 (adjacent) | Cite; overlap ruled out by D1 (full-text keyword pass) |
| **Prior course projects:** SegNetVis, Tile2Net-Inspector, Shadow-Impact-Analysis | GitHub, Dec 2025 | D1 | Cite them in the proposal and state your delta explicitly |

---

## 4. Load-bearing literature table

A condensed table. Full tables with every column are in the deep-dive files. Evidence levels: **F** = full text, **A** = abstract, **2** = secondhand.

| Paper | Yr | Venue / status | Relevance | Key fact or limitation | Code / data open? | Ev |
|---|---|---|---|---|---|---|
| Hosseini et al., Tile2Net | 2023 | CEUS (PR-J) | D1 target | Network breaks blamed on the centerline fitting; "probabilistic" gap filling is proposed as future work; **inference path saves no probability map** (FACT, code) | Code BSD-3; weights on Figshare; **training code/data not released**; CUDA-only | F + code |
| Xenopoulos et al., Calibrate | 2022/23 | TVCG (PR-J) | D1 target | Learned reliability diagram (EBM) + brushable subgroups; tabular, one-vs-rest only | MIT; dormant since 2022; `interpret` API break (one-line fix) | F + code |
| Zhang et al., PathwayBench | 2024 | arXiv (PP) | D1 | Tile2Net over-fragments (avg CC 5.01 vs GT 1.65, Seattle); no uncertainty or localization | MIT/ODbL via Git LFS; stale Python 3.8 pins | F |
| Zhang, Howe, Caspi, Prophet | 2024 | arXiv (PP) | D1 | Uses mean pixel probability as edge "confidence"; **never validated** | Partly | F (skim) |
| Gupta et al., Topology-aware uncertainty | 2023 | NeurIPS (PR-C) | D1 | Structure-wise uncertainty beats pixel-wise for proofreading; **curvilinear only** | Yes | F |
| Wang et al., On calibrating semantic segmentation | 2023 | CVPR (PR-C) | D1 | Multi-scale testing and boundaries drive miscalibration | – | A |
| SegNetVis (prior student) | 2025 | Course repo | D1 | Pixel ↔ network linked tool; no calibration analysis; Manhattan (in the training set) | Yes (no license) | README + code |
| Qiao et al., TIME | 2026 | ICML 2026 (PR-C) | D2 target | Pattern-level evaluation by 7 binarized STL features; "joint patterns" left open; leaderboard is filter → table, no linking | Outputs Apache-2.0 (2.9 GB, per-window); data CC-BY-NC | F (digest) + code |
| Xu et al., mTSeer | 2021 | CHI (PR-C) | D2 (lab) | Model- and instance-level comparison of 5 trained forecasters; explicit future work | – | F |
| Kjærnli et al., EXPRTS | 2024 | arXiv (PP) | D2 | Instance space + error drilldown (pre-foundation models); users struggled with PCA axes | 0 stars, no license, stale pins | F (digest) |
| Kang, Hyndman, Smith-Miles, instance spaces | 2017 | IJF (PR-J) | D2 | Feature-space view of forecast difficulty (classical methods) | – | 2 |
| Jander et al., Causal analysis for TSFMs | 2026 | arXiv (PP) | D2 | Synthetic: TSFMs overestimate persistence; regime-switch collapse thresholds; **asks for "in vivo validation"** | – | F (digest) |
| *The Spectrum Is Not Enough* | 2026 | arXiv (PP) | D2 risk | Spectral indices cannot capture what TSFMs exploit | – | A |
| Aksu et al., GIFT-Eval; Shchur et al., fev-bench | 2024 / 25 | arXiv (PP) | D2 | Dataset- or config-level results only | Apache-2.0 | F (digest) |
| Kahng et al., LLM Comparator | 2024/25 | TVCG (PR-J) | D3 target | One judge, repeated runs; future work = ">2 models", not multiple judges; schema has no judge-identity field | Apache-2.0, **archived**; Python library needs Vertex | F |
| Zheng et al., MT-Bench / Chatbot Arena | 2023 | NeurIPS D&B (PR-C) | D3 target | GPT-4 vs. human 66% / 85%; human–human 63% / 81% (Table 5) | **CC-BY-4.0, 1.4 MB**; FastChat agreement script is numpy-only | F |
| Shankar et al., EvalGen | 2024 | UIST (PR-C) | D3 | Criteria drift; multi-grader inter-rater reliability left as future work | In ChainForge (MIT) | F |
| Yagubyan, Coin Flip Judge | 2026 | arXiv (PP, single author) | D3 | 13.6% flip rate; cross-judge κ 0.51; future work: open judges, same-item human comparison | – | F |
| Mukherjee et al., Geometry of LLM-as-Judge | 2026 | EMNLP 2026 (accepted) | D3 | Inter-judge consensus ≠ human alignment | Raw outputs | F (skim) |
| Kohli, Nine Judges, Two Effective Votes | 2026 | arXiv (PP) | D3 | Correlated errors: 9 judges ≈ 2 independent votes | – | F (skim) |
| Sunkavalli, LLM judges as raters (ASAP essays) | 2026 | arXiv (PP), under review at JLA | D3 edu | Huge severity spread; r = .47–.56; DIF and open-weight models listed as out of reach | Harness claimed | F (skim) |
| Langfuse Score Analytics | 2025–26 | product docs | D3 baseline | Compares ≤2 score sources; κ/confusion matrix; no slicing, no rationales | OSS | F (docs) |
| Paulo & Belrose; Gerasimov et al.; Karne | 2025–26 | arXiv (PP) | S3a (avoid) | SAE seed instability and auto-interp generalization already studied | Yes | A |
| Kerrigan, Barr, Bertini, SAEfarer | 2026 | VIS 2026 short | S3a (avoid) | VA of SAE features for text classifiers | – | A (verified) |
| Atzberger et al., sensitivity of text spatializations | 2024 | VIS (PR-C) | S3c | 42,817 layouts; open code/data | Yes | A |
| Bai et al., Survey of explainable knowledge tracing | 2024 | Applied Intelligence (PR-J) | S2a | "Evaluation methods for explainable KT are lacking" | – | A |
| Lad & Mueller, segmentation label errors | 2023 | ICML-W | S4c | Algorithm shipped in cleanlab | Apache-2.0 | A |

---

## 5. Deep-dive findings

### 5.1 D1: Segmentation reliability in the Tile2Net pipeline (maps, default project)
Full notes: `lit_notes_open/deep_1_tile2net_segrel.md`.

- **Gap verdict: partly addressed.**
  - *Done:*
    - a linked pixel↔network tool (SegNetVis, prior cohort);
    - Tile2Net's graph-level fragmentation (PathwayBench);
    - pixel probability used as edge confidence, but unvalidated (Prophet);
    - structure-level uncertainty for curvilinear masks (Gupta 2023).
  - *Open (no hit in about 25 targeted queries plus the Tile2Net and Calibrate citation lists):*
    - (i) is Tile2Net's pixel confidence calibrated, and where not;
    - (ii) what share of network errors is segmentation-caused vs. vectorizer-caused;
    - (iii) can a probability-map "bottleneck confidence" separate true missing links from true gaps.
- **Why (ii) is clean.** An **oracle-mask ablation**: run Tile2Net's *own* polygon→network code on the ground-truth mask. With the same code and parameters, any difference is segmentation-induced by construction.
- **Key feasibility facts (FACT, from code and URLs):**
  - a ~10–20-line patch is needed to dump the 4-class softmax;
  - inference is CUDA-only (use the 11 GB box or SLURM);
  - NYC Planimetrics (March 2022 flight) is open and matches the NYC 2022 ortho server;
  - there is no crosswalk ground truth in NYC;
  - Manhattan is in the training data, so **use Queens / Bronx / Staten Island** (ask the instructor to confirm).
- **Biggest risk.** Tile2Net's own authors attribute network breaks to the centerline stage. If the vectorizer dominates, pixel confidence explains little. The oracle ablation detects this in weeks 1–2, and "the vectorizer dominates" is itself reportable.
- **Structure.** Option A (RQ1 calibration only, ~30 h) nests inside Option B (RQ1–3, ~40 h). The go/no-go for Option B is about Oct 27: the oracle network must run on 50 tiles.

### 5.2 D2: Window-level failure regimes of time-series foundation models
Full notes: `lit_notes_open/deep_2_tsfm_va.md`.

- **Gap verdict: partly addressed.** "Relate per-series error to interpretable features" is **already done**: TIME (ICML 2026) for 12+ TSFMs, EXPRTS, and Kang 2017. mTSeer (Silva lab) did instance-level forecaster-comparison VA before foundation models.
- **What survives:**
  - (a) *window-level*, context-conditioned failure regimes;
  - (b) cross-model disagreement as a diagnostic object;
  - (c) held-out validation of discovered regimes;
  - (d) an **"in vivo" test** of synthetic TSFM failure modes. Jander et al. (Aug 2026) explicitly request this.
- **Pilot by the D2 agent** on TIME's released per-window outputs. These numbers are SYNTHESIS and **provisional**, pending a row-order check.
  - TSFMs lose to seasonal naive on only 1–3% of series. The "fails vs. simple baselines" framing is weak unless you add AutoETS/ARIMA, which TIME lacks.
  - About 86% of error variance is within datasets, and about 66% within a single series across windows. So existing dataset- or variate-level stratifications miss most of it.
  - Chronos-2 and TimesFM-2.5 differ by more than 25% on about 32% of windows.
  - No single static feature correlates beyond |ρ| = 0.26.
- **Feasibility is unusually good (FACT).** TIME-Output publishes per-window quantile predictions and metrics for 30 models (Apache-2.0). **The MVP needs zero model inference.** Local Chronos-2 / TimesFM-2.5 (with an MLX backend for M1) is only for a reproduction check.
- **Biggest risk.** Interpretable features may explain little of window-level failure (weak pilot correlations; the July 2026 impossibility result). The fallback is a documented null result plus a disagreement-focused tool.

### 5.3 D3: Decomposing LLM-judge disagreement (LLM; education optional)
Full notes: `lit_notes_open/deep_3_llm_judges.md`.

- **Gap verdict: partly addressed.**
  - *Every ingredient exists:*
    - single-judge side-by-side VA (LLM Comparator);
    - criteria-alignment tools (EvalGen, EvalAssist, Evalet, MultEval);
    - two-source agreement dashboards (Langfuse);
    - one-layer statistical papers in 2026 (Coin Flip, Geometry, Nine Judges, Whose Gold?, and the essay-rater audit).
  - *Open:* an integrated VA tool that aligns multiple judges × prompt variants × repeats × **multiple human votes per item**, and splits item-level disagreement into four sources: human ambiguity, judge instability, feature-linked bias, and rationale divergence.
    - Evidence: VIS 2025/2026 programs and about 55 LLM Comparator citers contain no such tool (FACT).
- **Reproduction target (FACT).** Recompute Zheng et al. Table 5 from the released CC-BY-4.0 files: 961 cells have ≥2 human votes (575 unanimous / 386 split). Then rerun with local judges.
- **Objective evaluation.** Planted-bias recovery (category noise, length bias, position bias) plus a null control. The tool must flag the planted slices, including at least one that changes pooled κ by less than 0.05, and must flag nothing on the null.
- **Compute.** gpt-oss:20b (14 GB) and qwen3.8:27b (18 GB) each span two 11 GB cards. The minimum version is about 13 GPU-h unattended; the full design is about 36–46 GPU-h. SPECULATION: unmeasured. Test 10 items in week 1.
- **Education stretch.** ASAP sets 7/8 (two human raters), or PERSUADE 2.0 (CC BY-NC-SA, with demographic fields). This enables the DIF (differential item functioning, a subgroup-bias test) slice that the 2026 essay-audit paper says it could not do.
- **Biggest risk.** "Not novel enough" and scoop risk. It is structurally Visagreement-for-judges, and PAIR/KAIST/IBM are one step away.

---

## 6. Gap synthesis table

| Potential gap | Evidence | Closest papers | Already addressed? | Novelty confidence (why) | Data | Compute | Eval difficulty | Viz fit | Solo feasibility | Main risk |
|---|---|---|---|---|---|---|---|---|---|---|
| **G1. Calibration of Tile2Net pixel confidence on held-out imagery** | No hit among 52 Tile2Net citers or 20 Calibrate citers; Track A asks it but no one has published it | Calibrate, Wang 2023, Barfoot 2024 | No (for aerial sidewalks) | **Moderate-low alone.** It is a course track question, so classmates may do it. Stronger with G2. | NYC open | GPU inference | Low (ECE, CIs) | High | High | Looks like other Track A projects |
| **G2. Share of network errors from segmentation vs. vectorizer** | Tile2Net authors state it only qualitatively; PathwayBench gives aggregates only | Tile2Net, PathwayBench, TraversRL | No | **Moderate.** A clean ablation design that answers the authors' own text. | NYC open | CPU | Low–med | High | Med | Plumbing (oracle network) |
| **G3. Probability-based triage of true vs. false network gaps** | Prophet uses probability, unvalidated; Tile2Net future work | Prophet, Gupta 2023 | No | **Low–moderate.** Road-graph papers may do it internally (≈55% confidence it is unscooped). | NYC open | CPU | Med (label definition) | High | Med | Near-tautology if poorly defined |
| **G4. Window-level, context-conditioned TSFM failure regimes, validated held-out** | TIME and GIFT-Eval are dataset/variate-level; the pilot says variance is window-level | TIME, EXPRTS, mTSeer, Break-even | No | **Moderate.** The pilot motivates it quantitatively; the field moves weekly. | Open (TIME-Output) | None for the MVP | Low (AUROC, lift, CIs) | High | **High** | Features explain little (possible null) |
| **G5. In-vivo test of synthetic TSFM failure modes** | Explicitly requested by Jander et al. 2026 | Jander 2026, Context parroting | No | **Moderate.** A named open question. | Open | Low | Med | Med | High | Regime-switch detection noise |
| **G6. Multi-judge × multi-human disagreement decomposition VA** | No tool found; LLM Comparator is single-judge | LLM Comparator, Visagreement, Coin Flip, Geometry | Partly (ingredients) | **Moderate for the course, low–moderate for a paper** (a domain port of Visagreement) | CC-BY-4.0 | 13–46 GPU-h unattended | Low (planted patterns) | High | High | "Not novel" / scoop |
| G7. KT explanation reliability | 2024 survey says evaluation is lacking | pyKT, Swamy 2022 | Partly | Low–moderate ("XAI eval applied to a new domain") | ASSISTments (form), EdNet | Low | Med | Med | High | Generic |
| G8. Label-error triage VA for segmentation | No academic VA; a Cleanlab product demo exists | Lad & Mueller 2023 | Partly (product) | Low–moderate | VOC / Cityscapes | Low | Low (injected noise) | Med | High | Product overlap |
| G9. CLIP-embedding DR sensitivity (Atzberger extension) | No direct hit | Atzberger 2024, FADEx, Jeon | Partly | Low (the lab and SNU crowd the space) | Open | Low | Med | High | High | Lab-overlap framing |
| G10. SAE dashboard reliability | Answered in 2025–26 | Paulo & Belrose, Gerasimov, Karne, SAEfarer | **Yes** | – | – | – | – | – | – | Scooped |

---

## 7. Tempting but bad choices (open-topic edition)

| Tempting idea | Why it's bad here | Evidence |
|---|---|---|
| "A linked pixel ↔ network debugging tool for Tile2Net" | **Last year's cohort built it** (SegNetVis); the instructor has seen it | §3, D1 §1 |
| "SAE feature dashboard / check SAE stability across seeds" | Answered by 2025–26 preprints, and **SAEfarer (Barr, Bertini) is at VIS 2026** | S3a |
| "Mapper/TDA to compare explanations or diagnose models" | The instructor lab's own line through 2026 (GALE → MOUNTAINEER → FADEx); no public code, so reproducing means reimplementing | S5b |
| "Explainer disagreement for student-success prediction" | The instructor lab published it 4 times (2024–25) | S2c |
| "Is LLM essay grading reliable?" (a statistical study) | Several preprints per month in 2026; saturated | S2b |
| "UMAP/t-SNE reliability for embeddings" (general) | FADEx (VIS 2026, the lab) + Jeon/Seo (CHI 2025, VIS 2026) | S3c |
| "Stratify TSFM errors by seasonality/trend in a dashboard" | TIME (ICML 2026) already did pattern-level stratification with a leaderboard | D2 |
| "LLM judges are biased: here is my measurement" | Known since 2023; 2026 papers quantify flips, κ and correlation | D3 §3 |
| "Train a new segmentation/forecasting/LLM model" | Novelty would sit in the model, not the Vis4ML question; compute; Tile2Net training code is unreleased | – |
| Rashomon / multiplicity VA in vision | No single reproduction target, which conflicts with the course's reproduce-first rule | S4d |
| Weather foundation models (GraphCast/Pangu) error VA | Heavy data (ERA5) and compute; a domain workshop at best within 40 h | S5c |
| A controlled user study as the main contribution | Recruiting, IRB, power; not feasible solo | soccer review §6 |

---

## 8. Final shortlist

These are four open-topic candidates plus the soccer baseline. None is declared the winner. Hour estimates are SPECULATION tuned to your stated skills; the deep-dive files have line-item budgets.

### 8.1 Candidate D1: Calibration and error attribution in Tile2Net (maps)

- **Research question.** On held-out NYC imagery (Queens):
  - (1) where is Tile2Net's per-pixel confidence miscalibrated (class, boundary distance, canopy/shadow, tile edge)?
  - (2) what share of pedestrian-network topology errors is segmentation-induced vs. vectorizer-intrinsic (oracle-mask ablation)?
  - (3) *stretch, gated:* does a path "bottleneck confidence" separate true missing links from true gaps better than gap length or mean-buffer confidence?
- **Why it matters.** Tile2Net's authors want "probabilistic" gap filling but fear hallucinated links. PathwayBench shows fragmentation but not its cause. Calibration tells users when to trust the maps.
- **Evidence for the gap.** §5.1 and G1–G3. Tile2Net and Calibrate citation sweeps found no hit. SegNetVis has no calibration analysis and uses Manhattan, which is in the training set.
- **Reproduce.**
  - Tile2Net polygon-level evaluation (paper Table 4 style) on the held-out area.
  - Calibrate's learned reliability diagram (fix the `interpret` API).
  - Stronger version: PathwayBench's Tile2Net Seattle row.
- **Minimum viable project.** RQ1 plus Calibrate-style views re-implemented in Streamlit/Plotly (learned reliability curve, brushable pixel-feature histograms, linked map), with temperature scaling as a control. About 30 h.
- **Stronger version.** RQ1–RQ3; second area of interest; Seattle via PathwayBench; a second segmentation model; comparison with Gupta's structure-wise uncertainty.
- **Dataset.** NYC Orthos 2022 plus NYC Planimetrics Sidewalk and Roadbed (open), about 1.5 km² in Queens.
- **ML.** Tile2Net `satellite_2021.pth`, inference only, with a softmax-dump patch.
- **Visualization.**
  - Stratified reliability view;
  - pixel-feature brushing;
  - a linked map of confident errors;
  - a gap-triage list with the probability field and maximin path;
  - an error-attribution bar per block.
  - Why these views: the object of study is *spatial* miscalibration and its downstream graph effect. A scalar ECE hides both.
- **Evaluation.**
  - ECE (equal-width and equal-mass), Brier and NLL with block-cluster bootstrap CIs;
  - error-attribution rates;
  - AUROC/AUPRC vs. baselines;
  - pre-registered H1–H3 falsification criteria.
- **Risk.** Vectorizer dominance; install/GPU friction; ground-truth misregistration; classmate overlap.
- **Compute.** GPU for inference (SLURM or the RTX box); the rest on CPU.
- **Course fit.** It is the default project plus an A×B hybrid, which is explicitly allowed. It extends two of the instructor's own tools.
- **Publication path.** Add 2–3 cities, a second model, gap-repair impact on routability, and a small mapper-user study. Venues: VIS short paper, the VIS Uncertainty Vis workshop (no segmentation paper 2024–26: FACT), CVPR EarthVision.

### 8.2 Candidate D2: Window-level failure regimes of time-series foundation models

- **Research question.**
  - Where do zero-shot TSFMs (Chronos-2, TimesFM-2.5, Moirai-2, TiRex) fail at the forecast-window level, relative to strong classical baselines (AutoETS/ARIMA/Theta) and to each other?
  - Are failures predicted better by context-window features than by TIME's static variate patterns, using leave-dataset-out validation?
  - Do the synthetic failure modes of Jander et al. (persistence overestimation, regime-switch collapse) show up "in vivo"?
- **Why it matters.** Leaderboards aggregate at dataset level, but (provisionally) about 66% of error variance is within a single series. Practitioners need to know *when* to distrust a zero-shot forecast.
- **Evidence for the gap.** §5.2, G4–G5. TIME's pattern analysis is single-feature and static. mTSeer's future work matches. Jander et al. ask for in-vivo validation.
- **Reproduce.**
  - Recompute TIME's pattern-level results from the released outputs (pandas, about 3 h).
  - Rerun Chronos-2 and seasonal naive on 2–3 small tasks and match the saved per-window metrics.
- **Minimum viable project.** Classical baselines via statsforecast; context-window features; failure labels; a variance decomposition; a leave-dataset-out failure predictor; a 5-view Streamlit tool. **No TSFM inference needed.**
- **Stronger version.** H4 dose-response for regime switches; a context-length what-if probe on brushed windows; GIFT-Eval replication; a small expert study.
- **Dataset.** TIME-Output, 50 short-horizon tasks (about 50–150 MB subset), plus raw TIME (220 MB, CC-BY-NC).
- **ML.** Released TSFM outputs; statsforecast; tsfeatures/catch22; a decision tree or SliceLine slice finder.
- **Visualization.**
  - A failure-regime map (UMAP/PCA of context features, with lasso);
  - conditional-error small multiples;
  - a leaderboard-flip matrix inside the selection;
  - a window drilldown with quantile fans;
  - a slice table with held-out lift.
  - Why these views: failure is multivariate and window-level, and the rank-flip view directly tests "aggregates hide it".
- **Evaluation.**
  - Variance shares, with CRPS and a same-family noise floor;
  - leave-dataset-out AUROC (ΔAUROC ≥ 0.03 threshold);
  - held-out slice lift ≥ 1.5× vs. random and single-feature slices;
  - dose-response CIs.
- **Risk.** Features explain little (a possible null, which is reportable); the unverified row ordering; weekly scoop risk.
- **Compute.** MacBook is enough for the MVP. The RTX box or MLX for optional reruns.
- **Course fit.** Model assessment, black-box, clustering/DR and time-series lectures. It extends mTSeer.
- **Publication path.** Add a second benchmark, the full in-vivo analysis, the what-if probe and an expert study. Venues: VIS short paper or workshops, TSFM workshops (ICLR TSALM-style), IJF.

### 8.3 Candidate D3: Decomposing LLM-judge disagreement (with an education option)

- **Research question.** When several local open-weight judges (gpt-oss-20b, qwen3.8:27b, plus the released GPT-4 judgments), with prompt variants, judge MT-Bench items that have multiple human votes:
  - can a linked-view tool split item-level judge–human disagreement into human ambiguity, judge instability, feature-linked bias and rationale divergence?
  - does it recover planted bias patterns that pooled κ hides?
- **Why it matters.** Aggregate agreement numbers are fragile: protocol choices move accuracy from 0.55 to 0.90 (AUTHOR CLAIM, Rao & Callison-Burch 2026). Consensus between judges ≠ alignment with humans (Geometry, EMNLP 2026). Practitioners need item-level diagnosis.
- **Evidence for the gap.** §5.3 and G6. LLM Comparator is single-judge and archived. Coin Flip's future work asks for open judges and same-item human comparison.
- **Reproduce.**
  - Zheng et al. Table 5 exactly (about 1 h).
  - The same table with local judges and cluster-bootstrap CIs.
  - The LLM Comparator workflow via its JSON format and hosted demo.
- **Minimum viable project.** Two local judges plus GPT-4 released judgments, one prompt variant, position swap; views = agreement matrix (baseline), decomposition scatter, feature small multiples with permutation-null bands, item drilldown; planted P1–P3 plus null N0.
- **Stronger version.** A second prompt variant; repeat-run instability; rationale clustering; an Arena-140k contamination check; **education:** ASAP 7/8 or PERSUADE 2.0, with severity/halo and a DIF slice.
- **Dataset.** `lmsys/mt_bench_human_judgments` (CC-BY-4.0, 1.4 MB). Optional: `lmarena-ai/arena-human-preference-140k` (CC-BY-4.0); PERSUADE 2.0 (CC BY-NC-SA).
- **ML.** Ollama judges; sentence-transformers for rationales.
- **Visualization.** Six views; see D3 §8.4.
  - Why these views: the decomposition scatter separates the three sources that pooled κ mixes, and the null bands stop over-reading.
- **Evaluation.** Exact reproduction ±1 pp; H2–H4 statistics (mixed-effects model, cluster permutation); planted-pattern recovery (top-3, BH q < 0.05) with zero false flags on the null.
- **Risk.** Novelty and scoop; local-judge parse failures; unmeasured throughput.
- **Compute.** 13–46 GPU-h unattended on the RTX box; everything else on the MacBook.
- **Course fit.** The NLP/LLM lecture (Nov 3) and model assessment; a Visagreement-style design.
- **Publication path.** Add a second domain (education), an n = 6–8 user study against a Langfuse-style baseline, and the n_eff and shared-axis diagnostics. Venues: CHI HEAL workshop, VIS short paper, EDM/LAK (education version).

### 8.4 Soccer baseline (from `literature_review.md` §11)

- **Soccer C1:** physics-anchored evaluation of explanation disagreement for xG. It extends Visagreement and MOUNTAINEER.
- **Soccer C2:** split-regime dependence of VAEP/xG explanations. Lowest risk, highest evaluation clarity, thinner visualization story.
- Soccer C3 (LLM wordalisation) and C4 (projection audit) are weaker than the open-topic candidates above. See the soccer review.

### 8.5 Scan-level alternatives (not deep-dived; treat as provisional)

| Alternative | Why it's here | What would need checking first |
|---|---|---|
| **KT explanation reliability (education)** | A 2024 survey states the evaluation gap; pyKT is a clean target; no lab overlap found | dblp/lab check; whether it reads as "Visagreement on KT" |
| **Segmentation label-error triage VA** | Algorithm open (cleanlab); objective evaluation via injected noise | Hands-on test of Cleanlab's product demo (Vizzy) for overlap |
| **CLIP-embedding DR sensitivity** | Clean Atzberger 2024 target with open code | Framing against FADEx (lab) and Jeon/Seo |
| **Road-network topology-error VA (SpaceNet)** | Same gap as D1 on bigger benchmarks, no lab overlap | Weaker course hook; baseline repo availability |

---

## 9. Final comparison tables (12a / 12b), soccer included

### 9a. Evidence and scope

| Candidate question | Evidence of gap | Closest prior work | Public data/code | Reproduction target | Minimum viable extension | Evaluation clarity | Visualization fit | Solo feasibility | Main failure mode | Evidence still needed |
|---|---|---|---|---|---|---|---|---|---|---|
| **D1. Tile2Net calibration + error attribution** | No calibration or error-attribution paper among Tile2Net/Calibrate citers; SegNetVis has no calibration | Tile2Net, Calibrate, PathwayBench, Prophet, Gupta 2023, SegNetVis | NYC ortho + Planimetrics (open); Tile2Net BSD-3; pycalibrate MIT | Tile2Net Table-4 evaluation; Calibrate learned curve | Per-pixel stratified calibration + linked map (+ oracle ablation) | High (RQ1/2), medium (RQ3) | High | Good (~30–40 h) | Vectorizer dominates; install friction | Which boroughs are held out; softmax-dump spike; oracle network on 50 tiles |
| **D2. TSFM window-level failure regimes** | TIME/GIFT-Eval are dataset/variate-level; pilot variance share; Jander asks for in-vivo | TIME, mTSeer, EXPRTS, Kang 2017, Jander 2026 | TIME-Output Apache-2.0 (per-window); models Apache/NC | Recompute TIME pattern-level results; local Chronos-2 rerun | Classical baselines + context features + leave-dataset-out failure predictor + linked views | **High** | High | **Very good** (~41 h, no inference in MVP) | Features explain little (null) | Row-order check; CRPS rerun of the pilot; weekly scoop check |
| **D3. LLM-judge disagreement decomposition** | No multi-judge × human VA in VIS 2025/26 or LLM Comparator citers | LLM Comparator, Visagreement, Coin Flip, Geometry, EvalGen | MT-Bench CC-BY-4.0; FastChat; Ollama judges | Zheng Table 5 exact + local-judge rerun; LLM Comparator workflow | Decomposition views + planted-bias recovery + null | **High** (planted patterns) | High | Good (~40 h + unattended GPU) | "Not novel" / scoop | Throughput test; CHI 2026/IUI 2026 sweep; ask Silva about a Visagreement-style framing |
| **Soccer C1. Physics-anchored xG explanation disagreement** | 0/25 soccer XAI papers test local agreement/stability | Visagreement, MOUNTAINEER, Tsai 2026 | StatsBomb open; Visagreement repo | Visagreement on an xG MLP | Semantic perturbations + disagreement-predicts-violation + planted leak | Medium-high | High | Good (~40 h) | Trivial agreement on distance/angle | Cefis & Carpita full text |
| **Soccer C2. Split-regime dependence of explanations** | Davis 2024 has no explanation protocol; Peters 2026 is feature-leakage only | Davis 2024, Peters 2026, Calibrate | StatsBomb + Wyscout; socceraction | socceraction VAEP with a proper split | 3 regimes × 5 seeds; SHAP change; reliability diagrams | **High** | Medium-high | **Very good** | Null result for xG | Peters 2026 full text |

### 9b. Risk and reward

These ratings are qualitative, with reasons. There is no overall score and no winner.

| Candidate | Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|---|
| **D1. Tile2Net** | **Excellent.** The default project; Track A explicitly; extends two lab tools | **Medium.** RQ2/RQ3 answer authors' stated gaps; the tool alone is not novel | **Medium.** CUDA-only install, patch, oracle-network plumbing | **Low–Med.** Open, aligned 2022 data; held-out status unverified; no crosswalk ground truth | **Medium.** 4 linked Plotly views over rasters and a graph | High (RQ1/2) / Med (RQ3) | …you want maps and the instructor's buy-in, can get GPU time in week 1, and will headline a measured, possibly negative, finding |
| **D2. TSFM regimes** | **High.** Five lectures; extends mTSeer | **Medium.** Window-level + in-vivo is open; the stratification half is taken | **Low–Med.** No inference for the MVP; stale benchmark pins | **Low.** Open on HF; small subset | **Low–Med.** 5 Plotly views; test lasso-linking early | **High** | …you want the lowest setup risk with quantitative, falsifiable criteria and can accept a null interpretability result |
| **D3. LLM judges** | **High.** LLM lecture + model assessment; Visagreement-style | **Medium.** Integration + validation novelty; findings are known | **Low–Med.** Black-box Ollama; throughput unmeasured | **Low.** MT-Bench CC-BY, 1.4 MB | **Medium.** The rationale view is the hardest | **High** | …you want an exact reproduction, LLM portfolio value and a CHI-workshop path, and accept "tool + validation" novelty |
| **Soccer C1** | Excellent | Moderate–High | Low–Moderate | Low | Moderate | Medium–High | …soccer motivation matters and you want the closest tie to Visagreement/MOUNTAINEER |
| **Soccer C2** | Very good | Moderate | Low | Low | Low–Moderate | **High** | …you want the safest path and accept visualization in a supporting role |

**How they differ** (SYNTHESIS, not a ranking):
- **D2 and Soccer C2** have the least execution risk.
- **D1** has the strongest instructor alignment, and also the most classmate overlap.
- **D3** has the cleanest reproduction number and the most scoop exposure.
- **D1 and D3** have the richest visualization story.

---

## 10. If I were independently doing this review, here is what I should verify

1. **dblp for Silva, Nonato, Miranda, Barr and Bertini, 2024–2026.** It was blocked for every agent. Look specifically for LLM-evaluation, forecasting and segmentation-uncertainty work that the OpenAlex sweeps might have missed.
2. **Prior-cohort projects.** Search GitHub for `tile2net`, `mtseer`, `llm judge` + `DS-GA 3001` or `VisML`, created Dec 2025, and ask the TA whether a project archive exists. SegNetVis shows course-internal overlap is real.
3. **D1: which NYC areas `satellite_2021.pth` was trained on.** Ask Silva or the TA, or open a Tile2Net issue.
4. **D1: whether Tile2Net's inference writes any probability map.** Run `examples/example.sh`; the code reading says no.
5. **D2: TIME's row ordering between `metrics.npz` and `features/*.csv`.** Check it in `zqiao11/TIME`, then re-run the pilot with CRPS as well.
6. **D2: the TIME leaderboard.** Confirm that its per-pattern tab is filter → table with no linking. Check for any TIME follow-up doing window-level analysis (citers of arXiv 2602.12147).
7. **D3: the exact reproduction.** Recompute 66% / 85% / 63% / 81% from the two MT-Bench parquet files. If this fails, D3's reproduction story changes.
8. **D3: throughput.** On your RTX box, time one 1,500-token judge prompt for each model, with thinking off. Confirm both models fit concurrently.
9. **D3: scoop sweep.** CHI 2026 full program; IUI 2026; the newest arXiv cs.HC papers with "judge" + interactive; recent work from Kahng/PAIR, KAIST (Juho Kim), IBM (Ashktorab), Arawjo, Shankar.
10. **The claim that SAE, Mapper and student-success XAI are saturated.** Skim Gerasimov 2026, SAEfarer, FADEx and Explainalytics abstracts yourself. If you disagree, those areas reopen.
11. **The course's JavaScript/D3 expectation for 2026**, and whether Streamlit/Plotly is acceptable.
12. **Whether reproducing an instructor-lab tool is welcomed.** Ask at the Oct 6 project discussion. Candidates D1, D3 and Soccer C1 depend on it.

---

## 11. Suggested next steps before Oct 20

These are practical, not conclusions.

- **Oct 6 (lecture: "Black-box Interpretation & Project Discussion").** Bring 2–3 one-paragraph pitches, for example D1, D2 and D3, or swap one for Soccer C2. Ask Silva three questions:
  - (a) which he'd welcome;
  - (b) whether extending Calibrate, mTSeer or Visagreement is encouraged;
  - (c) for D1, which boroughs are held out.
- **Before choosing, run one 1–2 h spike per finalist:**
  - D1: install Tile2Net on the GPU box and run the Boston example.
  - D2: download one TIME-Output model folder and verify the row ordering.
  - D3: run the MT-Bench agreement recomputation, then one Ollama judge call.

  The spike that fails is the strongest evidence you will get.
- **Proposal (4 pages).** Reuse §4/§5 of the chosen deep-dive file for related work. State the pre-registered hypotheses and the grade-safe MVP versus the gated stretch goals.

*AI-use disclosure for the course: produced by Claude (Opus 5.5) with 8 sub-agents (5 Sonnet broad scans, 3 Opus deep dives). The lead agent verified the load-bearing claims against primary sources, as listed in §0.2.*
