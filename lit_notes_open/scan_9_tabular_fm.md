# Scan 9: Tabular foundation models (TFMs) + XAI / visual analytics

Date: 2026-09-29. Tools: arXiv API, OpenAlex, gh api, HF API, 9 WebSearch, 6 WebFetch. Labels: FACT (I saw it in a fetched abstract/page/API), CLAIM (authors' claim), SYNTH (my synthesis), SPEC (speculation). Anything not listed as verified is "unverified".

**Big caveat.** This field moves monthly. Several of the closest papers below are from May-Aug 2026, so "nobody did X" claims have a short half-life. Re-run the falsification queries just before the Oct 20 proposal.

## 0. Instructor-lab overlap (applies to all sub-areas)

- OpenAlex author sweep, Claudio T. Silva (A5003584200) and Cláudio Silva (A5052577630), works since 2023, filtered by titles matching tabular/explain/attribut/shap/calibr/interpret/feature/fair/foundation/in-context/uncertain/rashomon. Hits (FACT): "Visagreement: Visualizing and Exploring Explanations (Dis)Agreement" (2024 preprint, TVCG 2025), "Exploring the Relationship Between Feature Attribution Methods and Model Performance" (2024 arXiv), "T-Explainer" (arXiv 2024, IEEE Intelligent Systems 2025), "A Visualization-Driven Decision Support System for Selecting Feature Attribution Methods" (2025, SSRN). These are all attribution-comparison work on classical models.
- **Zero hits for TabPFN / tabular foundation model / in-context** in the instructor's OpenAlex list. Web search "Claudio Silva" + TabPFN/in-context/tabular foundation model also found no link (FACT for what the search returned; OpenAlex coverage of VIDA is incomplete, so "unverified" that nothing exists). Also should check ctsilva.github.io/publications by hand (not done).
- Prior cohort check: `gh search repos` for "TabPFN visualization", "DS-GA 3001 visualization machine learning", "visml silva nyu", "Visualization for Machine Learning NYU final project" returned nothing relevant (FACT). The 2024 syllabus page exists at ctsilva.github.io/2024-VisML-CDS/ (found in search). I could not see past project lists.
- SYNTH: the lab's stated open question is that feature attributions disagree across models/explainers. TFMs add a new axis: attribution is cheap (see below) but *which context rows* drove a prediction is a new object. That framing is an extension of their question, not "Visagreement on TabPFN".

## 1. Area card (a): TFM capabilities and failure analysis

**Key papers**
1. Hollmann et al., "Accurate predictions on small data with a tabular foundation model" (TabPFN v2), Nature 2025. Peer-reviewed. OpenAlex confirmed. Code open (Apache-2.0 code).
2. Qu et al., TabICL, arXiv 2502.05564 (ICML 2025, unverified venue). Code open (soda-inria/tabicl, BSD-3 text, GitHub shows NOASSERTION). TabICLv2 (arXiv 2602.11139, "ICML 2026" per repo README) is the current default; README says "permissive license". FACT.
3. TabPFN-2.5 / TabPFN-3 technical report (arXiv 2605.13986) and TabPFN-3.5 (current default in README). FACT: repo README lists 2.5/2.6/3/3.5 weights as non-commercial; TabPFN-2 weights under Prior Labs License (Apache-2.0 + attribution).
4. Erickson et al., "TabArena: A Living Benchmark" arXiv 2506.16791, NeurIPS 2025 D&B spotlight. FACT: CC BY 4.0, code at tabarena.ai / autogluon/tabrepo (Apache-2.0, updated today). CLAIM: GBDTs stay competitive; foundation models strongest on small datasets. Number of datasets (I recall 51) unverified.
5. McElfresh et al., "When Do Neural Nets Outperform Boosted Trees on Tabular Data?", NeurIPS 2023 D&B (from memory, unverified this session): 19 algorithms x ~176 datasets, meta-feature analysis. This is the classic meta-feature stratification.
6. Herre et al., "Explaining Tabular Foundation Model Differences Through Meta-Features", arXiv 2605.28418 (May 2026, preprint). FACT from abstract: uses TabArena results, model-agnostic meta-features, FDR-controlled tests. Findings: NN vs tree gap has no meta-feature surviving FDR; foundation vs non-foundation has one robust association that fails leave-one-dataset-out; TabICLv2 vs TabPFN-2.6 has one that generalizes. **This nearly kills the "meta-feature failure analysis of TFMs" idea.**
7. Shaheen et al. (Hutter group), arXiv 2608.17957 (Aug 2026): TFM pretraining on a single real table; number of features predicts usefulness. Preprint.
8. Other TFMs: TabDPT (HF Layer6/TabDPT exists, license unverified), Mitra (HF autogluon/mitra-finetune, Apache-2.0 tag), LimiX (HF stable-ai/LimiX-*, license tag "other"; the LimiX-Team GitHub path I guessed 404'd, so repo unverified), TabuLa-8B (HF mlfoundations/tabula-8b, Llama-3 license; 8B is too big for the 11 GB GPUs unquantized). CARTE not verified.
9. "Realistic Evaluation of TabPFN v2 in Open Environments" (arXiv 2025, seen in OpenAlex, authors/venue unverified): distribution shift study, so shift is also partly covered.

**Saturation:** *Benchmarking and meta-feature failure analysis: saturated/active* (TabArena, TALENT, McElfresh, Herre). Ship-ready model zoo is huge.
**Open data:** OpenML-CC18 and TabArena datasets are openly downloadable via OpenML (CC BY 4.0 for the TabArena curation; individual dataset licenses vary, check per dataset; I did not enumerate them). TALENT: 112 datasets, per the ESANN paper below.

**RQs**
- Grade-safe: Reproduce a TabArena slice (e.g. 10-15 small datasets, TabPFN + TabICLv2 + LightGBM/CatBoost) and reproduce Herre et al.'s meta-feature test on a subset. Extension: add *instance-level* stratification (per-row regions where TFM loses to GBDT, via a Plotly linked-view scatter of loss difference over a UMAP/feature space). Evaluation: does a subgroup-discovery method (e.g. shallow tree on loss difference) find stable regions across seeds? Falsification: queries "tabular foundation models fail GBDT meta-features" (found Herre), "TabPFN error slices" (not yet run - unverified). Risk: subjective viz value.
- Higher-upside: none strong here; dataset-level story is taken.

**Feasibility (40 h):** High for reproduction. TabPFN-3.5 on M1 via MPS (README says Apple Silicon GPU auto-supported; CPU limited to 5000 samples unless env override) and the 11 GB GPUs (README: ~8 GB VRAM works, 16 GB for some large). Front-end light (Streamlit/Plotly).
**Publication:** workshop only (e.g. NeurIPS/ICML TRL or "Table Representation Learning" workshop, IEEE VIS poster). **Red flag:** the dataset-level version is scooped.

## 2. Area card (b): Explaining TFMs (feature and context-row attribution, multiplicity)

**Key papers**
1. Rundel et al., "Interpretable Machine Learning for TabPFN", arXiv 2403.10923 (2024; xAI 2024, venue unverified). FACT: adapts PDP/ICE/SHAP, exploits that ICL can "simulate retraining" by changing context; efficient exact-ish Shapley. Code via shapiq/tabpfn-extensions. 
2. PriorLabs/tabpfn-extensions (Apache-2.0, pushed 2026-09-25): interpretability module with SHAP, PDP, feature selection. FACT.
3. Miftachov et al., "Interpretable Tabular Foundation Models via In-Context Kernel Regression" (KernelICL), arXiv 2602.02162 (Feb 2026, preprint). FACT: predictions as weighted average of training labels; interpretability quantified by weight-distribution perplexity; 55 datasets; code link not seen. **This is context-row explanation, by design, for a modified model.**
4. Understanding Context Sampling in TabPFN on Small Tabular Datasets, Abdullah, arXiv 2607.26628 (Jul 2026, single-author preprint). FACT: predictions unstable on small contexts; no per-row valuation; 15 OpenML datasets; code github.com/mohammed1916/tabmx.
5. Dong et al., "Algorithmic Recourse of In-Context Learning for Tabular Data", ICML 2026 (arXiv 2605.31272). FACT: zeroth-order recourse for black-box ICL. Mostly LLM-ICL.
6. Balef et al., "Towards Understanding Layer Contributions in Tabular ICL Models", arXiv 2511.15432 (Eggensperger group): layers-as-painters analysis of TabPFN and TabICL; redundancy. Preprint.
7. Hu and Ghelichi, "Topological Signatures of Context-Level Reliability in TabPFN", arXiv 2607.17962 (Jul 2026). FACT: persistent homology of internal representations correlates with dataset-level reliability; synthetic benchmarks; no code mentioned. Relevant to course topic TDA (Nov 10).
8. McCarter, "What exactly has TabPFN learned to do?" (2025 blog/ICLR blogposts style), black-box inductive-bias probing. Zhang et al., "Towards Fair ICL with Tabular FMs" (arXiv 2505.09503), fairness for ICL context selection.
9. ExplainerPFN (OpenAlex 2026, arXiv, zero-shot feature importance estimation with a TFM): the reverse direction (TFM as explainer). Details unverified.
10. EquiTabPFN (arXiv 2502.06684): target-permutation equivariance gap.

**Instructor-lab overlap:** none found on TFMs (see section 0). Adjacent: Visagreement / MOUNTAINEER / T-Explainer (attribution comparison).
**Saturation:** feature-level explanation of TabPFN = *active but mostly done* (Rundel + extensions ship it). **Context-row (data attribution) explanation for unmodified TabPFN/TabICL = thin** as of my searches. Two WebSearches with "data attribution / valuation / influence + TabPFN" found only the papers above; the closest are KernelICL (modifies the model) and the context-sampling paper (aggregate only). No visual analytics paper found for any TFM (query "visual analytics tabular foundation model", OpenAlex, returned nothing relevant). Limitation: this is 3-4 queries, so status is "thin (unconfirmed)".

**Open data:** OpenML datasets (small binary/multiclass, 500-5000 rows keeps context-attribution cheap). Verified reachable only via papers' claims and the OpenML ecosystem; I did not download any.

**RQs**
- Grade-safe: **Reproduce Rundel et al. (SHAP/PDP for TabPFN via tabpfn-extensions/shapiq) on 3-5 OpenML datasets and compare with SHAP on LightGBM**: how much do TabPFN and GBDT attributions disagree (rank correlation, top-k overlap)? Extension: Streamlit/Plotly dashboard for the disagreement. Careful: this is squarely Visagreement territory (SYNTH), so frame as "does the disagreement they studied across explainers also appear across model *families*, esp. TFM vs GBDT" and cite them as the motivator. Falsification queries: "Interpretable Machine Learning for TabPFN" (exists, no VA), "Visagreement TabPFN" (no hit found in my sweep). Risk: weak novelty.
- Higher-upside: **Context-row attribution for TabPFN/TabICL, with a visual analytics interface.** Method: leave-one-context-row-out / Shapley-over-context-rows (cheap since no retraining: one forward pass per subset) as data-valuation for a test prediction; compare against attention-based and KernelICL-style weights (KernelICL gives the "transparent" baseline, but only for its modified head). Evaluate with (i) removal/insertion fidelity curves (remove top-k valued rows, measure log-loss change), (ii) agreement with kNN in embedding space, (iii) stability across context subsamples and feature/row permutation ensembles (Rashomon-style multiplicity: does the *set* of influential rows change across TabPFN's internal ensemble members?). VA piece: linked view of test point, top influential rows, their labels, their location in a DR projection (course topic Oct 20), plus multiplicity spread. Risks: compute grows with context size (limit to <=1000 rows), novelty could vanish given the pace (KernelICL, context-sampling papers appeared Feb/Jul 2026), and evaluation of a VA piece is subjective, so lean on the quantitative fidelity + stability study. Falsification queries run: "TabPFN which training examples influence prediction ... data attribution", "tabular in-context learning context row importance data valuation", "TabPFN Rashomon predictive multiplicity context sampling". Not yet run: Google Scholar "in-context data valuation tabular", "TabICL attention rollout".

**Feasibility (40 h):** Medium-High. All models run on M1/11 GB GPUs for small contexts. Front-end: Plotly/Streamlit fine. Engineering risk: efficient batched forward passes over context subsets (SPEC: use tabpfn `fit` once and mask/subset; each subset re-fits the KV cache).
**Publication:** IEEE VIS short paper / VISxAI workshop, or NeurIPS/ICML workshop on TRL/XAI. If the fidelity study is strong, a full XAI-venue paper is conceivable (SPEC).
**Red flags:** fast-moving competition; "explanation" fidelity metrics contested; overlap with lab must be framed carefully.

## 3. Area card (c): Calibration and uncertainty (TFM vs GBDT)

**Key papers**
1. De Melo Costa et al., "High Performance, Low Reliability: Uncertainty Benchmarking for Tabular Foundation Models", arXiv 2605.28554, ESANN 2026 (peer-reviewed). FACT: 112 TALENT datasets; TFMs best AUC but lower *conditional* coverage under conformal prediction than GBDTs. Code released (link not opened).
2. "TabPFN beyond Tabular Data: Calibration and Accuracy on Multimodal Embeddings", arXiv 2607.11007 (Jul 2026, preprint): TabPFN best mean rank on NLL and ECE among 9 heads across 22,820 episodes. Authors Zhang et al. (unverified beyond first).
3. Benchmarking TFMs for Conditional Density Estimation in Regression (arXiv 2603.26611) and Distributional Regression with TFMs via proper scoring rules (arXiv 2603.08206): TabPFN variants top on CDE loss/CRPS; calibration mixed at larger n. Titles seen in search results; details as reported by the search snippet only.
4. TabPFN-2.5 report (Prior Labs), TabArena (calibration not the focus).

**Saturation:** *Active/crowded* for aggregate benchmarks. Untouched-seeming: *where* in the input space TFM uncertainty is miscalibrated (conditional-coverage failures) explained visually, but the ESANN paper already flags conditional coverage.
**Instructor-lab overlap:** Calibrate (VIDA, listed in the brief; venue not verified by me) is a calibration VA tool, which makes a TFM-calibration VA a natural extension. Unverified whether it handles tabular.

**RQs**
- Grade-safe: Reproduce De Melo Costa et al. on ~10-20 TALENT/OpenML datasets (reliability diagrams, ECE, split-conformal coverage, size-stratified coverage), TabPFN + TabICLv2 vs LightGBM/CatBoost with matching post-hoc calibration. Extension: show that post-hoc temperature/Platt on GBDT closes the gap (or not), i.e. a fair-comparison check, and a linked-view coverage-by-region explorer. Falsification: "tabular foundation models calibration uncertainty conformal TabPFN TabICL versus gradient boosting" (found ESANN paper, which is the reproduction target, not a scoop of the extension).
- Higher-upside: explain *why* conditional coverage fails, using the context-row attribution from section 2 (miscalibrated regions = sparse/conflicting context). Adds compounding risk (two research steps in 40 h). 

**Feasibility:** High for the grade-safe RQ (no heavy compute; split conformal is a few lines). **Publication:** workshop. **Red flag:** novelty over ESANN paper is thin unless tied to explanation.

## 4. Licensing and compute checks (FACT unless noted)

- TabPFN code: Apache-2.0. **Weights are separately licensed.** TabPFN-2.5/2.6/3/3.5 weights are non-commercial licenses; TabPFN-2 weights are Apache-2.0 + attribution (Prior Labs License). First use requires logging in at ux.priorlabs.ai and accepting the license, token via `TABPFN_TOKEN`, or manual HF download (the HF repo is gated: `Prior-Labs/tabpfn_3_5`). A course project is research/non-commercial, so fine, but **plan for the login step on the SLURM/GPU machines** (headless token path documented).
- TabICL: BSD-3-Clause text in repo; HF mirrors tagged bsd-3-clause; TabICLv2 README says permissive. Not gated as far as I saw. Easiest for reproducibility.
- Mitra: HF autogluon/mitra-finetune Apache-2.0 tag. LimiX: license "other" (read before use). TabuLa-8B: Llama-3 license, 8B, too big for one 11 GB card without quantization.
- Compute: README recommends GPU, ~8 GB VRAM ok, 16 GB for large; CPU limited to 5000 rows (override env var); Apple Silicon MPS supported (PyTorch >= 2.13 recommended). M1 Pro 32 GB is enough for contexts <= a few thousand rows. TabPFN-3.5 states up to 1M rows and 20k features, irrelevant at course scale.
- Benchmarks: TabArena code Apache-2.0, curation CC BY 4.0; OpenML-CC18 via openml python client (not downloaded here, so "unverified locally").

## 5. Final triage table

| Sub-area | Best RQ | Novelty evidence | Feasibility | Grade-safety | Pub upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) TFM failure analysis | Reproduce TabArena slice + meta-feature test; extend to instance-level failure slices | Herre et al. 2605.28418 + McElfresh 2023 cover dataset-level | High | High | Low | no | Dataset-level story scooped; instance-level is small twist |
| (b) Feature attribution for TFM vs GBDT | Attribution disagreement across model families with dashboard | Rundel 2024 done; Visagreement-adjacent | High | High | Low-Med | maybe | Safe reproduction, but weak novelty and lab overlap |
| (b) Context-row attribution + multiplicity + VA | Leave-out/Shapley over context rows for TabPFN/TabICL, fidelity + stability, linked-view | Only KernelICL (modified head) and aggregate context-sampling study found; no VA found | Med | Med | Med-High | **yes** | Thin literature, clear reproduction anchor (Rundel/tabpfn-extensions), tie to DR + TDA topics |
| (c) TFM calibration | Reproduce ESANN coverage benchmark + fair post-hoc calibration + coverage explorer | ESANN 2026 paper already the finding | High | High | Low-Med | **yes (as the grade-safe backbone)** | Cheap, clean reproduction; combine with (b) for the story |

## 6. Recommended combo (SYNTH)
Reproduce (c) or Rundel-style SHAP as the "reproduce" half (both cheap), extend with context-row attribution (b) as the "extend" half, and use miscalibrated regions from (c) as the use-case for the VA. Keep scope to <= 1000-row contexts, 5-10 OpenML datasets, TabPFN-2 (Apache+attribution, no gating headaches? still needs license accept) and TabICLv2 (BSD) as the two models.

## 7. Biggest red flags
1. Field velocity: closest neighbors appeared Feb-Jul 2026; re-check before Oct 20.
2. Lab-overlap framing: must be presented as extending the Visagreement/Calibrate question to context-row attribution, not re-doing attribution comparison.
3. Evaluation of context attribution: no ground truth; must rely on removal/insertion and stability metrics.
4. Unverified in this scan: TALENT dataset licenses, TabArena dataset count, ctsilva.github.io/publications manual check, ESANN/TabICL venue details, whether Calibrate handles tabular.
