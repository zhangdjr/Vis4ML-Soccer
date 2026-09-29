# Scan 7: Microscopy / cell segmentation ML + visual analytics

Scan date 2026-09-29. Budget used: 8 WebSearch, about 8 WebFetch, plus OpenAlex/arXiv/gh/aws CLI checks. Labels: FACT (I saw it in a fetched page or API), AUTHOR CLAIM, SYNTHESIS, SPECULATION, "unverified".

## 0. Instructor-lab overlap and prior-cohort check (do first)

- FACT: OpenAlex sweep of Claudio T. Silva (A5003584200, works since 2019) with queries `cell`, `microscopy`, `segmentation`, `biomedical`, `image`: **zero microscopy or cell-segmentation hits**. Nearest biomedical items are breast-cancer gene-subset classification (2022/2024), Motion Browser (upper-limb, 2019), BDIViz (biomedical schema matching, 2025), HuBar (fNIRS). Image-segmentation items are urban only (CitySurfaces, Urban Mosaic, sidewalk mapping). PipelineProfiler and Melody are generic ML-VA.
- FACT: A web search for "Claudio Silva" plus microscopy/single-cell found nothing. His NYU page mentions "biotechnology" among interdisciplinary domains, with no specific paper.
- Nonato and Miranda: I did NOT sweep them individually (unverified). Nonato has a long history of projection/DR-reliability work, which matters for sub-area (b). Check his projection-distortion papers before writing the proposal.
- Prior cohort: `gh search repos` for "DS-GA 3001", "visualization machine learning Claudio Silva", "cellpose visualization", "segmentation quality visual analytics microscopy" gave **no VisML student repos**. The "DS-GA 3001" hits are other courses. This is weak evidence, since student repos are often private. SYNTHESIS: the microscopy angle is probably unclaimed by the lab.
- SYNTHESIS: because the lab has no microscopy history, the student's background is a differentiator. The risk is the opposite one: the instructor may not be able to judge domain novelty, so the proposal must state the VA/ML-assessment angle explicitly (course topics: model assessment, DR, clustering).

## 1. Area card (a): Generalist cell segmentation, failure analysis, QC without ground truth, VA for QC

**Instructor-lab overlap:** none found (see section 0).

**Key papers 2022-2026** (verified via OpenAlex/arXiv/fetch unless noted):

1. Stringer & Pachitariu et al., *Cellpose-SAM: superhuman generalization for cellular segmentation*, bioRxiv 2025 (preprint; 225 citations per OpenAlex). AUTHOR CLAIM: beats inter-human agreement, robust to channel order, size, noise, blur. Code: `MouseLand/cellpose`, BSD-3, pushed 2026-06. Weights `mouseland/cellpose-sam` on HF, BSD-3 (FACT).
2. Pachitariu et al., *Cellpose 2.0*, Nat Methods 2022 (peer-reviewed), and *Cellpose3*, bioRxiv 2024 (image restoration). Open.
3. Israel et al. (Van Valen lab), *CellSAM*, Nat Methods 2025 (peer-reviewed; bioRxiv/arXiv 2311.11004). Code `vanvalenlab/cellSAM`, Apache-2.0 (FACT via gh).
4. Archit et al., *Segment Anything for Microscopy* (micro-SAM), Nat Methods 2025. Code MIT, actively maintained (pushed 2026-09-29).
5. Archit & Pape, *Revisiting foundation models for cell instance segmentation*, MIDL 2026 (arXiv 2603.17845). Independent comparison of CellPoseSAM, CellSAM, uSAM, SAM, SAM2, SAM3, plus automatic prompt generation (APG). Code in the micro-sam repo. AUTHOR CLAIM per search snippet: SAM2 fails on some datasets; CellPoseSAM best in aggregate.
6. Ma et al., *The multimodality cell segmentation challenge* (NeurIPS 2022 CellSeg), Nat Methods 2024. MEDIAR (Lee et al., arXiv 2212.03465, code MIT) is the winning-tier open baseline.
7. Das, Roy & Zun, *High-Throughput Low-Cost Segmentation of Brightfield Microscopy Live Cell Images*, arXiv 2508.14106, accepted in a 2026 journal. Search-snippet claim (not verified in the paper itself): Cellpose-SAM generalization fails on unstained brightfield. This is an **out-of-distribution failure anecdote**, not a systematic study.
8. *Microscopy Cell Segmentation: Review and Benchmarking of Task-Specific and Foundation Models*, J. Imaging 2026 (PMC13412841). Compares StarDist, CellSAM, Cellpose-SAM, YOLO-SAM on 5 datasets; mean mAP is 0.768 for Cellpose-SAM against 0.435 for StarDist. It does **describe** failures (brightfield low-contrast, dense bacteria), but aggregate-level, no per-instance error taxonomy. Repo link not stated (unverified).
9. Cosarinsky et al., *In-Context RCA / ConfIC-RCA*, arXiv 2503.04522, TMI (accepted per arXiv page). Segmentation-quality estimation without GT with conformal guarantees. Code open. Validated on 10 **medical** tasks; the paper text says it "can tackle" cell images, but I did not verify results on cells.
10. Stillwagon et al., *DINOCell*, arXiv 2604.10609 (preprint): SEG 0.784 on LIVECell, +10.4% over SAM-based models (AUTHOR CLAIM).
11. Older reference for QC: QANet (arXiv 1904.08503), reference-free segmentation quality network (natural/medical images).

**Where do benchmarks explain WHERE they fail?** SYNTHESIS from the above: they report per-dataset or per-modality aggregate scores plus prose ("dense", "low contrast", "brightfield"). I found no paper that computes **per-image or per-instance failure attributes** (density, size, contrast, focus) and relates them to metric drops, or that lets a user browse failures. This gap seems real but thin-looking: it may exist in supplements I could not read. Falsification queries run: "Cellpose-SAM OR CellSAM OR micro-SAM failure cases ... error taxonomy" (only aggregate results), "Cellpose-SAM limitations failure OOD..." (found the brightfield anecdote and Archit & Pape).

**QC without GT:** Cellpose itself exposes flow-error and cell-probability outputs (FACT from Cellpose docs, not re-verified today), and multi-model consensus is an obvious proxy. Found no microscopy-specific paper that **validates** such proxies against GT as an estimator of per-instance F1/IoU. ConfIC-RCA is the closest general method, medical-only. Query run: "predicting segmentation quality without ground truth microscopy ...". The Ferrante paper itself notes the goal is hard for instance segmentation in general (snippet-level claim).

**VA tools for segmentation QC:** existing tools are pipelines/GUIs (Cell-ACDC, BMC Biol 2022, has manual correction; napari; CellProfiler Analyst); several GitHub pipelines include "QC visualizations". None looks like a research-VA contribution (no linked-view error-analysis system found). Thin.

**Saturation verdict:** models: saturated (active arms race, BSD/MIT code). Failure analysis and QC-proxy validation: **thin to active**. Cellpose-SAM is 16 GB VRAM-hungry per a snippet-level claim (UNVERIFIED for inference; see feasibility).

**Open data (license verified where noted):**

| Dataset | License | Notes |
|---|---|---|
| Cellpose "cyto/nuclei/cyto2" training data | HHMI terms: non-commercial, educational, research use only (FACT, cellpose.org/dataset) | size unverified; research use is fine for a course project |
| LIVECell (Sartorius) | CC BY-NC 4.0 (FACT: repo README via search; S3 bucket returns 200; images.zip about 1.2 GB per bucket listing) | 8 cell lines, phase contrast, COCO JSON; good for per-cell-line failure analysis |
| NeurIPS 2022 CellSeg | CC-BY-NC-ND per search snippet (unverified on Zenodo). "ND" is a risk only if redistributing derivatives; analysis is fine | multimodal (brightfield, fluorescence, phase, DIC); has labeled train/tuning sets |
| TissueNet | modified Apache, non-commercial academic (FACT via search snippet). Download via DeepCell may need a free account/key (unverified) | tissue imaging, 2-channel; a Cellpose 2.0 human-in-the-loop version is on Janelia figshare |
| BBBC collection | mostly CC0/free (unverified per set) | small, easy |
| Cell Tracking Challenge | unverified license | skip |

- FACT: Cellpose-SAM weights BSD-3, cellpose repo BSD-3, micro-SAM MIT, MEDIAR MIT, StarDist BSD-3, CellSAM Apache-2.0.

**Candidate RQs**

- *Grade-safe RQ1a:* "Which measurable image attributes (density, cell size, contrast, blur, cell line) predict Cellpose-SAM / micro-SAM / StarDist failures on LIVECell + NeurIPS-CellSeg, and can an interactive linked view expose them?"
  - Reproduction target: Cellpose-SAM (or Archit & Pape 2026) numbers on a public split.
  - Extension: compute per-image attribute table, per-instance error types (merge, split, missed, false positive), a Plotly/Streamlit linked view (attribute scatter/UMAP, error gallery, overlay). 
  - Evaluation: reproduce published F1@0.5 within tolerance; regression or tree of failure on attributes; user-task walkthrough with the student's own known cases (subjective, weak).
  - Falsification: the queries above found no per-image failure-attribute study, but the absence is not proven.
- *Higher-upside RQ1b:* "Does model disagreement (Cellpose-SAM vs micro-SAM vs StarDist vs CellSAM) and Cellpose flow-error rank per-instance/per-image segmentation quality without GT, and where do they lie?"
  - Reproduction target: same as above plus ConfIC-RCA conformal idea as a baseline.
  - Extension: Spearman/AUROC of proxy versus GT IoU across modalities, conformal calibration on held-out cell lines.
  - Extra risk: proxies may just correlate with image difficulty (uninformative); needs 4 model inference runs. Flow-error API access unverified. This is a quantifiable, non-subjective result, better for a workshop paper (e.g. a bio-imaging or MICCAI-workshop-type venue; unverified).
  - Falsification: no paper found validating it for microscopy, but the multi-model agreement idea is generic and likely to exist in some medical-imaging form.

**Feasibility (40 h):** High for 1a. Inference for Cellpose-SAM on M1 32 GB (MPS) or one 11 GB RTX: SAM ViT-L backbone inference is likely OK on 11 GB for moderate tiles (SPECULATION; test first). Data are small (LIVECell 1.2 GB). Frontend: Streamlit/Plotly only.
**Publication path:** grade-safe: none beyond the course. 1b: workshop (MICCAI/ISBI/NeurIPS workshop on bio-image analysis or the VIS-adjacent VisInPractice; unverified names).
**Red flags:** subjective evaluation of the VA piece; Cellpose-SAM tuned on the same public data (contamination when evaluating; Cellpose-SAM training used Cellpose, TissueNet, LIVECell-like sets, so test-set leakage must be checked in the paper); domain-biology reviewers may say "already known that brightfield is hard".

## 2. Area card (b): Cell representations, morphological profiling, DR reliability

**Instructor-lab overlap:** none found on cell profiling. Nonato's DR-quality/projection work is the natural bridge (unverified list).

**Key papers**

1. Kraus et al. (Recursion), *ViTally Consistent: Scaling Biological Representation Learning for Cell Microscopy*, arXiv 2411.02572 (preprint). MAE ViTs. `recursionpharma/maes_microscopy` repo license NOASSERTION (check the model license before use). OpenPhenom HF model license not shown in metadata (unverified).
2. Kenyon-Dean et al., *CellCLIP*, NeurIPS 2025 (OpenAlex record, 0 cites; author list unverified). Contrastive text-guided perturbation embeddings.
3. Fay/Recursion, *RxRx3-core*, arXiv 2503.20158. HF dataset `recursionpharma/rxrx3-core`, 1M-10M rows, about 40 GB storage (FACT via HF API; license field not returned, unverified).
4. Cell-DINO, PLoS Comput Biol 2025 (peer-reviewed); Cell Painting encoder evaluation papers.
5. *Same Encoder, Different Winner: A Paired-View Framework for Cell Painting Encoder Evaluation*, arXiv 2609.12761 (2026-09-11, preprint, days old). Finds that four community protocols (replicate mAP, scIB batch integration, CellProfiler feature prediction, cross-batch recall) **rank encoders differently**, and that segmentation retains 94% of replicate mAP but only 32% of R@10 on RxRx3-core. Promises to release checkpoints and code (AUTHOR CLAIM; not yet released as far as I verified).
6. Moshkov et al., *Learning representations for image-based profiling of perturbations*, Nat Commun 2024 (peer-reviewed).
7. Chandrasekaran et al., JUMP-CP consortium: `jump-cellpainting/datasets` (BSD-3, active). Also *Cell Painting: a decade of discovery*, Nat Methods 2024, and *Progress and new challenges in image-based profiling*, Mol Syst Biol 2026.
8. Ferrante-adjacent: none. DR reliability generic: Mahmood/scDEED (bioRxiv 2023, reliability score per embedded point), *Assessing and improving reliability of neighbor embedding methods: map continuity* (PMC12125374), "distortions" package (bioRxiv 2025). All scRNA-seq-flavored.

**Is "are UMAP phenotype clusters real?" studied in cell profiling?** FACT: search for "UMAP Cell Painting ... distortion trustworthiness" returned only scRNA-seq / generic DR-reliability papers; Cell Painting papers use UMAP descriptively (snippet-level). Two other queries (OpenAlex "UMAP distortion cell morphology profiling ...", "visual analytics tool morphological profiling embeddings") found no dedicated study or VA tool; the closest is a Current Protocols "similarity clustering and single-cell visualization" protocol and ProbeScout (generic image search VA, arXiv 2609.24110, not cells). SYNTHESIS: the **DR-reliability question applied to image-based-profile embeddings, with batch/plate effects as a confounder, looks under-studied**. A distinct microscopy risk versus scRNA-seq: batch/well-position effects create clusters that are real in the data but not biological. Recent "Same Encoder" paper hints protocol/context sensitivity, and no one linked it to projections.

**Saturation:** representation-learning models: saturated/active (Recursion, Broad, many preprints). DR reliability in this domain: thin. VA: thin.

**Open data:** JUMP-CP on AWS Open Data: `s3://cellpainting-gallery/cpg0016-jump/` (FACT: listed anonymously with `aws s3 ls --no-sign-request`; many sources). Profiles are small: source_4 workspace/profiles is 277 objects, 3.4 GiB (FACT). Full images are huge (order of tens to hundreds of TB; exact size unverified), so **use profiles/embeddings only, or a handful of plates**. Gallery license CC0 (unverified for this run). RxRx3-core: about 40 GB, HF (license not returned by API; check the dataset card). Other gallery sets present (cpg0043-segmentation, cpg0039 livecellpainting) verified as listed.

**Candidate RQs**

- *Grade-safe RQ2a:* "Do UMAP/t-SNE neighborhoods of Cell Painting embeddings (CellProfiler features vs OpenPhenom/DINO) preserve kNN structure, and how much of the cluster structure is plate/batch rather than treatment?"
  - Reproduction target: JUMP-CP profile-level benchmarks (jump-cellpainting/datasets), plus scDEED/trustworthiness-continuity metrics (open code for trustworthiness in sklearn).
  - Extension: per-point reliability overlay, colored by batch/treatment, in a Plotly linked view with sample images.
  - Evaluation: trustworthiness/continuity, kNN treatment-label purity vs batch purity, sensitivity to n_neighbors/min_dist.
  - Falsification: queries above show no cell-profiling-specific study; "distortions" package and scDEED exist for RNA.
  - Risk: essentially "apply known DR-reliability to a new domain", which the brief flags as weak novelty.
- *Higher-upside RQ2b:* "Encoder rankings flip across protocols (arXiv 2609.12761). Do projection-based views reveal why?" Reproduce their finding on RxRx3-core with public encoders (once their code is out), then add a projection/DR-uncertainty diagnostic. Extra risk: their code release is not verified; encoder inference on RxRx3-core images needs GPU time; scooping risk is high given the paper is 18 days old.

**Feasibility:** 2a is High if you stick to profiles (no image download). Images for cell galleries: only a few plates. **Compute:** OpenPhenom ViT-S inference feasible on 11 GB GPU (SPECULATION).
**Publication path:** workshop (bio-imaging or DR/VIS workshop); main venue only with 2b plus strong user study.
**Red flags:** novelty weak for 2a; batch-effect handling is a domain rabbit hole; RxRx3-core license unverified.

## 3. Area card (c): Cell tracking / lineage (brief)

- Not searched (budget); skip. Cell Tracking Challenge data is open for research but the license is unverified. SYNTHESIS: tracking evaluation is heavy in engineering (time-lapse, lineage-graph metrics) and I see no clear open question. Saturated by CTC leaderboards. Recommendation: no.

## 4. Triage table

| Sub-area | Best RQ | Novelty evidence | Feasibility | Grade-safety | Pub upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) Failure attribution of generalist seg models (RQ1a) | Image attributes vs failure, linked view | Benchmarks report aggregate/modality scores only; no per-image attribute study found | High | High | Low-Med | **yes** | Uses student's segmentation experience, small open data, code BSD/MIT |
| (a) GT-free QC proxies (RQ1b) | Model disagreement/flow-error as quality estimator | No microscopy validation found; medical ConfIC-RCA exists | Med | Med | Med | **yes (as extension of 1a)** | Quantitative and falsifiable; risk that proxy tracks difficulty only |
| (b) UMAP reliability in cell profiling (RQ2a) | Trustworthiness + batch confounding of Cell Painting UMAPs | No profiling-specific study found; scRNA-seq versions exist | High | Med-High | Low-Med | maybe | Weak-novelty "new domain" risk, Nonato overlap on DR |
| (b) Encoder-ranking flips (RQ2b) | Reproduce arXiv 2609.12761 finding, add diagnostics | Paper is 18 days old, code unreleased | Low-Med | Low | Med | no | Scooping and reproducibility risk |
| (c) Cell tracking | none | CTC saturated | Low | Low | Low | no | Heavy engineering |

## 5. Summary judgment

Best target: **(a)**, RQ1a as the deliverable with RQ1b as the extension. Biggest red flag: the novelty claim ("no one has studied where generalist cell-segmentation models fail per image") rests on absence-of-evidence from snippet-level searches; benchmark supplements and the Cellpose-SAM paper's own analyses may contain it, so read them fully before committing. Also confirm test-set leakage (Cellpose-SAM pretraining data vs LIVECell/CellSeg test splits) and confirm Cellpose-SAM inference runs on MPS or 11 GB CUDA in an hour-long smoke test.
