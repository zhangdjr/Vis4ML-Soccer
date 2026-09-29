# Scan 6 (S6): Earth observation ML + visual analytics

Date: 2026-09-29. Labels: FACT (verified this session via API/fetch), AUTHOR CLAIM (from an abstract only), SYNTHESIS, SPECULATION.
Tile2Net not re-reviewed (covered in deep_1 / scan_1).
Tools: OpenAlex author sweep for Claudio T. Silva (A5003584200), arXiv abstract fetches, HF and GitHub APIs, about 8 WebSearch calls. Caveat: several 2026 arXiv items were found through a search-API summary and then spot-checked (EarthShift, 2608.16614, 2607.05207, 2605.12678, 2605.08113, 2510.19586, 2606.07780, 2412.05600 abstract pages fetched). Items marked "search-only" were not opened.

## Instructor-lab overlap (whole slice)

- FACT (OpenAlex, Silva 2023-2026): no paper on EO foundation models, embeddings, or XAI for satellite models. Overlapping items in EO or geospatial:
  - Brewer, Valdrighi, Solunke, Rulff, Piadyk, Lv, Poco, Silva, "Granularity at Scale: Estimating Neighborhood Socioeconomic Indicators from High-Resolution Orthographic Imagery and Hybrid Learning," IEEE JSTARS 2024 (doi 10.1109/jstars.2024.3368018, arXiv 2309.16808). This is aerial imagery to socioeconomic indicators, which is close to the geographic-fairness angle.
  - "Visual Analytics for Profiling Land Use Changes," SIBGRAPI 2023 (Silva is an author; title found via OpenAlex, details unverified).
  - "The State of the Art in Visual Analytics for 3D Urban Data," CGF 2024 (Miranda co-author).
  - "StreetTransformer," SIGSPATIAL 2025 workshop on Urban Mobility Foundation Models.
- Lab XAI-VA work (MOUNTAINEER TVCG 2024, T-Explainer, Explainalytics/feature-attribution selection tool) is general-purpose. It is not applied to multispectral EO. SYNTHESIS: the lab knows EO-adjacent aerial imagery and XAI-VA separately, but not their intersection.
- Prior cohorts: `gh search repos` for "DS-GA 3001" and "visualization for machine learning" (created after Aug 2025) found no VisML-course projects. Not proof of absence (repos may be private).
- Also relevant: the 2026-style AlphaEarth and Major-TOM projects on GitHub are generic application repos (crop mapping, irrigation), not VA. Few exist on GitHub, so the "embedding VA" space is thin in code form.

---

## Area card (a): EO foundation models, benchmarks, embeddings as data

**Key papers (2023-2026):**
1. Marsocci et al., PANGAEA: A Global and Inclusive Benchmark for Geospatial Foundation Models, arXiv 2412.04204 (2024); IEEE GRSM 2025 per OpenAlex. Preprint plus magazine. Code: `VMarsocci/pangaea-bench`, GPL-3.0 (FACT). It explicitly says existing benchmarks are biased toward North America and Europe. It reports aggregate metrics per dataset; there is no per-location failure map (AUTHOR CLAIM from abstract; not verified by reading the paper).
2. GEO-Bench-2 (arXiv 2511.15658). 19 permissive datasets, "capability groups" (resolution, bands, temporality). Code `The-AI-Alliance/GEO-Bench-2` returned 404 via gh (the repo name shown in search results; verify). Ranks models by capability group, not by geography or season.
3. Copernicus-Bench, in "Towards a Unified Copernicus Foundation Model for Earth Vision" (Wang et al., ICCV 2025, arXiv 2503.11849). 15 datasets across Sentinel missions (search-only).
4. Corley, Lehmann, ... Kerner, "No One Knows the State of the Art in Geospatial Foundation Models," arXiv 2605.12678 (preprint). FACT from abstract: 152-paper audit, 46 cross-paper disagreements of at least 10 pp for the same model and benchmark, 39% of GFM papers released no weights. Signals reproduction is genuinely hard and valued.
5. Doerksen and Kerner, EarthShift, arXiv 2605.29330 (preprint). FACT: 8 GFMs, 11 tasks, temporal/geographic/sensor shifts, 15-20% OOD drop; code and data at earthshift.github.io; paper CC BY-NC-ND. Covers geography and sensor as aggregate shift types, not fine-grained maps.
6. Lehmann, Gawlikowski, Ekim, Corley, Zhu, "Beyond Accuracy: Assessing Calibration of GFMs and Their Sensitivity to Distribution Shifts," arXiv 2608.16614 (preprint). FACT: 16 frozen encoders, 4 classification and 5 segmentation datasets, GFMs overconfident under shift, CKA analysis. Code link not confirmed.
7. Kolluru et al. (NASA IMPACT), "Land cover and flood type govern the detection limits of satellite-based flood mapping...," arXiv 2606.07780 (preprint). FACT: Prithvi-EO-2.0 on 19 OOD flood events, 23 failure modes, tree cover and built-up areas nearly undetected, some "errors" due to reference-data inconsistency. This is the closest thing to a "where and why does it fail" study, and it is manual/tabular, not an interactive tool. Code/data not confirmed.
8. Mocharla and Patel, "Probing Geospatial SSL Representations with Environmental Signals," arXiv 2607.05207 (preprint). FACT: DINO/MAE/MoCo on SSL4EO, probed against co-located ERA5 variables; ERA5 annotations released.
9. Adjei, "Do Foundation Model Embeddings Improve Cross-Country Crop Yield Generalisation? Leave-One-Country-Out in Sub-Saharan Africa," arXiv 2605.08113 (preprint). FACT: 6,404 maize observations, 5 countries, Prithvi-EO-1.0 embeddings give no advantage; all R^2 negative cross-country; code and data public. A model negative result.
10. Rahman, Barrett, Last, "Characterizing AlphaEarth Embedding Geometry for Agentic Environmental Reasoning," arXiv 2604.18715 (preprint). FACT: 12.1M CONUS samples, effective dimensionality 13.3 of 64, non-Euclidean, vector arithmetic fails.
11. Czerkawski et al., "Global and Dense Embeddings of Earth: Major TOM Floating in the Latent Space," arXiv 2412.05600 (preprint). FACT: released four global embedding datasets, CC BY-SA 4.0.
12. "EarthEmbeddingExplorer: A Web Application for Cross-Modal Retrieval of Global Satellite Images," arXiv 2603.29441, ICLR 2026 ML4RS workshop tutorial track (author names not verified). Open code `OpenGeoScope/EarthEmbeddingExplorer` (MIT, 15 stars, FACT). Retrieval plus similarity map; it is not error analysis.

**Also useful:** 2608.03804 "How Usable Are GFMs? Evaluation of 89 Models" (search-only summary: one-third lack practitioner support), 2608.04792 (biomass with GFMs), 2508.00858 "Deploying GFMs in the Real World: WorldCereal" (search-only).

**Do benchmarks analyze WHERE/WHY models fail?** SYNTHESIS: mostly no. PANGAEA, GEO-Bench-2 and Copernicus-Bench report aggregate rankings, with at most grouping by dataset characteristics. Failure analysis appears in one-off preprints (EarthShift by shift type; 2606.07780 by land cover and flood type; 2605.08113 by country). None provides a reusable, interactive per-location, per-season, per-cloud error explorer that I could find.

**Embedding VA:** FACT: no peer-reviewed VA-venue paper on EO embedding exploration found (queried OpenAlex and WebSearch: "visual analytics embedding exploration satellite foundation model AlphaEarth"). Only application-level notebooks (CARTO, Element 84, a Liverpool building-level UMAP/Kepler.gl tutorial) plus the EarthEmbeddingExplorer retrieval app. ctsilva.github.io/publications was not fetched this time; the OpenAlex sweep is the basis.

**Saturation verdict:** Foundation models and benchmarks: saturated and fast-moving (dozens of GFMs; a 2026 paper argues nobody knows the SOTA). Failure-analysis and VA layer: thin. Released-embedding analysis: active in 2026 (many application preprints), but UMAP-on-map dashboards are trivially reproduced, which limits novelty.

**Open data / models (all FACT via API this session):**
| Asset | License | Notes |
|---|---|---|
| Google Satellite Embedding V1 (AlphaEarth), 64-d, 10 m, annual 2017-2024 | CC BY 4.0 | Earth Engine free for research/education/nonprofit after registration (needs a GEE account, not NDA). Also `gs://alphaearth_foundations`, "provider pays" (requester-pays: a small egress cost may fall on the student's GCP billing). Verified on developers.google.com/earth-engine. |
| Major-TOM Core-AlphaEarth-Embeddings (HF) | CC BY 4.0 | Prototype subset, about 62k tif files, about 6 TB used storage per HF API: too big to download whole; sample. |
| Major-TOM Core-S2L2A-249k (248,719 chips, 384x384, 12 bands, about 352 GB) plus precomputed embeddings: Clay v1.5 (1024-d), SatCLIP (512-d), DINOv2, SigLIP, FarSLIP, OlmoEarth (in 249k variants) | CC BY-SA 4.0 | Clay embeddings parquet about 2.2 GB, SatCLIP about 0.3 GB. Zero-training entry point. Global uniform sample with lat/lon. |
| Major-TOM Core-VIIRS-Nighttime-Light, Core-DEM, Core-LULC (README only, appears empty) | CC BY-SA 4.0 | LULC set appears empty (2 files), so labels would need to be joined from ESA WorldCover (separate, open; not verified this session). |
| Prithvi-EO-2.0 (100M/300M/600M, plus 300M-TL-Sen1Floods11 fine-tune) | Apache-2.0 (HF API for 300M) | `NASA-IMPACT/Prithvi-EO-2.0` repo MIT. |
| TerraMind-1.0-base | Apache-2.0 | `IBM/terramind` Apache-2.0. |
| Clay | Apache-2.0 | `Clay-foundation/model`, active (pushed 2026-05). |
| TorchGeo | MIT | Loaders for EuroSAT, BigEarthNet, etc. |
| EuroSAT | MIT (torchgeo HF mirror; GitHub `phelber/EuroSAT` MIT) | About 2.2 GB in the HF mirror. |
| Sen1Floods11 | License not stated in the README I read (commonly CC BY 4.0, unverified) | About 14 GB, `gs://sen1floods11`. |
| BigEarthNet v2 | HF mirror `earthnets/BigEarthNetV2` exists; license unverified this session | Large. |

**Compute:** Prithvi-300M, Clay-L, and TerraMind-base are ViT-scale; inference and linear probing fit on an 11 GB GPU or M1 (MPS) for chips at 224. Fine-tuning 600M is heavier (SPECULATION). Precomputed Major-TOM embeddings need no GPU at all.

### RQs for (a)

**RQ-A1 (grade-safe): Reproduce a GFM linear-probe result and add an error-by-context breakdown.**
- Reproduction target: PANGAEA or GEO-Bench-2 numbers for one model (e.g., Prithvi-300M or Clay) on one or two datasets, or the Sen1Floods11 Prithvi fine-tune.
- Minimal extension: a Plotly/Streamlit linked view (error map, confusion by biome, season, cloud fraction, sensor).
- Evaluation: does the reproduced number match the paper to within about 1-2 pp; do slice metrics reveal significant subgroup gaps (bootstrap CIs).
- Falsification check (queries run): OpenAlex/WebSearch "visual analytics embedding exploration satellite foundation model"; "geographic disparity per-country error analysis geospatial foundation model." Found EarthShift (aggregate shifts), 2606.07780 (manual flood failure taxonomy), 2605.08113 (per-country). Partly done in papers; the interactive reusable tool is not found.
- Risk: dataset-specific metadata (cloud, season) may need to be derived.

**RQ-A2 (higher upside): Zero-training "latent-space audit" of EO foundation models using released embeddings.**
- Reproduction target: Czerkawski et al. Major-TOM embeddings paper (open embeddings), plus the AlphaEarth-geometry preprint (2604.18715) which found effective dimensionality about 13 and failure of vector arithmetic.
- Minimal extension: build linked views (map + UMAP/Mapper + neighborhoods) over Clay, SatCLIP, DINOv2, and AlphaEarth embeddings on the same 249k Major-TOM chips; quantify cross-model neighborhood agreement (kNN overlap, CKA) and which geographic regions/land covers the models disagree on, relating to probe failures.
- Evaluation: retrieval agreement metrics against ESA WorldCover labels (would need joining), and case studies. Ties to course topics (DR Oct 20, clustering Oct 13, TDA Nov 10).
- Falsification check: arXiv/OpenAlex/WebSearch found EarthEmbeddingExplorer (retrieval only) and the geometry paper (no VA, agentic focus). No paper comparing multiple GFM embedding spaces spatially with a VA tool surfaced. Not conclusive; "Earth Embeddings" (arXiv 2608.03410, search-only title) should be checked before proposing.
- Extra risk: crowded space in 2026; UMAP dashboards weak novelty; need label join.

**Feasibility (about 40 h):** Data: small (parquet, about 3 GB across models). Compute: laptop enough for RQ-A2; RQ-A1 needs one GPU. Front-end: Plotly/Streamlit with pydeck or folium works. Medium-high.

**Publication path:** IEEE VIS short/poster or EarthVis workshop; ICLR ML4RS workshop, NeurIPS/ICML workshops on EO. **Red flags:** field moves quickly; UMAP-of-embeddings already common; GPL-3.0 PANGAEA code requires care if redistributing.

---

## Area card (b): Spatial distribution shift, geographic fairness, error and uncertainty maps

**Instructor-lab overlap:** Brewer et al. JSTARS 2024 (socioeconomic estimation from aerial imagery, NYC-focused per abstract; generalization across cities is the natural next question). Miranda's urban-climate vis (per scan_5). Course default project (Tile2Net "Segmentation Detective") does error overlays for sidewalks, so overlap is with pipeline error maps, not with global EO.

**Key papers:**
1. Koh et al., WILDS, ICML 2021 (peer-reviewed; `p-lambda/wilds` MIT, FACT). Includes FMoW-wilds (region shift, worst-region accuracy) and PovertyMap-wilds (country shift). Old but is the standard reproduction target.
2. EarthShift (2605.29330), see above.
3. Lehmann et al. calibration under shift (2608.16614).
4. Adjei leave-one-country-out crop yield (2605.08113): full public code, small data, good reproduction target.
5. Kolluru et al. flood detection limits (2606.07780).
6. Rey, Mnih, Neumann, Overlan, Purves, "Uncertainty evaluation of segmentation models for Earth observation," arXiv 2510.19586 (preprint). FACT: PASTIS and ForTy, ensembles, Stochastic Segmentation Networks, per-pixel uncertainty as error detectors. Code not confirmed.
7. VibE (arXiv 2503.20112), VA workflow for subgroup-based semantic error analysis of CV models (search-only; general CV, not EO). A method one could port.
8. "Interactive Visualization and Representation Analysis Applied to Glacier Segmentation," arXiv 2112.08184 (2021): older EO segmentation VA tool with linked views of imagery, ground truth, and predictions (search-only). Shows the concept exists; not recent.
9. Ratsakatika and Zotta, 2609.28194 (Sept 2026, search-only): buffered spatial validation narrows GFM-embedding advantage. Confirms spatial-CV leakage is a live topic.
10. Shang et al. 2606.29664 (search-only): agriculture GFM benchmark, all three models degrade under regional shift.

**Saturation verdict:** Distribution-shift benchmarking: active, becoming crowded (EarthShift, GFM shift papers in 2026). Geographic-fairness framing for EO: active but mostly aggregate. Interactive error/uncertainty-map VA: thin (glacier 2021 tool is the only VA hit). Evidence: WebSearch "visual analytics ... satellite ... model failures map linked views" returned VibE (general CV) and one 2021 glacier paper.

**Open data:** WILDS FMoW (about 50-70 GB, SPECULATION on size; license: FMoW is a custom research license, verify) and PovertyMap (DHS surveys; requires registration for the original DHS data but WILDS distributes a processed version, verify terms). Prefer Sen1Floods11 (14 GB, `gs://sen1floods11`, has 11 events across continents with metadata geojson: FACT the repo lists `Sen1Floods11_Metadata.geojson`), EuroSAT (small; Europe only), or Adjei's public crop dataset.

**RQs:**
- **RQ-B1 (grade-safe):** Reproduce Adjei's leave-one-country-out (or Sen1Floods11 leave-one-event-out) evaluation with a released Prithvi/Clay model; add an interactive per-region error/uncertainty map plus embedding-shift view (why does region X fail: covariate shift in embeddings vs label shift?). Evaluation: predict per-region error from embedding-distance-to-train (correlation, held-out regions). Falsification: queries "geographic disparity per-country error analysis GFM" found per-country R^2 only, no embedding-shift diagnosis tool. Medium confidence novel.
- **RQ-B2 (higher upside):** "Diagnose, don't just measure": VA that links OOD region error to embedding shift and to reference-label inconsistency (2606.07780 found part of the "error" was label disagreement). Extra risk: needs multiple reference labels, more data engineering; possible EarthShift authors scoop.

**Feasibility:** Data medium (Sen1Floods11 14 GB plus GCS download); compute: inference on 11 GB GPU; frontend: Plotly maps. Medium. **Publication path:** EarthVis / VIS short; ICLR ML4RS workshop; CVPR EarthVision. **Red flags:** WILDS FMoW/PovertyMap terms and size; label noise; "fairness" framing needs a defensible metric (worst-region accuracy).

---

## Area card (c): XAI for EO validated against spectral/physical ground truth

**Instructor-lab overlap:** MOUNTAINEER, Explainalytics (feature-attribution decision support), Visagreement, T-Explainer are all attribution-comparison work (general; images and tabular). This is direct overlap with the method side; the EO spectral-ground-truth angle is not covered. Also Miranda climate-vis (different).

**Key papers:**
1. Höhl et al., "Opening the Black-Box: A Systematic Review on Explainable AI in Remote Sensing," arXiv 2402.13791 (2024; dblp lists as CoRR preprint). FACT via search: about 1 explainability paper per 70 ML papers; notes missing XAI evaluation that accounts for spectral properties.
2. Klotz, Burgert, Demir, "On the Effectiveness of Methods and Metrics for Explainable AI in Remote Sensing Image Scene Classification," arXiv 2507.05916 (IEEE JSTARS per abstract page). FACT: 5 attribution methods (Occlusion, LIME, GradCAM, LRP, DeepLIFT), 10 metrics, 3 datasets; code promised at `git.tu-berlin.de/rsim/xai4rs` (not verified). Does not evaluate against spectral ground truth (abstract silent). This is the direct competitor/reproduction target.
3. Mamalakis et al., synthetic benchmark for attribution in geoscience (Environmental Data Science 2022) and fidelity paper (AIES 2022); Bommer et al., XAI evaluation in climate science (2023/24); Krell et al. 2025 (from scan_5).
4. Mocharla and Patel (2607.05207): probing SSL representations with ERA5 environmental variables, a physical-signal probe of representations rather than attributions.
5. Rahman et al. (2604.18715): AlphaEarth embedding geometry.

**Saturation verdict:** EO-XAI methods: active. Band-level attribution validated against known spectral indices (e.g., does a vegetation model attend to NIR/red as NDVI does; water models to NIR/SWIR as NDWI/MNDWI): thin as a named benchmark, but a single ML paper could cover it as an evaluation section, and Klotz et al. is 2025 on same theme. WebSearch on "band-wise attribution multispectral ... vs NDVI" returned only generic spectral-index pages, no such paper (weak evidence: search engine, not scholarly index). SYNTHESIS: the scan_5 EuroSAT idea remains open but its ceiling is low: it is "known method, new domain," and the lab has three attribution-comparison VA tools, so a reviewer will ask what is new relative to MOUNTAINEER/Explainalytics.

**Open data:** EuroSAT (MIT, FACT, 2.2 GB in HF mirror; 13 bands, 10 classes, 64x64), Sen1Floods11 (water masks give natural ground truth: MNDWI/NDWI from bands), BigEarthNet v2 (large). Physical ground truth is computable from bands themselves (NDVI, NDWI, NDBI), avoiding external labels.

**RQs:**
- **RQ-C1 (grade-safe):** Reproduce Klotz et al.-style attribution evaluation (code, once released) on EuroSAT; extension: compute band-level attribution mass and compare it to a physics-derived expectation (the index-relevant bands per class). Evaluation: rank correlation between attribution band importance and ablation-based band importance and index-based expectation; Spearman with bootstrap CIs. Falsification: WebSearch above; also Höhl review says spectral-aware XAI evaluation is missing. Not exhaustively checked (OpenAlex search returned only generic reviews).
- **RQ-C2 (higher upside):** Test whether GFM attributions (Prithvi/TerraMind on Sen1Floods11) respect physical priors, and whether shortcut cues (e.g., cloud shadow, terrain) explain the 2606.07780 failure modes. Extra risk: attribution on ViT patch tokens is unreliable, and results are then hard to interpret; heavier engineering.

**Feasibility:** EuroSAT scale: High (M1 laptop). C2: Medium. **Publication path:** workshop (xAI4CV/EarthVision); VIS unlikely without a strong VA contribution. **Red flags:** overlap with the lab's attribution-comparison tools; Klotz code availability uncertain; "physical ground truth" for deep nets is contestable (models need not use the same bands humans use).

---

## Final triage table

| Sub-area | Best RQ | Novelty evidence | Feasibility | Grade-safety | Pub upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) GFM embeddings and benchmarks | A2: cross-model latent-space audit on Major-TOM precomputed embeddings (Clay, SatCLIP, DINOv2, AlphaEarth) with linked map/DR views | Only retrieval app (EarthEmbeddingExplorer) and a non-VA geometry preprint found; "Earth Embeddings" 2608.03410 to check | High (zero training, about GB-scale parquet) | High | Med | maybe | Fits DR/clustering/TDA topics and the student's map interest; risk is UMAP-dashboard novelty and label join |
| (a) GFM benchmark reproduction | A1: reproduce a PANGAEA/GEO-Bench-2 probe and add error-by-context slices | Benchmarks report aggregates; slices exist only as one-off preprints | Med | High | Med | yes | Clean reproduction (repo exists); "no one knows SOTA" paper supports value of reproduction |
| (b) Spatial shift and error maps | B1: leave-one-region-out with embedding-shift diagnosis and error/uncertainty map (Adjei code or Sen1Floods11) | Per-country/per-event numbers exist; diagnosis tool not found; 2021 glacier VA only | Med | High | Med-High | yes | Best mix of open reproduction target, map-centric VA, and a real stated gap (label inconsistency vs true failure in 2606.07780) |
| (c) XAI vs spectral truth | C1: band-level attribution vs index-based expectation on EuroSAT | Klotz 2025 and Höhl review leave spectral ground truth open; but lab has 3 attribution-VA tools | High | High | Low-Med | no (fallback) | Safe but low ceiling and direct instructor overlap; keep as fallback only |
| (c) GFM attribution | C2: do Prithvi/TerraMind attributions respect physical priors | Unverified; ViT attribution unreliable | Low-Med | Low | Med | no | Engineering and validity risk |

## Biggest red flags
1. Field velocity: several 2026 preprints (EarthShift, calibration, "usability of 89 models") are scooping the "evaluate GFMs under shift" idea, so novelty must come from the diagnostic VA layer, not from the finding that models degrade.
2. Several claims here come from abstract-level fetches; PANGAEA/GEO-Bench-2 per-slice reporting was not verified in full text. Verify before writing the proposal.
3. Licenses to double-check: FMoW/PovertyMap (WILDS), Sen1Floods11 (README silent), BigEarthNet v2; GPL-3.0 on PANGAEA code.
