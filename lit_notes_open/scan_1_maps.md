# Scan 1 (S1): Maps / Urban AI + Visual Analytics

Scan date: 2026-09-28. Budget used: 2 WebSearch calls (of shared ~45); remainder via WebFetch (course site, arXiv, dblp) and Bash/curl (OpenAlex API, GitHub API). No papers/URLs below are fabricated; anything I could not directly confirm is marked "unverified."

---

## (a) Default project: Visual Analytics for AI-Generated Urban Infrastructure Maps (Tile2Net)

### Instructor-lab overlap — **HIGH, and central**
- **FACT**: The default project *is* built on the instructor's own lab's tool. Tile2Net = Hosseini, Sevtsuk, Miranda, Cesar Jr., Silva, *Computers, Environment and Urban Systems* 2023, "Mapping the walk: A scalable computer vision approach for generating sidewalk network datasets from aerial imagery" (DOI 10.1016/j.compenvurbsys.2023.101950; 52 citations per OpenAlex). Code: github.com/VIDA-NYU/tile2net — verified live (HTTP 200), BSD-3-Clause, 228 stars, last push 2026-09-12 (still actively maintained).
- **FACT** (via WebFetch of `ctsilva.github.io/2026-VisML-CDS/slides/default-project.html`): the course itself frames the problem exactly as sub-area (a) does — "design visualization systems to help domain experts understand, debug, and analyze ML pipelines that generate urban infrastructure maps from aerial imagery," built around Tile2Net. It names the failure mode explicitly: **"Errors cascade through pipeline"** across four stages (image ingestion → segmentation → raster-to-polygon → polygon-to-network), and states this makes it "hard to identify failure points." It offers three pre-scoped tracks: **Segmentation Detective** (pixel-level failure diagnosis, linked views of predictions/confidence/error overlays), **Network Quality Inspector** (topological/geometric quality-coded maps, automatic flagging), **Urban Time Traveler** (temporal diff maps across imagery vintages). Provided data: NYC Planimetrics, USGS NAIP, Google Earth Engine imagery; OSM, city GIS portals, NYC Open Data, USGS EarthExplorer as ground truth/comparison. (Course site itself flags this as "tentative"/"being updated," so treat exact task list as a strong signal, not a contract.)
- **FACT** (dblp/OpenAlex, Fabio Miranda, now UIC): the lab has kept publishing urban-VA infrastructure post-Tile2Net, but on **general urban VA authoring**, not specifically on diagnosing AI-generated map/segmentation errors: *Curio: A Dataflow-Based Framework for Collaborative Urban Visual Analytics* (2024/2025), *Urbanite: A Dataflow-Based Framework for Human-AI Interactive Alignment in Urban Visual Analytics* (2025, arXiv 2508.07390), *StreetWeave* (2025, declarative grammar for street-overlaid multivariate viz), *VA-Blueprint* (2025), and a survey *"Assessing the landscape of toolkits, frameworks, and authoring tools for urban visual analytics systems"* (2024).
  - **AUTHOR CLAIM (mine, not exhaustive)**: Urbanite is the closest thing to overlap — it addresses human-AI "alignment" gaps between intent/process/outcome in urban-analytics pipelines built with LLMs — but its abstract (WebFetched from arXiv 2508.07390) is about LLM-driven analytics construction, not about visualizing segmentation/topology errors of a CV pipeline like Tile2Net. I did not find a Miranda/Silva paper that builds an interactive tool specifically for Tile2Net-style error diagnosis. **This should be read in full before finalizing scope** — it's close enough that a student could accidentally duplicate part of it.

### Key 2023–2026 papers
1. Hosseini et al. 2023, *CEUS* — Tile2Net. Peer-reviewed. Code+pretrained weights open (BSD-3). **The reproduction target.**
2. Zhang, Howe, Mehta, Bolten, Caspi, arXiv:2407.16875 (2024) — **PathwayBench**: "Assessing Routability of Pedestrian Pathway Networks Inferred from Multi-City Imagery." Preprint (>2 yrs on arXiv, no confirmed peer-reviewed venue found — flag as possibly workshop/journal-under-review). Introduces the exact thing sub-area (a)/(b) asked about: a benchmark + **routability-centered metric family** (not just pixel IoU) for pedestrian graph extraction, 3,000 km² across 8 cities, manually vetted ground truth. Code/data release location **not verified** (searched GitHub org guesses, all 404; a WebSearch turned up no repo link either — treat as "code status unconfirmed").
3. "Mind the gap: revealing sidewalk networks at scale," 2026, DOI 10.1088/2632-072x/ae35b9 (IOP journal — peer-reviewed venue, exact journal name not fully confirmed from the abstract fetch). Addresses city-scale sidewalk data gaps.
4. "Sidewalk networks: Review and outlook," 2023 — review, 50 citations. Good scoping source, not a reproduction target itself.
5. "Scalable Label-efficient Footpath Network Generation Using Remote Sensing Data and Self-supervised Learning," 2023 — alternative pipeline to Tile2Net.
6. TraversRL: "Traversable Pedestrian Pathway Generation With Reinforcement Learning," 2026, LNCS — RL framing of the same problem.
7. UrbanVGGT: "Scalable Sidewalk Width Estimation from Street View Images," 2026 (arXiv + ISPRS archives) — adjacent CV task (width, not network topology).
8. Curio / Urbanite / StreetWeave / VA-Blueprint (Miranda lab, 2024–2025) — general urban-VA infrastructure; relevant as prior art on VA design patterns for this domain, not on error diagnosis specifically.

### Saturation verdict: **split**
- The CV/remote-sensing side (extracting sidewalk/pathway networks from imagery) is **active, borderline crowded**: 52 works cite the Tile2Net paper (2023–2026), spanning crosswalk detection, curb-ramp detection, wheelability, sidewalk width, tactile paving, disability parking, etc. Almost all of these are CV/photogrammetry papers optimizing extraction, not visualization papers.
- The VA side — an **interactive tool for inspecting/debugging where and why these AI pipelines fail**, especially tying pixel-level segmentation errors to graph-level routability/topology errors — is **thin to open**. I found zero papers matching "interactive/linked-view visual analytics for Tile2Net-style pipeline error diagnosis" in OpenAlex, dblp, or the citing-works list. This matches the course's own framing of the problem as unsolved.

### Open data (verified)
- **Tile2Net** code+weights: github.com/VIDA-NYU/tile2net (BSD-3, verified live).
- **NYC Planimetrics / NYC Open Data**, **USGS NAIP** (public domain), **OSM** (ODbL) — all standard, openly downloadable, named explicitly by the course site as provided ground truth/imagery sources.
- **PathwayBench** ground-truth dataset (3,000 km², 8 cities) — existence confirmed via arXiv abstract; **exact download URL/license not verified** in this scan — check before committing to it as a benchmark.

### Candidate research questions
1. **RQ-A1**: Reproduce Tile2Net on 1–2 new cities (using NAIP + OSM ground truth), then build a linked-view tool (Plotly/Streamlit — matches student's frontend skill level) that fuses **pixel-level segmentation confidence** with **PathwayBench-style routability metrics** to localize where graph-level errors originate in the pipeline (segmentation → raster-to-polygon → polygon-to-network). Reproduction target: Tile2Net (code open). Minimal extension: routability-metric overlay + interactive drill-down from graph error to source pixels. Evaluation: does the tool help a user (self-evaluated case studies) find failure classes faster than raw output inspection? **Falsification check**: searched "Tile2Net visual analytics error" / "sidewalk network extraction error visualization" (OpenAlex, dblp) — no hit. Not fully falsifiable without reading Urbanite in full (see overlap risk above).
2. **RQ-A2 (Network Quality Inspector track, closest to course's own scoping)**: Implement topology/routability quality-coded maps (à la PathwayBench metrics) over Tile2Net outputs for several US cities, and visualize **where** (which street/blocks) failures cluster — this doubles as a bridge into sub-area (c) (spatial disparity). Reproduction target: Tile2Net + PathwayBench metrics (metric code not confirmed open — may need reimplementation, which is itself a legitimate scope item).
3. **RQ-A3 (lowest-risk)**: Straight reproduction of Tile2Net + a simpler "Segmentation Detective" linked-view (predictions/confidence/error overlay, à la course's track 1) as the safe fallback if metric reimplementation (RQ-A1/A2) proves too time-consuming.

### Feasibility in ~40h
- Data: fully open, course-endorsed (NAIP/NYC data + OSM). Low risk.
- Compute: Tile2Net inference is a segmentation model — well within M1 Pro/SLURM capability; no LLM needed for this track. Low risk.
- Frontend burden: **this is the main risk given weak JS/D3.** Course explicitly expects "linked views" — Plotly/Streamlit/Jupyter can realistically deliver this (matches stated skill), but interactive map+graph linked views are more frontend work than the student's other options. Medium risk.
- PathwayBench metric reimplementation (if code isn't found) is a real scope risk — budget time to fall back to simpler pixel/IoU-based error visualization (RQ-A3) if needed.

### Publication path
Plausible: VIS short paper / poster, or a EurVis/workshop paper on ML-pipeline debugging visualization; IEEE VIS "Vis4ML"-style venues have historically taken exactly this kind of applied diagnostic-VA paper. Realistic upside given this is literally the professor's own tool and there's a visible gap.

### Red flags
- **Default project = expected to be popular.** Many classmates will likely attempt Tile2Net-based projects, so differentiation matters (this doesn't block feasibility but affects publication novelty — check what classmates do, not just literature).
- Course site is explicitly marked tentative/"actively updated" — the exact three-track framing could change before Oct 20.
- PathwayBench code/license unconfirmed — verify before RQ-A1/A2 commit.
- Read Urbanite (arXiv 2508.07390) in full before scoping — nearest instructor-lab neighbor.

---

## (b) Road/building extraction + topology-aware error/uncertainty visualization

### Instructor-lab overlap — **none found**
Searched OpenAlex for Silva/Nonato/Miranda + "topology-aware," "APLS/TOPO/clDice," "road network extraction visualization" — no matches. This appears to sit outside VIDA-NYU's recent publication footprint (their urban-VA work is sidewalk/pedestrian-focused per (a), not road/building segmentation metrics). **SYNTHESIS**: this is a genuinely separate literature (remote sensing / medical-imaging-adjacent topology metrics) from the course's own prior work — lower overlap risk than (a), but also less obviously "their gap to fill."

### Key 2023–2026 (and foundational pre-2023) papers
1. **APLS** metric — from the SpaceNet Road Network challenge (Van Etten et al., "SpaceNet Road Detection and Routing Challenge," pre-2023; foundational, widely cited — **FACT** it's the standard road-topology metric, exact citation not re-verified this session).
2. **clDice** — Shit et al., CVPR 2021, "clDice — a Novel Topology-Preserving Loss Function for Tubular Structure Segmentation" (pre-2023 but still the dominant topology-aware *loss*, heavily used in 2023–2026 follow-ups). Metric/loss paper, not a visualization paper.
3. "Directional Connectivity-based Segmentation of Medical Images," 2023 — topology-preserving segmentation, medical domain (transferable idea, different application).
4. "RoadGIE: Towards A Global-Scale Aerial Benchmark for Generalizable Interactive Road Extraction," 2026 — new large benchmark; **worth checking for VA angle** given "interactive" in the title (not yet read in depth this scan — flag for deep dive if pursued).
5. "CP-loss: Connectivity-preserving Loss for Road Curb Detection," 2021 — topology-preserving loss, roads.
6. "RING-Net," 2022 — road extraction network with connectivity focus.
7. "Aerial Road Segmentation in the Presence of Topological Label Noise," 2021 — directly about label/topology error, but a modeling paper not a VA paper.
8. DeepGlobe Road Extraction Challenge (Demir et al., CVPR workshop 2018) — foundational dataset, still used as benchmark 2023–2026 (site verified live, HTTP 200).

### Saturation verdict: **split, same pattern as (a)**
- **Metrics + loss functions for topology-aware segmentation are saturated** — APLS, TOPO, clDice and variants are well-established and repeatedly reused across 2019–2026 papers (10+ found in a single OpenAlex query).
- **Interactive visual analytics of topology errors specifically** — i.e., a tool that *shows a human* where and why a segmentation-to-graph pipeline broke topology, rather than just reporting a scalar metric — is **thin**. None of the ~15 papers surfaced across two targeted OpenAlex queries ("topology-aware error visualization road network extraction segmentation," "APLS TOPO clDice road network extraction metric") is a visualization-focused paper; all are CV/modeling papers. **This is the same open niche as (a), generalized to roads/buildings** — plausibly the same underlying research question, with roads/buildings giving access to bigger, more standardized benchmarks (SpaceNet, DeepGlobe) than sidewalks do.

### Open data (verified)
- **SpaceNet** — registry.opendata.aws/spacenet/ verified live (HTTP 200). Includes road/building footprint labels across multiple cities; standard license is open (Apache-2.0/CC, per AWS Open Data norms — **exact license text not re-verified this session**, check before use).
- **Microsoft Global ML Building Footprints** — github.com/microsoft/GlobalMLBuildingFootprints verified live, 1,962 stars, license field = "Other" (documented elsewhere as ODbL-compatible — verify exact terms before use).
- **DeepGlobe Road Extraction** — deepglobe.org verified live (HTTP 200); license historically CC BY-NC-SA-ish for the challenge data (not re-verified this session).
- **OSM** (ODbL) as ground truth/comparison, as in (a).

### Candidate research questions
1. **RQ-B1**: Take a standard road-extraction model + SpaceNet, compute the standard topology metrics (APLS, TOPO, clDice), and build an interactive VA tool that visualizes **where in the raster→vector→graph pipeline** topology violations arise (dead-ends, disconnected components, false junctions), with drill-down from graph-level error back to the source pixels/tiles. This is structurally the same idea as RQ-A1/A2 but on a bigger, more standard benchmark. Reproduction target: any open SpaceNet road-extraction baseline with public code (e.g., a CRESI/City-scale road extraction baseline — **not independently re-verified this session, check code availability before commit**). Minimal extension: the interactive linked-view diagnosis layer itself (the visualization is the contribution, not a new metric). **Falsification check**: queries above found no matching VA paper — but this is a well-trodden CV area, so a broader/deeper search (Google Scholar, VIS proceedings directly) before committing is advisable; my OpenAlex coverage of VIS-venue short papers specifically is uncertain.
2. **RQ-B2**: Uncertainty visualization for building-footprint extraction (Microsoft footprints vs. a reproduced segmentation model) — visualize per-building confidence/uncertainty and connect it to downstream errors (e.g., double-counted or merged buildings). Lower novelty than B1 (uncertainty maps for segmentation are a well-worn VA topic generally), so treat as fallback only.

### Feasibility in ~40h
- Data: SpaceNet/DeepGlobe/MS footprints are bigger and more standardized than sidewalk data — arguably **easier** to get working end-to-end than (a), because baselines and pretrained models are more widely available.
- Compute: segmentation models here are comparable to Tile2Net in cost — fits M1 Pro/SLURM.
- Frontend: same risk profile as (a) — linked views needed; Streamlit/Plotly should suffice.
- Slightly **less tied to a "reproduce the professor's paper" narrative** than (a) — could be seen as a parallel/independent contribution rather than an extension of Tile2Net, which the course may or may not want (re-read syllabus reproduction requirement).

### Publication path
Similar to (a): VIS short paper / workshop. Possibly stronger novelty story than (a) since it's less overlapping with instructor's own recent work (differentiator, not duplicate).

### Red flags
- Topology metrics themselves are saturated — do **not** propose "a new topology metric," only a visualization/diagnosis layer over existing ones.
- Baseline model code availability for SpaceNet-style road extraction needs re-verification (I did not confirm a specific open baseline repo this session).
- Risk of becoming "generic uncertainty visualization applied to a new domain" if RQ-B2 is chosen instead of B1 — the brief explicitly warns this has weak novelty.

---

## (c) Map/geospatial model fairness or spatial coverage disparity

### Instructor-lab overlap — **none found**
No Silva/Nonato/Miranda hits in OpenAlex for "spatial disparity," "AI map quality fairness," or neighborhood-level coverage bias. **SYNTHESIS**: this looks like genuine white space relative to the instructor's own publication record, though it is adjacent to Miranda's broader "urban computing for climate/environmental justice" interests (found: "Urban Computing for Climate and Environmental Justice: Early Perspectives," 2024 — **AUTHOR CLAIM**: thematically adjacent (environmental justice framing) but not the same question (AI *map-quality* disparity specifically); worth a closer read given it's the same lab).

### Key 2023–2026 papers
1. "Coverage and bias of street view imagery in mapping the urban environment," 2025 — **directly relevant**: found via Tile2Net citation list, studies where/how street-view-based urban mapping under-covers certain areas. Closest existing paper to the sub-area's core question, but about *street-view coverage*, not about *AI model output quality disparity* specifically — a meaningfully different (upstream) question.
2. "On the Opportunities and Challenges of Foundation Models for GeoAI (Vision Paper)," 2024 — discusses bias/fairness in GeoAI generally at a high level; vision paper, not empirical.
3. "GeoAI for Science and the Science of GeoAI," 2024 — same, high-level/positional.
4. Miranda et al., "Urban Computing for Climate and Environmental Justice," 2024 — adjacent theme, same lab, not the same question.
5. No paper found directly measuring **"does Tile2Net (or any comparable AI mapping pipeline) perform systematically worse in lower-income/older/tree-canopy-dense neighborhoods"** — searched OpenAlex twice with different phrasings, no hit.

### Saturation verdict: **thin (possibly too thin — check for a reason)**
Very few directly on-point papers. Two readings are possible: (i) genuine open gap, matching the brief's hoped-for niche; or (ii) the question is under-studied because it's hard to do rigorously (need reliable ground truth *and* socioeconomic covariates *and* enough spatial coverage to get statistical power on disparity claims) — a common reason for thinness that isn't pure novelty. **SPECULATION**: given how directly this maps onto "environmental justice" framing the Miranda lab is already circling (per item 4 above), I'd guess a lab paper on this exact question is more likely to appear in the next 1–2 years than in areas (a)/(b) — meaning a student who picks this now faces real scoop risk within the ~9-week project window, not just abstractly.

### Open data
- Same pipeline outputs as (a)/(b) (Tile2Net or a SpaceNet/DeepGlobe model), joined against **US Census / ACS** (median income, race/ethnicity by tract — openly downloadable, no credential) as the disparity covariate, and **tree-canopy** layers (e.g., USGS/NLCD, open) as a confound. All open; the *combination/join* is the main engineering task, not new data acquisition.

### Candidate research questions
1. **RQ-C1**: Run Tile2Net (or a reproduced road/building model) across several US cities/neighborhoods with openly available imagery, compute per-neighborhood accuracy/topology-error rates against OSM ground truth, and visualize disparity against Census covariates (income, imagery vintage/resolution) as a linked map+scatter VA tool. Reproduction target: Tile2Net (or SpaceNet baseline). Minimal extension: the disparity-analysis + visualization layer. Evaluation: statistical association between error rate and covariates + a VA tool to explore it. **Falsification check**: OpenAlex queries "spatial disparity AI generated map quality," "OSM building footprint extraction fairness bias neighborhood" — no direct hit found.
2. This sub-area is best framed as an **add-on lens on top of RQ-A2 or RQ-B1** (both already produce per-location error maps) rather than a standalone project — doing it standalone risks becoming primarily a socioeconomic-statistics project with a thin VA contribution.

### Feasibility in ~40h
- Data: open, but **integration burden is real** (imagery + model output + OSM ground truth + Census join + confound control) — this is the main risk, not compute.
- Compute: same as (a)/(b).
- Statistical rigor: disparity claims invite reviewer skepticism (confounds: imagery vintage, resolution, tree cover, urban vs. suburban form all correlate with income) — courses grading this could ding it for causal overreach if not carefully hedged as descriptive/exploratory.
- **Best used as an extension bolted onto (a) or (b), not a standalone 40h project.**

### Publication path
Weaker standalone venue fit than (a)/(b) unless paired with a strong VA contribution; could fit a "fairness in ML" workshop framing (matches Nov 24 lecture topic on interpretable ML and fairness) if timed well.

### Red flags
- **Subjective/confounded evaluation** risk is high (the brief's own listed red-flag category applies directly).
- Very thin literature cuts both ways — could mean open gap, or could mean it's a known-hard, hard-to-review question.
- Best treated as a feature of (a)/(b), not its own project, given the 40h budget.

---

## Final triage table

| Sub-area | Best RQ | Novelty evidence (1 line) | Feasibility | Grade-safety | Publication upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) Tile2Net error-diagnosis VA | RQ-A1/A2: linked-view tool fusing segmentation confidence + routability/topology metrics for Tile2Net pipeline errors | 52 citing CV papers, zero VA papers found combining error-diagnosis + interactivity; course site itself frames this as unsolved | Med (frontend burden, PathwayBench metric code unverified) | High (matches course's own explicit default-project scoping) | Med (VIS short paper plausible) | **Yes** | Directly course-endorsed, data/compute low-risk, clearest open question, but must read Urbanite first to avoid near-duplicate |
| (b) Road/building topology-error VA | RQ-B1: interactive topology-error diagnosis tool over SpaceNet/DeepGlobe using APLS/TOPO/clDice | Metrics saturated (10+ papers), zero VA-specific papers found in same search pattern as (a) | Med-High (bigger, more standard benchmarks/baselines than sidewalks) | Med (less obviously "the" default project, but same skill fit) | Med | **Yes** | Same underlying gap as (a) but on more mature/standardized data; good backup or parallel option, less overlap risk with instructor's very recent work |
| (c) Spatial disparity of map quality | RQ-C1: disparity analysis + VA layer on top of (a)/(b) outputs vs. Census covariates | Very thin literature; one close-but-different paper found ("coverage and bias of street view imagery," 2025) | Low-Med (integration + confound burden) | Low-Med (subjective/confounded evaluation risk flagged explicitly in brief) | Low-Med | Maybe, **only as an add-on to (a) or (b)**, not standalone | Real gap but likely thin because it's hard to do rigorously, not just unstudied; scoop risk from Miranda lab's adjacent environmental-justice work |

---

**Labels used**: FACT (verified via WebFetch/OpenAlex/GitHub API this session), AUTHOR CLAIM (my inference from search coverage, not exhaustive), SYNTHESIS (my interpretation combining multiple facts), SPECULATION (explicitly flagged guesses about future risk). Anything not labeled inline in a card is treated as FACT verified via the tool calls in this session.
