# Finding a Defensible Research Gap: Soccer × Visualization for ML

**Prepared for:** solo project, NYU DS-GA 3001 Visualization for Machine Learning (Fall 2026, Prof. Claudio Silva)
**Compiled:** 2026-09-27, by Claude (Opus 5.5) acting as lead reviewer, with six parallel literature sub-agents
**Supporting notes:** `lit_notes/01`–`06` (full per-slice tables, search queries, and caveats; roughly 27k words)

> **How to read this document.** Every substantive claim is tagged with one of four labels:
> - **FACT**: verified in a primary source (paper, dataset, repo).
> - **AUTHOR CLAIM**: something a paper asserts that was not independently checked.
> - **SYNTHESIS**: my conclusion from comparing several sources.
> - **SPECULATION**: a plausible idea without enough evidence behind it.
>
> Evidence levels are also marked where it matters:
> - **FULL TEXT**: the relevant sections were read.
> - **ABSTRACT**: abstract or landing page only.
> - **SECONDHAND**: from a snippet or from another paper's description.
>
> Absence claims ("nobody has done X") are always SYNTHESIS with a coverage caveat. See §0.3.

---

## Table of contents

0. [Scope, method, and limits of this review](#0-scope-method-and-limits-of-this-review)
1. [Executive summary: the uncomfortable answers](#1-executive-summary-the-uncomfortable-answers)
2. [Course context (verified)](#2-course-context-verified)
3. [Consolidated literature table](#3-consolidated-literature-table)
4. [Deep dives on the papers that decide the question](#4-deep-dives-on-the-papers-that-decide-the-question)
5. [Verdicts on Directions A, B, C](#5-verdicts-on-directions-a-b-c)
6. [Verdicts on Question families 1–7](#6-verdicts-on-question-families-17)
7. [Gap synthesis table](#7-gap-synthesis-table)
8. [Data: what you can actually download](#8-data-what-you-can-actually-download)
9. [What transfers from Silva's sports visual analytics](#9-what-transfers-from-silvas-sports-visual-analytics)
10. [Tempting but bad choices for this project](#10-tempting-but-bad-choices-for-this-project)
11. [Final shortlist: four candidate research questions](#11-final-shortlist-four-candidate-research-questions)
12. [Final comparison table](#12-final-comparison-table)
13. [If I were independently doing this literature review, here is exactly what I should verify](#13-if-i-were-independently-doing-this-literature-review-here-is-exactly-what-i-should-verify)

---

## 0. Scope, method, and limits of this review

### 0.1 Method
The review ran six parallel search slices:
1. Soccer XAI (explainable AI for soccer models)
2. LLM and natural-language explanations
3. General XAI evaluation in ML and visualization research
4. Tactical representations and the reliability of dimensionality reduction (DR)
5. Open datasets and tools
6. Sports-analytics methodology, Silva's work, and soccer visual analytics (VA)

Each slice used web search, arXiv, the Semantic Scholar, OpenAlex, Crossref and Europe PMC APIs, and GitHub/PyPI/figshare APIs, plus PDF downloads with `pdftotext` and `grep`. I then **independently re-checked the load-bearing claims** myself:
- MOUNTAINEER authorship
- Visagreement venue
- The flipped-sign experiment in Ichmoukhamedov et al.
- The Rahimian et al. wordalisation paper
- Jeon et al. "Stop Misusing t-SNE and UMAP"
- Tang et al. 2023
- Peters et al. 2026 (via Crossref)
- Tsai et al. 2026
- PassAI's authors and venue (one sub-agent had them wrong; corrected)
- Repo existence and licenses for 9 code bases

In total, about **120 distinct sources** were touched. About **35 are load-bearing** and appear in §3.

### 0.2 Coverage
- 2022–2026, with emphasis on 2024–2026. Older foundational work is included only when it defines a method still in use (e.g., the disagreement metrics, Distill t-SNE).

### 0.3 Known limits
Treat "open gap" claims with the skepticism below.
- **The Semantic Scholar API was rate-limited (HTTP 429) for most of the session**, so citation-graph sweeps ("everything citing X") are incomplete. OpenAlex and arXiv were used as fallbacks.
- **The session's web-search budget (200 calls) was exhausted** near the end. A few late checks used direct page fetches instead.
- **Paywalled and unread**, all potentially important:
  - Cefis & Carpita 2024 (*Statistics*)
  - Anzer & Bauer 2022 (*DMKD*)
  - Kranzinger et al. 2025 (full text)
  - The Visagreement TVCG full text (read via the author's IJCAI-DC summary)
  - The journal version of Rahimian et al. 2025 (SAGE blocked; the arXiv version was read in full)
- **Workshop proceedings with poor indexing were not swept item by item:** MLSA @ ECML-PKDD 2022–2025, the StatsBomb Conference, CVsports, and MIT Sloan. This is the single most likely place for a "gap" in this document to turn out to be already filled. See the §13 checklist.

---

## 1. Executive summary: the uncomfortable answers

1. **Your instructor's own lab has already published the obvious version of Direction A.**
   - FACT: Claudio Silva co-authored these tools for comparing and evaluating feature-attribution explanations on tabular data:
     - MOUNTAINEER (IEEE TVCG; arXiv 2406.15613)
     - Visagreement (IEEE TVCG 31(10), 2025)
     - GALE (ICML 2022 workshop)
     - A visualization-driven attribution-method selection system (Elsevier *Information Systems*, 2025)
     - SUBPLEX (IEEE CG&A 2022)
   - So "compare SHAP vs LIME vs IG on a soccer model in a dashboard" is, to this grader, *their tool applied to a new dataset*.
   - SYNTHESIS: that is not fatal. It makes Visagreement the **ideal reproduction target**. But it means the extension must answer a question those papers leave open. Visagreement's authors report that disagreement did *not* correlate with their proxy quality metrics, because real datasets lack ground truth.

2. **The obvious version of Direction C is already done, including on soccer-flavoured data.**
   - FACT: Rahimian, Flisar & Sumpter (*J. Sports Analytics* 2025) built LLM "wordalisations" of an xG model. Wordalisation means turning model numbers into readable prose.
   - FACT: Ichmoukhamedov, Hinns & Martens (arXiv 2412.10220) already measure rank, sign and value faithfulness of SHAP narratives. One of their datasets is FIFA Man of the Match. They flipped SHAP signs and found LLMs "self-correct" back to intuition about 15–31% of the time.
   - A 2026 wave of "verify-and-repair" papers treats narrative faithfulness *measurement* as solved.
   - What remains is narrower. The main remaining gap is ablation-tested "absent-feature" hallucination: does the text credit features the model does not use?

3. **Direction B as stated ("learn embeddings, UMAP them, find tactical states") is already done and is methodologically weak.**
   - FACT: Tang et al. (StatsBomb Conf. 2023) trained an autoencoder on 360 freeze-frames, ran k-means per pitch zone, and picked k "manually … with the best interpretability".
   - FACT: Baron et al. (2024) show UMAP of action embeddings on open StatsBomb data, with visual checks only.
   - FACT: EventGPT (2025/26) reads inter-cluster distances off t-SNE, which is exactly the misuse documented by Jeon et al. (IEEE VIS 2026 / TVCG).
   - The **reliability audit** of such claims (Question family 6) is a real but narrow gap. It is a *domain application* of a mature VIS methodology.

4. **Soccer XAI is overwhelmingly display-only.** This is the strongest literature-synthesis finding.
   - Of **25** soccer papers (2021–2026) applying an explanation method to a soccer prediction model, **3** quantitatively evaluate the explanations:
     - PassAI's journal version (a ROAR faithfulness test)
     - Tsai et al. 2026 (global seed/model/method agreement)
     - Rahimian et al. 2025 (LLM text sign accuracy)
   - **0** test local (per-shot) agreement, **0** test input-perturbation stability, and **0** evaluate with coaches or analysts.
   - SYNTHESIS: this is a genuine *observed* gap (category B in your brief). But its value depends on the fact that the *general* methodology is saturated, so the novelty must be soccer-specific: known geometry as partial ground truth, semantically meaningful perturbations, and leakage structure.

5. **The most under-rated direction is methodological.**
   - Davis et al. (*Machine Learning* 2024) lay out how sports data violate i.i.d. assumptions and prescribe grouped and temporal splits.
   - Peters et al. (*RQES* 2026) show that **leaky features change SHAP rankings** in a pass-turnover model.
   - Several published soccer models use random row-level splits: Mead et al. 2023, TacticAI 2024, and the socceraction example notebooks, whose printed metrics are in-sample.
   - I found no paper that holds features fixed and varies *only the split regime* to see how explanations move. It is cheap and uses only public data. Its main risk is a null result for xG.

6. **Feasibility reality check.**
   - You have roughly **38–45 hours in total** (3–4 h/week × about 11 weeks), and the **4-page proposal is due Oct 20**, about 3 weeks away.
   - Every viable option below is a **reproduction plus one controlled extension plus lightweight linked views**. Nothing else fits.
   - FACT: the course syllabus says you must "demonstrate both the prior work, and your final research project" in class. The reproduction is not optional.

---

## 2. Course context (verified)

FACT (ctsilva.github.io/2026-VisML-CDS and /syllabus, fetched 2026-09-27):

| Item | Detail |
|---|---|
| Instructor | Claudio Silva. Grader: Bhavya Matam. |
| Project weight | 45% of the course grade |
| Project types | "Reproduce prior work or implement a proposed research idea of your choosing." You must demo both the prior work and your project. |
| Team size | 2–3 expected; **solo allowed**. Groups are fixed once formed. Team formation was **Week 2 (Sept 15), already past**. Confirm your solo status with the instructor or TA. |
| Milestones | **Proposal, 4 pages, Oct 20 (10%)**. Update, 1 page, Nov 3 (10%). Presentations Dec 1 and Dec 8. **Final report, 8 pages, Dec 14 (25%)**. |
| AI tools | Allowed; you must disclose which parts were AI-generated. |
| Default project | "Visual Analytics for AI-Generated Urban Infrastructure Maps" (Silva's Tile2Net line of work) |
| Front-end expectations | UNVERIFIED for 2026. A sub-agent reports that the 2024 syllabus listed JavaScript/D3 as expected background. Ask whether a Jupyter/Streamlit/Plotly linked-view tool is acceptable. Silva's own lab ships Jupyter widgets (Calibrate, SUBPLEX, PipelineProfiler), so it very likely is. |

**Lecture schedule, and where each candidate lands:**

| Date | Lecture | Candidates it supports |
|---|---|---|
| Oct 6 | Black-box interpretation **& Project Discussion** | **Bring 2–3 candidates to this session.** |
| Oct 13 | Clustering | Candidate 4 |
| Oct 20 | Dimensionality reduction | Candidate 4 |
| Oct 27 | Deep-learning visualization | – |
| Nov 3 | NLP/LLM visualization | Candidate 3 |
| Nov 10 | Topological data analysis | Candidate 1. MOUNTAINEER uses Mapper, a TDA summary graph. |
| Nov 24 | Interpretable ML and fairness | Candidates 1 and 2 |

---

## 3. Consolidated literature table

This table covers the load-bearing papers only. The full tables (about 150 rows) are in `lit_notes/`.

**Column key:**
- Status: **PR-J** = peer-reviewed journal; **PR-C** = peer-reviewed conference; **WS** = workshop; **PP** = preprint; **T** = tool/framework; **D** = dataset paper; **S** = survey; **Th** = thesis; **GL** = grey literature / industry conference.
- Compute: **L** = laptop CPU; **M** = modest GPU; **H** = heavy or proprietary pipeline.
- Ev (evidence level): **F** = full text; **A** = abstract; **2** = secondhand.

### 3a. Soccer XAI and explanation-of-soccer-models

| Paper | Yr | Venue / Status | Research question | Data (public?) | Model | Explanation / representation | Viz | Evaluation *of explanations* | Main finding | Limitation / future work | Code? | Data? | Compute | Ev |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Takamido, Ota, Nakamoto, **PassAI** | 2025 | IEEE Access 13 (PR-J); arXiv 2503.08945 is an earlier, *different* version | Predict and explain pass success from tracking image + passer stats | 95 J1 League games, Data Stadium (**no**) | ConvNeXt-T + MLP fusion | Gradient modality share; Grad-CAM; stats gradients | Heatmaps on rendered pitch | **ROAR faithfulness in the journal version only** (ROAR = RemOve And Retrain: delete top-attributed inputs, retrain, measure the accuracy drop). No method or model comparison, no stability test | Grad-CAM passes ROAR at 1% removal, but the curve is non-monotonic; the stats gradients fail it (AUTHOR CLAIM) | Proprietary data. One method. No random-removal baseline found in the text (tables are images). Input image contains the ball's *arrival point* (post-pass leakage; SYNTHESIS) | No | No | H | F |
| Tsai et al., *Limited transferability from elite leagues to university competition* | 2026 | arXiv 2605.10796 (PP) | Are explanations stable across seeds, models and methods, and under domain shift? | Big-5 via scraping (yes-ish); NTHU university matches (no) | RF, MLP | SHAP; Counterfactual Impact Score | Rank plots | **Seed stability, cross-model and cross-method agreement** (Spearman, **global only**) | Elite leagues give stable rankings; the university domain gives reordering and instability (AUTHOR CLAIM) | Match-level goal difference from 17 event counts. Not local. Not xG/pass/VAEP | No | Partly | L | F |
| Cefis & Carpita, *Accuracy and explainability of statistical and ML xG models* | 2024 | *Statistics* 59(2) (PR-J) | Compare 8 xG classifiers on accuracy and explainability | 7,801 Serie A shots (likely proprietary) | 8 classifiers incl. XGBoost, cloglog | "Appropriate explainability metrics" (UNVERIFIED) | ? | **Cross-model** (method unknown) | Distance, angle and visual angle dominate for all classifiers (AUTHOR CLAIM) | **Paywalled, not read. Must read before claiming a cross-model xG gap** | ? | ? | L | A |
| Rahimian, Flisar, Sumpter, *Automated explanation of ML models of footballing actions in words* | 2025 | *J. Sports Analytics* 11 (PR-J); arXiv 2504.00767 | Turn xG contributions into engaging LLM text | StatsBomb open (**yes**) | Logistic-regression xG per competition | c_j = β_j(x_j − mean) → template text → LLM restyling | Streamlit app | LLM judge (same family as the generator, temperature 1) checks the sign of **2 features** | Accuracy vs engagement trade-off (AUTHOR CLAIM) | No human evaluation; no check for hallucinated or absent features; non-linear models left open | **Yes** (Peggy4444/shotsGPT, GPL-3.0) | Yes | L + LLM | F |
| Cavus & Biecek, *Explainable expected goal models* | 2022 | IEEE DSAA (PR-C) | Aggregated profiles to compare players/teams | 315k Understat shots (yes, scraped) | GBM family (AutoML) | Aggregated ceteris-paribus profiles | Profile curves | **None** | Profiles can compare players | Oversampling distorts calibration and explanations | Yes | Yes | L | F |
| Cavus, Stańdo, Biecek, *Glocal explanations of xG models* | 2023/25 | arXiv → xAI 2025 CCIS (PR-C) | Group-level SHAP for scouting | Understat (yes) | same | Aggregated SHAP | Bars/profiles | **None** | Useful for goalkeeper blind spots (AUTHOR CLAIM) | No robustness analysis | Yes | Yes | L | F |
| Mead, O'Hare, McMenemy, *xG: improving performance and demonstrating value* | 2023 | PLOS ONE (PR-J) | Do player, team and psychological features help xG? | Wyscout public + FBref (yes) | LR, trees, MLP | SHAP, odds ratios | Summary plots | Informal cross-model | Player value matters (AUTHOR CLAIM) | **70/30 random shot-level split**, so player identity leaks across train/test (SYNTHESIS) | Partly | Yes | L | F |
| Iapteff et al., *Interpretable xG with Bayesian mixed models* | 2025 | Frontiers Sports (PR-J) | Interpretable-by-design xG | StatsBomb open, 63k shots (yes) | Bayesian GLMM | Coefficients + SHAP waterfalls | Waterfall plots | **None** | Competitive with StatsBomb xG (AUTHOR CLAIM) | – | Yes | Yes | L | 2 |
| Anzer & Bauer, *Expected passes* | 2022 | DMKD (PR-J) | Pass difficulty from tracking | DFL (no) | XGBoost | SHAP | SHAP plot | **None found** | – | Proprietary | No | No | M | 2 |
| Sahasrabudhe & Bekkers, *GNN deep-dive into counterattacks* | 2023 | MIT Sloan SSAC (GL) | Gender-specific GNNs for counterattacks | Graphs released (partly) | GNN | Permutation importance | Bars | **None** (no repeated runs) | Men's and women's importance differ; interpreted as real | Explanation variance never ruled out | Yes | Graphs | M | F |
| Peters, Parmar, Davies, James, *When timing matters: temporal leakage in pass-turnover models* | 2026 | *Res. Q. Exerc. Sport* (PR-J) | How much do post-pass features inflate xPT (expected pass turnover)? | 256k EPL passes (likely no) | Mixed LR, RF, GBM | SHAP | SHAP summary | Diagnostic: **leakage changes SHAP rankings** | AUC drops 0.08–0.18 once leaky features are removed (FACT, abstract) | Feature leakage, not split leakage | ? | ? | L | A |
| Van Haaren, *"Why would I trust your numbers?"* | 2021 | IJCAI AISA WS | Explainable-by-design xG with coach zones | Goal.com (no) | EBM | Shape functions | Zone maps | **None** (no user test of "easier to explain") | Soft zones as accurate as distance/angle (AUTHOR CLAIM) | Temporal split (good practice) | – | No | L | F |
| Kranzinger et al., *Scoping review of XAI in sports science* | 2025 | *Discover AI* (S) | – | 19 studies | – | SHAP dominates | – | – | "Limited validation of explanations with domain experts" (FACT, abstract) | Calls for comparative evaluation | – | – | – | A |
| Zhu, *Applications of XAI in football: a systematic review* (protocol) | 2026 | OSF registration (protocol) | – | – | – | – | – | – | Review underway | Scooping risk for a *survey-type* project only | – | – | – | A |

### 3b. General XAI evaluation (what is already saturated)

| Paper | Yr | Venue / Status | Topic | Key finding | Relevance | Code? | Ev |
|---|---|---|---|---|---|---|---|
| Krishna et al., *The disagreement problem in explainable ML* | 2022/24 | TMLR (PR-J) | Cross-method agreement | Methods often disagree; defines feature/rank/sign agreement metrics | The canonical metric set; reuse it rather than inventing new metrics | via OpenXAI | A |
| Han, Srinivas, Lakkaraju, *Which explanation should I choose?* | 2022 | NeurIPS (PR-C) | Theory | LIME, KernelSHAP, IG and others are local function approximations; predicts which methods should agree | Some disagreement is expected by definition, so observing it is not a discovery | Yes | A |
| **Solunke, …, Nonato, C. Silva, MOUNTAINEER** | 2024 | IEEE TVCG (PR-J) | VA comparison of attributions (7 methods; 6 models) | Mapper graphs of attributions. Limitations: one-vs-one comparison only, scales to "a few dozen" explanation sets | **Instructor's lab. Directly overlaps Direction A.** | Mentioned in paper | F |
| **P. Silva, …, C. Silva, Nonato, Visagreement** | 2025 | IEEE TVCG 31(10) (PR-J) | VA for disagreement | Tabular binary classification, local methods via Captum. **No correlation between proxy quality metrics and disagreement.** Disagreement is higher where the model is wrong | **Instructor's lab. The reproduction target.** | **Yes** (priscylla/visagreement; small, no license file, 2023) | A + F (DC summary) |
| P. Silva, C. Silva, Nonato, *Feature attribution methods and model performance* | 2024 | AAAI AI4Ed WS | Agreement vs accuracy | Strong correlation between the two (education data) | Contested by Pereira et al. 2026, so this is a live question | ? | A |
| Silva Sousa, …, C. Silva, Nonato, *Explainalytics* | 2025 | *Information Systems* (PR-J) | VA for attribution-method selection | 7 metrics, 5 views, 10-user study (SECONDHAND) | Instructor's lab again | Library | 2 |
| Laberge et al., *Partial order in chaos* | 2023 | JMLR (PR-J) | Rashomon consensus (the set of near-equally accurate models is the "Rashomon set") | Keeps only attribution statements that every good model agrees on, giving a partial order | Principled output for cross-model agreement | Yes | A |
| Donnelly et al., *Rashomon Importance Distribution* | 2023 | NeurIPS (PR-C) | Rashomon | Stable importance distribution across good models | Gold standard, but heavy | Yes | A |
| Hwang et al., *Explanation multiplicity in SHAP* | 2026 | arXiv (PP) | Stability | L2-style stability numbers **mask** instability in the top-k ranking | Report rank and top-k identity, not just L2 | ? | A |
| Agarwal et al., *OpenXAI* | 2022 | NeurIPS D&B (PR-C) | Benchmark | 22 metrics: faithfulness (PGI/PGU), stability (RIS/ROS), and agreement with a reference | The metric library to use | **Yes** (MIT) | A |
| Slack et al., *Fooling LIME and SHAP* | 2020 | AIES (PR-C) | Adversarial | A scaffolded model can look innocuous to LIME/SHAP | Template for a positive control ("plausible but unfaithful") | Yes | A |
| Frye et al., *Shapley explainability on the data manifold* | 2021 | ICLR (PR-C) | Off-manifold | Off-manifold SHAP can mislead | Justifies semantic, on-manifold perturbations | ? | A |
| Nauta et al., *From anecdotal evidence to quantitative evaluation* | 2023 | ACM CSUR (S) | Meta | 1 in 3 XAI papers evaluate only anecdotally | Frames evaluation as the contribution | – | A |
| Bommer et al., *Finding the right XAI method (climate)* | 2024 | AMS AIES (PR-J) | Domain evaluation | Metric-based domain evaluation, **not** physical ground truth | Leaves room for "domain physics as partial ground truth" | Yes | A |
| Conti et al., *Weight of evidence for alignment and stability of explanations* | 2026 | ECML XKDD WS | Domain-prior alignment | Hypothesis tests of explanations against domain-knowledge references | **Closest general method to "soccer geometry as ground truth"**; weakens its novelty | ? | A |
| Higgs et al., *XAI for geoscientific regression: Lorenz-63* | 2026 | arXiv (PP) | Known-dynamics testbed | A known physical system exposes XAI failure modes | Confirms the idea is live, not new | ? | A |

### 3c. LLM and natural-language explanations

| Paper | Yr | Venue / Status | Setting | Faithfulness evaluated? | Key finding | Code? | Ev |
|---|---|---|---|---|---|---|---|
| **Ichmoukhamedov, Hinns, Martens, *How good is my story?*** | 2024 | arXiv 2412.10220 (PP) | LLM verbalizes SHAP | **Yes**: a second LLM extracts rank/sign/value (RA/SA/VA), validated extractor | Flipped-sign tables: LLMs revert to the intuitive sign about 15–31% of the time (FIFA MoM included). Hallucinated features are **dropped** from scoring | Yes | F |
| Zytek et al., *Explingo* | 2024 | IEEE BigData (PR-C) | LLM verbalizes SHAP | Yes (accuracy and completeness graders, 96% agreement with humans) | Few-shot exemplars lower accuracy; **Mistral-7B narrator accuracy 0.98/4** | Yes | F |
| He & Martens, *Agentic XAI narratives* | 2026 | arXiv (PP) | Narrator + critic loop | Yes (RA/SA/VA + absent features as the critic signal) | Repair loop cuts unfaithful narratives by 90% (AUTHOR CLAIM) | Yes | F (skim) |
| Martens et al., *XAIstories* | 2025 | *Decision Support Systems* (PR-J) | Narratives | **No** (persuasiveness only) | Over 90% of lay users found them convincing | ? | A |
| Caut, …, Sumpter, *Representing data in words* | 2025/26 | arXiv (PP) | Wordalisation of stats | LLM judge + 26 human raters | Wordalisation as faithful as a stats text, but more engaging | Yes | F |
| Turpin et al., *LMs don't always say what they think* | 2023 | NeurIPS (PR-C) | Self-explanation | Yes (biasing features) | Chain-of-thought rationalizes biased answers | Yes | A |
| Mayne et al., *A positive case for faithfulness* | 2026 | arXiv (PP) | Self-explanation | Yes (simulatability) | Features that need domain expertise drive egregious unfaithfulness | ? | F (grep) |
| Xu et al., *Survey of large models in sports* | 2026 | arXiv (S) | – | – | Names hallucination as an open challenge; **does not mention** faithfulness to a predictive model | – | F (grep) |

### 3d. Tactical representations and DR reliability

| Paper | Yr | Venue / Status | Data (public?) | Representation | Viz | Evaluation of structure | Main finding / limitation | Code? | Ev |
|---|---|---|---|---|---|---|---|---|---|
| **Tang, Wang, Zhang, *Clustering football game situations via deep representation learning*** | 2023 | StatsBomb Conf. (GL) | 360 EPL 21/22 (not in open data) | CNN autoencoder (3 pretext heads) → k-means per zone | Heatmaps | **None**: k chosen "manually … best interpretability" | 95 hand-named situations. The authors concede samples may not fit their label | **Yes** (MIT) | F |
| Baron, Hocevar, Salehe, *A foundation model for soccer* | 2024 | arXiv (PP) | StatsBomb open (**yes**) | Transformer action embeddings | **UMAP** | **Visual only** | Circular: x/y and action type are inputs, so their appearance in UMAP is expected (SYNTHESIS) | Yes | F |
| Hong et al., *EventGPT* | 2025/26 | arXiv (PP) | EPL (likely no) | Player embeddings | **t-SNE** | Visual; reads *inter-cluster adjacency* | A textbook example of the misuse Jeon et al. document | No | F (grep) |
| Wang et al. (DeepMind), *TacticAI* | 2024 | Nature Comms (PR-J) | 7–9k EPL corners (**no**) | Equivariant GNN | t-SNE (illustrative) | 5-expert blinded retrieval study | Validates the embedding, not the 2D picture. **80/20 random split** | No | F |
| Bauer, Anzer, Shaw, *Team formations in context* | 2023 | JSA (PR-J) | DFL tracking (**no**) | CNN phase detection | Formation plots | Labels; inter-labeler F1 about 0.78 | Phases need proprietary tracking | No | F |
| Li & Link, *Intention-driven match phases* | 2026 | arXiv (PP) | 7 matches TRACAB (no) | Temporal GAT | Timelines | Labels from **a single annotator, no inter-rater reliability** (FACT) | Shows how subjective phase labels are | No | F |
| Sotudeh, *Formation identification survey* | 2025 | Frontiers (S) | – | – | – | – | **StatsBomb, Wyscout and Stats Perform formation labels agree in only 30% (13/44) of cases** (FACT) | – | F |
| **Jeon, Park, Shin, Seo, *Stop misusing t-SNE and UMAP for visual analytics*** | 2025/26 | IEEE VIS 2026 / TVCG (arXiv 2506.08725) | 136 VA papers | – | – | Literature review + interviews | Misuse is common (>40% give no justification); caused by "limited DR literacy" | – | F |
| Atzberger et al., *Large-scale sensitivity analysis of text spatializations* | 2024 | IEEE VIS (PR-C) | 3 text corpora | 6 embeddings × DR × hyperparameters × seeds | 42,817 layouts | Quantitative stability | **The methodological template, and the saturation threat** | Yes | A |
| Jeon et al., *ZADU* / *Classes are not clusters* / adjusted internal validation measures | 2023–25 | VIS / TVCG / TPAMI | – | Distortion metrics library; label-based metrics that don't assume classes form clusters | – | – | The metric toolkit | **Yes** (zadu, MIT) | A |
| Jung et al., *GhostUMAP2* | 2025 | TVCG (PR-J) | – | Point-wise UMAP stability | – | – | UMAP positions are often "determined mostly by chance" (AUTHOR CLAIM) | Likely | A |
| Chari & Pachter; Lause, Berens, Kobak | 2023 / 2024 | PLOS CB (PR-J) | scRNA-seq | – | – | Quantitative | A live debate about whether 2D embeddings mislead; cite both sides | Yes | F / A |

### 3e. Methodology, datasets, frameworks

| Paper / resource | Yr | Venue / Status | Key content | Relevance | Ev |
|---|---|---|---|---|---|
| **Davis et al., *Methodology and evaluation in sports analytics*** | 2024 | *Machine Learning* 113 (PR-J) | Sports data are not i.i.d. (temporal dependence, subject consistency). Prescribes temporal and subject-level splits, leave-one-competition-out, and Brier/log-loss **with reliability diagrams** | The methodological anchor. Proposes **no protocol for evaluating explanations** (FACT) | F |
| Robberechts & Davis, *How data availability affects xG* | 2020 | MLSA WS | AUROC is inappropriate for xG; use Brier. Don't mix data providers | Evaluation standards for xG | F |
| Van Roy et al., *xT vs VAEP* | 2020 | AAAI WS | Split-half robustness: xT ρ=0.89 vs VAEP ρ=0.25 | Reliability framing | F |
| Yeung, Ide, Someya, Fujii, **OpenSTARLab** | 2025 | arXiv / Complex & Intelligent Systems (T) | Converters to a unified event format, event models, RL | **Ships no data.** Half its benchmark uses non-open data. **No DR or cluster tooling** | F |
| Bassek et al., **IDSSE dataset** | 2025 | *Scientific Data* (D) | 7 DFL matches, 25 Hz, CC BY | The only open optical tracking with official events | A + verified download |
| **StatsBomb Open Data** | live | (D) | 3,961 matches; 426 with 360 frames | Primary data source | Verified by counting |

---

## 4. Deep dives on the papers that decide the question

### 4.1 Visagreement and MOUNTAINEER (Silva lab): the prior work your grader wrote
- **FACT (MOUNTAINEER, full text):**
  - Uses Mapper (a topological summary graph) over attribution vectors, linked back to data and predictions.
  - Case study 1 compares 7 Captum methods on HELOC credit data.
  - Case study 2 compares LIME across 6 model classes on ACSEmployment. That is already a cross-model comparison.
  - Evaluated with expert interviews.
  - Limitations: one-vs-one comparison only, and it scales to "a few dozen" explanation sets.
  - Future work: multi-class, regression, and comparing multiple graphs.
- **FACT (Visagreement, via the author's IJCAI-2025 doctoral-consortium summary; TVCG text not read):**
  - Tabular binary classification, local methods (KernelSHAP, LIME, DeepLift, IG via Captum), and a "(dis)agreement space".
  - Findings:
    - (1) No correlation between explanation-quality metrics and agreement level.
    - (2) The highest-disagreement instances have much worse model performance.
    - (3) No consistent feature drives disagreement.
  - Evaluated with 4 experts.
- **FACT (repo check):** `priscylla/visagreement` exists with a `load_model(model, X_train, X_test, y_test)` + Captum API. It has 0 stars, was last pushed in 2023, and has no license file. Budget time for fixing it.
- **SYNTHESIS: why this matters.**
  - Finding (1) is open *because* real datasets lack ground truth.
  - Soccer shot geometry supplies **partial** ground truth. Examples:
    - Mirror symmetry across the pitch's long axis
    - Monotone decrease with distance
    - Iso-distance arcs that change angle but not distance
  - That lets you ask the question Visagreement couldn't: *does disagreement actually flag wrong explanations?*
  - This turns "I applied your tool" into "I tested your tool's open question in a domain that has ground truth."

### 4.2 Soccer XAI count synthesis (the observed gap)
FACT/SYNTHESIS (`lit_notes/01`): N = 25 soccer papers applying an explanation method to a soccer prediction model, 2021–2026.

| Criterion | Count | Papers |
|---|---|---|
| Quantitatively evaluate explanations at all | **3** (+1 unverified) | PassAI; Tsai 2026; Rahimian 2025. Unverified: Cefis & Carpita |
| Removal/perturbation faithfulness | **1** | PassAI |
| Compare explanation methods quantitatively | **1** | Tsai (global) |
| Compare across models quantitatively | **1** (+1 unverified, 4 informal) | Tsai; Cefis & Carpita? |
| Seed/retraining stability | **1** | Tsai (global) |
| Input-perturbation stability | **0** | – |
| **Local** (per-shot/per-pass) agreement or stability | **0** | – |
| Human evaluation of explanations | **0** | – |
| Display-only | ~19–21 | Cavus ×2, Anzer & Bauer, Forcher ×2, Sahasrabudhe & Bekkers, Iapteff, and others |

**Caveat:** unindexed workshops (MLSA, StatsBomb Conf.) were not swept item by item.

### 4.3 PassAI: a good citation, but not a reproduction target
- FACT:
  - The journal version (IEEE Access 13:132884–98) adds ROAR. The arXiv v1 has none and calls for it.
  - Data is proprietary J1 League (Data Stadium). There is no code.
  - The rendered input image includes a line to where the ball *arrived*, which is post-pass information.
- SYNTHESIS:
  - A faithful reproduction is impossible, and a re-implementation on StatsBomb 360 is heavy for 40 hours.
  - Cite it as (a) the only soccer faithfulness test and (b) evidence that its authors call for "quantitative and concrete evaluation methods for generated explanations".

### 4.4 Rahimian et al. 2025 and Ichmoukhamedov et al. 2024: why Direction C is mostly taken
- **FACT (Rahimian, full text + code):**
  - Pipeline: logistic-regression xG → contribution c_j = β_j(x_j − mean_j) (exact linear SHAP) → deterministic template sentences for features with |c_j| > 0.1 → LLM restyling.
  - Accuracy is checked by a Gemini judge at temperature 1 on only 2 features.
  - The template layer produces sentences like "…left foot, which had the maximum positive contribution because the shot was with the right foot". This is faithful to a mean-centred model, but it reads as a contradiction.
- **FACT (Ichmoukhamedov, full text; checked independently):**
  - Rank, sign and value agreement via a validated extractor.
  - Datasets include FIFA Man of the Match.
  - The manipulation experiment (inverted ranks, flipped signs) shows LLMs revert to intuitive signs.
  - Extracted features that are not in the SHAP table are **omitted** from scoring.
- **SYNTHESIS: what's left.**
  - (O2) *Absent-feature hallucination*, tested by **ablation**. Retrain xG without, e.g., goalkeeper features, keep goalkeeper words available to the narrator, and measure how often the narrative credits goalkeeper positioning.
  - (O1) *Natural* prior–model conflicts, which arise from mean-centred binaries and collinear pressure features, versus Ichmoukhamedov's synthetic flips.
  - These are real but incremental over Ichmoukhamedov.

### 4.5 Tang et al. 2023, Baron et al. 2024, Jeon et al. 2025/26: why Direction B must become a reliability question
- FACT:
  - Tang is the closest prior art: 360 freeze frames → autoencoder → k-means per zone → 95 named situations. There are no stability checks. The code is public (MIT).
  - Baron reads UMAP visually, and the properties it "discovers" are model inputs.
  - Jeon et al. document systemic misuse of t-SNE and UMAP in VA papers.
  - Atzberger et al. (VIS 2024) already did a large-scale sensitivity analysis for *text* spatializations.
- SYNTHESIS: the only defensible form is an **audit**: "which published-style tactical claims survive seed, hyperparameter and encoder changes, and agree with non-input proxies?" It is a domain application of VIS methodology, and it is honest about that.

### 4.6 Davis et al. 2024 + Peters et al. 2026: the methodological gap
- FACT:
  - Davis et al. prescribe grouped and temporal splits and calibration diagnostics.
  - Peters et al. show SHAP rankings shift when leaky post-pass features are removed.
  - Mead 2023 and TacticAI used random splits.
  - socceraction's example notebooks print in-sample metrics.
- SYNTHESIS:
  - Nobody has held features fixed and varied *only the split regime* to measure how explanations change.
  - **Risk:** shots are close to conditionally independent, so the xG effect may be small. VAEP labels ("goal within the next 10 actions") make consecutive actions near-duplicates, so a random action-level split plausibly leaks. That is a hypothesis to test.

---

## 5. Verdicts on Directions A, B, C

| Direction | Verdict | Why (with evidence) |
|---|---|---|
| **A: Prediction + explainability** | **Viable, but only in a specific form.** The generic form is **saturated and owned by your grader's lab**. | Generic cross-method disagreement: Krishna 2022/24, Han 2022, MOUNTAINEER, Visagreement, Explainalytics. Generic noise stability: OpenXAI, Quantus, Alvarez-Melis. The soccer-specific observed gap is 0/25 papers on local agreement or perturbation stability (§4.2). The defensible form uses **soccer geometry as partial ground truth** to test Visagreement's open question. Novelty is moderated by Conti 2026 and Higgs 2026, which do "domain knowledge as reference" in other fields. |
| **B: Tactical representations / phases** | **As stated: do not do it.** As a *projection-reliability audit*: viable and the most visualization-native, but novelty is domain-application only. | Done: Tang 2023, Baron 2024, EventGPT, Rothe 2026. Phase labels need proprietary tracking and are low-agreement (30% vendor formation agreement; a single annotator in Li & Link). Methodology mature: ZADU, Atzberger 2024, GhostUMAP2, Jeon 2025. |
| **C: Soccer + LLM explanations** | **Weaker than it looks.** The core question is already answered in general ML *and* touched on soccer data. One narrow gap remains. | Rahimian 2025; Ichmoukhamedov 2024 (FIFA MoM, flipped signs); Explingo 2024; the 2026 verify-and-repair wave. Remaining: ablation-tested absent-feature hallucination; natural conflicts; a local-model version. Also the weakest *visualization* story of the three. |

**Adjacent direction surfaced by the review (not in your original three):**
- **D: Evaluation-protocol dependence of explanations** (split, leakage). See Candidate 2. It is the most "methodological", uses the least compute, and matches Silva's calibration/subgroup work (Calibrate, TVCG 2023).

---

## 6. Verdicts on Question families 1–7

| # | Family | Already addressed? | Verdict |
|---|---|---|---|
| 1 | Explanation faithfulness via VA | General: yes (OpenXAI PGI/PGU, Visagreement, Explainalytics). Soccer: PassAI ROAR only (1 method, no random baseline) | **Open only with a ground-truth hook.** Faithfulness against *model* behaviour (deletion tests) is standard. Faithfulness against *known domain structure* is the soccer-specific angle. Integrated Gradients and saliency need a differentiable model (MLP). For a GBM use TreeSHAP; don't force IG onto trees. |
| 2 | Robustness to semantically meaningful perturbations | General (generic noise): saturated. Semantic on-manifold: thin (Frye 2021 motivates it). Soccer: **0 papers** | **Most soccer-specific sub-question.** Example perturbations: move a shot along an iso-distance arc; mirror it; shift one defender inside or outside the shot cone. "Does the explanation change smoothly?" becomes testable against a known expectation. Report rank and top-k measures, not just L2 (Hwang 2026). |
| 3 | Cross-model explanations | General: active (Rashomon literature; MOUNTAINEER case 2). Soccer: Tsai 2026 (global, match-level), Cefis & Carpita 2024 (**unread**) | **Plausibly open locally for xG**, but you must read Cefis & Carpita first. Use a small Rashomon set (GBM seeds and hyperparameters + GAM + logistic regression within ε log-loss) and report Laberge-style partial orders. |
| 4 | LLM explanation faithfulness | **Largely addressed** (Ichmoukhamedov, Explingo, He & Martens, Rahimian) | **Only the narrow ablation / absent-feature version is open.** Local gpt-oss-20b is feasible but self-grading is a flaw; use deterministic lexicon checks. |
| 5 | Tactical representation | **Addressed** (Tang, Baron, Rothe, TacticAI) | **Don't pursue as discovery.** |
| 6 | Projection reliability | General: mature. Sports: **none found** (search-limited) | **Real but narrow**; a domain audit. Strongest course fit (Oct 13 and Oct 20 lectures). |
| 7 | Human-centered evaluation | Soccer: **0** controlled studies of explanation designs; 7 of 13 soccer VA papers use 2–4-expert interviews/case studies. Silva's lab norm is 4–8 experts, think-aloud | **Real gap, infeasible as the main contribution** for a solo 40-hour project (recruiting, IRB). **Use as a light add-on:** 3–6 think-alouds with soccer-literate people, reported qualitatively; or a forward-simulation pilot with classmates, labelled as a pilot. Forward simulation = users predict what the model will output. |

---

## 7. Gap synthesis table

"Novelty confidence" is justified in words, as requested.

| Potential gap | Evidence | Closest papers | Already addressed? | Novelty confidence (why) | Data availability | Compute | Evaluation difficulty | Visualization fit | Solo feasibility | Main risk |
|---|---|---|---|---|---|---|---|---|---|---|
| **G1. Does explanation disagreement (cross-method, cross-model) actually flag explanations that violate known soccer geometry?** | 0/25 soccer papers test local agreement; Visagreement found no link between disagreement and proxy quality *because no ground truth*; geometry gives partial ground truth | Visagreement, MOUNTAINEER, Krishna, Conti 2026, Tsai 2026, Cefis & Carpita | Partly: general disagreement yes; soccer local no; "domain prior as reference" exists in general (Conti 2026) | **Moderate.** The contribution is a *test of an open question from a named prior paper* in a domain with ground truth. Not a new method. Weakened by Conti 2026 and Higgs 2026, strengthened by the Silva-lab link | **High** (StatsBomb open: shot freeze-frames for every match) | L | Medium: needs a crisp "violation" definition and a positive control | High (pitch + attribution small multiples + disagreement scatter) | **Good** | Explanations may simply all agree on distance/angle (trivial). Mitigate with context features and a planted-leak control |
| **G2. Stability of local soccer explanations under semantic perturbations** | 0 soccer papers; generic noise stability saturated | OpenXAI (RIS), Frye 2021, Hwang 2026 | General generic: yes. Soccer semantic: no | **Moderate-low alone**; better folded into G1 | High | L | Low-medium (perturbations are well-defined) | High | Good | Reads as "apply OpenXAI to soccer" unless tied to expected behaviour |
| **G3. Split-regime dependence of explanations (fixed features)** | Davis 2024 prescriptions; Peters 2026 (feature leakage changes SHAP); random splits in published models; in-sample socceraction metrics | Davis 2024, Peters 2026, Tsai 2026, Calibrate | Not found (search-limited) | **Moderate.** Concrete, cheap, clearly framed. Risk that it is "obvious" to methodologists | High | L | Low (paired comparisons across regimes) | Medium-high (bump charts, reliability diagrams, pitch diff maps) | **Very good** | Null result for xG; VAEP result may be "leakage is bad, obviously". Must show *which* explanations move and why |
| **G4. Absent-feature hallucination in LLM narratives of soccer models (ablation-tested)** | Rahimian checks 2 features; Ichmoukhamedov drops hallucinated features from scoring | Rahimian 2025, Ichmoukhamedov 2024, He & Martens 2026 | Measurement framework yes; this specific test no | **Moderate-low**: an incremental design choice on a well-populated 2024–26 line. Fast-moving, so scoop risk | High (shotsGPT uses open data) | L + local LLM (hours) | Medium: extractor reliability is the crux | **Medium-low** (claim-to-contribution linking view) | Good | Self-grading local LLM; weak visualization component |
| **G5. Reliability of tactical claims from 2D projections of soccer embeddings** | Baron UMAP visual-only; EventGPT misuse; Tang k by interpretability; no sports DR audit found | Jeon 2025, Atzberger 2024, ZADU, Tang 2023 | General method yes; sports no (search-limited) | **Moderate-low**: domain application of a mature methodology. Value lies in the *claim audit* and non-circular proxies | Medium (open 360 is anonymous, and censored to the camera view) | M (encoder seeds on HPC) | Medium-high ("what counts as surviving?") | **Very high** | Medium | "UMAP is unstable" is known; weak encoder makes everything unstable |
| **G6. Controlled user study of soccer explanation designs** | 0 soccer controlled studies; Kranzinger calls for it | TacticAI (not explanations), Silva-lab think-alouds | No | High novelty, **but infeasible** | – | L | Very high (recruiting, IRB, power) | High | **Poor** | Can't recruit enough analysts |
| G7. Faithfulness of GNN explanations for soccer tracking models | 0 papers; GNN work is display-only | Sahasrabudhe & Bekkers, Afshar 2026 | No | Moderate | Low-medium (open tracking is tiny) | M-H | High | Medium | **Poor** | Data scarcity + GNN explainability is its own swamp |
| G8. PassAI re-implementation on open data with a leakage-free input and a stronger ROAR | PassAI's thin ROAR, arrival-point leakage | PassAI | No | Moderate | Medium (360 anonymous/censored) | M-H (CNN retraining for ROAR) | Medium | Medium | **Poor at 40 h** | ROAR retraining × methods × fractions blows the budget |

---

## 8. Data: what you can actually download

Verified 2026-09-27 by counting files and range-requests; see `lit_notes/05`.

| Resource | Access | License | Scale | Key point for you |
|---|---|---|---|---|
| **StatsBomb Open Data** (`hudl/open-data`) | ✅ verified | Non-commercial user agreement; credit + logo in publications; no raw redistribution | **3,961 matches**; **426 with 360 frames** (WC22 64, Euro20 51, Euro24 51, WWC23 64, WEuro22 31, WEuro25 31, plus single-club seasons) | **Every shot in every match has a freeze-frame with player IDs**, so defender and goalkeeper features for xG come essentially free. SPECULATION: at about 25 shots/match that is on the order of 10^5 shots; count them before committing. `statsbomb_xg` is a free reference model. The repo is 7.4 GB, so stream files; don't clone. 360 frames are anonymous and limited to the camera's visible area (censored defender counts). |
| Wyscout public (Pappalardo 2019) | ✅ | **CC BY 4.0** | 1,941 matches (2017/18 big-5 + WC18 + Euro16) | Cleanest license; no freeze-frames, no xG |
| SkillCorner Open Data | ✅ | MIT | **20** A-League matches, 10 Hz (expanded 2026-09-08) | Best-licensed open tracking; Git-LFS; includes phase-of-play and EPV columns |
| IDSSE (Bassek 2025) | ✅ | CC BY 4.0 | 7 DFL matches, 25 Hz optical | Case studies only |
| Metrica sample | ✅ | No license file | 3 anonymized matches | Demo only |
| PFF FC WC 2022 tracking | ⚠️ form-gated; terms unverified | ? | 64 matches | Stretch goal only |
| OpenSTARLab | ✅ packages | Apache-2.0 | **No data of its own** | Loader/benchmark tooling; no DR/cluster tools |
| Google Research Football | ⚠️ **archived** | Apache-2.0 | simulated | Avoid |
| SoccerNet | ⚠️ NDA for video | mixed | 550 games | Off-scope |

**Tooling:**
- `kloppy`: backbone loaders.
- `socceraction`: SPADL/VAEP/xT. Needs Python <3.13 and numpy <2, so use a separate environment.
- `un-xPass`: pass success on 360.
- `mplsoccer`: pitch plots.
- `shap`, `captum`, `OpenXAI`: attribution methods and metrics.
- `zadu`: DR metrics.
- Visagreement: the Silva-lab tool.
- `shotsGPT`: the wordalisation pipeline.

**Data-driven SYNTHESIS:** Candidates 1–3 all run on StatsBomb open events with shot freeze-frames on a laptop. Only Candidate 4 benefits from the HPC (multiple encoder seeds).

---

## 9. What transfers from Silva's sports visual analytics

FACT base: Baseball4D (IEEE VAST 2014), StatCast Dashboard (IEEE CG&A 2016), Baseball Timeline (EuroVis 2018), CS:GO action valuation (BigData 2020), ggViz (CHI PLAY 2022), Calibrate (TVCG 2023), SUBPLEX (CG&A 2022), Visagreement (TVCG 2025). No Silva soccer paper was found. SYNTHESIS on what to borrow:

1. **Audit the data first.** Baseball4D was partly built to check tracking accuracy. For you, that means surfacing StatsBomb 360 visible-area censoring and event-labelling artefacts. Davis 2024 cites an xG artefact: corner-square shots converting at 72%.
2. **Query across many plays, not single-play replay.** Brush subgroups of shots (zone, body part, tournament) and compare model and explanation behaviour per subgroup.
3. **Calibration per subgroup.** Calibrate's learned reliability diagram with brushed subgroups is directly reusable for xG (its paper names sports betting as a motivation).
4. **Temporal splits plus Brier plus calibration curves**, as in Silva's own CS:GO valuation model.
5. **Treat explanations as objects to validate, not truth** (SUBPLEX, Visagreement).
6. **Ship Jupyter-native linked views**, not bespoke D3 apps. This is exactly right for your skill profile.
7. **Evaluate with 4–8 experts via think-aloud.** That is the accepted norm in this instructor's own TVCG papers.

---

## 10. Tempting but bad choices for this project

| Tempting idea | Why it's bad *here* | Evidence |
|---|---|---|
| "Compare SHAP, LIME and IG on an xG model in an interactive dashboard" | This is Visagreement/MOUNTAINEER on a new dataset, and **the grader wrote those tools** | MOUNTAINEER, Visagreement |
| "Use an LLM to explain xG predictions and check if it's faithful" | Rahimian 2025 did the pipeline; Ichmoukhamedov 2024 did the faithfulness metrics and prior-override experiment (on FIFA data); 2026 papers do repair loops | §4.4 |
| "Learn possession embeddings, UMAP them, discover tactical states" | Done (Tang 2023, Baron 2024). Circular when inputs reappear in the plot. Classic DR misuse. Produces pretty pictures and weak conclusions | Jeon 2025/26; §4.5 |
| Tactical phase segmentation (build-up / transition / …) | Needs proprietary tracking; labels are subjective (single annotator; 30% vendor formation agreement) | Bauer 2023, Li & Link 2026, Sotudeh 2025 |
| Reproduce PassAI | Proprietary J1 data, no code; the input leaks the arrival point; ROAR requires retraining a CNN many times | §4.3 |
| Generic "SHAP stability under Gaussian noise" | Saturated (Alvarez-Melis 2018, OpenXAI, Quantus); off-manifold perturbations are themselves criticized | §3b |
| Controlled user study of explanation designs as the main contribution | Recruiting analysts, IRB, statistical power; not feasible solo in 40 h | §6, family 7 |
| GNN / transformer on tracking data (TacticAI-style) | Open tracking is 7–20 matches; GPU-heavy; GNN explanations are an unsolved swamp | §8, G7 |
| A soccer "foundation model" or next-event transformer | Compute- and engineering-heavy; the novelty is in the model, not the visualization | Baron 2024, EventGPT, OpenSTARLab |
| SoccerNet / broadcast-video CV pipeline (tempting given your CV strength) | Video behind an NDA, heavy compute, and the output is a CV system, not a Vis4ML study | `lit_notes/05` |
| RL / Google Research Football / OpenSTARLab RLearn | GRF archived; simulated distribution; RL is a separate project | `lit_notes/05` |
| A systematic review of soccer XAI | An OSF-registered systematic review is already underway (July 2026); a course wants a system or experiment | Zhu 2026 OSF |
| Full Rashomon Importance Distribution / exhaustive Rashomon enumeration | Theoretically lovely, computationally heavy; use a small sampled Rashomon set instead | Donnelly 2023 |
| A polished standalone D3 web application | Your weakest skill; the course rewards the scientific question. Use Jupyter/Streamlit/Plotly linked views | Silva lab practice |
| LLM-as-predictor self-explanation for soccer | Saturated NLP subfield (Turpin, Lanham, Madsen, Parcalabescu); weak soccer-specific angle | §3c |

---

## 11. Final shortlist: four candidate research questions

No winner is declared. Each candidate has a different risk profile. Hour estimates are **SPECULATION** based on your stated skills plus coding-agent help; your pace will differ.

---

### Candidate 1: Physics-anchored evaluation of explanation disagreement for xG

**Research question.** For xG models trained on public StatsBomb shots, (i) do local attributions respect known shot-geometry constraints under semantically meaningful perturbations, and (ii) do explanation-disagreement signals, across methods (à la Visagreement) and across a small set of equally accurate models, actually identify the shots whose explanations violate those constraints?
- Constraints tested:
  - Invariance under mirroring across the pitch's long axis
  - Stable distance attribution along an iso-distance arc
  - Monotonicity in distance

**Why it might matter.** Explanation-disagreement tools assume disagreement signals untrustworthy explanations. Visagreement's authors could not test that on real data because there was no ground truth. Soccer supplies partial ground truth, so this tests a stated open question from the instructor's own line of work.

**Evidence for the gap.**
- 0/25 soccer papers test local agreement or perturbation stability (§4.2).
- Visagreement's no-correlation finding rests on proxy metrics (FACT, DC summary).
- Tsai 2026 is global and match-level only.
- Counter-evidence that tempers novelty: Conti 2026 (WoE alignment with domain hypotheses) and Higgs 2026 (known-dynamics testbed) do "domain knowledge as reference" elsewhere.

**Closest prior work to reproduce.**
- **Visagreement** (run its library on one of its paper's datasets, then on an xG MLP). Fallback if the repo is broken: reproduce its disagreement metrics (Krishna et al.) with OpenXAI.
- Plus a standard xG baseline, validated against `statsbomb_xg` (AUC, Brier, reliability diagram).

**Minimum viable project.**
1. Build an xG table: open-play shots from 4 tournaments. Features: distance, angle, body part, and freeze-frame features (defenders in the shot cone, nearest defender, goalkeeper position).
2. Train logistic regression, XGBoost and an MLP, with a tournament-grouped split.
3. Compute attributions: TreeSHAP for XGBoost; KernelSHAP, LIME and IG for the MLP.
4. Run 3 perturbation tests with pre-registered expectations.
5. Compute a per-shot "violation score" and a per-shot disagreement score. Test whether disagreement predicts violation (rank correlation, AUROC).
6. Add a **positive control**: an xG model with a planted leak or scaffold (Slack-style), to show the pipeline can detect a known-bad explanation.

**Stronger version.**
- A Rashomon set of about 20–50 near-equivalent models, with Laberge partial orders per feature.
- Conditional vs interventional SHAP on correlated geometry (Aas 2021, `shapr`).
- A second task (pass success via un-xPass on 360).
- Mapper-based comparison as a direct MOUNTAINEER extension (Nov 10 TDA lecture).
- 3–5 think-alouds.

**Dataset.** StatsBomb Open Data: WC22, Euro20, Euro24, WWC23 shots with shot freeze-frames. Verify shot counts first.

**ML.** Logistic regression, XGBoost, 2-layer MLP (PyTorch, for Captum). All train in seconds to minutes.

**Visualization** (Plotly/Streamlit or Jupyter widgets; linked views):
- (a) Pitch view: the shot, its perturbation path, and attribution glyphs along the path.
- (b) Small multiples of attributions per method and model.
- (c) A disagreement-vs-violation scatter; brushing selects shots on the pitch.
- (d) A reliability diagram per subgroup.

Why: the scientific question is spatial ("does the explanation behave sensibly as the shot moves?"), so the pitch view is essential, not decorative.

**Evaluation.**
- Pre-registered expected behaviours.
- Violation rates per method and model with bootstrap CIs.
- Does disagreement predict violation (AUROC vs chance)?
- Does the planted-leak control get flagged (detection rate)?
- Multiple seeds.
- Report top-k and rank measures, not just L2.

**Risk.**
- (1) Every method agrees on distance and angle, giving a trivial null result. Mitigation: focus the constraints on the *context* features, and rely on the positive control.
- (2) "Physics ground truth" is only partial: real xG isn't perfectly mirror-symmetric (footedness). You must choose constraints that are defensible.
- (3) The overlap with the grader's work must be framed as an extension, explicitly.

**Compute.** MacBook is sufficient; HPC is optional for Rashomon sweeps and seeds.

**Rough hours.** Data 6, models 4, reproduction 6, perturbations + metrics 8, viz 8, write-up and presentation 8, for about **40 h**. Tight but feasible.

**Course fit.** It directly concerns black-box interpretation, model assessment and visual comparison of explanations, and it evaluates a VIS tool's premise.

**Publication path.** You would need to add:
- A second domain task (passes) and a Rashomon analysis
- A formal violation taxonomy
- A comparison against Visagreement's and MOUNTAINEER's own metrics
- A small expert study

Plausible venues: the MLSA workshop (ECML-PKDD), an XAI workshop (xAI World Conf., XKDD), or a VIS short paper. A conversation with the Silva/Nonato group is a natural route; SPECULATION.

---

### Candidate 2: Do soccer model explanations depend on the evaluation protocol?

**Research question.** Holding features and model class fixed, how much do (i) test metrics and calibration, and (ii) global and local explanations of VAEP and xG models change across split regimes, and which features absorb the change? The regimes are random action-level, grouped-by-match, and leave-one-tournament-out. Can a linked visual view expose leakage-driven attributions?

**Why it might matter.** Sports data are not i.i.d. (Davis 2024). If explanations published from random-split models differ from properly-split ones, then some published "insights" are protocol artefacts.

**Evidence for the gap.**
- Davis 2024 prescribes splits but has no explanation protocol (FACT).
- Peters 2026 shows *feature* leakage changes SHAP (FACT, abstract).
- Tsai 2026 covers *domain shift*.
- Random splits appear in Mead 2023 and TacticAI 2024.
- socceraction's example metrics are in-sample (FACT, notebooks).
- No paper varying only the split was found (search-limited).

**Closest prior work to reproduce.**
- socceraction VAEP on StatsBomb (public notebooks), re-run with a proper split.
- Plus Robberechts & Davis 2020's evaluation protocol (Brier + calibration).
- Optionally, Calibrate's subgroup reliability view (Silva lab).

**Minimum viable project.**
1. Build VAEP (scores/concedes classifiers, XGBoost) and one xG model on StatsBomb tournaments.
2. Train under 3 split regimes, with 5 seeds each.
3. Report Brier/log-loss/ECE (expected calibration error) and reliability diagrams.
4. Measure global SHAP rank change (Kendall τ, top-k Jaccard).
5. Measure per-action attribution change on a *shared* held-out tournament.
6. Show which features gain importance under leakage.

**Stronger version.**
- Add temporal splits across multiple seasons (Wyscout 2017/18 or StatsBomb league seasons).
- Add a planted-leak feature as a control.
- Show that player ratings derived from VAEP change rank under split regime.

**Dataset.** StatsBomb open tournaments (and optionally 2015/16 big-5 seasons, for match-grouped splits with more data); Wyscout public (CC BY) as a replication set.

**ML.** socceraction VAEP (XGBoost) plus logistic regression xG. Laptop-scale.

**Visualization.**
- (a) Bump chart of feature-importance rank across regimes.
- (b) Reliability diagrams per regime and subgroup (Calibrate-style).
- (c) Pitch heatmaps of the per-zone attribution difference between regimes.
- (d) A possession timeline of VAEP values under each regime (Baseball-Timeline-style static summary).

Why: the finding is a *difference*, and paired views make differences visible.

**Evaluation.** Pre-specified effect-size thresholds (e.g., τ < 0.8 between regimes counts as "materially different"), seed variance as the noise floor, and bootstrap CIs.

**Risk.**
- (1) Null result for xG. Say so up front; that is still a finding.
- (2) The VAEP effect may be "obvious leakage". The contribution must be *which explanations move and why* (e.g., time-in-possession or location features absorbing the label overlap).
- (3) The visualization story is thinner than Candidates 1 and 4 unless the linked views are well designed.

**Compute.** MacBook.

**Rough hours.** Data 6, reproduction 6, regimes and seeds 6, explanation metrics 6, viz 8, write-up 8, for about **40 h**. The lowest-risk candidate.

**Course fit.** Model assessment (Sept 22), black-box interpretation, and interpretable ML lectures; calibration visualization is Silva-lab territory.

**Publication path.** You would need to add:
- Multiple tasks (xG, VAEP, pass success)
- Multiple data sources (StatsBomb + Wyscout)
- A reanalysis of a published model's claims under proper splits
- Guidelines

Natural venues: MLSA @ ECML-PKDD, JQAS (Journal of Quantitative Analysis in Sports), or *J. Sports Analytics*. There is a clear methodological-note format.

---

### Candidate 3: Ablation-tested faithfulness of LLM wordalisations of soccer models

**Research question.** When an LLM narrates an xG model's contributions (the Rahimian et al. pipeline, run locally with gpt-oss-20b):
- (i) How often does the narrative misstate sign or rank?
- (ii) How often does it credit feature groups the model does not use? This is measured by retraining the model without the group while keeping the group's vocabulary available.
- (iii) Are errors concentrated where the model contradicts football intuition?
- (iv) Does a claim-to-contribution linked view let a reader catch errors?

**Why it might matter.** Soccer is a domain where narrators hold strong priors, and "defensive pressure" or "goalkeeper positioning" are exactly the words an LLM will reach for. Coaches may trust fluent text over bars (Marusich 2026; Feustel 2025).

**Evidence for the gap.**
- Rahimian checks 2 features with a self-judge (FACT).
- Ichmoukhamedov drops hallucinated features from scoring and uses only synthetic conflicts (FACT).
- No ablation-based absent-feature test was found.
- Counter-evidence: He & Martens 2026 flag absent features as a critic signal. Read it carefully; it may narrow this further.

**Closest prior work to reproduce.** Rahimian et al. via `shotsGPT` (GPL-3.0, StatsBomb open data), ported to Ollama. Plus Ichmoukhamedov et al.'s RA/SA/VA metrics (code public).

**Minimum viable project.**
1. Reproduce the Case 2 vs Case 4 comparison on one competition.
2. Add RA/SA extraction.
3. Run 2 feature-group ablations (goalkeeper features; pressure features).
4. Compute the absent-feature mention rate using a **deterministic lexicon** (not LLM self-judging).
5. Stratify by prior agreement.

**Stronger version.**
- Add a second narrator model (a different family) and an XGBoost model with TreeSHAP (Rahimian's stated open problem).
- A small reader study using the linked view.

**Dataset.** StatsBomb open data (as in shotsGPT).

**ML.** Logistic regression xG (existing) plus gpt-oss-20b via Ollama. The M1 Pro with 32 GB fits it; SPECULATION: about 5–30 s per narrative. The HPC can batch.

**Visualization.** Narrative text with each extracted claim highlighted and linked to its contribution bar and to the pitch. Mismatches are flagged. A corpus-level matrix shows feature × error type.

**Evaluation.**
- Error rates with CIs.
- Extractor validated on scrambled tables plus a manual check of about 50 narratives.
- Ablation vs non-ablation difference.

**Risk.**
- (1) Incremental over Ichmoukhamedov; a fast-moving area with scoop risk.
- (2) Local LLM quality: Explingo's 7B narrator was poor. Is that a finding or a confound?
- (3) **Weakest visualization contribution**; the project could drift into NLP evaluation.
- (4) The shotsGPT port may take longer than expected (old `openai==0.28` API).

**Compute.** MacBook for development; HPC for batch generation (tens of GPU-hours at most).

**Rough hours.** Port 5, reproduction 6, ablations 5, extraction and validation 8, viz 6, write-up 8, for about **38 h**.

**Course fit.** The Nov 3 NLP/LLM visualization lecture, and interpretable ML. The weakest of the four on "visualization is scientifically necessary".

**Publication path.** You would need to add:
- Multiple narrators, including frontier models if an API becomes available
- Multiple soccer tasks
- A human reader study

Plausible venues: an HCXAI (CHI workshop) or XAI workshop, or JSA as a follow-up to Rahimian.

---

### Candidate 4: Which tactical claims survive? A reliability audit of 2D projections of soccer situation embeddings

**Research question.** When 2D projections of learned soccer situation embeddings are used to make tactical claims, which claims survive changes in encoder training seed, projection method, projection hyperparameters and projection seed? And do the surviving structures agree with objective event-data proxies that are *not* encoder inputs?
- Embeddings: a reproduction of Tang et al. 2023 on open 360 frames.
- Example claims: "counter-attack situations form a distinct cluster"; "cluster X lies between Y and Z".
- Projection methods: UMAP, t-SNE, PCA, PaCMAP.
- Proxies: StatsBomb `play_pattern`, possession outcome.

**Why it might matter.** Soccer representation papers already read tactics off t-SNE and UMAP pictures (EventGPT, Baron, Tang). VIS research says this is often unjustified (Jeon 2025). Nobody has audited it for sports.

**Evidence for the gap.**
- No sports DR-reliability study was found (search-limited).
- Tang chose k by interpretability.
- Baron's properties are circular.
- Counter-evidence: Atzberger 2024 already did large-scale sensitivity analysis for text, so the method is not new.

**Closest prior work to reproduce.** Tang et al. `FootballSituation` (MIT), retrained on open 360 (WC22 + Euro20 + Euro24). Alternatively, reproduce Baron et al.'s UMAP figure.

**Minimum viable project.**
1. Train the autoencoder with 3 seeds.
2. Run a DR grid: 4 methods × 3 hyperparameter settings × 5 seeds.
3. Compute ZADU metrics (trustworthiness/continuity, steadiness/cohesiveness, label-based T&C on proxies).
4. Measure cluster stability (bootstrap Jaccard; ARI across runs).
5. Test 3–4 pre-specified "tactical claims" for survival.

**Stronger version.**
- Separate the three sources of instability (encoder vs projection vs clustering in high-dimensional space vs 2D).
- Add a non-learned hand-feature embedding baseline.
- Validate on SkillCorner/IDSSE tracking.
- Add the point-wise reliability overlays (GhostUMAP2, scDEED-style).

**Dataset.** StatsBomb 360 (tournaments; about 150–200 matches, many thousands of frames). Mind the anonymity and visible-area censoring.

**ML.** Small CNN autoencoder (Tang's code) plus sklearn/umap-learn/openTSNE/PaCMAP plus `zadu`.

**Visualization.**
- (a) Small multiples of projections across seeds and hyperparameters.
- (b) Point-wise stability overlay.
- (c) Linked pitch view of the freeze-frames behind a brushed region.
- (d) A "claim checker" panel showing each claim's survival rate.

This is the most visualization-native candidate: the object of study *is* the visualization.

**Evaluation.** Survival rates per claim with CIs; proxy agreement in high-dimensional vs 2D space (to separate "no structure" from "projection artefact", following van der Hoorn 2025); and comparison with the hand-feature baseline.

**Risk.**
- (1) "t-SNE/UMAP are unstable" is already known, so the contribution must be the *soccer claim audit*.
- (2) Encoder training on anonymous, censored 360 may be finicky; Tang's code is from 2023.
- (3) Defining "a claim survives" is a judgment call; pre-register it.
- (4) Proxy labels may not form clusters even in high-dimensional space ("classes are not clusters").

**Compute.** HPC is useful (encoder seeds, DR grid); still modest (hours of GPU/CPU).

**Rough hours.** Data 6, reproduction of Tang 8, DR grid + metrics 8, claim tests 4, viz 8, write-up 8, for about **42 h**. The highest schedule risk.

**Course fit.** Strongest: Oct 13 clustering and Oct 20 DR lectures; it critiques projection use, which is core VIS research.

**Publication path.** You would need to add:
- Multiple representation models (Tang + a transformer event model)
- Tracking data
- A catalogue of published soccer DR claims re-tested

Plausible venues: a VIS short paper, EuroVis short, or the MLSA workshop.

---

## 12. Final comparison table

The comparison is split into two tables that share row labels, so each stays readable:
- **12a** uses the columns requested in the original brief.
- **12b** adds risk and reward columns (course fit, research upside, technical/data risk, visualization burden).

### 12a. Evidence and scope

| Candidate question | Evidence of gap | Closest prior work | Public data/code | Reproduction target | Minimum viable extension | Evaluation clarity | Visualization fit | Solo feasibility | Main failure mode | Evidence still needed |
|---|---|---|---|---|---|---|---|---|---|---|
| **1. Physics-anchored evaluation of explanation disagreement (xG)** | 0/25 soccer papers test local agreement or stability; Visagreement's disagreement–quality link untested for lack of ground truth | Visagreement, MOUNTAINEER (Silva lab), Tsai 2026, Cefis & Carpita 2024, Conti 2026 | StatsBomb open (yes); Visagreement repo (yes, small, no license file); shap/captum/OpenXAI (yes) | Visagreement on its own data → on an xG MLP | 3 semantic perturbation tests + disagreement-predicts-violation test + planted-leak control | Medium-high (pre-registered constraints; AUROC; control detection) | High | Good (~40 h) | Everything agrees on distance/angle, giving a trivial result | Read Cefis & Carpita; read Visagreement TVCG full text; check the MLSA and xAI-conference proceedings for soccer XAI evaluation |
| **2. Split-regime dependence of soccer model explanations (VAEP, xG)** | Davis 2024 has no explanation protocol; Peters 2026 feature-leakage only; random splits in published models | Davis 2024, Peters 2026, Tsai 2026, Calibrate | StatsBomb + Wyscout (yes); socceraction (yes) | socceraction VAEP with a proper split + Brier/calibration protocol | 3 regimes × 5 seeds; global and local SHAP change; reliability diagrams | **High** (paired comparisons, seed noise floor) | Medium-high | **Very good** (~40 h) | Null result for xG / "obvious" for VAEP | Peters 2026 full text (did they vary splits?); a sweep for "cross-validation SHAP sports" |
| **3. Ablation-tested faithfulness of LLM wordalisations** | Rahimian checks 2 features by self-judge; Ichmoukhamedov drops hallucinated features, synthetic conflicts only | Rahimian 2025, Ichmoukhamedov 2024, He & Martens 2026, Explingo | StatsBomb (yes); shotsGPT (GPL); SHAPnarrative-metrics (yes); gpt-oss-20b local | shotsGPT Case 2 vs Case 4 | Feature-group ablation + lexicon-based absent-feature rate + prior-stratified errors | Medium (extractor reliability) | Medium-low | Good (~38 h) | Incremental over 2024–26 work; drifts into NLP evaluation | He & Martens 2026 full text; the JSA version of Rahimian (human evaluation added?) |
| **4. Reliability audit of tactical claims from 2D projections** | No sports DR audit found; soccer papers read t-SNE/UMAP visually (Baron, EventGPT, Tang) | Jeon 2025/26, Atzberger 2024, ZADU, Tang 2023 | StatsBomb 360 (yes, anonymous/censored); FootballSituation (MIT); zadu (MIT) | Tang et al. retrained on open 360 (or Baron UMAP) | Encoder-seed × DR-grid audit of 3–4 pre-specified claims vs non-input proxies | Medium (what counts as "surviving" must be pre-registered) | **Very high** | Medium (~42 h; highest schedule risk) | "UMAP is unstable" already known; encoder won't train cleanly | Citation sweep of ZADU, Atzberger, Jeon for sport/trajectory; confirm Tang's code runs on open 360 |

### 12b. Risk and reward

These ratings are qualitative. Each one comes with its reason, and the reasons summarize §11. There is still no overall score and no winner, as the original brief asked.

"**Visualization burden**" is the front-end engineering effort. It is different from "Visualization fit" in 12a, which asks how scientifically necessary the visualization is.

"**Best suited if…**" replaces a "recommended use" ranking. It says which priorities a candidate serves, not which candidate is best.

| Candidate | Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|---|
| **1. Physics-anchored evaluation of explanation disagreement (xG)** | **Excellent**. Covers black-box interpretation (Oct 6), TDA/Mapper (Nov 10) and interpretable ML (Nov 24). Tests an open question from the instructor's own TVCG tool. You must present this overlap as an extension, not a re-application. | **Moderate–High**. It tests a named open question (Visagreement: does disagreement track correctness?) in a domain with partial ground truth. Tempered by Conti 2026 and Higgs 2026, which use domain knowledge as a reference in other fields. | **Low–Moderate**. The models are standard. The risks are that the Visagreement repo is unmaintained (2023, no license file) and that defining a defensible "violation" takes care. | **Low**. Every StatsBomb open shot has a freeze-frame. Count shots per tournament before committing. | **Moderate**. Pitch view with perturbation paths, attribution small multiples, and a linked disagreement-vs-violation scatter. Doable in Plotly/Streamlit. | Medium–High | You want the closest tie to Silva's line of work and a clean XAI-evaluation question. |
| **2. Split-regime dependence of soccer model explanations (VAEP, xG)** | **Very good**. Covers model assessment (Sept 22) and interpretable ML, in the lineage of Calibrate (Silva lab). It is less visualization-native than 1 or 4. | **Moderate**. A clear methodological note for the sports-analytics audience (MLSA, JQAS). The upside is capped by the risk of a null result for xG. | **Low**. Standard pipeline. The main friction is socceraction's pinned dependencies (Python <3.13, numpy <2). | **Low**. StatsBomb open data plus Wyscout (CC BY) as a replication set. | **Low–Moderate**. Mostly static paired views: rank bump charts, reliability diagrams, pitch difference maps. | **High** | You want the lowest-risk path and accept that the visualization supports the finding rather than being the object of study. |
| **3. Ablation-tested faithfulness of LLM wordalisations** | **Good**. Covers the Nov 3 NLP/LLM lecture, but the visualization's role is the weakest of the four. | **Low–Moderate**. Incremental over Ichmoukhamedov 2024 and He & Martens 2026 in a fast-moving area, so the scoop risk is real. | **Moderate**. Porting shotsGPT from `openai==0.28`/Azure to Ollama. Local-model parsing reliability (Explingo's 7B scored 0.98/4). Generation throughput. | **Low**. StatsBomb open data, as used by shotsGPT. | **Moderate**. A custom claim-to-contribution linking view (text spans ↔ bars ↔ pitch). | Medium | LLM/NLP portfolio value matters most to you, and you accept a thinner visualization contribution. |
| **4. Reliability audit of tactical claims from 2D projections** | **Excellent**. Covers clustering (Oct 13) and DR (Oct 20). The object of study *is* the visualization. | **Moderate**. A domain application of a mature VIS methodology. The value is in the soccer claim audit and in using proxies that are not model inputs. Fits a VIS/EuroVis short paper or MLSA. | **Moderate–High**. Tang et al.'s code is from 2023. The encoder must train on anonymous, visible-area-censored 360 frames. The DR grid and the definition of a "claim" add schedule risk. | **Moderate**. 360 frames are anonymous and limited to the camera view, and cover tournaments only. Tang's original EPL 2021/22 data is not open, so you can't match their numbers exactly. | **Moderate–High**. Small multiples across seeds and hyperparameters, point-wise stability overlays, a linked pitch view, and a claim-checker panel. | Medium | You want the most visualization-native project and have some slack on the HPC and in your schedule. |

**Rows present in the earlier ChatGPT matrix but not shortlisted here**, with where they are handled:
- **Tracking + tactical states:** §10. Needs proprietary data, and labels are subjective. A limited version could use SkillCorner's open data, which includes vendor phase-of-play labels (20 matches; §8).
- **Pass/shot explanation system:** folded into Candidate 1. A system without a research question is on the §10 list.
- **Possession-value explanations:** VAEP is covered in Candidate 2.
- **Model comparison visualization:** overlaps with MOUNTAINEER.
- **Passing networks, player UMAP:** too conventional on their own. Player UMAP is covered by Candidate 4's critique.
- **Full video analytics:** §10.

---

## 13. If I were independently doing this literature review, here is exactly what I should verify

These are the checks most likely to *change* conclusions. They are listed without the conclusion you "should" reach.

1. **Read the Visagreement TVCG full text** (NYU library), not just the author's DC summary. What exactly were its quality metrics, and what did it conclude about disagreement vs correctness? Does it mention ground-truth or domain-knowledge evaluation as future work?
2. **Search for 2025–2026 Silva/Nonato-group follow-ups** to MOUNTAINEER and Visagreement: Priscylla Silva's thesis, Vitória Guardieiro, Parikshit Solunke. Check whether they already extend to Rashomon sets or domain ground truth.
3. **Read Cefis & Carpita 2024 (*Statistics*)**. Is their cross-model xG explainability comparison local or global, quantitative or descriptive?
4. **Check PassAI's IEEE Access tables** (images in the PDF). Is there a random-removal baseline in the ROAR experiment?
5. **Read Ichmoukhamedov et al. 2412.10220 and He & Martens 2603.20003** closely. Is absent-feature hallucination already measured? Has 2412.10220 been published with added experiments?
6. **Get the JSA journal version of Rahimian et al. 2025**. Did it add human evaluation or broader accuracy checks compared with arXiv v1?
7. **Skim the tables of contents** of MLSA @ ECML-PKDD 2023–2025 (Springer CCIS), StatsBomb Conference 2023–2025 papers, and CVsports 2024–2026 for "explain", "SHAP", "stability", "UMAP", "t-SNE", "leakage".
8. **Run citation searches** (Google Scholar "cited by") on Jeon et al. 2506.08725, ZADU, Atzberger et al. 2407.17876 and GhostUMAP2 for any sports or trajectory application.
9. **Read Peters et al. 2026 (RQES) full text.** Do they vary the split scheme, or only the feature set? Did they use grouped CV throughout?
10. **Read Tsai et al. 2605.10796.** Confirm the analysis is global-only and match-level, and check its related-work section for soccer XAI-evaluation papers this review missed.
11. **Count shots in StatsBomb open data**, with and without 360, per tournament. That tells you whether Candidates 1 and 3 have enough samples for per-subgroup CIs.
12. **Try the Visagreement and FootballSituation repos** for 30 minutes each. Do they install and run? A broken reproduction target changes feasibility more than any paper does.
13. **Search the Davis/KU Leuven group's 2025–2026 output** (Robberechts, Van Roy, Bransen, Devos) for explanation evaluation or split-sensitivity work. They are the most likely group to have done Candidate 2 quietly.
14. **Check the 2026 course expectations** directly with Silva or the TA at the Oct 6 project discussion. Is a notebook/Streamlit linked-view tool acceptable? Is a solo project confirmed? Is reproducing the instructor's own tool welcomed or awkward?
15. **Check the Zhu 2026 OSF systematic-review protocol** for its inclusion criteria and planned completion date, as an independent count of soccer XAI papers against the N=25 here.

---

*AI-use disclosure for the course: this document was produced by Claude (Opus 5.5) with six parallel sub-agents running literature searches. Load-bearing claims were spot-checked by the lead agent against primary sources, as listed in §0.1. Per-slice notes with full search logs are in `lit_notes/`.*
