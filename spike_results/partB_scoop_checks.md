# Part B: literature scoop checks (D5, D4, D2)

Run on 2026-09-29, 13:10–13:40 EDT, from the HPC login node. I used only curl, WebSearch and WebFetch. **Time spent: about 30 min.**

Labels:
- **FACT** means I saw it in a primary source, and the URL is given.
- **SYNTHESIS** means it is my inference.

A "scoop" is a paper that already answers the candidate's research question.

**Verdict scale:** none / tangential / partial-scoop / full-scoop.

**Bottom line:** I found **no full scoop** for any of the three candidates. There are two new partial overlaps worth citing: RBQE (D5, adjacent domain) and regime-stratified TSFM evaluation (D2). Neither changes the ranking **D5 > D2 > D4**. Details below.

---

## CHECK 1: D5, cross-model agreement as GT-free per-cell QC (correlated errors between generalist segmenters)

### Sources and queries
- **Semantic Scholar metadata and citations:**
  - BISCUIT (DOI 10.12688/f1000research.171889.1) → `citationCount: 0`.
  - MARC (arXiv:2609.13665) → `citationCount: 0`.
- **OpenAlex:** BISCUIT is W4416395959. `cited_by_count 0`, and `filter=cites:W4416395959` gives count 0.
- **Systematic citer sweep (S2):**
  - Cellpose-SAM (DOI 10.1101/2025.04.28.651001): 256 citers.
  - micro-SAM (DOI 10.1038/s41592-024-02580-4): 334 citers.
  - Together that is 547 unique citers.
  - CellSAM (DOI 10.1038/s41592-025-02879-w): 69 citers.
  - I filtered all abstracts with the regex `agree|consensus|disagree|correlated error|error consisten|quality control|without ground truth|failure detect|quality estimat|uncertaint|reference-free|annotation-free|ranking`.
- **arXiv API queries (2025–26 hits):**
  - "cell segmentation" × {agreement, disagreement, consensus, "quality control", "failure detection", "ground truth" free}
  - Cellpose AND SAM AND errors
  - "segmentation quality" AND microscopy
  - "instance segmentation" AND "quality estimation"
  - "model agreement" AND segmentation
  - segmentation AND "inter-model"
  - "correlated errors" / "error consistency" AND segmentation
- **WebSearch (Google-style):** "ground-truth-free quality control cell segmentation 2026", correlated errors between generalist segmenters, per-cell QC consensus on bioRxiv, annotation-free evaluation of segmenters.
- **micro-SAM:**
  - Nature Methods full text (Europe PMC PMC11903314 XML).
  - GitHub repo (`computational-cell-analytics/micro-sam`) tree, commit history and raw files.
  - torch-em loader history (`constantinpape/torch-em`).
- **Cellpose-SAM:** bioRxiv full text was blocked today (HTTP 429 and bot wall on biorxiv.org, the bioRxiv source XML and a reader proxy). I also checked `MouseLand/cellpose/paper/cpsam/*.py` (no split info there).

### (a) Citers of BISCUIT and MARC
- **FACT:** BISCUIT has **0 citers** in both S2 and OpenAlex (https://api.semanticscholar.org/graph/v1/paper/DOI:10.12688/f1000research.171889.1, https://api.openalex.org/works/W4416395959).
- **FACT:** MARC has **0 citers** in S2. It was posted on 2026-09-12 (https://arxiv.org/abs/2609.13665).
- **FACT:** The F1000 page lists only v1 (18 Nov 2025). There are two referee reports, both "Approved with Reservations": Pape (24 Nov 2025) and Bankhead & Nicolás-Sáenz (05 Jan 2026). The authors' response dated 06 Jul 2026 added object-level score differences. Per the page it does **not** address whether the uncorrelated-errors assumption holds for overlapping methods (https://f1000research.com/articles/14-1277).
  - Caveat: this comes from a WebFetch summary. It agrees with the earlier session's full-text reading.
- **Verdict: none.** Nobody has picked up the question yet.

### (b) 2025–26 papers on correlated errors, agreement or GT-free QC

| Paper | What it does | Verdict |
|---|---|---|
| **Gupta & Singla, "Cross-Model Agreement as a Deployment-Time Reliability Signal for Automatic Polyp Segmentation" (RBQE)**, arXiv 2609.10495, 2026-09-09 | **FACT:** Agreement Dice between a primary model and an independently trained "referee" flags failures. Ablates referee independence vs architectural diversity: same-arch referee AUC 0.923, cross-arch SegFormer 0.960. A "prompt-coupled MedSAM referee underperforms despite maximal architectural diversity". Every trainable referee was trained on the **same** Kvasir-SEG data. Colonoscopy only, **image-level, semantic (not instance), no microscopy, no generalist/foundation segmenters** (https://arxiv.org/abs/2609.10495, html v1) | **partial (method precedent, adjacent domain), not a scoop.** SYNTHESIS: this is the closest analogue. It shows the independence/diversity axis matters, but it never varies *shared training data* or *shared pretrained backbone* among generalists, and it has no per-object analysis. Cite it as the medical precedent. It also strengthens D5's framing ("which kind of independence matters, per cell, for foundation segmenters?") |
| **Miao et al. (Hickey lab), "Foundation cell segmentation models performance on live microscopy and spatial-omics data"**, bioRxiv 2026.04.18.719315, PMC13131665 | **FACT:** Instance-level mutual-NN matching and IoU between model pairs, on CODEX only. "The three SAM-based models, Cellpose-SAM, μSAM, and CellSAM, showed the highest degree of agreement with each other. This similarity likely arises from their shared foundation in the SAM architecture." They do **not** test agreement against GT error, and they do not propose agreement as QC (https://pmc.ncbi.nlm.nih.gov/articles/PMC13131665/) | **tangential / motivating** (already in deep_5). It is observational evidence for D5's hypothesis, and D5 would be the GT-based test of it |
| Wang et al., "Evaluating New AI Cell Foundation Models on Challenging Kidney Pathology Cases", arXiv 2510.01287 (SPIE 2026) | **FACT:** Pairwise agreement of *human quality ratings* (Good/Medium/Bad) per patch across CellViT++ variants and Cellpose-SAM. Notes that CellViT++ variants agree most, "reflecting shared instance segmentation framework". It does not relate agreement to error and does not discuss correlated errors (https://arxiv.org/html/2510.01287v1) | tangential (patch-level ratings, histopathology) |
| Talks, …, Kreshuk, "Unsupervised Source-Free Ranking of Biomedical Segmentation Models Under Distribution Shift", arXiv 2503.00450 v4 (Apr 2026) | **FACT:** Ranks models by the *self*-consistency of their predictions under perturbations. It works at model level, for semantic and instance segmentation, and cites Cellpose-SAM and μSAM (https://arxiv.org/abs/2503.00450) | tangential. It does model selection, not per-cell QC, and uses no cross-model agreement. Useful as a GT-free baseline family |
| MARC, arXiv 2609.13665 | **FACT:** Regresses a cross-method consensus map for spatial transcriptomics (Xenium). Its only validation is against the consensus itself (cell-level Spearman 0.79 against consensus), never against GT errors | tangential (it assumes consensus is a valid surrogate). Already in deep_5 |
| "Detecting cell segmentation errors using doublet methods", bioRxiv 10.64898/2026.09.15.751798 | **FACT (S2 abstract):** Transcript-admixture doublet scores used as a segmentation QC signal in spatial transcriptomics. Not image-based and not cross-model | tangential |
| Conformal Prediction Sets for Instance Segmentation (arXiv 2602.10045); RUAC (arXiv 2605.10603); EvanySeg (2409.14874); ConfIC-RCA (2503.04522); CRISP (bioRxiv 2026.04.16.718947) | Single-model uncertainty or learned GT-free quality estimation. None tests cross-model error correlation | tangential |

- **FACT (negative):** Across the 547 Cellpose-SAM/micro-SAM citers and the 69 CellSAM citers, no abstract tests correlated errors or error consistency between generalist segmenters, or validates per-cell cross-model agreement against GT.
- **Verdict for (b): no scoop.** The closest items are RBQE (adjacent domain, image-level) and Miao (observational, no GT link).

### (c) NeurIPS22 CellSeg splits used by micro-SAM and Cellpose-SAM

**micro-SAM**
- **FACT (paper):** The Nature Methods text says only that NeurIPS CellSeg is "part of the training set (evaluated on a separate test split)". It also says: "comparisons on DeepBacs, PlantSeg (Root) and NeurIPS CellSeg are heavily biased in our favor, because our model was trained on the training splits". The paper does not name which split served as validation (PMC11903314 full-text XML).
- **FACT (paper-era evaluation code):** In `finetuning/evaluation/preprocess_datasets.py::for_neurips_cellseg`, the docstring reads: "we infer on the `TuningSet` … for validation: use the true val set used … for testing: use the `TuningSet` and `TestForSam`" (https://github.com/computational-cell-analytics/micro-sam/blob/main/finetuning/evaluation/preprocess_datasets.py).
  - Validation is `_get_image_and_label_paths(..., split="val", val_fraction=0.1)`, a 10% random split of **Training-labeled**.
  - The last commits to this file are from 2024-02 to 2024-04 ("Update plots for the paper").
  - So **for the paper's (v2) results, NeurIPS Tuning was a *test* set, not the validation set.**
- **FACT (torch-em):** Before commit dc7e9733c1 (2024-05-20), NeurIPS `split="val"` meant a random `val_fraction` split of Training-labeled. From that commit on, `"val"` maps to `Tuning.zip` and `"test"` to `Testing/Public` (https://github.com/constantinpape/torch-em/blob/main/torch_em/data/datasets/light_microscopy/neurips_cell_seg.py).
- **FACT (current LM generalist script):** `obtain_lm_datasets.py` (last changed 2025-02-20, #822, "Update LM generalist scripts") builds the validation loader with `get_neurips_cellseg_supervised_dataset(split="val")` (https://github.com/computational-cell-analytics/micro-sam/blob/main/finetuning/generalists/training/light_microscopy/obtain_lm_datasets.py). Its 14 datasets match the `doc/datasets/lm_v3.md` / `lm_v4.md` lists.
- **FACT (default model):** `micro_sam/util.py` sets `_DEFAULT_MODEL = "vit_b_lm"` → `bioimage.io/diplomatic-bug/1.2`. The same file comments "'/1.2/files' corresponds to v4 models".
- **SYNTHESIS (dates inferred, not stated by the authors):**
  - The **v3/v4 LM generalists, including today's default `vit_b_lm`, were very likely trained with NeurIPS *Tuning* as the validation split.** That means checkpoint selection and ReduceLROnPlateau, not gradient updates.
  - The **v2 (paper) model used a 10% split of Training-labeled** as validation, and used Tuning as test.
  - Either way, **Testing/Public was not used in training or validation** by either scheme.
- **Implication for D5 (SYNTHESIS):**
  - Treat NeurIPS **Public-Test (50)** as the clean held-out set.
  - Treat **Tuning (101) as "model-selection-exposed" for micro-SAM v4**. Report it separately, or pin micro-SAM v2 weights if Tuning must be fully clean.

**Cellpose-SAM**
- **Earlier session's FACT (deep_5 §2.4; could not re-verify today because bioRxiv blocked every fetch):** It trained on 616 of 1,000 NeurIPS Training-labeled images. The paper says: "We did not evaluate performance on the validation or test set for this dataset". So **Tuning and Public-Test were not in cpsam training.**
- **FACT:** The `MouseLand/cellpose/paper/cpsam/` scripts contain no NeurIPS split references, so they neither confirm nor contradict this.
- **Action:** Re-verify from the bioRxiv PDF on a non-HPC machine if it matters for a written claim.

### Conclusion for D5
**No scoop.** The core question has not been tested anywhere I could find. That question is whether the "uncorrelated errors" assumption holds between generalist cell segmenters that share backbones or training data, and whether per-cell agreement validly flags GT errors.
- New must-cite: **RBQE (2609.10495)**. It is a medical, image-level precedent showing that referee independence and diversity shape agreement-based failure detection.
- D5's differentiation should be stated explicitly: instance-level, generalist foundation segmenters, and shared-pretraining/shared-data structure (SAM lineage), measured against GT.
- Ranking effect: **none; D5 stays #1**.

---

## CHECK 2: D4, Zoobot vs volunteer disagreement; attributions vs GZ3D masks

### Sources and queries
- arXiv API:
  - au:Walmsley_M, au:Masters_K, au:Spindler_A, au:Geron_T
  - au:Walmsley AND abs:galaxy / abs:Zoobot
  - abs:Zoobot
  - "Galaxy Zoo" × {uncertainty, saliency, "sparse autoencoder", calibration, disagreement}
  - GZ3D / "Galaxy Zoo: 3D"
  - galaxy morphology × {aleatoric, epistemic, "vote fractions" AND uncertainty}
  - galaxy × {Grad-CAM, attribution}
  - All filtered to 2025–26.
- S2 citers of ZooBot:3D (2606.16507), GZ3D (2108.02065), the SAE paper (2510.23749) and GZ Evo (2512.23691).
- WebSearch: "ZooBot:3D"; Zoobot saliency vs GZ3D masks; Zoobot posterior calibration vs vote disagreement / aleatoric-epistemic.
- ZooBot:3D full HTML (WebFetch).

### (a) ZooBot:3D
- **FACT:** Spindler, Walmsley, Masters, Géron, Garland, Simmons, Popp, "Deep Learning Segmentation of Spiral Arms and Bars for 600,000 Galaxies in DESI", arXiv 2606.16507 (2026-06-15), https://arxiv.org/abs/2606.16507.
  - It is a U-net that predicts "for each pixel, the fraction of volunteers that included that pixel in their mask".
  - It computes **no saliency, attributions or Grad-CAM of any classifier**. The only comparison to masks is an expert visual survey (WS23) of GZ3D masks vs machine maps vs SpArcFiRe.
  - It has **no calibration analysis against vote fractions or volunteer disagreement**.
  - It uses GZ-DESI automated vote fractions only for sample selection.
  - Stated future work: a small citizen-science project to label rings.
- **Verdict: tangential.** It supplies a reference and a baseline, not a scoop. SYNTHESIS: scoop risk from this group remains, because they hold both the masks and the classifier.

### (b) Walmsley / Masters / Spindler, 2025–26

| Paper | Relevant content | Verdict |
|---|---|---|
| Wu & Walmsley, "Re-envisioning Euclid Galaxy Morphology … Sparse Autoencoders", arXiv 2510.23749 (NeurIPS ML4PS 2025) | **FACT (abstract):** SAEs on Zoobot and MAE features. SAE features align with GZ labels better than PCA and find features outside the decision tree. It has no calibration, no uncertainty decomposition and no mask comparison | tangential |
| Walmsley et al., "Galaxy Zoo Evo", arXiv 2512.23691 | **FACT:** Dataset paper. It names "learning under uncertainty from crowdsourced labels" as a target topic but does not do it | tangential (it signals intent, which is a scoop risk) |
| Butterworth & Spindler, "The effects of image augmentations when training ML models in astronomy", arXiv 2604.24862 | **FACT (abstract):** Zoobot augmentations × dataset size on GZ DECaLS. It has no calibration or disagreement analysis | none |
| Euclid Q1 cluster morphology (2609.08962); GZ CEERS / JWST / Cosmic Dawn catalogues; AGN–bars–bulges (2603.28208) | Science and catalogue papers | none |
| Non-group, 2026: Zhang et al., VLM teacher for Zoobot (2608.02300); Prakash et al. OOD/calibration on hard 9-class labels (2608.16654); Kendiukhov AION-1 causal audit (2608.23626) | **FACT (abstracts):** None calibrates against vote distributions with a small-N correction, and none compares attributions to GZ3D masks | tangential |

- **FACT (negative):** No 2025–26 paper found does any of these:
  - (i) small-N-corrected calibration of Zoobot posteriors against volunteer votes;
  - (ii) aleatoric vs epistemic decomposition compared with human disagreement;
  - (iii) attributions compared with GZ3D bar/spiral masks.
- **Caveat:** S2 covers only a few 2025–26 GZ3D citers (6), so non-arXiv astro venues are under-searched.

### Conclusion for D4
**No scoop.** Both RQ-A (small-N decomposed calibration) and RQ-B (attributions vs GZ3D masks) remain open. The risk profile is unchanged: the owning group is active, and ZooBot:3D and GZ Evo explicitly advertise these directions. Ranking effect: **none; D4 stays #3**.

---

## CHECK 3: D2, window-level failure analysis of time-series foundation models

### Sources and queries
- **S2 citations:**
  - TIME (arXiv:2602.12147): 19 citers, all abstracts read.
  - Jander et al. (arXiv:2608.24303): **0 citers**.
- **arXiv API (2026):**
  - "time series foundation" × {regime, "window-level", "per-window", "when do", predict AND failure, persistence, "structural break", "meta-features", "instance-level", "performance prediction", stratified, "aggregate metrics", diagnostic, "visual analytics", interpretab, "error analysis"}
  - "foundation models" AND "time series" AND "failure modes"
  - Chronos AND TimesFM AND failure
- **WebSearch:** Jander title; per-window error prediction from context features; TIME pattern-level citers.

### (a) Citers and related 2026 work

| Paper | What it does | Verdict |
|---|---|---|
| **Wang et al., "Do Time Series Foundation Model Benchmarks Hide Regime-Dependent Failures? Evidence from Traffic Speed Forecasting"**, arXiv 2606.18367 (2026-06-16) | **FACT:** Covers Chronos-T5-Base, Chronos-Bolt-Small and Moirai-1.1-R on METR-LA and PEMS-BAY. It adds **regime-stratified, per-forecast-window evaluation** (free-flow / congested / transition). Transition-window MAE is 11 vs 3 mph, and 90% PI coverage drops to as low as 55%. These failures are "invisible in aggregate metrics". It adds a post-hoc bimodal mixture fix (BMA). It does **not** predict failure from context features and does not discuss persistence bias. It cites neither TIME nor Jander (https://arxiv.org/abs/2606.18367, html v2). Its regime labels appear to be defined from the window's observed values, not from context-only features (SYNTHESIS from the "all steps >55 mph" definition) | **partial-scoop** of the "in-vivo regime-switch failure" element, in one domain (traffic). SYNTHESIS: it pre-empts the claim that regime-switch failures show up in real data and are hidden by aggregates. It does not pre-empt the other elements: cross-domain window-level failure prediction from context features on TIME, in-vivo tests of *persistence overestimation*, a leave-dataset-out predictor, or a VA tool. **Must cite**, and D2's novelty statement must be narrowed |
| Wan et al., "Forecast Collapse in Time-Series Foundation Models", arXiv 2608.14106 | **FACT:** Near-flat TSFM forecasts on hourly equity returns, tied to low target predictability. It is cross-series ranking, not window-level regimes | tangential (a failure mode, but a different one) |
| Cakiroglu et al., "The Spectrum Is Not Enough", arXiv 2607.13006 | **FACT:** Spectral indices cannot predict when a TSFM or context helps. It proposes a configuration-level "coverage deficit" diagnostic, validated leave-one-dataset-out | already known; main **risk**, not a scoop (configuration-level, not per window) |
| Pandey et al., GITCO, arXiv 2606.05332 | **FACT:** "Context sensitivity profiles" map series meta-features to the expected accuracy *gain* from context intervention (TimesFM 2.5, GIFT-Eval) | tangential (it is close to "features → performance" but is about gain from context editing) |
| Choi et al., Non-stationarity in TSFM embeddings (2604.16428); Wang, Noise Titration (2603.22219) | **FACT:** Synthetic sweeps show model-specific failure modes and collapse under regime shifts | tangential (synthetic side, like Jander) |
| Dai et al., ORCA (2606.14222); TimeRouter (2606.11625) | Learn residuals / routing from context, for adaptation or selection, not diagnosis | tangential |
| TIME citers, all 19 (Aurora-X, t0, WPBench, MUSE-Bench, Tabby, FlowTSFM, "When Does Retrieval Help", Expert-Guided Forecast Editing, EPF TSFM evaluation, AQ Arena, TS-ICL, TSFMAudit, Toto 2.0, fev-bench, etc.) | **FACT (S2 abstracts):** These are model, benchmark or leaderboard papers. "When Does Retrieval Help" stratifies by window length relative to seasonal period, for retrieval plug-ins on deep forecasters. WPBench adds "structure-aware diagnostics" for wind power | none/tangential |

- **FACT:** Jander et al. has **no citers** yet (S2), so no in-vivo follow-up exists. The paper is at https://arxiv.org/abs/2608.24303, and its data and code are on Zenodo 22082090 and GitHub `MSCA-DN-Digital-Finance/tsfm_causal_analysis`.

### Conclusion for D2
**No full scoop, but there is one partial scoop.** Wang et al. 2606.18367 already shows, in real traffic data, per-window regime-dependent TSFM failures that aggregate metrics hide.

D2 remains open on four elements:
- (i) predicting window-level failure from *context-only* features across TIME's 50 datasets, with leave-dataset-out evaluation;
- (ii) in-vivo tests of Jander's specific synthetic modes (persistence overestimation, regime-switch collapse) across domains;
- (iii) the visual-analytics framing;
- (iv) classical baselines as the failure reference.

Recommended edits:
- Cite 2606.18367 as the single-domain precedent.
- Drop any "first to show that aggregates hide regime failures" claim.
- Lead with prediction from context and the cross-domain link to Jander.

Ranking effect: D2's novelty margin is slightly thinner. It is **still #2**, because D4 carries a larger scoop risk from the data owners.

---

## Overall effect on ranking (D5 > D2 > D4)
**Unchanged.**
1. **D5:** New precedent in medical segmentation (RBQE), but no one has done the generalist-cell-segmenter, per-instance, shared-lineage test. The micro-SAM v4 default very likely used NeurIPS Tuning for validation, so use Public-Test as the clean set.
2. **D2:** One partial scoop (2606.18367, traffic). Narrow the claim to context-feature prediction plus cross-domain in-vivo validation of Jander's modes.
3. **D4:** Nothing new. ZooBot:3D does no attributions and no calibration, and the SAE paper is tangential.
