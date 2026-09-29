# Deep dive D2: Visual analytics for diagnosing failures of time-series foundation models (TSFMs)

Written 2026-09-28. This is a falsification-first review, following `DEEP_BRIEF.md`.

**Labels.**
- FACT: verified in this session with a URL, API call or repo.
- AUTHOR CLAIM: stated by a paper, not independently checked.
- SYNTHESIS: my inference.
- SPECULATION: a guess.

**Evidence levels.**
- FULL TEXT: I read the paper body. Either I read it directly (pdftotext), or through a WebFetch of the arXiv HTML, which gives a model-generated digest of the full text. I mark the second case as FULL TEXT (digest).
- ABSTRACT: I read only the arXiv or API abstract.
- SECONDHAND: I only saw the title or a search-engine summary.

**Tools used.**
- arXiv API: about 45 queries.
- OpenAlex: the budget ran out mid-session. The shared-IP daily cap was hit (HTTP 429 "Insufficient budget"), so author-level lab checks fell back to the arXiv API and web.
- Semantic Scholar: one citation pass worked, then it returned 429.
- dblp: blocked (bot check, then 429).
- WebFetch: 7 full-text digests.
- `gh api`: 25 repos.
- Hugging Face API: models, datasets and spaces.
- WebSearch: 7 calls.
- A **small data pilot** on the TIME benchmark's released per-window outputs: 31 MB downloaded, into the scratchpad only.

---

## 0. Bottom line (read this first)

1. **The core candidate RQ, as worded, is largely already done.** Relating per-series error to interpretable series characteristics has been done three times:
   - Kang, Hyndman and Smith-Miles, "instance spaces", in the *International Journal of Forecasting* (IJF), 2017, for classical methods.
   - EXPRTS, 2024: an instance-space view, per-series error and drilldown, for deep-learning forecasters.
   - **TIME** (ICML 2026, Feb 2026): "pattern-level evaluation" of 12+ TSFMs, stratified by 7 STL features, with an interactive Gradio leaderboard.

   The per-characteristic stratification half of the RQ is therefore not novel. FACT for all three, sources in §2.
2. **The instructor's lab has prior art the broad scan missed.** **mTSeer** (Xu, Yuan, Wang, **Silva**, Bertini; CHI 2021) is "an interactive system for the exploration, explanation, and evaluation of multivariate time-series forecasting models", comparing models "in both model and instance level" (FACT, FULL TEXT).
   - Its future-work section explicitly asks for:
     - "more features like the trend of time series"
     - "a larger number of candidate models"
     - "the instance-level evaluation can be expanded"
   - This is overlap, and also the best framing hook available.
3. **Part of the RQ's premise is weak on current data.** On TIME's short-horizon tasks, my pilot finds that four 2025-era TSFMs lose to *seasonal naive* on only **1–3 % of series**. All four lose together on 0.5 % of series and on about 4 % of windows (SYNTHESIS from my own pilot, §5). "Fails relative to simple baselines" is a rare event at the series level.
4. **What survives is a narrower, sharper gap.** Every published stratification is at dataset level or static series level: GIFT-Eval, TIME, spectral predictability, break-even analysis. In my pilot, **about 86 % of the variance in log relative error lies within datasets, and about 66 % lies within a single series across forecast windows.** TSFMs also often disagree at window level: Chronos-2 and TimesFM-2.5 differ by more than 25 % on about 32 % of windows.
   - So a **window-level, context-conditioned, multi-model** failure-regime analysis would address variance that no existing leaderboard view touches.
   - So would an explicit **"in vivo" test of the synthetic failure modes** reported by *Causal Analysis for TSFMs* (Aug 2026). Its authors list this as future work.
   - I have seen no evidence that anyone has done either.
5. **Feasibility is unusually good.** TIME publishes *per-window quantile predictions and metrics for 30 models* (2.9 GB total, about 100 MB per model), plus 30+ tsfeatures per variate. The minimum viable project therefore needs **zero model inference**. Local reruns of small TSFMs (Chronos-2 at 120 M parameters, Moirai-2-small at 11 M, TiRex at 35 M) serve only as a reproduction check. All FACT.

**Verdict: partly addressed.** A grade-safe project is very likely. A workshop paper is plausible if the window-level and in-vivo angle holds up. **Biggest risk:** interpretable features may explain little of the window-level failure variance. The pilot's single-feature Spearman |ρ| is ≤ 0.26. A recent impossibility result (*The Spectrum Is Not Enough*, Jul 2026) argues that spectrum-based features cannot tell you when a TSFM helps. The tool could end up showing noise.

---

## 1. Instructor-lab check (Silva / Nonato / Miranda / VIDA)

| Work | Lab link | Relation to D2 | Evidence |
|---|---|---|---|
| **mTSeer: Interactive Visual Exploration of Models on Multivariate Time-series Forecast.** K. Xu, J. Yuan, Y. Wang, **C. Silva**, **E. Bertini**. CHI 2021, DOI 10.1145/3411764.3445083 | Silva and Bertini (ex-NYU) are co-authors | **Direct VA precedent.** Compares 5 trained models (RF, MLP, LSTM, CNN, VAR) at model and instance level, with SHAP "push-up/push-down" glyphs, period and length analysis. Case studies are PM2.5, traffic and the Istanbul stock exchange, with 2 experts. No foundation models, and no cross-dataset characteristics. | FACT, FULL TEXT (PDF at `lit_notes_open/pdfs/mtseer.pdf`, read via pdftotext) |
| **TiVy: Time Series Visual Summary for Scalable Visualization.** G. Chan, **L. G. Nonato**, T. Palpanas, **C. Silva**, J. Freire. IEEE VIS 2025 (arXiv 2507.18972) | Nonato, Silva | Time-series *data* visualization (symbolic sequential-pattern summaries), not model diagnosis. Could be a component for summarizing many forecast windows. | FACT (authors via arXiv API), ABSTRACT |
| **Time Series Information Visualization: A Review of Approaches and Tools.** Ortigossa, Dias, Nascimento, **Nonato**. IEEE Access 2025 (arXiv 2507.14920) | Nonato | A survey. Useful for related work. | FACT, ABSTRACT |
| Miranda 2023–2026 (ProWis weather-ensemble simulation, Urban Toolkit, spatiotemporal lensing) | Miranda | Simulation or urban visualization. No forecasting-model VA found. | FACT (arXiv author query), ABSTRACT titles |
| Xenopoulos, Rulff, Guardieiro, Solunke, Barr | – | No time-series or forecasting ML work found (arXiv author queries). dblp was unavailable, so this is **incomplete**. | Partially verified |

SYNTHESIS: the broad scan said "no instructor-lab overlap". **That is wrong.** mTSeer is a Silva/Bertini forecasting-model-comparison VA system.
- Overlap is moderate: 2021, pre-TSFM, a single-dataset workflow, trained models, feature attributions.
- It is also an asset. Pitch the project as *"mTSeer's stated future work, at foundation-model and benchmark scale."*
- Worth asking Silva directly whether Ke Xu or Jun Yuan have an unpublished follow-up.

---

## 2. Literature table

Columns follow the brief. Code "Y*" means a repo exists but has no license file.

| Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **It's TIME: next-gen TSF benchmarks** (Qiao, …, Wen, Long) | 2026 | ICML 2026 (AUTHOR CLAIM, arXiv comment and README) | Peer-reviewed | Leakage-free zero-shot TSFM benchmark, with *pattern-level* evaluation | 50 new datasets, 98 tasks; HF `Real-TSF/TIME` (CC-BY-NC-4.0, 220 MB); outputs in `Real-TSF/TIME-Output` (Apache-2.0 card, 2.9 GB) | 7 STL features per **variate** (trend strength and linearity, seasonal strength and correlation, residual ACF-1, spectral entropy, ADF stationarity), **binarized at the global median**. Variates with the same code are retrieved and scores aggregated by geometric mean. | Gradio leaderboard: overall, per dataset, per window (forecast plot), per pattern (7 radio buttons leading to a table) | 12 TSFMs vs seasonal naive; MASE, CRPS | Gains grow with trend strength. Strong seasonality separates models. "Rankings exhibit high sensitivity to stationarity" (Toto is 3rd on non-stationary data, 6th on stationary). High entropy "acts as a performance equalizer". | "Joint patterns are possible" but only single features are analyzed. Sparsity of some pattern combinations. "Most appropriate aggregation level remains an open question." | Y* (`zqiao11/TIME`: 71 stars, pushed 2026-09-23, GitHub license null, README badge says Apache-2.0) | arxiv.org/abs/2602.12147 | FULL TEXT (digest), plus leaderboard code inspected |
| **GIFT-Eval** (Aksu, Woo, …, Sahoo) | 2024 | Venue unverified | Preprint on arXiv | General TSF benchmark | 23 datasets, 144 k series; HF `Salesforce/GiftEval` (Apache-2.0, 1.6 GB) | Stratified by domain, frequency, term and variate count. 6 features (trend, seasonality, entropy, Hurst, stability, lumpiness) used **to characterize datasets**, not errors. | Feature heatmaps by domain and frequency; 4 qualitative failure plots | 17+ models (now 90+ on the leaderboard) | Foundation models are best overall. Chronos and TimesFM degrade at medium and long horizons. Deep-learning models win at high frequency. | Partial pretraining leakage; the long-horizon gap | Y (Apache-2.0). Leaderboard results are **per config only** (`all_results.csv`) | arxiv.org/abs/2410.10393 | FULL TEXT (digest) |
| **fev-bench** (Shchur et al.) | 2025 | Preprint (v4) | Preprint | Realistic benchmark with covariates | 100 tasks; HF `autogluon/fev_datasets` (0.66 GB, license "other") | Win rate and skill score with paired bootstrap confidence intervals | Streamlit leaderboard | 9 TSFMs plus statistical and deep-learning baselines | Covariates matter. Chronos-2 has a 91 % win rate. | **No per-characteristic or instance-level analysis** (digest). Not fully leakage-free. | Y (Apache-2.0) | arxiv.org/abs/2509.26468 | FULL TEXT (digest) |
| **EXPRTS** (Kjærnli, …, Gundersen) | 2024 (v3) | arXiv, "under review"; its own BibTeX is `@misc 2025` | **Preprint** (no venue found) | VA plus interpretable time-series generation for robustness | Electricity, SF traffic, M4-monthly (open) | 4 STL features, PCA "instance space" (builds on Kang et al. 2017), decomposition-based transformations | Bokeh: instance space, series plus forecast, MASE by horizon, transform panel | 3 use cases, **10-person** interview study | Out-of-distribution augmentation improves robustness | Univariate only (a "major limitation"). Users struggled to relate PCA axes to features. | Y* (`AneoGroup/EXPRTS`: 0 stars, no license; stale pins: py3.9, numpy 1.20.3, pandas 1.2.1, bokeh 2.3.3, gluonts 0.12.5) | arxiv.org/abs/2403.03508 | FULL TEXT (digest) |
| **TimeTuner** (Hao, Shi, Ye, Zeng) | 2023 | IEEE VIS 2023 (TVCG 2024) | Peer-reviewed | How time representations (smoothing, sampling) affect forecasters | Sunspots, air pollutants | Counterfactuals, correlation matrix, bivariate stripes | Multiple coordinated views | Expert feedback | Helps guide feature engineering | Not about TSFMs | Y* (`CatherineHao/TimeTuner`: 22 stars, no license, TF 2.10 + Vue 3) | arxiv.org/abs/2307.09916 | ABSTRACT, plus repo |
| **mTSeer** (Xu, Yuan, Wang, Silva, Bertini) | 2021 | CHI 2021 | Peer-reviewed | Compare multivariate forecasting models at model and instance level | PM2.5, traffic, ISE (UCI) | SHAP feature importance, period and length detection | Coordinated views, box glyph, "dif-lines" | 3 case studies, 2 experts | Ranking by RMSE hides instance-level differences | "More features like the trend…", "a larger number of candidate models", "instance-level evaluation can be expanded". Scalability past about 10 features. | Not found | doi 10.1145/3411764.3445083 | FULL TEXT |
| **Visualising forecasting algorithm performance using time series instance spaces** (Kang, Hyndman, Smith-Miles) | 2017 | IJF 33(2) | Peer-reviewed | Where in feature space each method works | M3 | Features, 2D projection, generated series filling gaps | Instance-space footprints | – | M3 is not diverse | – | R (tsfeatures ecosystem). A Python `instancespace` package exists (SoftwareX 2025) | doi 10.1016/j.ijforecast.2016.09.004 | SECONDHAND (search summary) plus OpenAlex metadata |
| **Spectral Predictability as a Fast Reliability Indicator** (Wang, Quan, Yang, Srivastava) | 2025 | Preprint | Preprint | Does spectral predictability Ω stratify model families? | GIFT-Eval leaderboard (51 models, 28 datasets) | Ω = 1 − H/H_max per **dataset** | LOWESS curves | Spearman ρ = −0.65 between Ω and error | TSFMs win when Ω > 0.5; the advantage vanishes when Ω < 0.4 | Stable-spectrum assumption; needs series longer than about 1000 points | Not checked | arxiv.org/abs/2511.08884 | FULL TEXT (digest) |
| **The Spectrum Is Not Enough: When Context Helps** | 2026 | Preprint | Preprint | Can spectral indices predict when context or a TSFM helps? | Surrogate pairs | Impossibility result under phase randomization; proposes a "coverage deficit" diagnostic | – | – | **Spectrum-based indices cannot capture the beyond-second-order structure that TSFMs exploit** | – | – | arxiv.org/abs/2607.13006 | ABSTRACT |
| **When Do Foundation Models Pay Off? Break-even analysis** (Tan Jerome, Simon) | 2026 | Preprint | Preprint | When do FMs beat Naive, ETS, ARIMA and XGBoost? | 30 Monash/M-style datasets | Six **dataset-level** features; break-even over training fraction | Tables and plots | Leave-one-out decision rule | FMs win unconditionally on 15 of 30. The rule "n < 700 and S ≥ 0.05, use zero-shot" resolves 10 of 30. | "Reliable automated prediction at this benchmark scale remains an open problem"; needs 50+ datasets | Y (`nicolaisi/fm_breakeven`; the URL in the paper uses a hyphen and returns 404) | arxiv.org/abs/2607.04919 | FULL TEXT (digest) |
| **Causal Analysis for TSFMs** (Jander, van Heeswijk, Mes) | 2026 (Aug) | Preprint | Preprint | Pre-deployment detection of TSFM biases | Synthetic only (6 generators) | Interventions on generator parameters, ceteris paribus | Trajectory and scatter plots | Chronos-2 and TimesFM-2.5 | Both **overestimate persistence** (AR(1), Hurst). **Regime switch**: Chronos-2 fails at τ ≥ 50, TimesFM-2.5 at τ ≥ 25. TimesFM-2.5 smooths the energy-release pattern. | Fixed 200-step window. Calls for **"in vivo validation of our findings through real-world datasets."** | Not checked | arxiv.org/abs/2608.24303 | FULL TEXT (digest) |
| **Context parroting** (Zhang, Gilpin) | 2025 | ICLR 2026 | Peer-reviewed | Do TSFMs just copy the context? | Dynamical systems, ECG | Parroting baseline | – | – | Parroting beats TSFMs; shared failure "converging to the mean" | – | – | arxiv.org/abs/2505.11349 | ABSTRACT |
| **Zero-shot TSFMs on cloud data** (Toner et al.) | 2025 | ICLR 2025 ICBINB workshop | Workshop | Do TSFMs work on cloud telemetry? | Cloud series | – | Failure plots | vs linear baselines | "Widespread failure", erratic forecasts | – | – | arxiv.org/abs/2502.12944 | ABSTRACT |
| **How Foundational are FMs for TSF?** (Karaouli et al.) | 2025 | NeurIPS 2025 BERT2S workshop | Workshop | Zero-shot dependence on the pretraining domain | – | – | – | – | Zero-shot ability is tied to the pretraining domains | – | – | arxiv.org/abs/2510.00742 | ABSTRACT |
| **Evaluation Choices Decide the Forecasting Leaderboard** (Islam, Mohammed) | 2026 | Preprint | Preprint | Do evaluation choices flip rankings? | Private panel plus M5 | Vary only the unit of analysis, pooling, interval scoring | – | 25 methods incl. 6 TSFMs | Changing the unit of analysis moves the baseline from rank 2 to rank 23; interval-vs-point rank ρ = 0.02 | – | Ancillary files | arxiv.org/abs/2609.27867 | ABSTRACT |
| **Synapse** (Das et al., Google) | 2025 | Preprint | Preprint | Arbitration across TSFMs | GIFT-Eval-style | Context-dependent weighting of TSFMs | – | – | TSFMs have "specialized performance profiles"; arbitration beats ensembles | – | – | arxiv.org/abs/2511.05460 | ABSTRACT |
| **Dissecting Chronos (SAEs)** (Mishra) | 2026 | ICLR 2026 TSALM workshop | Workshop | Mechanistic interpretability of Chronos-T5 | – | TopK sparse autoencoders (SAEs), 392 ablations | – | ΔCRPS | Mid-encoder change-detection features are the most causally critical | – | – | arxiv.org/abs/2603.10071 | ABSTRACT |
| **Universal Redundancies in TSFMs** (Bao, …, Gilpin) | 2026 | Preprint | Preprint | Component redundancy; the heads behind parroting and seasonality bias | – | Ablations, direct logit attribution | – | – | Specific heads cause parroting and seasonality bias | – | – | arxiv.org/abs/2602.01605 | ABSTRACT |
| **On the Internal Semantics of TSFMs** | 2025 | Preprint | Preprint | Which layers encode which concepts | Synthetic | Linear probes | – | – | Early layers encode AR(1), level shifts, trend. Composition degrades probes. | – | – | arxiv.org/abs/2511.15324 | ABSTRACT |
| **Non-stationarity in TSFM embedding space** (Choi, Shook, Dubrawski) | 2026 | Preprint | Preprint | Can embeddings detect mean, variance and trend shifts? | Synthetic | Probing sweeps | – | – | "Distinct, model-specific failure modes" | – | – | arxiv.org/abs/2604.16428 | ABSTRACT |
| **Who should I trust? VA for comparing net-load models** / **Forte** | 2024–25 | IEEE PES Grid Edge 2025 / ISGT NA 2024 | Peer-reviewed (power-systems venues) | Domain VA for comparing forecasters | Net load (not open?) | Coordinated views by solar penetration and hour | Yes | Case studies | – | Domain-specific; not TSFMs | – | arxiv.org/abs/2407.21299, 2311.06413 | ABSTRACT |
| **MSPT-vis** (Hao, Chen, Lv, …) | 2026 | J. Visualization 29(1) | Peer-reviewed | VA of a custom multi-scale patch transformer | – | Attention and patch views | Yes | – | – | Custom model, not a TSFM | – | doi 10.1007/s12650-025-01092-3 | SECONDHAND |
| **Probe-Driven VA for Diagnosing … Air Quality Forecasting Models** (Lin et al.) | 2026 | IEEE VIS 2026 (accepted list) | Peer-reviewed | Probing physical knowledge in a deep-learning air-quality forecaster | – | – | – | – | – | Domain deep-learning model, not a TSFM (SPECULATION from the title) | – | ieeevis.org 2026 papers list | SECONDHAND (title and authors only) |
| **Decoding Latent Spaces of TSFMs for VA** (Santamaria-Valenzuela et al.) | 2025 | Under review, IJIMAI | Preprint | Are MOMENT embeddings interpretable for VA? | 5 datasets | Projection of embeddings | Scatter plots | – | "Limited improvement in interpretability" | Needs better projections | – | arxiv.org/abs/2504.20099 | ABSTRACT |

---

## 3. Falsification log: what I searched, and what returned nothing

**Queries that returned nothing relevant.** These are arXiv API unless noted.
- `abs:"foundation model" AND abs:forecast* AND abs:"visual analytics"`: **0 results**.
- `abs:"slice discovery" AND abs:forecast*`: **0**.
- `abs:"subgroup discovery" AND forecast* AND "time series"`: **0**.
- `abs:"instance space analysis" AND forecast*`: **0**. The web search `"instance space" forecasting foundation models Chronos TimesFM` also found nothing.
- `abs:"meta-learning" AND tsfeatures AND foundation`: **0**.
- `abs:"foundation model*" AND forecast* AND disagreement`: **0**.
- `abs:"time series foundation" AND "when to trust"`: **0**.
- `abs:"time series foundation" AND "context length" AND sensitiv*`: **0**. Context-length effects appear only as sub-results inside benchmarks (Simeone 2602.10848; Michael et al. 2609.25788).
- `abs:"time series foundation" AND synthetic AND "stress test*"`: **0**.
- `abs:"zero-shot" AND forecast* AND leaderboard* AND aggregat*`: **0**.
- `cat:cs.HC AND "foundation model*" AND "time series"`: only health-sensing or LLM papers, no forecaster diagnosis.
- GitHub search: `"time series foundation model visual analytics"` gave 0; `"tsfm failure analysis"` gave 0.
- IEEE VIS paper JSON for **2024 (436 items) and 2025 (309 items)**, grepped for forecast, time series and foundation model: **no forecasting-model VA papers**.
- The VIS **2026** accepted list contains one domain paper, the air-quality probe VA. Otherwise there are forecast-*communication* papers such as "Seeing Through the Forecast Clutter".
- TVCG in OpenAlex (2023+) with "forecast" in the title: only TimeTuner is model-VA.

**Queries that did falsify parts of the RQ.**
- `abs:"GIFT-Eval"` led to **TIME** (pattern-level, feature-stratified TSFM evaluation) and **Spectral Predictability**.
- `abs:"time series foundation" AND mechanistic` led to **Break-even analysis** (dataset features vs FM advantage).
- Semantic Scholar citations of TimeTuner (12 citers) show no TSFM VA tool. EXPRTS has 0 citers.
- Semantic Scholar citations of TIME (19 citers, all ML papers) show none building a VA or error-regime tool on TIME outputs. FACT as of today.
- A web search for Nonato/Silva forecasting VA surfaced **mTSeer**.

SYNTHESIS: the *stratification* idea is saturated. So is *instance space plus linked views for forecasters*. The *foundation-model* part of the ML literature is dense, with weekly benchmark papers. The **intersection** is empty in what I could find: window-level failure regimes, cross-TSFM disagreement, held-out validation of discovered regimes, and linking to synthetic causal findings. I have moderate confidence in that; the literature moves weekly.

---

## 4. Reproduction targets: verified

| Asset | Status | License | Size / compute | Notes |
|---|---|---|---|---|
| `Real-TSF/TIME-Output` (HF dataset) | FACT: public, not gated. Last modified **2026-09-28** (actively updated). | Apache-2.0 (dataset card) | 2.89 GB total. `results/{model}/{dataset}/{freq}/{term}/` holds `metrics.npz` (per series × window × variate: MSE, MAE, RMSE, MAPE, sMAPE, MASE, ND, CRPS), `predictions.npz` (9 quantiles, float16) and `config.json`. `features/{ds}/{freq}/test.csv` or `full.csv` hold 30+ tsfeatures per variate. | **30 models**: Chronos-2, Chronos-bolt, TimesFM-1.0/2.0/2.5/3, Moirai_base, Moirai2, TiRex, Toto and five Toto-2.0 sizes, sundial_base, Kairos, visiontspp, Timer-S1, Tabby, t0_beta and others. The **only classical baseline is seasonal_naive** (FACT). Metrics for one model over all 98 tasks total about 7 MB. |
| `Real-TSF/TIME` (raw data) | FACT: public | **CC-BY-NC-4.0** (fine for coursework; do not redistribute commercially) | 220 MB | Needed only to recompute context-window features and to run statsforecast baselines. |
| `zqiao11/TIME` code | FACT: 71 stars, pushed 2026-09-23 | GitHub license null; README badge says Apache-2.0 (inconsistent) | – | **Stale pins**: `numpy~=1.26.0`, `gluonts~=0.15.1`, `datasets~=2.17.1`, `scipy~=1.11.3`, Python 3.11. Use a separate conda env. |
| TIME leaderboard space | FACT: Docker/Gradio, `src/tab.py` (1153 lines) | Apache-2.0 | – | The per-pattern tab is **7 radio buttons (feature high/low) leading to a leaderboard table**. The per-window tab is dropdowns leading to one forecast plot. **No scatter, no brushing, no linking, no disagreement view** (FACT, from the code). |
| `amazon-science/chronos-forecasting` | FACT: 5.9 k stars, pushed 2026-09-17 | Apache-2.0 | Chronos-2 is 120 M params (478 MB), `chronos-2-small` 28 M, Bolt-small 48 M (191 MB), Bolt-base 205 M (821 MB) | Loose pins: `torch>=2.2,<3`, `transformers>=4.41,<6`. CPU works. The README shows `device_map="cuda"`, and MPS is untested by me (unverified). |
| `google-research/timesfm` | FACT: 33.9 k stars, v3.0.2 | Code and weights ≤ 2.5 are Apache-2.0. **TimesFM 3.0 weights are non-commercial** (FACT, README). | 2.5-200M is 925 MB; 2.0-500M is 3.99 GB | **An MLX backend exists for Apple silicon** (`pip install timesfm[mlx]`, FACT), which suits the M1 Pro. |
| `SalesforceAIResearch/uni2ts` (Moirai) | FACT: pushed 2026-06-02 | Code Apache-2.0; `moirai-2.0-R-small` weights **CC-BY-NC-4.0** | 11 M params (46 MB) | – |
| TiRex (`NX-AI/tirex`) | FACT | "other" (NXAI community license) | 35 M (141 MB) | Read the license before use. |
| `SalesforceAIResearch/gift-eval` | FACT: pushed today | Apache-2.0; data on HF (1.59 GB) | – | Stale pins, the same as TIME. Public results are per-config aggregates only, so it is less useful for instance-level work. |
| `autogluon/fev` | FACT | Apache-2.0 | Data 0.66 GB | Summary tables only. |
| `Nixtla/statsforecast` | FACT | Apache-2.0 | CPU, fast | For AutoETS, AutoARIMA and AutoTheta, which TIME lacks. |
| `Nixtla/tsfeatures` (Apache-2.0); `pycatch22` (**GPL-3.0**) | FACT | – | CPU | catch22's GPL license matters only if you redistribute a combined tool. |

**Can the student run the models?** On an M1 Pro (CPU or MPS) or one 11 GB GPU, Chronos-Bolt-small/base, Chronos-2, TimesFM-2.5-200M, Moirai-2-small and TiRex are all feasible for zero-shot inference.
- SYNTHESIS: weights are ≤ 1 GB, which is well under 11 GB.
- AUTHOR CLAIM: Simeone (2602.10848) ran Chronos-Bolt, Chronos-2 and Moirai-2 on a Ryzen CPU with 16 GB and no GPU.
- TimesFM-2.0-500M is 4 GB of weights. It is fine on the GPU box and probably slow on CPU (SPECULATION).

**Can a published result be reproduced in a few hours?** Yes, two ways (SYNTHESIS):
- (a) Recompute TIME's pattern-level numbers for 7 features × 2 levels from the released features and metrics, and compare them with the paper's figures. This is pure pandas, about 3 h.
- (b) Rerun Chronos-2 and seasonal naive on 2–3 small TIME tasks, and check that the per-window MASE and CRPS match the saved `metrics.npz`. About 3–4 h, most of it environment setup against the stale pins.

EXPRTS and TimeTuner are poor reproduction targets:
- Neither has a license.
- Their pins are stale: py3.9/numpy 1.20/bokeh 2.3; TF 2.10 + Vue 3.
- TimeTuner needs a JS frontend, which is the student's weak spot.

---

## 5. Pilot: does the "leaderboards hide failures" premise hold? (SYNTHESIS, my own numbers)

**Setup.**
- TIME-Output, the **50 short-horizon tasks**.
- `metrics.npz` for seasonal_naive, Chronos-2, TimesFM-2.5, Moirai2 and TiRex. That is 6,633 series-variates and 95,711 windows, 31 MB.
- The unit is the MASE ratio (TSFM / seasonal naive).

**Caveat.** Series and variate rows were joined to feature rows by assuming the orders match: `len(features) == S × V` held for 49 of 50 datasets. **I have not verified this ordering in TIME's code.** It is check #2 in §11.

> **Correction 2026-09-29 (`spike_results/A2_time_tsfm.md`):** the positional ordering was **wrong** for 34 of 50 datasets (88% of series-variates). The two feature-ρ rows below ("Largest single-feature Spearman ρ …") are therefore **void**. With the correct name join: seasonal_strength ρ = −0.21, x_entropy +0.21, length −0.11. The window-level variance and disagreement rows do not use features and are unchanged (0.662, 31.8%).

| Quantity | Value |
|---|---|
| Median relMASE vs seasonal naive: Chronos-2 / TimesFM-2.5 / Moirai2 / TiRex | 0.663 / 0.677 / 0.692 / 0.690 |
| Share of series where the model is worse than seasonal naive | 1.2 % / 1.6 % / 3.0 % / 2.3 % |
| Share of series where **all 4** are worse than seasonal naive | **0.5 %** |
| Mean share of windows per series where all 4 lose to seasonal naive | **3.8 %** |
| Series with Chronos-2 MASE > 1 / all four MASE > 1 | 10.5 % / 9.2 %. When one model fails, **almost all do**, which fits Jander et al.'s "concentration risk". |
| Share of variance of log relMASE (Chronos-2) **within datasets**, series level | **0.86** |
| Share of variance of window-level log relMASE **within a series** (across windows) / within a task | **0.66** / 0.92 |
| Window-level abs(log(Chronos-2 / TimesFM-2.5)): median / 90th percentile; share of windows differing by more than 25 % | 0.116 / 0.564; **31.8 %** |
| Series-level "best TSFM" share | Chronos-2 48 %, TimesFM-2.5 22 %, TiRex 16 %, Moirai2 15 %. Chronos-2 wins at task level in 37 of 50 tasks. |
| Largest single-feature Spearman ρ with Chronos-2 relMASE | length −0.26, missing_rate +0.15. Seasonal strength, trend, entropy and spikiness all have abs(ρ) < 0.08. |
| Largest ρ with cross-model disagreement | missing_rate −0.20, length −0.20, x_entropy −0.19, seasonal_corr +0.18 |

**What this implies.** SYNTHESIS, conditional on the ordering check.
1. "TSFMs lose to *seasonal naive*" is too rare to be the headline. The informative targets are:
   - (i) losing to a strong *classical* baseline (AutoETS, AutoARIMA, Theta), which TIME does not include, so it is a real extension;
   - (ii) absolute failure (MASE > 1, CRPS-based skill < 0);
   - (iii) disagreement between TSFMs.
2. Most of the variation sits **below** the granularity of every existing stratification. GIFT-Eval, fev, break-even and spectral predictability work at dataset level. TIME works at the level of a static variate. A window-level view is scientifically motivated, not decorative.
3. Static per-variate features explain very little on their own. That supports TIME's own call for "joint patterns". It also raises the risk in §9. Window-context features, such as a level shift near the forecast origin, may explain more; this is untested.
4. **Noise-floor warning.** Part of the within-series variance may be MASE-denominator noise. Use CRPS as well, and use a same-family control (Chronos-2 vs Chronos-Bolt disagreement) as a noise floor.

---

## 6. Gap verdict: **partly addressed**

| Sub-claim of the original RQ | Status | Evidence |
|---|---|---|
| Stratify TSFM error by interpretable series characteristics | **Already done** | TIME (pattern-level, 7 features, 12+ TSFMs); spectral predictability (Ω, GIFT-Eval); break-even (6 dataset features). All FACT. |
| Linked-view VA relating per-series error to features | **Done for pre-TSFM forecasters** | EXPRTS (instance space + error + drilldown, 10-user study); mTSeer (instance-level model comparison, Silva lab); Kang et al. 2017 (instance spaces) |
| An interactive TSFM leaderboard with drilldown | **Partly done** | TIME leaderboard: filter-then-table plus a single-window plot. Not linked, and no multi-model or disagreement view (FACT, code). |
| **Window-level, context-conditioned** failure regimes across TSFMs | **Open** (no hits) | §3 queries; the pilot shows this is where most of the variance lives |
| **Cross-TSFM disagreement** as a first-class VA object | **Open in VA**; ML-side routing and ensembles exist | Synapse, portfolios and TimeRouter use disagreement for arbitration, not diagnosis |
| **Held-out validation** that discovered regimes generalize | **Open** | No TSFM paper validates slices on held-out datasets or domains. Break-even flags this as open: "reliable automated prediction… remains an open problem". |
| **In-vivo test of synthetic causal failure modes** (persistence bias, regime-switch collapse) | **Open, and explicitly requested** | Jander et al. (Aug 2026): "in vivo validation of our findings through real-world datasets" |
| Context-window what-if probing of TSFMs in a VA tool | **Open for TSFMs**; EXPRTS did transformation what-ifs for deep-learning models | EXPRTS; Simeone (context-length sweep, no VA) |

---

## 7. Refined research question and hypotheses

**RQ (refined).** Where do current zero-shot TSFMs fail, at the level of individual forecast windows, relative to strong classical baselines and to each other?
- Are those failures predictable from interpretable **context-window** characteristics better than from TIME-style static variate patterns?
- Do the failure modes found by synthetic causal analysis (persistence overestimation, regime-switch collapse) explain a measurable share of real-benchmark failures?

A linked-view VA tool is the instrument for discovering candidate failure regimes. Held-out statistical validation is the test.

- **H1 (hidden heterogeneity).** More than 50 % of the variance in window-level relative error (TSFM vs best classical baseline) lies within series. Pilot: 0.66 vs seasonal naive. It is replicated with CRPS and with an AutoETS reference. *Falsified if* the share is below 0.3 once noise-floor-corrected.
- **H2 (context beats static patterns).** A shallow model predicting window failure has higher AUROC from context-window features (catch22 or tsfeatures on the last L points, plus changepoint and level-shift distance to the origin, spike-near-origin, context length) than from:
  - TIME's 7 binary variate codes; and
  - dataset identity.

  Validation is leave-dataset-out. *Falsified if* ΔAUROC < 0.03, or AUROC < 0.6. In that case, report a negative result consistent with *The Spectrum Is Not Enough*.
- **H3 (shared vs model-specific failures).** Most failure windows are shared across ≥ 3 of 4 TSFMs (pilot: about 88 % conditional overlap for MASE > 1). A minority are model-specific and cluster in identifiable regimes, for example long-horizon and high-frequency tasks for decoder models, as GIFT-Eval reported.
- **H4 (in vivo).** Windows whose context contains a regime switch within τ steps of the origin show larger relative error for TimesFM-2.5 than for Chronos-2 when 25 ≤ τ < 50. Jander et al. predict TimesFM breaks at τ ≥ 25 and Chronos-2 at τ ≥ 50. Windows with high estimated persistence show positive bias in forecast persistence (the ACF of the forecast median exceeds the ACF of the realized path). *Falsified if* the effect confidence intervals include 0.
- **H5 (VA-found regimes generalize).** Regimes found interactively, or by a slice-finder, on 50 % of datasets keep a failure-rate lift of ≥ 1.5× on the held-out 50 %. They should do so more often than equal-size random slices and TIME's single-feature median splits.

---

## 8. Project design

### 8.1 Minimum viable project (grade-safe; no TSFM inference needed)

- **Data.** TIME-Output, the **50 short-horizon tasks**.
  - 5 TSFMs: Chronos-2, Chronos-bolt, TimesFM-2.5, Moirai2, TiRex. Toto is optional.
  - Plus seasonal_naive.
  - Metrics and predictions come to roughly 50–150 MB. I measured about 31 MB for metrics of 5 models; predictions are about 3–5× larger (SPECULATION from one file).
  - Raw TIME data, 220 MB, for context features and classical baselines.
- **Reproduction.**
  - (R1) Recompute TIME's pattern-level results for the 7 features and match the paper and leaderboard.
  - (R2) Rerun Chronos-2 and seasonal naive locally on 2–3 small tasks, for example `CPHL/H/short` (4 series × 28 windows). Match the saved per-window MASE and CRPS.
- **Extension.**
  1. Add **AutoETS, AutoARIMA and AutoTheta** via statsforecast on the same windows, on CPU.
  2. Compute **context-window features** per window.
  3. Define failure labels: relMASE vs best classical > 1, MASE > 1, and CRPS skill < 0.
  4. Build a **Streamlit + Plotly linked-view tool** (§8.3).
  5. Run a **decision-tree or SliceLine-style slice finder** with leave-dataset-out validation (H2, H5).
- **Deliverable claims.**
  - Where TSFMs fail relative to classical methods.
  - That the failures are mostly window-level.
  - Which context regimes predict them, or an honest null result.

### 8.2 Stronger version (for the paper)

1. **H4 in-vivo test.**
   - Detect regime switches and persistence in each context (ruptures/PELT, and a Hurst or AR(1) estimate).
   - Estimate a dose-response curve of error vs τ, per model.
   - Include a synthetic-to-real bridge: generate Jander-style series and place them in the same feature space, which is Kang et al.'s idea of filling the instance space.
2. **Context what-if probe.** For brushed windows, rerun Chronos-2 and TimesFM-2.5 with:
   - context truncated to {64, 128, 256, 512, 1024, 2048};
   - or the pre-shift segment removed.

   Show a sensitivity curve. This is cheap: tens of windows × 6 lengths × 2 small models.
3. Replicate on **GIFT-Eval**: run inference yourself on 5–10 configs, since its public results are aggregate only.
4. A small **expert study** (4–6 forecasting practitioners) with pre-registered tasks: "find a regime where the leaderboard order flips".

### 8.3 Visualization: views and why each is necessary

1. **Failure-regime map.** A 2D UMAP or PCA of window-context features, coloured by relative error or disagreement, with lasso brushing.
   - Why: failure appears multivariate and window-level. TIME's own limitation is single-feature analysis. EXPRTS users struggled to read PCA axes, so pair the map with view 2 rather than relying on it alone.
2. **Conditional error small multiples.** Binned relative error vs each feature, one line per model, with bootstrap CIs. Shown for the brushed selection vs everything.
   - Why: this makes view 1 interpretable, and shows *which* feature drives the regime.
3. **Leaderboard-flip matrix.** Pairwise win rates of the models inside the selection vs globally.
   - Why: this directly tests the "aggregates hide it" claim by showing rank reversals inside a regime.
4. **Window drilldown.** The context, the realized future, and quantile fans for all TSFMs plus AutoETS and seasonal naive.
   - Why: to verify the mechanism. Examples are flat forecasts, parroting, a missed level shift, or mean reversion. TIME itself notes that flat forecasts can score competitive MASE, so metrics alone mislead.
5. **Slice table.** Rules produced by the slice finder or the user, with support, lift, CI, and **held-out lift**.
   - Why: this turns the VA into a falsifiable claim (H5).

### 8.4 Evaluation

- **Metrics.** MASE and CRPS per window (saved by TIME; computed for the classical baselines). Relative error vs the best classical method. AUROC and PR-AUC for failure prediction. Slice lift with bootstrap 95 % CIs.
- **Baselines for "finding regimes".**
  - Dataset or domain grouping, as in the GIFT-Eval and fev style.
  - TIME's binary pattern codes.
  - The best single-feature threshold.
  - Random slices of equal size.
- **Controls.**
  - Compute features **only on the context**. TIME's `test.csv` features may include the target region, which is a leakage risk (SYNTHESIS).
  - A same-family noise floor: Chronos-2 vs Chronos-Bolt.
  - Leave-dataset-out or leave-domain-out splits.
  - Holm correction across slices.
- **Success criteria.**
  - Course: R1 matches the TIME tables within about 2 %. R2 median per-window difference is below 1 %. The tool works and is demoed with 3 case studies.
  - Research: H1 and H2 confirmed, plus at least 2 regimes with held-out lift ≥ 1.5× whose CI excludes 1 (H5), or a clean negative result on H2/H4.

### 8.5 Hour budget (about 40 h)

| Task | Hours |
|---|---|
| Proposal: related work (§2 is most of it), design, 4 pages | 3 |
| Data pulls (TIME-Output subset, raw TIME), env for TIME code (stale pins) | 3 |
| R1: recompute pattern-level results | 3 |
| R2: local Chronos-2 and seasonal naive rerun, matching `metrics.npz` | 3.5 |
| statsforecast baselines on the 50 short tasks | 2.5 |
| Window-context features (tsfeatures/catch22, changepoints) | 3.5 |
| Failure labels, variance decomposition (H1), failure predictor (H2) | 3 |
| Streamlit + Plotly linked views (5 views) | 6 |
| Slice finder with held-out validation (H5); H3 overlap analysis | 3 |
| H4 in-vivo regime-switch test (a minimal version) | 2.5 |
| Case studies and figures | 2 |
| Nov 3 update (1 page) | 0.5 |
| Final 8-page report and presentation or demo | 5.5 |
| **Total** | **41** |

If the budget is tight, cut H4 to the stronger version. The MVP stays intact.

---

## 9. Failure modes and mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| **Features explain little.** The pilot's single-feature abs(ρ) is ≤ 0.26, and the *Spectrum Is Not Enough* impossibility result applies. | **High** | Use window-context and non-spectral features (level shifts, spikes near origin, missingness, length). Use joint rules. Pre-register H2's null. A negative result is still reportable and ties directly to 2026 theory. The disagreement views keep the tool useful either way. |
| Assumed series/variate-to-feature row ordering is wrong | Medium | Read TIME's `saver` and feature scripts (1 h). Otherwise recompute features yourself from raw data. This is required anyway, for context-only features. |
| Per-window MASE noise inflates the within-series variance | Medium | Use CRPS as well, the same-family noise floor, and aggregation over horizon steps. |
| Scooped by the TIME authors (TIME-Output was updated today) or by a benchmark paper that adds window-level analysis | Medium | Stress what they are unlikely to do: VA evaluation, held-out regime validation, and the in-vivo causal test. Put a preprint up soon after December. |
| Stale pins in TIME / GIFT-Eval conflict with Chronos or TimesFM | Medium | Use separate conda envs. The MVP only needs numpy/pandas on the released outputs. |
| Instructor sees it as "mTSeer again" | Low–medium | Frame it explicitly as mTSeer's future work (more models, trend-type features, expanded instance-level evaluation) at TSFM scale. Cite it prominently. |
| License issues (TIME data CC-BY-NC, Moirai-2 CC-BY-NC, TimesFM-3 non-commercial, catch22 GPL) | Low for coursework | Do not redistribute data or weights. Keep the tool code MIT or Apache and call catch22 as a dependency. |
| Front-end burden | Low | Streamlit plus Plotly selection events are enough, no D3 needed. `streamlit-plotly-events` or Plotly's `on_select` are unverified for lasso in the current Streamlit version; test it in week 1. |

---

## 10. Publication path

- **Venues.**
  - VIS short paper, or the VIS workshops (TREX, VISxAI), for the VA framing.
  - A time-series foundation-model workshop, for the in-vivo and regime findings. ICLR TSALM ran in 2026, and NeurIPS had BERT2S in 2025. Whether they recur in 2027 is SPECULATION.
  - IJF or ISF, if it is framed as instance-space analysis for TSFMs.
- **What must be added after the course.**
  - A second benchmark (GIFT-Eval with own inference, or fev).
  - The full H4 dose-response with a synthetic bridge.
  - The what-if context probe.
  - A small expert study (4–6 people).
  - A comparison of slice-finding methods.
  - Release the tool plus the derived window-level table.

---

## 11. Novelty confidence (in words)

**Low** for "a linked-view tool relating per-series error to tsfeatures". Kang 2017, EXPRTS, mTSeer and the TIME leaderboard cover it. **Medium** for the refined RQ.

**Why medium.** Three things are separately and specifically unaddressed in what I could find:
- (a) window-level, context-conditioned failure regimes;
- (b) TSFM disagreement as a diagnostic object;
- (c) held-out validation of discovered regimes.

My pilot gives a quantitative reason these matter: most of the variance is below dataset and variate granularity. The **in-vivo test** is new and was explicitly requested by an August 2026 paper, which is the kind of "named open question" the brief asks for.

**Why not high.**
- The field publishes benchmark papers weekly.
- TIME already saves the window-level outputs and could add such a view.
- ML-side routing papers (Synapse, portfolios, TimeRouter) already exploit context-dependent model differences, though not for diagnosis.
- My falsification of VIS 2026, and of 2026 workshops in general, is incomplete.

---

## 12. 12b-style row

| Criterion | Rating | One-line reason |
|---|---|---|
| Course fit | **High** | Hits model assessment (Sept 22), black-box interpretation (Oct 6), clustering and DR (Oct 13/20) and time series (Nov 17). The reproduce-then-extend structure maps cleanly to TIME. |
| Research upside | **Medium** | The window-level, disagreement and in-vivo angle is open and topical. The stratification half is taken, so the upside depends on H2/H4 results. |
| Technical risk | **Low–medium** | No inference is needed for the MVP. Small models run on CPU or MLX. The main risk is stale pins in the benchmark code. |
| Data risk | **Low** | All data is open on HF (TIME CC-BY-NC, GIFT-Eval Apache). Sizes are ≤ 3 GB, and a 150 MB subset is enough. |
| Visualization burden | **Low–medium** | 5 Plotly views in Streamlit, with no D3. Lasso-linking needs a week-1 spike test. |
| Evaluation clarity | **High** | Quantitative, falsifiable criteria: variance share, leave-dataset-out AUROC, held-out slice lift, dose-response CIs. It does not rest on subjective "insight". |
| Best suited if… | – | …the student wants a low-risk, data-ready project with a real chance of a clean workshop result, and accepts that the headline may be a well-documented null result on interpretability. |

---

## 13. Verify-this-yourself checks

1. **Open the TIME leaderboard** (huggingface.co/spaces/Real-TSF/TIME-leaderboard). Confirm the per-pattern tab is binary radio filters leading to a table, with no brushing or linked scatter. The claim is based on `src/tab.py`.
2. **Verify the row ordering.** In `zqiao11/TIME`, read `src/timebench/evaluation/saver.py` and the feature script. Confirm that the series and variate indices in `metrics.npz` align with the rows of `features/*.csv`. Then rerun the §5 pilot with CRPS as well as MASE.
3. **Check the EXPRTS venue.** Look at the Gundersen / NTNU publication page for a 2025–2026 published version; I found only arXiv "under review". Confirm the mTSeer framing with Prof. Silva, and ask whether Ke Xu or Jun Yuan have follow-ups.
4. **Weekly scoop check until the Oct 20 proposal.**
   - arXiv: `abs:"time series foundation" AND (abs:"window-level" OR abs:"instance-level" OR abs:"failure regime")`.
   - Semantic Scholar citations of 2602.12147 (TIME) and 2608.24303 (Causal Analysis).
5. **Test CPU/MPS inference on the M1.** Time Chronos-2 on the M1 with `device_map="cpu"` and `"mps"`, and TimesFM-2.5 with `timesfm[mlx]`, on 100 windows before promising the what-if probe.

---

### Appendix: session notes

- The broad scan's statement "TimeTuner code openness unverified" is now resolved: `CatherineHao/TimeTuner` exists, with 22 stars, no license, TF 2.10 and Vue 3 (FACT).
- The broad scan's statement "no instructor-lab overlap" is **corrected**: see mTSeer above.
- EXPRTS is still a preprint. Its own README BibTeX is `@misc … 2025`.
- The break-even paper's code URL `github.com/nicolaisi/fm-breakeven` returns 404. The actual repo is `nicolaisi/fm_breakeven` (FACT).
- Pilot data and scripts are in the session scratchpad only and are not committed. They are reproducible in about 20 minutes from the TIME-Output URLs in §4.
