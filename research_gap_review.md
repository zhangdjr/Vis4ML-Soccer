# Research-Gap Review: Soccer, Open Topics, and Science Domains

**Project:** solo final project, NYU DS-GA 3001 *Visualization for Machine Learning* (Prof. Claudio Silva), Fall 2026
**Compiled:** 2026-09-29 by Claude (Opus 5.5), lead reviewer. The review took three rounds and 19 sub-agents (list in §0.2). The lead reviewer checked every claim the conclusions depend on.
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
10. [Ranking (requested)](#10-ranking-requested)
11. [What you should verify yourself](#11-what-you-should-verify-yourself)
12. [Next steps before Oct 20](#12-next-steps-before-oct-20)
13. [Appendix: corrections, the other system's report, AI-use disclosure](#13-appendix)

---

## 0. Scope, method, limits

### 0.1 Constraints used throughout
- **Solo**, with about **40 h total** (3–4 h/week). The **4-page proposal is due Oct 20**; the 1-page update Nov 3; the **8-page report Dec 14**; presentations Dec 1 and 8.
- The course requires you to **reproduce prior work and extend it**, and to demo both.
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
- **Pilot numbers and compute estimates are unmeasured on your machines:**
  - D2's variance-share pilot assumes an unverified row ordering.
  - D3's Ollama throughput is estimated from public benchmarks.

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

**My current ranking** (you asked; details in §10):
1. **D5, ground-truth-free QC for generalist cell segmentation.**
2. **D2, window-level failure regimes of time-series foundation models.**

The **interest-driven alternative is D4** (Zoobot / Galaxy Zoo, astronomy).

---

## 2. Course context

FACT, from ctsilva.github.io/2026-VisML-CDS, its syllabus and the default-project page:

| Item | Detail |
|---|---|
| Weight | Project = 45% of the grade: proposal 10%, update 10%, final 25% |
| Type | "Reproduce prior work or implement a proposed research idea"; demo both |
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
| CV eval | Rashomon/multiplicity for segmentation | higher-upside (§6) | No single reproduction target |
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
- **Reproduce.** The NeurIPS22 CellSeg per-modality F1 table for 1–2 models on its Public-Test set, and a Cellpose-SAM benchmark number on LIVECell test.
- **Minimum viable project (grade-safe).** Per-instance error taxonomy (merge / split / miss / false positive), attribute small multiples and an error-consistency matrix on LIVECell. This is a strict subset, done by Nov 3.
- **Stronger version.** RQ1–RQ3 with within-image AUROC, Zenk's risk-coverage metric (AURC), and a **triage view evaluated by simulated inspection**: errors found per K cells inspected, against GT. No user study is needed.
- **Data (FACT, with the leakage map checked).**
  - **NeurIPS22 Public-Test** (50 labelled images): held out from all models.
  - NeurIPS22 Tuning (101 images): possibly micro-SAM validation, unconfirmed.
  - **LIVECell test**: Cellpose-SAM and micro-SAM trained on LIVECell *train*, and CellSAM never saw LIVECell.
  - BBBC038 is in every model's training data, so it is **unusable** for held-out claims.
  - Licenses: LIVECell CC BY-NC 4.0; NeurIPS22 CC-BY-NC-ND (fine for research).
- **Models and compute.** Repos are maintained and permissively licensed.
  - CellSAM weights need a free DeepCell API key.
  - Cellpose 4 cannot load cyto3, so cyto3 needs its own environment.
  - StarDist is a poor fit for phase-contrast whole cells.
  - Cellpose-SAM inference has been profiled on a 12 GB card; an M1 path exists.
- **Visualization.** Error map overlaid on images; attribute small multiples; model × model error-consistency matrix; a triage list ranked by each QC signal, linked to image crops.
  - *Why:* the claim is about **which instances** fail and **whether failures co-occur**. Aggregate F1 hides exactly that.
- **Evaluation.**
  - Error-consistency κ with confidence intervals (CIs), same-lineage vs. cross-lineage pairs.
  - Within-image AUROC and AURC for each QC signal vs. the attribute-only baseline.
  - Simulated-inspection curves.
  - Pre-registered falsification criteria.
- **Risk.** Agreement may just encode image difficulty. The within-image AUROC and the attribute baseline are designed to catch this, and a negative result is still reportable. Other risks: 3 environments, and instance-matching engineering.
- **Course fit.** Model assessment, black-box signals, DL visualization; a Visagreement-like "disagreement as evidence" framing without overlap.
- **Publication path.** Add 2–3 more datasets and a small user study with biologists. Venues: a BioImage/CVPR CVMI workshop, MICCAI workshops, or a VIS short paper. Its answer to a named reviewer question is the hook.
- **12b.** Course fit **High** · upside **Medium** (answers a named question) · technical risk **Medium** (your background lowers it) · data risk **Low–Med** · viz burden **Low–Med** · evaluation clarity **High** (GT exists).

### 5.2 D2: Window-level failure regimes of time-series foundation models
Full notes: `lit_notes_open/deep_2_tsfm_va.md`.

- **Research question.** Where do zero-shot TSFMs (Chronos-2, TimesFM-2.5, Moirai-2, TiRex) fail at the **forecast-window** level, relative to strong classical baselines (AutoETS/ARIMA/Theta, which TIME lacks) and to each other? Are failures predicted better by context-window features than by TIME's static patterns, under leave-dataset-out validation? Do Jander et al.'s synthetic failure modes (persistence overestimation, regime-switch collapse) show up "in vivo"?
- **Evidence for the gap.**
  - TIME (ICML 2026) stratifies statically by 7 features. EXPRTS and Kang 2017 cover pre-TSFM instance spaces. mTSeer (lab) asks for "more features… more models… expanded instance-level evaluation".
  - Jander et al. (Aug 2026) request "in vivo validation".
  - Pilot (SYNTHESIS, provisional): about 66% of error variance is within a single series across windows. Chronos-2 and TimesFM-2.5 differ by more than 25% on about 32% of windows.
- **Reproduce.** TIME's pattern-level results from released outputs (pandas, about 3 h), plus a local Chronos-2 rerun on 2–3 tasks.
- **MVP.** Classical baselines, context features, failure labels, variance decomposition, a leave-dataset-out failure predictor, and 5 linked views. **No TSFM inference needed:** TIME-Output releases per-window predictions for 30 models under Apache-2.0.
- **Stronger version.** The in-vivo dose-response test; a context-length what-if probe; GIFT-Eval replication.
- **Viz.** Failure-regime map, conditional-error small multiples, leaderboard-flip matrix, window drilldown, held-out slice table.
- **Evaluation.** Variance shares (with CRPS and a noise floor); ΔAUROC ≥ 0.03; held-out slice lift ≥ 1.5×; dose-response CIs.
- **Risk.** Features may explain little of window-level failure. The pilot shows |ρ| ≤ 0.26, and a July 2026 impossibility result argues against spectral features. A null result is still reportable. The field moves weekly.
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
- **Risk.** **Brightness confound.** Bars are central and bright, so attributions must beat a light-profile baseline. There is also scoop risk from the Walmsley/Masters/Spindler group (who own the data), and a ZooBot:3D segmentation model now exists. RQ-A alone is incremental.
- **12b.** Fit **High** · upside **Medium** (RQ-B → ML4PS / VIS short) · technical **Medium** · data **Low** · viz **Low–Med** · evaluation **High** (RQ-A) / Med-High (RQ-B).

### 5.4 D3: Decomposing LLM-judge disagreement (LLM; education option)
Full notes: `lit_notes_open/deep_3_llm_judges.md`.

- **RQ.** With local judges (gpt-oss-20b, qwen3.8:27b) plus the released GPT-4 judgments on MT-Bench items that have ≥2 human votes: split item-level judge–human disagreement into **human ambiguity, judge instability, feature-linked bias, and rationale divergence**, and recover planted biases that pooled κ hides.
- **Gap.** LLM Comparator (TVCG 2025, archived) is single-judge. The 2026 statistical papers (Coin Flip, Geometry EMNLP 2026, Nine Judges) each cover one layer. VIS 2025/26 and about 55 citers of LLM Comparator show no multi-judge VA (FACT).
- **Reproduce.** Zheng et al. Table 5 **exactly**: 66/85/63/81%, from CC-BY-4.0 data (1.4 MB, 961 cells with ≥2 votes).
- **Evaluation.** Planted P1–P3 must be flagged in the top 3 with BH q < 0.05; a null control must produce no flags.
- **Compute.** About 13 GPU-h (MVP) to 36–46 GPU-h (full), unattended; unmeasured.
- **Education stretch.** ASAP 7/8 or PERSUADE 2.0, including a DIF (differential item functioning) slice.
- **Risk.** "Not novel", since it is structurally Visagreement-for-judges, and scoop risk from PAIR, KAIST and IBM.
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

- **C1: physics-anchored evaluation of explanation disagreement for xG.** Does cross-method or cross-model disagreement flag explanations that violate shot geometry (mirror invariance, iso-distance arcs, monotonicity)? It extends Visagreement's open question: in its case study, disagreement did not correlate with explanation-quality metrics, and the authors had no GT to check. Data: StatsBomb open. Risk: every method may simply agree on distance/angle. 12b: fit Excellent · upside Moderate (my earlier "Moderate–High" was generous; it is on par with D2) · technical Low–Mod · data Low · viz Moderate · evaluation Med–High.
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
- **Constraints cap the ceiling.** High upside usually needs a new method, a user study, big data, or being first in a fast area. All of those conflict with a solo 40 h, open-data, reproduce-first project.
- **The open gaps are narrow.** They are integration or measurement gaps.
- **I rated upside as expected value, including null-result risk.**

**Upside depends on results more than on the topic.** These are the versions that could reach "High" (a VIS full paper or a strong workshop paper):

| Higher-upside option | Built on | What would make it High | Extra risk vs. its grade-safe version | Grade-safety if chosen as the *main* project |
|---|---|---|---|---|
| **D4 RQ-B: human ambiguity → explanation unreliability (GZ3D masks)** | D4 | A clean result that attributions track masks beyond a brightness baseline, and fail more where volunteers disagree. This offers a general lesson for XAI evaluation with soft labels | Brightness confound; mask alignment; scoop | Medium (keep RQ-A as a fallback) |
| **D5 full: correlated-error audit + GT-free triage** | D5 | Shows BISCUIT-style QC fails for same-lineage models, and a QC signal beats attribute baselines within images | Agreement may be trivial; 3 environments | Medium-High (the taxonomy MVP is a floor) |
| **D2-max: in-vivo test as the headline** | D2 | Synthetic TSFM failure modes confirmed on real benchmarks, with held-out regimes | A null result is likely-ish | Medium-High (MVP needs no inference) |
| **D1-max: probabilistic gap triage + routability repair** | D1 | Calibrated bottleneck confidence repairs networks and improves PathwayBench traversability | Plumbing, GPU install | Medium |
| **Rashomon / multiplicity VA for segmentation** | scan_4 | The first VA of seed-level disagreement for segmentation, tied to annotation ambiguity | No single reproduction target (course rule); multi-seed training | Low-Medium |
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
| D2 | TIME (ICML 2026); mTSeer (CHI 2021, lab); EXPRTS (arXiv); Kang et al. (IJF 2017) | Prior art and reproduction target | F |
| D2 | Jander et al., Causal analysis for TSFMs (arXiv 2026); *The Spectrum Is Not Enough* (arXiv 2026) | Named open question; main risk | F/A |
| D4 | Walmsley et al., GZ DECaLS (MNRAS 2022); GZ DESI (2023); Zoobot (JOSS 2023); Walmsley 2020 | Reproduction targets; what's already done | F |
| D4 | Masters et al., GZ3D (2021); Bhambra et al. (2022) | Masks; closest attribution work | F |
| D4 | Baan et al. 2022 (calibration to human disagreement); Singh et al. 2025/26 | General-ML baseline; open small-N problem | F |
| D3 | LLM Comparator (TVCG 2025); Zheng et al. MT-Bench (NeurIPS 2023); EvalGen (UIST 2024) | Targets and prior VA | F |
| D3 | Coin Flip Judge (arXiv 2026, single author); Geometry of LLM-as-Judge (EMNLP 2026); Nine Judges (arXiv 2026); Sunkavalli essay audit (arXiv 2026) | What's already known | F |
| D1 | Tile2Net (CEUS 2023); Calibrate (TVCG 2023); PathwayBench (arXiv 2024); Prophet (arXiv 2024); Gupta et al. (NeurIPS 2023) | Targets and neighbours | F |
| Soccer C1 | Visagreement (TVCG 2025); MOUNTAINEER (TVCG 2024); Krishna et al. disagreement (TMLR 2024); Tsai et al. (arXiv 2026); PassAI (IEEE Access 2025) | Prior art; the 0/25 local-agreement finding | F/A |
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
| **D5 Cell-seg QC** | BISCUIT assumption untested; reviewer question unanswered; Zenk is image-level, radiology only | BISCUIT, SEG, MARC, Zenk, NeurIPS22 | LIVECell (NC), NeurIPS22 (NC-ND); repos open; CellSAM key | NeurIPS22 per-modality F1; Cellpose-SAM LIVECell number | Error taxonomy + consistency matrix (MVP) → within-image QC AUROC + simulated triage | **High** | High | Good (~40 h) | Agreement = difficulty only | Nonato/Miranda/Bertini dblp; BISCUIT/MARC citers; micro-SAM validation set |
| **D2 TSFM regimes** | TIME is static; pilot variance share; Jander requests in-vivo | TIME, mTSeer, EXPRTS | TIME-Output Apache-2.0 | TIME pattern tables; Chronos-2 rerun | Classical baselines + context features + leave-dataset-out predictor | **High** | High | **Very good** | Features explain little | Row order; CRPS pilot; weekly scoop check |
| **D4 Zoobot** | Aggregate noise floor only; no GZ3D attribution study | GZ DESI/DECaLS, Bhambra 2022, Baan 2022 | GZ DESI CC-BY-NC-SA; GZ3D DR17; Zoobot GPL | GZ DESI Fig. 5; DECaLS Fig. 19 | Small-N decomposed calibration (+ GZ3D attribution) | High / Med-High | High | Good | Brightness confound; scoop | ZooBot:3D paper; Walmsley group 2026 output |
| **D3 LLM judges** | No multi-judge × human VA | LLM Comparator, Visagreement, Coin Flip | MT-Bench CC-BY-4.0 | Zheng Table 5 exact | Decomposition views + planted biases + null | **High** | High | Good | "Not novel" / scoop | Throughput test; CHI/IUI 2026 sweep |
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

## 10. Ranking (requested)

This ranking weights your stated order: **grade first, then publication**, with 3–4 h/week. Your interests break ties. It supersedes my earlier top-2 (D2, D3), because D5 and D4 were not reviewed yet at that point.

1. **D5: cross-model agreement as ground-truth-free cell-segmentation QC.**
   - Every result is checked against GT, so evaluation is objective.
   - The motivation is a **specific, citable, unanswered reviewer question**.
   - No instructor overlap was found, and it uses your strongest skill.
   - It has a clean floor: an error taxonomy plus a consistency matrix by Nov 3.
   - Risks: three model environments, and a possibly trivial "agreement = difficulty" result, which the design is built to detect.
2. **D2: window-level failure regimes of time-series foundation models.**
   - The **lowest execution risk** of anything reviewed: released per-window outputs mean no inference for the MVP, and it runs on a laptop.
   - The criteria are pre-registered and falsifiable, and it has a named open question plus a lab hook (mTSeer).
   - Weakness: none of your interest domains, and a real chance the result is null.

**If you'd rather follow interest:**
- **D4 (astronomy)** is the pick. Its higher-upside half (GZ3D attribution vs. ambiguity) has the most distinctive story of all the candidates.
- **D1** is the pick if maps and instructor buy-in matter most.

**What would change this:**
- A failed week-1 spike: D5's three environments won't install, or D2's row ordering is wrong and features must be recomputed. Swap in the next candidate.
- Silva's feedback on Oct 6. For example, a strong preference for the default project moves D1 up.

---

## 11. What you should verify yourself

These are the checks most likely to change the conclusions.

1. **dblp for Silva, Nonato, Miranda, Barr and Bertini, 2023–2026.** It was blocked for every agent. Look for microscopy, astronomy, forecasting and LLM-evaluation work.
2. **Prior-cohort projects.** Ask the TA for last year's project list, and search GitHub for repos created in Dec 2025.
3. **D5:**
   - (a) read BISCUIT's open reviews yourself;
   - (b) check whether BISCUIT or MARC have citers that test the "uncorrelated errors" assumption;
   - (c) confirm whether NeurIPS22 Tuning was micro-SAM's validation set;
   - (d) install Cellpose-SAM and micro-SAM, and run them on 5 LIVECell test images.
4. **D2:**
   - (a) verify the TIME row ordering in `zqiao11/TIME`;
   - (b) re-run the pilot with CRPS;
   - (c) run a weekly arXiv scoop check on TIME and Jander citers.
5. **D4:**
   - (a) check the ZooBot:3D (2026) paper for any attribution-vs-mask analysis;
   - (b) review Walmsley group 2025–26 output, including the SAE-on-Zoobot work;
   - (c) download one GZ3D FITS file and one Legacy Survey cutout, and check their alignment.
6. **D3:** recompute the MT-Bench 66/85/63/81% numbers (about 1 h), and time one Ollama judge call.
7. **D1:** ask which NYC boroughs are held out, and run Tile2Net's Boston example on the GPU box.
8. **Soccer C1:** read Cefis & Carpita 2024 (paywalled; NYU library).
9. **Licenses you would publish under:**
   - GZ DESI (NC-SA, plus its code-release clause);
   - LIVECell (NC) and NeurIPS22 (NC-ND);
   - TIME data (NC);
   - StatsBomb (credit + logo).
10. ~~The JS/D3 expectation~~: resolved. Silva says it is not required (§2).
11. **Whether extending a lab tool is welcomed** (Calibrate, Visagreement, mTSeer). Ask at the Oct 6 discussion.

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

---

## 13. Appendix

### 13.1 Corrections made to sub-agent claims (FACT)
- **PassAI's authors** are Takamido, Ota and Nakamoto (IEEE Access 2025), not "Tsutsui et al.".
- **Priscylla Silva** *is* instructor-lab (first author of Visagreement with C. Silva and Nonato). One scan called her "unrelated".
- **Time-series VA does overlap with the lab:** mTSeer (CHI 2021, C. Silva, Bertini). The broad scan had said "no overlap".
- **P. Silva's AAAI AI4Ed 2024 paper exists** (arXiv 2405.13957); one deep dive could not find it.
- **My earlier "Moderate–High" upside for Soccer C1 was generous.** It is on par with D2.
- **My earlier top-2 (D2, D3) is superseded** by §10, now that D4 and D5 have been reviewed.

### 13.2 The other research system's report (`deep-research-report.md`)
Compared in round 1. In the claims I checked, about 5 of its ~12 table entries were materially wrong:
- **PassAI:** misattributed and misdescribed.
- **Decroos 2019:** described as VAE tracking embeddings; it is VAEP on event data.
- **TacticAI:** it is Nature Comms 2024 and uses t-SNE, not "AAAI 2023" with UMAP.
- **Off-ball defensive roles:** an HMM on tracking data (arXiv 2026), not a NeurIPS 2023 CNN on video.
- **Bauer 2023:** uses tracking data, not event data.

It also missed MOUNTAINEER/Visagreement, Ichmoukhamedov 2024 and Tang 2023, which undercut its top-ranked gaps.

It does agree with this review that soccer XAI rarely evaluates its explanations. It also usefully pointed to SkillCorner's open phase-of-play labels.

### 13.3 AI-use disclosure (for the course)
- **Who:** produced by Claude (Opus 5.5) with 19 sub-agents: 6 Opus in round 1; 5 Sonnet scans + 3 Opus deep dives in round 2; 5 Sonnet scans + 2 Opus deep dives in round 3.
- **Verification:** the lead agent verified the load-bearing claims against primary sources (§0.2).
- **Your responsibility:** the research design choices and final judgments are yours to make and defend.
