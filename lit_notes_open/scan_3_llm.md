# Scan 3 (S3): LLM-side visualization for ML

Scope: (a) interpretability viz for small open models, (b) LLM-as-judge reliability + viz,
(c) embedding/DR reliability for text/CLIP. Today: 2026-09-28. WebSearch calls used: ~19/45 shared budget.
Labels: FACT (verified via fetch/search), AUTHOR CLAIM (paper's own framing, unverified independently),
SYNTHESIS (my inference across sources), SPECULATION (guess, flagged as such).

---

## (a) Interpretability visualization with small open models (SAEs, logit/tuned lens, attention viz)

**Instructor-lab overlap.**
- No Silva/Nonato/VIDA-NYU paper found on SAEs, logit lens, or LLM mechanistic interpretability
  specifically (checked ctsilva.github.io/publications, ResearchGate profile, and targeted search).
  DBLP direct fetch was blocked by bot-protection (Anubis), so this is not exhaustive — SYNTHESIS
  from partial sources, not a clean FACT of "no overlap."
- **Adjacent overlap, not VIDA but named in the brief:** Daniel Kerrigan et al. (Enrico Bertini's
  group, Bertini now at Northeastern, formerly NYU), **"Exploring Text Classification Models with
  Sparse Autoencoders,"** arXiv:2609.21142, posted Sept 17, 2026 — a concept-based **visual
  analytics tool for exploring SAE features on text classifiers.** FACT (arXiv abstract page
  confirmed). This is the single closest overlap to sub-area (a)'s "SAE feature dashboard"
  framing found in this scan — it's fresh (11 days old at scan time), a VA paper (not just an ML
  paper), and by someone the brief explicitly told us to check. **This changes the calculus**: the
  most natural "build a dashboard to explore SAE features" idea is already an active VIS-adjacent
  paper by a near-instructor-network author, submitted weeks ago.

**Key 2023-2026 papers** (one line each):
1. Paulo & Belrose, **"Sparse Autoencoders Trained on the Same Data Learn Different Features,"**
   arXiv:2501.16615 (Jan 2025), preprint, code open (EleutherAI) — the founding result: only ~30%
   feature overlap across seeds on Llama-3-8B SAEs; TopK less stable than L1-ReLU. FACT.
2. Gerasimov et al., **"Unstable Features, Reproducible Subspaces: Understanding Seed Dependence in
   Sparse Autoencoders,"** arXiv, June 10 2026, preprint — directly re-explains (1)'s instability as
   basis ambiguity in reproducible low-rank subspaces; proposes cross-seed pooling. FACT (found via
   arXiv API). **This is essentially the exact RQ implied by the brief, already answered.**
3. Brzozowski & Chung, **"Ablating Archetypes: The Stability of Archetypal SAEs is an Artifact of
   Initialization and Metric Design,"** arXiv, June 1 2026, preprint — shows a claimed stability fix
   was an artifact of deterministic init. FACT.
4. Noël, **"Where You Measure Decides What You Measure: Position Selection in Ablation-Based SAE
   Evaluation,"** arXiv, 2026, preprint — evaluation-protocol sensitivity (7.6-11.9% swings by probe
   position). FACT (title/venue via OpenAlex fetch, not independently re-read).
5. Karne, **"How Far Do Auto-Interpretation Labels Generalize: A Controlled Study Across Languages,
   Scripts, and Rewordings,"** arXiv, May 29 2026, preprint — directly tests whether auto-interp
   feature labels hold up under paraphrase/script changes. FACT. This is the brief's "do auto-interp
   labels hold up" question, already run.
6. Kerrigan et al. (Bertini lab), as above, arXiv:2609.21142, Sept 2026, preprint — VA tool, not yet
   confirmed peer-reviewed/VIS-accepted (unverified).
7. Tooling status: SAELens/TransformerLens/Neuronpedia are actively maintained as of Sept 2026 —
   e.g. a live GitHub issue shows TransformerLens 4.0.0 (released ~Sept 21, 2026) broke SAELens's
   import chain by removing `HookedTransformer`. FACT (GitHub issue observed). Neuronpedia shipped
   Gemma Scope 2 (Gemma 3) and an "Assistant Axis" feature in early 2026 per its own blog — AUTHOR
   CLAIM, not independently verified beyond the blog post existing.
8. Tuned lens follow-up: search turned up mixed/negative results — "Tuned Lens yields at best
   marginal gains and is worse than LogitLens on some architectures" (from a synthesis of 2025-26
   lens papers, not one single paper I pinned down) — SYNTHESIS, weakly sourced, treat as a soft
   signal that lens-method novelty is thinning, not a hard fact.

**Saturation verdict: ACTIVE bordering on SATURATED for the exact RQs implied by the brief.**
Evidence: the two most obvious reproduction+extension angles — "is SAE seed instability real /
what causes it" and "do auto-interp labels generalize" — both have direct, dedicated 2026 preprints
(items 2 and 5) published within the last four months, and a VA-flavored SAE-exploration tool
(item 6) from a near-instructor-adjacent author is *three weeks old* at scan time. This is a fast-
moving field where a solo 40-hour project risks landing exactly on ground already re-covered by a
May-Sept 2026 arXiv preprint. Attention-viz tools (BertViz, LLM Transparency Tool) and logit/tuned
lens are comparatively older and more settled — good for reproduction, weak for extension.

**Open data:** Gemma Scope SAEs for Gemma-2-2B (`google/gemma-scope-2b-pt-res`, HF, gated but free,
Google's license) — FACT, verified HTTP 200. Pythia suite (`EleutherAI/pythia-70m` and siblings, HF,
Apache-2.0) — FACT, verified 200. GPT-2 small (`openai-community/gpt2`, HF, MIT-like) — FACT,
verified 200. Llama-3.2-1B (`meta-llama/Llama-3.2-1B`, HF, Llama license, gated but open-request) —
FACT, verified 200. All fit on the M1 Pro 32GB; SAE inference (not training) on Gemma Scope features
is feasible without the 4xGPU cluster.

**Candidate RQs:**
1. *"Does cross-seed feature pooling (Gerasimov et al. 2026) actually make Neuronpedia-style feature
   dashboards more trustworthy for a human reader, or does it just look more stable?"* Reproduction
   target: Paulo & Belrose (2501.16615, open code) + Gerasimov et al. (2026, check code availability
   — not yet verified). Minimal extension: build a small Streamlit/Plotly side-by-side dashboard
   showing single-seed vs. pooled features on Gemma-2-2B + Gemma Scope, run a small human-rating
   study (n small, exploratory) on label plausibility. Falsification check: this is close to what
   Kerrigan et al. (2026) and the "auto-interp generalization" paper (Karne 2026) already do —
   queries run: "SAE seed instability visualization dashboard", "auto-interp label reliability
   dashboard 2026" (no exact VA-evaluation-of-dashboards paper found, but adjacent ML papers exist).
   This is a *maybe*, not a clean opening.
2. *"Do practitioners misread SAE feature dashboards the way Jeon et al. show practitioners misread
   t-SNE/UMAP?"* — i.e., apply the "Stop Misusing t-SNE and UMAP" methodology (interview + literature
   audit) to SAE feature dashboards instead of DR plots. Reproduction target: Jeon et al.
   arXiv:2506.08725 (VIS 2026, methodology, not code-heavy). Minimal extension: a mini literature
   audit (20-30 papers/blog posts using Neuronpedia dashboards) + a handful of practitioner
   interviews or a survey. Evaluation: qualitative, coded misuse categories. Falsification: not
   found directly done for SAE dashboards as of scan date — best opening in this sub-area, but
   *heavy on qualitative HCI method*, a weak match for this student's stated frontend/quant profile
   and the course's DR/DL grading rubric may want something more quantitative.
3. *"Reproduce Paulo & Belrose's seed-instability finding on a smaller model (GPT-2 small or
   Pythia-160M) — does the 30% overlap number hold, and does dashboard-visible feature stability
   correlate with downstream steering-vector usefulness?"* — clean reproduction (code is open), the
   extension (steering-vector correlation, smaller model) is modest but real, and it's compute-light
   enough for the M1 Pro. Risk: incremental, might read as "yet another instability replication."

**Feasibility (40h):** data/models: easy (all open, small). Compute: easy for inference-only work
on Gemma-2-2B/GPT-2/Pythia; SAE *training* from scratch is a stretch but feasible on the 4xGPU
cluster in a pinch, not needed if reusing Gemma Scope's pretrained SAEs. Frontend: RQ1/RQ3 fit
Streamlit/Plotly (student's stated strength); RQ2 needs interview/qualitative skills not in the
student's stated toolkit.

**Publication path:** VIS short paper or a mechanistic-interpretability workshop (ICLR/NeurIPS
interpretability workshops) if the extension is sharp; otherwise a solid but not novel course
project.

**Red flags:** (1) the two "obvious" RQs are already directly answered by preprints from the last
4 months — high plagiarism-of-idea risk without a sharper twist; (2) auto-interp label quality
evaluation is inherently subjective/human-judgment-heavy; (3) tooling churn (TransformerLens 4.0
breaking SAELens, observed live) means real risk of burning hours on environment setup instead of
research.

---

## (b) LLM-as-a-judge reliability + visualization

**Instructor-lab overlap.** No Silva/Nonato/VIDA paper found specifically on LLM-judge reliability
or judge-evaluation visual analytics. SYNTHESIS/weak-negative (same DBLP-blocked caveat as above).
The closest VIDA-adjacent hit was incidental: "LEVA: Using Large Language Models to Enhance Visual
Analytics" (TVCG 2024) — but that's LLM-*assisting*-VA, not judge-reliability VA, so it's a
false-positive for overlap, not a real one. Note as checked-and-ruled-out.

**Key 2023-2026 papers:**
1. Shankar, Zamfirescu-Pereira, Hartmann, Parameswaran, Arawjo, **"Who Validates the Validators?
   Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences,"** UIST 2024
   (arXiv:2404.12272), peer-reviewed, code open (EvalGen) — the brief's named anchor paper. FACT.
2. **EvalLM**, Kim et al. (KAIST), CHI 2024 (arXiv:2309.13633), peer-reviewed, code open
   (`kixlab/EvalLM` GitHub, verified reachable) — interactive criteria-based LLM output evaluation,
   n=12 user study showing 59% fewer prompt revisions vs. manual eval. FACT.
3. **ChainForge**, Arawjo et al., CHI 2024 (arXiv:2309.09128), peer-reviewed, code open
   (`ianarawjo/ChainForge`, verified reachable) — visual prompt/hypothesis-testing toolkit. FACT.
4. **LLM Comparator**, Kahng et al. (Google PAIR), IEEE VIS 2024 / TVCG 2025
   (arXiv:2402.10524), peer-reviewed, adopted internally at Google (400+ users, 1000+ experiments
   per the paper's own claim — AUTHOR CLAIM) — side-by-side judge-output visual analytics, the
   brief's named tool. FACT (existence/venue); adoption numbers unverified beyond paper's own text.
5. Zufle et al., **"Calibrating LLM Judges for Human and AI Conversations,"** arXiv, 2026, preprint —
   pairwise judging suffers on long transcripts + positional bias; pointwise only moderately
   correlates with humans. FACT (title/venue), reinforces known position-bias story rather than
   opening new ground.
6. Rao & Callison-Burch, **"JEV vs. LLMs as Rubric Judges: Cheaper, Faster, and Wrong in the Same
   Places,"** arXiv, 2026, preprint — LLM judges and cheap classifiers make correlated errors, a
   systematic-bias framing. FACT (title/venue only).
7. "The Coin Flip Judge? Reliability and Bias in LLM-as-a-Judge Evaluation," arXiv:2606.13685
   (2026), preprint — title alone suggests a reliability/variance framing close to the brief's ask;
   not read in full, flagged for the student to check directly. Unverified beyond title.

**Saturation verdict: ACTIVE, not saturated for a VA angle, but the underlying ML finding (judges
are biased) is very well trodden.** The bias catalog (position, verbosity, self-preference) is
common knowledge by now (dozens of 2024-2026 papers); what's comparatively less crowded is a
**visual analytics tool that specifically surfaces *disagreement* between multiple open local
judges** (e.g., gpt-oss-20b vs. Qwen-27B vs. a human) rather than a single judge's bias. LLM
Comparator does side-by-side model comparison, not judge-vs-judge disagreement diagnosis; EvalGen
and EvalLM are about criteria alignment, not multi-judge disagreement visualization. This gap is a
SYNTHESIS from the papers found, not confirmed by an explicit "nobody has done this" statement —
treat as a plausible but unconfirmed opening.

**Open data:** `lmsys/chatbot_arena_conversations` (HF, verified 200, CC-BY-NC-4.0-ish per lmsys
norms — license should be double-checked before use), `lmsys/mt_bench_human_judgments` (HF, verified
200), `ScalerLab/JudgeBench` (HF, verified 200), `allenai/reward-bench-2` (HF, verified 200). All
openly downloadable, no credentials needed — FACT (HTTP 200 checks run this session).

**Candidate RQs:**
1. *"Do two local open judges (gpt-oss-20b, Qwen-27B via Ollama) disagree with each other, and with
   humans, in the same *patterns* the bias literature predicts (position, verbosity, length), and
   can a small linked-view tool (à la LLM Comparator) make that disagreement legible?"* Reproduction
   target: LLM Comparator (Kahng et al., VIS 2024, code open) rebuilt/adapted for a judge-disagreement
   view instead of model-output comparison; validate against MT-Bench human judgments as ground
   truth. Minimal extension: swap "compare two models' outputs" for "compare two judges' verdicts on
   the same outputs," add a verbosity/position-bias annotation layer. Evaluation: agreement
   statistics (Cohen's kappa) + a qualitative walk-through of disagreement clusters. Falsification
   check: queries run — "LLM judge disagreement visualization tool", "multi-judge comparison visual
   analytics 2025 2026" — no exact match found (LLM Comparator is model-comparison, not
   judge-comparison), but this is a moderate-confidence gap, not a verified-empty one.
2. *"Reproduce EvalGen's alignment workflow using only local models (no paid API) — does
   criteria-alignment quality degrade when the 'validator' is gpt-oss-20b instead of GPT-4?"*
   Reproduction target: Shankar et al. UIST 2024 (EvalGen, code likely open — check
   `github.com/eecs-ubc` or the paper's own release, not yet verified this session). Minimal
   extension: local-model substitution + a small-scale replication of the human-alignment gap.
   Feasible, safe, less flashy.

**Feasibility (40h):** compute: fully feasible — gpt-oss-20b and Qwen-27B already run on Ollama on
the 4xGPU boxes per the student's setup; no paid API needed, matching the "maybe no paid LLM API"
constraint exactly. Data: fully open, verified. Frontend: Streamlit/Plotly linked views suit this
well (student's stated strength) — this is probably the **best frontend-effort match in the whole
slice**.

**Publication path:** VIS short paper (judge-disagreement visualization) or an evaluation/NLP
workshop (e.g., GEM, NLP4ConvAI, or a "trustworthy NLP" workshop). Modest but real upside.

**Red flags:** (1) "LLM judges are biased" alone is not novel — the RQ must be framed around the
*visualization/disagreement-surfacing* angle, not the bias-existence finding itself; (2) subjective
human-alignment evaluation is inherently soft; (3) EvalGen's own code/reuse status not yet
independently verified — check before committing.

---

## (c) Embedding/DR reliability for text or CLIP embeddings beyond one domain

**Instructor-lab overlap — the strongest in this slice.** VIDA-NYU (Silva + Nonato) is **actively
publishing in general DR-reliability/explanation space right now**:
- **FADEx: Feature Attribution and Distortion-based Explanation of Dimensionality Reduction**,
  Meneses, Ortigossa, Silva, Nonato — **accepted IEEE VIS 2026** (per the VIS 2026 accepted-papers
  listing). FACT (found via search, appears on ieeevis.org 2026 program listing per search
  snippet — recommend the student directly confirm on ieeevis.org before relying on this). This is
  general DR explanation, not LLM/text-embedding-specific, but it's squarely the same
  methodological neighborhood (why does my DR projection look the way it does) as sub-area (c).
- Nonato & Aupetit have a standing DR-quality-metrics taxonomy (AUTHOR CLAIM per search snippet,
  not independently read this session) — this is prior, well-known lab territory, consistent with
  the brief's MOUNTAINEER/SUBPLEX/Calibrate warning.
- **This is not the LLM-embedding-specific niche**, but it means "DR reliability" as a broad topic
  is thoroughly the instructor's home turf. A project must differentiate by being *about text/CLIP
  embeddings specifically*, not general DR metrics, to avoid directly restating VIDA's own agenda.

**Key 2023-2026 papers:**
1. Atzberger, Cech, Scheibel, Döllner, Behrisch, Schreck, **"A Large-Scale Sensitivity Analysis on
   Latent Embeddings and Dimensionality Reductions for Text Spatializations,"** IEEE VIS 2024 /
   TVCG (arXiv:2407.17876), peer-reviewed, code + data open (Git + Zenodo, per paper) — 42,817
   data points across 3 corpora x 6 embeddings x grid-searched DR hyperparams; exactly the paper
   named in the brief. FACT.
2. Atzberger, Barz-Cech, Scheibel, Döllner, Schreck, **"Evaluating text embeddings for two-
   dimensional text corpora representations,"** 2026 (follow-up journal piece, DOI resolved), status
   peer-reviewed per venue metadata — a direct continuation of (1). FACT (existence), not read in
   depth.
3. Jeon, Park, Shin, Seo, **"Stop Misusing t-SNE and UMAP for Visual Analytics,"** IEEE VIS 2026
   (arXiv:2506.08725), peer-reviewed, scheduled Nov 10 2026 — literature audit of 136 papers +
   practitioner/expert interviews on DR misuse; general DR, not text-specific, but the brief names
   it explicitly. FACT.
4. Jeon et al. (SNU HCI lab), **"Unveiling High-dimensional Backstage: A Survey for Reliable Visual
   Analytics with Dimensionality Reduction,"** CHI 2025 (arXiv:2501.10168), peer-reviewed — 133-paper
   survey/taxonomy of DR-reliability issues. FACT. **Signals this exact space (DR reliability
   broadly) is already comprehensively surveyed by a rival lab (SNU), on top of VIDA's own work** —
   two non-instructor and instructor labs both very active here.
5. van der Hoorn et al., **"Why Can't I See My Clusters? A Precision-Recall Approach to
   Dimensionality Reduction Validation,"** arXiv:2509.04222 (Sept 2025), preprint — supervised
   precision/recall metric for missing cluster structure. FACT.
6. Ren, Hohman, Lin, Moritz (Apple), **"Embedding Atlas: Low-Friction, Interactive Embedding
   Visualization,"** IEEE VIS 2025 (arXiv:2505.06386), peer-reviewed, code open (`apple/embedding-
   atlas`, verified reachable, Apache-2.0 per repo convention — license not independently
   double-checked) — scalable browser-based embedding explorer, WebGPU, up to millions of points.
   FACT.
7. Nomic Atlas (2023) and Latent Scope (2024) — commercial/open-source embedding explorers, cited
   as predecessors to Embedding Atlas per search summaries. AUTHOR-CLAIM-adjacent (from a
   third-party "best tools" roundup, not the primary sources), treat lineage claim as SYNTHESIS.

**Saturation verdict: SATURATED for "DR reliability in general" (VIDA + SNU HCI lab both
publishing surveys and new metrics in 2025-2026); ACTIVE but narrowing for "text/CLIP embedding
DR reliability specifically."** Atzberger et al.'s 2024 paper + its 2026 follow-up already cover
"how do embedding model choice and DR hyperparameters jointly affect text-corpus map layouts" in
a large-scale, systematic way with open code/data — a very close reproduction target. The tooling
side (Embedding Atlas, Nomic Atlas, Latent Scope) is about *browsing* embeddings, not about
*validating* the layout's trustworthiness — that gap (tools that let you explore atlases but don't
tell you when the atlas is lying to you) looks like the most concrete, least-covered opening found
in this slice.

**Open data:** any open text-embedding source works (e.g., 20 Newsgroups, arXiv abstracts, or
CLIP-embedded image sets like a CC-licensed subset of LAION or COCO captions) — the student would
need to pick and verify a specific open corpus; not verified individually this session beyond
noting these are standard open sets. This is a gap in this scan — SYNTHESIS/incomplete, flagged.

**Candidate RQs:**
1. *"Does Atzberger et al.'s large-scale sensitivity-analysis methodology hold when extended to
   CLIP (image-text) embeddings instead of pure text embeddings — i.e., is layout instability worse
   or different in multimodal embedding spaces?"* Reproduction target: Atzberger et al.
   arXiv:2407.17876 (code + data open). Minimal extension: rerun their pipeline (or a scaled-down
   version — fewer corpora/embeddings given the 40h budget) on a CLIP-embedded open image-caption
   dataset. Evaluation: reuse their own 10 layout-similarity metrics for an apples-to-apples
   comparison. Falsification check: queries run — "CLIP embedding dimensionality reduction
   sensitivity analysis", "multimodal embedding projection stability 2025 2026" — no direct hit
   found, best-supported open RQ in this slice, but *not exhaustively confirmed absent* (OpenAlex
   query for this exact phrase returned no matches, a weaker signal than a targeted arXiv full-text
   search would give).
2. *"Build a lightweight 'trust layer' on top of an Embedding Atlas-style browser that flags regions
   of a 2D layout where DR distortion (using Atzberger's or Jeon's metrics) is high, and test
   whether it changes how a small group of users interpret text-embedding clusters."* Reproduction
   target: Embedding Atlas (code open) + a DR-quality metric from Jeon/Nonato-Aupetit taxonomy.
   Minimal extension: overlay distortion-aware shading/uncertainty glyphs. Evaluation: small user
   study (qualitative) or a quantitative "does the flagged region correlate with known
   ground-truth cluster corruption" check. Heavier engineering (JS/WebGPU familiarity helps, and
   the student says frontend is weak) — a real feasibility risk.
3. *"Does the specific finding that TopK SAE features are less seed-stable (from sub-area a)
   correlate with embedding-space DR instability for the same texts?"* — a cute cross-sub-area
   bridge, but likely over-scoped for 40h and reads as scope-creep; not recommended as primary RQ.

**Feasibility (40h):** RQ1 is the most feasible — reuses an open pipeline, swaps in one new open
dataset, keeps to Jupyter/Plotly, no heavy frontend needed. RQ2 is higher-risk given the student's
self-rated weak JS/D3 skills, since Embedding Atlas is a WebGPU/TypeScript tool. Compute: DR +ROC/CLIP
embedding extraction is CPU/single-GPU-friendly, well within the M1 Pro or the SLURM cluster.

**Publication path:** VIS short paper or a workshop on multimodal representation learning /
visualization; RQ1 in particular has a clean "systematic study, extends a named VIS 2024 paper"
framing that VIS reviewers tend to like.

**Red flags:** (1) heaviest instructor-overlap sub-area in this slice — Silva/Nonato are actively
publishing in DR reliability/explanation at VIS 2026 (FADEx), so **any RQ here must explicitly
differentiate as text/multimodal-embedding-specific**, not general DR metrics, or it will read as
restating the advisor's own agenda; (2) SNU HCI lab (Jeon/Seo) has just surveyed this entire space
(CHI 2025) and published the "stop misusing" paper (VIS 2026), so the general framing is crowded
from two directions, not just one; (3) picking and licensing a genuinely open CLIP/text corpus for
RQ1 was not independently verified this session and needs a first-week check.

---

## Final triage table

| sub-area | best RQ | novelty evidence (1 line) | feasibility | grade-safety | pub upside | deep dive? | why |
|---|---|---|---|---|---|---|---|
| (a) SAE reliability | Cross-seed pooled dashboards vs. single-seed, human-rated plausibility | Gerasimov 2026 + Karne 2026 already answer the two obvious RQs directly | Med-High | Med | Med | maybe | Compute/data trivial, but the two sharpest RQs were scooped in the last 4 months by name-matched preprints; needs a genuinely new angle (e.g., dashboard usability, not stability metric) |
| (b) LLM-judge + viz | Multi-local-judge disagreement visualization (gpt-oss-20b vs Qwen-27B) vs. LLM Comparator | LLM Comparator does model-vs-model, not judge-vs-judge; no exact match found (moderate-confidence gap) | High | High | Med | **yes** | Best fit to student's compute (Ollama, no paid API), best frontend match (Streamlit/Plotly linked views), open benchmarks verified, no instructor-lab overlap found |
| (c) Text/CLIP embedding DR reliability | Extend Atzberger et al.'s sensitivity-analysis pipeline to CLIP/multimodal embeddings | No direct hit for "multimodal embedding DR sensitivity analysis"; open code/data to build on | Med | Low-Med | Med-High | **yes** | Clean reproduction target with open code+data and a concrete, named extension, but sits closest to VIDA's own current agenda (FADEx, VIS 2026) — must be framed carefully to avoid overlap, and must independently source/verify a CLIP dataset in week 1 |

## Overlap checks run (falsification queries, non-exhaustive)
"Silva Nonato LLM visual analytics", "VIDA NYU LLM interpretability 2025 2026", "Bertini LLM visual
analytics 2025 2026", "SAE seed instability visualization dashboard", "LLM judge disagreement
visualization tool", "CLIP embedding dimensionality reduction sensitivity analysis", "multimodal
embedding projection stability 2025 2026", plus OpenAlex/arXiv API queries above. DBLP was blocked
by bot protection (Anubis) — Silva/Nonato checks for (a)/(b) are not exhaustive; student should
re-run a direct dblp.org search manually before finalizing a proposal.
