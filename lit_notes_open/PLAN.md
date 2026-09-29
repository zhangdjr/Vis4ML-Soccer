# Open-topic Vis4ML research-gap review: handoff plan

Written 2026-09-28 by the lead reviewer before a Claude Code restart. The restart raises the web-search cap to 600 per session.

## Goal
Run the same kind of skeptical, falsification-first literature review as `../literature_review.md`, but across **any topic**, not just soccer. The soccer candidates stay in the final comparison as a baseline.

## Decisions already made by the user
- **Method:** a broad scan first, then a deep dive on the 3–4 most promising areas.
- **Include as a candidate:** the course default project, "Visual Analytics for AI-Generated Urban Infrastructure Maps" (Silva's Tile2Net line). Check its literature with the same skepticism as everything else.
- **Prior work:** the user has **no** reusable prior biomedical/scientific-imaging code, models or data.
- **Data:** **openly downloadable only.** No credentialed data (no MIMIC/PhysioNet-credentialed sets).
- **Soccer:** **included.** Candidates 1–4 from `../literature_review.md` §11 sit alongside the new ones in the final 12a/12b tables.
- **Compute:**
  - MacBook M1 Pro, 32 GB
  - SLURM HPC cluster
  - Local Ollama models: gpt-oss-20b and others
  - A personal GPU machine running a ~27B Qwen model (exact model and VRAM: see open questions)
  - Maybe no paid LLM API
- **Constraints:** solo; about 40 hours total (3–4 h/week); proposal (4 pages) due **Oct 20**; update Nov 3; final Dec 14. Course facts are in `../literature_review.md` §2.
- **Budget:** about $130 of tokens left this month. Be economical.
- **Independence:** do not read the other research system's reports (`../deep-research-report.md`) while researching. Comparison happens afterwards.

## Output
- `../literature_review_open.md`: same template as the soccer review. That means FACT / AUTHOR CLAIM / SYNTHESIS / SPECULATION labels, evidence levels, a literature table, a gap table, "tempting but bad", 3–5 candidates, 12a + 12b comparison tables (12b uses the columns course fit, research upside, technical risk, data risk, visualization burden, evaluation clarity, and "best suited if…"), and a verification checklist.
- Per-slice notes in this folder.

## Phase 1: broad scan (cheap)
Map about 8–12 candidate areas. For each area, record:
- Data availability (open downloadable?)
- Saturation (what already exists in 2024–26)
- Instructor-lab overlap (Silva/Nonato/VIDA-NYU)
- Course-lecture fit
- Whether visualization is scientifically necessary
- Fit to the user's skills (strong ML/CV/PyTorch; weak frontend)
- Feasibility in 40 hours

Starting areas, to be confirmed or pruned:
1. Course default: VA for AI-generated urban maps (Tile2Net, sidewalk segmentation errors, topology)
2. Segmentation / CV model evaluation VA on open data (failure-slice discovery, uncertainty, calibration of segmentation)
3. LLM interpretability visualization with local open-weight models (SAE features, attention, logit lens), if the GPU permits activation access
4. LLM-as-judge / evaluation reliability visualization
5. Embedding / DR reliability for a domain (text, images, CLIP), beyond soccer
6. Model comparison / Rashomon / underspecification VA on tabular or vision data
7. Data-centric: label-error / dataset-artifact discovery VA (e.g., cleanlab-style on open image datasets)
8. Fairness / subgroup-performance VA (open tabular or vision)
9. Time-series / forecasting model VA (the Nov 17 lecture)
10. Topological data analysis for ML (the Nov 10 lecture; Mapper on representations)
11. Scientific ML (e.g., open climate or remote-sensing datasets)
12. Soccer baseline (already done; reuse)

## Phase 2: deep dive
Deep-dive the top 3–4 areas with the same falsification protocol as the soccer review: citing-paper checks, 2025–26 follow-ups, and repo/data verification. Soccer does not need re-running.

## Lessons from the soccer run
- Semantic Scholar rate-limits (HTTP 429) quickly. Use OpenAlex, the arXiv API and Crossref as fallbacks.
- The WebSearch budget is shared by all sub-agents.
- Sub-agents occasionally misattribute papers (e.g., PassAI's authors). Verify load-bearing claims yourself.
- The instructor's lab had already published the obvious idea (MOUNTAINEER, Visagreement). **Always check Silva/Nonato/VIDA-NYU publications for each area first.**

## Answers from the user (2026-09-28)

- **GPU machine.** 4× RTX GPUs, 11 GB each. So far it has run Ollama only (`qwen3.8:27b`), never HF transformers.
  - SYNTHESIS: interpretability work that needs activations is feasible only for small models, roughly ≤2–4B in fp16 on one 11 GB card. Examples: Gemma-2-2B with the public Gemma Scope SAEs, Pythia, GPT-2.
  - The 27B model is usable only as a black-box text generator via Ollama.
- **Career.** Serve both industry and PhD, leaning toward **publication for PhD applications**. But the **primary goal is a good grade**, so the project must be feasible, not high-risk.
- **Domain.** Any domain is fair game. Take saturation warnings seriously.
  - Nice-to-haves only (do not force them): soccer (done), **maps**, **education**.
- **Budget plan approved.** Sonnet agents for the broad scan, Opus for the deep dives and verification, about 10 agents total.

### Broad-scan slices (Sonnet)

| Slice | Areas covered | Output file |
|---|---|---|
| S1 | Maps / urban: the course default project plus urban segmentation/map-quality VA | `scan_1_maps.md` |
| S2 | Education ML: knowledge tracing XAI, LLM grading/feedback reliability, dropout/fairness | `scan_2_education.md` |
| S3 | LLM-side Vis4ML: small-model interpretability viz, LLM-as-judge reliability, text/CLIP embedding reliability | `scan_3_llm.md` |
| S4 | CV / data-centric model-evaluation VA: segmentation failure slices, label errors, uncertainty/calibration, Rashomon in vision, subgroup fairness | `scan_4_cv_eval.md` |
| S5 | Time series, TDA, scientific ML (climate / remote sensing) | `scan_5_ts_tda_sci.md` |

## Status (2026-09-28)
Done. Final report: `../literature_review_open.md`. Deep dives: `deep_1_tile2net_segrel.md`, `deep_2_tsfm_va.md`, `deep_3_llm_judges.md`.
