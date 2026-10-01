# Research-Gap Review: FULL DETAILS

> This is the complete evidence document. For the short version, read **`research_gap_review.md`** first. Section numbers here (§0–§13) are the ones the short version cites.

**Project:** solo final project, NYU DS-GA 3001 *Visualization for Machine Learning* (Prof. Claudio Silva), Fall 2026
**Compiled:** 2026-09-29 by Claude (Opus 5.5), lead reviewer. The review took three rounds and 19 sub-agents (list in §0.2). The lead reviewer checked every claim the conclusions depend on.
**Last updated 2026-09-29 (evening):** the feasibility-spike and verification results (§11 "Verified", §12 "Spike results") are now folded into §0, §1, §5, §7 and §9, so earlier sections no longer contradict them.
**This is the single consolidated document.** It replaces `literature_review.md` (round 1) and `literature_review_open.md` (round 2), which are archived as `lit_notes/round1_soccer_review.md` and `lit_notes_open/round2_open_review.md`. Per-area evidence (full paper tables, search logs, PDFs) lives in:
- `lit_notes/01–06_*.md`: soccer
- `lit_notes_open/scan_1–10_*.md`: broad scans
- `lit_notes_open/deep_1–5_*.md`: deep dives
- `lit_notes_open/pdfs/`

> **Labels.**
> - **FACT:** verified against a primary source (paper, repo, API, dataset).
> - **AUTHOR CLAIM:** a paper's own statement.
> - **SYNTHESIS:** my inference across sources.
> - **SPECULATION:** a guess.
>
> "No paper found" is always SYNTHESIS and carries the coverage caveats in §0.3.

---

## Contents
0. [Scope, method, limits](#0-scope-method-limits)
1. [Executive summary](#1-executive-summary)
2. [Course context](#2-course-context)
3. [Instructor-lab and prior-cohort overlap map](#3-instructor-lab-and-prior-cohort-overlap-map)
4. [Everything we looked at (≈35 sub-areas) and the verdicts](#4-everything-we-looked-at-35-sub-areas-and-the-verdicts)
5. [Candidate details](#5-candidate-details)
6. [Higher-upside tier](#6-higher-upside-tier)
7. [Load-bearing literature by candidate](#7-load-bearing-literature-by-candidate)
8. [Tempting but bad choices](#8-tempting-but-bad-choices)
9. [Side-by-side comparison (12a evidence / 12b risk–reward)](#9-side-by-side-comparison)
10. [Ranking: top 5](#10-ranking-top-5-updated-2026-09-29-after-the-spikes)
11. [What you should verify yourself](#11-what-you-should-verify-yourself)
12. [Next steps before Oct 20](#12-next-steps-before-oct-20)
13. [Appendix: corrections, the other system's report, AI-use disclosure](#13-appendix)

---

## 0. Scope, method, limits

### 0.1 Constraints used throughout
- **Solo**, with about **40 h total** (3–4 h/week). The **4-page proposal is due Oct 20**; the 1-page update Nov 3; the **8-page report Dec 14**; presentations Dec 1 and 8.
- **Course format:** "reproduce prior work **or** implement a proposed research idea", then demonstrate the prior work and the project. **Correction 2026-09-29:** earlier text said reproduce *and* extend, which is stronger than the syllabus. Reproduction is optional; candidates keep a small one as validation.
- **Data:** openly downloadable only. No credentialed or NDA data.
- **Compute:** MacBook M1 Pro (32 GB); a SLURM cluster; a 4 × 11 GB RTX box running Ollama (`qwen3.8:27b`, `gpt-oss-20b`, and others). Activation-level work is realistic only for ≤2–4B models. There may be no paid LLM API.
- **Skills:** strong in ML, PyTorch, CV and segmentation; moderate in NLP and statistics; **weak in JS/D3**. So Streamlit/Plotly/Jupyter linked views are preferred.
- **Priorities:** (1) a good grade (feasible, low-risk); (2) a plausible workshop or short-paper path for PhD applications.
- **Interests (nice to have):** soccer, maps, education, Earth observation, microscopy, astronomy.

### 0.2 Method (three rounds)

| Round | Scope | Agents | Output |
|---|---|---|---|
| 1 | Soccer only: explainable AI, LLM explanations, tactics and dimensionality reduction (DR), datasets, methodology, Silva's sports visual analytics (VA) | 6 (Opus) | `lit_notes/01–06` |
| 2 | Open topics: maps, education, LLMs, CV evaluation, time series / TDA / science | 5 broad scans (Sonnet) + 3 deep dives (Opus: D1 Tile2Net, D2 time-series foundation models, D3 LLM judges) | `lit_notes_open/scan_1–5`, `deep_1–3` |
| 3 | User-chosen science domains plus two lead picks: Earth observation (EO), microscopy, astronomy, tabular foundation models, NFL tracking | 5 broad scans (Sonnet) + 2 deep dives (Opus: D4 astronomy, D5 microscopy) | `scan_6–10`, `deep_4–5` |

**Lead-reviewer checks, done this session:**
- Existence, license and status of about 25 repos and datasets.
- Venue and authors for MOUNTAINEER, Visagreement, SAEfarer, TIME, mTSeer, Coin Flip Judge, Ichmoukhamedov et al., Jeon et al., Tang et al., PassAI, Peters 2026, Tsai 2026, MARC and BISCUIT.
- The course site and the default-project page.
- The prior-cohort repo SegNetVis.

### 0.3 Known limits (read before trusting any "open gap")
- **Search services were throttled throughout.** dblp was blocked for every agent. Semantic Scholar rate-limited almost immediately. OpenAlex's free daily budget ran out mid-round in rounds 2 and 3.
- **As a result, the broad scans missed instructor-lab overlap three times** that the deep dives later caught: mTSeer, a prior-cohort project, and Priscylla Silva's lab membership. Assume some overlap may still be missing. Check dblp yourself (§11).
- **Not swept item by item:** CHI/UIST/IUI 2026 full programs, MLSA and StatsBomb proceedings, IEEE TGRS/JSTARS, and the Astronomy & Computing and MNRAS back catalogues.
- **Pilot numbers and compute estimates:** ~~unmeasured~~ **measured on 2026-09-29** (§12 Spike results):
  - D2's original pilot used the **wrong** row ordering for 34 of 50 datasets. It was redone with a name-based join, which reproduces TIME's numbers within 0.1%.
  - D3's throughput was measured on an L40S: the minimum run takes about 4.7 h.
  - D5 and D4 pipelines run end to end.

---

## 1. Executive summary

**What we did.** Three rounds of falsification-first literature review, covering soccer, about 20 open-topic sub-areas, and 5 science domains. Seven areas got deep dives. Every "gap" was stress-tested against the latest 2025–26 work and against the instructor's own lab.

**What we found.**
1. **The instructor's group and collaborators have published the obvious idea in most Vis-for-ML areas:**
   - explanation disagreement (Visagreement, MOUNTAINEER, Explainalytics);
   - calibration VA (Calibrate);
   - Mapper/TDA and DR explanation (GALE → MOUNTAINEER → FADEx, VIS 2026);
   - forecasting-model VA (mTSeer);
   - urban segmentation (Tile2Net);
   - SAE-based model VA (SAEfarer, VIS 2026, with Barr);
   - student-success explainable AI (XAI).

   The working strategy is to **extend one of their tools on a question their papers leave open, and say so explicitly.**
2. **Many tempting areas are saturated or were just scooped** (§8). The course default project is also crowded: last year's cohort already built the obvious Tile2Net tool.
3. **The defensible gaps are narrow, measurable ones:** an untested assumption, a named future-work item, or a finding that aggregate leaderboards hide. That is why research upside tops out at **Medium** for grade-safe designs (§6 explains the higher-upside versions).
4. **Seven candidates survive deep review** (§5). There are also two strong scan-level options: EO spatial-shift diagnosis and tabular-FM context attribution. NFL tracking is **ruled out**: the Big Data Bowl terms forbid use outside the contest.

**My current top 5** (details in §10):
1. **D5:** ground-truth-free QC for generalist cell segmentation (microscopy).
2. **D2:** window-level failure regimes of time-series foundation models.
3. **D4:** Zoobot vs. volunteer disagreement (astronomy; the most distinctive, higher-upside story).
4. **D3:** decomposing LLM-judge disagreement.
5. **Soccer C1:** local, physics-anchored explanation disagreement for xG.

Honorable mentions: D1 (Tile2Net, the default project) and Soccer C2.

**Feasibility spikes (2026-09-29): all four passed** (D5, D2, D3, D4; D1 skipped), and the ranking is unchanged. Two corrections came out of them:
- D2's novelty claim is narrower because of Wang et al. 2606.18367.
- D2's old pilot correlations are void.

D3 is cheaper than estimated. Details are in §12.

---

## 2. Course context

FACT, from ctsilva.github.io/2026-VisML-CDS, its syllabus and the default-project page:

| Item | Detail |
|---|---|
| Weight | Project = 45% of the grade: proposal 10%, update 10%, final 25% |
| Type | "Reproduce prior work **or** implement a proposed research idea"; demonstrate "both the prior work, and your final research project". **Either is allowed.** Earlier rounds over-read this as "and extend" (corrected 2026-09-29) |
| Team | 2–3 expected, **solo allowed**; team formation was Sept 15 (confirm solo status) |
| Deliverables | 4-page proposal (Oct 20); 1-page update (Nov 3); 8-page report in "conference paper format, e.g., IEEE VIS" (Dec 14); GitHub repo; 3–5 min demo video (default project) |
| AI policy | Allowed; disclose AI-generated parts |
| Front end | **FACT (Silva, via the student, 2026-09-29): JS/D3 is not required. The topic just has to fit the class.** Streamlit/Plotly/Jupyter tools are fine. |
| Default project | VA for AI-generated urban infrastructure maps (Tile2Net). Tracks: A *Segmentation Detective*, B *Network Quality Inspector*, C *Urban Time Traveler*; hybrids allowed |
| Lectures | 9/22 model assessment · 9/29 white-box · **10/6 black-box + project discussion** · 10/13 clustering · 10/20 DR · 10/27 DL vis · 11/3 NLP/LLM · 11/10 TDA · 11/17 time series · 11/24 interpretable ML and fairness |

---

## 3. Instructor-lab and prior-cohort overlap map

FACT unless marked. Barr (Capital One) and Bertini (ex-NYU, now Northeastern) co-author often with the lab.

| Lab / course work | Venue | Touches | Use it as |
|---|---|---|---|
| Tile2Net (Hosseini, Sevtsuk, Miranda, Cesar, Silva) | CEUS 2023 | D1 | Reproduction target |
| Calibrate (Xenopoulos, Rulff, Nonato, Barr, Silva) | TVCG 2023 | D1, D4 | Reproduction target and hook (classification-only, hard binary labels) |
| mTSeer (Xu, Yuan, Wang, C. Silva, Bertini) | CHI 2021 | D2 | Framing hook: its stated future work |
| Visagreement (P. Silva, Guardieiro, Barr, C. Silva, Nonato) | TVCG 2025 | Soccer C1, D3, D4, tabular FM, KT | Design inspiration; a "domain port" risk for papers |
| MOUNTAINEER / GALE | TVCG 2024 / ICML-W 2022 | Soccer C1, D4 RQ-B, TDA | Avoid re-doing; D4 can cite as method |
| Explainalytics; AAAI AI4Ed 2024 (P. Silva) | Inf. Sys. 2025 / AAAI-W | Student-success XAI | Avoid |
| FADEx (Meneses, Ortigossa, Silva, Nonato) | VIS 2026 | Generic DR reliability | Avoid |
| SAEfarer (Kerrigan, Barr, Bertini) | VIS 2026 short | SAE dashboards | Avoid |
| TiVy (Chan, Nonato, Palpanas, Silva, Freire) | VIS 2025 | D2 (a component) | Optional |
| **Prior-cohort repos** (SegNetVis, Tile2Net-Inspector, Shadow-Impact-Analysis) | GitHub, Dec 2025 | D1 | Cite; state your delta |
| No overlap found | – | Microscopy (Nonato/Miranda sweep incomplete), astronomy (only an old AMNH vis tie), EO FMs, tabular FMs, LLM judges | – |

---

## 4. Everything we looked at (≈35 sub-areas) and the verdicts

✅ deep-dived and viable · ◐ scan-level viable · ✗ avoid

| Domain | Sub-area | Verdict | One-line reason |
|---|---|---|---|
| Soccer | XAI faithfulness/stability for xG (C1) | ✅ | 0/25 soccer XAI papers test local agreement or stability; physics gives partial ground truth (GT); Visagreement overlap |
| Soccer | Split-regime dependence of explanations (C2) | ✅ | Davis 2024 gives no explanation protocol; Peters 2026 covers feature leakage only |
| Soccer | LLM wordalisation faithfulness | ◐ | Ichmoukhamedov 2024 already did RA/SA/VA + flipped signs (incl. FIFA data); only ablation-based absent-feature hallucination is left |
| Soccer | Tactical embeddings + UMAP | ✗ / ◐ as audit | Done (Tang 2023, Baron 2024); only a reliability audit remains |
| Soccer | Tactical phase segmentation | ✗ | Proprietary tracking; subjective labels |
| Maps | Tile2Net calibration + error attribution (D1) | ✅ | The tool is done (SegNetVis); the measured findings are open |
| Maps | Road topology-error VA (SpaceNet) | ◐ | Same gap on bigger benchmarks; weaker course hook |
| Maps | Spatial disparity of map quality | ◐ add-on | Confound-heavy |
| Education | Knowledge-tracing explanation reliability | ◐ | A 2024 survey says evaluation is missing; reads as "Visagreement on KT" |
| Education | LLM grading reliability | ✗ (folded into D3) | Several preprints per month |
| Education | Student-success XAI | ✗ | The instructor lab published it 4 times |
| LLM | LLM-judge disagreement decomposition (D3) | ✅ | Ingredients exist; the integration + planted-bias validation is open |
| LLM | SAE dashboards / reliability | ✗ | Scooped in 2025–26; SAEfarer at VIS 2026 |
| LLM | Text/CLIP DR sensitivity | ◐ | FADEx and Jeon/Seo crowd the space |
| CV eval | Segmentation calibration VA | merged → D1/D4 | Calibrate → pixels |
| CV eval | Segmentation label-error triage | ◐ | Cleanlab product demo overlap |
| CV eval | Slice discovery for segmentation | ◐ | Check VibE's scope |
| CV eval | Rashomon/multiplicity for segmentation | higher-upside (§6) | No single reproduction target. That is **no longer a penalty**, since reproduction is optional (2026-09-29) |
| CV eval | Concept-based explanations | ✗ | Saturated |
| Time series | TSFM window-level failure regimes (D2) | ✅ | TIME did static pattern stratification; window-level and in-vivo are open |
| TDA | Mapper for explanations | ✗ | The lab's line through 2026; no public code |
| Science | Climate/EuroSAT XAI vs physics | ◐ fallback | Low ceiling; attribution-VA overlap |
| **EO** | Spatial-shift error maps + embedding-shift diagnosis (Sen1Floods11 / crop yield) | ◐ **strong** | Open reproduction targets; stated gap (label inconsistency vs true failure, 2606.07780); 2026 "GFMs degrade" preprints crowd the ML half |
| EO | Zero-training embedding audit (Major-TOM, AlphaEarth) | ◐ | Feasible; UMAP-dashboard novelty risk |
| EO | XAI vs spectral truth | ✗ fallback | Klotz 2025 + lab overlap |
| **Microscopy** | GT-free per-cell QC via cross-model agreement (D5) | ✅ | BISCUIT's "uncorrelated errors" assumption is untested; a reviewer asked about it and got no reply |
| Microscopy | Cell Painting UMAP reliability | ✗ | Known method, new domain |
| Microscopy | Cell tracking | ✗ | Heavy |
| **Astronomy** | Zoobot calibration vs volunteer disagreement (D4 RQ-A) | ✅ (incremental) | Aggregate noise floor already in GZ DESI; the small-N decomposed audit is open |
| Astronomy | Attributions vs ambiguity on GZ3D masks (D4 RQ-B) | ✅ higher-upside | No paper found; data verified |
| Astronomy | Embedding anomaly discovery reliability | ◐ | Could merge with D4 |
| Astronomy | Light curves / photo-z calibration VA | ◐ low | Domain overhead; partial overlap |
| **Tabular FM** | Context-row attribution + multiplicity for TabPFN/TabICL | ◐ **strong** | Only KernelICL + one aggregate study; moves monthly; Visagreement framing needed |
| Tabular FM | Failure by meta-features | ✗ | Herre et al. 2026 + McElfresh 2023 |
| Tabular FM | Conformal calibration benchmark | ◐ grade-safe core | ESANN 2026 reproduction |
| **NFL** | Big Data Bowl tracking | ✗ | Terms: "strictly confidential", destroy after the contest, contest-only use |

---

## 5. Candidate details

Each candidate follows the template you asked for in round 1. Hour budgets are SPECULATION tuned to your profile; line items are in each deep-dive file.

### 5.1 D5: Does cross-model agreement tell you which cells were segmented wrong? (microscopy)
Full notes: `lit_notes_open/deep_5_microscopy_qc.md`.

- **Research question.** On held-out open microscopy data:
  - **RQ1:** Does instance-level agreement across generalist segmenters (Cellpose-SAM, micro-SAM, CellSAM, Cellpose3/cyto3) rank per-cell segmentation quality without ground truth? It is compared against model-internal signals (Cellpose flow error and cell probability, SAM-style predicted IoU), test-time-augmentation self-consistency, and an attribute-only model (size, density, contrast).
  - **RQ2:** Is the "uncorrelated errors" assumption violated? This is measured as instance-level error consistency (Cohen's κ, the chance-corrected agreement between two raters) between models that share a backbone or training data vs. those that do not.
  - **RQ3:** Where are the *silent failures*, where all models agree and are all wrong?
- **Why it matters.** Biologists pick and QC segmenters with agreement-based tools (BISCUIT, SEG) that assume independent errors. If generalist models share their failures, those tools give false confidence exactly where it matters.
- **Evidence for the gap.**
  - FACT, full text + open reviews: BISCUIT (F1000Research 2025) states the assumption verbatim. Reviewer **Bankhead** asks whether it holds "between overlapping methods, trained on overlapping training sets". **Pape** asks for object-level disagreement. There is no author reply to Bankhead in the July 2026 version.
  - MARC (arXiv 2609.13665, Sept 2026) admits consensus "may reinforce" failure modes.
  - Zenk et al. (MedIA 2024) benchmark failure detection in *3D radiology*, *per image*, and leave other levels to future work.
  - The NeurIPS22 challenge and Miao 2026 report only coarse per-modality or S/N splits.
  - **Must cite: RBQE (arXiv 2609.10495),** the closest precedent (FACT, verified 2026-09-29). It uses two-model agreement to flag failed *polyp* segmentations and shows that referee independence matters. But it works per image, not in microscopy, and never varies shared pretraining. **Partial precedent, not a scoop.**
  - FACT, 2026-09-29: BISCUIT and MARC have 0 citers. No citer of Cellpose-SAM, micro-SAM or CellSAM tests correlated errors against GT.
  - **Spike signal (8 LIVECell test images, 2,006 cells; SYNTHESIS, too small to be a finding):**
    - Cellpose-SAM vs micro-SAM error κ = 0.52, and 80% of Cellpose-SAM's errors are shared.
    - 113 silent failures.
    - Agreement vs per-cell IoU: ρ = 0.62–0.76, or 0.51–0.62 within an image, which is comparable to Cellpose flow error (ρ = −0.63).
- **Reproduce.** The NeurIPS22 CellSeg per-modality F1 table for 1–2 models on its Public-Test set, and a Cellpose-SAM benchmark number on LIVECell test.
- **Minimum viable project (grade-safe).** Per-instance error taxonomy (merge / split / miss / false positive), attribute small multiples and an error-consistency matrix on LIVECell. This is a strict subset, done by Nov 3.
- **Stronger version.** RQ1–RQ3 with within-image AUROC, Zenk's risk-coverage metric (AURC), and a **triage view evaluated by simulated inspection**: errors found per K cells inspected, against GT. No user study is needed.
- **Data (FACT, with the leakage map checked).**
  - **NeurIPS22 Public-Test** (50 labelled images): held out from all models.
  - NeurIPS22 Tuning (101 images): **resolved 2026-09-29, treat it as model-selection-exposed** for micro-SAM v4. Its current training script uses `split="val"`, which maps to Tuning. Use **Public-Test** as the clean held-out set.
  - **LIVECell test**: Cellpose-SAM and micro-SAM trained on LIVECell *train*, so its test split is **in-distribution** for them (held-out images, not a domain shift). CellSAM never saw LIVECell.
  - NeurIPS22's ND license likely forbids publishing derived overlays, so use it for evaluation only.
  - BBBC038 is in every model's training data, so it is **unusable** for held-out claims.
  - Licenses: LIVECell CC BY-NC 4.0; NeurIPS22 CC-BY-NC-ND (fine for research).
- **Models and compute.** Repos are maintained and permissively licensed.
  - CellSAM weights need a free DeepCell API key.
  - Cellpose 4 cannot load cyto3, so cyto3 needs its own environment.
  - StarDist is a poor fit for phase-contrast whole cells.
  - Cellpose-SAM inference has been profiled on a 12 GB card; an M1 path exists.
  - **Spike (2026-09-29):** Cellpose-SAM and micro-SAM run at 0.13–1.6 s per image using ≤4.4 GB of GPU memory. **CellSAM runs** (DeepCell key obtained), but it is also SAM-based and zero-shot bimodal on LIVECell. micro-SAM's AMG default thresholds return zero masks, so use AIS. **RQ2 still needs a non-SAM model** (Cellpose3 `cyto3`, in its own env).
- **Visualization.** Error map overlaid on images; attribute small multiples; model × model error-consistency matrix; a triage list ranked by each QC signal, linked to image crops.
  - *Why:* the claim is about **which instances** fail and **whether failures co-occur**. Aggregate F1 hides exactly that.
- **Evaluation.**
  - Error-consistency κ with confidence intervals (CIs), same-lineage vs. cross-lineage pairs.
  - Within-image AUROC and AURC for each QC signal vs. the attribute-only baseline.
  - Simulated-inspection curves.
  - Pre-registered falsification criteria.
- **Risk.** Agreement may just encode image difficulty. The within-image AUROC and the attribute baseline are designed to catch this, and a negative result is still reportable. Other risks: 3 environments, and instance-matching engineering.
- **Course fit.** Model assessment, black-box signals, DL visualization; a Visagreement-like "disagreement as evidence" framing without overlap.
- **Lab hook (FACT, Visagreement full text).** Its Case Study 2 *conjectures* that model accuracy is worse where explanation methods disagree. The authors call this "the first time such a possibility has been pointed out", but say it is "not comprehensive enough to assert". D5 tests the analogous claim (disagreement signals error) rigorously, per instance, against GT, in a new modality. That is a natural way to pitch it to Silva.
- **Publication path.** Add 2–3 more datasets and a small user study with biologists. Venues: a BioImage/CVPR CVMI workshop, MICCAI workshops, or a VIS short paper. Its answer to a named reviewer question is the hook.
- **12b.** Course fit **High** · upside **Medium** (answers a named question) · technical risk **Medium** (your background lowers it) · data risk **Low–Med** · viz burden **Low–Med** · evaluation clarity **High** (GT exists).

### 5.2 D2: Window-level failure regimes of time-series foundation models
Full notes: `lit_notes_open/deep_2_tsfm_va.md`.

- **Research question.** Where do zero-shot TSFMs (Chronos-2, TimesFM-2.5, Moirai-2, TiRex) fail at the **forecast-window** level, relative to strong classical baselines (AutoETS/ARIMA/Theta, which TIME lacks) and to each other? Are failures predicted better by context-window features than by TIME's static patterns, under leave-dataset-out validation? Do Jander et al.'s synthetic failure modes (persistence overestimation, regime-switch collapse) show up "in vivo"?
- **Evidence for the gap.**
  - TIME (ICML 2026) stratifies statically by 7 features. EXPRTS and Kang 2017 cover pre-TSFM instance spaces. mTSeer (lab) asks for "more features… more models… expanded instance-level evaluation".
  - Jander et al. (Aug 2026) request "in vivo validation".
  - **Pilot, redone 2026-09-29 with the correct name-based join (SYNTHESIS):**
    - The within-series share of window-level error variance is **0.66 by MASE but 0.45 by CRPS**.
    - Chronos-2 and TimesFM-2.5 differ by more than 25% on 31.8% of windows (MASE) or 27.7% (CRPS).
    - The original pilot's feature correlations are **void**.
  - **Partial overlap found 2026-09-29: Wang et al., arXiv 2606.18367** (traffic). It stratifies per-window errors by regime and shows aggregates hide transition-window failures. **So drop any "first to show aggregates hide failures" claim.** Lead with *predicting* failure from context features across TIME, plus the in-vivo test of Jander's modes.
- **Reproduce.** TIME's pattern-level results from released outputs (pandas, about 3 h), plus a local Chronos-2 rerun on 2–3 tasks. **Done in the spike:** TIME Tables 4–5 reproduced within 0.1%.
- **Implementation rule.** Always join by `(dataset_id, series_name, variate_name)`, never by row position.
- **MVP.** Classical baselines, context features, failure labels, variance decomposition, a leave-dataset-out failure predictor, and 5 linked views. **No TSFM inference needed:** TIME-Output releases per-window predictions for 30 models under Apache-2.0.
- **Stronger version.** The in-vivo dose-response test; a context-length what-if probe; GIFT-Eval replication.
- **Viz.** Failure-regime map, conditional-error small multiples, leaderboard-flip matrix, window drilldown, held-out slice table.
- **Evaluation.** Variance shares (with CRPS and a noise floor); ΔAUROC ≥ 0.03; held-out slice lift ≥ 1.5×; dose-response CIs.
- **Risk.** Features may explain little of window-level failure.
  - With the corrected join, the strongest single-feature correlations are about |ρ| ≈ 0.21 (seasonal strength, entropy).
  - A July 2026 impossibility result argues against spectral features.
  - H1 (most variance is within series) holds for MASE but not for CRPS, so pre-register H1 per metric.
  - Chronos-2 vs Chronos-Bolt is **not** a valid noise floor; use seed or sample reruns.
  - A null result is still reportable. The field moves weekly, and the novelty margin is thinner after Wang et al.
- **12b.** Fit **High** · upside **Medium** · technical **Low–Med** · data **Low** · viz **Low–Med** · evaluation **High**.

### 5.3 D4: Zoobot vs. volunteer disagreement (astronomy)
Full notes: `lit_notes_open/deep_4_astro_zoobot.md`.

- **RQ-A (grade-safe).** Is Zoobot's Dirichlet-multinomial posterior calibrated to the observed volunteer votes, per question and per stratum (question depth, magnitude, redshift, size, imaging region), once counting noise at 5–40 votes is corrected for? Which component tracks human disagreement: **aleatoric** (predicted ambiguity, the uncertainty the model expects to be inherent in the image) or **epistemic** (MC-dropout / 5-seed ensemble spread, the model's own uncertainty about its parameters)?
- **RQ-B (higher upside).** For "bar?" and "spiral arms?", do attributions (IG, Grad-CAM, SmoothGrad, occlusion) agree less, and fall on GZ3D volunteer masks less often, for volunteer-ambiguous galaxies? Is human ambiguity or model uncertainty the better predictor of explanation unreliability?
- **Evidence for the gap.**
  - *Already done (FACT, full text):* the GZ DESI noise floor ("as accurate as ~15 volunteers"); GZ DECaLS calibration for only 2 binary questions; soft-label models tracking annotator entropy (general ML).
  - *Open:* small-N-corrected instance calibration, which Baan et al. 2022 list as open. Aleatoric/epistemic decomposition vs. human disagreement. **No paper evaluates galaxy-CNN attributions against GZ3D masks.** Bhambra 2022 used bar *lengths*, and its future work suggests Zoobot-style models.
- **Reproduce.** GZ DESI Fig. 5 (noise floor) and GZ DECaLS Fig. 19 (calibration).
- **Data (FACT).**
  - GZ DESI on HF: CC-BY-NC-SA-4.0, not gated. Its **clause requires releasing trained-model source code at publication**. The `tiny` config (798 test galaxies) is too small to slice.
  - GZ3D on SDSS DR17 (verified live): 29,813 FITS files with 4 volunteer-mask layers. About 9,800 galaxies have ≥3 bar drawers.
  - Legacy Survey cutouts align on the sky.
- **Models.** Zoobot is GPL-3.0, v2.9, maintained. The HF encoders (Apache-2.0) are encoders only, so you must fine-tune a decision-tree head.
- **Viz.** A Calibrate-style reliability view for *soft, multi-question* labels (Calibrate handles hard binary labels only, which is the hook); an image grid linked to vote distributions; attribution overlays with masks; slice views.
- **Evaluation.** PIT and coverage tests with pre-registered thresholds (RQ-A); pointing-game/IoU against masks **vs. a brightness baseline** (RQ-B).
- **Risk (in-lab).** Visagreement's authors list adapting the tool to **image** data as near-future work (FACT, full text). RQ-B (attribution agreement on galaxy images) is therefore both a welcome extension and a possible in-lab scoop. Ask Silva.
- **Spike (2026-09-29): PASS.** Fine-tuning takes 3 s per epoch on an A10. The GZ3D mask aligns with its Legacy Survey cutout to about 1 px.
  - Bar masks cover pixels 30× brighter than the cutout mean, so **the brightness baseline is mandatory**.
  - Use Legacy Survey cutouts as model input, because GZ3D images have the MaNGA hexagon drawn on them.
- **Risk.** **Brightness confound.** Bars are central and bright, so attributions must beat a light-profile baseline. There is also scoop risk from the Walmsley/Masters/Spindler group (who own the data), and a ZooBot:3D segmentation model now exists. RQ-A alone is incremental.
- **12b.** Fit **High** · upside **Medium** (RQ-B → ML4PS / VIS short) · technical **Medium** · data **Low** · viz **Low–Med** · evaluation **High** (RQ-A) / Med-High (RQ-B).

### 5.4 D3: Decomposing LLM-judge disagreement (LLM; education option)
Full notes: `lit_notes_open/deep_3_llm_judges.md`.

- **RQ.** With local judges (gpt-oss-20b, qwen3.8:27b) plus the released GPT-4 judgments on MT-Bench items that have ≥2 human votes: split item-level judge–human disagreement into **human ambiguity, judge instability, feature-linked bias, and rationale divergence**, and recover planted biases that pooled κ hides.
- **Gap.** LLM Comparator (TVCG 2025, archived) is single-judge. The 2026 statistical papers (Coin Flip, Geometry EMNLP 2026, Nine Judges) each cover one layer. VIS 2025/26 and about 55 citers of LLM Comparator show no multi-judge VA (FACT).
- **Reproduce.** Zheng et al. Table 5 **exactly**: 66/85/63/81%, from CC-BY-4.0 data (1.4 MB, 961 cells with ≥2 votes).
- **Evaluation.** Planted P1–P3 must be flagged in the top 3 with BH q < 0.05; a null control must produce no flags.
- **Compute (measured 2026-09-29).** On one L40S (46 GB), both judges fit in GPU memory at once: qwen3.6:27b at about 70 tok/s, serial only; gpt-oss:20b at 113 tok/s with 4 parallel requests. **The minimum run (3,844 calls) takes about 4.7 h.** On the shared lab server it would take about 30 h for one judge.
  - Note: the cluster used `qwen3.6:27b`, and your GPU box has `qwen3.8:27b`. Pick one and stick with it.
- **Reproduction done in the spike:** MT-Bench agreement reproduced exactly (66.0 / 84.6 / 63.0 / 81.0%).
- **Education stretch.** ASAP 7/8 or PERSUADE 2.0, including a DIF (differential item functioning) slice.
- **Risk.** "Not novel", since it is structurally Visagreement-for-judges, and scoop risk from PAIR, KAIST and IBM. Also (FACT, full text) Visagreement's authors list adapting the tool to **text** data as "a challenge we intend to address in the near future". Ask Silva whether that is in progress.
- **12b.** Fit **High** · upside **Medium** · technical **Low–Med** · data **Low** · viz **Medium** · evaluation **High**.

### 5.5 D1: Tile2Net calibration and error attribution (maps, default project)
Full notes: `lit_notes_open/deep_1_tile2net_segrel.md`.

- **RQ.** On held-out NYC imagery (Queens):
  - (1) where is pixel confidence miscalibrated?
  - (2) what share of network topology errors comes from segmentation vs. the vectorizer? This is tested by running Tile2Net's own vectorizer on the GT mask (an oracle-mask ablation).
  - (3) *stretch, gated:* does a path "bottleneck confidence" separate true missing links from true gaps?
- **Gap.** The tool half is done (SegNetVis). Tile2Net's confidence calibration and the error attribution are open (no hit among 52 Tile2Net citers or 20 Calibrate citers). Tile2Net's authors ask for "probabilistic" gap filling as future work. PathwayBench shows fragmentation but not its cause.
- **Feasibility (FACT).** The inference path saves no probability map (about a 10–20 line patch); CUDA-only; NYC Planimetrics 2022 + 2022 orthos are open and aligned; no crosswalk GT; Manhattan is in the training data.
- **Risk.** The vectorizer may dominate, which is still reportable. Install friction; classmate overlap.
- **12b.** Fit **Excellent** · upside **Medium** · technical **Medium** · data **Low–Med** · viz **Medium** · evaluation **High** (RQ1/2) / Med (RQ3).

### 5.6 Soccer C1 and C2 (round 1)
Full notes: `lit_notes/`, plus `lit_notes/round1_soccer_review.md` §11.

- **C1: physics-anchored evaluation of explanation disagreement for xG.** Does cross-method or cross-model disagreement flag explanations that violate shot geometry (mirror invariance, iso-distance arcs, monotonicity)? It extends Visagreement's open question. FACT, full text: its Case Study 1 found "no definitive correlation" between disagreement and explanation quality, but quality was measured only by the proxy metrics sensitivity and infidelity. SYNTHESIS: a GT-based test of that relationship is still open, and soccer geometry could supply one. **Narrowed 2026-09-29:** Cefis & Carpita 2024 already compare xG explanations across models *globally* (Spearman ρ on SHAP and RGE rankings). So the claim must be *local, per-shot* disagreement against physics GT. Data: StatsBomb open. Risk: every method may simply agree on distance/angle. 12b: fit Excellent · upside Moderate (my earlier "Moderate–High" was generous; it is on par with D2) · technical Low–Mod · data Low · viz Moderate · evaluation Med–High.
- **C2: split-regime dependence of soccer explanations (VAEP, xG).** Holding features fixed, how do calibration and SHAP change across random, by-match and leave-one-tournament-out splits? Gap: Davis 2024 gives no explanation protocol; Peters 2026 covers feature leakage only. Risk: a null result for xG. 12b: fit Very good · upside Moderate · technical Low · data Low · viz Low–Mod · evaluation High.

### 5.7 Strong scan-level options (not deep-dived; treat as provisional)

| Option | Grade-safe core | Higher-upside angle | Why not deep-dived | Check first |
|---|---|---|---|---|
| **EO spatial-shift diagnosis** (`scan_6`) | Reproduce Adjei's leave-one-country-out crop-yield study (arXiv 2605.08113, code + data) or Sen1Floods11 leave-one-event-out | Error/uncertainty map linked to embedding-shift diagnosis. It separates label inconsistency from true failure, a stated gap in 2606.07780 | 2026 "GFMs degrade under shift" preprints crowd the ML half; novelty must come from the diagnostic VA layer | Sen1Floods11 license; EarthShift and the calibration preprints |
| **Tabular-FM context attribution** (`scan_9`) | Reproduce the ESANN 2026 conformal-coverage benchmark (TabPFN vs GBDT) | Which context rows drive a TabPFN/TabICL prediction: fidelity and stability under resampling and reordering, with a linked DR view | Moves monthly; Visagreement framing required | KernelICL (2602.02162) and follow-ups; TabPFN ≥2.5 weights are non-commercial and gated (TabPFN-2 and TabICL are open) |
| **KT explanation reliability** (`scan_2`) | pyKT + deletion faithfulness | Cross-explainer disagreement on DKT/SAKT/AKT | Reads as "Visagreement on KT" | dblp overlap |

---

## 6. Higher-upside tier

**Why grade-safe designs top out at Medium (SYNTHESIS).**
- **Constraints cap the ceiling.** High upside usually needs a new method, a user study, big data, or being first in a fast area. All of those conflict with a solo 40 h, open-data project.
- **The open gaps are narrow.** They are integration or measurement gaps.
- **I rated upside as expected value, including null-result risk.**

**Upside depends on results more than on the topic.** These are the versions that could reach "High" (a VIS full paper or a strong workshop paper):

| Higher-upside option | Built on | What would make it High | Extra risk vs. its grade-safe version | Grade-safety if chosen as the *main* project |
|---|---|---|---|---|
| **D4 RQ-B: human ambiguity → explanation unreliability (GZ3D masks)** | D4 | A clean result that attributions track masks beyond a brightness baseline, and fail more where volunteers disagree. This offers a general lesson for XAI evaluation with soft labels | Brightness confound; mask alignment; scoop | Medium (keep RQ-A as a fallback) |
| **D5 full: correlated-error audit + GT-free triage** | D5 | Shows BISCUIT-style QC fails for same-lineage models, and a QC signal beats attribute baselines within images | Agreement may be trivial; 3 environments | Medium-High (the taxonomy MVP is a floor) |
| **D2-max: in-vivo test as the headline** | D2 | Synthetic TSFM failure modes confirmed on real benchmarks, with held-out regimes | A null result is likely-ish | Medium-High (MVP needs no inference) |
| **D1-max: probabilistic gap triage + routability repair** | D1 | Calibrated bottleneck confidence repairs networks and improves PathwayBench traversability | Plumbing, GPU install | Medium |
| **Rashomon / multiplicity VA for segmentation** | scan_4 | The first VA of seed-level disagreement for segmentation, tied to annotation ambiguity | Multi-seed training compute. ("No reproduction target" no longer counts against it, since reproduction is optional) | Medium |
| **D3 + education DIF** | D3 | Local LLM graders' subgroup bias at equal quality, which the 2026 audit lists as out of reach | Licensing, local-model rating reliability | Medium |
| **Tabular-FM context attribution** | scan_9 | The first VA/faithfulness study of which context rows drive TabPFN predictions | Fast-moving; not deep-verified | Medium-Low |

---

## 7. Load-bearing literature by candidate

A condensed list. Full columns (RQ, data, method, viz, evaluation, limitations, code, evidence level) are in the per-area files. Evidence: **F** = full text, **A** = abstract, **2** = secondhand.

| Candidate | Key papers (venue, status) | Role | Ev |
|---|---|---|---|
| D5 | BISCUIT (F1000Research 2025, peer-reviewed with open reviews) | The untested assumption + reviewer challenge | F |
| D5 | Zenk et al., failure detection benchmark (MedIA 2024) | Method template; image-level, radiology only | F |
| D5 | NeurIPS22 CellSeg (Ma et al., Nature Methods 2024); Cellpose-SAM (bioRxiv 2025); micro-SAM (Nature Methods 2025); CellSAM | Reproduction targets and models | F/A |
| D5 | MARC (arXiv 2609.13665, 2026); SEG (bioRxiv 2023); Chen & Murphy (MBoC 2023) | Consensus-based QC; admits reinforcement risk | F |
| D5 | **RBQE (arXiv 2609.10495, 2026)** | Closest precedent: two-model agreement for polyp failure detection (image-level) | A (verified) |
| D2 | TIME (ICML 2026); mTSeer (CHI 2021, lab); EXPRTS (arXiv); Kang et al. (IJF 2017) | Prior art and reproduction target | F |
| D2 | Jander et al., Causal analysis for TSFMs (arXiv 2026); *The Spectrum Is Not Enough* (arXiv 2026) | Named open question; main risk | F/A |
| D2 | **Wang et al. (arXiv 2606.18367, 2026)** | Partial overlap: per-window regime stratification (traffic) | A (verified) |
| D4 | Walmsley et al., GZ DECaLS (MNRAS 2022); GZ DESI (2023); Zoobot (JOSS 2023); Walmsley 2020 | Reproduction targets; what's already done | F |
| D4 | Masters et al., GZ3D (2021); Bhambra et al. (2022) | Masks; closest attribution work | F |
| D4 | Baan et al. 2022 (calibration to human disagreement); Singh et al. 2025/26 | General-ML baseline; open small-N problem | F |
| D3 | LLM Comparator (TVCG 2025); Zheng et al. MT-Bench (NeurIPS 2023); EvalGen (UIST 2024) | Targets and prior VA | F |
| D3 | Coin Flip Judge (arXiv 2026, single author); Geometry of LLM-as-Judge (EMNLP 2026); Nine Judges (arXiv 2026); Sunkavalli essay audit (arXiv 2026) | What's already known | F |
| D1 | Tile2Net (CEUS 2023); Calibrate (TVCG 2023); PathwayBench (arXiv 2024); Prophet (arXiv 2024); Gupta et al. (NeurIPS 2023) | Targets and neighbours | F |
| Soccer C1 | Visagreement (TVCG 2025); MOUNTAINEER (TVCG 2024); Krishna et al. disagreement (TMLR 2024); Tsai et al. (arXiv 2026); PassAI (IEEE Access 2025); **Cefis & Carpita (*Statistics* 2024; read in full: global-only cross-model concordance)** | Prior art; the 0/25 local-agreement finding | F/A |
| Soccer C2 | Davis et al. (Machine Learning 2024); Peters et al. (RQES 2026) | Methodology gap | F/A |
| General | Nauta et al. (ACM CSUR 2023); Jeon et al., "Stop Misusing t-SNE and UMAP" (VIS 2026); Atzberger et al. (VIS 2024) | Framing | A/F |

---

## 8. Tempting but bad choices

| Tempting idea | Why it's bad here |
|---|---|
| SHAP vs LIME vs IG dashboard (any domain) | Visagreement / MOUNTAINEER / Explainalytics already do this, from the grader's lab |
| LLM explains a soccer or tabular model, then check faithfulness | Rahimian 2025 + Ichmoukhamedov 2024 + the 2026 verify-and-repair wave |
| Embeddings → UMAP → "discover" clusters (soccer, cells, EO, galaxies) | Done repeatedly; circular when inputs reappear in the plot; Jeon et al. document the misuse |
| Tactical phase segmentation | Proprietary tracking; single-annotator labels; vendor formation agreement only 30% |
| Pixel ↔ network tool for Tile2Net | Last year's cohort built it |
| SAE dashboards / SAE stability | Scooped in 2025–26; SAEfarer at VIS 2026 |
| Mapper/TDA for explanations | The lab's own line; no public code |
| Student-success explainer disagreement | The lab published it 4 times |
| "Is LLM grading reliable?" | Several preprints per month |
| "Stratify TSFM errors by seasonality/trend" | TIME (ICML 2026) did it |
| "GFMs degrade under geographic shift" (as the finding) | 2026 preprints (EarthShift and others) |
| Tabular-FM failure by dataset meta-features | Herre et al. 2026 + McElfresh 2023 |
| Cell Painting UMAP reliability | A known method in a new domain |
| NFL Big Data Bowl | Terms forbid non-contest use and redistribution; data must be destroyed after the contest |
| PassAI reproduction | Proprietary data, no code, arrival-point leakage |
| SoccerNet / broadcast video, GraphCast/ERA5, SoccerTrack | NDA or heavy compute; not a Vis4ML question within 40 h |
| Controlled user study as the main contribution | Recruiting, IRB, statistical power |
| Polished standalone D3 app | Your weakest skill; the course rewards the question |

---

## 9. Side-by-side comparison

### 9a. Evidence and scope

| Candidate | Evidence of gap | Closest prior work | Public data/code | Reproduction target | Minimum viable extension | Eval clarity | Viz fit | Solo feasibility | Main failure mode | Evidence still needed |
|---|---|---|---|---|---|---|---|---|---|---|
| **D5 Cell-seg QC** | BISCUIT assumption untested; reviewer question unanswered; Zenk is image-level, radiology only | BISCUIT, SEG, MARC, Zenk, NeurIPS22 | LIVECell (NC), NeurIPS22 (NC-ND); repos open; CellSAM key | NeurIPS22 per-modality F1; Cellpose-SAM LIVECell number | Error taxonomy + consistency matrix (MVP) → within-image QC AUROC + simulated triage | **High** | High | Good (~40 h) | Agreement = difficulty only | ✔ dblp, citers and Tuning checked (2026-09-29). Still needed: a non-SAM model (cyto3); a Public-Test run; cite RBQE |
| **D2 TSFM regimes** | TIME is static; pilot variance share; Jander requests in-vivo | TIME, mTSeer, EXPRTS | TIME-Output Apache-2.0 | TIME pattern tables; Chronos-2 rerun | Classical baselines + context features + leave-dataset-out predictor | **High** | High | **Very good** | Features explain little; thinner novelty (Wang et al.) | ✔ row order fixed, CRPS pilot done. Still needed: weekly scoop check |
| **D4 Zoobot** | Aggregate noise floor only; no GZ3D attribution study | GZ DESI/DECaLS, Bhambra 2022, Baan 2022 | GZ DESI CC-BY-NC-SA; GZ3D DR17; Zoobot GPL | GZ DESI Fig. 5; DECaLS Fig. 19 | Small-N decomposed calibration (+ GZ3D attribution) | High / Med-High | High | Good | Brightness confound; scoop | ZooBot:3D paper; Walmsley group 2026 output |
| **D3 LLM judges** | No multi-judge × human VA | LLM Comparator, Visagreement, Coin Flip | MT-Bench CC-BY-4.0 | Zheng Table 5 exact | Decomposition views + planted biases + null | **High** | High | Good | "Not novel" / scoop | ✔ reproduction exact, throughput measured (~4.7 h). Still needed: CHI/IUI 2026 sweep |
| **D1 Tile2Net** | No calibration or attribution among citers; SegNetVis has neither | Tile2Net, Calibrate, PathwayBench | NYC open; BSD-3/MIT | Tile2Net Table 4; Calibrate curve | Stratified pixel calibration (+ oracle ablation) | High / Med | High | Good (~30–40 h) | Vectorizer dominates; install | Held-out boroughs; softmax spike |
| **Soccer C1** | 0/25 local-agreement studies | Visagreement, MOUNTAINEER | StatsBomb open | Visagreement on xG MLP | Semantic perturbations + disagreement-predicts-violation | Med-High | High | Good | Trivial agreement | Cefis & Carpita full text |
| **Soccer C2** | No split-only study | Davis 2024, Peters 2026 | StatsBomb + Wyscout | socceraction VAEP | 3 regimes × 5 seeds, SHAP change | **High** | Med-High | **Very good** | Null for xG | Peters full text |
| *EO shift diagnosis* (scan) | Stated label-inconsistency gap | Adjei 2026, Kolluru 2026 | Mostly open (verify) | Adjei LOCO | Error map + embedding-shift view | Med-High | High | Good | Crowded ML half | Licenses; 2026 preprints |
| *Tabular-FM context attribution* (scan) | Thin (KernelICL only) | KernelICL, ESANN 2026 | TabICL BSD; TabPFN-2 Apache | ESANN coverage | Context-row fidelity + stability view | Med-High | Med-High | Good | Scooped monthly | Deep verification |

### 9b. Risk and reward

These ratings are qualitative, and each has its reason in §5. No overall score is given here; see the ranking in §10.

| Candidate | Course fit | Research upside | Technical risk | Data risk | Viz burden | Eval clarity | Best suited if… |
|---|---|---|---|---|---|---|---|
| **D5 Cell-seg QC** | High | Medium (named open question) | Medium | Low–Med | Low–Med | **High** | …you want your segmentation skills + objective GT-validated results + no lab overlap |
| **D2 TSFM regimes** | High | Medium | **Low–Med** | **Low** | Low–Med | **High** | …you want the lowest setup risk and accept a possible null |
| **D4 Zoobot** | High | Medium (High if RQ-B works) | Medium | Low | Low–Med | High | …astronomy excites you and you'll control the brightness confound |
| **D3 LLM judges** | High | Medium | Low–Med | Low | Medium | High | …you want an exact reproduction + LLM portfolio + optional education |
| **D1 Tile2Net** | **Excellent** | Medium | Medium | Low–Med | Medium | High / Med | …maps + instructor buy-in matter most, and the install spike passes |
| **Soccer C1** | Excellent | Moderate | Low–Mod | Low | Moderate | Med–High | …soccer motivation matters most |
| **Soccer C2** | Very good | Moderate | Low | Low | Low–Mod | High | …you want the safest soccer path; viz supporting |
| *EO shift diagnosis* | High | Med–High | Medium | Low–Med | Medium | Med–High | …maps/EO interest and you'll verify licenses early |
| *Tabular-FM context attribution* | High | Med–High | Low–Med | Low | Medium | Med–High | …you want a fresh, cheap area and accept fast-moving scoop risk |

---

## 10. Ranking: top 5 (updated 2026-09-29, after the spikes)

> **See also:** `research_gap_review.md` §3. It has this grade-first ranking with extra fields (upside, difficulty, compute, scoop risk, Silva fit), plus an **ambitious / publication-first ranking (3b)** for the Fall 2028 PhD cycle.

**Weights:** (1) **grade safety**, meaning feasibility, evaluation clarity and low setup risk; (2) **publication path**, meaning a named open question and no scoop; (3) **your interests** as the tie-breaker. The spike results (§12) are factored in.

| Rank | Candidate | Why it is here | Main risk | What would move it |
|---|---|---|---|---|
| **1** | **D5: cross-model agreement as ground-truth-free cell-segmentation QC** (microscopy) | Objective GT-based evaluation; a **specific, unanswered reviewer question** (BISCUIT); no lab overlap (dblp-verified); your strongest skill; **the spike passed** (both models run in <2 s per image, and early κ = 0.52 with 113 silent failures says the question is live) | "Agreement = difficulty" triviality; still needs a non-SAM model (cyto3); must cite RBQE | ↓ if cyto3 won't run or Public-Test shows no structure; ↑ if Silva likes the "test the Visagreement conjecture" pitch |
| **2** | **D2: window-level failure regimes of time-series foundation models** | **Lowest execution risk** (released outputs, no inference, TIME reproduced within 0.1%); pre-registered, falsifiable criteria; lab hook (mTSeer) | **Novelty thinner after the spikes** (Wang et al. overlap; H1 fails for CRPS; features \|ρ\| ≈ 0.2, so a null result is plausible); none of your interest domains | ↓ if the weekly scoop check finds window-level failure *prediction* on TIME; a close call with #3 and #4 |
| **3** | **D4: Zoobot vs. volunteer disagreement** (astronomy) | Your interest; **the spike passed easily** (3 s per epoch; GZ3D masks align to about 1 px); RQ-B (explanations vs. human ambiguity on GZ3D masks) is the **most distinctive story** of any candidate | The brightness confound is real (bar pixels 30× brighter); RQ-A alone is incremental; **Visagreement's authors plan an image extension** (possible in-lab scoop); the data owners could scoop RQ-B | ↑ above D2 if Silva says the image extension is not in progress and you want the higher-upside path |
| **4** | **D3: decomposing LLM-judge disagreement** (LLM; education option) | **Exact reproduction** (MT-Bench 66.0 / 84.6 / 63.0 / 81.0%); planted-bias evaluation with a null control is objective; **the minimum run is now about 4.7 h** on an L40S (not 13–46 h) | Novelty is "integration + validation"; it is structurally Visagreement-for-judges; **Visagreement's authors plan a text extension**; PAIR/KAIST/IBM could ship it first | ↑ if you value the LLM portfolio and Silva confirms no in-lab text work |
| **5** | **Soccer C1: local, physics-anchored explanation disagreement for xG** | Your soccer motivation; trivial data access (StatsBomb open); the gap **survived** Cefis & Carpita (their comparison is global-only); ties to Visagreement's Case Study 1 (quality was measured only by proxies) | Every method may simply agree on distance/angle, giving a trivial result; Visagreement/MOUNTAINEER overlap needs careful framing; no spike was run | ↑ if soccer matters more to you than domain novelty |

**Honorable mentions:**
- **D1 (Tile2Net):** the best instructor alignment (the default project), but it has the most classmate overlap and its install spike was skipped. It jumps into the top 3 only if Silva strongly prefers the default project.
- **Soccer C2 (split-regime explanations):** the safest soccer option with the clearest evaluation, but the visualization plays a supporting role.
- **EO spatial-shift diagnosis** and **tabular-FM context attribution:** promising, but only scan-level, not deep-dived or spiked.

**How to use this on Oct 6:** pitch **#1 plus one or two of #2–#4**. For example: D5 (safest strong), D2 (lowest risk) and D4 (your interest). Then let Silva's answers on the Visagreement image/text extension and on lab-tool extensions break the tie.

**Update history:**
- The earlier top 2 were D2 then D3 (before D4 and D5 were reviewed), then D5 then D2 (after the round-3 deep dives).
- After the spikes (2026-09-29), D5 stays #1. D2 stays #2, but with a thinner margin. D3 rises in feasibility but not in novelty.

---

## 11. What you should verify yourself

These are the checks most likely to change the conclusions.

**Status as of 2026-09-29:**
- ✅ **Done:** completed by the verification/spike session. Evidence is in "Verified on 2026-09-29" below, §12 Spike results, and `spike_results/`.
- 🟡 **Open, yours:** only you can do it (ask a person, read, or decide).
- ⏳ **Open, later:** do it after you choose a topic, or on a schedule.
- ➖ **Only if needed:** matters only for a specific candidate.

**Still on your plate:** items 2, 3(a), 4(c) and 11 (item 7 only if you pick D1). Everything else is done.

1. ✅ **dblp for Silva, Nonato, Miranda, Barr and Bertini, 2023–2026.** Done via dblp SPARQL + OpenAlex. **No lab paper touches microscopy, astronomy, TSFMs or LLM-as-judge.**
2. 🟡 **Prior-cohort projects.** *Optional.* Ask the TA for last year's project list. The GitHub search was done in earlier rounds, and it only found the Tile2Net repos.
3. **D5:**
   - (a) 🟡 read BISCUIT's open reviews yourself. *Optional.* They are the hook for your pitch. The session confirmed the author response still does not answer Bankhead.
   - (b) ✅ BISCUIT/MARC citers: 0 citers. New must-cite precedent: **RBQE (arXiv 2609.10495)**, a partial precedent, not a scoop.
   - (c) ✅ NeurIPS22 Tuning: treat it as model-selection-exposed for micro-SAM v4. Use **Public-Test** as the held-out set.
   - (d) ✅ Cellpose-SAM + micro-SAM installed and run on 8 LIVECell test images (A1 **PASS**). CellSAM runs too.
4. **D2:**
   - (a) ✅ TIME row ordering: it **was wrong** in the old pilot. The fix is a name-based join, which reproduces TIME within 0.1%.
   - (b) ✅ Pilot re-run with CRPS: the within-series share is 0.45 (vs 0.66 by MASE).
   - (c) ⏳ weekly arXiv scoop check on TIME and Jander citers. It was done once (new partial overlap: Wang et al. 2606.18367); repeat weekly until Oct 20 if you pick D2.
5. **D4:**
   - (a) ✅ ZooBot:3D checked: no attributions or calibration analysis, so no scoop.
   - (b) ✅ Walmsley group 2025–26 checked (SAE-on-Zoobot, GZ Evo): tangential.
   - (c) ✅ GZ3D mask aligns with the Legacy Survey cutout to about 1 px (A4 **PASS**).
6. ✅ **D3:** MT-Bench numbers reproduced exactly (66.0 / 84.6 / 63.0 / 81.0%). Ollama timing measured: the minimum run takes about 4.7 h on an L40S.
7. ➖ **D1 (only if you pick Tile2Net):** ask which NYC boroughs are held out, and run the Boston example. Skipped at your request.
8. ✅ **Soccer C1:** both Cefis & Carpita papers read. C1 narrows to *local, per-shot* disagreement.
9. ✅ **Licenses:** all checked (details below). One gap remains: no explicit GZ3D/SDSS license was found, so cite SDSS's standard acknowledgment.
10. ✅ **JS/D3 expectation:** resolved. Silva says it is not required (§2).
11. 🟡 **Ask Silva on Oct 6:**
    - (a) whether extending a lab tool is welcome (Calibrate, Visagreement, mTSeer);
    - (b) **whether Visagreement's planned image/text extension is already in progress** (§13.2);
    - (c) which pitched topic he prefers;
    - (d) confirm your solo status.

### Verified on 2026-09-29
Full notes: `spike_results/partB_dblp_licenses.md` and `spike_results/partB_scoop_checks.md`. Items 3(d), 4(a–b), 5(c) and 6 are covered by the spikes in §12.

- **Item 1, dblp.**
  - The dblp web API is now behind an Anubis bot check. The official SPARQL endpoint worked, cross-checked against OpenAlex.
  - FACT: 2023–26 records (non-CoRR in parentheses): Silva 66 (37), Nonato 33 (21), Miranda 39 (24), Barr 19 (11), Bertini 14 (10), after disambiguation.
  - FACT: **no title by any of the five touches cells or microscopy, galaxies or astronomy, TSFMs, or LLM-as-judge.**
  - SYNTHESIS: the nearest items are tangential and useful as citations:
    - Visagreement (D5/D3 framing);
    - TiVy, Nonato's time-series vis review, and Bertini's COVID multi-forecast study (D2);
    - Calibrate, Mountaineer, and Barr's faithfulness-metric disagreement paper (D4).
  - Outside the lab: arXiv 2608.14106 "Forecast Collapse in TSFMs" is tangential to D2.
- **Item 3(b), D5 scoop.** FACT: BISCUIT and MARC have **0 citers**.
  - FACT: BISCUIT's 2026-07-06 author response adds object-level scores but still does not answer Bankhead's independence question.
  - FACT: none of the 547 keyword-filtered citers of Cellpose-SAM and micro-SAM, nor the 69 citers of CellSAM, tests correlated errors or per-cell agreement against GT.
  - **New closest precedent (must cite): RBQE, arXiv 2609.10495**. It uses two-model agreement to flag failed polyp segmentations, and shows that referee independence matters. But it is image-level, not microscopy, and never varies shared pretraining.
  - Verdict: partial precedent, **no scoop**.
- **Item 3(c), NeurIPS22 Tuning.**
  - FACT: micro-SAM's *paper-era* (v2) code used Tuning as a **test** set.
  - FACT: the *current* LM generalist training script uses torch-em `split="val"`, which has mapped to `Tuning.zip` since May 2024.
  - SYNTHESIS: today's default `vit_b_lm` (v4) very likely used Tuning for checkpoint selection. **Treat Tuning as model-selection-exposed for micro-SAM v4, and Public-Test (50 images) as the clean held-out set.**
  - Cellpose-SAM trained on 616 Training-labeled images only (earlier session's reading; bioRxiv was unreachable today).
- **Item 5(a–b), D4 scoop.** FACT: ZooBot:3D (arXiv 2606.16507) is a U-Net predicting per-pixel volunteer-vote fractions, with no attributions and no calibration analysis.
  - The SAE-on-Zoobot paper (2510.23749), GZ Evo, and Butterworth & Spindler 2026 are tangential.
  - Verdict: **no scoop**; the risk from the group that owns the data is unchanged.
- **Item 4(c), D2 scoop.** FACT: TIME has 19 citers, all benchmark or model papers. Jander et al. has 0.
  - **New partial overlap: Wang et al., arXiv 2606.18367** (traffic). It stratifies per-window errors by regime and shows aggregates hide transition-window failures.
  - SYNTHESIS: **drop any "first to show aggregates hide failures" claim.** Lead with *predicting* failure from context features across TIME, plus the in-vivo test of Jander's failure modes.
- **Item 9, licenses (FACT).**
  - **LIVECell:** CC BY-NC 4.0 for data and models, MIT for code. It is a plain public S3 bucket with no AWS Open Data Registry entry.
  - **NeurIPS22 CellSeg:** Zenodo 10719375, CC BY-NC-ND 4.0. The 50 Public-Test images are inside the 2.9 GB `Testing.zip`. SYNTHESIS: ND likely forbids hosting overlays or derived masks in a public tool, so use it for internal evaluation only.
  - **GZ DESI (HF):** CC BY-NC-SA 4.0, plus "all models trained on these datasets [must] be released as source code by publication". The Zenodo catalogues (8360385, 4573248) are CC BY 4.0 but have no images.
  - **TIME:** the data is CC BY-NC 4.0 since 2026-05-25. **TIME-Output is Apache-2.0.** The GitHub code has no LICENSE file, and the README and pyproject disagree (MIT vs Apache).
  - Still open: no explicit GZ3D/SDSS license was found.
- **Item 8, Soccer C1: both Cefis & Carpita papers read, 2026-09-29.** Details are in `lit_notes/01_soccer_xai.md` §3.3.
  - **The target paper**: *Statistics* 59(2):426–445, doi 10.1080/02331888.2024.2445305. FACT:
    - 8 classifiers on 7,801 Serie A shots (train 22/23, test 23/24), with 26 features including proprietary tracking.
    - Explanations are **global only**: SHAP and RGE feature rankings per model.
    - **Cross-model concordance = Spearman ρ between the global rankings**: statistical models ρ > 0.7, ML models < 0.7, cross-group about 0.5.
    - The top features are consistent: distance (x), shot angle, shooter visual angle.
    - There is no per-shot or local analysis, no perturbations and no geometric ground-truth checks.
    - 13 citers (Semantic Scholar); none does local xG explanation disagreement.
  - **The earlier paper you sent**: *"A new xG model for football analytics"*, JORS 76(1), doi 10.1080/01605682.2024.2323669. FACT: logistic regression only, 660 shots, odds ratios plus hand-built what-if scenarios. It does not bear on C1.
  - SYNTHESIS:
    - **C1 survives, but its novelty claim must narrow.** "First cross-model comparison of xG explanations" is **falsified** at the global level. The defensible gap is **local, per-shot disagreement across methods and models, evaluated against physics-derived ground truth** (mirror symmetry, monotonicity, iso-distance arcs).
    - Cite Cefis & Carpita 2024 as the global baseline that C1 goes beyond.
    - Their linear-y logistic models, which cannot be mirror-symmetric, are a ready example of why global rank agreement (ρ about 0.5–0.7) does not certify local explanations.
    - The ML-overpredicts-goals result (RF +19%) is a small hook for C2 (calibration by split regime).
    - Their data is proprietary, so C1 stays on StatsBomb open.
    - C1's position in the ranking does not change.

---

## 12. Next steps before Oct 20

- **Now to Oct 5 (about 3 h):** one 1–2 h feasibility spike each for your top 2.
  - **D5:** install Cellpose-SAM and micro-SAM, then run them on 5 LIVECell test images and compute per-cell IoU against GT.
  - **D2:** download one TIME-Output model folder, check the row ordering, and reproduce one pattern-level number.
  - If astronomy wins your heart instead, **D4:** fine-tune a Zoobot head on GZ DESI for one question.
- **Oct 6 (project discussion):** pitch 2–3 one-paragraph options to Silva. Ask about lab-tool extensions, and about solo status.
- **Oct 7–20:** write the proposal.
  - Related work: reuse §4–5 of the chosen deep-dive file.
  - Method: state pre-registered hypotheses, the grade-safe MVP, and the gated stretch goals.
  - Timeline: use the deep-dive hour budget.
  - Disclose AI use.

### Spike results (2026-09-29, BC Andromeda HPC)
Per-spike details, commands and timings are in `spike_results/A1…A4*.md`. Code, envs, data and outputs are in `~/vis4ml_spikes/`, a symlink to `/projects/weilab/zhangdjr/vis4ml_spikes`. All compute ran in SLURM jobs: `gtml` L40S 46 GB, `weilab` A10 23 GB, and `short` CPU.

| Spike | Verdict | Key numbers |
|---|---|---|
| **A1 D5 microscopy** | **PASS** | Cellpose-SAM and micro-SAM (`vit_b_lm`) on 8 LIVECell test images (one per cell type, 2,006 GT cells). **0.13–1.6 s per image**, ≤4.4 GB GPU. Pooled F1@0.5: Cellpose-SAM 0.86, micro-SAM AIS 0.78. Cellpose mostly *misses* cells; micro-SAM AIS mostly *merges* them (407 GT cells in merges). **Error κ between the two = 0.52**, and 80% of Cellpose-SAM errors are shared. 113 "silent failures" (both wrong, agreeing at IoU ≥ 0.5). Agreement vs per-cell IoU: ρ = 0.62–0.76 (0.51–0.62 within image), comparable to Cellpose flow error (ρ = −0.63). |
| A2 D2 TSFMs | **PASS, with correction** | **The previous pilot's row-order assumption was wrong for 34/50 datasets (88% of series-variates).** The correct join is by name; it was verified by content (100%) and reproduces TIME Tables 4–5 **within 0.1%** (Chronos-2 seasonal = 1 / 0: 0.5653 / 0.6544 vs 0.565 / 0.654). Within-series variance share: **0.66 (MASE) but 0.45 (CRPS)**. Chronos-2 vs TimesFM-2.5 windows differing by >25%: 31.8% (MASE) / 27.7% (CRPS). The old feature-ρ table is void: with the correct join, seasonal_strength ρ = −0.21 and x_entropy +0.21, and length is no longer the top feature. |
| A3 D3 LLM judges | Step 1 **PASS**; step 2 **PASS** | MT-Bench agreement reproduced **exactly** (turn 1): GPT-4 vs human 66.0 / 84.6%, human–human 63.0 / 81.0% (all within 1 pp). Throughput: on the lab server `cscigpu08`, `qwen3.6:27b` generates about 25 tok/s (about 30 h for 3,844 calls, one judge); `gpt-oss:20b` is not installed there. **On one Andromeda L40S with user-space Ollama 0.34.4, both judges are resident (32 / 46 GB). Qwen generates about 70 tok/s but serially (Ollama has no parallel support for its hybrid-SSM architecture); gpt-oss manages 113 tok/s at 4 parallel. The minimum run takes about 4.7 h.** |
| A4 D4 astronomy | **PASS** | Zoobot 2.9.0 / ConvNeXt-nano encoder + Dirichlet-multinomial head, GZ DESI `tiny`, "smooth-or-featured" (dr5): test NLL 4.32 → 2.73, vote-fraction MAE 0.36 → 0.16 in 2 epochs, **3 s per epoch** on A10. A GZ3D bar/spiral mask aligns with its Legacy Survey DR10 cutout to 1 px (0.26″); centre mask within 0.39″. |

What this changes (SYNTHESIS):
- **D5 stays #1.** Both models install in one env and run in under 2 s per image, and the core measurement works. The early signal is interesting: substantial shared errors between two SAM-lineage models, with agreement about as informative as internal scores.
  - Still missing for RQ2: a **non-SAM third model**, Cellpose3 `cyto3` in a separate env. **CellSAM now runs** (DeepCell key supplied), but it is SAM-based too: a SAM encoder plus a DETR box prompter. Zero-shot on LIVECell it is bimodal by cell type: F1 0.92 on BV2 and SkBr3, 0.02–0.31 on the other six at default settings. Its κ with Cellpose-SAM is 0.19. Pair κ must be read against each pair's accuracy gap (see `spike_results/A1_microscopy_seg.md`).
  - micro-SAM AMG's default thresholds return **zero masks** on LIVECell, so its predicted-IoU head is poorly calibrated. Use AIS, and report AMG with tuned thresholds only on non-test data.
- **D2 stays #2, but two claims must change.**
  - (i) Every per-variate feature analysis must join by `(dataset_id, series_name, variate_name)`, as the TIME leaderboard does.
  - (ii) Pre-register H1 per metric. H1 (>50% of variance within series) holds for MASE but not for CRPS (0.45), though it clears the 0.3 falsification line for both.
  - The "same-family noise floor" (Chronos-2 vs Chronos-Bolt) is not a noise floor; use seed or sample reruns. Together with the Wang et al. partial overlap (§11), D2's novelty margin is thinner.
- **D4 is technically easy.** The pipeline and alignment both work. The bar mask covers pixels 30× brighter than the cutout's mean, so the brightness-baseline requirement is real. Use Legacy Survey cutouts as model input, because GZ3D images have the MaNGA hexagon drawn in.
- **Disk:** `~/vis4ml_spikes` is **49 GB**, far over the ~10 GB budget. It sits on `/projects/weilab` (12 TB free); `/home` is unchanged at 15 GB free. Breakdown:
  - about 13 GB of conda envs plus their hardlinked package cache. d5 alone is about 12 GB (CUDA libraries and napari); Zoobot and CellSAM were added to the same env to avoid more torch installs;
  - **32 GB of user-space Ollama plus qwen3.6:27b and gpt-oss:20b**;
  - 1.7 GB of CellSAM weights, 1.7 GB of other weights, and 0.44 GB of data.
  - Easy cuts: `ollama/models` (30 GB) if D3 is dropped; `envs/d5` if D5 is dropped.


### Spike round 2 (2026-09-29, ambitious versions of D5 and D4; details in `spike_results/B1_d5_hierarchy.md` and `B2_d4_ambiguity.md`)

| Spike | Verdict | Key numbers |
|---|---|---|
| **B1 D5 error hierarchy** | **PASS (feasible); first signal contradicts H-mono** | 3 Cellpose-SAM fine-tune seeds (30 min each, one L40S each; seeds vary data order + augmentation, after fixing cellpose's per-epoch `np.random.seed`). cyto3 and livecell_cp3 via a 2 MB cellpose-3 overlay (livecell_cp3 is still downloadable). 8 models on 40 LIVECell test images (10,292 cells). **Per-cell κ [95% image-bootstrap CI]:** seed–seed **0.92** [0.90, 0.93] > cpsam vs its fine-tune 0.88 > cyto3 vs livecell_cp3 0.84 > **cpsam vs cyto3 (SAM vs non-SAM, same Cellpose lineage) 0.79** > micro-SAM vs cyto3 0.61 ≈ **cpsam vs micro-SAM (both SAM) 0.58** [0.52, 0.62]. The ordering holds after κ/κ_max. Agreement-QC AUROC rises as κ falls (0.71 for seeds → 0.75–0.85 for cross-lineage pairs); silent failures fall from 166 to 67 per 1,000 cells. **In-distribution caveat:** every model except CellSAM saw LIVECell train. |
| **B2 D4 ambiguity vs explanations** | **PASS (feasible), with a RED FLAG** | 100 GZ3D galaxies pre-matched (MaNGA drpall × Zenodo GZ DECaLS/DESI vote catalogues, 66 MB; no 17.5 GB download). Legacy Survey DR10 cutouts on the GZ3D field; masks WCS-aligned, orientation verified 100/100. Zoobot bar head, 3 seeds on `tiny` (13–25 s each; ρ(model, volunteer bar fraction) = 0.68). IG / SmoothGrad / Grad-CAM / occlusion in 3 min. **Brightness alone localizes the volunteer bar mask with median AUPRC 0.83; attributions reach 0.29–0.44 and beat brightness on only 10–17% of galaxies.** Over a light-profile model they add +0.005 to +0.02 AUPRC. Human vote entropy correlates *positively* with cross-method agreement (ρ +0.24, as in Jukić), but this vanishes after controlling for model uncertainty. The one surviving hint: partial ρ(H_h, ΔAUPRC) = −0.27 [−0.47, −0.04], n = 79, uncorrected over 13 tests. |

What this changes (SYNTHESIS):
- **D5 ambitious stays the lead, with a sharper hook.** The "shared SAM encoder → shared errors" story (H-mono) is *not* what the data show. **Shared lineage (objective, recipe, training data) predicts shared errors; the backbone does not.** That is a clean, reportable deviation, and it answers the Bankhead/BISCUIT question in an unexpected direction.
  - The proposal should pre-register **both** H-mono and H-lineage.
  - A held-out dataset (NeurIPS22 Public-Test, range-readable, so no 2.9 GB download) is the must-have next step.
  - Estimate: about 18–20 h.
- **D4 ambitious is feasible but riskier than thought.** The measurable effect sits *on top of* a dominant brightness signal.
  - The primary outcome must be the gain over a light-profile model.
  - The first signal is one uncorrected hint, and the direction of the agreement effect is opposite to the naive hypothesis (as Jukić warned).
  - Estimate: about 21–24 h. The 17.5 GB download is needed only for a stronger model, not for the question.
- **Recommended pitch for Oct 6:** lead with **D5 ambitious** (H-mono vs H-lineage); offer **D4 RQ-B** as the higher-risk alternative, with the brightness-dominance result stated up front.

### Spike round 3 (2026-09-30, D5 pre-registered kill tests; details in `spike_results/C1–C5`, pre-registration in `PREREG_D5.md` + Amendments 1–4)

| Task | Verdict | Key numbers |
|---|---|---|
| **C1 held-out replication** (NeurIPS22 Public-Test, 50 images, 6,040 cells; range-read 253 MB of `Testing.zip`) | **H1 ✓, H2 ✓, H3 ✓, H4 inconclusive; K1 passed** | Accuracy: Cellpose-SAM 0.95, cyto3 0.89, micro-SAM 0.85, CellSAM 0.72, livecell_cp3 0.47. **κ(cpsam, cyto3) 0.33 vs κ(cpsam, micro-SAM) 0.17: diff +0.16 [0.09, 0.25]; κ/κ_max +0.19 [0.06, 0.32]**; Holm p 0.004. Seeds 0.91–0.93. ρ(κ, QC AUROC) −0.30 [−0.41, −0.14], but it **reverses (+0.23)** when the target is the pair's stronger model. H4: agreement AUROC 0.898 vs Cellpose flow error (`flow_threshold=0`) 0.885, Δ +0.01 [−0.04, 0.07]. On LC200 (194 images; secondary) H4 is **falsified** (0.74 vs 0.84). |
| **C2 robustness** | K3 ✓, K5 ✓ (heterogeneous), **K4 triggered, K6 triggered** | GT-median diameter doesn't remove the effect: κ 0.35. **Accuracy-matched subset (20 images): H1 diff −0.03 [−0.13, 0.21]**. On LC200 the κ/κ_max gap is only +0.02. The raw effect sits in the modalities where micro-SAM is weak (phase-contrast cultured cells, bacteria), and is ≈ 0 in brightfield. Top-5% triage: a same-family reference (cyto3) finds **5.7 pp more** Cellpose-SAM errors than a cross-family one (micro-SAM). |
| **C3 difficulty null (K2)** | **passed** | H1 diff after decile stratification +0.17 [0.09, 0.25]; by image +0.14; image × tercile +0.17; image × CellSAM error (exploratory) +0.10 [0.02, 0.19]. The permutation-null κ is about 0.02. The attribute error model is weak on Public-Test (AUROC 0.50–0.57). |
| **C4 mechanism** | H5 partly; H6 supported (small) | Merge/split κ: cpsam–cyto3 0.25 > cpsam–micro-SAM 0.09, but micro-SAM AIS vs AMG (same weights) 0.35. From-scratch U-Nets (5 × 17–39 min): seeds 0.86 vs disjoint data 0.825 (+0.035 [0.030, 0.041]). Out of distribution, the LIVECell-only models fail *together* (κ 0.73–0.84). |
| **C5 pitch figure** | done | `spike_results/fig_kappa_by_level.png`, `fig_kappa_vs_auroc.png` |

What this changes (SYNTHESIS):
- **D5 is still the lead, and still safe as a course project.** The floor results replicate on held-out data: the seed hierarchy, the error taxonomy, and agreement-QC ≫ attribute baselines.
- **But "family, not encoder" should not be the headline.** It passes its pre-registered test, but it is accuracy-confounded (K4), modality-dependent, and mislabelled: Cellpose-SAM uses ViT-L while micro-SAM uses ViT-B. A stronger, better-supported headline: **"error consistency is high in-distribution and collapses out of distribution. Shared training distribution, not a shared foundation backbone, predicts shared failures there. And agreement-based QC is only as good as the model's own flow-error signal."**
- **Newly opened question:** the only same-encoder cross-family pair (micro-SAM vs CellSAM, both ViT-B) is the most consistent cross-family pair on Public-Test (κ 0.39). A narrow H-mono may hold; test micro-SAM `vit_b_lm` vs `vit_l_lm`.

### Spike round 4 (2026-09-30, "what makes a good reference for agreement-based QC?"; details in `spike_results/D0–D6`, pre-registration in `PREREG_D5.md` Part B + Amendments 5–7)

| Task | Verdict | Key numbers |
|---|---|---|
| **D0 N1 choice** | done; frozen before any model run | **N1 = mCellSeg** (Zenodo 20174259, May 2026, CC BY 4.0): 198 DIC/BF images, 15,975 whole cells, HEK-293T + HUVEC; in no model's training list. N2 = 200 fresh LIVECell test images; N3 = 3 leave-one-type-out folds (SH-SY5Y, BV2, SKOV3) |
| **D1 re-analysis (round-3 data)** | exploratory | ORs confirmed: PT 16.7 vs 5.6, LC200 91.6 vs 22.7. H1 under log-OR +1.10 (PT), +1.40 (LC200), but null on PT when accuracy-matched or R8-stratified. R1 dry run mixed (PT +0.22, LC200 −0.02, N2 +0.05). AUROC version supported everywhere. recall@5% has a 0.05/e_t ceiling (Amendment 6) |
| **D2 new models** | done | Round-3 "Cellpose-SAM" was **v2** (cellpose 4.2.1.1 default); v1 added. µSAM `vit_l_lm` added. CellposeDINO-L/B (DINOv3 backbone, exploratory) via a separate overlay. Encoder table from the installed code: Cellpose-SAM v1/v2 = SAM ViT-L (patch 8); µSAM ViT-B/L; CellSAM = SAM ViT-B; cyto3 = CNN |
| **D3 confirmatory on N1** | **nothing supported** | Native scale: every Cellpose model, cyto3 and CellSAM fail the > 0.6 error rule (Cellpose-SAM error 0.652; cells 50–327 px), so **R1–R3 untestable**. Rescaled (Amendment 7, outcome-blind): **R1 inconclusive (+0.05 [−0.02, 0.17]); R2 falsified (−0.06 [−0.28, 0.15])**, also null at R7's matched operating point and under R8; R3 untestable (µSAM ViT-L error 0.604). R5 not supported (≈ flow error); R6 supported (+0.03–0.05 AUROC) |
| **D4 controlled shift (R4)** | **falsified** | 18 own models (3 seeds × U-Net/µSAM-from-vanilla-SAM × 3 folds). Cross − seed Δ = **−0.57 [−0.72, −0.43]**, negative in every fold: shift makes *seeds* less alike. BV2 shows κ halving while log-OR rises (margin artefact) |
| **D5 figures** | done | `fig_kappa_by_level_v2.png` (κ + log-OR, 5 groups); `fig_reference_tradeoff.png` (PT, N2, N1-rescaled) |

What this changes (SYNTHESIS):
- **"What makes a good reference" does not survive as a confirmed headline.** Its robust part is secondary: {f, o} predicts *ranking* quality (AUROC) far better than κ on all four datasets. For budgeted triage, reference choice saturates when the target's error rate exceeds the budget.
- **Better-supported headline, from the nulls plus a consistent exploratory pattern:** shared failures come from **shared training data and recipe, not a shared foundation backbone.**
  - Cellpose recipe with DINOv3 vs with SAM: log-OR 4.95 (N2) and 4.55 (N1), close to v1↔v2 at 5.9/5.0.
  - Different recipes: 2.6–3.2.
  - The family effect is null on N1, there is no shared-checkpoint effect, and shift does not separate architectures.
- **Lesson:** held-out sets need a GT-only scale check in their inclusion criteria. Pre-register the recipe-vs-backbone contrast on a new, scale-checked set for the proposal.

---

## 13. Appendix

### 13.1 Corrections made to sub-agent claims (FACT)
- **PassAI's authors** are Takamido, Ota and Nakamoto (IEEE Access 2025), not "Tsutsui et al.".
- **Priscylla Silva** *is* instructor-lab (first author of Visagreement with C. Silva and Nonato). One scan called her "unrelated".
- **Time-series VA does overlap with the lab:** mTSeer (CHI 2021, C. Silva, Bertini). The broad scan had said "no overlap".
- **P. Silva's AAAI AI4Ed 2024 paper exists** (arXiv 2405.13957); one deep dive could not find it.
- **My earlier "Moderate–High" upside for Soccer C1 was generous.** It is on par with D2.
- **My earlier top-2 (D2, D3) is superseded** by §10, now that D4 and D5 have been reviewed.

### 13.2 Visagreement full-text check (2026-09-29; the user supplied the full text)
The earlier rounds read Visagreement only through the author's IJCAI-DC summary. The full text confirms these FACTs:
- **Scope:** tabular data, **binary classification only**, local feature-importance methods (10 Captum methods) on PyTorch MLPs. Datasets: COMPAS, Adult, German Credit, HELOC, Diabetes, plus 2 synthetic.
- **Metrics:** Krishna et al.'s FA/SA/RA/SRA metrics map each instance into a "(dis)agreement space". LAMP projects that space with the corners as control points. LAMP is a DR method that places chosen control points first and positions the other points relative to them.
- **Evaluation:** 3 case studies plus a 4-expert think-aloud evaluation.
- **Case Study 1 (quality):** "no definitive correlation" between disagreement and quality, measured **only** by sensitivity and infidelity. Some method sets (Input×Gradient, IG, Shapley Value Sampling) agree more when quality is good.
- **Case Study 2 (accuracy):** with the RA/SRA metrics, instances in the disagreement area have worse balanced accuracy and F1. The agreement area is mostly label 1 and the disagreement area mostly label 0. The authors call it a conjecture that is "not comprehensive enough to assert".
- **Case Study 3 (features):** no feature globally drives disagreement. Feature "switching" may reflect the off-manifold problem.
- **Stated limitations and future work:** binary only; **"not appropriate for handling disagreements in image and text data … a challenge we intend to address in the near future"**; performance on larger data; about 4 methods is the practical maximum.

Implications (SYNTHESIS):
- (a) D5, D3 and Soccer C1 can each be pitched as a rigorous, GT-based test of Visagreement's Case Study 1/2 conjectures.
- (b) The planned image/text extension is an **in-lab scoop risk** for D4 RQ-B and D3. **Ask Silva on Oct 6** whether it is in progress.
- (c) Earlier wording saying the authors "had no GT" was my inference. It is corrected to "quality was measured only by proxy metrics".

### 13.3 The other research system's report (`deep-research-report.md`)
Compared in round 1. In the claims I checked, about 5 of its ~12 table entries were materially wrong:
- **PassAI:** misattributed and misdescribed.
- **Decroos 2019:** described as VAE tracking embeddings; it is VAEP on event data.
- **TacticAI:** it is Nature Comms 2024 and uses t-SNE, not "AAAI 2023" with UMAP.
- **Off-ball defensive roles:** an HMM on tracking data (arXiv 2026), not a NeurIPS 2023 CNN on video.
- **Bauer 2023:** uses tracking data, not event data.

It also missed MOUNTAINEER/Visagreement, Ichmoukhamedov 2024 and Tang 2023, which undercut its top-ranked gaps.

It does agree with this review that soccer XAI rarely evaluates its explanations. It also usefully pointed to SkillCorner's open phase-of-play labels.

### 13.4 AI-use disclosure (for the course)
- **Who:** produced by Claude (Opus 5.5) with 19 sub-agents: 6 Opus in round 1; 5 Sonnet scans + 3 Opus deep dives in round 2; 5 Sonnet scans + 2 Opus deep dives in round 3.
- **Verification:** the lead agent verified the load-bearing claims against primary sources (§0.2).
- **Your responsibility:** the research design choices and final judgments are yours to make and defend.
