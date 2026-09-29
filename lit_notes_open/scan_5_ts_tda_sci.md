# Scan 5: Time Series, TDA, and Scientific ML (broad scan)

Scope per brief: (a) time-series XAI/VA + foundation forecasters, (b) TDA/Mapper for ML, (c) scientific ML VA (climate/remote sensing). Labels: FACT (verified via arXiv/OpenAlex/GitHub in this session), AUTHOR CLAIM (from a paper's own abstract, unverified further), SYNTHESIS (my inference), SPECULATION (guess).

Queries run: WebSearch ×2 ("GALE Mapper visual analytics... Xenopoulos Silva TVCG"; "Fabio Miranda NYU visual analytics climate urban data"). Everything else via `export.arxiv.org/api/query`, `api.openalex.org/works`, `api.github.com` (curl), and two WebFetch calls (ctsilva.github.io/publications; mlanthology page — low yield, see below).

---

## (a) Time-series model visual analytics / XAI

**Instructor-lab overlap:** None found for time-series XAI or forecasting specifically. FACT: ctsilva.github.io/publications (fetched) lists no time-series work; Miranda's 2024-25 output is urban/climate *simulation* visualization (heat, rainfall — see area (c)), not model-XAI for forecasters. SYNTHESIS: this sub-area looks genuinely open relative to VIDA, which is unusual for this course.

**Key 2023–2026 papers** (one line each):
1. Höllig, Thoma, Grimm, **XTSC-Bench: Quantitative Benchmarking for Explainers on Time Series Classification** (arXiv 2023, preprint). Benchmark comparing saliency/SHAP-style TSC explainers on faithfulness metrics. Code open: FACT, `github.com/JHoelli/XTSC-Bench` (verified reachable).
2. Bento, Saleiro, Cruz, Figueiredo, Bizarro, **TimeSHAP: Explaining Recurrent Models through Sequence Perturbations** (KDD 2021; still the standard baseline cited by later work). Peer-reviewed. Code open (Feedzai `timeshap` repo, widely used).
3. Nayebi et al., **WindowSHAP** (J. Biomedical Informatics 2023, peer-reviewed). Windowed Shapley attribution for TSC, more efficient than TimeSHAP.
4. Meng, Wagner, Triguero, **Explaining time series classifiers through meaningful perturbation and optimisation** (Information Sciences 2023, peer-reviewed).
5. Hao, Shi, Ye, Zeng, **TimeTuner: Diagnosing Time Representations for Time-Series Forecasting with Counterfactual Explanations** (2023, VA venue — TVCG-style; not VIDA, Wei Zeng's HKUST(GZ) group). Directly relevant VA tool for forecaster diagnosis. Code openness: unverified (no GitHub repo found under guessed names — SPECULATION that it may not be public).
6. Kjærnli et al., **EXPRTS: Exploring and Probing the Robustness of Time Series Forecasting Models** (arXiv 2024). Interactive tool for stress-testing forecasters. Code open: FACT, `github.com/AneoGroup/EXPRTS` (verified reachable).
7. Schlegel, Oelke, Keim, El-Assady, **An Empirical Study of XAI Techniques on Deep Learning Models for Time Series Tasks** (2020, Konstanz/ETH VA group) — foundational, still cited; establishes saliency evaluation protocol later benchmarks extend.
8. Jang, Kim, Yang, **TIMING: Temporality-Aware Integrated Gradients for Time Series Explanation** (2025, preprint) — recent attribution method, ML-only, no VA component.
9. Foundation forecasters (object of study, not VA): **Chronos** (`amazon-science/chronos-forecasting`), **TimesFM** (`google-research/timesfm`), **Moirai/Moirai-MoE/Moirai 2.0** (`uni2ts`) — all FACT, repos verified live; Moirai 2.0 preprint Nov 2025.
10. Peter et al., **Software Frameworks for XAI in TS Classification: A Systematic Review** (arXiv Aug 2026) — recent survey, useful for scoping.

**Saturation verdict: active, not saturated for the VA angle specifically.** Evidence: the *methods* literature (attribution algorithms, benchmarks like XTSC-Bench, ProtoScore, TIMING, DeltaSHAP) is dense and still growing weekly through Sept 2026 — mostly ML-venue papers, not VA-venue. But genuine interactive **visual analytics tools for diagnosing forecaster failures** are sparse: I found only TimeTuner (2023) and EXPRTS (2024) as close analogues, neither from VIDA, and none targeting the new foundation forecasters (Chronos/TimesFM/Moirai) specifically. No VA paper surfaced that visualizes *foundation-forecaster* failure modes (zero-shot extrapolation errors, context-length sensitivity) — this looks like the actual open niche, not "apply SHAP to a new dataset."

**Open data (verified via curl HTTP 200):**
- UCR/UEA Time Series Archive — `cs.ucr.edu/~eamonn/time_series_data_2018/`, standard academic license, ~128 UCR + 30 UEA datasets, small (MB range). Verified reachable.
- Monash Time Series Forecasting Repository — `forecastingdata.org`, CC-BY-4.0-style academic use, ~30 datasets. Verified reachable.
- ETT (Electricity Transformer Temperature) — `github.com/zhouhaoyi/ETDataset`, MIT-license repo, small (~MB). Verified reachable.
- M4 / M5 competition data — `github.com/Mcompetitions/M4-methods` and `M5-methods`, open competition data, medium size (M5 ~30k series). Both repos verified reachable.
- Chronos/TimesFM/Moirai pretrained weights and inference code are all open-weight and runnable on a single 11GB GPU or CPU for zero-shot inference (SYNTHESIS: fine-tuning full Moirai/TimesFM is heavier; zero-shot rollout + error visualization is the feasible slice for this student).

**Candidate research questions:**
1. **RQ-A1 (visual diagnosis of foundation-forecaster failure modes).** *Reproduction target:* run Chronos-bolt/TimesFM/Moirai zero-shot on Monash + M4 subsets (public eval code in each repo). *Minimal extension:* a Streamlit/Plotly linked-view tool clustering forecast-error patterns (by context length, seasonality strength, distribution shift) linked to per-series attribution (WindowSHAP/integrated gradients) — EXPRTS's approach retargeted at foundation models. *Evaluation plan:* case studies + a self-administered check comparing tool-driven vs. blind-metric error hypotheses. *Falsification check:* arXiv searches for "visual analytics"+"Chronos/TimesFM/foundation"+"forecast" and "Chronos"+"failure" found no VA tool, only ML-side failure papers (e.g. "Causal Analysis for Time Series Foundation Models," Aug 2026). Looks unclaimed but adjacent ML work (MAUSAM, "Numerical models outperform AI weather forecasts...") is growing fast — time-sensitive.
2. **RQ-A2 (reproduce + stress-test an attribution benchmark).** *Reproduction target:* XTSC-Bench (open code). *Minimal extension:* port its faithfulness metrics from classifiers to Chronos/TimesFM and add a linked-view UI to browse metric disagreements. *Evaluation plan:* deletion/insertion faithfulness curves + the UI. *Falsification check:* no evidence anyone has ported XTSC-Bench-style metrics to foundation forecasters (arXiv "XTSC"+"foundation model": no hits); lower upside than RQ-A1, lower risk too.

**Feasibility in ~40h:** Data: trivial (all small, direct-download). Compute: zero-shot inference on 11GB GPUs or CPU is fine; no fine-tuning needed. Front-end burden: Streamlit/Plotly linked views match the student's stated skill (weak D3, comfortable with Streamlit) — **good fit**.

**Publication path:** VIS/TVCG short paper, or a workshop at a forecasting-adjacent venue (e.g., NeurIPS Time Series workshop). Plausible but competitive given how fast the ML side moves.

**Red flags:** (1) the *methods* side is crowded and fast-moving — a purely new attribution method would be scooped; the VA angle is the safer bet. (2) "Foundation forecaster + XAI" is attracting attention fast (weekly arXiv output as of Sept 2026); a 2-month window may close before the report is due. (3) Evaluating "is this a good visual explanation" is inherently subjective — needs a concrete quantitative proxy (faithfulness/deletion metrics) to avoid an ungradeable, purely qualitative deliverable.

---

## (b) TDA for ML (Mapper / persistent homology)

**Instructor-lab overlap: HEAVY — this is the most-mined sub-area in the whole slice.** FACT (verified via OpenAlex + arXiv, full author lists pulled):
- **GALE: Globally Assessing Local Explanations** — Xenopoulos, Chan, Doraiswamy, Nonato, Barr, Silva (ICML 2022 workshop). Uses Kepler Mapper to compare feature-attribution methods via Mapper-graph structural signatures.
- **MOUNTAINEER: Topology-Driven Visual Analytics for Comparing Local Explanations** — Solunke, Guardieiro, Rulff, Xenopoulos, Chan, Barr, **Nonato, Silva** (TVCG 2024, arXiv 2406.15613). Direct successor to GALE.
- **TopoMap: 0-dimensional Homology Preserving Dimensionality Reduction** — Doraiswamy, Tierny, P. J. S. Silva, **Nonato, C. Silva** (IEEE VIS 2020). TDA-for-DR, same lab.
- **FADEx: Feature Attribution and Distortion-based Explanation of Dimensionality Reduction** — Meneses, Ortigossa, **Silva, Nonato** (arXiv 2026, i.e. *this year*).
- **"Enriching Dimensionality Reduction with Distortion Cues"** — Computers & Graphics 2026 (VIDA-affiliated, same DR-explanation line).
This means both halves of the brief's sub-area — (i) Mapper for explanation-comparison and (ii) topological evaluation of DR — are actively being worked by Silva/Nonato **through 2026**, not just historically. Overlap is not fatal per the brief, but this is about as saturated-by-the-instructor as it gets.

**Key 2023–2026 papers beyond VIDA (yes, active elsewhere):**
1. Rathore, Chalapathi, Palande, Wang, **TopoAct: Visually Exploring the Shape of Activations in Deep Learning** (Computer Graphics Forum 2021; arXiv 2019) — Bei Wang's group (Utah), not VIDA. Predates but still the standard non-VIDA Mapper-on-activations reference.
2. Yan, Sevastjanova, El-Assady, Wang, **TopoAlign: Topology-Aware Visual Representation Alignment** (arXiv, May 2026) — confirms Mapper/topology-for-model-diagnosis is **still active outside VIDA** as of 2026 (Bei Wang + ETH's El-Assady, a VA researcher).
3. Ballester, Casacuberta, Escalera, **Topological Data Analysis for Neural Network Analysis: A Comprehensive Survey** (arXiv Dec 2023) — broad survey, confirms the field but is not itself a VA tool.
4. Purvine, Brown, Jefferson, Joslyn, **Experimental Observations of the Topology of CNN Activations** (2022, PNNL) — non-VIDA, activation-topology angle.
5. "Priscylla Silva" (unrelated name to Claudio Silva — verified different person via WebSearch), **Visual Analytics for Guiding Feature Attribution Method Selection** (IJCAI 2025) — adjacent feature-attribution-comparison VA work, worth checking for method overlap with GALE/MOUNTAINEER but NOT instructor-lab.
6. Kepler Mapper (`scikit-tda/kepler-mapper`) — FACT, repo verified live, is the common open-source substrate nearly all of the above build on.

**Saturation verdict: saturated at VIDA, but the general question ("is Mapper-based model diagnosis active beyond Silva lab?") is answered YES** — TopoAct → TopoAlign is an independent, ongoing line (Utah/ETH), and it hasn't converged with VIDA's line. **Open question GALE/MOUNTAINEER leave, per their own framing:** Mapper is well known to be sensitive to its hyperparameters (cover resolution/overlap, lens/filter choice, clustering algorithm) — the brief specifically flags "parameter sensitivity/stability" and I could not find a paper (VIDA or otherwise) that systematically visualizes or quantifies **Mapper-graph stability under parameter perturbation for the explanation-comparison use case** specifically (as opposed to Mapper stability in the pure-TDA literature, which is a much older, separate topic). This is a plausible, narrow, falsifiable gap.

**Open data:** No dedicated "dataset" — this sub-area consumes model activations/attributions computed from standard open models (e.g., ImageNet-pretrained CNNs, or small open-weight LLMs the student can run) plus attribution outputs from SHAP/IG/LIME (all open-source libraries). Kepler Mapper itself is open source (verified). No license or credentialing issues.

**Candidate research questions:**
1. **RQ-B1 (reproduce MOUNTAINEER's stability question).** *Reproduction target:* GALE or MOUNTAINEER — **but note**: no public code repo verified for either (GitHub search for "mountaineer"/"GALE"+topology/explanations: zero hits; may use a different name or be closed — check directly with authors before committing). *Minimal extension:* if code surfaces, a systematic sensitivity sweep of Mapper hyperparameters (resolution, gain, clusterer) on the same explanation-comparison task, visualized as a parameter-space small-multiples view. *Evaluation plan:* graph-edit/bottleneck distance between Mapper graphs across parameter settings. *Falsification check:* arXiv/OpenAlex searches for "Mapper parameter sensitivity explanation" and "Mapper stability feature attribution" found nothing specific — looks open, if the base code can be recovered. Safe for grading (instructor can validate fast) but must be pitched as filling *their* stated gap, not independent discovery.
2. **RQ-B2 (independent-lab alternative: extend TopoAct/TopoAlign).** *Reproduction target:* TopoAct (Bei Wang group; code openness not yet verified). *Minimal extension:* apply Mapper-on-activations to a small open-weight LLM (Gemma-2-2B, feasible on the 11GB GPUs) and compare against Gemma Scope SAE features as an alternate explanation signal. *Evaluation plan:* case studies + a quantitative agreement score. *Falsification check:* arXiv search "Mapper"+"sparse autoencoder" returned nothing — plausibly open, but SPECULATION (not an exhaustive SAE-literature search).

**Feasibility in ~40h:** Compute is fine (Kepler Mapper is CPU-cheap; a 2-4B model fits the 11GB GPUs). Data is fine (no license issues). **Frontend burden is the real risk**: MOUNTAINEER/GALE-style tools are genuinely nontrivial VA systems (interactive Mapper-graph views, linked coordinated views) — reproducing the *visualization* faithfully with weak D3/JS skills in 40h is optimistic; a Streamlit/Plotly-based simplified reproduction (static or lightly-interactive Mapper graphs via `kmapper`'s built-in HTML export, plus custom linked plots) is more realistic than a full from-scratch VA system.

**Publication path:** Workshop paper (e.g., a TDA-in-ML workshop) or VIS short paper if the parameter-stability angle is developed rigorously; ceiling is capped by direct instructor-lab overlap unless clearly framed as extending their stated open question.

**Red flags:** (1) **Could not verify public code for GALE or MOUNTAINEER** — this must be checked directly (email authors or check TVCG supplemental) before committing to this as a reproduction target; if no code exists, "reproduce" becomes "reimplement from the paper," which is much more than 40h. (2) Heaviest instructor-lab overlap of anything in this scan (four papers through 2026) — high risk of "we already showed this" pushback unless the RQ is framed as a stated gap. (3) TDA has a real learning-curve/engineering cost (Mapper hyperparameters, persistence computation) on top of frontend work — double burden for a student with weak frontend skills.

---

## (c) Scientific ML with visual analytics (climate / remote sensing)

**Instructor-lab overlap: partial, but on a different angle.** FACT (via WebSearch on Miranda, cross-checked with ctsilva.github.io fetch which showed no matches): Fabio Miranda (VIDA-trained, now UIC) works on **urban-climate simulation visualization** — volumetric weather-simulation exploration for urban heat, "Urban Computing for Climate and Environmental Justice" (2024 overview), PluvWeb rainfall-impact system (SIBGRAPI 2025) — i.e., visualizing *physical simulation output*, not **evaluating ML model attributions against physical ground truth**, which is what the brief's sub-area (c) actually targets. SYNTHESIS: overlap exists in "climate + visualization," but the specific angle (XAI-for-climate-ML validated against physics) looks open relative to VIDA.

**Key 2023–2026 papers:**
1. Bommer, Kretschmer, Hedström, Bareeva, Höhne, **Finding the right XAI method — A Guide for the Evaluation and Ranking of XAI Methods in Climate Science** (arXiv Mar 2023; journal version AI for the Earth Systems 2024, AUTHOR CLAIM on venue — title/authors confirmed via arXiv, journal placement not independently re-verified this session). Ranks attribution methods against physically-motivated criteria.
2. Mamalakis, Ebert-Uphoff, Barnes, **Neural network attribution methods for problems in geoscience: A novel synthetic benchmark dataset** (Environmental Data Science 2022, peer-reviewed). Ground-truth-known synthetic benchmark for evaluating XAI faithfulness in geoscience — the "Mamalakis synthetic benchmarks" cited in the brief.
3. Mamalakis, Barnes, Ebert-Uphoff, **Investigating the Fidelity of XAI Methods for CNNs in Geoscience** (AI for the Earth Systems 2022, peer-reviewed) — companion paper.
4. Höhl, Obadić, Fernández-Torres, Najjar, Oliveira, Akata, **Opening the Black-Box: A Systematic Review on XAI in Remote Sensing** (arXiv 2024). Directly scopes sub-area (c)'s remote-sensing half.
5. O'Loughlin, Li, Neale, O'Brien, **Moving beyond post-hoc XAI: Lessons learned from dynamical climate modeling** (Geoscientific Model Development 2025 / preprint 2024) — recent critical perspective, useful as a "what's still unsolved" pointer.
6. Krell, Mamalakis, King, Tissot, Ebert-Uphoff, **The influence of correlated features on neural network attribution methods in geoscience** (Environmental Data Science 2025) — 2025 follow-up, shows the line is still active, not closed out in 2022.
7. **Error in ERA5 2m Temperature identified using GraphCast** (arXiv, Jan 2026) — a concrete instance of using a foundation weather model's errors diagnostically against reanalysis; not itself a VA tool, but directly answers the brief's question that GraphCast-error analysis exists.
8. **MAUSAM: An Observations-focused assessment of Global AI Weather Prediction Models During the South Asian Monsoon** (arXiv Sept 2025) — failure-mode analysis of AI weather models (Pangu/GraphCast-class) against observations, regional focus.
9. **Numerical models outperform AI weather forecasts of record-breaking extremes** (arXiv Aug 2025) — high-profile failure-analysis paper on AI weather models vs. physical NWP, relevant "where does the foundation model break" framing.
10. Roussel, Böhm, **Geospatial XAI: A Review** (ISPRS Intl. J. Geo-Information 2023, peer-reviewed) — broader geospatial-XAI framing including remote sensing.

**Saturation verdict: active but VA-thin.** Evidence: the *ML/XAI-methods* and *failure-analysis* literature for climate/weather (Mamalakis line, Bommer, GraphCast-error papers, MAUSAM) is substantial and still growing through Jan 2026. But I found **zero dedicated interactive visual-analytics tools** for browsing/comparing these errors (the closest VA-adjacent hit, Forte, is for power-grid net-load forecasting trust, not climate/weather). This matches the brief's framing: "does VA of weather/climate foundation models' errors exist?" — **answer: the error-analysis exists (GraphCast/ERA5, MAUSAM), the *visual-analytics* layer on top of it does not appear to, as of this scan.**

**Open data (verified reachable):**
- WeatherBench 2 — `github.com/google-research/weatherbench2`, Apache-2.0-style open repo, verified reachable (HTTP 200); ERA5-based evaluation framework, data is derived from ECMWF ERA5 (open, free registration via Copernicus CDS, not credentialed/NDA).
- EuroSAT — `github.com/phelber/EuroSAT` verified reachable; Sentinel-2 based, open, small (~27k images, ~2GB), MIT-style academic license.
- BigEarthNet — official site `bigearth.net` verified reachable (HTTP 200; the specific mirror repo I guessed, `kai-tub/bigearthnet`, was a 404 — wrong repo name, not evidence against the dataset itself); Sentinel-1/2, CC0/open, large (~590k patches, tens of GB — a real compute/storage consideration for a 40h/32GB-laptop budget).
- GraphCast / Pangu-Weather inference code: both have public repos (DeepMind GraphCast, Huawei Pangu — not re-verified by direct curl this session, SPECULATION based on general knowledge that they are open-weight; **flag as needing direct verification before committing**).

**Candidate research questions:**
1. **RQ-C1 (VA layer on top of GraphCast/Pangu vs. ERA5 error diagnosis).** *Reproduction target:* the "Error in ERA5 2m Temperature identified using GraphCast" line — rerun GraphCast (or a smaller open weather-ML baseline) zero-shot against a WeatherBench2 ERA5 subset. *Minimal extension:* a Streamlit/Plotly linked-view tool browsing spatial/temporal error patterns linked to physical covariates (orography, land-sea mask) — VA layered on top of ML error-attribution work that already exists. *Evaluation plan:* case studies tied to known physically-implausible failure modes (extremes, per "numerical models outperform AI..."). *Falsification check:* arXiv "visual analytics"+"weather"+"forecast" surfaced no foundation-model-specific VA tool (closest, Forte, is a different domain). Moderate confidence this is open.
2. **RQ-C2 (XAI-for-remote-sensing VA, ground-truthed).** *Reproduction target:* Mamalakis-style synthetic benchmark (open, small) applied to EuroSAT/BigEarthNet-subset classifiers. *Minimal extension:* a linked-view tool comparing attribution maps (Grad-CAM/IG/SHAP) against physically-plausible ground truth (known spectral bands per land-cover class) instead of qualitative eyeballing. *Evaluation plan:* faithfulness + physical-plausibility scoring, visualized. *Falsification check:* Höhl et al. 2024 surveys this space but as a review, not a VA tool — no dedicated interactive VA tool found.
3. **RQ-C3 (lower-risk fallback).** Same as RQ-C2 but capped to EuroSAT alone (small, CPU-feasible), skipping BigEarthNet's storage burden — the "grade-safety first" version.

**Feasibility in ~40h:** Data: EuroSAT trivial; BigEarthNet and full ERA5/WeatherBench2 are storage/compute-heavy (tens of GB, GPU-hours) — **use small subsets**, a real 40h risk if underestimated. Compute: zero-shot inference with GraphCast/Pangu on a laptop is likely infeasible without the SLURM cluster (SYNTHESIS — these are large graph-neural or transformer models; would need the cluster, not the 11GB GPU box). EuroSAT-scale CNN work is easy on the M1 Pro. Frontend: Streamlit/Plotly fits well.

**Publication path:** Domain workshop (e.g., Climate Change AI, or a VIS/EarthVis-adjacent workshop) more plausible than a core VIS venue, since the novelty is largely in the domain application rather than a new VA technique.

**Red flags:** (1) Physical-ground-truth evaluation is exactly what makes this defensible against "subjective XAI eyeballing," but building or sourcing a *good* synthetic/physical ground truth is nontrivial extra work beyond the 40h if not reusing Mamalakis's existing benchmark directly. (2) Full-scale ERA5/WeatherBench2/GraphCast work is compute-heavy and may exceed laptop+11GB-GPU capacity — needs the SLURM cluster, adding scheduling risk. (3) BigEarthNet's size is a real burden — prefer EuroSAT unless a small BigEarthNet subset is explicitly available.

---

## Final triage table

| Sub-area | Best RQ | Novelty evidence (1 line) | Feasibility | Grade-safety | Pub. upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) TS foundation-model failure VA | RQ-A1: linked-view diagnosis tool for Chronos/TimesFM/Moirai zero-shot errors | No VA-specific paper found pairing foundation forecasters with an interactive diagnosis tool (only EXPRTS/TimeTuner for classical models) | High | Med-High | Med | **yes** | Best novelty/feasibility ratio; no VIDA overlap; fits Streamlit/Plotly skills; but time-sensitive (fast-moving ML field) |
| (b) TDA/Mapper stability for explanation comparison | RQ-B1/B2: parameter-sensitivity study of Mapper graphs, or Mapper-vs-SAE agreement on a small LLM | GALE→MOUNTAINEER→FADEx line is active through 2026 at VIDA; TopoAlign (2026) shows non-VIDA activity too; parameter-stability angle unclaimed | Med (frontend + TDA double burden; code availability for GALE/MOUNTAINEER unverified) | Med (very safe if framed as instructor's stated gap; risky if code missing) | Med-High (closest to a "known" publishable niche) | **maybe** | Only pursue after verifying GALE/MOUNTAINEER code exists; otherwise pivot to TopoAct-based reproduction |
| (c) Climate/remote-sensing XAI VA | RQ-C2/C3: ground-truthed attribution-method comparison VA on EuroSAT | Höhl et al. 2024 reviews the space but no dedicated VA tool found; Mamalakis benchmark is reusable and open | High (EuroSAT-scale) / Low (full ERA5/GraphCast scale) | High (EuroSAT scale) | Low-Med | maybe | Good grade-safety at EuroSAT scale, but capped novelty/pub. upside unless scaled to weather (which raises compute risk) |

**Note on the "GraphCast/ERA5" version of (c):** if the student wants higher publication upside and is willing to use the SLURM cluster, RQ-C1 (weather foundation-model error VA) has better novelty than RQ-C2, at the cost of feasibility — a genuine trade-off to flag for the deep-dive phase.
