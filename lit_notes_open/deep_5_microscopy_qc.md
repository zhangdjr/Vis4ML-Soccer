# Deep dive D5: Failure analysis and ground-truth-free QC of generalist cell-segmentation models, with visual analytics

Review date: 2026-09-29. Falsification-first. Labels:
- **FACT**: I checked it this session in full text, code, an API response or a file listing.
- **AUTHOR CLAIM**: what a paper says about itself.
- **SYNTHESIS**: my inference from several facts.
- **SPECULATION**: a guess, flagged as one.

Evidence levels for sources are FULL TEXT, ABSTRACT or SECONDHAND.

**Budget used.** 16 WebSearch calls. The rest went through:
- WebFetch
- Europe PMC full-text XML
- bioRxiv JATS XML
- arXiv PDFs (read with `pdftotext`)
- OpenAlex, until its free daily budget ran out partway through (see §9)
- the Crossref and Zenodo APIs
- `gh api`
- `curl -I` and HTTP range reads of zip central directories

Full texts are saved as `lit_notes_open/pdfs/d5_*.txt`.

---

## 0. TL;DR verdict

**The grade-safe RQ ("which attributes predict failures, and do models fail on the same instances?") is only partly open.**

Coarse attribute analyses already exist:
- The NeurIPS22 challenge reports results per modality and explains the DIC failures by "very low contrast".
- Miao et al. 2026 split one dataset into high vs. low S/N.
- Archit & Pape (MIDL 2026) mark in-domain vs. out-of-domain datasets for each model.

None of them does any of the following (FACT for the papers I read in full; absence elsewhere is unproven):
1. per-instance error typing (merge / split / miss / false positive) across generalist models;
2. modeling of failure against continuous per-instance attributes;
3. measuring whether models fail on the same cells.

**The higher-upside RQ is the better one, and it now has a named, citable motivation.**
- Two tools already rank segmentations by inter-model agreement: BISCUIT (F1000Research 2025) and SEG (Sims et al., bioRxiv 2023).
- Both rest on an explicit, untested assumption. BISCUIT states it verbatim: *"Assuming that model prediction inaccuracies are uncorrelated between models, the model with the lowest score yields predictions closest to the ground truth."*
- In the open peer review, **Peter Bankhead** asks: *"Is that assumption likely to be true – or is there any way to assess whether it is true? … I would not necessarily expect it to be true whenever comparisons are made between overlapping methods, trained on overlapping training sets."* **Constantin Pape** asks for object-level (IoU) rather than pixel-level disagreement.
- In the version dated 06 Jul 2026 there is no author response to Bankhead's point. FACT, FULL TEXT including the reviews.
- MARC (arXiv 2609.13665, 12 Sep 2026) names the same limitation for spatial transcriptomics: consensus "remain[s] a surrogate rather than the ground truth, i.e., failure modes across segmentation methods may be reinforced". FACT, FULL TEXT.
- Zenk et al. (MedIA 2024) is the key benchmark. It shows that pairwise-Dice ensemble disagreement is the best image-level failure detector for **3D radiology semantic segmentation**. It explicitly leaves "other levels of failure detection" to future work, and it documents "silent" failures where ensemble members agree on a wrong region. FACT, FULL TEXT.

**Recommended RQ (refined).** On held-out open microscopy data (the LIVECell test split plus the NeurIPS22 labeled Tuning and Public-Test sets):
1. Does **instance-level cross-model agreement** rank per-cell segmentation quality without GT? Compare it against model-internal signals (Cellpose flow error and cell probability, SAM-style predicted IoU) and against an attribute-only baseline.
2. Is the "uncorrelated errors" assumption behind agreement-based QC violated? Measure instance-level error consistency between models that share a backbone and training data versus those that do not.
3. Where are the **silent failures** (all models agree, GT disagrees)? How many of them are label errors rather than model errors?

A Streamlit/Plotly triage view built on this ranking is evaluated by **simulated inspection**: errors found per K cells inspected, against GT. That needs no user study for the course.

**The grade-safe version is a strict subset:** error taxonomy, attribute small multiples and an error-consistency matrix on LIVECell. It can be finished by the Nov 3 update.

**Single biggest risk:** the agreement signal may only encode *image difficulty*. Dense, low-contrast images make all models disagree *and* fail, so a high AUROC could be trivial. The design (§7.5) controls for this in two ways:
- **within-image** AUROC;
- an **attribute-only** baseline that the proxy must beat.

If agreement adds nothing beyond attributes, that is still a reportable negative result about BISCUIT-style QC, but it weakens the workshop story.

---

## 1. Scope and course fit

- **Course topics hit** (SYNTHESIS):
  - model assessment (Sept 22): failure detection, risk-coverage;
  - black-box interpretation (Oct 6): signals from outputs only;
  - clustering / DR (Oct 13 / 20): optional instance-embedding view of the error space;
  - DL visualization (Oct 27).
- **Instructor fit** (SYNTHESIS): the lab's *Visagreement* (TVCG 2025) is about visualizing disagreement between explanation methods. This project applies "disagreement as a signal" to segmentation models, which is a natural hook for Silva without overlapping any of the lab's papers (§3).
- **The student's segmentation experience matters here.** The core engineering is instance matching, error typing and mask post-processing: routine for someone who has done segmentation, and error-prone for someone who has not.

---

## 2. Reproduction-target verification

### 2.1 Repos, licenses, pins (FACT via `gh api`, 2026-09-29)

| Repo | License | Last push | Latest release | Deps / pins | Notes |
|---|---|---|---|---|---|
| MouseLand/cellpose | BSD-3 | 2026-06-14 | v4.2.1.1 (2026-06-14) | `setup.py`: numpy>=1.20, torch>=1.6, torchvision, opencv-python-headless, fastremap, segment_anything. Loose pins, **not stale** | Built-in models are `cpsam` (Apr 2025), **`cpsam_v2`** (Jun 2026, "includes a fix in the training for low contrast regions"), `cpdino` and `cpdino-vitb` (DINOv3 backbone). FACT from the Cellpose docs. **cp4 refuses to load cp3 models** (v4.0.8 changelog: "fail on loading cp3 model in cp4"), so cyto3 needs a separate `cellpose<4` env |
| computational-cell-analytics/micro-sam | MIT | 2026-09-29 | v1.8.14 (2026-09-06) | `environment.yaml`: pytorch>=2.5, napari>=0.7.1,<0.10, torch_em>=0.9, segment-anything>=1.0.1, python-elf>=0.9. Conda-forge is recommended | APG is implemented in the repo (Archit & Pape 2026) |
| vanvalenlab/cellSAM | Apache-2.0 (code) | 2026-06-04 | no releases | pip from git | **Weights need a DeepCell API key** from users.deepcell.org, under a "modified Apache license for non-commercial academic use only" (FACT, `docs/API-key.md`). Free registration, not an NDA. Still a data-access risk |
| stardist/stardist | BSD-3 | 2026-02-14 | 0.9.2 (2025-12-15) | TensorFlow | Pretrained models are fluorescence-nuclei / H&E only. **Unsuitable for whole-cell phase-contrast LIVECell** (SYNTHESIS). Keep it only for a nuclei subset, or drop it |
| ScopeM/biscuit | MIT (repo); the notebook is CC BY-NC per a reviewer | 2025-12-09 | none | TF + PyTorch | The closest VA tool (§4) |
| lstrgar/seg (SEG, Sims 2023; `seg-eval` redirects here) | **none** | 2024-05-13 | none | | Pseudo-GT baseline. No license, so re-implement rather than copy |
| MIC-DKFZ/segmentation_failures_benchmark (Zenk) | Apache-2.0 | 2025-02-12 | | pins **pytorch 2.0.1**, CUDA 11.7 (stale) | Reuse only the AURC/risk-coverage logic (~30 lines). Don't install |
| murphygroup/CellSegmentationEvaluator | MIT | 2025-07-26 | | | Needs multichannel images, so it doesn't apply to LIVECell |
| sartorius-research/LIVECell | MIT (code) | 2023-06-29 | | | Data is CC BY-NC 4.0 |

**Model-internal QC signals: are they accessible? (FACT, code read)**
- **Cellpose.** `cellpose/dynamics.py:flow_error(maski, dP_net)` returns a per-mask mean-squared error between the predicted flows and the flows recomputed from each mask. `remove_bad_flow_masks(..., threshold=0.4)` drops masks above the threshold, and the docstring calls this "the QC step".
  - **Range-restriction confound:** with the default `flow_threshold=0.4`, every surviving mask has error < 0.4.
  - To use flow error as a ranking signal, run with `flow_threshold=0` and compute the errors afterwards. "Range restriction" means that you only observe the part of the score range that passed a filter, which deflates correlations.
- **micro-SAM.** In AMG mode (`AMGBase`) the outputs carry `predicted_iou` and `stability_score` per mask (FACT, `instance_segmentation.py` l.215–216).
  - Archit & Pape say APG's mask-decoder IoU predictions "give a quality estimate for each predicted mask" (AUTHOR CLAIM, FULL TEXT). They are used for NMS but never validated.
  - Whether APG's public API returns them per object is **unverified**; the code has an `output_mode` switch.
- **CellSAM.** The mask decoder "outputs pixel-wise probabilities for the cell and another IoU-based confidence value", and CellFinder box confidences are thresholded by k-means (AUTHOR CLAIM, FULL TEXT).

### 2.2 Weights (FACT via HF API + `curl -I`)

- **Cellpose-SAM.** `huggingface.co/mouseland/cellpose-sam`: files `cpsam`, `cpsam_v2`, `cpdino`, `cpdino-vitb`, license bsd-3-clause. `cpsam` resolves with HTTP 200 and Content-Length **1,233,587,898 B (~1.2 GB)**.
- **micro-SAM** `vit_b_lm`: downloaded automatically via pooch. Not re-checked this session (unverified URL).
- **CellSAM:** gated behind the token (above).

### 2.3 Data (FACT unless marked)

| Dataset | License | What I verified | Size |
|---|---|---|---|
| LIVECell (Edlund 2021) | **CC BY-NC 4.0** (README §LICENSE); code MIT | S3 `livecell-dataset/LIVECell_dataset_2021/` listed anonymously; `images.zip` returns 200 | images.zip 1.24 GB; `livecell_coco_test.json` 248.7 MiB (train 525.6, val 92.8) |
| NeurIPS22 CellSeg (Zenodo 10719375) | **CC BY-NC-ND 4.0** | Zip central directories read via HTTP range | Training-labeled 2.02 GB; **Tuning 0.63 GB = 101 images + 101 labels**; Testing 2.93 GB = **Public: 50 images + 50 labels (+2 WSI + labels)**; **Hidden: 400 images, no labels in the zip** (only osilab predictions and a readme) |
| BBBC038 / DSB2018 | **CC0** (BBBC page) | `stage1_train.zip` 200, 82.9 MB | Small, but **it is in the training data of every candidate model** (below). Not usable for held-out claims |
| Cellpose dataset | HHMI non-commercial research terms (from scan 7; not re-checked) | | Used by Cellpose-SAM for its OOD test set. Annotator-2 labels: release "upon publication" (AUTHOR CLAIM), and I could not find them released (unverified) |

- ND ("no derivatives") is fine for analysis. Don't redistribute modified NeurIPS22 labels.
- The NeurIPS22 hidden-test labels may sit elsewhere (grand-challenge), but I did not find them. Unverified.

### 2.4 Training-data leakage: the critical table

FACT, from the full texts of each paper. The quotes are in §5.

| Dataset split | Cellpose-SAM (cpsam) | Cellpose cyto3 | micro-SAM LM generalist | CellSAM generalist |
|---|---|---|---|---|
| LIVECell **train** | **trained** (sampling prob. 5%) | trained | **trained** | **held out** ("The LIVECell dataset was held out for zero-shot/few-shot tests") |
| LIVECell **test** | held out (evaluated) | held out | held out ("evaluated on a separate test split") | held out |
| NeurIPS22 Training-labeled | **616 of 1,000 images trained**; "We did not evaluate performance on the validation or test set for this dataset" | not listed | **trained** (train split) | not in the generalist; the NeurIPS-specific CellSAM variant trained on train+tuning |
| NeurIPS22 Tuning (101) | held out | held out | likely used as **val** (torch_em maps `val`→Tuning; the paper's use is unverified) | used by the NeurIPS variant only |
| NeurIPS22 Public test (50) | held out | held out | "test" split in torch_em, so held out (SYNTHESIS) | the NeurIPS variant used it "for validation"; the generalist held it out |
| DSB2018 / BBBC038 | **trained** (about half of the Cellpose nuclei set) | trained | **trained** (StarDist version) | **trained** |

**SYNTHESIS on leakage.**
- **LIVECell test** is split-level held out for all models but *in-distribution* for cpsam and micro-SAM. It is out-of-domain (zero-shot) for CellSAM.
- **NeurIPS22 Public-Test (50 images)** is the cleanest cross-model held-out set. The Tuning set (101) is a near-clean second set.
- **BBBC038 must not be used for "held-out" claims.**
- For GT-free QC, leakage matters less than for accuracy claims: QC ranks errors on the data being processed. But in-distribution data means fewer errors, so fewer positives. Report results **stratified by in-domain vs. out-of-domain per model**, as Archit & Pape do.

### 2.5 Compute (FACT unless marked)

- **11 GB GPU.** Cellpose-SAM Table S1 profiles inference on an RTX 4070S with 12 GB, and notes that exceeding GPU RAM "did slow down but was successful" (AUTHOR CLAIM). An 11 GB card should work.
- **M1 Pro (MPS).** The third-party repo `tamagnoP/cellpose-SAM-AppleMseries` reports **0.5–0.9 s per 512–1024 px 2D image** on an M2 Pro with cellpose 4.0.8 (AUTHOR CLAIM). Those are synthetic images with 4 objects; dense LIVECell images will be slower because of flow post-processing. Also: "MPS does not support 3D post-processing". 2D only here.
- **Estimate** (SPECULATION): about 1,700 images × 4–5 models × 1–5 s ≈ 2–10 GPU-hours. Feasible on SLURM or the RTX box.
- The student has not run HF/PyTorch on the RTX box before (PLAN.md), so a CUDA setup risk exists. The M1 is the fallback.

---

## 3. Instructor-lab and prior-cohort check

- **OpenAlex** (scan 7, same week): Silva (A5003584200), works since 2019, with queries `cell`, `microscopy`, `segmentation`, `biomedical`, `image` → **zero microscopy or cell-segmentation hits** (FACT per scan 7).
- **This session:**
  - OpenAlex hit its daily budget before I could sweep Nonato, Miranda, Xenopoulos, Rulff, Guardieiro, Solunke, Barr and Bertini individually. dblp was unreachable (HTTP 000).
  - Fallbacks:
    1. `ctsilva.github.io/publications/` (FACT: fetched; no title matching cell / microscop / segment / biolog / patholog / uncertain / disagree / failure / medical);
    2. a WebSearch for "Nonato" OR "Claudio T. Silva" OR "Fabio Miranda" with cell segmentation / microscopy / visual analytics → only unrelated cell-segmentation papers. The nearest item is Casaca & Nonato's *Laplacian Coordinates for Seeded Image Segmentation*, which is classical seeded segmentation (SECONDHAND).
  - **Individual sweeps of Nonato, Miranda, Bertini and the students are unverified.** Redo them tomorrow once the OpenAlex quota resets (§12).
- **Conceptual neighbour:** *Visagreement* (TVCG 2025, Silva/Nonato lab; FACT per deep_3/scan notes) visualizes explanation-method disagreement. It is a different object (feature attributions), but it is the natural framing citation.
- **Prior cohort** (FACT, `gh search repos`, created ≥ 2025-10-01): queries `cellpose`, `cellpose-sam`, `micro-sam`, `cell segmentation visualization`, `segmentation quality control`, `segmentation disagreement`.
  - About 80 repos, **none mentioning VisML, DS-GA 3001, Silva or NYU**.
  - Dec-2025 candidates (`phuongthao0515/Computer-Vision-Cellpose-SAM` 2025-12-02, `Tommer147/Cellpose-SAM_Colab` 2025-12-10, `hjhoppe-lang/CellPoseSam` 2025-12-04) have empty or unreadable READMEs. They are most likely other courses.
  - Two 2026 benchmark repos exist (`SharvenRane/cell-segmentation` "Cellpose vs StarDist vs SAM quantitative benchmark"; `YuvrajPuyam/cellseg-benchmark` on TissueNet). Neither mentions agreement or QC.
  - This is weak evidence, since student repos can be private.

---

## 4. Paper table

| Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Zenk et al., Comparative benchmarking of failure detection methods in medical image segmentation | 2024 | MedIA 99:103392 | peer-rev. | How should segmentation failure detection be evaluated, and which methods work? | 5 public 3D radiology sets (brain tumor, heart, kidney tumor, Covid, prostate), yes | Pixel-confidence aggregation, ensemble / MC-dropout pairwise Dice, quality regression, Mahalanobis, VAE | risk-coverage curves | **AURC** (risk-coverage); argues against Spearman/AUROC-only reporting | Ensemble + pairwise DSC is the best, and "a strong baseline … should be reported in future work" | Image-level only; "other levels of failure detection can be studied"; silent failures when "the ensemble members agree on the wrongly segmented region"; annotation shift | yes (Apache-2.0) | arxiv.org/abs/2406.03323 | FULL TEXT |
| Rantsiou et al., **BISCUIT** | 2025 | F1000Research 14:1277 | peer-rev. (open review, "reservations") | Help users pick a segmentation model visually | user images; demo data | Runs 11 models (Cellpose, Omnipose, StarDist…); pairwise overlap maps; mean pixel disagreement; object-level score added after review | side-by-side panels, overlap map, bar chart | none quantitative; qualitative demo | Visual inspection reveals differences that metrics hide (AUTHOR CLAIM) | **Assumes uncorrelated errors (untested; Bankhead)**; pixel-level (Pape); no micro-SAM; subjective | yes (MIT; notebook CC BY-NC) | doi.org/10.12688/f1000research.171889.1 | FULL TEXT + reviews |
| Sims et al., **SEG**: Segmentation Evaluation in absence of GT labels | 2023 | bioRxiv | preprint | Rank nucleus-segmentation methods per image without GT | 5 annotated HER2+ TMA cores (~70k nuclei) + 88-core unlabeled TMA | Ensemble majority-vote pseudo-GT from Gaussian-blurred centroids; leave-one-out method weights | — | pseudo-GT vs GT Dice >0.8; per-image P/R/F1 | Pseudo-GT ranks methods usefully (AUTHOR CLAIM) | Isotropic Gaussians; detection-level only (centroids, not masks); future: human-in-the-loop | yes (no license) | biorxiv 10.1101/2023.02.23.529809 | FULL TEXT via WebFetch summary |
| Shu et al., **MARC** | 2026 (12 Sep) | arXiv 2609.13665 | preprint | Predict multi-method consensus support for spatial-transcriptomics cell segmentation | Xenium kidney (4,642 held-out tiles) | U-Net regresses leave-one-method-out consensus maps | qualitative maps | Dice 0.90, IoU 0.82, cell-level Spearman 0.79 **vs consensus (not GT)** | Consensus is learnable from one mask + DAPI + transcripts | "consensus maps remain a surrogate rather than the ground truth … failure modes … may be reinforced"; single dataset | not stated | arxiv.org/abs/2609.13665 | FULL TEXT |
| Chen & Murphy, Evaluation of cell segmentation methods without reference segmentations | 2023 | Mol Biol Cell | peer-rev. | GT-free quality score for cell segmentation | 637 multichannel tissue images (HuBMAP), 4 modalities | 14 heuristic metrics → PCA score | scatter / PCA | Pearson 0.83 / 0.81 / 0.72 vs F1 / AvgF1 / SEG′ on **2 images** annotated by 2 experts | DL methods best; mask-matching post-processing helps | Needs multichannel images; per-image/method, not per-instance; weak GT validation | yes (MIT) | PMC10208095 | FULL TEXT |
| Archit & Pape, Revisiting foundation models for cell instance segmentation | 2026 | MIDL 2026 (PMLR 324) | peer-rev. | Compare CellPoseSAM, CellSAM, µSAM, SAM, SAM3 (+ APG) | 36 open 2D datasets | Benchmark; APG prompt generation | bar charts; in-domain bars textured | mSA @ IoU 0.5–0.95 | CellPoseSAM and APG consistently top-3; CellSAM < others; SAM2 "did not segment any objects for several datasets" | 2D only; box-prompt APG future work; no per-instance or failure analysis; IoU predictions used but not validated | yes (micro-sam, MIT) | arxiv.org/abs/2603.17845 | FULL TEXT |
| Pachitariu, Rariden, Stringer, **Cellpose-SAM** | 2025 | bioRxiv (still preprint per Crossref) | preprint | Generalist model beyond inter-annotator agreement | Combined open sets (see §2.4), license cc_by_nc | SAM ViT-L encoder + Cellpose flows | example galleries | error rate, AP; perturbation robustness (channel, size, noise, blur) | Error 0.163 vs inter-annotator 0.257; cyto3 0.292, CellSAM 0.328 (Cellpose test set) | Robustness via *synthetic perturbations*, no natural-attribute or per-instance analysis; flow-error threshold used only as a filter | yes (BSD-3) | biorxiv 10.1101/2025.04.28.651001 | FULL TEXT (JATS) |
| Archit et al., **Segment Anything for Microscopy** | 2025 | Nat Methods | peer-rev. | Fine-tune SAM for LM/EM, interactive + automatic | open (list in §2.4) | SAM fine-tuning, AIS decoder, napari | napari tool | mSA; user annotation time | Generalist beats default SAM; AIS competitive with Cellpose | Larger compute; "comparisons … heavily biased in our favor" on trained-on sets | yes (MIT) | PMC11903314 | FULL TEXT |
| Marks, Israel et al., **CellSAM** | 2025 | Nat Methods | peer-rev. | Foundation model with a box-prompt detector | open via DeepCell (token) | CellFinder (AnchorDETR) → SAM | galleries | 1−F1 on 10 sets; LIVECell zero-shot | ≈ specialist Cellpose; "we did not encounter catastrophic failure modes" (AUTHOR CLAIM) | Prompting is a limitation; the LIVECell test split was manually graded for **annotation quality** (good / medium / poor) | yes (Apache; weights gated) | PMC12695629 | FULL TEXT |
| Ma et al., NeurIPS22 multimodality cell segmentation challenge | 2024 | Nat Methods | peer-rev. | Universal cell segmentation benchmark | Zenodo, CC BY-NC-ND | 28 algorithms | dot/box plots per modality | F1 @ 0.5 on 422 hidden images | Top-3 robust; DIC failures from "very low contrast" | Aggregate / per-modality only; future 3D + classification | yes | arxiv.org/abs/2308.05864 | FULL TEXT |
| Miao et al. (Hickey lab, Duke), Foundation cell segmentation models performance on live microscopy and spatial-omics data | 2026 (Apr) | bioRxiv | preprint | Which generalist model for which modality; downstream impact | in-house phase/fluor (Duke repo) + HuBMAP CODEX | cyto3, cpsam, µSAM (APG), CellSAM, Mesmer, InstanSeg | overlays, tables | IoU / AP / SEG; **high vs low S/N split**; downstream clustering | "three SAM-based models … showed the highest degree of agreement with each other"; no universal winner on CODEX | Few images; in-house data; agreement not related to GT errors | stated `HickeyLab/Segmentation_Comparison` → **404 today** | PMC13131665 | FULL TEXT |
| Martí-Pérez et al., Microscopy cell segmentation: review and benchmarking | 2026 | J Imaging 12:297 | peer-rev. | Task-specific vs foundation models | 5 open sets | StarDist, CellSAM, Cellpose-SAM, YOLO-SAM | tables | mAP | Cellpose-SAM best in aggregate (scan 7) | Aggregate only | yes (github diegomartiperezz) | PMC13412841 | FULL TEXT (skimmed) |
| Cosarinsky et al., **ConfIC-RCA** | 2025/26 | IEEE TMI (per arXiv) | peer-rev. | Conformal intervals for GT-free segmentation quality | 10 medical sets incl. 2 WBC microscopy (semantic) | In-context RCA (UniverSeg / SAM2) + split conformal | — | coverage, interval width | Meaningful intervals, fast | Exchangeability breaks under shift; image-level semantic, not instances; human-in-the-loop studies needed | yes | arxiv.org/abs/2503.04522 | FULL TEXT (skimmed) |
| Chen et al., **Uni-Evaluator** | 2023 | IEEE VIS / TVCG | peer-rev. | Unified VA evaluation for classification / detection / instance segmentation | COCO-like | Probability-distribution formulation | matrix + table + grid | 2 case studies | Subset-level error discovery | GT-based; no microscopy; no GT-free QC | yes | arxiv.org/abs/2308.05168 | ABSTRACT |
| Haehn et al., **Guided Proofreading** of automatic segmentations for connectomics | 2018 | CVPR | peer-rev. | Rank candidate merge/split errors to speed EM proofreading | EM connectomics | CNN error classifiers trained on GT | proofreading UI | 20-novice user study; VI reduced "7.5x faster" | Ranked candidates beat manual proofreading | Needs a trained error classifier; EM only | yes | arxiv.org/abs/1704.00848 | SECONDHAND (search snippet) |
| Geirhos et al., error consistency | 2020 | NeurIPS | peer-rev. | Do two systems err on the same inputs? | ImageNet-style | Cohen's κ on trial-level errors | — | — | CNNs share errors strongly | classification only | yes | arxiv.org/abs/2006.16736 | ABSTRACT |
| Goel et al., Great models think alike … | 2025 | arXiv (ICML 2025 per venue listings; unverified) | preprint | Model-similarity metric (CAPA) | LLM benchmarks | chance-adjusted agreement on errors | — | — | "model mistakes are becoming more similar with increasing capabilities" | LLMs, not vision | yes | arxiv.org/abs/2502.04313 | ABSTRACT |
| Bruhns et al., Effects of segmentation errors on downstream analysis in multiplexed tissue imaging | 2025 | PLoS Comput Biol | peer-rev. | How much do segmentation errors distort phenotyping? | multiplexed tissue | **Simulated errors** via affine perturbations | — | clustering consistency | Moderate errors distort clusters | Synthetic errors | ? | PMC12456762 | ABSTRACT |

Also noted (ABSTRACT or page level):
- Toggle-Untoggle (bioRxiv 2025): per-cell toggle verification on cyto3 output, with no ranking.
- napari Mask Curator (Feb 2026): manual pick-best-mask among candidates; "does not compute automatic quality scores, IoU agreement … or ranking" (page summary).
- SAM IoU miscalibration under shift is known in general vision (RUAC, ICML 2026, arXiv 2605.10603; SQA-SAM arXiv 2312.09899). SECONDHAND.
- CASC-AI (arXiv 2502.07302): consensus used for *training* on noisy labels. Title only.

---

## 5. What the closest papers leave open (their own words, then my synthesis)

- **Zenk et al.** (FULL TEXT):
  - "we focused on image-level failure detection … other levels of failure detection can be studied".
  - "the ensemble is overconfident in this case, as the ensemble members agree on the wrongly segmented region. This results in such failures not being detected, i.e. silent."
  - "calibration of the pairwise DSC scores is a desirable secondary goal."
- **BISCUIT** (FULL TEXT):
  - The paper: "complementary scores, on top of the mean model disagreement, could be implemented".
  - Bankhead (reviewer): the assumption of uncorrelated errors needs discussion "whenever comparisons are made between overlapping methods, trained on overlapping training sets"; "Additional measurements … e.g. number of objects may help identify cases of over or under-segmentation."
  - Pape (reviewer): "object-level difference (e.g. based on IoU) … would likely be more meaningful".
- **MARC:** "failure modes across segmentation methods may be reinforced"; the candidate methods "are therefore not fully independent".
- **Archit & Pape:** APG's IoU predictions "give a quality estimate for each predicted mask". They are used for NMS, and never evaluated as a quality estimator.
- **Cellpose-SAM:** the flow-error threshold is "the quality control step", set to 0.4 in every analysis. It is never evaluated as a ranking signal.
- **CellSAM:** the authors themselves had to grade LIVECell test **annotation quality** by hand. GT noise is real there.
- **Miao et al. 2026:** SAM-based models agree most with each other. This is an observation, with no link to whether shared agreement means shared error.

**SYNTHESIS.** Three distinct communities each hold one piece:
1. Medical failure detection (Zenk, ConfIC-RCA): the evaluation protocol (AURC). Semantic, image-level, single-architecture ensembles.
2. Bio-image tooling (BISCUIT, SEG, Chen & Murphy): agreement-based model selection. Per-image, with no GT validation of the core assumption.
3. Model papers (Cellpose-SAM, micro-SAM, CellSAM, Archit & Pape): internal quality signals that exist but are unvalidated.

Nobody has put them together at the **instance level, across heterogeneous generalist models with known shared lineage, validated against GT**. The *error consistency* measurement (Geirhos-style κ) has not been applied to segmentation models at all, as far as I found.

---

## 6. Refined research questions and hypotheses

**RQ1 (grade-safe core).** On the LIVECell test split (8 cell lines) and the NeurIPS22 labeled held-out sets:
- What is the per-instance error profile (FN, FP, merge, split, poor boundary) of 4–5 generalist models?
- Which per-instance and per-image attributes predict each error type? Candidates: area, local density (neighbours within 2 diameters), touching fraction, local contrast / SNR, eccentricity, image modality or cell line.

**RQ2 (key novelty). Error consistency.** Do models fail on the *same* GT cells more than chance, and is the effect stronger for same-lineage pairs?
- Same-lineage pairs: cpsam ↔ cpsam_v2 ↔ cpdino (the Cellpose training set); µSAM ↔ CellSAM (SAM decoder).
- Cross-lineage pairs: e.g. cyto3 (CNN) ↔ µSAM.

**RQ3 (GT-free QC).**
- Does leave-one-out cross-model instance agreement rank per-cell quality of a target model (cpsam) better than:
  - (a) model-internal signals (flow error with `flow_threshold=0`, mean cell probability, µSAM/CellSAM predicted IoU);
  - (b) single-model test-time-augmentation (TTA) self-consistency (flip/rotate). This is the Zenk "ensemble" analogue for one model;
  - (c) an attribute-only baseline?
- Can it flag **false negatives**? Cells found by other models but missed by the target. Internal confidence cannot flag these by construction.

**RQ4 (VA triage).**
- Does a triage view that orders cells by the RQ3 score find more true errors per K inspected than random or confidence-only ordering (simulated inspection)?
- Among "silent" failures (all models agree, GT disagrees), what fraction are GT errors on visual adjudication?

**Hypotheses** (each falsifiable, each with a pre-registered threshold):
- **H1.** The error rate rises monotonically with density and touching fraction, and falls with contrast. The attribute effects differ across models: a model × attribute interaction is significant in a logistic GLMM with image as a random effect.
  - A GLMM (generalized linear mixed model) is a regression with random effects that absorbs per-image clustering.
- **H2.** Instance-level error-consistency κ is > 0 for all pairs, and is higher for same-lineage pairs than cross-lineage pairs (bootstrap CI excludes 0 for the difference).
  - *If H2 holds, agreement-based QC has a quantifiable blind spot.* That directly answers Bankhead.
- **H3.** For cpsam, cross-model agreement gives a within-image AUROC for "GT-IoU < 0.5 or unmatched" that is ≥ 0.05 higher than the best internal signal, and higher than the attribute-only baseline. Its AURC is lower.
  - **Falsified if** the agreement AUROC is within 0.02 of the attribute-only baseline.
- **H4.** Using *cross-lineage* reference models only yields better QC than same-lineage references (follows from H2).
- **H5.** At an inspection budget of 5% of cells, agreement-ranked triage recovers ≥ 2× the errors of random order. That is a low bar. The interesting comparison is against confidence-only ranking.
- **H6** (exploratory). ≥ 20% of sampled silent failures on LIVECell are label errors or ambiguous cells. This would matter for any LIVECell-based evaluation.

---

## 7. Project design

### 7.1 Data (exact subset)

- **LIVECell test, stratified subsample:** 50 images per cell line × 8 = **400 images** (704×520, phase contrast).
  - Download `images.zip` (1.24 GB) + `livecell_coco_test.json` (249 MB).
  - Convert COCO polygons to non-overlapping instance masks. Cellpose provides a `livecell_ann_to_masks` step, per the CellSAM methods.
  - Scale up to the full ~1,516 test images if time allows.
- **NeurIPS22:** Tuning (**101** labeled) + Testing/Public (**50** labeled, excluding the 2 WSIs).
  - Download `Tuning.zip` (0.63 GB) and `Testing.zip` (2.93 GB; only `Public/` is needed).
  - Modalities: brightfield, fluorescence, phase contrast, DIC. Modality labels for the tuning/public images must be derived. Unverified whether a metadata file exists.
- **Leakage labels.** Tag every (model, dataset) pair as in-domain or held-out, per §2.4.

### 7.2 Models

| Model | Family / lineage | Instance output | Internal signal |
|---|---|---|---|
| Cellpose-SAM `cpsam` (target) | Cellpose flows + SAM ViT-L | masks | flow error (threshold 0), cell probability |
| `cpsam_v2` and/or `cpdino-vitb` | same training lineage | masks | same |
| Cellpose `cyto3` (separate env, `cellpose<4`) | CNN, older lineage | masks | same |
| micro-SAM `vit_b_lm`, APG (or AIS) | SAM prompt/decoder lineage | masks | predicted IoU (if exposed), stability |
| CellSAM (optional, needs token) | SAM + DETR prompts; LIVECell held out | masks | IoU confidence, box confidence |

StarDist is dropped for LIVECell (pretrained nuclei-only). It could return on a fluorescence-nuclei subset only as an out-of-lineage reference.

### 7.3 Pipeline (the key engineering)

1. **Inference.** Save masks, flows and probabilities per model per image, with fixed parameters and default diameters. Run cpsam twice: `flow_threshold=0.4` (reproduction) and `0` (QC signals).
2. **Matching.**
   - Hungarian matching on IoU between prediction and GT.
   - Error taxonomy:
     - FN (GT unmatched at 0.5);
     - FP;
     - **merge** (a prediction overlapping ≥ 2 GT with intersection-over-GT > 0.5 each);
     - **split** (a GT covered by ≥ 2 predictions);
     - boundary (matched, IoU 0.5–0.75).
   - Hungarian matching is the optimal one-to-one assignment, used so that each GT cell is matched to at most one prediction.
3. **Attributes per GT cell:** area, equivalent diameter, eccentricity, local density, touching fraction (shared-boundary pixels / perimeter), local contrast and SNR (inside vs. dilated ring). Per image: cell line / modality, mean density, global contrast.
4. **Proxies per target-model instance:**
   - `agree_k` = max IoU with model k's instances; `agree_mean` = mean over references;
   - lineage-split versions;
   - TTA consistency;
   - internal signals.
5. **FN candidates:** instances predicted by ≥ m reference models with no target instance at IoU > 0.3.

### 7.4 Visualization, and why each view is scientifically necessary

1. **Image + error overlay (linked).** Error type colour-coded on the raw image, toggling per model.
   - *Necessary* to adjudicate silent failures and label errors (H6). No metric can separate "all models wrong" from "GT wrong".
2. **Attribute small multiples.** Error rate vs. each attribute, one line per model, faceted by cell line / modality.
   - *Necessary* to show the model × attribute interactions (H1). A single regression table hides where the curves cross.
3. **Error-consistency matrix.** Model × model κ heatmap, ordered by lineage. Plus an UpSet plot of which subset of models fails on each GT cell.
   - *Necessary* for H2. The UpSet plot shows the silent-failure mass (all-fail) against unique failures.
   - An UpSet plot is a matrix-based alternative to Venn diagrams for many-set intersections.
4. **Triage list + risk-coverage plot.** Cells ranked by the proxy, with a live AURC / precision@K curve against random and confidence rankings. Clicking a cell shows its crop in all models.
   - *Necessary* for RQ4. It is also the demo the course requires.
5. (Optional, ties to the Oct 20 lecture) a UMAP of per-cell attribute + proxy vectors, coloured by error type, to find error clusters. Only if time allows.

Implementation: Streamlit + Plotly (click events via `streamlit-plotly-events` or Plotly's native selections). Precompute everything into Parquet so the app does no inference.

### 7.5 Evaluation plan

- **Reproduction (course requirement):**
  - (a) cpsam AP@0.5 / error rate on the LIVECell test subset against Cellpose-SAM Fig. S2 (tolerance: within 0.03 AP, SPECULATION on the achievable match given the subset);
  - (b) the µSAM-APG and cpsam mSA on LIVECell and NeurIPS22 against Archit & Pape Table C.x.
  - Report deviations honestly: subset, preprocessing, `livecell_ann_to_masks`.
- **H1:** logistic GLMM (error ~ attributes × model + (1 | image)). Report odds ratios with CIs. Robustness check: gradient-boosted trees + partial dependence.
- **H2:**
  - Instance-level Cohen's κ between the binary error vectors of each model pair on the shared GT cells;
  - Chance-adjusted, bootstrapped over images;
  - Same- vs. cross-lineage contrast.
- **H3:**
  - AUROC, AUPRC and **AURC** (Zenk's protocol; risk = 1 − IoU of the matched GT, or 1 if FP), computed **within image** and pooled;
  - Spearman ρ with GT IoU;
  - partial correlation controlling for attributes;
  - baselines: random, internal signals, TTA, attribute-only logistic model (fit on NeurIPS Tuning, test on LIVECell and vice versa);
  - statistics: DeLong or bootstrap tests.
- **H4:** the H3 metrics for same-lineage-only vs. cross-lineage-only references.
- **H5 (simulated inspection):** errors recovered vs. number of cells inspected. Precision@K at K = 1, 5, 10% of cells. Compare against random and confidence-only ordering.
- **H6:** the student adjudicates 100 randomly sampled all-models-agree / GT-disagrees cells in view 1 (label error / model error / ambiguous). Report the proportion with a Wilson CI.
  - Weakness: single rater. Mitigation: have a second rater do 30 of them if possible.
- **Success criteria for the course:** reproduction within tolerance, and H1/H2 answered with CIs.
- **Workshop criterion:** H3 or H2 is clearly positive or clearly negative (both publishable).
- **Failure criterion:** the proxy only matches the attribute-only baseline *and* κ ≈ 0. That is an uninteresting null.

### 7.6 Reproduction component

The reproduction is Cellpose-SAM and micro-SAM-APG inference on public splits, checked against published numbers. The extension is the instance-level error / consistency / QC analysis and the triage VA. Both are demo-able in the same app.

### 7.7 Hour budget (≈40 h)

| Block | Hours | By |
|---|---|---|
| Env setup (cellpose 4 env, cellpose 3 legacy env, micro-sam conda env; optional CellSAM token) + 20-image smoke test on M1 and RTX | 4 | Oct 12 |
| Data download + LIVECell COCO→masks + NeurIPS Tuning/Public extraction | 2 | Oct 14 |
| Proposal (4 pp) with the smoke-test reproduction numbers | 3 | Oct 20 |
| Full inference, 4–5 models × ~550 images (mostly unattended) | 2 | Oct 25 |
| Matching + error taxonomy + attributes (unit-test on toy masks) | 5 | Oct 31 |
| Reproduction tables | 2 | Nov 2 |
| 1-page update (H1 + H2 first plots) | 1 | Nov 3 |
| Proxies (agreement, flow error, TTA, IoU preds) | 4 | Nov 12 |
| Statistics (GLMM, κ, AUROC/AURC, bootstrap) | 4 | Nov 19 |
| Streamlit app (4 views, precomputed Parquet) | 6 | Nov 28 |
| Adjudication of 100 silent failures | 2 | Nov 30 |
| Final report (8 pp) + slides + demo video | 5 | Dec 10 |
| **Total** | **40** | |

### 7.8 Main failure modes and mitigations

1. **The proxy encodes difficulty only.** Mitigation: within-image AUROC, partial correlations, and the attribute-only baseline (§7.5). A null is still a result about BISCUIT/SEG-style QC.
2. **In-distribution LIVECell has few errors for cpsam** (it trained on the LIVECell train split). Mitigation:
   - weight the analysis toward NeurIPS22 Public/Tuning (held out);
   - report in-domain vs. held-out separately;
   - LIVECell is in-domain for cpsam but zero-shot for CellSAM, which is itself a useful contrast.
3. **GT noise in LIVECell** (CellSAM authors graded test annotations good / medium / poor). Mitigation: H6 adjudication. Try to obtain the CellSAM LIVECell quality grades (unverified whether they are released in the CellSAM dataset).
4. **Environment conflicts** (cp3 vs cp4, TF for StarDist, napari pins in micro-sam). Mitigation: three separate envs that write masks to disk. Drop StarDist. CellSAM is optional.
5. **CUDA on the RTX box has never been used for PyTorch.** Mitigation: the M1 MPS path (2D only). Budget one SLURM job as a backup.
6. **The micro-SAM APG per-object IoU may not be exposed.** Mitigation: use AMG outputs, or call the predictor on the APG prompts directly (a few lines, SPECULATION). Otherwise drop that signal.
7. **Scooping.** MARC (Sept 2026), BISCUIT v2 and the Pape group (who reviewed BISCUIT and maintain APG) are all near this. Mitigation: post a preprint soon after the course. Frame around the *validated, instance-level, lineage-aware* angle.
8. **Weak VA novelty** ("just a dashboard"). Mitigation: the VA contribution is the adjudication workflow that separates label errors from correlated model errors (H6), plus the simulated-triage evaluation. Don't oversell the UI.

### 7.9 Publication path (what must be added after the course)

- **Add:**
  - 3–5 more held-out datasets from Archit & Pape's out-of-domain list (e.g. TOIAM, Usiigaci, Vicar; per-model training membership to be checked);
  - conformal calibration of the agreement score per instance (a ConfIC-RCA-style split-conformal, with a shift analysis);
  - a small user study with 3–5 bio-image analysts, measuring time to find planted or known errors, with vs. without the triage ranking. Guided Proofreading (CVPR 2018) is the precedent for this study design;
  - release of per-instance error tables as a benchmark resource.
- **Venues** (names real, dates unverified):
  - MICCAI UNSURE workshop (uncertainty);
  - BioImage Computing (BIC) workshop at ICCV/ECCV;
  - ISBI 4-page papers;
  - VIS VAHC workshop or a VIS short paper, if the VA and user study are strengthened;
  - F1000Research / Bioinformatics Advances as a tool-plus-benchmark note.
- **SYNTHESIS:** the strongest paper is "agreement-based QC of generalist cell segmentation has lineage-dependent blind spots". It is quantitative and answers a named reviewer's question.

---

## 8. Two options

| | Option A (grade-safe) | Option B (recommended: A + QC) |
|---|---|---|
| RQs | RQ1 + RQ2 | RQ1–RQ4 |
| Models | cpsam, cyto3, µSAM | + cpsam_v2/cpdino, CellSAM optional |
| Views | 1, 2, 3 | 1–4 |
| Risk | Low | Medium (proxy may be trivial; more envs) |
| Upside | course only; "per-instance failure atlas" | workshop paper |
| Cut line | all done by Nov 3 update | RQ3/RQ4 are added after Nov 3; if late, ship A |

---

## 9. Gap verdict: **partly addressed**

- **Already done:**
  - agreement-based per-image *model selection* (SEG 2023, BISCUIT 2025, Chen & Murphy 2023, MARC 2026);
  - image-level failure detection benchmarking in 3D radiology (Zenk 2024);
  - coarse per-modality / S/N failure summaries (NeurIPS22, Miao 2026);
  - aggregate cross-model benchmarks (Archit & Pape 2026, J Imaging 2026);
  - ranked-candidate proofreading in EM (Guided Proofreading 2018).
- **Open, as far as I found:**
  1. instance-level validation of cross-model agreement *against GT* for generalist LM cell-segmentation models;
  2. measuring error consistency and lineage effects, i.e. the uncorrelated-errors assumption that BISCUIT states and Bankhead questions;
  3. validation of Cellpose flow error and µSAM/CellSAM IoU heads as per-instance quality rankers;
  4. separating silent model failures from GT errors with a VA adjudication step.
- **Queries that returned nothing relevant:**
  - WebSearch: "cellpose flow error OR cell probability per-mask confidence correlates with IoU…"; "microscopy cell segmentation risk-coverage OR AURC OR selective failure detection foundation models disagreement 2025 2026"; "MetaSeg segment-wise IoU prediction … cell microscopy"; "visual analytics cell segmentation errors interactive linked views IEEE VIS OR EuroVis OR TVCG".
  - arXiv API: `abs:cell AND abs:segmentation AND abs:"visual analytics"` (0); `… "ground truth" AND quality AND estimat*` (0); `… failure AND detection` (6, all off-topic); `… disagreement` (4, off-topic except CASC-AI).
  - OpenAlex citers of Zenk (19): all radiology, none microscopy.
  - OpenAlex citers of Cellpose-SAM filtered by "disagreement consensus" (2, off-topic), "failure" (16, off-topic) or "uncertainty" (14, off-topic). Citers of micro-SAM filtered the same way turned up no QC-validation paper.
- **Not checked** (budget or quota):
  - OpenAlex citers of BISCUIT, SEG-2023 beyond the 5 found, and MARC;
  - Semantic Scholar;
  - VIS 2025/2026 full programs;
  - the CellSeg challenge citers beyond 11 (the OpenAlex quota ran out mid-sweep).

---

## 10. Novelty confidence (in words)

**Moderate.**

The *ingredients* are all known: ensemble disagreement as failure detection (Zenk), agreement-based pseudo-GT (SEG, BISCUIT), error consistency (Geirhos). A reviewer could call the combination "Zenk for microscopy".

What lifts it above a domain transfer:
1. The instance-level setting with **heterogeneous pretrained generalists whose training-data overlap is documented** (§2.4). This makes the lineage-dependence test (H2/H4) possible, which Zenk's same-architecture ensembles cannot address.
2. A **named, unanswered reviewer question** in the most directly related tool paper.
3. Unvalidated internal QC signals shipped in widely used tools (Cellpose's "QC step", µSAM's IoU head).

I read the full texts of the 5 closest papers (Zenk, BISCUIT, MARC, Archit & Pape, Cellpose-SAM) and of 5 more. My residual worry is a 2026 bioRxiv or MIDL/ISBI paper I missed from the Pape or Kreshuk groups, or a BISCUIT v2 that validates the assumption. Also, the OpenAlex citer sweeps were cut short.

---

## 11. 12b-style row

| Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|
| High: model assessment + black-box signals + DL viz; framing echoes the lab's Visagreement without overlap | Medium: workshop-grade and answers a named open question; main venue only with a user study + more datasets | Medium: 3 envs, instance matching, proxy-may-be-trivial risk; the student's segmentation background lowers it | Low–Medium: open data (CC BY-NC / NC-ND fine for research); CellSAM weights gated; leakage mapped | Low–Medium: 4 Plotly/Streamlit views, all precomputed | High: GT exists, so AUROC/AURC/κ are objective; only H6 adjudication is subjective | …the student wants a quantitative, GT-validated result that uses their segmentation skills, and is happy with a notebook-style app rather than a novel VA design |

---

## 12. Verify-this-yourself checks

1. **Smoke test** (1 h): run `cpsam` on 20 LIVECell test images on the M1 (MPS) and on one 11 GB RTX card.
   - Record s/image and peak memory.
   - Confirm that `flow_threshold=0` keeps all masks, and that `cellpose.dynamics.flow_error` returns one value per mask.
2. **micro-SAM:**
   - Confirm which NeurIPS22 split the LM generalist trained and validated on (micro-SAM supplementary, or `micro_sam/training` configs; torch_em maps val→Tuning, test→Testing/Public).
   - Confirm that APG can return per-object IoU predictions.
3. **CellSAM:** register at users.deepcell.org and confirm the token arrives quickly. If not, drop CellSAM (it only affects the SAM-lineage pair in H2).
4. **Instructor lab** (the OpenAlex quota was exhausted today): run `api.openalex.org/authors?search=Luis Gustavo Nonato` (A5011424640), Miranda (A5083220583), Bertini, and the students, filtered by segment / cell / microscop / uncertain / disagree. Also search Silva's Google Scholar for "segmentation disagreement".
5. **Scoop check before the proposal:**
   - Check F1000Research for a BISCUIT v2 (does it test the uncorrelated-errors assumption?).
   - Check OpenAlex citers of BISCUIT (W4416395959) and MARC.
   - Search bioRxiv/arXiv for "Pape" + "quality" / "uncertainty" 2026, and MIDL/ISBI 2026 programs for "failure detection microscopy".

---

## Sources (key URLs)

- Zenk et al. 2024: https://arxiv.org/abs/2406.03323 · code https://github.com/MIC-DKFZ/segmentation_failures_benchmark
- BISCUIT: https://doi.org/10.12688/f1000research.171889.1 (PDF https://f1000research.com/articles/14-1277/pdf) · https://github.com/ScopeM/biscuit
- SEG (Sims et al. 2023): https://www.biorxiv.org/content/10.1101/2023.02.23.529809v1 · https://github.com/lstrgar/seg
- MARC: https://arxiv.org/abs/2609.13665
- Chen & Murphy 2023: https://pmc.ncbi.nlm.nih.gov/articles/PMC10208095
- Archit & Pape 2026: https://arxiv.org/abs/2603.17845
- Cellpose-SAM: https://doi.org/10.1101/2025.04.28.651001 · models https://cellpose.readthedocs.io/en/latest/models.html · https://huggingface.co/mouseland/cellpose-sam
- micro-SAM: https://pmc.ncbi.nlm.nih.gov/articles/PMC11903314 · https://github.com/computational-cell-analytics/micro-sam
- CellSAM: https://pmc.ncbi.nlm.nih.gov/articles/PMC12695629 · https://github.com/vanvalenlab/cellSAM
- NeurIPS22 CellSeg: https://arxiv.org/abs/2308.05864 · data https://zenodo.org/records/10719375
- Miao et al. 2026: https://pmc.ncbi.nlm.nih.gov/articles/PMC13131665
- Martí-Pérez et al. 2026: https://pmc.ncbi.nlm.nih.gov/articles/PMC13412841
- ConfIC-RCA: https://arxiv.org/abs/2503.04522
- Uni-Evaluator: https://arxiv.org/abs/2308.05168
- Guided Proofreading: https://arxiv.org/abs/1704.00848
- Geirhos 2020: https://arxiv.org/abs/2006.16736 · Goel 2025: https://arxiv.org/abs/2502.04313
- Bruhns 2025: https://pmc.ncbi.nlm.nih.gov/articles/PMC12456762
- LIVECell: https://github.com/sartorius-research/LIVECell · s3://livecell-dataset/LIVECell_dataset_2021/
- BBBC038: https://bbbc.broadinstitute.org/BBBC038
- Apple-silicon Cellpose-SAM timings: https://github.com/tamagnoP/cellpose-SAM-AppleMseries
