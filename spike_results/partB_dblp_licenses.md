# Part B: dblp overlap sweep + license recheck

Run date 2026-09-29, on the HPC login node, using light curl/python only. Time spent: about 20 minutes wall-clock (13:10 to 13:30 EDT).
Labels: **FACT** means I saw it in a primary source, and the URL is given. **SYNTHESIS** means it is my inference.

Scratch copies of the raw responses (not committed) are in
`/tmp/claude-62321234/-projects-weilab-zhangdjr-Vis4ML-Soccer/1aeab7d9-0c12-4f78-a9e8-11fc6c25b320/scratchpad/dblp/` (`dblp_2023_2026_sparql.json`, `openalex_2023_2026.json`) and `.../scratchpad/lic/` (dataset cards, READMEs, Zenodo JSON, zip previews).

---

## 1. Commands / sources

### dblp access
- **FACT**: `https://dblp.org/search/publ/api` and `/search/author/api` now return an **Anubis "Making sure you're not a bot!" proof-of-work page** for every UA I tried: a Chrome UA, `curl/8.5`, `python-requests`, and a custom UA. The empty UA got HTTP 429. The `dblp.uni-trier.de` mirror also serves Anubis. `dblp.dagstuhl.de` gave no response. WebFetch on `dblp.org/pid/58/1774.html` got ECONNRESET. I did not try to solve the PoW. The blocked responses are saved as `author_*_ANUBIS_BLOCK.html`.
- **Workaround used (FACT)**: the official **dblp SPARQL endpoint `https://sparql.dblp.org/sparql`** is not behind Anubis. Query pattern (Python helper `q.py` in scratchpad):
  ```sparql
  PREFIX dblp: <https://dblp.org/rdf/schema#>
  SELECT ?pub ?title ?year (SAMPLE(?v) AS ?venue) ... WHERE {
    ?pub dblp:authoredBy <https://dblp.org/pid/PID> ; dblp:title ?title ; dblp:yearOfPublication ?year .
    OPTIONAL {?pub dblp:publishedIn ?v} ...
    FILTER(?year >= "2023"^^xsd:gYear) } GROUP BY ?pub ?title ?year
  ```
- **Cross-check (FACT)**: OpenAlex `https://api.openalex.org/works?filter=author.id:<ID>,from_publication_date:2023-01-01`. The keyword grep over it turned up no relevant hits that dblp had missed.
- One WebSearch per topic (D5 and D2 plus advisor names) looked for 2026 preprints not yet indexed. None found.

### Author disambiguation (FACT, from dblp SPARQL `dblp:affiliation`/`dblp:orcid`)
| Target | dblp pid | Evidence it is the right person | Homonyms excluded |
|---|---|---|---|
| Cláudio T. Silva | `s/ClaudioTSilva` (405 records total) | alias "Cláudio Teixeira Silva". Co-authors Nonato/Barr/Miranda | ~40 other "Cl(á)udio … Silva" pids, incl. `91/766-1`, `91/766-2`, `91/766` (Prolog tabling, avionics, VANETs) |
| Luis Gustavo Nonato | `59/17` (167), ORCID 0000-0002-8514-8033 | co-author set | `431/0056` "Luiz Gustavo Nonato" (1 paper, 2026, SVD centrality on hypergraphs). Possibly the same person split off, but irrelevant to D2–D5 |
| Fabio Miranda | `146/8277` (70), NYU → UIC, ORCID 0000-0001-8612-5805, fmiranda.me | affiliation | `262/3104-2` (Fed. Univ. Pará), `-3` (Cardiff), `-5` (NOVA Lisbon/Introsys) |
| Brian Barr | `270/8463` (33) | co-authors Silva/Nonato (Calibrate, Mountaineer, Visagreement) | Brian Barrett, Barry Brian Barrios, etc. **OpenAlex's "Brian Barr" A5071156124 merges in a physician**: the "cell" hits in OpenAlex are stem-cell transplant and cfDNA papers, which are spurious |
| Enrico Bertini | `58/1774` (124) | only one dblp pid | **OpenAlex A5059742070 is a different Enrico Bertini** (Bambino Gesù Children's Hospital, 1286 works). The right one is A5102923390 (Northeastern) |

---

## 2. dblp results (2023–2026)

Counts are unique dblp records (CoRR preprints and their venue versions counted separately). The non-CoRR count is in parentheses.

| Author | Total 2023–26 | 2023 | 2024 | 2025 | 2026 | non-CoRR |
|---|---|---|---|---|---|---|
| Cláudio T. Silva | 66 | 10 | 23 | 20 | 13 | 37 |
| Luis Gustavo Nonato | 33 | 2 | 9 | 15 | 7 | 21 |
| Fabio Miranda (146/8277) | 39 | 10 | 10 | 11 | 8 | 24 |
| Brian Barr (270/8463) | 19 | 2 | 7 | 9 | 1 | 11 |
| Enrico Bertini | 14 | 4 | 3 | 3 | 4 | 10 |

**Caveat (FACT)**: the SPARQL graph is a dblp snapshot, so the very newest records may lag. The latest Silva record is 2026 (e.g. CoRR 2607.25124 UrbanTrace).

### Every keyword hit
Keywords (case-insensitive): cell, microscop, segment, galax, astronom, forecast, time series, time-series, temporal, judge, LLM, language model, calibrat, uncertainty, disagree. Plain `disagree` misses "(Dis)Agreement", so I also added `(dis)agree` and scanned every title by hand. The manual additions are marked †.

| Year | Venue | Title | Author(s) in set | Keyword | Could overlap | Seriousness |
|---|---|---|---|---|---|---|
| 2023 | VR Workshops | Spatiotemporal-Memory-Guided Machine Perception for Augmented Reality | Silva | temporal | — | none |
| 2023 | TVCG | Calibrate: Interactive Analysis of Probabilistic Model Output | Silva, Nonato, Barr | calibrat | D4 (Zoobot calibration view), D3 (weakly) | tangential. A reliability-diagram VA tool for classifiers, not about ground-truth-free disagreement. D4 should cite it as prior art for its calibration panel |
| 2023 | CoRR | The Disagreement Problem in Faithfulness Metrics | Barr | disagree | D4 (attribution-vs-mask agreement), D5/D3 framing | tangential. About XAI faithfulness metrics disagreeing on tabular data, not model-vs-model output disagreement |
| 2023 | TVCG | Multiple Forecast Visualizations (MFVs): Trade-offs in Trust and Performance in Multiple COVID-19 Forecast Visualizations | Bertini | forecast | D2 | tangential. A perception/trust study of showing many forecasts, not TSFM failure diagnosis. Worth citing in D2 related work for multi-model forecast display |
| 2023 | TVCG | A Comparison of Spatiotemporal Visualizations for 3D Urban Analytics | Miranda | temporal | — | none |
| 2024 | CoRR | Your Co-Workers Matter: Evaluating Collaborative Capabilities of Language Models in Blocks World | Silva | language model | D3 | none. LLM agent evaluation, no judge disagreement |
| 2024 | CoRR / 2025 PacificVis | POEM: Interactive Prompt Optimization for Enhancing Multimodal Reasoning of LLMs | Silva | language model | D3 | none/tangential. VA for prompts, not judges |
| 2024 | SIBGRAPI | Space-Time Urban Explorer (crime/patrolling) | Nonato | temporal | — | none |
| 2025 | TVCG (OpenAlex also lists a 2024 preprint) † | **Visagreement: Visualizing and Exploring Explanations (Dis)Agreement** | Silva, Nonato, Barr | (dis)agree † | D5, D3, D4 | **real (conceptual), already known**. It is the lab's own "disagreement-as-signal" vis paper, but it covers *explanation* disagreement on tabular models. Already analysed in repo commit 542befe. D5/D3 must position against it explicitly |
| 2025 | CoRR / 2026 TVCG | TiVy: Time Series Visual Summary for Scalable Visualization | Silva, Nonato | time series | D2 | tangential. Scalable summarization of many series (DTW/sequential patterns). No forecasting models or failure regimes. Plausible reusable component (summarizing thousands of windows), and D2 should cite it |
| 2025 | CoRR / IEEE Access | Time Series Information Visualization – A Review of Approaches and Tools | Nonato | time series | D2 | tangential. A survey, for citing only |
| 2025 | IEEE Data Eng. Bull. | LLMs for Data Discovery and Integration | Silva | language model | D3 | none |
| 2025 | CoRR / 2026 TVCG | BDIViz: … Biomedical Schema Matching with LLM-Powered Validation | Silva | LLM | D3 | tangential. Uses an LLM as validator/checker for schema-match candidates (human-in-loop), not judge-vs-human disagreement decomposition |
| 2025 | CoRR / VR | AdaptiveCoPilot … NeuroAdaptive LLM Cockpit Guidance | Silva | LLM | — | none |
| 2025 | CoRR | DuoZone: … LLM-Guided Mixed-Initiative XR Window Management | Silva | LLM | — | none |
| 2025 | UMFM | StreetTransformer: … Vision Language Models | Silva | language model | — | none |
| 2026 | CoRR | UrbanTrace: LLM-Assisted Discovery … of Spatial Data | Silva | LLM | — | none |
| 2026 | CoRR / TVCG | Occlusion-Free Conformal Lensing for Spatiotemporal Visualization in 3D Urban Analytics | Miranda | temporal | — | none ("conformal" here is geometric, not conformal prediction) |
| 2024 | CoRR / 2024 TVCG † | Mountaineer: Topology-Driven VA for Comparing Local Explanations | Silva, Nonato, Barr | — † | D4 (attributions) | tangential |
| 2024 | CoRR † | Exploring the Relationship Between Feature Attribution Methods and Model Performance | Silva, Nonato | — † | D4 | tangential |
| 2026 | Inf. Syst. † | A visualization-driven decision support system for selecting feature attribution methods | Silva, Nonato | — † | D4 | tangential |
| 2026 | CoRR † | AgentTrails: Towards Trust and Reuse for Agentic Tasks | Silva | — † | D3 | none/tangential (trust in agent traces, not judge disagreement) |

**No title in the 2023–2026 window, for any of the five authors, mentions cells, microscopy, segmentation of biological images, galaxies/astronomy, time-series *foundation models*, or LLM-as-judge (FACT; full title lists are in the scratchpad JSON).** Silva's 2024 "PaleoScan" (fossil scanning) and "LookUp3D" are 3D scanning, not segmentation-QC.

---

## 3. Overlap verdicts

| Candidate | Verdict | Reasoning |
|---|---|---|
| **D5** (cross-segmenter agreement as GT-free per-cell QC) | **No direct overlap. One conceptual neighbour: Visagreement** (SYNTHESIS) | Nothing on microscopy or segmentation in the lab's output. Visagreement shows the lab values "disagreement as a signal". That helps D5 pitch to Silva, and it also means D5 must say clearly what is new: model-*output* agreement on instance masks with no GT, versus explanation agreement on tabular data |
| **D2** (TSFM window-level failure regimes) | **No direct overlap. Tangential: TiVy, TS-vis review (Nonato), MFVs (Bertini)** (SYNTHESIS) | TiVy is the closest. It is scalable summarization of many series, which could be used as a component, but it does no forecasting-model diagnosis. Outside the lab, an incidental find: arXiv 2608.14106 "Forecast Collapse in Time-Series Foundation Models" (Aug 2026, Wan … Huan Liu, FACT: https://arxiv.org/abs/2608.14106). It describes a TSFM failure mode (flat forecasts at low predictability) across 97 benchmark configs. That is tangential to D2 but should be read and cited. Not an advisor paper |
| **D4** (Zoobot vs volunteers; calibration; attributions vs GZ3D) | **No direct overlap. Tangential: Calibrate, Mountaineer, attribution-selection papers, Disagreement Problem in Faithfulness Metrics** (SYNTHESIS) | These are the lab's own XAI/calibration VA line, and D4's calibration and attribution panels would be read against them. That makes them good citations and good for advisor fit, with no scooping risk: no astronomy anywhere |
| **D3** (decomposing LLM-judge disagreement) | **No direct overlap** (SYNTHESIS) | Several LLM-tool papers, but none on judges or human-vs-LLM rating disagreement. Visagreement again is the conceptual neighbour |

**Ranking impact (SYNTHESIS)**: the dblp sweep changes no ranking. No candidate is scooped by the advisor group. Visagreement and Calibrate/Mountaineer slightly raise the advisor-fit case for "disagreement-as-signal" framings (D5, D4).

---

## 4. License table

| Resource | Primary-source terms (quoted) | (a) Course project | (b) Publication + released code/viz tool | Gotchas |
|---|---|---|---|---|
| **LIVECell** images, annotations, models | FACT (https://github.com/sartorius-research/LIVECell README, "LICENSE" section): *"All images, annotations and models associated with LIVECell are published under Attribution-NonCommercial 4.0 International (CC BY-NC 4.0) license. All software source code associated associated with LIVECell are published under the MIT License."* Repo LICENSE file = MIT (Sartorius AG 2021) | Yes (NC ok, attribute) | Yes, for non-commercial academic work. Adaptations such as crops, overlays and derived masks **may** be shared under BY-NC with attribution plus an indication of changes | NC only. **FACT: there is no AWS Open Data Registry entry**: `https://registry.opendata.aws/livecell/` returns 404, and there is no `livecell*.yaml` among the 1247 files of `awslabs/open-data-registry`. The data sits in a plain public bucket, `livecell-dataset.s3.eu-central-1.amazonaws.com` (HEAD images.zip → 200, Last-Modified 2022-01-20), so the license claim rests on the GitHub README only. NEXT_SESSION_TASKS.md calls it "on AWS", which is true of the bucket but not of the registry |
| **NeurIPS22 CellSeg** (Training-labeled, Tuning, Testing incl. Public/"OpenTest" + Hidden, unlabeled) | FACT (Zenodo https://zenodo.org/records/10719375, DOI 10.5281/zenodo.10719375, published 2024-02-27): metadata license `cc-by-nc-nd-4.0`. Description: *"Dataset License: CC-BY-NC-ND"*. Challenge site https://neurips22-cellseg.grand-challenge.org/dataset/: *"Data license: CC BY-NC-ND"*. Citation requested: Ma et al., Nature Methods 21:1103–1113 (2024). This is the only Zenodo dataset record: the others found are CC-BY baseline or micro-SAM checkpoints | Yes (use and display unmodified, NC, attribute) | Code release is fine. **Redistributing modified data is not**: ND forbids sharing "Adapted Material". Hosting crops, overlays or re-labelled GT inside a public viz tool is very likely an adaptation, so it is forbidden. (SYNTHESIS) Model-predicted masks are arguably new outputs rather than adaptations, but overlays and per-cell crops of the images are not. Safest path: the tool downloads from Zenodo at runtime, and only aggregate metrics are published | FACT (Zenodo zip preview of Testing.zip, 2.93 GB): contains `Public/` with 50 `OpenTest_*` images plus `*_label.tiff` labels, and `Hidden/` with 400 `TestHidden_*` images and **no hidden labels** visible. Also WSI + WSI-labels. Tuning.zip (625 MB) has 101 images + 101 labels. The Public-Test is therefore bundled inside the 2.9 GB Testing.zip, not a separate small download |
| **Galaxy Zoo DESI (HF `mwalmsley/gz_desi`)** | FACT (https://huggingface.co/datasets/mwalmsley/gz_desi README; API `cardData.license = cc-by-nc-sa-4.0`, not gated, last modified 2024-08-29): *"**License:** cc-by-nc-sa-4.0. We specifically require **all models trained on these datasets to be released as source code by publication**."* Also: *"For each specific dataset you use, please also cite the original Galaxy Zoo data release paper … and the telescope description paper"* | Yes | Yes, if: (1) NC; (2) **ShareAlike**, meaning any adapted data (e.g. image crops, attribution maps if derived from the images) is released under BY-NC-SA; (3) **any model trained or fine-tuned on it (even a calibration head / linear probe) is released as source code by publication**. Using a pretrained Zoobot for inference only arguably trains nothing (SYNTHESIS) | The code-release clause sits outside the CC license text, so its enforceability is unclear. Treat it as binding community norm (SYNTHESIS). **More permissive alternatives (FACT)**: Zenodo 8360385 "Galaxy Zoo DESI: Detailed Morphology Classifications for 8.7M Galaxies" v1.0.1 is **CC BY 4.0**. It holds Zoobot-predicted vote fractions and GZD-8 volunteer votes (96k galaxies), no images. Zenodo 4573248 GZ DECaLS volunteer + DL measurements is **CC BY 4.0**. **Zoobot code is GPL-3.0** (GitHub API), so a distributed tool that imports or bundles Zoobot inherits GPL obligations. GZ3D/SDSS: the citing page (https://www.sdss4.org/collaboration/citing-sdss/) states citation/acknowledgement requirements only, and I found no explicit license there. Open issue |
| **TIME dataset (HF `Real-TSF/TIME`)** | FACT (https://huggingface.co/datasets/Real-TSF/TIME card): `license: cc-by-nc-4.0`. *"2026-05-25: Update the dataset license to CC BY-NC 4.0 to ensure compliance with all constituent data providers."* | Yes | Yes, NC, attribute. Adaptations are allowed | Constituent terms (FACT, TIME paper App. B table, https://arxiv.org/html/2602.12147 v4): mostly CC BY 4.0, plus **OpenElectricity NEM CC BY-NC 4.0**, **WHO FluNet (Global Influenza) "permits non-commercial, not-for-profit use"**, IMF "Terms of Service" (Global Price), World Bank terms (Port Activity), NBER "Public Use", SG Open Data Licence 1.0, Apache-2.0, MIT, CC0, Public Domain. No ND anywhere |
| **TIME-ProcessedCSV** | FACT (card): `license: apache-2.0` | Yes | SYNTHESIS: treat as BY-NC. It is the same underlying data, and the Apache tag looks stale next to the 2026-05-25 relicensing | License conflict between sibling repos |
| **TIME-Output** (window-level metrics.npz, quantile predictions.npz, tsfeatures) | FACT (https://huggingface.co/datasets/Real-TSF/TIME-Output card, modified 2026-09-28): `license: apache-2.0` | Yes | Yes, permissive. This is D2's main input, so D2's derived artifacts (regime labels, window scores) can be released freely. Any ground-truth series shown alongside come from the BY-NC `TIME` set | Keep NOTICE/attribution. Whether predictions of NC-licensed series "inherit" NC is legally murky, but the provider labels them Apache (SYNTHESIS) |
| **TIME code (github.com/zqiao11/TIME)** | FACT: GitHub API `license: null`. **There is no LICENSE file** in the repo tree (77 paths). README badge reads `[License: MIT](…License-Apache--2.0…)` (label MIT, badge image Apache-2.0, link to Apache). `pyproject.toml`: `license = {text = "MIT"}` | Running it locally is fine | **Ambiguous.** Without a LICENSE file, default copyright applies, and the declared MIT/Apache is inconsistent. Do not vendor or redistribute TIME code; depend on it by pip/git URL, or ask the authors (open an issue) | Paper text itself is CC BY 4.0 on arXiv |

Incidental model-license facts relevant to D5 (FACT, GitHub API/READMEs): micro-SAM **MIT**. CellSAM (vanvalenlab/cellSAM) **Apache-2.0**; its README says the eval dataset "contains cellpose data which is subject to the cellpose license". Cellpose: GitHub API reports **BSD-3-Clause**, while the README badge says "GPL v3". README: *"All Cellpose models are trained on data that is licensed under CC-BY-NC. The Cellpose annotated dataset is also CC-BY-NC."*

**License bottom line (SYNTHESIS)**: every dataset permits a course project. Everything is **non-commercial**. If any part of the work is done for or funded by a commercial employer, NC terms bite, so keep the project strictly academic. For a published open tool:
- D2 (TIME-Output, Apache) is the least encumbered.
- D5 should build its public demo on **LIVECell (BY-NC, adaptations allowed)** rather than NeurIPS22 CellSeg (**ND, no redistribution of overlays or crops**). CellSeg should be used only for internal evaluation, with aggregate numbers reported.
- D4's HF gz_desi adds **SA + a mandatory code-release clause**. That is manageable for an open project, and avoidable by using the CC-BY Zenodo catalogues plus Legacy Survey cutouts.

---

## 5. Open issues

1. **dblp main site is behind Anubis PoW.** Results come from the dblp SPARQL snapshot, cross-checked with OpenAlex. A 2026-Q3 record might be missing. If certainty matters, have the student open https://dblp.org/pid/s/ClaudioTSilva.html (and 59/17, 146/8277, 270/8463, 58/1774) in a browser.
2. `431/0056` "Luiz Gustavo Nonato" is possibly a split profile of Nonato (1 paper, not relevant).
3. **CC BY-NC-ND vs model-predicted masks**: whether publishing predicted masks for CellSeg images (without the images) is "Adapted Material" is untested. Email neurips.cellseg@gmail.com if the D5 tool needs to host CellSeg-derived content.
4. **GZ DESI HF code-release clause**: it is unclear whether it covers post-hoc calibration or a probe trained on gz_desi labels. Assume yes.
5. **GZ3D / SDSS MaNGA** license: I found no explicit license on the SDSS citing page. Check the SDSS DR17 data-access or VAC page before redistributing GZ3D masks.
6. **TIME code has no LICENSE file.** Ask the authors, or avoid redistributing their code.
7. TIME-ProcessedCSV (Apache) vs TIME (BY-NC) is an inconsistent licence on overlapping data.
8. Did not verify per-source licenses of the constituent datasets inside NeurIPS22 CellSeg. The Nature Methods paper lists sources, some possibly with stricter original terms.
9. Cellpose license discrepancy (API BSD-3 vs README badge GPL-3). Check the LICENSE file if bundling Cellpose code.
