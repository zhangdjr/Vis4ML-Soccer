# Slice 4 (S4): CV / data-centric model evaluation with visual analytics — broad scan

Scope note: WebSearch is a shared budget across all scan agents; this slice used ~12 WebSearch calls plus OpenAlex/arXiv/GitHub API calls via WebFetch/Bash (not counted against the shared budget). Silva/Nonato/Miranda overlap checked via `ctsilva.github.io/publications`, VIDA-NYU site, and targeted searches; dblp.org blocked this session with a bot-check page, so dblp coverage is incomplete — flagged as unverified where relevant.

---

## (a) Failure-slice / error discovery for vision models

**Instructor-lab overlap:**
- FACT: **Calibrate** (Xenopoulos, Rulff, Nonato, Barr, Silva; TVCG 2023, DOI `10.1109/TVCG.2022.3209489`, arXiv:2207.13770) — VA tool for **classifier calibration** (reliability diagrams, subgroup/instance drill-down), not slice discovery per se and not segmentation-specific. Open code: `github.com/VIDA-NYU/pycalibrate`.
- SYNTHESIS: No VIDA/Silva/Nonato/Miranda paper found specifically on **slice discovery** or **systematic error discovery** for vision models. Checked authors of the main 2024–2025 slice-discovery-VA papers found below (AttributionScanner, VISLIX = Bosch Research group: Xuan, Ono, Gou, Ma, Ren; VibE = Yuan, Cavallo et al.) — none overlap with VIDA-NYU. dblp query for Silva was blocked (bot-check), so this is not exhaustive; unverified beyond what Google/OpenAlex surfaced.
- This is a genuine gap relative to the "instructor already did it" risk pattern from the soccer review.

**Key 2023–2026 papers:**
1. Eyuboglu et al., "Domino: Discovering Systematic Errors with Cross-Modal Embeddings," ICLR 2022 (arXiv:2203.14960). Peer-reviewed. CLIP-embedding mixture model for slice discovery. Code: `HazyResearch/domino`, Apache-2.0, 143 stars, last pushed Oct 2023 — **stale** (FACT, GitHub API).
2. Johnson, Cabrera, Plumb, Talwalkar, "Where Does My Model Underperform? A Human Evaluation of Slice Discovery Algorithms," AAAI HCOMP 2023. Peer-reviewed. N=15 user study: practitioners struggle to use slice-discovery output. Shows human-eval methodology here is itself open/contested.
3. Slyman, Kahng, Lee, "VLSlice: Interactive Vision-and-Language Slice Discovery," ICCV 2023. Vision-language, not segmentation.
4. Olesen, Weng, Feragen, Petersen, "Slicing Through Bias..." MICCAI 2024 Workshop (arXiv:2406.12142). **Classification** (chest X-ray), not segmentation; code link unverified.
5. Xuan, Ono, Gou, Ma, Ren, "AttributionScanner," arXiv:2401.06462 (v4 Feb 2025). Preprint, venue unconfirmed. VA system, metadata-free slice finding; code unverified.
6. Yan, Xuan, Ono, Guo, Mohanty, Kumar, Gou, Wang, Ren, "VISLIX," Computer Graphics Forum (EuroVis) 2025, DOI `10.1111/cgf.70125`, arXiv:2505.03132. Peer-reviewed. Foundation-model-driven slice discovery + NL insights, expert study on object detection; code unverified.
7. Yuan, Miao, Oh, Walker, Xue, Katolikyan, Cavallo, "VibE," IUI 2025 (arXiv:2503.20112). Peer-reviewed. CLIP+GPT-4 subgroup summarization across "three diverse CVML tasks" — task scope ambiguous, possibly includes segmentation (unconfirmed).
8. Ghosh et al., "LADDER," ACL Findings 2025. Classification only.
9. dcbench (`data-centric-ai/dcbench`) — slice-discovery **evaluation benchmark** (precision@k, CelebA/ImageNet). FACT: Apache-2.0, 72 stars, **last pushed 2022-06-08**, stale (GitHub API). Note: "SliceBench" is an unrelated **program-slicing** benchmark — false lead.

**Saturation verdict:** **Active, not saturated**, especially the VA-tool side (2024–2026: AttributionScanner, VISLIX, VibE, GH-ESD arXiv:2512.24592, "Error Slice Discovery via Manifold Compactness" arXiv:2501.19032). But almost all VA-tool work targets **classification/object detection**; segmentation-specific slice discovery is thin (only found via the medical-imaging classification paper, not true segmentation). Evaluation of slice-discovery methods is itself an open, small sub-literature (dcbench is stale; AAAI HCOMP human-eval paper is the main one). No VIDA overlap found.

**Open data:** dcbench slices are built on **CelebA** and **ImageNet** validation sets — both freely downloadable, no NDA (ImageNet requires free account registration; verified reachable). No segmentation-specific slice-discovery benchmark found.

**Candidate research questions:**
1. **RQ:** Does a slice-discovery method find coherent failure slices for a **segmentation** model (mIoU-based per-region failure), and does a lightweight VA interface improve hypothesis formation vs. the AAAI HCOMP protocol adapted to segmentation? Reproduction target: Domino + Johnson et al.'s eval protocol. Minimal extension: adapt Domino's scoring to per-instance IoU-below-threshold on Cityscapes/VOC, small Streamlit linked-view UI. Evaluation: precision@k on injected shortcut/rare slices (dcbench-style) + informal qualitative walkthrough. Falsification check: ran "segmentation slice discovery," "error slice discovery segmentation," "Domino segmentation" — nothing doing exactly this; closest is Olesen et al. (medical classification) and VibE (ambiguous scope, unconfirmed risk).
2. **RQ (safer):** Reproduce Domino on its original benchmarks, then refresh dcbench's precision@k protocol with a 2025-era embedding (CLIP ViT) as one added baseline, testing whether its 2022 conclusions still hold. Lower novelty, very safe.

**Feasibility in ~40h:** Data easy (all open; Cityscapes needs free registration). Compute fits M1 Pro/4xRTX11GB; keeping the VA layer Streamlit-based (not foundation-model-driven like VISLIX/VibE) avoids GPT-4/CLIP-scale infra dependence. Frontend burden low-medium in Streamlit/Plotly vs. custom D3 (matches weak-JS profile).

**Publication path:** VIS/EuroVis short paper if segmentation angle is genuinely novel; else a data-centric-AI/trustworthy-CV workshop (CVPR/ICCV).

**Red flags:** (1) VibE's scope ambiguous — confirm it excludes segmentation before committing. (2) HCOMP-style human eval is unrealistic solo; keep informal. (3) Domino repo stale since Oct 2023 — expect setup friction.

---

## (b) Uncertainty / calibration visualization for segmentation

**Instructor-lab overlap:**
- FACT: **Calibrate** (TVCG 2023) exists but is explicitly **classification-only** (reliability diagrams over class probabilities), not segmentation, not pixel-wise. This is the closest VIDA precedent and a natural "reproduce, then extend to segmentation" target.
- SYNTHESIS: No VIDA/Silva/Nonato paper found on segmentation uncertainty, pixel-wise calibration, or conformal-prediction visualization. (dblp check blocked; not exhaustive.)

**Key 2023–2026 papers:**
1. Xenopoulos, Rulff, Nonato, Barr, Silva, "Calibrate," TVCG 2023 (arXiv:2207.13770). Peer-reviewed. Open code (`VIDA-NYU/pycalibrate`). **Reproduction target.**
2. Mossina, Dalmau, Andéol, "Conformal Semantic Image Segmentation," CVPR Workshops 2024. Peer-reviewed. Heatmap viz of conformal sets on Cityscapes/ADE20K/LoveDA; open code `deel-ai-papers/conformal-segmentation`. Technical method paper, **not a VA system**.
3. "Conformal Prediction for Image Segmentation Using Morphological Prediction Sets" (deel-ai group), MICCAI 2025, code `deel-ai-papers/consema`. Same pattern: rigorous stats, minimal interactivity.
4. "CONSIGN: Conformal Segmentation Informed by Spatial Groupings via Decomposition," arXiv:2505.14113 (2025). Preprint.
5. "Controlling False Positives in Image Segmentation via Conformal Prediction," arXiv:2511.15406 (2025). Preprint.
6. Smith & Ferrie, "U-SEG: Uncertainty in SEGmentation," arXiv:2605.15421 (May 2026). Preprint. Large empirical study, semantic+panoptic segmentation uncertainty; technical benchmarking, not a viz tool.
7. "Uncertainty-aware segmentation quality prediction via deep learning Bayesian Modeling," ScienceDirect (skin cancer + liver) — venue/date unverified.
8. IEEE VIS runs an annual **Uncertainty Visualization workshop** (tusharathawale.github.io, 2024/2025/2026) confirming community activity, but no segmentation-specific accepted paper surfaced by name (program not fetched, budget-conscious).

**Saturation verdict:** **Active but split** — the *statistics/method* side (conformal prediction for segmentation) is quite active 2024–2026 with multiple open-source repos; the *visual-analytics/interactive-tool* side for segmentation uncertainty is **thin** — most papers show static heatmaps, not linked-view interactive systems. This is a plausible gap: "VA tool for interactively exploring pixel-wise uncertainty/calibration across an image dataset" doesn't appear to exist as a dedicated contribution, extending Calibrate's paradigm to segmentation.

**Open data** (all homepages FACT-verified reachable via curl, HTTP 200):
- **Cityscapes** — free non-commercial, needs email registration.
- **ADE20K / MIT SceneParsing** — open, some versions need simple registration.
- **Pascal VOC** — fully open, no registration.
- **Medical Segmentation Decathlon** — CC BY-SA-style, no registration (license detail per general knowledge, not re-verified).
- **ISIC Archive** — largely open/CC0-like, no NDA.
- **KiTS23** — CC BY-NC-SA 4.0 (verified via WebFetch of challenge page), 599 cases (489 train/110 test), no formal DUA.
- **LIDC-IDRI** (TCIA) — CC-style attribution, free TCIA account, no NDA (general knowledge, not deeply re-verified).

**Candidate research questions:**
1. **RQ:** Extend Calibrate's reliability-diagram + subgroup-drill-down paradigm from classification to **per-pixel calibration in semantic segmentation** (is confidence calibrated per-class/region-size/boundary-distance?). Reproduction target: Calibrate (open code). Minimal extension: linked-view Streamlit/Plotly tool with pixel-wise reliability diagrams stratified by class/boundary-distance + confidence-overlay image viewer, on Cityscapes/VOC with a pretrained DeepLabV3+/SegFormer. Evaluation: per-pixel ECE-style calibration error before/after temperature scaling, plus a qualitative "does it surface known failure patterns" walkthrough. Falsification check: ran "conformal prediction segmentation visualization interactive tool," "uncertainty visualization segmentation calibration VIS 2024 2025" — found rigorous stats papers with static heatmaps (Mossina et al., consema, CONSIGN) but no interactive tool extending Calibrate to segmentation. **Strongest, most defensible gap in this slice**: open VIDA reproduction target, clean minimal extension, open data, low engineering.
2. **RQ (higher-risk stretch):** Same tool applied to a medical task (MSD/ISIC) where miscalibration stakes are higher; adds domain-shift/preprocessing overhead — keep as stretch goal only.

**Feasibility in ~40h:** High. Pretrained backbones run inference-only on M1 Pro; 4xRTX11GB if fine-tuning/temperature-scaling needed. Streamlit/Plotly matches tool preference and weak-JS constraint (advantage over VISLIX/AttributionScanner's custom D3 UIs). Cityscapes registration is a minor lead-time item — start early.

**Publication path:** VIS/EuroVis short paper, or the (verified, annually-run) VIS Uncertainty Visualization workshop.

**Red flags:** (1) Haven't tested that `pycalibrate` still runs/adapts. (2) Segmentation calibration is crowded on the *stats* side — must keep the framing tool/interactivity-centric to differentiate from Mossina et al./CONSIGN. (3) Medical dataset licenses (MSD/KiTS/ISIC) have non-commercial/attribution nuances — fine for coursework, state explicitly in proposal.

---

## (c) Label-error discovery VA (Confident Learning / cleanlab) for segmentation

**Instructor-lab overlap:** SYNTHESIS: No VIDA/Silva/Nonato paper found on label-error discovery or confident learning (checked via targeted search; dblp blocked, not exhaustive). No overlap detected.

**Key 2021–2025 papers:**
1. Northcutt, Athalye, Mueller, "Pervasive Label Errors in Test Sets Destabilize Machine Learning Benchmarks," NeurIPS 2021 D&B track (~102 citations per OpenAlex). Peer-reviewed. Code: `cleanlab/label-errors`. Classification-only (ImageNet, MNIST, CIFAR).
2. Lad & Mueller, "Estimating label quality and errors in semantic segmentation data via any model," ICML 2023 Workshop on Data-centric ML (arXiv:2307.05080). Peer-reviewed workshop paper. **The segmentation-specific extension** — 7 label-quality scoring methods on modified SYNTHIA + DeepLabV3+/FPN; algorithmic basis for cleanlab's segmentation module.
3. cleanlab library: FACT (GitHub API) — 11,681 stars, Apache-2.0, actively maintained. Segmentation tutorial at `docs.cleanlab.ai/.../tutorials/segmentation.html` (FACT, HTTP 200).
4. "How we built Cleanlab Vizzy" (cleanlab blog) — interactive viz of confident learning, but a **product blog post, not a peer-reviewed paper** (AUTHOR CLAIM). UI inspiration only.
5. "Confident Learning-Based Label Correction for Retinal Image Segmentation" (PMC) — CL + human-in-loop re-annotation, medical segmentation; venue unverified.
6. "Robust machine learning for imperfect labeled image segmentation" — appears to be a **patent**, not a paper — do not cite as research.

**Saturation verdict:** **Algorithm solved, VA side open.** Confident learning for segmentation is done and shipped (Lad & Mueller 2023 + cleanlab). No peer-reviewed VIS-community interactive VA paper for triaging segmentation label errors exists — Vizzy is the closest artifact and is a blog post, not a paper. Same shape as (b): strong open reproduction target, no published VA layer.

**Open data:** SYNTHIA (used in Lad & Mueller) — synthetic, research-use (not independently re-verified). Better default: Cityscapes/VOC (already verified open) + injected synthetic label noise for ground-truth error evaluation.

**Candidate research questions:**
1. **RQ:** Build a VA tool (ranked "most-likely-mislabeled" list + per-image mask-vs-prediction overlay + confidence heatmap) on cleanlab's open segmentation label-quality scores; evaluate whether it finds synthetically injected label errors faster/more accurately than a ranked-list-only baseline. Reproduction target: Lad & Mueller 2023 + cleanlab (both open). Minimal extension: the interactive triage UI (existing docs are static tutorials only). Evaluation: precision@k/recall of injected errors + informal time-to-find. Falsification check: ran "visual analytics label errors confident learning cleanlab segmentation masks" — found the algorithm and library and the Vizzy blog post, no academic VA paper. **Biggest risk:** Vizzy may already cover much of the intended UI — must trial it before committing.
2. **RQ (safer fallback):** Reproduce Lad & Mueller's score comparison on VOC + injected noise instead of SYNTHIA, add one new visualization as the extension. Lower novelty, low risk.

**Feasibility in ~40h:** High — cleanlab is mature/documented; injecting synthetic label noise is simple scripting; compute is light (inference-only). Streamlit table + image viewer matches skill profile.

**Publication path:** Data-centric-AI workshop (CVPR/NeurIPS), or VIS short paper if interactivity is emphasized.

**Red flags:** (1) Must hands-on test Vizzy first — real overlap risk. (2) "Wrap a library in a dashboard" reads as low-novelty without a clear evaluation question quantifying the VA layer's benefit. (3) SYNTHIA license not independently re-verified — default to VOC/Cityscapes + synthetic noise instead.

---

## (d) Rashomon / underspecification / seed-variance in vision models

**Instructor-lab overlap:** SYNTHESIS: No VIDA/Silva/Nonato paper found specifically on Rashomon sets, model multiplicity, or underspecification in vision (targeted search, dblp blocked — not exhaustive). No overlap detected, but also no VIDA "toehold" to build from (weaker reproduction anchor than (b)/(c)).

**Key papers:**
1. D'Amour et al., "Underspecification Presents Challenges for Credibility in Modern Machine Learning," JMLR 2022 (arXiv 2020). FACT: foundational, general ML, not vision-specific, not visualization. Too broad to reproduce as-is.
2. Donnelly, Guo, Barnett, McTavish, Chen, Rudin, "Rashomon Sets for Prototypical-Part Networks..." (Proto-RSet), CVPR 2025. Peer-reviewed. Real-time interactive editing of ProtoPNet via Rashomon sets (bias removal + medical imaging); code unverified. Rudin lab (Duke), not VIDA.
3. "Exploring the Rashomon Set for Concept-Based Models," arXiv:2511.19636 (Nov 2025). Preprint, concept-bottleneck-specific, not general vision multiplicity.
4. "Resolving Predictive Multiplicity for the Rashomon Set," arXiv:2601.09071 (2026). Preprint, general ML.
5. "Systemizing Multiplicity: The Curious Case of Arbitrariness in Machine Learning," arXiv:2501.14959 (2025). Preprint survey.
6. "DIVERSE: Disagreement-inducing vector evolution for Rashomon set exploration," reported as ICLR 2026 per search snippet — **unverified**, not confirmed on OpenAlex/arXiv.
7. Adjacent VIS work is about ensemble/contour visualization for scientific fields, not learned models (e.g., "Probabilistic Inclusion Depth for Fuzzy Contour Ensemble Visualization," TVCG 2024) — a plausible but unverified methodological bridge to segmentation-mask disagreement.

**Saturation verdict:** Active on ML-theory side (2025-2026 has several Rashomon papers), thin on vision-VA side. No paper visualizes prediction multiplicity/seed-variance for **segmentation** with linked views.

**Open data:** Any segmentation set with easy multi-seed retraining (VOC or a Cityscapes subset); feasible on 4xRTX11GB for a small backbone, not for large-scale studies.

**Candidate research questions:**
1. **RQ:** Train ~10-20 seed-varied segmentation models on VOC/Cityscapes-subset; build a VA tool (contour-ensemble ideas, e.g. inclusion/band depth) to explore where models disagree and whether disagreement correlates with boundaries/rare classes/annotation ambiguity. No clean single reproduction target exists — this is a **novel combination**, which **raises grade-safety risk** given the course's reproduce-first requirement. Falsification check: ran "Rashomon set model multiplicity visualization computer vision," "visualizing prediction multiplicity segmentation ensemble disagreement" — no existing match found, but also no clean paper to reproduce.
2. **Fallback:** Reproduce a narrow slice of D'Amour et al. (seed-variance in per-class IoU) as the reproduction target, then add the multiplicity-viz tool as extension.

**Feasibility in ~40h:** Medium-risk — multi-seed retraining is compute/time-heavy for 3-4h/week, and lack of one clean reproduction target adds planning risk.

**Publication path:** Possible VIS short paper if framed around the visualization contribution; weaker than (b)/(c).

**Red flags:** (1) No single reproduction-target paper — more "synthesize two literatures," a process risk given the course's explicit requirement. (2) Multi-seed compute/time budget is tight. (3) Rashomon literature is theory/tabular-heavy; vision-specific work is newer/less consolidated, raising risk of being scooped before December.

---

## (e) Concept-based explanations (brief, saturation check only)

**Instructor-lab overlap:** Not checked in depth given the brief's instruction to keep this short; no obvious VIDA connection surfaced incidentally in other searches this session.

**Key papers (brief):** TCAV (Kim et al. 2018), CRAFT (Fel et al., CVPR 2023), Concept Bottleneck Models (Koh et al. 2020) and multiple 2024–2025 follow-ups found this session: "Concept Bottleneck Models Without Predefined Concepts" (arXiv:2407.03921, 2024), "Overlooked Factors in Concept-Based Explanations" (CVPR 2023, Ramaswamy et al.), "α-TCAV: A Unified Framework for Testing with Concept Activation Vectors" (arXiv:2605.15688, 2026), "Learning Concept Bottleneck Models from Mechanistic Explanations" (arXiv:2603.07343, 2026), plus a 2025 xAI-CV overview survey (arXiv:2509.18913).

**Saturation verdict: SATURATED.** Steady stream of incremental papers through 2026 (concept leakage, concept learnability, mechanistic-explanation variants), a recent general survey exists (xAI-CV, 2025), and the "sanity checks for saliency" critique lineage (Adebayo et al. 2018) is old and well-trodden. Per the brief's instruction, this is marked saturated and not developed into full candidate RQs — high risk of redoing well-covered ground.

---

## Final triage table

| Sub-area | Best RQ | Novelty evidence (1 line) | Feasibility | Grade-safety | Publication upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) Slice discovery for vision | Segmentation-specific slice discovery + lightweight VA, evaluated dcbench-style | No segmentation-specific slice-discovery VA paper found; dcbench stale since 2022 | Med-High | Med (VibE's exact task scope unconfirmed) | Med | maybe | Real gap but needs VibE full-text check first |
| (b) Uncertainty/calibration viz for segmentation | Extend Calibrate (VIDA, open code) from classification to per-pixel segmentation calibration | Conformal-segmentation papers (2024-2026) are static-heatmap/technical only; no interactive linked-view extension of Calibrate found | High | High | Med-High | **yes** | Clean open VIDA reproduction target + clear minimal extension + matches skill/tool profile |
| (c) Label-error discovery VA for segmentation | VA triage tool on top of cleanlab's segmentation label-quality algorithm | Algorithm (Lad & Mueller 2023) is open and shipped; no academic VA paper found, only a product blog demo (Vizzy) | High | Med (must check Vizzy overlap first) | Med | **yes** | Second-strongest gap; must hands-on test Vizzy before committing |
| (d) Rashomon/multiplicity in vision | Multi-seed segmentation ensemble + contour-ensemble-style disagreement visualization | No vision-segmentation-specific Rashomon-visualization paper found, but also no single clean reproduction target | Med (compute/time tight) | Low-Med (no clear reproduction anchor) | Med | maybe | Interesting but violates course's reproduce-first requirement more than (b)/(c) |
| (e) Concept-based explanations | — | Survey exists, steady incremental 2024-2026 output, old sanity-check critiques | — | — | — | no | Saturated per brief's own criterion |

---

## Labels used
FACT = independently verified this session (GitHub/arXiv/OpenAlex API, or HTTP reachability check). AUTHOR CLAIM = taken from abstract/blog without independent verification. SYNTHESIS = my inference across multiple search results. Several items marked "unverified" throughout where a single search snippet was the only source (e.g., some venue/date details, dblp-derived facts, blog-product comparisons) — these should be re-checked before the proposal is finalized.
