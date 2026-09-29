# Scan 10 (S10): American football tracking (NFL Big Data Bowl) x ML/VA

Scan date 2026-09-29. Labels: FACT (verified by me this session), AUTHOR CLAIM, SYNTHESIS, SPECULATION, unverified. Budget used: about 12 WebSearch, plus WebFetch, OpenAlex and `gh`. The arXiv API and Kaggle pages were unreachable (arXiv returned empty, Kaggle is JS-only and needs login). Anything that depends on a Kaggle page is therefore secondhand.

## 0. Headline

**The data licence is the dominant issue, and it is worse than "open".**

- FACT (official NFL Big Data Bowl Terms and Conditions text, retrieved through a search snippet; the direct page fetch returned 404, so treat it as one step removed): "Each Entrant agrees that it shall keep NGS Data strictly confidential and not transmit, duplicate, publish, redistribute, provide or communicate the data ... without the prior written consent of the Sponsor." Also: "Each Entrant shall destroy NGS Data in its possession following conclusion of this Contest." Also: entrants will not use, modify or create derivatives of NGS Data "for any purpose whatsoever other than to compete in this contest, unless expressly permitted otherwise by the Sponsor in writing".
- FACT (a per-edition Kaggle rule page I could not read): the search result for the Kaggle rules also says entrants may not make competition data available to non-participants.
- FACT (community practice): Yurko's CMU group publishes arXiv and journal papers on BDB data with GPL-3 code. Their repo README says: "Due to the terms and conditions, we are unable to publicly post the NFL Big Data Bowl sample tracking data. However, the data are available to download from the NFL Big Data Bowl 2021 Kaggle page" (`ryurko/nfl-ghosts`). So papers and code are tolerated, redistributing data is not, and users must fetch it from Kaggle.
- FACT: the 2019 data (91 games from 2017, weeks 1-6) sits in a public GitHub repo, `nfl-football-ops/Big-Data-Bowl` (about 420 MB, no LICENSE file, `license: null`). Official README says the contest is closed and calls for JQAS papers. This is the most "openly downloadable" file set, but it has no explicit licence.
- SYNTHESIS: this conflicts with the brief's "data must be openly downloadable, no credentialed data". Kaggle requires an account and rule acceptance, and the NFL terms say "use only to compete". Academic papers get published anyway, but a course project that shows the data in a demo sits in a grey zone. Ask the instructor before committing. An email to the NFL Football Operations BDB contact would be the clean fix (contact address: unverified).
- Mitigation: use only the 2019 GitHub release (no explicit licence, no Kaggle click-through), or do not redistribute anything and have the grader fetch the data themselves.

## 1. Sub-area (a): Data

Editions. Sources: secondhand summary repo `chasko-labs/nfl-big-data-bowl-2027/docs/history.md` (a personal notes repo, cross-checked with press releases in places; treat sizes and dates as unverified) plus the official NFL page and search results.

| Year | Theme | Data (per third-party notes, unverified unless marked) |
|---|---|---|
| 2019 | Open, pass offense and routes | 91 games, 2017 weeks 1-6. FACT: public GitHub. |
| 2020 | Rushing yards at handoff | Kaggle, train 2017-18, test 2019 |
| 2021 | Pass coverage | 2018 passing plays, about 2.17 GB across 20 CSVs |
| 2022 | Special teams | NGS plus PFF scouting, 2018-20 |
| 2023 | Pass rush and OL | 2021 season weeks 1-8, about 1.5 GB per week, plus PFF scouting |
| 2024 | Tackling | 2022 weeks 1-9 |
| 2025 | Pre-snap motion, formations | 2022 weeks 1-9, 16,124 plays, 136 games, `player_play.csv` with motion and coverage-assignment columns |
| 2026 | Two competitions. FACT: "Prediction" (predict per-player x,y per frame while the ball is in the air, $50k pool) and "Analytics" (University and Broadcast tracks). | Train 2023-24 seasons, scored on 2025 weeks 14-18 through an evaluation API. About 18k pass plays (per a preprint, unverified). |
| 2027 | Not announced | |

- FACT: tracking is 10 Hz for all 22 players plus the ball (x, y, s, a, o, dir, event, frameId). 2023 onward includes PFF scouting data, which is a separate proprietary layer.
- Ground-truth labels: 2025 `player_play` has PFF coverage assignments (man/zone, matchup ids). 2024 has tackles. 2026 has held-out future frames (only through Kaggle's API).
- Size is manageable: a few GB, and one or two weeks are enough for a 40 h project.
- Other open tracking: NBA SportVU 2015-16 logs on GitHub (`sealneaward/nba-movement-data`, labelled MIT, but the data provenance and NBA rights are dubious; I would not depend on it). The soccer open-data problem is already documented in the main review. No other open American football tracking exists that I found.

## 2. Sub-area (b): ML on BDB tracking

Prior work is dense and mostly statistical (Bayesian, random forest), with a smaller deep-learning branch.

Key 2023-2026 papers (code column is what I verified):

1. Yurko, Nguyen, Pelechrinis. "NFL Ghosts: defender positioning with conditional density estimation." arXiv 2406.17220 (revised June 2025), preprint, CC-BY. Random-forest conditional density gives "ghost" defenders as counterfactual baselines. Code: FACT, `ryurko/nfl-ghosts` (GPL-3). Data excluded, uses BDB 2021.
2. Nguyen, Yurko. "NFL step-and-turn." arXiv 2603.17866 (v-latest Sept 2026), preprint. Generative Bayesian model of step length and turn angle, simulates alternative ball-carrier movements. Data: BDB 2022 weeks 1-9. Code link: not on the abstract page (unverified).
3. Nguyen, Jiang, Ellingwood, Yurko et al. "Fractional tackles." arXiv 2403.14769, Scientific Reports 2025, peer-reviewed. Within-play tackle attribution. Code: unverified (repo search found none).
4. Song, Diewald, Siddiquee, Boomhower, Abdoo, Band, Lee. "Decoding Defensive Coverage Responsibilities in American Football Using Factorized Attention-Based Transformer Models." arXiv and Springer LNCS 2026 (likely a sports-analytics workshop volume; proceedings name unverified). Factorized attention (time x agent) predicts coverage assignments, matchups and targeted defender, about 89%+ accuracy per the abstract summary. Authors include the NFL NGS team (Mike Band, Keegan Abdoo). Code and data: unverified, probably not open because the data appear to be beyond the BDB release.
5. Michels, Bajons, Fischer. "Integrating Unsupervised and Supervised Learning for the Prediction of Defensive Schemes in American football." arXiv 2602.10784 (revised July 2026), preprint. HMM-derived features into elastic net and GBDT for man/zone. Code link: none on abstract page.
6. Any authors. "Toward Explainable Data and Sports Analytics: A Case Study on Pass Completion Prediction in American Football." The American Statistician 79(4), 2025, peer-reviewed. Small explainable feature set plus a one-equation physics-based model matches black-box performance (72% accuracy; ensemble 78%). Author list and code: unverified. Note that this is interpretable-by-design, not post-hoc explanation.
7. "PEP: a tackle value measuring the prevention of expected points." arXiv 2407.08508, preprint. Code: unverified.
8. Matthews and Nguyen et al. "STRAIN." arXiv 2305.10262, The American Statistician 2023. Pass-rush metric. Was a 2023 BDB finalist entry.
9. Kaggle BDB 2026 Prediction top solutions. FACT that writeups exist (a "1st Place Solution" writeup page was returned). Search snippets describe transformer variants (factorized spatio-temporal encoders, Gaussian NLL heads). Public GitHub repos with such solutions exist (for example `kaggler3000/nfl` "silver medal solution", `AceGiqMo/NFL-Big-Data-Bowl-2026`). Quality and licence vary, and the Kaggle terms for the code and the data are unread.
10. Stanford CS224W blog posts on GNNs for NFL pass-rush and pass prediction (Medium, not peer-reviewed). Evidence that GNN-on-BDB is a routine student project.

Saturation verdict for predictive modelling: **active and crowded.** Every year has hundreds of entrants (400+ in 2025 per third-party notes), so model novelty is hopeless in 40 h. But the field is mostly statistical, so "VA of the models" is thin (below).

## 3. Sub-area (c): Interpretability and VA on football tracking models

Queries run: (i) `Yingcai Wu OR Xiaolong Zhang visual analytics American football OR NFL`, (ii) `visual analytics American football tracking data IEEE VIS OR TVCG`, (iii) `explainability attention transformer NFL player tracking counterfactual defender Big Data Bowl`, (iv) `SHAP OR integrated gradients OR attention explanation faithfulness graph neural network football tracking NFL`, (v) `play similarity embedding NFL tracking`.

- FACT: I found no American-football VA paper in the VIS/TVCG/PacificVis tradition. Sports VIS (Yingcai Wu lab: table tennis, badminton, soccer ForVizor, VisCommentator, Sportify) is on other sports. This is absence in a search index, not proof.
- FACT: no paper found that measures explanation faithfulness or stability for an attention/GNN/transformer model on NFL tracking. The Decoding-Coverage transformer paper is a natural target (attention is presumably shown but not validated; unverified because the paper was paywalled to me).
- FACT: counterfactual "what if the defender were elsewhere" exists but not as a model-VA: NFL Ghosts and step-and-turn generate counterfactuals with statistical models and evaluate value. No interactive what-if tool found.
- Embeddings: play2vec (KDD 2019, `zhengwang125/play2vec`, code open, basketball/soccer) is the play-retrieval reference. I found no NFL play-embedding paper, and no reliability audit of projections of them. Thin evidence (one search), so verify before betting on it.
- SYNTHESIS: the "specific open question" stays the same as for soccer: explanation reliability of set/graph models. The cost is that a fast NFL result would also rely on a model trained on restricted data.

### Instructor-lab overlap (FACT unless marked)

- OpenAlex author sweep, Silva (A5003584200), 2019 onward with sports keywords: "Graph Neural Networks to Predict Sports Outcomes" (2021 IEEE Big Data, Silva is a co-author per OpenAlex; the arXiv version is 2207.14124; whether it is football is unverified), CS:GO valuing (2020), ggViz esports (CSCW 2022 / arXiv 2021), ESTA esports dataset (2022), "A Tracking System for Baseball Game Reconstruction" (arXiv 2020, the Baseball4D line), Calibrate (TVCG 2022), MOUNTAINEER (TVCG), T-Explainer (2024).
- No American football paper by Silva, Nonato or Miranda found (OpenAlex `football` search within Silva's works returned only the esports/sports items above). Nonato and Miranda author IDs: A5011424640, A5083220583 (a "Fábio Miranda" at UIC; identity match to the NYU-affiliated Miranda is unverified). I did not run separate football sweeps on them.
- Overlap risk if we do model-explanation VA: MOUNTAINEER, Visagreement, Calibrate, SUBPLEX (as in the soccer review). Calibrate is also a natural "black-box output" tool for tackle/completion probability calibration.
- Prior cohort: `gh search` found only 2022 Vis4ML repos (`SearidangPa/Vis4ML_FinalProject`, `janeadams/nba-vis4ml` is neural binary analysis, not basketball; `DanielKerrigan/vis4ml-class-notebooks`) and `Famveer/vis4ml` (2026, README not readable). No NFL/BDB project from this course found. A large number of Sept 2026 BDB GitHub repos exist (for example `jdpipping/model-stability`, "Model stability and uncertainty quantification across NFL Big Data Bowl prediction tasks", created 2026-09-16, README unread; and `Huerta9/nfl-coverage-predictor`). Someone else may be doing "model stability on BDB" right now.
- Notable: NYU students (Smit Bajaj, Viren Bhatia) were BDB 2024 finalists and Bajaj plus Vishakh Sandwar won 2025 (per third-party notes, unverified). NYU has a BDB culture. It is not a Silva-lab link.

## 4. Candidate research questions

RQ numbers are local to this file.

**RQ-S10-A (grade-safe): Reproduce a BDB coverage or completion model and audit attribution stability.**
- Reproduction target: NFL Ghosts (`ryurko/nfl-ghosts`, GPL-3, random forest CDE, leave-one-week-out CV; data BDB 2021 from Kaggle). Alternative target: "Toward Explainable..." (code unverified).
- Minimal extension: check how much ghost-defender EPV-difference rankings change with feature set, model class (RFCDE vs. gradient boosting) and CV scheme, with a Plotly linked view (play animation, ghost overlay, ranking table).
- Evaluation: rank correlation of defender ratings across model variants, split-half reliability across weeks.
- Falsification check: queries (iii), (iv) above found no such audit; GitHub `jdpipping/model-stability` is close in spirit and must be inspected. Risk: this is closer to sports stats than VisML.
- Feasibility: about 35 h. Compute is trivial. Front-end is a Plotly play viewer. Licence risk stays (0).

**RQ-S10-B (higher upside): Do attention weights or attribution scores of a set/graph transformer on tracking data reflect causal dependence?**
- Reproduction target: train a small graph/transformer model on BDB 2025 (coverage type from pre-snap motion) or a 2026 Kaggle public solution. Code for the Decoding-Coverage paper is unverified, so the safest reproduction is a public Kaggle/GitHub baseline.
- Minimal extension: compare attention, gradient x input and Shapley-style ablation of players (delete a defender) on faithfulness (deletion curves) and permutation stability. What-if edits: move one defender and see which explanation changes.
- Falsification: queries (iii) and (iv) returned nothing on NFL. General attention-vs-ablation faithfulness is saturated (Jain and Wallace 2019, Wiegreffe and Pinter 2019, from memory, unverified here). The novelty is the tracking-set domain, which the soccer review already called weak ("generic apply known XAI to a new domain").
- Risks: heavy model engineering, GPU time, and overlap with Silva-lab explanation work.

**RQ-S10-C (speculative): reliability audit of 2D projections of NFL play embeddings** (route/formation similarity, play2vec-style).
- Reproduction target: `zhengwang125/play2vec` (open code, but not BDB), or route clustering by the 2019 BDB winners (Sterken's `nsterken/Big-Data-Bowl-1` code exists per search results).
- Extension: apply Jeon et al.-style DR reliability checks (as in the soccer review's Candidate 4) to embeddings of plays; check cluster claims.
- Falsification: one embedding search, and the soccer review already found this route "mostly done or unreliable". This is a domain transplant of a mature methodology.

## 5. Feasibility, publication path, red flags

- Feasibility SYNTHESIS: technically very feasible. Tracking data is small (a few GB), 22 agents give a natural set/graph structure, labels exist (coverage assignments in 2025, tackles in 2024, completion), and the student has strong PyTorch. Front-end burden is a Plotly/Streamlit field animator.
- Publication path: MIT Sloan Sports Analytics Conference research papers, NESSIS, JQAS, The American Statistician, the CMU sports analytics workshop, or the VIS4Sports / MLSA (ECML-PKDD workshop) track; the VIS short-paper route is harder because a VIS reviewer will want a design-study novelty. SPECULATION: a VIS workshop (VISxAI) could accept a well-scoped faithfulness study.
- Red flags:
  1. **Licence**: NFL terms restrict use to the contest and require destroying data afterwards. Publishing is tolerated in practice but data cannot be shipped, so the grader and any reader must download from Kaggle themselves. This conflicts with "openly downloadable".
  2. Saturation: hundreds of Kaggle entrants a year, and 2026's Prediction competition just produced a wave of transformer solutions.
  3. Overlap with an active public repo (`jdpipping/model-stability`) and with instructor-lab tools (MOUNTAINEER, Visagreement, Calibrate).
  4. The 2023 onward data includes PFF scouting data, which is a separate rights holder.
  5. Kaggle rule pages for the years of interest were not readable by me. Per-edition rules could differ (2026 Prediction especially).

## 6. Triage table

| Sub-area | Best RQ | Novelty evidence (1 line) | Feasibility | Grade-safety | Publication upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) Data and licence | n/a | Terms: contest-only use, destroy after, no redistribution | n/a | Low (licence) | n/a | Only to resolve the licence question | Blocks everything else; ask instructor and NFL |
| (b) ML on BDB | RQ-S10-A (Ghosts reproduction plus stability audit) | Ghosts code open; no stability audit found in 2 queries | High | Med (data risk) | Low-Med | Maybe | Clean reproduction, but sports-stats rather than VisML |
| (c1) Explanation faithfulness on tracking transformers | RQ-S10-B | No NFL paper found in 2 queries; general method saturated | Med | Med-Low | Med | Maybe | Same weak "domain transplant" novelty as soccer, plus licence risk |
| (c2) Counterfactual / what-if VA (move a defender) | RQ-S10-B what-if part | Ghosts and step-and-turn are statistical counterfactuals; no interactive tool found | Med | Med | Med | Maybe (best VisML story) | Unusual for VisML; needs a trustworthy generative model |
| (c3) Play-embedding DR reliability | RQ-S10-C | No NFL embedding audit found (1 search) | Med | Med | Low-Med | No | Soccer review already flagged this as domain application of Jeon et al. |
| (d) Instructor-lab football | none | No Silva/Nonato/Miranda football paper found; baseball is theirs | n/a | n/a | n/a | No | No overlap to reproduce, no prior cohort project |

## 7. Bottom line

The NFL data is much richer than open soccer tracking, but its terms make it a poor fit for a course that needs openly reproducible data and grader-visible demos. If the instructor accepts "download from Kaggle yourself, no redistribution" (as CMU papers do), then RQ-S10-A/B is a workable reproduction-plus-extension with a real, but not novel, what-if VA story. Otherwise drop the slice.
