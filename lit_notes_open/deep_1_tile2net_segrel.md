# Deep dive D1: Segmentation reliability VA in the Tile2Net pipeline (± extending Calibrate to per-pixel segmentation)

Review date: 2026-09-28. Falsification-first. Labels: **FACT** (checked this session by reading code, an API response or full text), **AUTHOR CLAIM** (what a paper says about itself), **SYNTHESIS** (my inference from several facts), **SPECULATION** (a guess, flagged). Evidence levels for sources: FULL TEXT / ABSTRACT / SECONDHAND.

Budget used: 9 WebSearch calls. The rest went through WebFetch, the arXiv API, OpenAlex (its free daily budget ran out partway through, see §10), the Socrata (NYC Open Data) API, `gh api` and `curl -I`/range-GETs. Full texts saved in `lit_notes_open/pdfs/d1_*.txt`.

---

## 0. TL;DR verdict

- **The candidate RQ as written is only partly open. It has to be narrowed before it is defensible.**
  - **FACT:** The *tool* half ("linked views that let users trace graph errors back to pixel causes") has already been built, at student-project level, **in this same course last year**. `george-gideon-S/SegNetVis` (Dec 2025) advertises a dual-panel Segmentation Detective + Network Quality Inspector that "trace[s] errors from pixels to network topology" and includes a confidence filter. The instructor has seen this idea.
  - **FACT:** "Prediction confidence vs. accuracy" is listed as a Key Question of **Track A** on the course page. Plain calibration of Tile2Net is therefore *inside the default track*, and several classmates may attempt some version of it.
- **The quantitative half looks open.** Across ~25 targeted queries (§10) I found **no paper** that does any of the following:
  - (i) measures whether Tile2Net's (or any aerial-sidewalk model's) pixel confidence is *calibrated*;
  - (ii) *decomposes* downstream pedestrian-network topology errors into segmentation-induced vs. vectorization-induced;
  - (iii) tests whether probability-derived features predict which network gaps are real missing links.
- The closest neighbours each cover one piece but not the combination:
  - Gupta et al., NeurIPS 2023: structure-wise uncertainty for curvilinear roads and vessels.
  - Prophet (Zhang/Howe/Caspi 2024): stores mean pixel probability as an edge "confidence", but never validates it.
  - PathwayBench 2024: shows Tile2Net over-fragments networks, with no uncertainty angle.
- **Recommended RQ (refined):** *"In Tile2Net on held-out NYC imagery: (1) where is pixel confidence miscalibrated? (2) What share of network topology errors comes from segmentation vs. the polygon→network stage? (3) Does a 'bottleneck confidence' computed from the probability map separate true missing links from true gaps among predicted dead-end pairs?"*
  - Delivered with Calibrate-style linked views (learned reliability diagram + brushable pixel-feature histograms + linked map + gap list).
  - The grade-safe version (RQ1 plus views) is a strict subset, finished by the Nov 3 update.
- **Single biggest risk:** RQ3 turns out trivial or null. Most Tile2Net disconnections may come from the vectorizer, not low-confidence pixels. The Tile2Net paper itself admits that its centerline fitting "is not optimized for fitting the centerlines such that the endpoints of one skeleton topologically connect to the skeleton of another polygon, resulting in discontinuities" (AUTHOR CLAIM, FULL TEXT). If so, confidence cannot explain most graph errors.
  - The RQ2 oracle-mask ablation (§6) is designed to detect this cheaply in week 1–2.
  - A "the vectorizer dominates" finding is still a reportable result, not a dead project.

---

## 1. Course fit (default-project page, FACT via WebFetch)

Source: `ctsilva.github.io/2026-VisML-CDS/slides/default-project.html`.

- **Framing:** "design visualization systems to help domain experts understand, debug, and analyze ML pipelines that generate urban infrastructure maps from aerial imagery," built on Tile2Net.
- **Track A, "Segmentation Detective":**
  - Goal: "diagnose and understand failures of the semantic segmentation model at the pixel and tile level".
  - Key questions include the **"relationship between prediction confidence and accuracy"** and environmental factors (shadows, tree cover).
  - Required features: linked views (aerial image, GT, predictions, **confidence maps**), error overlay (FP/FN), filters (error type, **confidence threshold**), magnification lens.
- **Track B, "Network Quality Inspector":** disconnected components, intersections, automatic flagging (dead-ends, isolated subgraphs), diff vs. OSM/city data, graph metrics.
- **Track C, "Urban Time Traveler":** temporal diffs. Not relevant here.
- **Rules:**
  - "You can choose one direction or propose a **hybrid/variation**."
  - Deliverables: 4-page proposal (Oct 20); 8-page final in "conference paper format, e.g., IEEE VIS"; GitHub repo with setup instructions; 3–5 min demo video.
  - Students must "install and run the pipeline".
  - Team size and grading rubric are not stated on the page.

**Is D1 inside a track?** SYNTHESIS:
- The *calibration* half is squarely Track A.
- The *pixel→graph error* half is an explicit A×B hybrid, which the page allows.
- The Calibrate extension is not mentioned anywhere on the page. That is a plus for differentiation.

**Classmate overlap.** I cannot know it, but there is hard evidence of prior-cohort work.
- `gh search repos tile2net` returns 3 student repos created 2025-12-08 to 2025-12-15, the week of last year's final deadline (FACT):
  1. **SegNetVis** ("Dual-panel visualization system bridging pixel-level segmentation analysis with graph-level…")
     - Mapbox/deck.gl JS app.
     - Features: confidence heatmap "where available", confidence-range filter, TP/FP/FN overlay, lens, betweenness/bridges/articulation points, OSM overlay, human-in-the-loop flag validation.
     - Its README reports a Manhattan demo with "85.2% Mean IoU" vs. "40.6% Network Connectivity".
     - The code falls back to "synthetic demonstration data" when files are missing.
     - It has **no calibration analysis** (grep for calibrat/reliab: none).
  2. **Tile2Net-Inspector**: Boston Common example; IoU/P/R/F1/mean displacement error, dead-ends, intersections.
  3. **Shadow-Impact-Analysis**: NYC; claims shadows reduce road F1 by ~20%.
- SYNTHESIS: this confirms the default project ran in Fall 2025, with the same tracks, and that **"a linked pixel+network VA tool" is the modal default project**.
- The differentiator has to be a *measured, falsifiable finding*, not the tool. None of the three repos:
  - computes a reliability diagram;
  - uses held-out imagery deliberately (SegNetVis used Manhattan, which is in Tile2Net's training data);
  - separates segmentation-caused from vectorizer-caused graph errors.

---

## 2. Reproduction-target verification

### 2.1 Tile2Net (VIDA-NYU/tile2net)

- **Repo status (FACT via `gh api`):**
  - BSD-3-Clause, 228 stars, last push 2026-09-12, v0.5.0.
  - Actively refactored: Sept 2026 commits by D. Hodczak (UIC) on Nominatim and source lookup.
  - Python `>=3.11,<3.15`.
- **Dependencies (FACT, `requirements-dev.txt`):**
  - Mostly unpinned: torch, torchvision, geopandas>=1.0, shapely>=2.0.7,<3, osmnx, rasterio, numba, centerline, numpy>=1.26,<3.
  - One pin: `argh==0.29.4`. No stale pins.
  - Risk is the opposite of staleness: unpinned torch/CUDA means install-time drift.
- **Hardware (FACT):**
  - README: "one CUDA-enabled GPU for inference".
  - Issue #77 (2025): the maintainer confirms "does **not support inference using only CPU**".
  - The M1 Pro is out; use SLURM or the 4×RTX 11 GB box.
  - Whether HRNet-W48-OCR-Mscale inference on 1024² stitched tiles fits in 11 GB is **unverified**. SPECULATION: likely yes at batch size 1.
- **Weights (FACT):**
  - `satellite_2021.pth` (578,433,769 B) and `hrnetv2_w48_imagenet_pretrained.pth` (310,643,500 B), on Figshare DOIs 10.6084/m9.figshare.33315570.v1 / .33315558.v1.
  - SHA-256 pinned in `raster/weights.py`.
  - A range-GET of the first 1 KB returned HTTP 206. A HEAD on the signed S3 redirect returns 403, which is normal for presigned GET URLs.
  - Not downloaded, per the brief.
- **Classes (FACT, `tileseg/datasets/satellite.py`):**
  - `num_classes = 4`: Sidewalk (0), Road (1), Crosswalk (2), Background (3).
  - The paper's "footpath" is not a separate output class in the released model.
  - Architecture `ocrnet.HRNet_Mscale` with multi-scale/flip averaging (`trnval_utils.eval_minibatch`).
- **Does it output per-pixel probabilities? (FACT from code, not yet run.)**
  - `eval_minibatch` computes `softmax(output)` over the 4 classes, then `max_probs, predictions = output_data.max(1)`, and stores `assets['prob_mask'] = max_probs`.
  - The **full 4-class softmax is discarded** after that.
  - `ThreadedDumper.save_prob_and_err_mask` writes `{img}_prob.png` (max-prob quantised to uint8) **only if `'err_mask' in dump_dict`**. In `Inference.validate`, `dumpdict = dict(gt_images, input_images, img_names, assets)`, so `err_mask` is never a top-level key. SYNTHESIS: in the shipped inference path, **no probability map is written to disk.**
  - **Polygons are built from the colourised argmax PNG** (`self.map_features(tile, np.asarray(prediction_pil), …)`). Probabilities never reach the polygon/network stage.
  - **Implication (SYNTHESIS):** an estimated ~10–20-line patch is needed to dump `output_data` (float16, 4×H×W per stitched tile) next to each tile. At zoom 19 in NYC (≈0.226 m/px, my calculation), a 1.5 km² AOI is ≈450 tiles ≈ 29 M px ≈ 235 MB at fp16×4. Trivial.
  - A clean patch plus flag (`--dump_softmax`) is also a nice upstream PR to the instructor's repo.
- **Supported regions and imagery (FACT, `BASICS.md` and `raster/source.py`):**
  - Regions: NYC (higher res), NY/NJ/MA/DC/VA states, Alameda, LA, King County WA, Spring Hill TN, San Francisco (2014–2024 vintages), Maine (Vexcel).
  - The **Oregon classes are commented out** in `source.py`, despite the README's "supports the whole Oregon state". Treat Portland as unsupported.
  - The NYC source points to `NYC_Orthos_2024`. The ArcGIS org also hosts `NYC_Orthos_2022`; `?f=json` returns HTTP 200 for both.
  - Subclassing `ArcGis` with the 2022 server is a ~5-line change (SYNTHESIS). It matters because NYC Planimetrics were captured from the **March 2022** flight.
- **Training data and training code are not released (FACT):**
  - Issue #26 (2023): the maintainer said the data would be released "as soon as…"; there is no later release note in the README.
  - Issue #85 (2025): "This public release does not include training codes and training recipes."
  - Consequences: (a) no in-distribution test set to reproduce the paper's IoU table; (b) fine-tuning or retraining is not a realistic scope item.
- **Training geography (AUTHOR CLAIM, FULL TEXT + README):**
  - The 2023 paper trained on Cambridge 2018, DC 2019, and NYC park footpaths + roads (Manhattan and Brooklyn, 2018).
  - Boston was held out.
  - The README says the current model "is now trained on more data, **including Manhattan**".
  - **Queens, Bronx and Staten Island are the safest held-out NYC areas. This is unverified; ask the instructor (§13).**

### 2.2 Ground truth (FACT via Socrata API)

- **NYC Planimetric Database: Sidewalk** (`52n9-sdep`, dataset; updated 2025-12-10):
  - 50,865 polygon features; SODA GeoJSON export returns data.
  - Captured from the March 2022 NYC flyover at 0.5 ft GSD, to ASPRS Class 1 horizontal accuracy (per `Capture_Rules.md`).
  - **Roadbed** (`i36f-5ih7`) and **Curbs** (`5xvt-8cbk`) are also available.
  - **There is no crosswalk feature class**: the 22 feature classes listed in `Capture_Rules.md` contain no crosswalk. Crosswalk GT in NYC would need OSM `footway=crossing`, which is noisier.
- **NYC "Sidewalk Centerline"** (`a9xv-vek9`): a map-visualisation view (not a standalone dataset), rows last updated 2022-03-08, published 2016.
  - It could serve as an external network GT, but the parent table and its vintage need checking. Unverified.
- **NYC Land Cover 2017, 6-in** (`he6d-2qns`): 8 classes including tree canopy; 1.33 GB zip.
  - Usable as a canopy-occlusion stratifier. Alternatively, compute an Excess-Green/brightness proxy from the imagery at zero download cost.
- **PathwayBench** (`TaskarCenterAtUW/PathwayBench`, FACT):
  - Code MIT, data ODbL. Last push 2026-08-26.
  - City zips live in **Git LFS** and are downloadable via `media.githubusercontent.com`: DC 550 MB, Seattle 217 MB, Portland 52 MB, metadata 93 MB (HTTP 200 + Content-Length).
  - The README's Google-Drive links return **401 / sign-in redirect** unauthenticated. Use LFS.
  - Contents per sample: Bing aerial PNG, road graph, rasterised street map, **human-validated pedestrian graph (sidewalks + crossings)**, rasterised graph mask.
  - Metric scripts (`tessellate_area.py`, `compute_stats.py`, `summarize_stats.py`) have **stale pins**: Python 3.8, pandas 1.3.4, geopandas 0.12.2, osmnx 1.6.0, geonetworkx 0.5.3, matplotlib 3.4.3.
  - They also **hard-code `PROJ = 'epsg:26910'`** (UTM 10N). That is correct for Seattle/Portland and wrong for DC.
  - These pins conflict with Tile2Net's Python ≥3.11, so a separate conda env is required.

### 2.3 pycalibrate (VIDA-NYU/pycalibrate)

- **Repo status (FACT):** MIT; 17 stars; last push 2022-07-02; PyPI 0.0.3.
- **Python side:** ~300 lines (`calibrate.py`, `callbacks.py`, `helpers.py`, `reliabilitycurve.py`).
- **Frontend:** a React/D3 webpack bundle (`vis/src/...`, `vis/dist/calibration.js`) injected through `notebookjs` (last release 0.1.4, 2021-04-17).
- **The views (FACT, FULL TEXT Fig. 1 + code):**
  - (A) one-line Jupyter use;
  - (B, C) **Calibration View**: a traditional reliability diagram plus a **Learned Reliability Diagram**. The latter is an EBM fit of 1-D `label ~ confidence` over 256 uniform bins, evaluated on a 100-point grid, with a prediction histogram;
  - (D) **Instance View**: a table of instances in a brushed confidence range;
  - (E) **Feature View**: 20-bin histograms of every dataframe column; brushing a range defines a subgroup and draws a new curve;
  - (F) **Performance View**: the confusion matrix of the brushed region.
  - Multiclass is handled one-vs-rest via `selectedclass`.
- **Reusability (SYNTHESIS):**
  - The *algorithms* are directly reusable Python.
  - The *UI* is not reusable for pixels. `Calibrate.__init__` histograms every dataframe column, and the Instance View ships the **entire filtered table** to JS. With 10⁷ pixels it will not work.
  - For a student with weak JS, re-implementing the views in Plotly/Streamlit over a stratified pixel sample (~10⁶ rows) is cheaper than modifying the React bundle.
- **Breakage (FACT from source; confirm by running):**
  - `learned_reliability_diagram` calls `ExplainableBoostingClassifier(random_state=…, binning="uniform", max_bins=bins)`.
  - In current `interpret` (0.7.8), `EBMClassifier.__init__` has **no `binning` argument**, and uniform binning moved to `feature_types=['uniform']`. Expect a `TypeError`: a one-line fix, or pin an old `interpret`.
  - The current EBM also offers `monotone_constraints`. A *monotone* learned reliability curve is a small, defensible design improvement.
- **Stated limitations of Calibrate (AUTHOR CLAIM, FULL TEXT):**
  - it does "not directly tackle the issue of multiclass calibration visualization" (one-vs-rest instead);
  - the learned-curve model class can impose shapes;
  - its evaluation was limited to case studies plus a think-aloud and interviews with 4 practitioners;
  - encodings deserve user studies.
  - Nothing about images or segmentation (tabular COMPAS/Census/synthetic only).

---

## 3. Instructor-lab check

- **Method:** OpenAlex author IDs, then `works?filter=author.id:…,from_publication_date:2024-01-01`, run for:
  - Miranda (A5083220583, 30 works), Silva (A5003584200, 36), Nonato (A5011424640, 40), Xenopoulos (A5036139759, 8), Rulff (A5031297466, 13);
  - plus Hosseini (A5057515797, 12) and Sevtsuk (A5055980218, 22) since mid-2023.
- **FACT: none** of these ~160 titles concerns segmentation calibration, uncertainty in Tile2Net outputs, or VA of AI-generated map errors.
- Adjacent lab work (FACT, titles and venues):
  - Curio (TVCG 2024), StreetWeave (TVCG 2025), VA-Blueprint (TVCG 2025), Urbanite (TVCG 2025), *Occlusion-Free Conformal Lensing* (TVCG 2026: "conformal" here means conformal *maps* for lenses, not conformal prediction; SYNTHESIS from title), Autark (arXiv 2026), UrbanClipAtlas (CGF 2026), StreetTransformer (2025);
  - "Let's Go SideSeeing" (SSRN 2026, sidewalk assessment from street-level multimodal data);
  - "Generating Urban Forest Datasets from Satellite Imagery" (2025);
  - Sevtsuk's NYC Walks / Madina pedestrian-flow models.
- **Urbanite** (Moreira, Ferreira, Veiga, Hosseini, Miranda; VIS 2025 per the ieeevis.org 2025 paper list; FULL TEXT keyword pass):
  - An LLM-assisted dataflow framework for aligning intent, process and outcome in urban VA.
  - "Segmentation" and "Tile2Net" appear **0 times**.
  - Its only "uncertainty" is contributor disagreement in Project Sidewalk crowdsourced labels (Scenario 1).
  - **Overlap ruled out** (SYNTHESIS). It is worth citing as the lab's view of human-AI alignment in urban pipelines.
- **Calibrate → segmentation:** 19 OpenAlex citers of Calibrate (TVCG version) plus 1 citer of the arXiv version. The nearest is **Uni-Evaluator** (Chen et al., VIS 2023): a unified probability-distribution view for classification/detection/**instance** segmentation, with matrix, table and grid views. No reliability diagrams for dense pixels and no topology (ABSTRACT).
- **Verdict:** the lab has **not** done D1. Its own tools (Tile2Net, Calibrate) are the natural reproduction targets, so fit with instructor interest is good.

---

## 4. Paper table

| Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Hosseini, Sevtsuk, Miranda, Cesar, Silva, "Mapping the walk" (Tile2Net) | 2023 | CEUS 101:101950 | Peer-reviewed | Scalable sidewalk/crosswalk polygons + network from orthos | Imagery open; training labels **not released** | HRNet-OCR (mscale) seg → connected components → Douglas-Peucker → Voronoi centerline → heuristics (trim, crosswalk joining, R-tree gap closing) | Static maps | Test IoU (sidewalk 82.67, crosswalk 75.42, mIoU 84.51); polygon match % vs. city GIS (Boston 98.72% count / 77.9% area); centerline match % | Good pixel accuracy; footpaths are weak under canopy | Centerline fitting "not optimized… resulting in discontinuities between polygons"; "future work could… fill in missing gaps… using probabilistic techniques", but warns automated gap filling risks "creating pedestrian segments where they are not visible and hence may not exist" | Yes (BSD-3) | doi.org/10.1016/j.compenvurbsys.2023.101950 | FULL TEXT |
| Xenopoulos, Rulff, Nonato, Barr, Silva, "Calibrate" | 2022 (VIS) | TVCG | Peer-reviewed | Interactive, subgroup-aware calibration analysis | Tabular (COMPAS, Census, synthetic) | Learned reliability diagram (EBM); brushing | Linked reliability, feature, instance and confusion views in Jupyter | Case studies + interviews/think-aloud (n=4) | Binning changes conclusions; subgroup miscalibration visible | One-vs-rest only; learned-curve shape bias; no user study | Yes (MIT), dormant since 2022 | arxiv.org/abs/2207.13770 | FULL TEXT |
| Zhang, Howe, Mehta, Bolten, Caspi, "PathwayBench" | 2024 | arXiv 2407.16875 | Preprint ("under review"); no peer-reviewed venue found | Routability-centred benchmark for pedestrian graph extraction | Yes: ODbL, 3 US cities via LFS; 5 non-US cities "released soon" | Intersection-scale Voronoi tessellation (TIPs); avg CC, avg BC, TraversabilitySimilarity | Static | Tile2Net vs Pedestrianfer vs Prophet on DC, Portland, Seattle | **Tile2Net "overpredict[s] disconnected edges"**: avg CC 5.01 vs GT 1.65 (Seattle); TraversabilitySimilarity 0.04 (Portland) and 0.17 (Seattle) despite edge-F1 0.76–0.90 | Only 3 cities evaluated; no uncertainty analysis | Yes (MIT), stale pins | arxiv.org/abs/2407.16875 | FULL TEXT |
| Zhang, Howe, Caspi, "Reliable, Routable, and Reproducible" (Prophet + Skeptic) | 2024 | arXiv 2410.19762 (TRB-style; venue unverified) | Preprint | Statewide pedestrian-path collection with human QA | Partly | Graph hypothesis from roads + segmentation probabilities; SPSA node optimisation; prune corners with mean prob < threshold | Skeptic task-management review tool | Routability vs. SOTA; reduced vetting time | **Assigns each edge a "confidence" = mean pixel probability in a buffer**, for routing to avoid low-confidence edges | Confidence never validated for calibration or error prediction (none found in text) | Partly | arxiv.org/abs/2410.19762 | FULL TEXT (skim) |
| Gupta, Zhang, Hu, Prasanna, Chen, "Topology-Aware Uncertainty for Image Segmentation" | 2023 | NeurIPS | Peer-reviewed | Uncertainty per *structure* (branch/connection), not per pixel | DRIVE, ROSE, **ROADS** (aerial), PARSE | Discrete Morse theory; joint structure model; Probabilistic DMT (perturb-and-walk) | Structure-wise uncertainty maps; reliability diagrams | ECE, Dice, clDice, ARI, VOI, Betti; simulated proofreading clicks | Pixel-wise maps "highlight pixels along the boundary of all structures", which is not useful for proofreading; structure-wise uncertainty is better | "Not applicable in a setting beyond curvilinear segmentation" (sidewalks are areal polygons) | Yes (no license) | arxiv.org/abs/2306.05671 | FULL TEXT |
| Wang, Gong, Wang, "On Calibrating Semantic Segmentation Models" | 2023 | CVPR | Peer-reviewed | Systematic study of segmentation calibration | Cityscapes, ADE, etc. | Selective scaling | Reliability diagrams | ECE in-domain + shift | Capacity, crop size, **multi-scale testing** and mispredictions drive miscalibration | Natural images only | Not found | arxiv.org/abs/2212.12053 | ABSTRACT |
| Kirscher et al., "Rethinking Post-Hoc Calibration in Semantic Segmentation" | 2026 | TMLR | Peer-reviewed | Translation-invariant, decision-preserving post-hoc calibrators | Natural + medical | TI calibrators; class-conditional affine that preserves argmax | – | Calibration under corruption shift | TI variants improve calibration; decision-preserving variants avoid degrading segmentation | Not aerial | unverified | arxiv.org/abs/2607.01902 | ABSTRACT |
| Barfoot et al., "Average Calibration Error… / dataset reliability histograms" | 2024 | MICCAI (+ TMI submission 2506.03942) | Peer-reviewed / preprint | Pixel-wise calibration loss; dataset-level reliability visualisation | Medical | mL1-ACE | **Dataset reliability histograms** (static aggregation of per-image reliability diagrams) | ACE/MCE/Dice | −45% ACE on BraTS | Static, medical; no linking to downstream structure | Yes | arxiv.org/abs/2403.06759 | ABSTRACT |
| Karani, Dey, Golland, "Boundary-weighted logit consistency" | 2023 | MICCAI | Peer-reviewed | Calibration at ambiguous (boundary) pixels | Medical | Consistency regularizer | – | ECE | Miscalibration concentrates at label-ambiguous boundaries | Medical | – | arxiv.org/abs/2307.08163 | ABSTRACT |
| Moreira et al., "Urbanite" | 2025 | TVCG (VIS 2025) | Peer-reviewed | Human-AI alignment in LLM-built urban dataflows | OSM, Project Sidewalk | LLM dataflow + provenance | UTK/Vega-Lite | Usage scenarios | – | – | Yes (Curio family) | arxiv.org/abs/2508.07390 | FULL TEXT (keyword pass) |
| SegNetVis (student repo, George G. S.) | 2025-12 | Course project (DS-GA 3001, Fall 2025; inferred from dates and README phrasing) | Not a paper | Pixel ↔ network debugging tool for Tile2Net | Manhattan (in training set) | JS deck.gl; graph algorithms | Dual linked panels, lens, confidence filter | Demo only | "85.2% mIoU vs 40.6% connectivity" | No calibration; no held-out split; synthetic fallback data | Yes (no license) | github.com/george-gideon-S/SegNetVis | README + code grep |
| He et al., "Where can we help?" (VASS) | 2022 | TVCG (VIS 2021) | Peer-reviewed | VA diagnosis of segmentation for movable objects (driving) | Driving datasets | Context-aware spatial representation learning | Linked views | Case studies | – | No calibration or topology | unverified | doi.org/10.1109/tvcg.2021.3114855 | SECONDHAND |
| Chen et al., "Uni-Evaluator" | 2023 | TVCG (VIS 2023) | Peer-reviewed | Unified evaluation for classification/detection/instance segmentation | COCO-like | Probability-distribution formulation | Matrix, table, grid | 2 case studies | – | No dense-pixel calibration, no topology | Yes | arxiv.org/abs/2308.05168 | ABSTRACT |
| SEG-RobustEye | 2025 | IEEE VIS 2025 | Peer-reviewed (short?) | Robustness of medical segmentation under input transforms | Kvasir-SEG | Input optimisation | Instance summary, Dice distributions | – | – | Robustness, not calibration | unverified | pure.tue.nl (PDF) | SECONDHAND |
| Han, Wolfe, Howe, "TraversRL" | 2026 | ECCV 2026 | Peer-reviewed (accepted) | Grow a connected pathway graph with RL | Intersection datasets | RL over direction-distance actions | – | Buffered IoU, connectivity | Segmentation baselines "often produce disconnected or fragmentary graphs"; TraversRL >2× connectivity | – | unverified | arxiv.org/abs/2607.17479 | ABSTRACT |
| Mossina, Dalmau, Andéol, "Conformal Semantic Image Segmentation" | 2024 | CVPR-W | Peer-reviewed | Conformal sets for segmentation | Cityscapes, ADE20K, LoveDA | Split conformal | Static heatmaps | Coverage | – | Not VA; no topology | Yes (MIT, last push 2024-12) | github.com/deel-ai-papers/conformal-segmentation | SECONDHAND (scan 4) |

---

## 5. What the closest papers leave open (their own words, then my synthesis)

- **Tile2Net (FULL TEXT):**
  - The authors blame network discontinuities on the **vectorizer**: Voronoi-skeleton sensitivity to the interpolation distance, and skeletons fitted per polygon so that endpoints do not join. They propose "probabilistic techniques" for gap filling as future work, but caution against hallucinated links.
  - They never mention a probability or confidence map as an output.
  - SYNTHESIS: two testable questions come straight from the authors' own text:
    - how much of the topology error is vectorizer-caused vs. segmentation-caused (§6 RQ2);
    - can probability information separate real missing links from real gaps (RQ3)? This is exactly the triage that would make their proposed gap-filling safe.
- **PathwayBench (FULL TEXT):** quantifies Tile2Net's fragmentation (avg CC ≈3× GT; TraversabilitySimilarity 0.04–0.17), but gives only method-level tables. It has **no localisation or explanation of *why* each TIP fails**. SYNTHESIS: a per-TIP, error-cause map is a natural VA contribution on top of their metrics.
- **Prophet (FULL TEXT skim):** already turns pixel probabilities into edge confidences and threshold-prunes corner hypotheses, but **never checks whether those confidences are calibrated or predictive**. SYNTHESIS:
  - "use pixel probability to score edges" is *not* novel;
  - "*validate* whether it works for a segmentation-first pipeline like Tile2Net, and where it fails" is.
- **Gupta et al. (FULL TEXT):** argue that pixel-wise uncertainty is the wrong unit for topological proofreading. That is **a direct prior that H3 might be false** in its naive form (mean pixel confidence), which is why RQ3 uses a *path-bottleneck* (maximin) confidence, a structure-level quantity. Their method is restricted to curvilinear masks, and sidewalks are areal, so there is no direct scoop.
- **Calibrate (FULL TEXT):** tabular, one-vs-rest, no spatial stratifiers. SYNTHESIS: extending it to per-pixel segmentation raises new design needs:
  - (a) spatial/structural stratifiers: boundary distance, canopy, tile-edge distance, borough;
  - (b) linking a brushed reliability region to a *map*, not a table;
  - (c) cluster-aware uncertainty bands, since pixels are not i.i.d.
- **Segmentation-calibration ML papers (Wang 2023; Kirscher 2026; Barfoot 2024; Karani 2023, ABSTRACTS):** show the method side is active. Multi-scale testing (which Tile2Net uses) and boundary pixels are known miscalibration drivers. All of them evaluate static ECE on natural or medical benchmarks; **none looks at aerial pedestrian infrastructure or downstream graph quality**. SYNTHESIS: do not propose a new calibrator. Use temperature scaling (and optionally a decision-preserving variant) as a control.

---

## 6. Refined research questions and hypotheses

The original RQ's last clause ("Can a tool let users trace…?") is a usability claim. It is untestable in 40 h without a user study, and it has already been demoed by SegNetVis. I replace it with three measurable questions and keep the tool as the instrument for answering them and the vehicle for case studies.

- **RQ1 (calibration, Track A core):**
  - How well calibrated are Tile2Net's per-pixel class probabilities on held-out NYC imagery?
  - Where is miscalibration concentrated: class, distance to GT boundary, canopy/shadow, distance to stitched-tile edge, AOI?
  - Does temperature scaling fit on spatially disjoint blocks remove it?
- **RQ2 (error attribution, A×B hybrid):**
  - Of the topology errors in Tile2Net's final network (missing edges, false disconnections), what fraction appears when the *same* polygon→network code runs on the **GT mask** (vectorizer-intrinsic) vs. only on the predicted mask (segmentation-induced)?
- **RQ3 (gap triage):**
  - Among *candidate gaps* in the predicted network (pairs of dead-ends, or dead-end to edge, within 15 m), does a probability-map feature separate *true missing links* (GT connected there) from *true gaps* (GT not connected) better than geometry-only baselines?
  - The feature is the **bottleneck confidence**: the maximin of p(sidewalk) or p(sidewalk)+p(crosswalk) along the best pixel path between the two endpoints.
  - Terms:
    - **Maximin path** (also "widest path"): the path whose weakest pixel is as strong as possible.
    - **Bottleneck value**: that path's weakest pixel value. It equals the highest threshold at which the two endpoints are still in the same connected component of the thresholded probability map.
  - Worth reading up on: 0-dimensional superlevel-set persistence. It is the same quantity, and it ties into the Nov 10 TDA lecture.

**Hypotheses, each with a pre-registered falsification criterion:**

- **H1:** Tile2Net is **over-confident** overall (ECE ≥ 3% on held-out tiles). The over-confidence is significantly larger within 1 m of GT boundaries and under canopy than in open, interior pixels.
  - Falsified if the 95% cluster-bootstrap CI of the ECE difference (boundary minus interior) includes 0.
  - Motivation: multi-scale testing and RMI-style losses are known miscalibration factors (AUTHOR CLAIM, Wang 2023). Tile2Net uses mscale; the RMI loss is present in its code. SPECULATION: Tile2Net may therefore be under- or over-confident; the direction is unknown.
- **H2:** A **majority (>50%)** of predicted-network topology errors in the held-out AOI are *segmentation-induced* rather than vectorizer-intrinsic.
  - Falsified, and interesting, if the oracle-mask network already reproduces ≥50% of the errors. That result would say the Tile2Net authors' own diagnosis ("vectorizer discontinuities") dominates, and that confidence VA is the wrong lens for most errors.
- **H3:** Bottleneck confidence achieves **AUROC ≥ 0.75** for true-missing-link vs. true-gap. It beats both (a) gap Euclidean length alone and (b) mean confidence in a buffer (Prophet-style) by ≥0.05 AUROC.
  - Falsified if its CI overlaps baseline (a).
- **H4 (calibration matters for the tool):** after temperature scaling, a fixed threshold τ=0.5 on bottleneck confidence gives a precision/recall trade-off closer to its nominal meaning (reliability curve of the gap classifier closer to the diagonal).
  - Secondary; drop it if time is short.
  - Note: temperature scaling does not change the argmax, so the network itself is unchanged. What changes is what "0.8 confident" means in the tool.

---

## 7. Project design

### 7.1 Data (exact subset)

- **Main AOI (MVP):**
  - One **Queens** block, ≈1.5 km² (≈450 zoom-19 tiles, ≈29 M px). Pick it to include both dense-grid blocks and tree-lined blocks, e.g., parts of Jackson Heights/Elmhurst plus a Forest Hills strip.
  - The exact bounding box is to be chosen in week 1, after confirming training coverage (§13).
  - Imagery: **NYC_Orthos_2022** (same flight as Planimetrics). Change `NewYorkCity.server` via a subclass.
  - Pixel GT: rasterise Planimetrics **Sidewalk** → sidewalk; **Roadbed** → road; else → background.
  - Crosswalk handling: Tile2Net predicts crosswalks *on* roadbed, but there is no crosswalk GT. Evaluate (i) binary sidewalk-vs-rest calibration as the primary result, since this class drives the network, and (ii) top-label calibration on 3 merged classes {sidewalk, road∪crosswalk, background}.
- **Split:** divide the AOI into ~0.1 km² blocks. Half the blocks are for fitting temperature and choosing thresholds; half are for testing.
  - Why: pixels within a block are strongly autocorrelated. A pixel-level random split would leak.
- **Stratifiers (per pixel):**
  - class;
  - signed distance to GT boundary (via `scipy.ndimage.distance_transform_edt`);
  - canopy fraction in a 3 m window (land-cover raster, or an Excess-Green proxy computed from the imagery);
  - brightness (shadow proxy);
  - distance to stitched-tile edge (a stitching artefact check);
  - block ID.
- **Stronger version:**
  - Add a second held-out AOI in Staten Island or the Bronx (lower density).
  - Add **Seattle via PathwayBench** (Seattle 217 MB LFS) for crossings-inclusive, human-validated network GT, using Tile2Net's King County 2023 source.
  - PathwayBench's Bing imagery differs from King County orthos, so compare graphs geographically, not pixels.
  - Reproduce PathwayBench's Tile2Net Seattle row (edge-F1 0.90, avg CC 5.01, TraversabilitySimilarity 0.17) in a separate Python 3.8 env.

### 7.2 Models

- **Tile2Net `satellite_2021.pth`**, inference only, patched to dump the fp16 4-class softmax.
- **Recalibration controls:**
  - global temperature scaling (1 parameter);
  - optionally class-conditional temperature, a decision-preserving variant per Kirscher 2026 (ABSTRACT).
  - Why these: cheap, argmax-preserving (the network stays identical), and standard enough that nobody can say you invented a calibrator.
- **No retraining.** Training code and data are unreleased.

### 7.3 Pipeline for RQ2/RQ3 (the key engineering, ~10 h)

1. `net_pred` = Tile2Net network from the predicted argmax mask (standard output).
2. `net_oracle` = Tile2Net's *own* `map_features` + `build_pedestrian_network` run on a **GT mask rendered in Tile2Net's colour code**.
   - Both use the same code and parameters, so any difference between them is segmentation-induced by construction.
   - This stage is CPU-only.
   - Tile2Net's `RemoteInference` already reads segmentation PNGs from disk and vectorizes them (FACT, `inference.py`), so the hook exists.
3. **Vectorizer-intrinsic errors:** places where two GT sidewalk polygons touch or lie within 1 m (i.e., should be walkable-connected) but `net_oracle` has no path between them with detour ratio ≤1.5.
4. **Segmentation-induced errors:** edges or connections present in `net_oracle` but missing in `net_pred`.
   - Use a PathwayBench-style 4 m buffer for edge matching, plus local connectivity for node pairs.
5. **Candidate gaps (RQ3):** in `net_pred`, pairs (dead-end, dead-end) or (dead-end, nearest edge) within 15 m.
   - Label = 1 if `net_oracle` connects the corresponding locations within a detour ratio of 1.5, else 0.
6. **Features per candidate:**
   - bottleneck confidence (maximin over the pixel grid via a max-spanning-tree / Dijkstra variant restricted to a corridor);
   - mean and min confidence in a 2 m buffer of the straight segment (Prophet-style baseline);
   - gap length; angle alignment of the two dead-ends;
   - canopy; tile-edge flag.
7. **Evaluation:** AUROC and AUPRC with a block-cluster bootstrap. Plus a logistic model, for effect sizes only.

### 7.4 Visualization, and why each view is scientifically necessary

Implement in Streamlit + Plotly, with `pydeck` or Plotly mapbox for the map. Keep it pure Python; no hand-written JS.

1. **Stratified reliability view (Calibrate B/C, re-implemented).**
   - Traditional + learned (monotone EBM) reliability curves per class, with block-bootstrap bands.
   - Why necessary: ECE is a scalar. Showing *where on the confidence axis* the model over- or under-shoots, and in which stratum, is what answers RQ1. The learned curve avoids the bin-count artefacts Calibrate documents.
2. **Pixel-feature histograms with brushing (Calibrate E).**
   - Brush boundary distance, canopy or tile-edge distance, and a new curve is drawn.
   - Why: subgroup miscalibration is the hypothesis (H1). Brushing is how you *find* strata you did not pre-specify.
3. **Linked map (new; replaces Calibrate's Instance View).**
   - Pixels in the brushed reliability range (e.g., "confidence 0.9–1.0 but wrong") are painted on the aerial tile, with the GT/pred outline.
   - Why: in segmentation, the "instance" is a spatial pattern, and a table of pixels is meaningless. The map is what turns a calibration statistic into a failure *mechanism* (e.g., "confident errors are driveway aprons").
4. **Gap-triage list + network overlay (new, Track B link).**
   - Candidate gaps ranked by bottleneck confidence, coloured by label (true missing link / true gap / vectorizer-caused).
   - Clicking a gap shows the image chip, the probability field and the maximin path.
   - Why: this is the view where RQ2 and RQ3 become *visible case studies*. It is also the concrete answer to the Tile2Net authors' "probabilistic gap filling" worry.
5. **(Optional) Error-attribution summary.** A stacked bar per block: vectorizer-intrinsic vs. segmentation-induced errors, linked to the map.

### 7.5 Evaluation plan

- **Metrics:**
  - RQ1: ECE (15 equal-width bins and 15 equal-mass bins), class-wise ECE, Brier score, NLL; before and after temperature scaling; per stratum.
  - RQ2: counts and rates of each error type per km and per block.
  - RQ3: AUROC/AUPRC; precision at the recall of a Tile2Net-style fixed-distance gap-closing rule.
- **Baselines:**
  - H3: gap length only; mean-buffer confidence; random.
  - H1: interior-pixel stratum as the reference.
  - Also report a trivial constant-confidence model's ECE, as a sanity check.
- **Controls:**
  - A 1–2 px boundary band exclusion, re-run to separate label misregistration from model miscalibration.
  - 2024 vs. 2022 imagery, as a temporal-mismatch control, if time allows.
  - The oracle network (RQ2) is the built-in control for vectorizer effects.
- **Success criteria (course):** H1–H3 each *resolved* (supported or falsified) with CIs, plus 2–3 case studies in the tool. **A falsified H2 or H3 is still a success if reported cleanly.**
- **Failure criterion:** the pipeline cannot produce matched pred/oracle networks by Nov 3. Then fall back to the grade-safe version (§8).

### 7.6 Reproduction component (the course requires reproduce + extend)

- **Reproduce 1:** Tile2Net's polygon-level evaluation (paper Table 4 style: % of GT polygons matched and area-overlap %) on the held-out AOI. The paper reports Manhattan 98.25% (count) / 87.5% (area).
  - An exact match is not expected: the model has been retrained since, and the area and vintage differ.
  - The deliverable is "reproduced methodology, reported deviation, with explanation".
- **Reproduce 2:** Calibrate's learned reliability diagram on its example notebook (after the `interpret` fix), then show that it transfers to pixels.
- **Stronger:** reproduce PathwayBench's Tile2Net Seattle row.

### 7.7 Hour budget (≈40 h)

| Block | Hours |
|---|---|
| Tile2Net install on SLURM or the RTX box; run `examples/example.sh` (Boston Common) | 3 |
| Patch: dump softmax; NYC_Orthos_2022 source subclass | 2 |
| AOI selection, inference, Planimetrics download + rasterisation, stratifier rasters | 4 |
| Reproductions (Tile2Net Table-4-style; Calibrate learned curve + `interpret` fix) | 3 |
| RQ1 calibration analysis + temperature scaling + block bootstrap | 4 |
| RQ2 oracle network + error attribution | 5 |
| RQ3 candidate gaps + bottleneck + AUROC | 4 |
| Streamlit/Plotly linked views (4 views) | 6 |
| Case studies + figures | 2 |
| Writing: proposal 2, update 0.5, final report 4, demo video 0.5 | 7 |
| **Total** | **40** |

### 7.8 Main failure modes and mitigations

1. **Install or GPU friction.**
   - Evidence: Tile2Net's issue history shows stitching failures, CUDA-only inference, and dump-flag confusion (#16, #53, #77, #84).
   - Mitigation: run the Boston example in week 1 on the cluster. Pin a working torch/CUDA combination.
2. **GT–image misregistration and semantic mismatch.**
   - Details: Planimetric sidewalks include planting strips in NYC (AUTHOR CLAIM, Tile2Net §5); crosswalks are absent from the GT.
   - Mitigation: use 2022 imagery; run the boundary-band ablation; treat crosswalks as road in the 3-class analysis; restrict crosswalk-link claims to the Seattle/PathwayBench extension or OSM.
3. **The vectorizer dominates (H2 false).** Report it; make RQ2 the headline; RQ3 becomes a smaller secondary analysis.
4. **RQ3 tautology.** Bottleneck < 0.5 ⇔ the mask is disconnected, roughly by construction.
   - Mitigation: the *label* is GT connectivity, not mask connectivity. The question is whether *near-threshold* gaps are real, which is non-trivial.
   - Also report AUROC restricted to gaps with bottleneck in [0.2, 0.5].
5. **Too few gaps for statistics.**
   - SYNTHESIS: PathwayBench shows avg CC ≈5 per intersection polygon for Tile2Net in Seattle, so hundreds of dead-end pairs per km² are plausible.
   - Mitigation: if n < 150, add the second AOI.
6. **Pixel autocorrelation inflating significance.** Always use the block-cluster bootstrap. Never use pixel-level tests.
7. **Training leakage.** Avoid Manhattan; confirm with the instructor.
8. **Classmate overlap.** Headline the *findings* (H1–H3), not the tool. Cite SegNetVis explicitly as prior course work and explain the delta.
9. **pycalibrate breakage.** Re-implement ~60 lines of Python; do not touch the React bundle.

### 7.9 Publication path (what must be added after the course)

- **Generalisation:** 2–3 cities with different imagery (NYC, Seattle via PathwayBench, Boston as the paper's held-out city).
- **A second segmentation model**, e.g., a SegFormer trained on PathwayBench masks. This separates "Tile2Net-specific" from "segmentation-first pipelines in general".
- **Comparison to structure-wise uncertainty** (Gupta 2023 code, after skeletonising sidewalk masks). This tests whether cheap bottleneck confidence rivals Discrete Morse Theory (DMT, the topological tool Gupta et al. use to extract curvilinear structures).
- **Downstream impact:** repairing the top-k gaps ranked by calibrated bottleneck confidence; measure the change in TraversabilitySimilarity.
- **A small user study** with real mappers (OpenSidewalks / Taskar Center, or Tile2Net's users) on gap-triage speed and precision vs. an unranked list. This is what turns the tool claim into evidence.
- **Venues (deadlines unverified):**
  - IEEE VIS short paper, or the VIS Uncertainty Visualization workshop 2027. FACT: the 2024–2026 accepted lists contain **no** segmentation/ML-map paper, so it would be distinctive there.
  - CVPR EarthVision workshop; ACM SIGSPATIAL UrbanAI/GeoAI workshops.
- SYNTHESIS: the single most publishable element is RQ2+RQ3 as "error attribution + probabilistic gap triage for segmentation→graph pipelines". It directly answers stated future work in both Tile2Net and PathwayBench.

---

## 8. Two options (combined vs. grade-safe)

| | **Option A: grade-safe, track-aligned** | **Option B: combined (recommended if week-2 checkpoint passes)** |
|---|---|---|
| RQ | RQ1 only: "Is Tile2Net calibrated on held-out NYC imagery, and where not?" + qualitative link to network dead-ends | RQ1 + RQ2 + RQ3 (§6) |
| Track | A (Segmentation Detective), with the Calibrate extension as the twist | A×B hybrid |
| Reproduce | Tile2Net Table-4 style + Calibrate learned curve | Same, plus optional PathwayBench Seattle |
| Extension | Per-pixel Calibrate: stratified learned reliability, brushable pixel features, linked map; temperature scaling | + oracle-network attribution + bottleneck gap triage view |
| Hours | ~30 (10 h slack) | ~40 |
| Novelty | Low–moderate: calibration is a stated Track A question; no aerial-sidewalk calibration paper found | Moderate: no paper found combining attribution + probability-based gap triage for this pipeline |
| Main risk | Looks like other Track A projects unless the calibration findings are sharp | RQ2/RQ3 plumbing; H2 might show the vectorizer dominates |

**Recommendation:** nest them. Option A is the Nov 3 milestone. Decide on Option B only if by ~Oct 27 the oracle network (§7.3 step 2) runs end-to-end on 50 tiles. The proposal should state RQ1–RQ3, marking RQ3 as "stretch, gated".

---

## 9. Gap verdict: **partly addressed**

| Component | Status | Evidence |
|---|---|---|
| VA tool linking pixel errors/confidence to network defects for Tile2Net | **Already done (student-level)** | SegNetVis README/code (Dec 2025); course Tracks A/B define it |
| Pixel probability → edge confidence | **Done, unvalidated** | Prophet (2410.19762), FULL TEXT |
| Structure-level uncertainty for roads/vessels | **Done (curvilinear only)** | Gupta et al. NeurIPS 2023, FULL TEXT; "not applicable beyond curvilinear" |
| Graph-level quality of Tile2Net | **Done (aggregate)** | PathwayBench Table 2 |
| Calibration of Tile2Net / aerial sidewalk segmentation | **Open (no hit)** | Tile2Net citers (52), Calibrate citers (20), arXiv queries (§10) |
| Decomposing network errors into segmentation vs. vectorization | **Open (no hit)** | Stated qualitatively by the Tile2Net authors; no quantification found |
| Probability-based triage of true vs. false gaps in segmentation-first pedestrian networks | **Open (no hit)** | Proposed as future work by the Tile2Net authors; Prophet prunes with mean probability but never evaluates this |
| Calibrate extended to dense segmentation with spatial stratifiers and linked map | **Open at VA level; static analogues exist** | Barfoot 2024 dataset reliability histograms (static, medical) |

---

## 10. Queries run (including null results)

- **arXiv API:**
  - `ti:topology AND ti:uncertainty AND ti:segmentation` → only Gupta 2023.
  - `abs:uncertainty AND abs:"road extraction" AND abs:topolog*` → 0.
  - `abs:uncertainty AND abs:connectivity AND abs:"road network" AND abs:segmentation` → only Gupta 2023.
  - `abs:calibration AND abs:"remote sensing" AND abs:segmentation AND abs:"reliability diagram"` → 0.
  - `abs:uncertainty AND abs:sidewalk` → 6 results, none relevant.
  - `abs:uncertainty AND abs:"road extraction"` → 0.
  - `abs:uncertainty AND abs:"road network" AND abs:aerial AND abs:graph` → 0.
  - `abs:conformal AND abs:road AND abs:segmentation` → 0.
  - `ti:calibration AND ti:segmentation` → 30 results: all medical, LiDAR or natural images; none aerial, none topology-linked.
  - `abs:calibration AND abs:"remote sensing" AND ti:segmentation` → 3 open-vocabulary RS papers (2026), not confidence-vs-topology.
- **OpenAlex:**
  - cites:Tile2Net (W4321485756): 52, none on confidence/calibration/VA.
  - cites:Calibrate (W4297459698, W4288804967): 20, none on segmentation calibration.
  - cites:Gupta 2023 (two records): 16, none urban/pedestrian.
  - Lab author sweeps: §3.
  - Free-text searches ("sidewalk segmentation uncertainty confidence", "pedestrian network extraction uncertainty aerial imagery", "calibration semantic segmentation aerial remote sensing expected calibration error", "visual analytics segmentation uncertainty calibration interactive", "pixel uncertainty predicts topological errors road network segmentation", "reliability diagram semantic segmentation boundary distance"): only surveys, plus one SAR road-uncertainty paper (2021, Remote Sensing; not read, relevance unverified).
  - **The OpenAlex free daily budget ran out mid-session**, so some follow-ups were not run.
- **VIS programs:**
  - Uncertainty Vis workshop accepted lists 2024, 2025, 2026: no segmentation or ML-map paper (FACT).
  - ieeevis.org 2025/2026 paper lists grepped for calibrat/segmentation/uncertain/urban: only SEG-RobustEye (medical robustness), Urbanite, and scalar-field uncertainty work.
- **WebSearch:**
  - "Tile2Net confidence OR uncertainty OR calibration…" → no paper.
  - "uncertainty-aware road network graph extraction… edge confidence calibration" → none combining the topics.
  - "visual analytics semantic segmentation calibration reliability diagram…" → Barfoot, VASS, Calibrate; no combination.
- **GitHub:** `tile2net` → 3 prior-cohort student repos (§1).

---

## 11. Novelty confidence (in words)

- **Moderate-low for the tool, moderate for the findings.**
  - I am fairly confident (≈75%, SPECULATION) that no *published* paper measures Tile2Net's calibration or decomposes its topology errors by stage.
  - The search covered Tile2Net's full citation list and the PathwayBench/Howe line.
  - The segmentation-calibration literature is large, though, and my queries could miss an aerial-imagery calibration paper in a remote-sensing journal. That is a known blind spot: OpenAlex free-text is noisy, and I did not sweep IEEE TGRS/JSTARS directly.
- Less confident (≈55%) that the RQ3 framing is unscooped. Road-graph extraction work (RNGDet, SAM-Road, TraversRL) has many connectivity-repair ideas, and some may score candidate links by pixel probability internally, with no "calibration" or "uncertainty" in the title.
- **What would lower confidence most:** a 2025–26 PathwayBench/TraversRL follow-up from the Howe group that evaluates Prophet's edge confidences. They have the data and motivation (SPECULATION).
- The course-internal novelty bar is *lower* than the publication bar, but it is not zero, because SegNetVis already exists.

---

## 12. 12b-style row

| Area | Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|---|
| D1 Tile2Net segmentation reliability (+ Calibrate→pixels) | **High**: it is the instructor's default project and tool; Track A explicitly asks about confidence vs. accuracy; hybrids allowed | **Medium**: RQ2+RQ3 answer the Tile2Net and PathwayBench authors' stated gaps; the tool alone is not novel | **Medium**: CUDA-only install, softmax-dump patch, oracle-network plumbing, stale PathwayBench env | **Low–Medium**: NYC Planimetrics and orthos are open and aligned (2022), but there is no crosswalk GT and the held-out status of boroughs is unverified | **Medium**: 4 linked Plotly/Streamlit views over rasters + a graph; no custom JS needed | **High for RQ1/RQ2** (ECE, counts with CIs); **medium for RQ3** (AUROC, but the label definition needs care) | …you want the lowest-variance grade with the instructor's buy-in, can get GPU time in week 1, and are happy for the headline to be a measured (possibly negative) finding rather than a new tool |

---

## 13. Verify-this-yourself checks

1. **Training coverage.** Ask Prof. Silva or the TA (or open an issue for M. Hosseini/D. Hodczak) which NYC areas the current `satellite_2021.pth` was trained on. Is Queens safely held out? Your whole RQ1 depends on this.
2. **Softmax availability.** Run `examples/example.sh` with `--dump_percent 100`. Check whether any `*_prob.png` appears. My code reading says no. Then add the fp16 softmax dump and confirm that shapes align with stitched tiles.
3. **pycalibrate.** In a fresh env, `pip install pycalibrate`, then run `Example.ipynb` in JupyterLab 4. Confirm (a) the `interpret` `binning=` TypeError and (b) whether `notebookjs` renders at all. If (b) fails, skip the UI entirely.
4. **Oracle network feasibility (the Oct 27 gate).** Render 50 GT tiles in Tile2Net's colour code, feed them through the `RemoteInference`-style vectorization path, and confirm that you get a network. This is the single highest-value spike.
5. **PathwayBench (stronger version only).** Download `portland.zip` (52 MB) from the LFS media URL. Inspect `*_gt_graph.geojson` CRS and attributes. Confirm the Seattle test-area extent lies within Tile2Net's King County 2023 coverage.
6. **Prior cohort.** Skim SegNetVis and Tile2Net-Inspector for 15 min. In the proposal, write one paragraph on "what is new relative to prior course projects". The instructor will likely remember them.

---

## Sources (key URLs)

- Course default project: https://ctsilva.github.io/2026-VisML-CDS/slides/default-project.html
- Tile2Net repo: https://github.com/VIDA-NYU/tile2net ; paper PDF mirror: https://www.evl.uic.edu/documents/ceus_mappingthewalk_miranda.pdf
- pycalibrate: https://github.com/VIDA-NYU/pycalibrate ; Calibrate: https://arxiv.org/abs/2207.13770
- PathwayBench: https://arxiv.org/abs/2407.16875 ; https://github.com/TaskarCenterAtUW/PathwayBench
- Prophet/Skeptic: https://arxiv.org/abs/2410.19762
- Gupta et al. 2023: https://arxiv.org/abs/2306.05671 ; code https://github.com/Saumya-Gupta-26/struct-uncertainty
- Wang et al. 2023: https://arxiv.org/abs/2212.12053 ; Kirscher et al. 2026: https://arxiv.org/abs/2607.01902 ; Barfoot et al.: https://arxiv.org/abs/2403.06759 ; Karani et al.: https://arxiv.org/abs/2307.08163
- Urbanite: https://arxiv.org/abs/2508.07390 ; Uni-Evaluator: https://arxiv.org/abs/2308.05168 ; TraversRL: https://arxiv.org/abs/2607.17479
- VIS Uncertainty workshop lists: https://tusharathawale.github.io/uncertainty-vis-workshop-2026/accepted-papers.html (and the 2025 and 2024 pages)
- NYC Planimetrics Sidewalk: https://data.cityofnewyork.us/d/52n9-sdep ; capture rules: https://github.com/CityOfNewYork/nyc-planimetrics/blob/master/Capture_Rules.md
- Prior-cohort repos: https://github.com/george-gideon-S/SegNetVis , https://github.com/liu020301/Tile2Net-Inspector , https://github.com/Zihangyu600/Shadow-Impact-Analysis-on-Tile2Net-Road-Extraction-Accuracy
