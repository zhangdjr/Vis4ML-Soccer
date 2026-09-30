# D0: choice of the confirmatory held-out dataset N1 (PREREG_D5 Part B §B1)

Written 2026-09-30 on the HPC login node. Nothing heavy was run here: only Zenodo/HF/GitHub/Kaggle API calls, HTTP range reads of remote zips, and GT-only statistics computed on streamed masks. **No segmentation model has been run on any candidate.**

*Researched and drafted by a subagent; reviewed by the main session. **Committed before any model was run on N1 or any N1 data was downloaded beyond range-read GT masks.** This commit is the N1 pre-registration record.*
Labels: **FACT** = seen in a primary source (URL or file given). **SYNTHESIS** = my inference from facts. **SPECULATION** = plausible but unverified.

Working files (not committed; durable copies of the key ones, including `zdup.py`, `dup_res.json` and `mcs_masks.json`, are in `~/vis4ml_spikes/work/r4/d0_sources/`; the rest were in the session scratchpad): READMEs (`mcs_README.md`, `hspc_README.md`, `xi_*.md`), per-mask GT statistics (`mcs_masks_part1.jsonl` + `mcs_masks.json` for mCellSeg, `xi_masks.json` for Xiong), COCO jsons (`c6_*.json`, `rv_*.json`), and GT-overlay thumbnails (`viz/`). The per-model leakage sources are `/projects/weilab/zhangdjr/vis4ml_spikes/work/r4/d0_sources/leak_{cellpose,microsam,cellsam}.md`.

---

## 1. Summary

**N1 = mCellSeg** (Alam, Jackson, Lord & Meijering, UNSW; Zenodo [10.5281/zenodo.20174259](https://doi.org/10.5281/zenodo.20174259), v1.0, published 2026-05-15, **CC BY 4.0**). The labeled set has 200 images (100 HEK-293T, 100 HUVEC) and **16,199 whole-cell instance masks**. None of them is below 20 px (the smallest GT object is 200 px).
**N1 uses 198 of the 200 labeled images, with 15,975 GT cells.** Two images are dropped by a deterministic GT-only de-duplication rule (§6): each is the same field as another image, with 86–93% of instances matching at IoU > 0.5. The images are **label-free DIC/bright-field**: every image I inspected is a grayscale image stored as three identical RGB channels. Annotation is dense, manual and expert-reviewed, and the README says model-assisted annotation was explicitly avoided. The needed part is the `mCellSeg/labeled/` subtree: **660.6 MB compressed** (the full zip is 1,213.5 MB).

It passes all six B1 rules. It is not in any roster model's documented training or validation data, and it was first made public on 2026-05-14/15. That is after every roster checkpoint's training data was fixed, and 3 days before the `cpsam_v2` weights were uploaded. Its source is a lab (UNSW Lord/Meijering) that supplies none of the upstream sets.

**Runner-up: the Xiong three-murine-line phase-contrast set** (Zenodo [10.5281/zenodo.21440736](https://doi.org/10.5281/zenodo.21440736), 2026-07-19, CC BY 4.0 on the record). It has 243 images and 16,531 cells, all acquired in November 2025. It lost because:
- it is sparse (the median image has 2.8% foreground);
- some visible cells are not annotated;
- the annotation provenance is undocumented;
- a stray "LICENSE_SELECTION_REQUIRED.md" inside the zip contradicts the record's license.

**Main risk (SYNTHESIS):** mCellSeg cells are large. The median over images of the per-image GT median equivalent diameter is 130 px, the maximum is 327 px, and about 50% of cells are in images whose median diameter is above 120 px. Cellpose-SAM was trained on diameters of 7.5–120 px, and the B1 preprocessing does no rescaling. Cellpose-SAM's error rate on the 40× images is therefore the thing to watch (§7). I keep every non-duplicate image anyway (198) and pre-declare size strata as descriptive only (§6), rather than selecting images to suit the models.

---

## 2. Candidate table

The rules are B1's, applied in order. R1: 2-D LM instance masks. R2: public, with a license that allows analysis. R3: ≤ 2 GB for the needed part. R4: not in any roster model's training/validation data. R5: released after April 2025 (a preference). R6: ≥ 1,500 GT cells after the < 20 px exclusion.
"Sci" is the scientific tie-break: roster competence, whole-cell, label-free, dense and complete annotations, and a clean mask mapping.

| # | Name | URL / DOI | Release | Modality · cells | #img | #cells (≥ 20 px) | Needed size (verified) | License | R1 | R2 | R3 | R4 | R5 | R6 | Sci / verdict |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **mCellSeg** | zenodo 20174259; Kaggle mirror `tukunzil/mcellseg-microscopy-cell-segmentation-dataset` | Kaggle v1 2026-05-14; Zenodo 2026-05-15 | DIC + bright-field (grayscale-in-RGB) · HEK-293T, HUVEC, whole cell | 200 labeled (+100 unlabeled, unused); **198 used** after de-duplication | **16,199 (16,199)**, FACT, computed from all 200 masks; matches the README "~16,199". **15,975 in the 198 used images** | labeled/ = 649.3 + 11.3 MB compressed (full zip 1,213.5 MB; md5 `ed06d6e4c10e93b81703984cef852246`) | CC BY 4.0 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | **CHOSEN.** Dense (image foreground up to 0.96), manual, whole-cell, label-free, single label map with no overlaps. Risk: large cells (§7) |
| 2 | Xiong murine lines | zenodo 21440736 | 2026-07-19 (images acquired Nov 2025, per filenames) | phase contrast · OP9, MC3T3-E1, NRG, whole cell | 243 | 16,567 polygons; 16,531 visible in the label maps (16,495 ≥ 20 px) | raw_images 1,058.3 MB + instance_masks 2.9 MB compressed (full zip 1,897.4 MB) | CC BY 4.0 on the record; the zip's `LICENSE_SELECTION_REQUIRED.md` says "No license has been assigned" | ✔ | ✔ (ambiguity recorded) | ✔ | ✔ | ✔ | ✔ | **RUNNER-UP.** Very sparse (foreground 0.9–4.6%, median 2.8%); some round/bright cells are unannotated (thumbnail check); annotation method undocumented; masks are mutually exclusive (later polygons overwrite earlier ones; 36 ids lost) |
| 3 | HSPC2 | zenodo 22163254 | 2026-08-29 (images from Scanlon et al. 2022, *Sci Rep* 12:16218) | bright-field · primary human HSPC | 421 | 2,446 (not computed; the cells are large, so SPECULATION: ≈ all ≥ 20 px) | 381.5 MB zip | CC BY 4.0 | ✔ | ✔ | ✔ | ✔ | ~ (deposit is new; images are from 2022) | ✔ | Reject on Sci. 2–20 cells per image (median 6; counts cluster at 2/4/8), so these are time-lapse frames of expanding clones: strong pseudo-replication and sparse, easy scenes |
| 4 | Glioma C6 | zenodo 17352019; arXiv 2511.07286 | 2025-10-14 | phase contrast · rat C6 glioma | 75 | 12,028 cell polygons (spec train 5,387 / valid 738 / test 2,035; gen 3,868) + 7,833 "soma" polygons (not cells); all ≥ 89 px | dataset.zip 947.0 MB | Zenodo: CC BY-NC 4.0; paper: "CC BY-NC-ND 4.0" (conflict) | ✔ | ✔ (NC; ND per paper) | ✔ | ✔ | ✔ | ✔ | Reject on Sci. **Overlapping annotations by design** ("it includes overlapping annotations", arXiv), only 75 images for the image bootstrap, and a stricter, conflicting license |
| 5 | Revvity-25 | HF `YaroslavPrytula/Revvity-25`; arXiv 2508.01928 (CVPRW 2025) | CVPRW June 2025; HF 2025-08-05 | bright-field · cancer cells (unnamed) | 110 | 2,937 (all ≥ 20 px) | 172.0 MB | CC BY-NC 4.0 | ✔ | ✔ (NC) | ✔ | ✔ | ✔ | ✔ | Reject on Sci. Amodal/overlapping polygons (overlap is 3.3% of foreground on average; one image reaches 100%, suggesting duplicate polygons); about 27 cells per image; NC |
| 6 | Partaker benchmark | zenodo 20577330 | 2026-06-07 | phase contrast · *E. coli* co-culture | 45 | 17,501 | co-culture-1.zip 45.2 MB | CC BY 4.0 | ✔ | ✔ | ✔ | ~ | ✔ | ✔ | Reject. GT "initialized using the omnipose_bact_phase model and exhaustively corrected" (README). The GT is therefore shaped by a model whose training data (Omnipose bact_phase) is in cyto3, Cellpose-SAM, micro-SAM and CellSAM. Also bacteria (livecell_cp3 is likely > 0.6 error), 4 chamber positions, time-lapse |
| 7 | Zenodo 22039777 | zenodo 22039777 | 2026-08-21 | **materials micrographs** (Grain, EMPS, Aachen-Heerlen, UHCS, MetalDAM, Super/EBC) | 3,625 | n/a | n/a | CC BY; **access: restricted** | ✘ | ✘ | – | – | – | – | Fail R1: not cells; all repackaged materials-science sets |
| 8 | HeLa COCO | zenodo 17719468 | 2025-11-26 | inverted microscope (Olympus IX70 + ORCA), HeLa connexin lines | 104 (72 train / 32 val) | 1,757 (smallest 2,332 px) | 94.1 MB | CC BY 4.0 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ (barely) | Reject on Sci. The only category is "Alive cells", and some filenames contain "dead", so dead cells are not annotated. Train and val differ in magnification (median area 13k vs 38k px) |
| 9 | SAMCell PBL-HEK / PBL-N2a | github.com/saahilsanganeriya/SAMCell release v1 | 2025-04-14 | phase contrast · HEK293, N2a | 5 + 5 | 1,948 (1,865) + 1,714 (1,647) | 8.1 + 10.0 MB | repo MIT (no separate data license) | ✔ | ✔ | ✔ | ✔ | ✘ (before cpsam's posting) | ✔ | Reject: 5 images per set is useless for an image bootstrap |
| 10 | Aitslab EGFP-Galectin-3 | zenodo 12193890 (PMC11751569) | 2024-11-19 | fluorescence (EGFP-Gal3) · U2OS | 60 | > 2,200 (paper) | not checked | CC BY 4.0 | ✔ | ✔ | likely ✔ | ✔ | ✘ | ✔ | Reject: pre-April 2025, and the paper says not all cells are necessarily outlined |
| 11 | BM-Cells49 | figshare 33238200 | 2026-08-13 | stained bone-marrow smear (bright-field) · 49 blood-cell classes | 2,466 | not counted (annotation.json is 302 MB) | 586 MB zip | CC BY 4.0 | ✔ (borderline domain) | ✔ | ✔ | ~ (BCCD blood smears are in Cellpose-SAM training: a domain overlap) | ✔ | likely ✔ | Reject on Sci. Hematology smear, not culture; class-targeted annotation (SPECULATION: RBCs and unclassified cells are unannotated) |
| 12 | Duke live-microscopy benchmark | Duke RDR 10.7924/R4R505 (PMC13131665) | bioRxiv 2026-04-21 | phase contrast (B16-F10, A549), fluorescence A549 | small (A549: 6 fine-tune + 3 validation images, per paper) | not checked | not checked | CC BY 4.0 | ✔ | ✔ | ? | ✔ | ✔ | ? | Not vetted further: too few annotated images for N1 |
| – | Fallback: NeurIPS22 Tuning | zenodo 10719375 | 2022 | mixed | 101 | – | – | CC BY-NC-ND 4.0 | ✔ | ✔ | ✔ | ✘ (micro-SAM v3/v4 validation) | ✘ | ✔ | Not needed |

Notes:
- **mCellSeg, FACT, from all 200 masks streamed from the zip:**
  - Per image: 16–264 cells (median 62). Shapes range from 668×908 to 3440×3440. All masks are 2-D uint16.
  - Foreground fraction reaches 0.96.
  - 161 masks have non-consecutive ids, although the README says "consecutive". So we relabel (§5).
  - 1,470 labels (9.1%) have more than one 4-connected component. On five sampled masks, the minor components hold 0.001–2.8% of label pixels: thin protrusions and stray strokes.
  - All 200 image stems pair 1:1 with `<stem>_mask.tif`.
- **Modality, SYNTHESIS from the filename tokens plus five inspected images (all grayscale, identical channels):**
  - Filename tokens: `_BF`: 38 images / 3,366 cells. `_DIC`: 35 / 1,898. `czi-C_2`/`-C_3`: 46 / 2,384. Other names (confocal `C004Z…` series, HUVEC `CD7`/`FumGW`/`TREX` series): 81 / 8,551.
  - The inspected `HEK-GFP_…czi-C_2` image is a DIC/transmitted channel, not GFP fluorescence. The 1024² `HEK_6h_…` images are transmitted-light confocal. The HUVEC series is DIC.
  - So the whole set is label-free, as delivered. The README's "DIC and fluorescence" describes acquisition: fluorescence helped the annotators, but the released images are the transmitted channel.
  - The mSAMUNet GitHub README says "phase-contrast and fluorescence". That wording conflicts with the Zenodo README. I go with what the pixels show.

---

## 3. Per-model leakage evidence

The general argument has two independent legs.

- **(a) Lists (FACT):** mCellSeg is not named in any model's training or validation list, and neither are its images through any upstream set.
- **(b) Timeline (FACT, then SYNTHESIS):** mCellSeg first became public on Kaggle (v1, `lastUpdated 2026-05-14T08:21:56Z`, from the Kaggle API) and on Zenodo (created 2026-05-15T04:24Z). Every roster checkpoint's training data predates that:
  - cyto3: 2024/2025;
  - Cellpose-SAM v1: bioRxiv 2025-05-01;
  - micro-SAM v4 weights: last-modified 2025-06-12/13;
  - CellSAM v1.2: the NM 2025 paper models, announced Sept 2025.
  - `cpsam_v2` weights were uploaded to HF on 2026-05-17, **3 days** after mCellSeg went public. Training a ViT-L Cellpose model on the 22k-image mix (v1: 2,000 epochs on 8×H200) takes far longer than 3 days. So mCellSeg in `cpsam_v2`'s training set is implausible unless the UNSW authors shared data privately (SPECULATION; no evidence of this).
- **Upstream/source check (FACT, from mCellSeg README):** the images are new acquisitions at UNSW. Imaging used a Zeiss LSM 880 or a Zeiss Cell Discoverer 7. Samples were HEK-293T transfected with GFP-TREX1(D18N) and primary HUVEC (Lonza CC-2529A). Some filenames carry 2024 dates (`HEK_6h_20240306_…`).
  - The README and the paper abstract call it "a novel dataset". Neither cites a previous release.
  - The labs are UNSW's M. Lord (biomaterials) and E. Meijering (image analysis). Meijering is a Cell Tracking Challenge co-organizer, but the CTC 2D sets used by micro-SAM (`DIC-C2DH-HeLa`, `PhC-C2DH-U373`, …) are other labs' data on other cell lines. SYNTHESIS: no image overlap.

### 3.1 Cellpose-SAM v1 (`cpsam`)
- **FACT** (bioRxiv 10.1101/2025.04.28.651001v1, p.3, "Model design"; I spot-checked the quote against the extracted PDF text at `/tmp/claude-62321234/cp/cpsam_joined.txt`): "This dataset combines major currently available datasets: Cellpose, Cellpose Nuclei, Omnipose, TissueNet, LiveCell, YeaZ, DeepBacs, Neurips 2022, MoNuSeg, MoNuSAC, CryoNuSeg, NuInsSeg, BCCD, CPM 15+17, TNBC, LynSec, IHC TMA, CoNIC, PanNuke".
- Methods p.11: "We used 18 publicly available datasets for training Cellpose-SAM." NeurIPS22 contributed 504 images from its Training set (`leak_cellpose.md` §1).
- **Why mCellSeg is not in it:** mCellSeg is not listed, and it did not exist publicly until 2026-05-14.
- **Upstream sources:**
  - Cellpose cyto2 comes from IDR, CCDB:6843, BBBC, Jones et al. and Micro-Net plus user images. That was fixed before 2024.
  - NeurIPS22 Training is from 2022.
  - LIVECell, TissueNet and the others are all older third-party sets.
  - **Partial overlaps (SYNTHESIS):** modality only. DIC and bright-field cultured cells occur in cyto2 and NeurIPS22. HEK293 may occur in cyto2's internet-sourced images (unverifiable). No image-level overlap is possible.

### 3.2 Cellpose-SAM v2 (`cpsam_v2`)
- **FACT** (cellpose docs models.rst): "``cpsam_v2``: this is the CellposeSAM model released in June 2026 using the SAM-ViTL backbone, it includes a fix in the training for low contrast regions".
- README: "it now predicts fewer spurious masks in low-contrast regions."
- The HF commit uploading `cpsam_v2` is dated 2026-05-17.
- **No training-data list exists** (`leak_cellpose.md` §2).
- **Why mCellSeg is not in it:** SYNTHESIS, by timeline (above). SPECULATION: v2 reuses v1's data mix, since it is benchmarked on the same test sets. That would make mCellSeg absent for the same reason as for v1.

### 3.3 Cellpose3 `cyto3`
- **FACT** (Stringer & Pachitariu, *Nat Methods* 22:592, PMC11903308, Methods "Segmentation datasets"): "We used nine publicly available datasets for training … the super-generalist 'cyto3' model."
- The nine are: cyto2, Cellpose nuclei, TissueNet, LiveCell, Omnipose fluorescent, Omnipose phase, YeaZ phase, YeaZ bright-field and DeepBacs. The total is 8,402 training images (`leak_cellpose.md` §3).
- **Why mCellSeg is not in it:** not listed; published about 2 years after cyto3.

### 3.4 micro-SAM `vit_b_lm` / `vit_l_lm` (v4, the weights that micro_sam 1.8.14 downloads from `…/1.2/files/`)
- **FACT** (`doc/datasets/lm_v4.md`, spot-checked against the saved copy): "The `LM Generalist v4` model was trained on 14 different light microscopy datasets":
  - LIVECell, DeepBacs, TissueNet, PlantSeg (Root), NeurIPS CellSeg;
  - CTC (`BF-C2DL-HSC`, `BF-C2DL-MuSC`, `DIC-C2DH-HeLa`, `Fluo-C2DL-Huh7`, `Fluo-C2DL-MSC`, `Fluo-N2DH-SIM+`, `PhC-C2DH-U373`, `PhC-C2DL-PSC`);
  - DSB (StarDist subset), EmbedSeg, YeaZ, CVZ Fluo, DynamicNuclearNet, CellPose, OmniPose, OrgaSegment.
- The v4 weights files have last-modified dates 2025-06-12 (vit_b) and 2025-06-13 (vit_l).
- Validation splits come from the same 14 datasets (NeurIPS "val" = Tuning; `leak_microsam.md` §2–3).
- **Why mCellSeg is not in it:** not listed, not in any validation split, and it postdates the weights by 11 months.
- **Validation exposure: none.** The mSAMUNet paper trained its *own* micro-SAM-framework models on mCellSeg. It reports "mSAMUNet outperformed microSAM (F1 = 0.7071 vs. 0.6994)", per PubMed 42224852 / ScienceDirect S0169260726002245. Those are separate checkpoints and do not touch the official v4 weights (SYNTHESIS).
- **Partial overlap (SYNTHESIS):** `DIC-C2DH-HeLa` (CTC) and NeurIPS22 contain DIC cultured cells, so the modality is seen in training, but these are different cell lines and labs.

### 3.5 CellSAM (`cellsam_general`, v1.2)
- **FACT** (Marks et al., *Nat Methods* 2025, PMC12695629, Methods "Dataset construction", spot-checked in the saved PMC text): "To train CellSAM, we combined 10 separate datasets spanning a variety of modalities: TissueNet, DeepBacs, BriFiSeg, Cellpose, Omnipose, YeastNet, YeaZ, the 2018 Kaggle DSB, a collection of H&E datasets and an internally collected dataset of phase microscopy images across eight mammalian cell lines (Phase400). The LIVECell dataset was held out".
- Supp. Table 1 lists the Phase400 lines: 3T3, A549, CHO, **HEK293**, HeLa, HeLa-S3, PC3, RAW264 (`leak_cellsam.md`).
- **Why mCellSeg is not in it:** not listed; published after v1.2.
- **Partial overlap (FACT + SYNTHESIS):** the **HEK293 cell line** is in Phase400: 17 train images, phase contrast, Van Valen lab, 1608×1608. mCellSeg's HEK-293T images are a different lab, instrument and modality (DIC/BF), so there is no image overlap. Still, CellSAM has seen the HEK morphology in phase contrast. BriFiSeg (bright-field nuclei) is a modality neighbour only.

### 3.6 Our own B1 fine-tunes and `livecell_cp3`
- Both are trained on LIVECell only: A172, BT474, BV-2, Huh7, MCF7, SH-SY5Y, SkBr3, SK-OV-3, all Incucyte phase contrast.
- There is no HEK or HUVEC and no DIC. mCellSeg is out-of-distribution for them.

### 3.7 Runner-up (Xiong) leakage, briefly
- It is not in any of the five lists above.
- It became public 2026-07-19. The images are timestamped `Image__2025-11-08…` to `…-11-10`, i.e. acquired in November 2025. That is after every roster checkpoint's data, including `cpsam_v2` (weights uploaded 2026-05-17), so there is no possible overlap (FACT: dates; SYNTHESIS: conclusion).
- The authors fine-tuned their *own* Cellpose-SAM on 223 of the images (`checkpoints/primary_models/cellpose_sam_checkpoint_00019.pth`). That does not affect the official weights, but it means the Xiong GT was used to train a Cellpose-SAM derivative. **Unknown:** whether its annotations were bootstrapped from Cellpose-SAM predictions (no annotation protocol in the package).
- Cell-line overlap: none known. OP9, MC3T3-E1 and NRG appear in no listed set.

---

## 4. License: what we may and may not commit

- **FACT:** mCellSeg is **CC BY 4.0** on Zenodo (`license: cc-by-4.0`), in the README ("This dataset is released under the Creative Commons Attribution 4.0 International (CC BY 4.0) licence") and on Kaggle ("Attribution 4.0 International (CC BY 4.0)").
  - It carries no NC and no ND term.
  - The mSAMUNet code repo is MIT, which is irrelevant to the data.
- **Allowed:** analysis, and publishing derived masks, overlays and figures with attribution. Cite Alam et al., *Comput. Methods Programs Biomed.* 285:109470 (2026) and Zenodo 10.5281/zenodo.20174259.
- **Project policy (unchanged):** raw pixels and GT masks stay out of git because of their size. We may commit:
  - per-cell and per-image tables;
  - manifests with sha256;
  - small figure crops with GT or prediction overlays, with the attribution line "mCellSeg (Alam et al. 2026), CC BY 4.0".
- Runner-up Xiong: CC BY 4.0 per the Zenodo record. The zip's `LICENSE_SELECTION_REQUIRED.md` says "No license has been assigned automatically…". If Xiong is ever used, treat the record's license as governing, but do not publish its pixels without re-checking.

---

## 5. Exact download spec (for a SLURM job, not the login node)

- **Source:** `https://zenodo.org/api/records/20174259/files/mCellSeg.zip/content`
  - 1,213,469,890 B, md5 `ed06d6e4c10e93b81703984cef852246`, 507 entries.
  - Mirror: Kaggle `tukunzil/mcellseg-microscopy-cell-segmentation-dataset` (requires a login).
- **Option A (simplest):** `wget` the whole zip (1.21 GB) into `/projects/weilab/zhangdjr/vis4ml_spikes/data/n1/raw/`, verify the md5, then extract `mCellSeg/labeled/*` and `mCellSeg/README.md` only.
- **Option B:** range-read only the 400 `mCellSeg/labeled/` members (660.6 MB compressed) with `d0_sources/zx.py`.
  - Zenodo returned **HTTP 429** during the GT-statistics streaming here. Add about 1.5 s between members, with retry and backoff (as in scratchpad `zmask.py`).
- **Members used:** `mCellSeg/labeled/images/<stem>.tif` (200) and `mCellSeg/labeled/masks/<stem>_mask.tif` (200).
  - `mCellSeg/unlabeled/` (100 images) is **not used**.
  - Uncompressed: images 2,473.6 MB (RGB uint8, three identical channels); masks 1,649.1 MB (2-D uint16).
- **Conversion** into `/projects/weilab/zhangdjr/vis4ml_spikes/data/n1/`:
  1. `img = tifffile.imread(images/<stem>.tif)`. Assert `img.ndim == 3 and img.shape[-1] == 3`.
     - If all three channels are identical, write `images/<stem>.tif = img[..., 0]` (uint8, 2-D, zlib-compressed). This is equivalent under Part A §1 ("A 3-channel image with identical channels is treated as 2-D").
     - Otherwise write the RGB unchanged and log it. I expect 0 such images; five of five checked were identical.
  2. `m = tifffile.imread(masks/<stem>_mask.tif)`. Assert 2-D and `m.shape == img.shape[:2]`.
     - `m = skimage.segmentation.relabel_sequential(m)[0].astype(np.uint16)` (max 264 labels per image).
     - Write `labels/<stem>.tif` (zlib).
  3. **Overlaps:** none. It is a single label map, so each pixel belongs to at most one cell.
  4. **Ignore regions:** none. 0 = background and there is no ignore value. Unannotated objects (e.g. floating rounded cells or debris in some HUVEC fields) are background, as in the source.
  5. **Multi-component labels** (9.1% of labels): keep as **one instance**. Do not split them and do not drop fragments; the GT cell is the label id.
  6. Apply the Part A **< 20 px exclusion** in the pipeline as usual. 0 cells are expected to be excluded (the smallest GT object is 200 px).
  7. Stems are kept verbatim. One stem contains `+` (`Lipo+siRNA_HEK_40x_1_BF`). There are no spaces, and all 200 stems are unique case-insensitively. Check the loader's glob and CSV handling on `+`.
  8. Write `manifest.csv` (stem, H, W, n_cells, cell_line from filename, modality token, gt_median_diam, sha256 of image and label).
- **De-duplication (§6):** do **not** write the two dropped stems, `HEK-GFP-FumGW_P3_40x_3_BF` and `HEK_48h_TREX_z2_C004Z007`. List them in `manifest_dropped.csv` with the reason.
- **Expected output:** **198** `images/*.tif` and 198 `labels/*.tif`, **Σ n_cells = 15,975** (all 200 would give 16,199). Assert both numbers in the conversion job.
  - Per-image shapes over all 200: 70 × 2796², 62 × 1024², 38 × 1104×1376, 8 × 3440², 6 × 2048², 4 × 2208×2752, plus 12 smaller crops. The dropped pair removes one 2796² and one 1024² image.
  - Disk (SYNTHESIS): about 0.85 GB uncompressed for grayscale images, less with zlib; labels about 20–50 MB compressed.
  - Delete `raw/` after verification (home disk is tight, though this is under `/projects`).
- **Preprocessing at run time:** Part A §1 unchanged. The 1st–99.8th percentile uint8 scaling is applied to all models. micro-SAM is tiled (1024 / halo 256) on the 87 used images with a side over 1536 px.

## 6. Which images are used

**All labeled images except two GT-identified near-duplicates: 198 images, 15,975 cells.** There is no subsampling and no selection by size, modality or difficulty. This is decided now, before any model run.
- **De-duplication rule (GT-only, deterministic, fixed now):**
  - Screen: among same-shape image pairs, compute foreground IoU of the GT masks downsampled to 256². For every pair with IoU ≥ 0.6 (135 pairs), compute the share of instances in one mask that match an instance in the other at IoU > 0.5.
  - A pair is a **near-duplicate** if that share is ≥ 0.5 in either direction. Drop the lexicographically later stem.
  - FACT: exactly two pairs qualify; every other screened pair is ≤ 0.143:
    - `HEK-GFP-FumGW_P3_20x_1_BF` / `HEK-GFP-FumGW_P3_40x_3_BF`: 0.93 / 0.85. Same field despite the "20x"/"40x" names. **Drop `…40x_3_BF`** (109 cells).
    - `HEK_48h_TREX_z2_C004Z006` / `…Z007`: 0.85 / 0.86. Adjacent z-planes of one field. **Drop `…Z007`** (115 cells).
  - The screen also checked `HEK-GFP-FumGW_CO_20x_1_BF` vs `…CO_Hoechst_20x_1_BF` (154 vs 155 cells): foreground IoU 0.175, so not the same field. Script: scratchpad `zdup.py`; results in `dup_res.json`.
  - Limit: a duplicate that is shifted or rescaled, or has a different image shape, would not be caught.
- **Why no other subsampling:**
  - The < 20 px exclusion removes nothing.
  - 198 images is a reasonable image-bootstrap unit count.
  - Selecting images by cell size or modality to suit the models would be a researcher degree of freedom.
- **Pre-declared descriptive strata** (reported, not confirmatory), for the 198 used images. They are all defined from filenames or GT only:
  - (i) cell line as stated in the filename: `HUVEC` if the stem contains "HUVEC" (case-insensitive), else `HEK` if it contains "HEK", else `unstated`. FACT: this gives 52 / 91 / 55 images and 3,228 / 8,096 / 4,651 cells. The README says 100 HEK-293T + 100 HUVEC, so filenames alone cannot recover the true split: the `P*_40xoir-C_3`, `*Polymer*`, `P*_Hep_*` and `Lipo_20x` series name no line. Do not guess.
  - (ii) per-image GT median equivalent diameter ≤ 120 px (Cellpose-SAM's documented training range) vs > 120 px. FACT: 77 images / 8,015 cells are ≤ 120 px, and 121 / 7,960 are above.
  - (iii) modality token (`DIC`, `BF`, other).
- **Bootstrap unit:** the image, as pre-registered. Within-series correlation remains (§7.6).

## 7. Risks and caveats

1. **Cell size vs Cellpose-SAM's training range (SYNTHESIS, the main risk).** About half the cells sit in images with a median diameter above 120 px, reaching 327 px on the 40× 2796² / 3440² images. Cellpose-SAM's docs state "trained on images with a range of diameters from 7.5 to 120 pixels", and B1 preprocessing does not rescale.
   - If `cpsam` or `cpsam_v2` over-split large cells, their error rate could approach the 0.6 exclusion. R2 and R3 would then be "not testable" on N1.
   - Mitigating facts:
     - Round 3 found `cpsam` accuracy 0.95 on NeurIPS22 Public-Test, which also has large objects.
     - cyto3 and livecell_cp3 rescale through the auto-diameter.
     - micro-SAM tiles contain whole cells (cell ≤ 330 px < tile 1024).
   - The pre-registered K3 control (GT-median diameter) exists for the diameter-taking models. **No rescaling rule is added now**, so the Part A preprocessing is not changed after the fact. The ≤ 120 / > 120 stratum is reported descriptively.
2. **`cpsam_v2` training data is undocumented** (`leak_cellpose.md` §2). The exclusion of mCellSeg rests on the timeline (public 3 days before the weights were uploaded), not on a list.
3. **micro-SAM validation never touched mCellSeg** (FACT: the lm_v4 list and splits; the weights date from 2025-06). mSAMUNet's micro-SAM-based fine-tunes on mCellSeg are separate checkpoints. The v4 validation also used NeurIPS Tuning, which is irrelevant here.
4. **Modality description conflict.** The Zenodo README says "DIC and fluorescence", the GitHub README says "phase-contrast and fluorescence", and the pixels are transmitted-light grayscale. I inspected only 5 of 200 images visually. The conversion script should log any image whose three channels are not identical, or whose intensity statistics look like fluorescence (mostly dark background). Such images stay in: no post-hoc exclusion.
5. **Annotation quirks (FACT):**
   - Non-consecutive ids in 161/200 masks (fixed by relabelling).
   - 1,470 multi-component labels, kept as one instance each.
   - Some floating or rounded objects are unannotated (HUVEC fields). These can create unmatched predictions, i.e. FPs. Round-3 per-GT-cell error does not count FPs, but agreement-based QC over predicted cells would see them.
   - Annotations are drawn outlines (ImageJ/Labkit). Some boundaries between touching cells are approximate, and IoU-0.5 matching tolerates this.
6. **Correlated images (FACT: filenames; SYNTHESIS: effect).** Several series are replicate fields of one experiment (`P1…P4_…_40x_k`, `HUVEC_FumGW_01…12`, `HEK_6h_*_z1/z2`). `z1`/`z2` pairs have very different cell counts (e.g. 256 vs 68), so they are different fields, not z-planes of one field.
   - The instance-level screen in §6 found and removed exactly two same-field pairs.
   - Image-bootstrap CIs may still be somewhat optimistic because of the within-series correlation. A series-cluster bootstrap (cluster = stem with trailing field and plane indices stripped) can be reported as a sensitivity analysis. It is not pre-registered as primary.
7. **Cell-line and modality neighbours in training (partial overlap, recorded):** HEK293 is in CellSAM's Phase400 (phase contrast), and DIC cultured cells are in CTC `DIC-C2DH-HeLa` (micro-SAM) and NeurIPS22 (Cellpose-SAM, micro-SAM). There is no image-level overlap with any model.
8. **The mSAMUNet paper benchmarks micro-SAM on mCellSeg** (F1 0.70). This is a published accuracy number, not a leak, but it shows the set is "known" to the micro-SAM community. The official checkpoints predate it.
9. **Runner-up caveats**, in case mCellSeg must be abandoned before any model run (e.g. the download fails):
   - Xiong's sparsity (median 2.8% foreground) and unannotated cells;
   - its mutually exclusive rasterization (36 overwritten ids);
   - its license-file ambiguity.
   - Its needed part (1.06 GB compressed) fits the ≤ 2 GB rule.
