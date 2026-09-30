# D5 Onboarding: "When Do Cell Segmenters Fail Together?"

**Purpose.** This document takes you from "I skimmed a lot of topics" to "I can write a strong 4-page proposal for D5 by Oct 20." Read it top to bottom once, which takes about 60–75 minutes, then work through the reading list in Part 3.
**Written:** 2026-09-30, from the D5 deep dive, the novelty check, and two cluster spikes (A1, B1). Every number here comes from those files; §12 lists the sources.
**Labels:** **FACT** = checked against a source or our own run. **SYNTHESIS** = my inference. **SPECULATION** = a guess to test.

---

## Contents
1. [The project on one page](#1-the-project-on-one-page)
2. [Concepts you need (primer)](#2-concepts-you-need-primer)
3. [The literature: what exists, what's open, what to read](#3-the-literature-what-exists-whats-open-what-to-read)
4. [What we have already measured](#4-what-we-have-already-measured)
5. [Research questions and hypotheses](#5-research-questions-and-hypotheses)
6. [Study design](#6-study-design)
7. [The visual analytics part (and why it is scientifically necessary)](#7-the-visual-analytics-part-and-why-it-is-scientifically-necessary)
8. [Scope tiers, milestones and hours](#8-scope-tiers-milestones-and-hours)
9. [Risks and mitigations](#9-risks-and-mitigations)
10. [Course fit, the Silva connection, and the Oct 6 pitch](#10-course-fit-the-silva-connection-and-the-oct-6-pitch)
11. [Proposal blueprint (4 pages, section by section)](#11-proposal-blueprint-4-pages-section-by-section)
12. [Open decisions, things to verify, and where everything lives](#12-open-decisions-things-to-verify-and-where-everything-lives)

---

## 1. The project on one page

**The setting.** Biologists increasingly segment microscopy images with **generalist cell-segmentation models**, pretrained models meant to work on any cell image without retraining. The main ones are Cellpose-SAM, micro-SAM, CellSAM, and older Cellpose models such as cyto3. Nobody has ground truth for their own images, so they need a way to tell **which cells were segmented wrong** without it.

**The common trick.** Run several models and trust the cells where they **agree**. Tools like **BISCUIT** (F1000Research 2025) are built on this. BISCUIT states the assumption outright: *"Assuming that model prediction inaccuracies are uncorrelated between models, the model with the lowest score yields predictions closest to the ground truth"* (FACT, full text).

**The problem.** If two models tend to make the **same mistakes**, their agreement is false comfort: they agree *and* are both wrong. These are "silent failures". A peer reviewer of BISCUIT, Peter Bankhead, asked exactly this. He doubted the assumption holds "whenever comparisons are made between overlapping methods, trained on overlapping training sets". **The authors never answered him** (FACT, the open review, still unanswered in the July 2026 version).

**Your research question, in one sentence:**
> *When do generalist cell-segmentation models fail on the same cells, what about the models (random seed, fine-tuning, shared SAM encoder, decoding objective, training data) predicts it, and when does that make agreement-based quality control untrustworthy?*

**Why it is publishable, not just a course project.**
- It answers a **named, published, unanswered question** (Bankhead's).
- It extends a known classification result (Gontijo-Lopes et al., ICLR 2022: errors become less correlated as models differ more) to **per-cell instance segmentation**. Nobody has done this there (§3).
- It ties to your instructor's lab. Visagreement (Silva lab, TVCG 2025) only *conjectured* that disagreement signals error. You test the analogous claim rigorously, against ground truth.

**What we already found** (§4, preliminary, 40 LIVECell test images, 10,292 cells). Per-cell error agreement, measured with Cohen's κ (§2):
- two random seeds of the same model: **0.92**
- Cellpose-SAM vs the older Cellpose cyto3: **0.79** (different encoders, same Cellpose family)
- Cellpose-SAM vs micro-SAM: **0.58** (same kind of SAM encoder, different groups)

So **sharing a SAM encoder did *not* make errors alike. Belonging to the same model family did.** That goes against the intuitive guess, which is exactly the kind of result that makes a paper. And agreement worked better as a quality check when the two models were *less* alike (AUROC 0.71 → 0.78).

**The big caveat.** All of this is on LIVECell, which most of these models were trained on. The results must be **replicated on a held-out dataset** (NeurIPS22 CellSeg Public-Test) before you claim anything.

**Your role as the researcher.** You design the hypotheses, the metrics and the visualizations. The spikes proved the pipeline works. The science still needs you.

---

## 2. Concepts you need (primer)

Read these in order. Each one is a building block for the next.

**Instance segmentation.** Outlining *each individual object* (each cell) separately, as opposed to semantic segmentation, which only labels pixels as "cell" or "background". Cells touch and overlap, so separating neighbours is the hard part.

**Generalist cell segmenters used here:**

| Model | What it is | Instance "decoding" (how it turns pixels into separate cells) | Group / lineage |
|---|---|---|---|
| **Cellpose-SAM (`cpsam`)** | SAM's ViT-L image encoder + Cellpose's flow output | Predicts **flow fields** (each pixel points toward its cell's centre), then follows the flows to group pixels into cells | Cellpose (Stringer/Pachitariu, HHMI) |
| **Cellpose3 `cyto3`** | Older CNN (U-Net) Cellpose generalist | Same **flow-field** decoding | Cellpose |
| **`livecell_cp3`** | Cellpose 3 model trained specifically on LIVECell | Flow fields | Cellpose |
| **micro-SAM `vit_b_lm`** | SAM ViT-B fine-tuned for light microscopy | **AIS**: a decoder predicts foreground, centre and boundary-distance maps, followed by a watershed. (AMG, the other mode, is SAM's prompt-and-filter mask generator.) | micro-SAM (Archit, Pape) |
| **CellSAM** | SAM encoder + a DETR box detector that prompts SAM | Box prompts → SAM masks | Van Valen lab |

The table shows why "lineage" is a bundle. Cellpose-SAM and cyto3 share the **objective and decoding** (flows), the group and much of the **training data**, but *not* the encoder. Cellpose-SAM and micro-SAM share an **encoder type** (SAM), but *not* the decoding, the data recipe or the group.

**Matching and IoU.** IoU (intersection over union) measures how well a predicted cell outline overlaps a true one. It runs from 0 (no overlap) to 1 (identical). You pair predictions with ground-truth cells using **Hungarian matching**, an optimal one-to-one assignment, and call a GT cell "found" if its best match has IoU > 0.5.

**Error types per GT cell** (our taxonomy, `match.py`):
- **miss:** no prediction covers it.
- **merge:** one prediction swallowed it together with a neighbour.
- **split:** it was cut into several predictions.
- **false positive (FP):** a prediction with no real cell.
- (optional) **boundary error:** matched, but with IoU only 0.5–0.75.

Merges and splits come mostly from the *decoding* step. Misses can come from the encoder or the data. That difference matters for the hypotheses (§5).

**Error indicator.** For model *m* and GT cell *c*: *e_m(c)* = 1 if *m* got cell *c* wrong, else 0. Every per-cell analysis starts here.

**Cohen's κ as "error consistency"** (Geirhos et al., NeurIPS 2020). This is how often two models are wrong on the *same* cells, corrected for chance.
- Formula: κ = (observed agreement − expected agreement) / (1 − expected agreement), where expected agreement comes from each model's error rate alone.
- κ = 0 means the two models' errors are unrelated. κ = 1 means they fail on exactly the same cells.
- **Why not just count shared errors?** Two models that are each wrong 30% of the time will share some errors by pure chance. κ removes that.

**κ_max and the accuracy trap.** When two models have very different error rates, κ *cannot* reach 1. For example, if model A errs on 10% of cells and model B on 60%, they can't fail on the same cells. So always report **κ / κ_max** and each pair's accuracies. (This is why CellSAM's rows are uninformative: it misses about 72% of cells.)

**Image-level (cluster) bootstrap.** Cells in the same image aren't independent: a blurry image makes every cell harder. To get honest confidence intervals, you resample *whole images* with replacement, not individual cells, and recompute κ each time. We used 1,000 resamples.

**Ground-truth-free QC signals** (ways to guess "this cell is wrong" without GT):
- **Cross-model agreement:** the IoU between two models' outlines of the same cell. Low agreement flags a likely error.
- **Model-internal signals:** Cellpose's **flow error** (how self-consistent the predicted flows are; Cellpose uses it as its own "QC step" with a threshold of 0.4), the mean cell probability, and micro-SAM AMG's **predicted IoU**.
- **Test-time augmentation (TTA) consistency:** flip or rotate the image, re-segment, and see whether the outlines agree.
- **Attribute-only baseline:** predict errors from cell size, density, contrast and similar. This is critical: if agreement only beats this baseline by a hair, agreement is just "hard images are hard".

**Evaluating a QC signal:**
- **Within-image AUROC:** how well the signal ranks wrong cells above right cells *inside each image*, averaged over images. 0.5 is chance and 1 is perfect. "Within image" matters because otherwise the signal gets free credit for separating easy images from hard ones.
- **Risk–coverage curve and AURC** (Zenk et al. 2024). Keep only the cells the signal is most confident about ("coverage") and measure the error rate among them ("risk"). AURC is the area under that curve; lower is better. It is the standard failure-detection metric in medical segmentation.
- **Simulated inspection:** if a biologist checks the top-K flagged cells, how many real errors do they find? This replaces a user study.

**Silent failure.** A cell where both models are wrong, *and* they agree with each other (IoU > 0.5 between their outlines). Agreement-based QC will never flag it.

**In-distribution vs held-out.**
- **In-distribution:** the test images come from a dataset the model was trained on. Different images, same distribution.
- **Held-out:** the dataset was never used in training.

LIVECell *train* was used to train Cellpose-SAM, cyto3 and micro-SAM, so the LIVECell *test* split is in-distribution for them. **NeurIPS22 CellSeg Public-Test** is the clean held-out set.

**The model-relatedness hierarchy** (the "levels" in the paper):

| Level | What differs between the two models | Our example pair |
|---|---|---|
| L0b seed | only data order and augmentation during fine-tuning | 3 fine-tunes of `cpsam` |
| L0a reinit seed | random initialization from scratch | (to do: cyto3-architecture U-Nets from scratch) |
| L1 checkpoint variant | one is a fine-tune of the other | `cpsam` vs `cpsam_ft_s1` |
| L2 shared SAM encoder, different group | decoder, objective, data, group | Cellpose-SAM vs micro-SAM |
| L3 different encoder, same family | encoder (SAM ViT vs U-Net) | Cellpose-SAM vs cyto3 |
| L3′ different encoder, different family | everything | micro-SAM vs cyto3 |
| L4 same architecture, different data | training data | cyto3 vs `livecell_cp3` (imperfect: both saw LIVECell) |

The classification result (Gontijo-Lopes) predicts that error agreement falls as you go down the levels. **The interesting question is where segmentation breaks that order.**

---

## 3. The literature: what exists, what's open, what to read

### 3.1 The map: four groups, each holding one piece

1. **General ML: error consistency and diversity** (classification and LLMs).
   - *Gontijo-Lopes et al., ICLR 2022* (FACT, full text): on ImageNet, "model pairs that diverge more in training methodology (in order: reinitializations → hyperparameters → architectures → frameworks → datasets) produce increasingly uncorrelated errors." Their metric isn't chance-corrected, and they don't study QC.
   - *Geirhos et al., NeurIPS 2020:* the κ metric; different CNN architectures share errors strongly (κ up to 0.79).
   - *Klein et al. 2025:* bootstrap CIs for κ.
   - *Saxena et al., NeurIPS 2024:* the source of diversity decides whether agreement-based performance estimation works. Seeds that only change data order are *not enough*.
   - *Kim et al., ICML 2025:* more accurate LLMs share more errors.
   - **Role for you:** the framing precedent and the metric. You are *not* discovering that "errors decorrelate as models diverge".
2. **Medical-image failure detection** (semantic segmentation, per image).
   - *Zenk et al., MedIA 2024:* the benchmark and the AURC protocol. Seed ensembles are the best image-level failure detector. **Future work, in their words:** "other levels of failure detection can be studied". They also describe silent failures, where "the ensemble members agree on the wrongly segmented region."
   - *Kirscher et al. 2026* (arXiv, read in full): seed ensembles vs cross-validation-fold ensembles, image level only, no error-correlation measure, no microscopy.
   - *RBQE, arXiv 2609.10495, Sept 2026:* **your closest precedent.** Cross-model agreement flags failed polyp segmentations *per image*. A different-architecture referee beats a same-architecture seed referee (AUROC 0.960 vs 0.923). It is image-level and polyps only, measures no error correlation, and has no SAM-lineage levels.
3. **Bio-image tooling: agreement-based model selection.**
   - *BISCUIT (F1000Research 2025):* the assumption under test, plus Bankhead's and Pape's reviews. Pape asked for *object-level* disagreement, which is what you do.
   - *SEG (Sims 2023)*, *Chen & Murphy (MBoC 2023):* ground-truth-free evaluation per image.
   - *MARC (arXiv 2609.13665, Sept 2026):* admits consensus "may reinforce" shared failure modes.
4. **The model papers themselves.**
   - Cellpose-SAM (bioRxiv 2025), micro-SAM (Nature Methods 2025), CellSAM (2025), and the NeurIPS22 CellSeg challenge (Nature Methods 2024).
   - *Archit & Pape (MIDL 2026)* benchmark them all, but only in aggregate.
   - They each have internal quality signals (flow error, predicted IoU) that **nobody has validated** as per-cell error rankers.

**SYNTHESIS: the gap.** Nobody has combined these at the **per-cell level, across generalist models with known shared lineage, validated against ground truth.** The Geirhos-style κ hasn't been applied to segmentation models at all, as far as ~15 searches plus 3 external LLM cross-checks found.

### 3.2 Also cite (not threats)
- *"In search of truth: evaluating concordance of AI-based anatomy segmentation models"* (arXiv 2512.15921): GT-free agreement among 6 CT segmenters, semantic, with no relatedness hierarchy.
- *"Segmentation quality assessment by automated detection of erroneous surface regions"*, Comput. Biol. Med. 2023: ensembles that "largely agree on mistakes". So **frame silent failures as *quantified across the hierarchy*, not discovered.**
- *Bommasani et al. 2022* ("component sharing") and *Toups et al. 2023* ("systemic failure"): the vocabulary for why shared components might cause shared failures.
- *Visagreement* (Silva lab, TVCG 2025), Case Study 2: the conjecture that disagreement goes with model error.

### 3.3 Reading list, in priority order

Plan for about 6–8 hours spread over the next two weeks. For each paper, extract the listed items into your notes. They feed the proposal directly.

| # | Paper | Time | What to extract | Where it goes in the proposal |
|---|---|---|---|---|
| 1 | **BISCUIT** + its open peer reviews (F1000Research 14:1277; doi 10.12688/f1000research.171889.1) | 45 min | The exact "uncorrelated errors" sentence; Bankhead's and Pape's comments; what BISCUIT computes (pixel disagreement) | Intro (motivation), Related work |
| 2 | **Gontijo-Lopes et al., ICLR 2022**, "No One Representation to Rule Them All" (arXiv 2110.12899) | 60 min | Their 5 categories of model pairs; the "error inconsistency" metric and why it isn't chance-corrected | Related work; the hierarchy design |
| 3 | **Geirhos et al., NeurIPS 2020**, error consistency (arXiv 2006.16736) | 40 min | The κ definition, the expected-overlap formula, and how they report κ against accuracy | Methods (metric) |
| 4 | **Zenk et al., MedIA 2024** (arXiv 2406.03323) | 60 min | The AURC / risk-coverage protocol; why they argue against AUROC-only reporting; the silent-failure definition; their "future work" sentence | Methods (QC evaluation), Related work |
| 5 | **RBQE**, arXiv 2609.10495 | 30 min | Their referee design (seed vs architecture vs MedSAM) and results; what they *don't* do (per object, κ) | Related work: "closest work, and how we differ" |
| 6 | **Cellpose-SAM** (bioRxiv 10.1101/2025.04.28.651001) | 40 min | Architecture; training data (LIVECell sampling 5%; NeurIPS22 616 images); the flow-error "QC step" | Models and data (leakage) |
| 7 | **micro-SAM** (Nature Methods 2025) + **Archit & Pape, MIDL 2026** (arXiv 2603.17845) | 45 min | AIS vs AMG; training data; that APG's IoU predictions are "a quality estimate for each predicted mask" but never validated | Models; QC signals |
| 8 | **NeurIPS22 CellSeg challenge** (arXiv 2308.05864) | 30 min | What Public-Test contains (50 labeled images, modalities); the license (CC BY-NC-ND) | Data |
| 9 | **Saxena et al., NeurIPS 2024** (arXiv 2404.01542) | 30 min | Why data-order-only seeds give little diversity | Discussion of the L0b seed results |
| 10 | **Kirscher et al. 2026** (arXiv 2605.18329) | 20 min | Seed vs CV-fold ensembles for failure detection | Related work |
| 11 | **Visagreement** Case Study 2 (you have the full text) | 15 min | The disagreement ↔ accuracy conjecture and "not comprehensive enough to assert" | Intro (the lab hook) |
| 12 | *Optional:* Klein 2025, Kim 2025, Comput. Biol. Med. 2023, arXiv 2512.15921, Bommasani 2022 | 60 min | One sentence each | Related work |

All are open access (arXiv, PMC or F1000). None needs a download from you.

---

## 4. What we have already measured

These are preliminary: feasibility spikes, not results. Scripts and outputs are on the cluster in `~/vis4ml_spikes/work/{a1,b1}/`; write-ups are in `spike_results/A1_microscopy_seg.md` and `B1_d5_hierarchy.md`.

### 4.1 Spike A1 (8 LIVECell test images, 2,006 GT cells)
- **Speed:** Cellpose-SAM 0.26 s per image; micro-SAM AIS 0.13 s. Both fit easily on one GPU (FACT).
- **F1@0.5:** Cellpose-SAM 0.86; micro-SAM AIS 0.78.
- **The two models fail differently.** Cellpose-SAM mostly *misses* cells (255 misses, 87 cells in merges). micro-SAM mostly *merges* them (407 cells in merges). That is exactly the structure a per-cell error taxonomy exposes (FACT).
- **Internal signals vs per-cell IoU** (Spearman ρ): Cellpose flow error −0.63; cell probability +0.58; micro-SAM foreground +0.44; AMG predicted IoU +0.41.
- **Cross-model agreement vs Cellpose-SAM's per-cell IoU:** ρ 0.64 pooled, 0.53 within image. So agreement is about as informative as Cellpose's own flow error, and part of it is image difficulty (SYNTHESIS).
- **Gotcha:** micro-SAM AMG with default thresholds returns **zero masks** on LIVECell. Its predicted-IoU head is badly calibrated there. That is a small finding worth a sentence (FACT).

### 4.2 Spike B1 (40 LIVECell test images, 10,292 GT cells; the relatedness hierarchy)
**Setup:**
- 3 seeds of Cellpose-SAM fine-tuned on 200 LIVECell train images (25 per cell type), 30 min each on one L40S.
- Plus off-the-shelf cyto3, `livecell_cp3`, micro-SAM AIS and CellSAM.
- Error = no prediction with IoU > 0.5.
- κ with 1,000× image-level bootstrap 95% CIs.

| Level | Pair | Accuracies | **κ [95% CI]** | κ/κ_max | Silent failures per 1,000 cells | Within-image AUROC of agreement |
|---|---|---|---|---|---|---|
| L0b seed | cpsam FT seeds (3 pairs) | 0.75 / 0.75 | **0.92** [0.90–0.93] | 0.93 | 166 | 0.71 |
| L1 checkpoint variant | cpsam vs its fine-tunes | 0.76 / 0.75 | **0.88** [0.86–0.90] | 0.90 | 150 | 0.72 |
| L4 same arch, different data | cyto3 vs livecell_cp3 | 0.74 / 0.75 | **0.84** [0.82–0.86] | 0.86 | 138 | 0.74 |
| L3 different encoder, same family | cpsam (or FT) vs cyto3 / livecell_cp3 | ~0.75 / 0.74–0.75 | **0.79** [0.75–0.83] | 0.79–0.83 | 117–120 | 0.72–0.77 |
| L3′ different encoder, different family | micro-SAM vs cyto3 / livecell_cp3 | 0.65 / 0.74–0.75 | **0.61** [0.56–0.64] | 0.77–0.80 | 74–75 | 0.76–0.77 |
| L2 shared SAM encoder, different family | cpsam (or FT) vs micro-SAM | 0.75 / 0.65 | **0.58** [0.52–0.62] | 0.76–0.78 | 67–70 | 0.75–0.78 (0.83–0.85 flagging micro-SAM's errors) |
| — | CellSAM vs anything | 0.28 / … | 0.20–0.29 | not meaningful | 10–22 | — |

**How to read it (SYNTHESIS):**
1. **The top of the hierarchy behaves as classification predicts.** Seeds are most alike (0.92), then checkpoint variants (0.88).
   - Even two seeds share about **166 silent failures per 1,000 cells**.
   - Seed diversity is small because only data order and augmentation vary, the Saxena effect.
2. **The shared SAM encoder does not make errors alike.**
   - Cellpose-SAM vs micro-SAM (both SAM): **0.58**. Cellpose-SAM vs cyto3 (SAM vs U-Net): **0.79**. The CIs don't overlap.
   - The difference survives the accuracy correction (κ/κ_max 0.77 vs 0.83).
   - **Being in the same model family predicts shared errors. The encoder doesn't.**
3. **But "family" is a bundle.** It includes the flow-field objective and decoding, the training data and recipe, and the group. B1 can't yet tell which of these drives the effect. That is your main scientific job (§5, H-decoder).
4. **The QC consequence has the predicted sign.** The less correlated the pair, the better agreement works as a QC signal. AUROC rises from 0.71 (seeds) to about 0.78, and to 0.83–0.85 when Cellpose-SAM's agreement flags micro-SAM's errors. Silent failures fall from 166 to 67–70 per 1,000.
5. **Data vs architecture (H-data) isn't supported.** The same architecture on different data (0.84) is *more* consistent than cross-architecture pairs. But both of those models saw LIVECell, so the pair isn't clean.

**Caveats you must state in the proposal:**
- in-distribution data (LIVECell);
- 40 images, 5 per cell type;
- seeds vary data order only (no from-scratch seeds yet);
- the off-the-shelf levels change several things at once.

### 4.3 Round 3: pre-registered kill tests, held-out replication (2026-09-30)
Full details are in `spike_results/C1–C5`. Pre-registration: `PREREG_D5.md`. Amendments 1–3 were committed before any Public-Test statistic was computed.
- **Held-out set:** NeurIPS22 Public-Test, 50 images and 6,040 cells across 5 modalities. Secondary: LC200, 194 LIVECell test images.
- **Confirmatory results:**
  - **H1 supported:** κ(Cellpose-SAM, cyto3) 0.33 > κ(Cellpose-SAM, micro-SAM) 0.17, also after κ/κ_max.
  - **H2 supported:** seeds 0.91–0.93.
  - **H3 supported:** ρ = −0.30.
  - **H4 inconclusive:** agreement ≈ Cellpose's own flow error; both ≫ attributes.
- **Kill tests:**

  | Test | Result |
  |---|---|
  | K1 | passed |
  | K2 (hard cells) | passed, with a weak difficulty model on Public-Test |
  | K3 (diameter) | passed |
  | **K4 (accuracy)** | **triggered**: no H1 effect on the 20 accuracy-matched images; κ/κ_max gap only +0.02 on LIVECell |
  | K5 | passed, but the effect sits in 2 of 5 modalities |
  | **K6 (practical)** | **triggered**: a same-family reference triages Cellpose-SAM's errors as well or better |

- **Surprises:**
  1. Out of distribution, cross-model κ collapses (0.78 → 0.33) while seeds stay at 0.92.
  2. LIVECell-only models fail together out of distribution (κ 0.73–0.84).
  3. **Cellpose-SAM is ViT-L; micro-SAM and CellSAM are ViT-B.** The ViT-B pair is the most consistent cross-family pair (0.39).
  4. `flow_threshold=0` raises Cellpose-SAM's LIVECell recall from 0.75 to 0.83.
  5. The LIVECell test json lists 52 files twice with different annotations.
- **Reviewer corrections (Mac session, 2026-09-30; details at the end of `C1_heldout_replication.md`):**
  - Public-Test is a held-out *split*, **not OOD**, for Cellpose-SAM and micro-SAM, which were trained on NeurIPS22 Training. Read "surprise 1" as LIVECell vs NeurIPS22, confounded with error rate (25% vs 5%). Clean OOD needs own-trained models (the leave-one-cell-type-out probe).
  - A margin-free odds ratio still favors family on both datasets: roughly 17 vs 6 on Public-Test and 95 vs 23 on LC200. So K4 is partly a metric-definition question. Report log-OR alongside κ.
  - One mechanism, reference *independence* traded off against reference *accuracy*, explains K6, the H3 reversal and H4. Suggested new headline question: **"What makes a good reference for agreement-based QC?"**
- **Implication for the proposal:** pre-register the accuracy-matched analysis as primary, alongside raw κ and κ/κ_max. Rename L2 "shared SAM pretraining". Add a same-checkpoint contrast (micro-SAM ViT-B vs ViT-L). Frame RQ3 as "agreement vs the model's own signals", not "agreement vs nothing".

---

## 5. Research questions and hypotheses

This is the current version, updated after B1. **Pre-register** these in the proposal: write them down before you run the held-out experiment, with the thresholds that count as support or falsification. Pre-registration is what makes a null result publishable.

**RQ1 (floor; grade-safe): error profile.** Per GT cell, what errors (miss / merge / split / FP / boundary) do 4–6 generalist models make, and which cell and image attributes (area, local density, touching fraction, contrast, cell type or modality) predict them?
- **H1:** error rates rise with density and touching fraction and fall with contrast, and the attribute effects *differ by model*. Test with a model × attribute interaction in a logistic GLMM, a regression with a random effect per image that absorbs per-image clustering.

**RQ2 (core novelty): error consistency along the relatedness hierarchy.** How does per-cell error consistency (κ) change across the levels L0–L4, and **which component of relatedness predicts shared errors?**
- **H-lineage** (B1 supports it in-distribution): same-family pairs (e.g. Cellpose-SAM vs cyto3) have higher κ than shared-encoder cross-family pairs (Cellpose-SAM vs micro-SAM).
  - Falsified on held-out data if the κ difference's bootstrap CI includes 0.
- **H-decoder** (new; SYNTHESIS): the **instance-decoding objective** drives shared *merge/split* errors, while *misses* are driven more by data. Tests:
  - (i) κ **by error type**;
  - (ii) micro-SAM **AIS vs AMG**, which share the same encoder and weights with different decoding. A1 gave κ = 0.31, but AMG was weak, so it needs tuned thresholds on non-test data;
  - (iii) Cellpose-SAM vs cyto3 (same decoding, different encoder; B1: 0.79).
- **H-mono** (from the novelty check): shared-SAM pairs behave like seed copies. **Rejected in-distribution by B1.** Re-test on held-out data; a replicated rejection is a headline result.
- **H-data > H-arch:** training data decorrelates errors more than architecture does. Not supported in B1, but that pair wasn't clean. Needs a cleaner data contrast (own-trained U-Nets on different subsets).

**RQ3 (the practical consequence): when can agreement be trusted as QC?** Does per-cell cross-model agreement rank a target model's errors better than:
- (a) model-internal signals (flow error with `flow_threshold=0`, cell probability, predicted IoU);
- (b) TTA self-consistency;
- (c) an attribute-only baseline?

Does its quality depend on the reference model's relatedness?
- **H-QC1:** agreement's within-image AUROC is ≥ 0.05 higher than the best internal signal, and higher than the attribute baseline.
  - **Falsified** if it is within 0.02 of the attribute-only baseline.
- **H-QC2:** across pairs, QC AUROC is *negatively* related to κ (B1: the predicted sign). So **cross-family references give better QC** than same-family ones.

**RQ4 (visual analytics + silent failures).**
- **H-triage:** a triage list ordered by the best QC signal finds ≥ 2× more errors than random ordering in the top 5% of cells, and beats confidence-only ordering (simulated inspection).
- **H-label (exploratory):** among sampled silent failures, a substantial share (≥ 20%) are **label errors** or ambiguous cells, not model errors. You adjudicate about 100 of them in the tool. This matters because the CellSAM authors themselves graded LIVECell test annotations as good, medium or poor.

---

## 6. Study design

### 6.1 Data

| Dataset | Role | Size / subset | License | Leakage status |
|---|---|---|---|---|
| **LIVECell test** (Edlund 2021) | Main in-distribution set; 8 cell lines, phase contrast | 40 images done; scale to 100–400 (stratified per cell type) | CC BY-NC 4.0 (fine for research) | In-distribution for cpsam, cyto3, micro-SAM, livecell_cp3; **zero-shot for CellSAM** |
| **NeurIPS22 CellSeg Public-Test** (Zenodo 10719375) | **The clean held-out set.** Must-have | 50 labeled images (mixed modalities). Range-read from the 2.9 GB `Testing.zip`; no full download needed | CC BY-NC-ND 4.0: analysis OK, **don't publish derived masks or overlays** | Held out for all models |
| NeurIPS22 Tuning | Second held-out set, with a caveat | 101 labeled images | CC BY-NC-ND 4.0 | Likely used as micro-SAM's validation set, so it is "model-selection-exposed" |
| ~~BBBC038 / DSB2018~~ | **Don't use for held-out claims** | — | CC0 | In every model's training data |

**Held-out cell-type probe (cheap, strong):** train your own seeds on 7 of LIVECell's 8 cell types and test on the 8th. That gives controlled distribution shift without new data.

### 6.2 Models and training data (the leakage map)

| Model | LIVECell train | NeurIPS22 Training | NeurIPS22 Tuning | NeurIPS22 Public-Test |
|---|---|---|---|---|
| Cellpose-SAM | trained (5% sampling) | trained (616 of 1,000) | held out | held out |
| cyto3 | trained | not listed | held out | held out |
| micro-SAM `vit_b_lm` | trained | trained | likely validation | held out |
| CellSAM (generalist) | **held out** | not in the generalist | — | held out |

(FACT, from each paper's full text; see `lit_notes_open/deep_5_microscopy_qc.md` §2.4.)

### 6.3 The hierarchy experiment
- **Done in B1:** L0b (3 cpsam fine-tune seeds), L1, L2, L3, L3′, a rough L4.
- **To add:**
  - **L0a** from-scratch seeds (cyto3-architecture U-Nets, about 1–2 GPU hours each, overnight);
  - a **cleaner L4** (your own U-Nets trained on different data subsets);
  - **micro-SAM AIS vs AMG** (H-decoder);
  - optionally `cpsam_v2` / `cpdino`.
- **Always** report κ, κ/κ_max, both accuracies, the silent-failure rate and the AUROC, per pair, **per dataset**, and **per error type**.

### 6.4 Pipeline (most of it exists)
1. **Inference per model per image.** Save masks, plus flows and probabilities for the QC signals. Run Cellpose with `flow_threshold=0` to get *unfiltered* flow errors. With the default 0.4 threshold, every surviving cell has error < 0.4, which deflates correlations (a "range restriction" artifact).
2. **Matching and the error taxonomy:** `match.py`, IoU > 0.5.
3. **Per-cell attributes:** area, diameter, eccentricity, local density, touching fraction, local contrast.
4. **Per-cell QC signals:** pairwise agreement (the max IoU with each reference model), leave-one-out mean agreement, internal signals, TTA consistency.
5. **Statistics:**
   - κ with image-bootstrap CIs;
   - within-image AUROC / AUPRC and AURC (reuse ~30 lines of Zenk's logic);
   - partial correlations controlling for attributes;
   - a GLMM for H1;
   - Holm correction across the pre-registered tests.
6. **Precompute everything to Parquet** so the visualization app never runs a model.

### 6.5 Optional reproduction (not required by the syllabus, but useful)
Reproduce one or two published numbers, e.g. Cellpose-SAM's LIVECell accuracy, or a micro-SAM or Cellpose-SAM number from Archit & Pape's MIDL 2026 benchmark. It validates your pipeline and gives you "prior work" to demo. Budget 1–3 hours.

---

## 7. The visual analytics part (and why it is scientifically necessary)

The course wants visualization that *does science*, not decoration. Each view below answers a specific question that a table cannot. Build it in **Streamlit + Plotly** over the precomputed Parquet files. Silva confirmed JS/D3 isn't required.

| View | What it shows | Why it is necessary |
|---|---|---|
| **1. Hierarchy view** (the headline figure) | κ (with CI) per model pair, grouped by relatedness level. Toggles for dataset and error type; κ/κ_max shown alongside | This is RQ2. Grouping by level makes departures from the classification ordering *visible* (e.g. the SAM–SAM pair sitting below the same-family pair) |
| **2. κ vs QC-AUROC scatter** | One dot per pair: x = κ, y = QC AUROC | This is H-QC2 in one picture: "the more alike two models are, the worse their agreement is as a quality check" |
| **3. Error overlay + adjudication** | The raw image with each model's errors colour-coded (miss / merge / split); click a cell to see its crop across all models and label it *model error / label error / ambiguous* | Needed for H-label. No metric can tell "all models wrong" from "ground truth wrong"; only looking can |
| **4. UpSet plot of failures** | Which *subsets* of models fail on each cell (an UpSet plot is a Venn-diagram alternative for many sets) | Shows the silent-failure mass (all fail) against unique failures |
| **5. Triage list + risk–coverage curve** | Cells ranked by a chosen QC signal, with live precision@K and AURC against random and confidence-only ranking | RQ3/RQ4. It is also the practical demo: "here's how a biologist would find errors" |
| 6. *(Optional)* attribute small multiples | Error rate vs density/contrast, one line per model | H1: shows *where* model curves cross, which one regression table hides |

**The VA contribution** isn't "a dashboard". It is **an adjudication and triage workflow validated by simulated inspection**. Say that explicitly, and don't oversell the UI.

---

## 8. Scope tiers, milestones and hours

**Principle:** build the **floor first**, so every later step is upside.

| Tier | What | Done by | Your hours (est.) |
|---|---|---|---|
| **Floor (A-grade safe)** | RQ1 error taxonomy + RQ2 κ hierarchy on LIVECell *and* NeurIPS22 Public-Test, with CIs; views 1, 2 and 3 | Nov 3 update → Dec 14 | ~25–30 |
| **Core** | + RQ3 QC comparison (internal signals, TTA, attribute baseline, AURC); view 5; 100-cell adjudication | Dec 14 | +8–10 |
| **Ambitious (paper)** | + L0a from-scratch seeds, clean L4, AIS vs AMG, the held-out cell-type probe, more held-out datasets, a small user study | Mostly Jan–spring 2027 with Silva | +18–20 |

**Milestones:**
- **Oct 6:** pitch to Silva (paragraph in §10).
- **Oct 7–19:** read papers 1–8 (§3.3); write the proposal (§11).
- **Oct 20:** **proposal due** (4 pages).
- **Oct 20–Nov 2:** get NeurIPS22 Public-Test in; rerun B1's κ table on it; add the error-type split.
- **Nov 3:** **1-page update.** Headline: "Does the lineage result replicate on held-out data?"
- **Nov 3–25:** QC comparison, views 1–5, adjudication.
- **Dec 1 / Dec 8:** presentation.
- **Dec 14:** **final 8-page report** in IEEE VIS format.

About 3–4 h/week × 11 weeks ≈ 35–45 hours, enough for Floor + Core. GPU work runs overnight, so compute isn't the constraint.

---

## 9. Risks and mitigations

> **Pre-mortem and kill tests:** `PREREG_D5.md` §4 lists seven concrete ways D5 could fail later (K1–K7), each with the test that checks it *now* and the rule for when to drop or reframe the idea. The three oversights the pilot had not yet tested:
> - **K2:** shared errors might just be *hard or mislabeled cells*. Tested with a difficulty-controlled κ.
> - **K3:** the family effect might come from *shared preprocessing* (cyto3's auto-diameter).
> - **K4/K5:** accuracy-gap and threshold artifacts.
>
> Round 3 on the cluster (`NEXT_SESSION_TASKS_3.md`) runs all of these plus the held-out replication.


| Risk | How likely | Mitigation |
|---|---|---|
| **The lineage result doesn't replicate on held-out data** | Medium | It is still a finding either way: "in-distribution vs held-out error consistency differs". Pre-register so a null is reportable |
| **Agreement just encodes image difficulty** | Medium | Within-image AUROC; the attribute-only baseline; partial correlations. If agreement ≈ baseline, that is a clean negative result about BISCUIT-style QC |
| **"Lineage" stays a bundle** (can't isolate the cause) | High for off-the-shelf models | Own-trained controlled pairs (L0a, clean L4), AIS vs AMG, and the error-type split. Be honest in the paper that off-the-shelf levels change several things at once |
| **Accuracy differences distort κ** | Certain | Always report κ/κ_max and accuracies; add an accuracy-matched subset analysis |
| **Label noise in LIVECell** | Medium | The adjudication view (H-label); report the share of label errors |
| **Scoop** (the DKFZ, Pape or RBQE groups are close) | Low–Medium | Post a preprint soon after the course; re-check arXiv before Nov 3 ("error consistency segmentation", citers of RBQE and Gontijo-Lopes) |
| **Environment friction** (Cellpose 3 vs 4, micro-SAM pins) | Low now | Solved in the spikes: the `d5` env plus the `d5c3` overlay on the cluster |
| **License** (NeurIPS22 is ND) | Low | Use it for evaluation; don't publish its derived masks or overlays in the public tool |

---

## 10. Course fit, the Silva connection, and the Oct 6 pitch

**Course fit (lectures):**
- model assessment and performance metrics (κ, AUROC, AURC, calibration of QC signals);
- black-box interpretation (Oct 6; agreement as a black-box reliability signal);
- DL visualization (Oct 27);
- interpretable ML (Nov 24).

The visualization is the instrument for finding and adjudicating errors, not an add-on.

**The Silva connection (use it):**
- Visagreement's Case Study 2 *conjectures* that explanation-method disagreement goes with model error, calling it "not comprehensive enough to assert". D5 tests the same logic ("does disagreement signal error, and when does it fail?") **rigorously, per instance, against ground truth, in a new modality.**
- Calibrate (also his lab) is about trusting confidence scores. D5 asks when *agreement* scores can be trusted.

**Draft Oct 6 pitch** (about 45 seconds; adapt it into your own words):

> *"Biologists increasingly segment cells with generalist models like Cellpose-SAM and micro-SAM, and without ground truth they check results by seeing where several models agree. Tools like BISCUIT assume those models make independent errors, but a reviewer asked whether that holds for models sharing architectures or training data, and it was never answered. I want to measure, per cell, how often generalist segmenters fail together, and what about the models predicts it: random seed, fine-tuning, a shared SAM encoder, the decoding objective, or the training data. Then I want to test when that makes agreement-based quality control untrustworthy. I've run a pilot on 40 LIVECell images: surprisingly, two models sharing a SAM encoder agree on errors much less (κ 0.58) than two Cellpose-family models with different encoders (κ 0.79). And agreement works better as a quality check the less related the models are. The next step is replicating this on held-out data, and building a visual tool to triage and adjudicate the 'silent failures' where all models agree but are wrong. It's in the spirit of Visagreement's disagreement–error conjecture, tested against ground truth."*

**Questions to ask him:**
- Is this topic close enough to the class?
- Is building on Visagreement's framing welcome?
- Is the lab's planned image/text extension of Visagreement in progress? (It matters less for D5 than for D4.)
- Can you confirm solo status?
- **Would he advise a paper extension after Dec 14** if the result is strong? (You're targeting Fall 2028 PhD applications.)

---

## 11. Proposal blueprint (4 pages, section by section)

A suggested structure and page budget. It follows a conference-proposal shape, which suits the IEEE VIS final format.

**Working title:** *"When Do Cell Segmenters Fail Together? Model Relatedness, Per-Cell Error Consistency, and the Limits of Agreement-Based Quality Control."*

| Section | Pages | What to write | Draw from |
|---|---|---|---|
| **1. Introduction and motivation** | 0.5 | Generalist segmenters; no GT in practice; agreement-based QC (BISCUIT) and its independence assumption; Bankhead's unanswered question; silent failures; your RQ in one sentence; 3 bullet contributions | §1, §3.1; reading #1, #11 |
| **2. Related work** | 0.75 | Four paragraphs: (i) error consistency and diversity in classification (Gontijo-Lopes, Geirhos, Saxena, Klein, Kim); (ii) segmentation failure detection (Zenk, Kirscher, RBQE: "closest work; we differ in per-object κ, 5 levels, microscopy"); (iii) agreement-based bio-image QC (BISCUIT, SEG, MARC); (iv) generalist models and their internal quality signals (Cellpose-SAM, micro-SAM, CellSAM, Archit & Pape). End with the gap statement | §3; the check file's §4 |
| **3. Research questions and pre-registered hypotheses** | 0.5 | RQ1–RQ4 with H-lineage, H-decoder, H-QC1, H-QC2, H-triage, H-label. Give the thresholds (e.g. "falsified if the CI includes 0"; "within 0.02 of the attribute baseline") | §5 |
| **4. Data and models** | 0.4 | LIVECell test (in-distribution) + NeurIPS22 Public-Test (held out) + the cell-type probe; the leakage table; models and their lineage; licenses | §6.1, §6.2 |
| **5. Methods** | 0.6 | Matching and the error taxonomy; the per-cell error indicator; κ, κ/κ_max, image bootstrap; QC signals; within-image AUROC, AURC, simulated inspection; GLMM; the multiple-comparison plan | §2, §6.3–6.4 |
| **6. Visual analytics design** | 0.5 | Views 1–5 with one sentence each on the question it answers; a small sketch or screenshot (you can mock one from the B1 data) | §7 |
| **7. Preliminary results** | 0.35 | The B1 table (trimmed) and 2–3 sentences: the SAM-encoder surprise, the κ–AUROC direction, and the in-distribution caveat | §4.2 |
| **8. Timeline, risks, deliverables** | 0.4 | The milestone table; the top 3 risks and mitigations; the scope tiers (floor / core / ambitious) | §8, §9 |
| References | (extra) | About 15–20 | §3.3 |

**Tips:**
- **Lead with the question, not the tool.** Graders and reviewers reward a sharp, falsifiable question.
- **Put the preliminary result in.** Few proposals have one. It shows feasibility and a non-obvious finding.
- **State the caveats yourself** (in-distribution, bundled lineage) before a reader does.
- **Make one clear, labelled figure** for the proposal: κ by level, from B1's `kappa_by_level.csv`.
- Add an **AI-use disclosure** (the course requires it).

---

## 12. Open decisions, things to verify, and where everything lives

**Decisions for you:**
1. **The primary "target" model for QC** (the one whose errors you rank). Cellpose-SAM is the natural choice as the most accurate.
2. **How many LIVECell images** for the final run: 100–200 stratified is plenty.
3. **Include CellSAM?** It is nearly uninformative at default settings (72% error). Either keep it as an "incompetent model" contrast or drop it.
4. **Train from-scratch seeds (L0a)?** Needed for a clean paper; optional for the course.

**Verify yourself (quick):**
- Read BISCUIT's open reviews and confirm Bankhead's comment is still unanswered.
- Re-check arXiv for "error consistency" + segmentation, and for new citers of RBQE and Gontijo-Lopes, before Oct 20 and again before Nov 3.
- Confirm the NeurIPS22 Public-Test modalities and whether per-image modality labels exist.
- Confirm cyto3's training data includes LIVECell. It is secondhand so far, and it affects the L4 interpretation.

**Where everything lives:**

| What | Where |
|---|---|
| This onboarding doc | `D5_onboarding.md` |
| **Pre-registration + kill criteria** (hypotheses fixed before the held-out run) | `PREREG_D5.md` |
| Round-3 cluster brief (kill tests, held-out replication, pitch figure) | `NEXT_SESSION_TASKS_3.md` |
| The D5 deep dive (full paper table, leakage quotes, detailed design, hour budget) | `lit_notes_open/deep_5_microscopy_qc.md` |
| Novelty check for the hierarchy framing (must-cites, experiment sketch) | `lit_notes_open/check_d5_error_hierarchy.md` |
| Spike write-ups | `spike_results/A1_microscopy_seg.md`, `spike_results/B1_d5_hierarchy.md` |
| Summary and rankings across all candidates | `research_gap_review.md` (short); `research_gap_details.md` (full) |
| Code, envs, models, outputs | Cluster: `~/vis4ml_spikes/` (`work/a1`, `work/b1`; envs `d5` and the `d5c3` overlay; fine-tuned seeds in `work/b1/models/`) |

*AI-use note: this onboarding doc was produced with Claude from the project's research notes and spike results. The research design choices are yours to make and defend.*
