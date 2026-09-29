# Deep dive D3: Visual analytics for disagreement among LLM judges (local open-weight judges; optional education instantiation)

Written 2026-09-28. This is a falsification-first review following `DEEP_BRIEF.md`.

**Labels.** Claims are tagged FACT (I checked it this session), AUTHOR CLAIM (the paper's own statement, not independently checked), SYNTHESIS (my inference across sources), or SPECULATION.

**Evidence levels.** FULL TEXT means I read the PDF, extracted with `pdftotext` into `lit_notes_open/pdfs/`. ABSTRACT means I read the arXiv or OpenAlex abstract. SECONDHAND means a search snippet or a citation in another paper.

**Budget used.** About 14 WebSearch calls. Everything else went through the arXiv API, OpenAlex (its free daily budget ran out partway through), the Semantic Scholar API (one success, then HTTP 429), `gh api`, `curl` and WebFetch.

---

## 0. Bottom line (read this first)

- **Gap verdict: PARTLY ADDRESSED, with a specific integration still open.**
  - Every *ingredient* is published. These include:
    - single-judge side-by-side VA (LLM Comparator);
    - criteria-alignment tools (EvalGen, EvalAssist, MultEval, Evalet);
    - two-source agreement dashboards (Langfuse Score Analytics);
    - and a flood of 2026 *statistical* papers on judge disagreement: Coin Flip Judge, Geometry of LLM-as-Judge (EMNLP 2026), Nine Judges / Two Effective Votes, Whose Gold?, and the essay-scoring rater-effects audit.
  - I found **no VA tool that aligns several judges, their prompt variants, repeated runs and human labels on the same items and splits item-level disagreement into its causes.** Those causes are:
    - human ambiguity;
    - judge instability;
    - feature-linked systematic bias;
    - rationale divergence.
  - SYNTHESIS, at moderate confidence. It rests on negative search results listed in §6. It is not confirmed by any paper saying "nobody has done this".
- **The ML findings are NOT novel and must not be sold as such.** All of the following are already established (§3):
  - position, verbosity and self-preference bias;
  - roughly 14% run-to-run flip rate;
  - cross-judge κ ≈ 0.5;
  - "judges agree with each other more than with humans";
  - correlated panel errors;
  - protocol choices swinging agreement statistics.
- **The defensible contribution** is a linked-view "disagreement decomposition" tool, plus an **objective planted-pattern evaluation**: can the tool recover injected biases that pooled κ hides? It also includes a reproduction of MT-Bench agreement numbers with local judges.
- **Best reproduction targets.** Both are cheap and verified:
  1. Zheng et al. 2023, Table 5 agreement numbers. The data is CC-BY-4.0 on Hugging Face (HF), and FastChat's `compute_agreement.py` is pure numpy.
  2. The LLM Comparator workflow, via its JSON schema and hosted demo. The repo is Apache-2.0 but **archived**.
- **Instructor-lab overlap.** The lab has no LLM-judge paper. But **Visagreement (P. Silva, Guardieiro, Barr, C. Silva, Nonato; TVCG 2025)** is a VA tool for *(dis)agreement among explanation methods*. It is the direct structural analogue: "Visagreement for LLM judges". That helps the grade and is a framing risk for the paper (§5).
- **Single biggest risk: scoop plus "not novel enough".** The area produces several judge-reliability preprints a month. Google PAIR (the LLM Comparator team, and GROVE at VIS 2026) and KAIST (Evalet, EvalLM) are active VA groups that could ship a multi-judge view first.

---

## 1. Research question (refined after falsification)

The original candidate RQ asked whether the tool beats aggregate statistics at finding "systematic, actionable disagreement". That survives, but it has to be narrowed. "Where do judges disagree" is already answered statistically for many settings. What is *not* shown is whether a VA tool can **attribute** item-level disagreement to its sources, and whether that attribution finds structure that pooled κ/accuracy hides.

**Refined RQ.**

> When several local open-weight LLM judges (and prompt variants of each) judge MT-Bench items that have *multiple* human votes, can a linked-view VA tool break item-level judge–human disagreement into four components:
> - (a) human ambiguity (human–human split);
> - (b) judge instability (position-swap or repeat flips);
> - (c) feature-linked systematic bias (length gap, turn, category, answer-model family);
> - (d) rationale divergence (what judges cite when they depart from humans)?
>
> And does this tool recover planted bias patterns that leave the aggregate agreement matrix nearly unchanged?

**Hypotheses.**

- **H1 (reproduction).** Recomputing Zheng et al. Table 5 from the released files gives:
  - GPT-4 pair vs. human: 66% (S1) and 85% (S2);
  - human vs. human: 63% (S1) and 81% (S2);
  - all within ±1 pp.

  Local judges (gpt-oss-20b, qwen3.8:27b) will score below GPT-4 on S2 agreement with humans. That second part is SPECULATION; the size is unknown.
- **H2 (aleatoric share).** Judge–human disagreement is much higher on the 386 human-split cells than on the 575 human-unanimous cells (FACT: counts computed this session). So a large share of "judge error" in pooled numbers is human ambiguity.
- **H3 (hidden systematic structure).** On human-unanimous cells, the following predict disagreement, with interaction effects by judge:
  - the length gap between the two answers;
  - swap inconsistency;
  - turn 2;
  - category (math, reasoning, coding).

  Pooled κ does not show these interactions.
- **H4 (prompt sensitivity).** Rewording the judge prompt or rubric flips a non-trivial share of majority verdicts. Coin Flip reports 25% for GPT-4o-mini and GPT-4.1-mini (AUTHOR CLAIM). The flips cluster in an identifiable item subset.
- **H5 (tool validity).** In a planted-pattern test, the tool ranks each planted slice among its top-3 flagged slices, using a permutation null with Benjamini–Hochberg control. At least one planted pattern changes pooled κ by less than 0.05. A null-injection control produces no flagged slices.

---

## 2. Paper table (closest and most load-bearing work)

| Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence level |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Kahng et al., **LLM Comparator: Interactive Analysis of Side-by-Side Evaluation of LLMs** | 2024/25 | TVCG 31(1) 503–513 (VIS 2024); earlier CHI EA 2024 (2402.10524) | peer-reviewed | Help developers see when, why and how model A beats model B under an automatic side-by-side (SxS) judge | Example data: Chatbot Arena subset (371 examples, oasst-pythia-12b vs. alpaca-13b); judgments from Google Cloud APIs | **One** judge LLM, repeated 6× with half position-flipped and averaged; LLM-based rationale summarisation and soft clustering | Interactive table; score histograms; per-category win rates; rationale clusters; n-grams and custom functions | Observational study (6 users, CHI EA); usage survey; rationale-clustering experiment against k-means and fuzzy c-means (Jaccard 0.630, NMI 0.381) | AUTHOR CLAIM: broad internal adoption at Google | FACT, quoted from §10.2: "Imperfect clustering"; "Pre-configured undesirable patterns"; "multivariate data visualization… correlation between rating scores and fine-grained criteria"; "Combining with explainable AI methods"; "**Comparing more than two models**". **No mention of multiple judges or judge-vs-human.** JSON schema's `individual_rater_scores` has no judge-identity field | Yes. Apache-2.0, **archived** (last push 2025-02-11). Python lib depends on `vertexai` (a paid API). Hosted demo returns HTTP 200 | doi.org/10.1109/tvcg.2024.3456354 ; github.com/PAIR-code/llm-comparator | FULL TEXT (TVCG PDF and CHI EA PDF) |
| Shankar et al., **Who Validates the Validators? (EvalGen)** | 2024 | UIST 2024 (2404.12272) | peer-reviewed | Align LLM-generated evaluators (assertions) with human grades | Two pipelines; user study | Mixed-initiative criteria and assertion generation inside ChainForge; users grade a sample | ChainForge nodes, report card | Offline alignment + qualitative study (9 practitioners) | "Criteria drift": users' criteria change as they grade | FACT, §8.3: binary assertions only; wants finer-grained tracking; multi-user grading "have to consider inter-rater reliability and handle disagreements"; only 2 pipelines, small expert sample | Yes, inside ChainForge (MIT, active) | arxiv.org/abs/2404.12272 | FULL TEXT |
| Yagubyan, **The Coin Flip Judge? Reliability and Bias in LLM-as-a-Judge** | 2026 | arXiv 2606.13685 (PDF stamp reads 23 Apr 2026) | preprint, single independent author | Run-to-run reliability of judges | 29 hand-built competitive pairs, 10 categories | 2 OpenAI judges (GPT-4o-mini, GPT-4.1-mini); 50 pairwise + 50 pointwise trials per item; temperature and prompt ablations | Static plots | Flip rate, ICC(2,1), reliability curves | AUTHOR CLAIM: 13.6% mean flip rate; 72% A-position bias for GPT-4o-mini; cross-judge 76% (κ = 0.51); prompt rewording flips 25%; t = 0 does not remove inconsistency; 11 trials to reach 95% fidelity | FACT, §6.5: only one provider; "open-source Llama/Mistral… **primary direction for future work**"; 29 items; self-preference confound; "**we do not directly compare to human annotators on the same task instances**" | Not found | arxiv.org/abs/2606.13685 | FULL TEXT |
| Mukherjee et al., **The Geometry of LLM-as-Judge: Why Inter-LLM Consensus Is Not Human Alignment** | 2026 | EMNLP 2026 (2606.03043) | accepted (per arXiv comment) | Is judge–judge agreement evidence of human alignment? | Two Indic benchmarks, 8 languages | Judge score vectors; spread, effective rank, angle to human; LL/LH/HH agreement triple; 42 judges | Static geometric plots (σ-ratio vs. angle; LL−LH gap segments) | Reference-matched agreement | AUTHOR CLAIM: on subjective rubrics judges agree with each other as much as humans do, yet reach 58–66% of human agreement and "concentrate on an axis humans do not weight"; ensembles converge on the judges' shared axis | FACT: predictive, not causal; one verifiable rubric; uneven human references; closed APIs may drift | Raw outputs released (App. O) | arxiv.org/abs/2606.03043 | FULL TEXT (skimmed; limitations read) |
| Kohli, **Nine Judges, Two Effective Votes: Correlated Errors Undermine LLM Evaluation Panels** | 2026 | arXiv 2605.29800 | preprint | Real information value of judge panels | ChaosNLI (100 annotators per item), RewardBench | Kish n_eff, Condorcet null | Static | Accuracy against a Condorcet ideal | AUTHOR CLAIM: 9 judges ≈ 2 independent votes; the best single judge ≥ the panel; aggregation closes ≤ 11% of the gap | FACT: classification tasks only; "open-ended generation evaluation… may differ"; they avoid RewardBench 2 because some of its gold labels are LLM-derived | Not checked | arxiv.org/abs/2605.29800 | FULL TEXT (skimmed) |
| Sunkavalli, **LLM Judges as Raters: pre-registered audit of severity, halo, reliability, version instability in essay scoring** | 2026 | arXiv 2608.29517, under review at J. Learning Analytics | preprint | Rater effects of LLM essay judges | ENEM/Essay-BR (MIT) and **ASAP sets 1, 2, 7, 8** (350 essays each; 2 trained raters) | 12 judges on Amazon Bedrock (Claude, Nova, Llama, Qwen3-32B); many-facet Rasch, G-theory, K = 3 replications; 110,571 calls | Static | Pre-registered tests | AUTHOR CLAIM: severity spans 219 of 1000 points; judge–human r only .47–.56; version upgrades shift severity; replication gives stability, not accuracy | FACT: one platform; 100-token cap "forecloses rationale-first scoring"; no demographics, so no fairness (DIF) analysis; "pinned open-weights checkpoints would remove the mutable-alias risk"; ASAP text cannot be redistributed | Tensor and harness said to be released (link not verified) | arxiv.org/abs/2608.29517 | FULL TEXT (skimmed) |
| Verga et al., **Replacing Judges with Juries (PoLL)** | 2024 | arXiv 2404.18796 (Cohere) | preprint | Does a panel of small judges beat one large judge? | KILT QA, Chatbot Arena-style | 3 small-model panel; voting | Static | κ against humans; cost | AUTHOR CLAIM: the panel has higher κ, less intra-model bias, and is 7× cheaper | FACT: 3 settings; limited panels; panel selection left open | Not checked | arxiv.org/abs/2404.18796 | FULL TEXT (skimmed) |
| Zheng et al., **Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena** | 2023 | NeurIPS 2023 D&B | peer-reviewed | Can strong LLMs approximate human preference? | **MT-Bench human judgments: CC-BY-4.0, 3,355 votes (human split), 2,400 GPT-4 pair judgments** (FACT, HF API) | Pairwise/single grading; position swap | Static | S1/S2 agreement | FACT (Table 5): GPT-4 pair vs. human 66% (S1) / 85% (S2); human–human 63% / 81% | Position, verbosity and self-enhancement bias; limited math grading | FastChat (Apache-2.0, active; `llm_judge` extra pins `openai<1`) | arxiv.org/abs/2306.05685 | FULL TEXT |
| Silva P., Guardieiro, Barr, Silva C., Nonato, **Visagreement: Visualizing and Exploring Explanations (Dis)Agreement** | 2025 | TVCG 31(10) 7862–7875 | peer-reviewed (**instructor lab**) | How often, when and where do explanation methods disagree? | Tabular binary classification | Agreement metrics across local feature-attribution methods | Linked VA tool | 4 experts | AUTHOR CLAIM: disagreement relates to explanation quality and model accuracy | Tabular and binary only (AUTHOR CLAIM, abstract) | `priscylla/visagreement` (Python, **no license**, last push 2023-10) | doi.org/10.1109/tvcg.2025.3558074 | ABSTRACT (TechRxiv PDF returned 403) |
| Kim T.S. et al., **Evalet: functional fragmentation** | 2025/26 | CHI 2026 (2509.11206) | peer-reviewed | Holistic judge scores hide which parts of the output drove them | User-supplied | Split outputs into fragments tied to criteria | Fragment-level visualisation across outputs | N = 10 user study | AUTHOR CLAIM: 48% more evaluation misalignments found | One judge; no multi-judge or human-label alignment (SYNTHESIS from abstract) | unverified | arxiv.org/abs/2509.11206 | ABSTRACT |
| Chiang et al., **MultEval** | 2026 | CHI-adjacent symposium (ACM 10.1145/3808045.3808093); 2604.26679 | peer-reviewed | *Multiple human stakeholders* disagree on judge criteria | Case study | Consensus-building interface | Disagreement surfacing for humans | Formative study + case study | Criteria are socially negotiated | Human–human disagreement, not judge–judge | unverified | arxiv.org/abs/2604.26679 | ABSTRACT |
| Ashktorab et al., **EvalAssist** (IBM) | 2024–25 | AAAI 2025 demo; CHI 2025 HEAL workshop; 2507.02186 | peer-reviewed demo/workshop | Criteria development for LLM-as-a-judge | User tasks | Direct and pairwise judges; positional-bias check; uncertainty | Web UI | n = 15 practitioners | Users change criteria and evaluator models | Criteria-centric; no multi-judge × human disagreement analytics (SYNTHESIS) | Yes, Apache-2.0, active | github.com/IBM/eval-assist | ABSTRACT |
| Jung, …, Kahng, **Who Defines "Best"? Interactive user-defined evaluation of LLM leaderboards** | 2026 | FAccT 2026 (2604.21769) | peer-reviewed | Leaderboard rankings depend on prompt slices | LMArena data | Slice re-weighting | Interactive interface (design probe) | Qualitative study | Rankings vary by slice | Human preferences only; no judge comparison | unverified | arxiv.org/abs/2604.21769 | ABSTRACT |
| Rao & Callison-Burch, **Agreement Metrics for LLM-as-Judge Evaluation: What to Report and Why** | 2026 | arXiv 2606.00093 | preprint | Protocol choices behind judge–human agreement numbers | 3 published evaluations | Measurement-protocol analysis | Static | — | AUTHOR CLAIM: protocol choice alone moves accuracy from 0.551 to 0.899 and pushes κ across zero **without changing a verdict** | — | Ancillary files | arxiv.org/abs/2606.00093 | ABSTRACT |
| Jha, **Whose Gold? Annotator-pool disagreement is large at the item level** | 2026 | arXiv 2608.15980; submitted to NeurIPS 2026 HAIC workshop | preprint | Does annotator-pool identity matter? | MultiPref; **MT-Bench** | Pool comparison, bootstrap | Static | — | AUTHOR CLAIM: on unanimous MT-Bench cells, authors vs. experts differ on 30.5% and reverse on 8.5%, yet the leaderboard is unchanged; an LLM judge tracks crowd over experts | Code "upon acceptance" | pending | arxiv.org/abs/2608.15980 | ABSTRACT |
| Coscia et al., **iScore** | 2024 | IUI 2024 (2403.04760) | peer-reviewed | Interpret LLM summary-scoring models in education | Summaries | Revision tracking, model-weight views | Linked views | 3 learning engineers, 1 month | Accuracy +3 pp in a case study | One scoring model; no judge disagreement | demo video | arxiv.org/abs/2403.04760 | ABSTRACT |
| Langfuse **Score Analytics** (docs) | 2025–26 | product docs | industry tool | Compare two score sources | — | Pearson/Spearman/MAE/RMSE; Cohen's κ/F1 | Heatmap, confusion matrix, trend | — | — | FACT (docs): "supports comparing up to two scores at a time"; no item drill-down, no metadata slicing, no text/rationale analysis described | OSS | langfuse.com/docs/evaluation/evaluation-methods/score-analytics | FULL TEXT (docs page) |

**Other verified context** (ABSTRACT or metadata only):
- JudgeLM (ICLR 2025)
- Prometheus (ICLR 2024) and Prometheus 2 (EMNLP 2024)
- JudgeBench (ICLR 2025; HF, MIT)
- RewardBench 2 (ICLR 2026; HF, ODC-BY)
- Bavaresco et al., "LLMs instead of Human Judges?" (ACL 2025)
- Thakur et al., "Judging the Judges" (GEM 2025)
- Wang et al., "LLMs are not Fair Evaluators" (2305.17926)
- Ye et al., "Justice or Prejudice?" (2410.02736)
- Panickssery et al., self-preference (2404.13076)
- SPADE (2401.03038)
- EvalLM (CHI 2024; MIT)
- ChainForge (CHI 2024; MIT, active; hosts EvalGen)
- Zeno (MIT, **archived** 2023)
- EvalTree (COLM 2025: weakness profiling, not judges)
- Judge Arena (AtlaAI HF Space, running)
- GROVE, "Beyond One Output" (Reif et al., VIS 2026: distributions of generations, not judges)
- DeLucia et al., "Same Verdict, Different Reasons" (2604.16383: judge vs. clinician rationales differ even when verdicts agree)
- Tapwal et al., rationalization bias (2605.23970)
- Lail, "Decomposing LLM-Judge Uncertainty" (2609.06444, MIT code)
- He et al., "Experts Rise Where LLMs Disagree" (2609.26926)
- Ehara, EDM 2026 poster, predicting judge–human disagreement using gpt-oss-120b and Qwen3-235B
- Seßler et al., LAK 2025, LLM vs. 37 teachers on 20 essays

---

## 3. What is already known (so the project does not "re-find" it)

These are AUTHOR CLAIMs from the sources in §2. Together they are the "known" baseline the tool should *reproduce visually* rather than claim as discoveries.

1. **Position bias.** Most judges favour the first position (Zheng 2023). GPT-4o-mini picked position A 72% of the time (Coin Flip). The standard fix is to swap positions and count inconsistent verdicts as ties (MT-Bench "S1").
2. **Verbosity and length bias; self-enhancement or self-preference** (Zheng 2023; Panickssery 2024; Ye 2024).
3. **Stochastic instability.** A 13.6% mean pairwise flip rate at t = 1. Setting t = 0 reduces flips but does not remove them. Majority votes need about 11 trials for 95% fidelity (Coin Flip, 29 items, OpenAI-only).
4. **Cross-judge disagreement is moderate, and consensus is not alignment.**
   - Cross-judge κ ≈ 0.51 (Coin Flip).
   - Judges reach only 58–66% of human agreement on subjective rubrics while agreeing with each other at human-like levels (Geometry, EMNLP 2026).
   - Panels behave like about 2 effective votes because of correlated errors (Nine Judges).
   - PoLL claims panels beat a single judge (Verga 2024), so the literature is **contested** on panels. That makes it a good thing for a tool to *show* rather than *assert*.
5. **Aggregate agreement numbers are fragile.**
   - Protocol choices alone move accuracy from 0.55 to 0.90 (Rao & Callison-Burch 2026).
   - Leaderboards stay invariant while item-level labels differ by 30% across annotator pools on MT-Bench (Whose Gold?).
   - SYNTHESIS: this is the strongest *scientific* argument that item-level visualisation is necessary, not just convenient.
6. **Education.**
   - LLM essay judges differ hugely in severity even when their correlations with humans are similar (r = .47–.56).
   - Replication buys stability, not validity (Sunkavalli 2026).
   - Human-rater baselines exist: PERSUADE 2.0 IRR before adjudication is weighted κ = .745 (FACT, preprint p.3). ASAP has two raters per essay (FACT via Sunkavalli).
7. **Rationales.**
   - Judges and clinicians "rarely cite the same explanation" even when verdicts agree (DeLucia 2026).
   - Judge rationales anchor on non-evidential cues (Tapwal 2026).
   - LLM Comparator's LLM-based rationale clustering is admittedly error-prone (FACT, §10.2).

**SYNTHESIS: what remains open for VA.**

- None of the §3 papers provides an *interactive* way to see, per item, which of the four components (human ambiguity, instability, feature-linked bias, rationale divergence) explains a judge–human disagreement.
- None tests whether a visual tool recovers such structure better than pooled statistics.
- The statistical papers each isolate *one* layer: Coin Flip handles instability; Geometry, the shared axis; Nine Judges, correlation; Sunkavalli, severity.

---

## 4. Tools check: is multi-judge / judge-vs-human disagreement VA already done?

| Tool | Multiple *judges*? | Judge vs. human? | Human–human ambiguity? | Rationale analysis | Item-level linked slicing | Verdict |
|---|---|---|---|---|---|---|
| LLM Comparator (TVCG 2025) | **No.** Repeated runs of one judge; the schema has no rater-identity field (FACT) | Not built in. Human labels could be hacked in as a custom field (SYNTHESIS) | No | **Yes** (LLM clustering) | Yes (tags, custom fields) | Model A vs. B, not judge vs. judge |
| EvalGen / ChainForge | Several evaluators, one per criterion | Yes (user grades) | No (future work: inter-rater reliability) | No | Limited | Criteria alignment |
| EvalAssist | Choosable evaluator model | Partly | No | Judge explanations shown | Limited | Criteria development |
| Evalet (CHI 2026) | No | Misalignment finding | No | Fragment-level | Yes | Single judge |
| MultEval | No (multiple *humans*) | — | Yes (stakeholders) | Yes | — | Human criteria negotiation |
| Langfuse Score Analytics | **Two** sources max (FACT) | Yes (κ, confusion matrix) | No | No (FACT: text not supported) | No (FACT) | **The "aggregate statistics" baseline** |
| Who Defines "Best"? | No | Human preferences only | No | LLM-generated rationales of user preferences | Slices | Leaderboard VA |
| Judge Arena (Atla) | Many judges ranked by crowd votes | Humans pick the better judge | No | Shows judge critiques | No | Leaderboard of judges |
| iScore (IUI 2024) | No | Yes | No | No | Yes | Education, one model |

**Verdict: no.** Among the 55 works citing the two LLM Comparator versions (OpenAlex `cites:` for W4402401807, W4396833346, W4391940865), none is a multi-judge × human disagreement VA tool (FACT, list scanned). VIS 2025 and VIS 2026 accepted-paper titles contain no LLM-judge VA paper (FACT: scraped from ieeevis.org). So, partly addressed at the ingredient level and open at the integration level.

---

## 5. Instructor-lab overlap (checked exhaustively within budget)

**Method.** I scanned OpenAlex works since 2023-06 for these author IDs:
- C. Silva A5003584200
- Nonato A5011424640
- F. Miranda A5083220583
- Xenopoulos A5036139759
- Rulff A5031297466 / A5082777407
- Guardieiro A5033315494
- Solunke A5082090092
- P. Silva A5006036247 / A5057148519
- Bertini A5102923390

That gave 207 works in total, which I grepped for LLM, judge, grading, essay, rubric, evaluation, education and agreement. I could not identify B. Barr's author ID; he appears as a co-author on Visagreement. DBLP was not checked.

**Findings.**

- **Visagreement (TVCG 2025; P. Silva, Guardieiro, B. Barr, C. Silva, Nonato).** FACT: it exists, with the venue and abstract confirmed. This is a VA tool for "how often, when, and where explanation methods agree or disagree", with agreement metrics and linked views.
  - SYNTHESIS: this project is *Visagreement's question moved to LLM judges*: judges ↔ explainers, items ↔ instances, verdict/rationale ↔ attribution vectors.
  - For the course this is a **plus**. You can explicitly adopt Visagreement's framing ("how often / when / where / why") and cite it as design inspiration.
  - For a paper it is a **novelty risk**: a reviewer could call it a domain port. The paper therefore needs a component Visagreement lacks: the human-ambiguity axis, instability separation, rationale divergence, and planted-pattern validation.
- **P. Silva & Costa, "Assessing LLMs for Automated Feedback Generation in Learning Programming Problem Solving"** (arXiv 2503.14630, 2025). This tests 4 closed LLMs on 45 student solutions and finds 63% of hints accurate. FACT from the abstract. It is adjacent to the education instantiation (LLM reliability in education), but it has no judge disagreement and no VA. I did not find the AAAI AI4Ed 2024 item the task mentions under her OpenAlex profile. Her other listed items are IJCAI doctoral-consortium papers on feature-attribution selection. **Unverified; ask or check her CV.**
- **POEM** (He, Wang, Liu, Wu, **C. Silva**, Qu; PacificVis 2025). This is interactive prompt optimisation for multimodal reasoning. It is adjacent (prompt-variant VA) and not about judges.
- **BDIViz** (VIS 2025, with LLM-powered validation). Not related.
- **No Silva/Nonato/Miranda paper on LLM-as-a-judge, LLM grading VA, or judge disagreement was found.** FACT within the OpenAlex scan. DBLP was not checked.

---

## 6. Falsification log: queries that returned nothing relevant

- **arXiv API**, `abs:"multiple judges" AND (abs:visual OR abs:interface OR abs:interactive)`: 5 hits, none relevant.
- **arXiv API**, `(abs:"LLM judges" OR abs:"LLM-as-a-judge") AND abs:"visual analytics"`: only Lexara (evaluates conversational VA agents; not relevant).
- **arXiv API**, `(abs:grading OR abs:scoring) AND abs:"visual analytics" AND (LLM)`: only iScore (2024) is relevant; the rest are unrelated.
- **arXiv API**, `cat:cs.HC AND (LLM judge terms)`, 60 newest: tools about *criteria* (MultEval, iRULER, LLARS, Evalet), no multi-judge disagreement VA.
- **VIS 2025 accepted list** (ieeevis.org): no LLM-judge or judge-disagreement title. The closest are ConceptViz and "Understanding LLM Behaviors through Interactive Counterfactual…".
- **VIS 2026 accepted list**: no judge VA. The closest are GROVE ("Beyond One Output"), TreeTracer ("Exposing the Unsaid"), and "Verify-First: Visual Triage of… Instability in LLM-Generated Analytical Explanations".
- **OpenAlex citing works of LLM Comparator** (about 55): RAGExplorer, Evalet, DSCode Comparator, Who Defines Best, CALLM, JailbreakHunter and others; none multi-judge.
- **Kahng arXiv author list** through 2026-07: no multi-judge VA.
- **WebSearch**:
  - "visual analytics multiple LLM judges disagreement human labels interactive tool 2025" and "meta-evaluation LLM judges visual analytics system human-LLM disagreement explorer": ML papers only.
  - "comparing LLM judges … interactive visualization": only the Kahng FAccT paper and LLM Comparator.
  - "visual analytics LLM essay scoring disagreement human raters dashboard EDM LAK AIED": CLARA (a collaboration dashboard, not a grader-disagreement VA) and the Sunkavalli audit (statistical).
- **Semantic Scholar** "visual analytics LLM judge disagreement": 12 hits; only "Dissecting Atomic Facts" (a 2-page VA poster *concept* on fact-decomposition annotation disagreement, Keim lab) is VA. Then HTTP 429.

**Caveats.** The CHI 2026 full program, UIST 2025 and IUI 2026 were not scraped. This is a negative search, so treat the gap as moderate-confidence.

---

## 7. Reproduction target (verified)

**Primary: R1, MT-Bench judge–human agreement (Zheng et al. 2023, Table 5).**

- Data (FACT):
  - `lmsys/mt_bench_human_judgments` on HF: not gated, **CC-BY-4.0**, last modified 2023-07-20.
  - Splits: `human` (3,355 rows) and `gpt4_pair` (2,400 rows).
  - The parquet files are 739 KB and 650 KB. I downloaded them to scratch only.
  - Fields: `question_id, model_a, model_b, winner, judge, conversation_a/b, turn`.
- Structure I computed this session (FACT):
  - 1,814 unique (question, model pair, turn) cells have human votes;
  - **961 cells have ≥2 human votes**: 575 unanimous and 386 split;
  - all 80 questions are covered, with turns balanced (485 / 476);
  - there are 65 human judge IDs (2,668 "expert" votes and 687 "author" votes);
  - the median conversation length per side is 313 words (p90: 685).
- Code (FACT):
  - `lm-sys/FastChat` is Apache-2.0, not archived, last push 2026-05-01.
  - `fastchat/llm_judge/compute_agreement.py` depends only on numpy/json.
  - `data/judge_prompts.jsonl` holds the official pairwise judge prompts.
  - **Stale pin:** the `llm_judge` extra requires `openai<1`. Do not install it. Call Ollama's OpenAI-compatible endpoint with your own 50-line client, and reuse only the prompts and the agreement script.
- **R1a (exact).** Recompute 66% / 85% (GPT-4 pair vs. human, S1/S2) and 63% / 81% (human vs. human) from the released files.
  - Pass: within ±1 pp. This takes about an hour.
- **R1b (substitution).** Rerun the same pairwise protocol with local judges, with position swap and inconsistent verdicts counted as ties, then fill the same table with bootstrap confidence intervals.
  - Cluster the bootstrap by question: there are 80 questions and items are not independent.
- **R2 (VA reproduction): LLM Comparator workflow.**
  - Export local-judge outputs into LLM Comparator's JSON: `individual_rater_scores` for the swap runs, `custom_fields` for the human winner and the other judge's verdict. Load it in the hosted demo (FACT: pair-code.github.io/llm-comparator returns HTTP 200).
  - For rationale clusters, reimplement the pipeline: summarise, label clusters, soft-assign. The Python library's `GenerationModelHelper` is an abstract class (FACT), so an `OllamaGenerationModelHelper` subclass takes about 30 lines. For embeddings, use a sentence-transformers model instead of Vertex.
  - **Deliverable:** a demo of what LLM Comparator can and cannot show for judge disagreement. This is the motivating "gap demo".

**Why this is the most defensible target.**
- Exact numbers exist to hit.
- The data is tiny and permissively licensed.
- It is judge-centric.
- The instructor can verify it.

The LLM Comparator part makes the "reproduce prior VA work" requirement concrete.

---

## 8. Project design

### 8.1 Data

**Core: MT-Bench human judgments.** Use the **961 cells with ≥2 human votes**, so the human-ambiguity axis is defined on every item. The remaining 853 single-vote cells can be a hold-out.

**Optional secondary: a 2025-era contamination check.** `lmarena-ai/arena-human-preference-140k` is CC-BY-4.0, not gated, 135,634 rows, about 3 GB (FACT: HF API). Sample 500 items.
- Why: MT-Bench and its GPT-4 judgments have been public since 2023, so current judges may have seen them. That is SPECULATION.
- Avoid `lmsys/chatbot_arena_conversations`. It is gated, and its model outputs are CC-BY-NC-4.0 (FACT: HF card).
- RewardBench 2 (ODC-BY) uses LLM-derived gold labels for some categories (AUTHOR CLAIM, from Nine Judges), so it is weaker as "human" ground truth.

**Education (stretch only).**
- **ASAP sets 7 or 8.** Two independent trained raters plus trait scores, so human–human disagreement and halo are observable (FACT via Sunkavalli).
  - Access needs a Kaggle login and rules acceptance (the page returns HTTP 200; the rules text was not read).
  - The **text cannot be redistributed** (AUTHOR CLAIM via Sunkavalli). Fine for the course; for a paper, release only IDs and scores.
- **PERSUADE 2.0.** CC BY-NC-SA 4.0 (FACT: README). Over 25,000 essays, grades 6–12, holistic 1–6 scores that were double-blind with 100% adjudication (weighted κ = .745 before adjudication). It has gender, race/ethnicity, economic, **ELL** and disability fields (FACT: preprint).
  - It allows the **DIF/fairness slice that Sunkavalli explicitly could not do**.
  - *Unverified:* whether per-rater scores (not just the adjudicated final) are in the CSVs. The data is on Google Drive (the link returns HTTP 200).
- SciEntsBank / Beetle: licence not verified. Skip.

### 8.2 Models (judges)

**J1: `gpt-oss:20b`.**
- 14 GB on Ollama (FACT). It spans 2 × 11 GB GPUs.
- It is a mixture-of-experts model (MoE: only part of the network runs per token), with about 3.6B active parameters. So decode is relatively fast even when split (SYNTHESIS).
- Use `reasoning_effort=low` and cap the output.

**J2: `qwen3.8:27b` (q4_K_M).**
- 18 GB on Ollama (FACT: library tags page). It also spans 2 GPUs.
- It is a hybrid-thinking model. **Disable thinking** (`think:false`), or output length explodes (SPECULATION, but standard for Qwen3-family).

**J0: the released GPT-4 pair judgments.** Free, from the `gpt4_pair` split. This is a fourth, reference judge at zero compute.

**J3 (optional, smaller or different family):** for example `gemma3:12b` (8.1 GB, FACT). It fits one card. Run it after J1 and J2, because J1 + J2 use about 32 GB of the 44 GB.

**Prompt variants (2 per judge):**
- V1 = the official FastChat pairwise prompt.
- V2 = a rubric-structured rewording that is semantically equivalent (helpfulness, correctness, depth), with the rationale required *before* the verdict.

**Controls:**
- position swap on every call;
- temperature 0 with a fixed seed for the main runs;
- a **repeat-run subset** of 150 cells × 5 runs at default temperature, per judge, to estimate instability separately from the swap.

### 8.3 Compute estimate

**Status: SYNTHESIS from public reports. Not measured on the student's machine.**

**Public data points:**
- Qwen3.8-27B Q4_K on a single *22 GB modded* 2080 Ti: about 37 tok/s decode *with* MTP (multi-token prediction) drafting.
- Layer split across 2 cards gave +3.6% ("noise"); prefill was about 777 tok/s (ai-muninn benchmark via WebFetch; AUTHOR CLAIM).

Ollama uses layer split and likely no MTP drafting, so on 2 × 11 GB cards assume **~15–25 tok/s decode and ~400–700 tok/s prefill** for J2. For gpt-oss-20b on Turing-class 11 GB cards split in two, assume **~40–70 tok/s decode** (no direct benchmark found; RTX 3060 12 GB reports of about 30 tok/s involved CPU offload; SPECULATION).

**Per-call tokens:**
- Input is about 1,500 tokens: two conversations of median 313 words each plus the template.
- Output is about 250 tokens for J2 (rationale ≤ 150 words plus verdict), and about 400 for J1 (reasoning plus answer).

**Per-call time:**
- J2 ≈ 2.5 s + 12.5 s ≈ **15 s**.
- J1 ≈ 1.5 s + 7 s ≈ **9 s**.
- J3 ≈ **8 s**.

**The brief's example: 1,000 items × 3 judges × 2 variants, with position swap (×2) = 12,000 calls.**

| Judge | Calls | s/call | GPU-hours |
|---|---|---|---|
| J2 qwen3.8:27b | 4,000 | 15 | ≈16.7 |
| J1 gpt-oss:20b | 4,000 | 9 | ≈10.0 |
| J3 gemma3:12b | 4,000 | 8 | ≈8.9 |
| Repeat subset (150 × 5 × 2 judges × swap) | 3,000 | ~12 | ≈10.0 |
| **Total** | 15,000 | — | **≈45.6 GPU-h sequential** |

**Wall-clock:**
- Running J1 and J2 concurrently on separate GPU pairs gives about **17 h** for the main runs, plus about 9 h (J3), plus about 10 h (repeats) ≈ 36 h. This is unattended over one to two weekends.
- `OLLAMA_NUM_PARALLEL=2–4` may raise aggregate throughput 1.5–2× (SPECULATION).
- **Student time is about 2–3 h** of harness work and monitoring, *if* results go to a resumable JSONL cache keyed by (judge, variant, cell, order, run).

**Minimum viable fallback:** 961 cells × 2 judges × 1 variant × swap = 3,844 calls ≈ 13 GPU-h.

### 8.4 Visualization: views and why each is scientifically necessary

The stack is Streamlit + Plotly with `on_select` selection linked through `st.session_state`. Streamlit is a Python library for building data apps. This means no D3.

1. **Agreement matrix, the baseline view.** This is the judges × judges × human-pool matrix (expert vs. author) with S1/S2 agreement and κ, plus cluster-bootstrap CIs.
   - *Necessary as the control condition.* It is exactly what Langfuse-style tools and papers report.
2. **Disagreement-decomposition scatter.** One point per cell.
   - x = human–human agreement (from the ≥2 votes);
   - y = judge-consensus agreement with the human majority;
   - colour = judge instability (the swap/repeat flip rate);
   - facet = judge.
   - Quadrants separate "humans ambiguous", "judges confidently wrong" and "judges unstable".
   - *Necessary because* pooled κ mixes these three sources. H2 is tested here.
3. **Feature-conditioned small multiples.** Disagreement rate against length gap, turn, category, answer-model family (including GPT-family answers judged by gpt-oss, for self-preference), and position, drawn with **permutation-null bands** and BH-adjusted flags.
   - *Necessary for* H3 and H5. The bands keep the eye from over-reading noise.
4. **Rationale-divergence view.** Rationale clusters, embedded locally and labelled by a local LLM, cross-tabulated by judge × (agrees / disagrees with humans).
   - It answers "what do judges cite when they depart from humans?"
   - *Necessary because* verdict agreement hides reason disagreement (DeLucia 2026). It is also LLM Comparator's "why" panel, extended across judges.
5. **Prompt-variant flip view.** A slope or bump chart per item from V1 to V2 per judge, linked to the other views.
   - *Necessary for* H4: whether flips concentrate in the same ambiguous items.
6. **Item drill-down.** Both answers, every judge's verdict and rationale under both orders and variants, plus the individual human votes.
   - *Necessary for* actionability: checking a pattern against the raw text.

### 8.5 Evaluation

**Reproduction metrics:** see R1a / R1b in §7.

**Statistical tests:**
- H2: judge–human disagreement on split vs. unanimous cells, by cluster permutation.
- H3: mixed-effects logistic model of disagreement ~ features × judge with a random effect for question. The coefficients are reported *and* shown in view 3.
- H4: flip rate between V1 and V2 with CIs; test whether flips concentrate in low human-agreement cells.

**Planted-pattern evaluation (the key objective test, H5).** This creates synthetic judges with known ground truth. Most need no extra LLM calls.

- **P1, category-specific label noise.** Flip 20% of J1's verdicts, only on `coding` items.
- **P2, a length-biased judge.** For items where the length gap exceeds its 75th percentile, override the verdict to "longer wins" with p = 0.5.
- **P3, a position-biased judge.** Use one order only and override to "A wins" with p = 0.3 on turn-2 items.
- **N0, a null control.** Permute J1's verdicts within (category × turn) strata, preserving the marginals.

**Success criteria:**
- P1–P3 each appear among the top-3 flagged slices, BH q < 0.05.
- At least one of them changes pooled κ against humans by less than 0.05. This shows the aggregate masks it.
- N0 produces zero flagged slices.

**Failure criteria:** the tool misses a planted slice, *or* N0 produces flags, which would mean the slice view makes false discoveries.

**Optional mini user study (stretch):** 3–5 participants. Give them a *baseline* (Langfuse-style κ matrix plus confusion matrices plus a spreadsheet) and the *tool*, on planted-pattern tasks. Measure found / not found and time. Report it as formative only.

### 8.6 MVP vs. stronger version

**MVP (grade-safe):**
- R1a + R1b (2 local judges + GPT-4 released, 1 prompt variant, swap);
- R2 LLM Comparator demo;
- views 1, 2, 3, 6;
- planted-pattern evaluation P1–P3 + N0.

Everything is deterministic, and the success criteria are objective.

**Stronger (paper-leaning):**
- \+ second prompt variant and repeat-run instability;
- \+ J3;
- \+ rationale-divergence view 4;
- \+ contamination check on Arena-140k;
- \+ education instantiation: ASAP set 7/8 with two raters, trait scores (halo) and severity strips. Optionally PERSUADE, sliced by ELL status (DIF: differential item functioning, i.e. whether a rater treats subgroups differently at equal quality). Sunkavalli lists the missing DIF analysis as a limitation;
- \+ the mini user study.

### 8.7 Hour budget (about 40 h)

| Task | h |
|---|---|
| Setup, data loading, R1a exact reproduction | 3 |
| Ollama judge harness (prompts, swap, parse, retry, JSONL cache) | 4 |
| Launch and monitor runs; fix parse failures | 2 |
| R1b local-judge agreement table + cluster bootstrap | 2 |
| R2 LLM Comparator JSON export + hosted demo | 2 |
| Proposal (4 pages, due Oct 20) | 4 |
| Item features (length, category, turn, family, swap consistency, human agreement) | 2 |
| Rationale embedding and clustering (local) | 3 |
| Streamlit linked views 1–3, 5–6 (view 4 if time allows) | 8 |
| Planted-pattern evaluation + null control + statistics | 3 |
| Nov 3 update | 1 |
| Final 8-page report + slides + demo recording | 6 |
| **Total** | **40** |

The stretch items (education, user study) are not included and would need +6–10 h.

### 8.8 Main failure modes and mitigations

1. **Local judges are weak or produce parse failures.**
   - Mitigation: constrained output (the `[[A]]` / `[[B]]` / `[[C]]` tags from FastChat, or JSON mode) and retry-then-tie logging.
   - Use J0 (GPT-4) so the tool always has a strong judge.
   - "Judges are bad" is still a valid tool demo.
2. **Thinking-mode output blow-up (Qwen) or reasoning verbosity (gpt-oss).** Disable or limit it, and measure tok/s on 10 items in week 1 before committing to the full grid.
3. **Everything is "already known".** Frame the findings as *confirming known effects visually*. The contribution is decomposition plus planted-pattern validity, not new bias discoveries.
4. **Ambiguity measured from only 2–3 votes.** State it as a noisy estimate. Use Beta-binomial shrinkage, or report human agreement only for cells with ≥3 votes (400 cells, FACT) as a robustness check.
5. **Multiple comparisons in slice discovery.** Use permutation nulls, BH correction, and the N0 null control.
6. **Streamlit linking friction.** Keep to 4–5 views and use Plotly selection events. If linking fails, fall back to filter widgets plus the drill-down table.
7. **Contamination** (MT-Bench is public since 2023). Report it as a limitation, and do the Arena-140k check in the stronger version.
8. **Self-preference confound.** gpt-oss is OpenAI-family and MT-Bench answers include GPT-4 and GPT-3.5, so treat answer-model family as an explicit slice rather than an uncontrolled confound.

### 8.9 Publication path (after the course)

Venue dates are unverified; confirm them.

- **CHI 2027 workshop: HEAL** (Human-centered Evaluation and Auditing of Language Models). EvalAssist appeared at the CHI 2025 edition (FACT: search result). This is the most natural short venue.
- **VIS 2027 short paper**, or a VIS workshop such as VISxAI or an NLVIZ-style workshop.
- **EDM / LAK 2027**, if the education instantiation is done.

**What must be added for a paper:**
1. A second domain (the education instantiation, or Arena-140k).
2. A small user study against the Langfuse-style baseline (n = 6–8 practitioners).
3. Integration of the n_eff or "shared-axis" diagnostics (Nine Judges, Geometry) as views, and a comparison with them.
4. An explicit positioning against Visagreement and LLM Comparator.
5. A release of the tool plus judgment tensors.

---

## 9. Gap verdict

**PARTLY ADDRESSED.**

- *Already done:*
  - single-judge SxS VA with rationale clustering (LLM Comparator);
  - criteria-alignment interfaces (EvalGen, EvalAssist, Evalet, MultEval);
  - two-source agreement dashboards (Langfuse);
  - statistical decomposition of judge unreliability, one layer per paper (Coin Flip, Geometry, Nine Judges, Sunkavalli, Whose Gold?);
  - disagreement VA for *explanation methods*, in the instructor lab (Visagreement).
- *Open:* an integrated VA tool that aligns many judges × variants × repeats × *multiple* human votes on the same items, separates the four disagreement sources per item, and is validated by planted-pattern recovery against a pooled-statistics baseline.
  - Evidence: the negative results in §6.
  - LLM Comparator's own future work lists ">2 models" and "multivariate… correlation between rating scores and fine-grained criteria", but not judges.
  - Coin Flip explicitly lists open-source judges and same-item human comparison as future work.
  - Sunkavalli explicitly lists open-weight checkpoints, rationale-first scoring and DIF as out of reach.

## 10. Novelty confidence (in words)

**Moderate for the course, low-to-moderate for a paper.**

- I am fairly confident no *published* VA tool does multi-judge × human disagreement decomposition as of 2026-09-28. VIS 2025/2026 programmes and LLM Comparator's citing works were checked directly.
- But three things lower the confidence:
  1. The scientific content of each view is already known from 2026 statistical papers, so reviewers will ask "what did the tool reveal that Geometry or Nine Judges did not?"
  2. Visagreement makes the VA design pattern itself prior art from the instructor's lab.
  3. The area moves weekly. PAIR, KAIST, IBM and the Arawjo/Shankar groups are all one step away.
- The planted-pattern validity test and the explicit human-ambiguity axis are the most defensible novel pieces. The open-weight, local-only, fully reproducible angle is a real but modest differentiator.

## 11. 12b-style row

| Candidate | Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|---|
| D3: multi-judge disagreement VA (MT-Bench, local judges) | **High.** Hits the Nov 3 NLP/LLM lecture and model assessment; echoes the instructor's Visagreement | **Medium.** Integration novelty plus a validity test; the ML findings are known | **Low–Med.** Black-box Ollama calls only; the throughput estimate is unmeasured | **Low.** MT-Bench is CC-BY-4.0 and 1.4 MB; education data has licence friction | **Medium.** 4–6 Plotly views in Streamlit; the rationale view is the hardest | **High** for reproduction and planted patterns; soft for "actionability" | …you want a grade-safe project with exact reproduction numbers, low setup risk, and a credible CHI-workshop or VIS-short path, and you accept that the novelty is in the tool and its validation, not in new findings about judges |

## 12. Verify-this-yourself checks

1. **Exact reproduction, about 1 h.** Download the two MT-Bench parquet files. Adapt FastChat's `compute_agreement.py`, which expects JSON; convert the parquet. Confirm 66% / 85% (GPT-4 vs. human) and 63% / 81% (human vs. human). If this fails, the reproduction story needs rethinking before Oct 20.
2. **Throughput.** On the 4 × 11 GB machine, run `ollama run qwen3.8:27b --verbose` and `gpt-oss:20b` on one real 1,500-token MT-Bench judge prompt. Record prompt-eval and eval rates, confirm thinking is off, and check `nvidia-smi` shows both models resident at once. Then re-scale §8.3.
3. **Scoop sweep before the proposal.**
   - Search arXiv for cs.HC with "judge" + (visual OR interactive), from Sept 2026 onward.
   - Check the CHI 2027 and IUI 2027 preprints, and the latest papers from Kahng, Reif (PAIR), Tae Soo Kim / Juho Kim (KAIST), Arawjo, Shankar and Ashktorab (IBM).
   - Check the CHI 2026 full programme, which I did not scrape.
4. **Instructor alignment.** Ask Prof. Silva (or check dblp for Silva/Nonato 2026) whether "Visagreement-style disagreement analytics for LLM judges" is welcome and not already in progress in the lab. Also confirm the P. Silva AAAI AI4Ed 2024 paper; I could not find it in OpenAlex.
5. **Education data.** If you do the stretch:
   - open the PERSUADE 2.0 CSV (Google Drive) to confirm whether per-rater holistic scores exist or only adjudicated finals;
   - read the Kaggle ASAP-AES rules page for course use and redistribution.
