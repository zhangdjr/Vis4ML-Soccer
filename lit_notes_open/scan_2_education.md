# Scan 2 — Education ML + Visual Analytics / XAI

Scope: (a) knowledge tracing (KT) interpretability, (b) LLM automated grading reliability, (c) student-success/dropout XAI + fairness, (d) LA dashboards (brief). Today: 2026-09-28. ~19 WebSearch calls used (of shared 45-call budget) plus WebFetch/curl verification.

---

## (a) Knowledge tracing — interpretability, skill-representation viz, explanation reliability

**Instructor-lab overlap:** None found. Targeted searches for Silva/Nonato/Miranda + knowledge tracing returned nothing; dblp direct fetch was blocked by a bot-wall (Anubis), so this is **not fully verified** — recommend a manual Google Scholar check before committing. [SYNTHESIS: likely low overlap, but unconfirmed by dblp.]

**Key 2023–2026 papers:**
1. Liu et al., **pyKT**, NeurIPS 2022 Datasets & Benchmarks (arXiv 2206.11460) — peer-reviewed, standardizes preprocessing + 21 DLKT models over 7–9 datasets. Code open: `github.com/pykt-team/pykt-toolkit` (FACT, verified reachable, HTTP 200).
2. Bai et al., **"A Survey of Explainable Knowledge Tracing"**, arXiv 2403.07279 / *Applied Intelligence* 2024 — peer-reviewed. Taxonomy: ante-hoc vs. post-hoc explainability; runs contrast/deletion experiments; explicitly states **"current evaluation methods for explainable knowledge tracing are lacking."** (FACT — this is the key gap statement for this sub-area.)
3. **simpleKT**, arXiv 2302.06881 (2023) — tough-to-beat attention baseline, code open, widely used as pyKT baseline.
4. **AlignKT**, arXiv 2509.11135 (2025, preprint) — ideal-state alignment for explicit knowledge-state modeling.
5. Question-centric multi-experts contrastive learning framework, arXiv 2403.07322 (2024, preprint) — claims joint accuracy + interpretability gain.
6. Knowledge-ontology-enhanced explainable KT, *ScienceDirect* 2024 (peer-reviewed) — ante-hoc, ontology-regularized.
7. "Explainable KT via Probabilistic Embeddings and Pattern-based Reasoning", arXiv 2605.09369 (2026, preprint) — very recent, post-hoc pattern extraction.
8. **XES3G5M**, OpenReview/NeurIPS 2023 D&B — KT benchmark dataset with auxiliary info (Q-matrix, text), useful for skill-representation visualization work.

**Saturation verdict: ACTIVE, not saturated on evaluation.** Many papers propose new KT architectures and *claim* interpretability (crowded, weak novelty if you just add another one), but the 2024 survey itself names evaluation-of-explanations as an open gap — a narrower, genuinely thinner niche.

**Open data (verified):**
- **ASSISTments 2009/2015** — free for research, request form at `sites.google.com/view/assistmentsdata/home` (verified reachable, HTTP 302 to a Google-hosted form; exact license terms not independently re-read this session — standard practice is "free for research/education").
- **EdNet** (Riiid) — public via `github.com/riiid/ednet` (FACT, verified HTTP 200); ~130M+ interactions, CC-style open release per repo.
- **Junyi Academy** — public on Kaggle, `kaggle.com/datasets/junyiacademy/...` (verified HTTP 200).
- **Statics2011** — via CMU PSLC DataShop; URL not verified this session (curl failed on shell quoting, not a content check) — **needs re-verification**.
- **Eedi / NeurIPS 2020 Education Challenge** — anonymized (QuestionIds/UserIds), distributed via CodaLab competition page; >20M multiple-choice responses. License/access terms not independently confirmed this session — flag as **unverified**.

**Candidate research questions:**
1. **Faithfulness/reliability audit of post-hoc explainers on pyKT models.** Reproduction target: pyKT toolkit (Liu et al. 2022) + the contrast/deletion protocol from the 2024 survey. Minimal extension: apply a Visagreement-style *disagreement metric* (cross-explainer agreement, not just single-explainer faithfulness) across SHAP/attention/LIME explanations of DKT/SAKT/AKT trained on ASSISTments2009, and link it to an interactive Plotly/Streamlit view. Eval: deletion/insertion-AUC faithfulness + pairwise agreement (Spearman/Jensen-Shannon, following Swamy et al. 2022's methodology — see (c)). **Falsification check:** searched "knowledge tracing interpretability survey 2024 2025", "pyKT reproducibility... unreliable evaluation" — found the 2024 survey does contrast/deletion but *not* multi-explainer disagreement; no VA tool for this found. Not exhaustively falsified — one more targeted search recommended before committing.
2. **Visualize learned skill embeddings** (t-SNE/UMAP/DR) across KT model variants (simpleKT/AKT/SAKT) vs. the ground-truth Q-matrix, using XES3G5M's auxiliary info as a validation signal. Directly ties into the course's DR week (Oct 20) and DL-viz week (Oct 27).

**Feasibility in ~40h:** Good fit. Data is small-to-medium (ASSISTments2009 fits on a laptop), pyKT handles preprocessing, DKT/SAKT/AKT train fine on a single 11GB GPU. Front-end via Streamlit/Plotly matches stated skill level (weak D3 is not a blocker).

**Publication path:** LAK/EDM short paper; possibly VIS short paper if the disagreement-visualization angle is emphasized.

**Red flags:** Crowded "new architecture + interpretability claim" literature — must frame explicitly around *evaluating explanation reliability*, not proposing a new KT model, to keep novelty defensible. dblp overlap check incomplete (see above).

---

## (b) LLM automated grading / feedback reliability

**Instructor-lab overlap:** None found for Silva/Nonato/Miranda specifically on LLM grading.

**Key 2023–2026 papers (very high output rate — see saturation note):**
1. Yavuz et al., *British Journal of Educational Technology* 2025 — peer-reviewed, LLM EFL essay grading reliability/validity, rubric-based.
2. "Exploring LLM Autoscoring Reliability... Generalizability Theory", arXiv 2507.19980 (2025, preprint).
3. "Assessing Reliability and Validity of LLMs for Automated Assessment of Student Essays in Higher Education", arXiv 2508.02442 (2025, preprint).
4. "Opening the blackbox of LLM-based AES: feature weighting patterns and score validity", *ScienceDirect* 2026 (peer-reviewed, very recent).
5. "Investigating first-language bias in LLM-based AES... TOEFL essays", arXiv 2607.14605 (2026, preprint) — bias/fairness angle.
6. Kortemeyer (2024) — benchmarks GPT-4 on SciEntsBank; strong overall, struggles on contradictory/technical cases.
7. "Estimating LLM Grading Ability and Response Difficulty... IRT", arXiv 2605.00238 (2026, preprint) — psychometric framing of grader reliability.
8. **S-GRADES**, arXiv 2603.10233 (2026, preprint) — generalization of grading models across evaluative settings.
9. **iLLuMinaTE**, arXiv 2409.08027 (2024, preprint) — LLM-XAI framework for actionable student feedback (bridges (b) and (c)).

**Saturation verdict: HEAVILY SATURATED on "is-it-reliable" studies** — the arXiv numbering alone (2601/2602/2603/2605/2607-series, i.e. Jan–Jul 2026) shows several new preprints per month, mostly psychometric (IRT, generalizability theory) or bias-audit framed. **Thinner niche:** none of the papers found combine this with an *interactive visual-analytics tool* for exploring grader disagreement (by rubric dimension, LLM vs. LLM, LLM vs. human) — existing work is statistical/tabular, not VA. That gap is the most defensible angle here.

**Open data (verified):**
- **ASAP/ASAP-AES** — Kaggle competition (`kaggle.com/c/asap-aes`, verified HTTP 200); competition-data terms apply (research use standard, **exact commercial-reuse terms not re-confirmed — check Kaggle rules page**).
- **PERSUADE 2.0** — CC BY-NC-SA 4.0 (per ScienceDirect description); mirrored at `github.com/scrosseye/persuade_corpus_2.0` (FACT, verified HTTP 200) and Kaggle. ~25,000 argumentative essays, discourse annotations.
- **SciEntsBank / Beetle** (from SemEval-2013 Task 7) — standard ASAG research-use benchmarks; precise license not re-verified this session.
- **Mohler dataset** — public, research use, University of North Texas short-answer set.

**Candidate research questions:**
1. **Visual analytics for LLM-grader disagreement with open-weight models.** Reproduction target: a recent reliability study (e.g., the Generalizability-Theory paper, or Yavuz et al. 2025) reproduced on ASAP/PERSUADE using locally-runnable models (gpt-oss-20b, Qwen ~27B via Ollama — fits "maybe no paid API" constraint). Extension: a Streamlit/Plotly tool exposing disagreement conditioned on rubric dimension, essay length/topic, and the first-language-bias axis from arXiv 2607.14605. **Falsification check:** ran "LLM-as-grader ... disagreement visual analytics 2024 2025" and "SciEntsBank Mohler disagreement" — found only statistical/IRT-based agreement analyses, no interactive VA tool. Not exhaustively falsified given budget; treat as a plausible but not fully confirmed gap.
2. Rubric-wording sensitivity: reproduce a prompt-paraphrase instability finding and visualize score drift across paraphrases/models.

**Feasibility in ~40h:** Medium. NLP/LLM skill is only moderate per constraints, and prompt-engineering iteration for stable rubric-following is fiddly, but no activation-level work is needed (black-box scoring only) so the 11GB-GPU / Ollama setup suffices. Streamlit/Plotly front end fits skills well.

**Publication path:** EDM/LAK short paper; possibly an NLP-education workshop.

**Red flags:** The single biggest risk is being scooped between now and December — this sub-area produces new preprints almost weekly. Needs airtight differentiation (VA angle + open-weight-model angle) or it reads as "yet another reliability study." Grading ground truth is itself noisy (human rater disagreement), a known confound this literature acknowledges but doesn't solve.

---

## (c) Student success / dropout prediction — XAI, fairness, SHAP disagreement

**Instructor-lab overlap: STRONG and directly on-point — the single most important finding in this scan.**
- Priscylla Silva, Claudio T. Silva, Luis Gustavo Nonato, **"Exploring the Relationship Between Feature Attribution Methods and Model Performance"**, AAAI 2024 AI4Ed workshop (arXiv 2405.13957) — FACT, confirmed via arXiv fetch. Compares 9 explanation methods; finds a strong Spearman correlation between explainer agreement and model performance, on student-success-type data.
- **Visagreement** — Priscylla Silva, Guardieiro, Barr, Claudio Silva, Nonato, *TVCG* 2025 — FACT (confirmed via search + priscyllasilva.com.br project page). VA tool for the feature-attribution (dis)agreement problem: settings panel + "(Dis)Agreement Space View."
- **Explainalytics** — Priscylla Silva et al., **"Visual Analytics for Guiding Feature Attribution Method Selection"**, IJCAI 2025 (FACT, confirmed via ijcai.org proceedings PDF) — a closely related/sibling tool to Visagreement; a journal version also appears on ScienceDirect ("A visualization-driven decision support system for selecting feature attribution methods"). **Unverified** whether Explainalytics and Visagreement are the same system renamed or two distinct tools from the same PhD pipeline — worth resolving before proposing anything adjacent.
- Net effect: this exact idea (explainer disagreement + model performance + a VA tool to explore it) has now been published by this lab **four times** (workshop → TVCG → IJCAI → journal) across 2024–2025. This is the headline red flag for the whole slice.

**Other key 2022–2026 papers:**
1. Swamy, Radmehr, Krco, Marras, Käser, **"Evaluating the Explainers"**, EDM 2022 (arXiv 2207.00551) — FACT, code open at `github.com/epfl-ml4ed/evaluating-explainers` (verified HTTP 200). Founding paper of the "explainer disagreement in education" niche — precedes and likely influenced the VIDA-NYU line above. Finds LIME/SHAP-family explainers disagree substantially on the same BiLSTM student-success models.
2. Swamy et al., "Trusting the Explainers: Teacher Validation of XAI for Course Design", arXiv 2212.08955 (~2022/23) — human-subject angle, thin on visualization.
3. **MultiModN**, Swamy et al., NeurIPS 2023 (arXiv 2309.14118) — intrinsically-interpretable modular multimodal network; applied to education among other domains. Code open at `github.com/epfl-iglobalhealth/MultiModN` (FACT, verified via search; not independently curl-checked).
4. **InterpretCC** / "Intrinsic User-Centric Interpretability through Global Mixture of Experts", Swamy et al., ICLR 2025 (arXiv 2402.02933) — ante-hoc alternative to the whole post-hoc-disagreement problem.
5. **Interpret3C**, Swamy et al., EDM/AIED 2024 (Springer) — interpretable student clustering.
6. "A Human-Centric Approach to Explainable AI for Personalized Education", arXiv 2505.22541 (2025, preprint) — appears to be a synthesis/position piece from the same group; useful as a recent state-of-the-field summary.
7. **iLLuMinaTE**, arXiv 2409.08027 (2024) — LLM-XAI framework for student feedback (crosses into (b)).
8. OULAD regional fairness audit, *MDPI Sustainability* 2026 (peer-reviewed, open access) — region-stratified recall gap of 0.25 (0.55 Wales vs. 0.80 West Midlands) — standard fairness audit, no XAI-disagreement angle.
9. "A Unified Survival Benchmark for Temporal Dropout Risk Prediction in Learning Analytics", arXiv 2604.08870 (2026, preprint) — reframes dropout as survival analysis; very recent, code/data openness not verified.
10. VertiDaX, *PeerJ Computer Science* 2026 — explainable hybrid DL dropout model with vertical data integration.

**Saturation verdict: SATURATED on "explainer disagreement in student-success prediction" specifically** (two labs, ~6 papers, 2022–2025, escalating from workshop to TVCG/IJCAI). **Less saturated: fairness-subgroup-conditioned explanation disagreement** — the OULAD fairness audits found don't use XAI-disagreement methods, and the disagreement/VA papers found don't slice by protected subgroup. This intersection appears open but is **not exhaustively falsified** given the search budget.

**Open data (verified):**
- **OULAD** — Open University Learning Analytics Dataset, CC BY 4.0, ~32,593 students, 7 courses. URL `analyse.kmi.open.ac.uk/open_dataset` verified reachable (curl 200, following redirect).
- **xAPI-Edu (Kalboard 360)** — Kaggle, ~480 students, Jordan K-12 platform logs; verified reachable (curl 200). Much smaller/thinner than OULAD.

**Candidate research questions:**
1. **Subgroup-conditioned explainer disagreement on OULAD.** Reproduction target: the Silva/Nonato AAAI-2024-workshop methodology (or Swamy et al. EDM 2022) reproduced on OULAD dropout prediction. Minimal extension: slice the agreement analysis by protected subgroup (region, disability, gender), following the fairness-audit paper's grouping, and expose it through a small linked-view Streamlit/Plotly tool (disagreement heatmap × subgroup). **Falsification check:** searched "OULAD dropout SHAP disagreement fairness 2024 2025" — found fairness audits (no disagreement metric) and disagreement/VA papers (no subgroup slicing), but not the combination. Given the brief's own note that "overlap is not fatal, can make a great reproduction target," this is arguably the *safest* choice precisely because the reproduction half is well-trodden by the professor's own lab — but the extension must be sharp enough not to look like a rename of Visagreement.
2. Compare an intrinsically-interpretable model (InterpretCC) against post-hoc explainer disagreement (Visagreement-style) on the same OULAD task — does ante-hoc interpretability actually reduce the disagreement problem, or just relocate it?

**Feasibility in ~40h: HIGH** — tabular data, no CV/NLP needed (best skill match of the whole slice), OULAD is well-documented, SHAP/LIME are lightweight on a laptop CPU. Streamlit/Plotly front end is squarely in the stated comfort zone.

**Publication path:** LAK/EDM short paper more realistic than VIS, given how close this sits to the professor's own recent TVCG/IJCAI work — a VIS submission would need very clear differentiation language.

**Red flags:** **This is the biggest red flag in the entire slice.** Direct, recent (2024–2025), repeated instructor-lab work on almost exactly this question. Any proposal here needs the professor's explicit sign-off on how it differs from Visagreement/Explainalytics, or it risks reading as redundant with (or a minor variant of) work already done in-house.

---

## (d) Learning analytics dashboards (brief, per instructions)

Systematic review (ACM DL 2024, "Have Learning Analytics Dashboards Lived Up to the Hype?") found **no evidence** dashboards reliably improve achievement — mostly negligible/small effects. Recent instances (Thermos dashboard, *JLA* 2024; de Vreugd et al., *JCAL* 2025 on reference frames/goal-setting; 2024–25 instructor-sensemaking and multimodal team-teaching dashboard studies) confirm this remains an active but methodologically soft HCI/LA area, evaluated mainly via user studies rather than technical contribution. No instructor-lab overlap found. **Not recommended for deep dive** — treat only as motivating background (why better-validated explanation/interpretability work, not just more dashboards, is needed).

---

## Triage table

| Sub-area | Best RQ | Novelty evidence (1 line) | Feasibility | Grade-safety | Pub. upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) KT interpretability | Faithfulness + cross-explainer disagreement audit of DKT/SAKT/AKT on pyKT | 2024 survey explicitly states explanation-evaluation methods are lacking (FACT) | Med-High | **High** | Med | **Yes** | No instructor-lab overlap found (though dblp check incomplete); clean repro target (pyKT); good skill fit |
| (b) LLM grading reliability | VA tool for LLM-grader disagreement using open-weight local models | Field is saturated on stats/IRT reliability studies; VA + open-weight angle not found in search | Med | Med | Med-High | Maybe | Very hot topic = scoop risk; needs sharp differentiation to avoid "yet another reliability paper" |
| (c) Student-success XAI + fairness | Subgroup-conditioned explainer disagreement on OULAD | Fairness audits and disagreement/VA papers exist separately; combination not found (not exhaustively falsified) | **High** | Low-Med | Med | Maybe, only with explicit differentiation | **Biggest red flag of the slice**: professor's own lab published this core idea 4 times, 2024-2025 (workshop→TVCG→IJCAI→journal) |
| (d) LA dashboards | — | Systematic review shows weak effects; well-trodden HCI territory | — | — | — | No | Not enough technical/VA novelty; brief only per instructions |

---

## Summary (~200 words)

Two sub-areas deserve a deep dive. **(a) Knowledge-tracing explanation reliability** is the safest bet: no instructor-lab overlap surfaced (though a dblp check was blocked and should be redone), a 2024 peer-reviewed survey explicitly names explanation-evaluation as an open gap, pyKT gives a clean reproduction target, and the compute/skill fit (tabular/sequence data, PyTorch, Streamlit/Plotly) is excellent. **(c) Student-success XAI/fairness** has the best data and lowest engineering burden of the whole slice (OULAD, tabular, CC BY 4.0), and a plausible untested intersection (fairness-subgroup-conditioned explainer disagreement) — but only as a second choice, because of the scan's single biggest red flag.

**Biggest red flag overall:** the instructor's own lab (Priscylla Silva, Claudio Silva, Nonato) has published essentially the same idea — feature-attribution disagreement plus a VA tool to explore it, on student-success-shaped data — four times in two years (AAAI-2024 workshop → TVCG 2025 Visagreement → IJCAI 2025 Explainalytics → a ScienceDirect journal version). Any project in sub-area (c) needs an explicit, professor-verified account of how it differs, or it risks reading as redundant with work already done in-house.
