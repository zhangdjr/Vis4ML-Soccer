# D5 Onboarding: "When Does Agreement Between Cell Segmenters Reveal Their Errors?"

**Purpose.** This document takes you from "I skimmed a lot of topics" to "I can write a strong 4-page proposal for D5 by Oct 20." Read it top to bottom once, which takes about 60–75 minutes, then work through the reading list in Part 3.
**Written:** 2026-09-30 from the D5 deep dive, the novelty check and two cluster spikes (A1, B1). **Rewritten the same day after rounds 3 and 4** (pre-registered held-out tests). Every number comes from those files; §12 lists the sources.
> **What changed in the rewrite.** The original headline, "same family shares errors, a shared SAM encoder doesn't", **did not survive** held-out testing. Two later headlines didn't either (§4.3–4.4). The project is now framed around a **visual-analytics tool plus a pre-registered evaluation of when agreement-based QC works**, built on the results that *did* replicate. §1, §5, §7, §8, §10 and §11 are new. §2–§4 are kept as background and history.
**Labels:** **FACT** = checked against a source or our own run. **SYNTHESIS** = my inference. **SPECULATION** = a guess to test.

---

## Contents
1. [The project on one page](#1-the-project-on-one-page)
2. [Concepts you need (primer)](#2-concepts-you-need-primer)
3. [The literature: what exists, what's open, what to read](#3-the-literature-what-exists-whats-open-what-to-read)
4. [What we have already measured](#4-what-we-have-already-measured)
5. [Research questions and hypotheses](#5-research-questions-and-hypotheses)
6. [Study design](#6-study-design)
7. [The visual analytics part (now the centre of the project)](#7-the-visual-analytics-part-now-the-centre-of-the-project)
8. [Scope tiers, milestones and hours](#8-scope-tiers-milestones-and-hours)
9. [Risks and mitigations](#9-risks-and-mitigations)
10. [Course fit, the Silva connection, and the Oct 6 pitch](#10-course-fit-the-silva-connection-and-the-oct-6-pitch)
11. [Proposal blueprint (4 pages, section by section)](#11-proposal-blueprint-4-pages-section-by-section)
12. [Open decisions, things to verify, and where everything lives](#12-open-decisions-things-to-verify-and-where-everything-lives)

---

## 1. The project on one page

**The setting.** Biologists increasingly segment microscopy images with **generalist cell-segmentation models**, pretrained models meant to work on any cell image without retraining. The main ones are Cellpose-SAM, micro-SAM, CellSAM, and older Cellpose models such as cyto3. Nobody has ground truth for their own images, so they need a way to tell **which cells were segmented wrong** without it.

**The common trick.** Run several models and trust the cells where they **agree**. Tools like **BISCUIT** (F1000Research 2025) are built on this. BISCUIT states the assumption outright: *"Assuming that model prediction inaccuracies are uncorrelated between models, the model with the lowest score yields predictions closest to the ground truth"* (FACT, full text). A peer reviewer, Peter Bankhead, doubted this holds "whenever comparisons are made between overlapping methods, trained on overlapping training sets". **The authors never answered** (FACT, still unanswered in the July 2026 version).

**Your research question, in one sentence:**
> *When does per-cell agreement between generalist cell segmenters reveal their errors, and when does it fail, compared with the model's own confidence, and depending on which reference models you compare against?*

**What you build:** a **visual-analytics (VA) tool** that loads several models' segmentations of the same images and shows:
- where the models disagree, per cell;
- each model's own confidence signal;
- a ranked triage list of cells to inspect;
- an adjudication view for the cells where every model agrees but may be wrong ("silent failures").

**What you evaluate**, against ground truth, with hypotheses pre-registered before the final run: how well each ranking finds real errors, and when it breaks.

**What we already know** (4 spike rounds, 4 datasets, details in §4). These are the parts that **replicated**:
1. **Copies of a model fail on the same cells.** Seeds and fine-tunes have near-identical errors on every dataset (κ ≈ 0.9; log odds ratio 5–8). Agreement between them is nearly useless as a QC signal.
2. **Agreement between *different* models is a strong QC signal, but only about as good as the model's own confidence.** Cellpose-SAM's flow error matches cross-model agreement on 4 datasets (AUROC about 0.80–0.90 for both). Combining them adds little (+0.002 to +0.013).
3. **Averaging several references beats the best single one** (pre-registered, replicated): +0.03 to +0.05 AUROC.
4. **"Related models fail together" depends on data.** Cellpose-SAM and cyto3 share more errors than Cellpose-SAM and micro-SAM on LIVECell and NeurIPS22 (data both saw related versions of). On a **brand-new** dataset (mCellSeg), the effect disappears.
5. **Measurement pitfalls that change conclusions**, each with a concrete demonstration (§2, §4.4): κ vs odds ratio under different error rates; recall-at-budget ceilings; held-out sets outside a model's size range.

**Why this is worth doing.**
- It gives an **evidence-based answer to Bankhead's question** and a practical recommendation to BISCUIT-style tools: compare against several *unrelated* models, show the model's own confidence alongside agreement, and don't read agreement as correctness for model copies.
- It ties to your instructor's lab. Visagreement (Silva lab, TVCG 2025) *conjectured* that disagreement signals error. You test the analogous claim **per instance, against ground truth**, and build the tool that lets a user act on it.
- The VA part is not decoration: no metric can tell "all models wrong" from "ground truth wrong". Only looking can (§7).

**What it is *not* any more:** a causal claim about *which* model component (encoder, decoder, data) causes shared failures. Three such headlines failed on new data. That history is itself useful: it is the honest, pre-registered record your proposal can point to.

**Your role as the researcher.** You design the hypotheses, the views and the evaluation. The spikes proved the pipeline works and told you where the solid ground is.

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

**Log odds ratio (log-OR) of joint error.** A second dependence measure that, unlike κ, does **not** depend on the two models' error rates. It compares the odds that model B is wrong when A is wrong against the odds that B is wrong when A is right. 0 means independent errors; larger means more shared. **Why you need both:** in round 4, on one dataset κ halved (0.52 → 0.25) while log-OR *rose* (2.94 → 3.48), purely because one model's error rate changed. κ alone would have given the wrong conclusion.

**Recall at an inspection budget, and its ceiling.** "If a biologist checks the top 5% of flagged cells, what share of all errors do they find?" If the model is wrong on 28% of cells, even a perfect ranking finds at most 5/28 ≈ 18% of errors in the top 5%. So always draw the **ceiling** (budget ÷ error rate). Round 4's main confirmatory test came out inconclusive partly because every reference sat at this ceiling.

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
| 6 | **Cellpose-SAM** (bioRxiv 10.1101/2025.04.28.651001) | 40 min | Architecture; training data (LIVECell sampling 5%; NeurIPS22 **504** images, corrected in round 4); the flow-error "QC step" | Models and data (leakage) |
| 7 | **micro-SAM** (Nature Methods 2025) + **Archit & Pape, MIDL 2026** (arXiv 2603.17845) | 45 min | AIS vs AMG; training data; that APG's IoU predictions are "a quality estimate for each predicted mask" but never validated | Models; QC signals |
| 8 | **NeurIPS22 CellSeg challenge** (arXiv 2308.05864) | 30 min | What Public-Test contains (50 labeled images, modalities); the license (CC BY-NC-ND) | Data |
| 9 | **Saxena et al., NeurIPS 2024** (arXiv 2404.01542) | 30 min | Why data-order-only seeds give little diversity | Discussion of the L0b seed results |
| 10 | **Kirscher et al. 2026** (arXiv 2605.18329) | 20 min | Seed vs CV-fold ensembles for failure detection | Related work |
| 11 | **Visagreement** Case Study 2 (you have the full text) | 15 min | The disagreement ↔ accuracy conjecture and "not comprehensive enough to assert" | Intro (the lab hook) |
| 12 | *Optional:* Klein 2025, Kim 2025, Comput. Biol. Med. 2023, arXiv 2512.15921, Bommasani 2022 | 60 min | One sentence each | Related work |

All are open access (arXiv, PMC or F1000). None needs a download from you.

---

## 4. What we have already measured

**Read this section as history.** §4.1–4.2 are the pilots that motivated the original headline; §4.3–4.4 are the pre-registered tests that retired it. The take-aways that survived are listed in §1. Scripts and outputs are on the cluster in `~/vis4ml_spikes/work/{a1,b1}/`; write-ups are in `spike_results/A1_microscopy_seg.md` and `B1_d5_hierarchy.md`.

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

### 4.4 Round 4: "what makes a good reference?" (2026-09-30)
Full details are in `spike_results/D0–D6`. Pre-registration: `PREREG_D5.md` Part B, with Amendments 5–7 (all dated, "before"/"after" labelled).
- **Data:**
  - **N1 = mCellSeg** (new, May 2026, CC BY): 198 DIC/bright-field images, 15,975 whole cells, in no model's training data.
  - **N2:** 200 fresh LIVECell test images.
  - **N3:** 3 LIVECell leave-one-cell-type-out folds with 18 own-trained models.
- **Confirmatory results: nothing supported.**
  - **R1–R3 were untestable on native-scale N1.** Its cells are 50–327 px, outside Cellpose's training range, so Cellpose-SAM missed 65% of cells and failed the exclusion rule.
  - On an outcome-blind rescaled re-run:
    - **R1 (reference accuracy + independence beat κ for top-5% triage) is inconclusive.**
    - **R2 (the family effect) is falsified.**
    - **R3 (shared ViT-L checkpoint) is still untestable.**
  - **R4 (shift separates architectures more than seeds) is falsified:** seeds diverge more.
- **Robust secondary result.** Reference accuracy and independence (f, o) predict how well agreement *ranks* errors (AUROC) far better than κ, on all four datasets. Top-5% triage saturates once the target's error rate is ≫ 5%.
- **Exploratory lead for the proposal.** The Cellpose recipe with a DINOv3 backbone fails on nearly the same cells as Cellpose-SAM (log-OR 4.6–5.0, close to v1↔v2), while different recipes sit at 2.6–3.2. **Shared training data and recipe, not the foundation backbone, predicts shared failures.** Pre-register this.
- **Corrections:**
  - Round 3's "Cellpose-SAM" was **v2** (the cellpose 4.2.1.1 default). Round 4 added v1.
  - Cellpose-SAM used **504** NeurIPS22 training images, not 616 (616 is LynSec).
  - The micro-SAM weights that micro_sam 1.8.x downloads (v4) use NeurIPS22 Tuning as validation.
- **Reviewer note (Mac session; full text at the end of `D6_verdict.md`):**
  - The "robust" {f, o}-predicts-AUROC result is nearly definitional, since binary AUROC ≈ (2 − o − f)/2 (Spearman 0.58–0.94 with the real AUROC). Do not headline it.
  - Three rounds in a row produced an exploratory headline that failed on new data, so the DINO "recipe > backbone" lead gets a low prior.
  - **Recommendation:** stop chasing a headline. Frame the project as a VA tool + pre-registered evaluation of when agreement-based QC works.
- **Lesson.** A held-out set needs a GT-only scale check, e.g. median cell diameter inside the models' training range, as an inclusion criterion.

---

## 5. Research questions and hypotheses

**Rewritten after round 4.** Each hypothesis below either (a) already replicated in the spikes, so you are confirming it on a *new* held-out set, or (b) is a descriptive question with no pass/fail claim. Nothing here depends on a single striking contrast. Pre-register them in the proposal, with thresholds.

**RQ1 (floor; grade-safe): what errors do generalist segmenters make, and where do they overlap?**
- Per GT cell, the error type (miss / merge / split / boundary) for 5–7 models, and the **overlap structure**: which subsets of models fail on each cell (UpSet view, §7).
- Error dependence for every pair, reported as **κ, κ/κ_max and log odds ratio** with image-bootstrap CIs.
- **H1 (replication):** seed and fine-tune copies have higher error dependence than every cross-model pair (log-OR CI excludes 0). It held on 4 datasets; low risk.
- *Descriptive (no claim):* which cell attributes (size, density, touching, contrast) go with errors, per model. Round 3 found these attributes weak on held-out data (within-image AUROC 0.50–0.57). That is itself worth reporting.

**RQ2 (core): how well does agreement reveal errors, compared with the model's own confidence?**
- Signals, per cell, for a target model (Cellpose-SAM):
  - (a) agreement with each single reference model;
  - (b) mean agreement over several references;
  - (c) the model's own signal (flow error, with `flow_threshold=0`; cell probability);
  - (d) a combination of (b) and (c);
  - (e) an attribute-only baseline.
- Metrics: within-image AUROC, AURC (risk–coverage), recall of errors at fixed inspection budgets (1%, 5%, 10%), always shown **with the ceiling** (budget ÷ error rate).
- **H2 (replication):** mean agreement over ≥ 2 unrelated references beats the best single reference (AUROC Δ > 0, CI excludes 0). Supported on N2 and N1.
- **H3 (replication):** agreement with a copy of the target (seed / fine-tune) is worse than agreement with any cross-model reference.
- **H4 (two-sided, the honest question):** does agreement add to the model's own signal? Pre-register it as an *equivalence* test: "the combination beats flow error by < 0.02 AUROC" (the spikes' result) vs "≥ 0.02". Either outcome is a finding for BISCUIT-style tools.

**RQ3 (generalization): does the picture hold on new data and other targets?**
- Repeat RQ2 with each model as the target, not only Cellpose-SAM.
- Repeat on a **new, scale-checked held-out dataset** (§6.1), chosen and frozen before any model runs on it.
- *Descriptive:* where the family effect appears and where it doesn't (LIVECell, NeurIPS22, mCellSeg). Report it as "depends on shared training data", not as a general law.

**RQ4 (visual analytics): silent failures and adjudication.**
- **H5 (triage):** in simulated inspection, a triage list ordered by the best signal finds ≥ 2× more errors than random ordering in the top 5% of cells (well below ceiling).
- **H-label (exploratory):** among about 100 adjudicated silent failures (all models agree, GT disagrees), what share are **ground-truth label errors** or ambiguous cells? Report it with a Wilson CI. This matters: round 3 found the LIVECell test file lists 52 images twice with *different* annotations.

**What to leave out** (tested, and it didn't hold up; mention it in one paragraph as the pre-registered history):
- "Same family > shared SAM encoder" (falsified on mCellSeg);
- "Reference accuracy + independence predicts triage better than κ" (inconclusive; the AUROC version is close to true by definition);
- "Shared SAM checkpoint raises dependence" (untestable, exploratory data point the other way);
- "Distribution shift separates architectures more than seeds" (falsified);
- the CellposeDINO "recipe > backbone" lead (exploratory, low prior).

---

## 6. Study design

### 6.1 Data

| Dataset | Role | Size / subset | License | Leakage status |
|---|---|---|---|---|
| **LIVECell test** (Edlund 2021) | Main in-distribution set; 8 cell lines, phase contrast | 40 images done; scale to 100–400 (stratified per cell type) | CC BY-NC 4.0 (fine for research) | In-distribution for cpsam, cyto3, micro-SAM, livecell_cp3; **zero-shot for CellSAM** |
| **NeurIPS22 CellSeg Public-Test** (Zenodo 10719375) | **The clean held-out set.** Must-have | 50 labeled images (mixed modalities). Range-read from the 2.9 GB `Testing.zip`; no full download needed | CC BY-NC-ND 4.0: analysis OK, **don't publish derived masks or overlays** | Held out for all models |
| NeurIPS22 Tuning | Second held-out set, with a caveat | 101 labeled images | CC BY-NC-ND 4.0 | Likely used as micro-SAM's validation set, so it is "model-selection-exposed" |
| ~~BBBC038 / DSB2018~~ | **Don't use for held-out claims** | — | CC0 | In every model's training data |
| **mCellSeg** (Zenodo 20174259, May 2026) | Used in round 4 (N1). Genuinely new to every model | 198 images, 15,975 cells, DIC/bright-field | CC BY 4.0 | Held out for all, but **cells are 50–327 px**, outside Cellpose's 7.5–120 px range. Needs rescaling, and its round-4 results are now "seen" |
| **A new held-out set (to choose)** | The confirmatory set for the final evaluation | ≥ 1,500 cells | must allow analysis | **Scale check first:** median GT cell diameter inside every model's documented range, checked from GT alone before any model runs |

**Held-out cell-type probe:** done in round 4 (3 folds, 18 own-trained models). It falsified R4 and is not needed for the course project.

### 6.2 Models and training data (the leakage map)

| Model | LIVECell train | NeurIPS22 Training | NeurIPS22 Tuning | NeurIPS22 Public-Test |
|---|---|---|---|---|
| Cellpose-SAM | trained (5% sampling) | trained (**504** of 1,000; corrected in round 4, 616 was LynSec) | held out | held out |
| cyto3 | trained | not listed | held out | held out |
| micro-SAM `vit_b_lm` | trained | trained | likely validation | held out |
| CellSAM (generalist) | **held out** | not in the generalist | — | held out |

(FACT, from each paper's full text; see `lit_notes_open/deep_5_microscopy_qc.md` §2.4. Round 4: the installed "Cellpose-SAM" is **v2** (June 2026, training data undocumented); micro-SAM v4 weights use NeurIPS22 Tuning as validation; see `spike_results/D0_n1_choice.md`.)

### 6.3 Model roster (final)
- **Target:** Cellpose-SAM (v2, the cellpose 4.2.1.1 default). Repeat with the other models as targets for RQ3.
- **References:** cyto3, micro-SAM `vit_b_lm` (AIS), micro-SAM `vit_l_lm`, CellSAM (only where its error rate is ≤ 0.6), Cellpose-SAM v1.
- **Copies** (for H1/H3): 3 Cellpose-SAM fine-tune seeds, already trained in B1.
- **Drop:** `livecell_cp3` (a LIVECell specialist that fails elsewhere; useful only as an "unrelated but bad" contrast), micro-SAM AMG (weak).
- **Always** report each model's accuracy next to any dependence or QC number, because both depend on it.

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

## 7. The visual analytics part (now the centre of the project)

The course wants visualization that *does science*, not decoration. The tool is the main deliverable, and each view answers a question a table cannot. Build it in **Streamlit + Plotly** over precomputed Parquet files (no model runs inside the app). Silva confirmed JS/D3 isn't required.

| View | What it shows | Why it is necessary |
|---|---|---|
| **1. Error overlay + adjudication** (the core view) | The raw image with each model's outline and error colour-coded; click a cell to see its crop across all models, the agreement scores and the model's own confidence; label it *model error / label error / ambiguous* | Needed for H-label. No metric can tell "all models wrong" from "ground truth wrong" |
| **2. Triage list + budget curve** | Cells ranked by a chosen signal (single reference, mean of references, own confidence, combination). Live recall at a budget, **with the ceiling line drawn**, and the risk–coverage curve | RQ2/RQ4, and the practical demo: "here's how a biologist would find errors". Drawing the ceiling prevents the misreading round 4 found |
| **3. UpSet plot of failures** | Which subsets of models fail on each cell; click a bar to load those cells in view 1 | Shows the silent-failure mass (all fail) against model-specific failures |
| **4. Reference chooser** | For the target model, one dot per reference: x = how often the reference is wrong where the target is right (false alarms), y = the share of the target's errors it shares; colour = AUROC | Lets a user see *why* one reference flags errors better than another. Explanatory, not a claimed finding (see §4.4 reviewer note) |
| **5. Dependence matrix** | κ and log-OR for every pair, toggled; per dataset | RQ1. Showing both makes the κ-vs-log-OR pitfall visible |

**The VA contribution** is **a triage and adjudication workflow, validated by simulated inspection, that shows agreement and the model's own confidence side by side.** Say that explicitly, and don't oversell the UI.

**Optional (core tier):** a short think-aloud with 1–2 people who segment cells (e.g. a lab-mate or a biology student). Even 2 sessions give qualitative evidence for the VA section.

---

## 8. Scope tiers, milestones and hours

**Principle:** build the **floor first**, so every later step is upside. Most of the computation already exists on the cluster; your hours go into the tool and the writing.

| Tier | What | Done by | Your hours (est.) |
|---|---|---|---|
| **Floor (A-grade safe)** | RQ1 + RQ2 on LIVECell and NeurIPS22 Public-Test (numbers already computed); views 1, 2, 3 | Nov 3 update → Dec 14 | ~22–28 |
| **Core** | + RQ3 on a new scale-checked held-out set (pre-registered); views 4–5; 100-cell adjudication (H-label) | Dec 14 | +8–10 |
| **Ambitious (paper)** | + 1–2 more held-out datasets; more targets; a small think-aloud study; release the tool | Jan–spring 2027 with Silva | +15–20 |

**Milestones:**
- **Oct 6:** pitch to Silva (paragraph in §10).
- **Oct 7–19:** read papers 1–8 (§3.3); write the proposal (§11). Pre-register H1–H5 in it.
- **Oct 20:** **proposal due** (4 pages).
- **Oct 20–Nov 2:** export the existing per-cell tables to Parquet; build views 1 and 2.
- **Nov 3:** **1-page update.** Headline: "the tool works on LIVECell + NeurIPS22; H2/H4 results".
- **Nov 3–25:** choose and freeze the new held-out set (scale check first), run RQ3, build views 3–5, adjudicate about 100 silent failures.
- **Dec 1 / Dec 8:** presentation (live demo of views 1–2).
- **Dec 14:** **final 8-page report** in IEEE VIS format.

About 3–4 h/week × 11 weeks ≈ 35–45 hours, enough for Floor + Core.

---

## 9. Risks and mitigations

> **History.** `PREREG_D5.md` Parts A and B record two pre-registered rounds with kill tests. They caught the original headline (round 3) and its replacement (round 4) before you invested time. Keep pre-registering: write the final hypotheses and the new held-out set into the proposal *before* running anything on that set.

| Risk | How likely | Mitigation |
|---|---|---|
| **"Agreement ≈ own confidence" reads as a negative result** | Medium | Frame it as the answer to a practical question (should BISCUIT-style tools show model confidence too? Yes). Negative results about a widely used assumption are publishable, especially pre-registered |
| **Forking paths** (finding a new "interesting contrast" in the data and chasing it) | High, based on rounds 2–4 | Only pre-registered hypotheses count as findings. Anything new goes in an "exploratory" section, never the headline |
| **No clean held-out set** | Medium | Scale-check candidates from GT first. Fallback: report LIVECell + NeurIPS22 Public-Test + rescaled mCellSeg, labelled by how "held out" each is |
| **Label noise** (GT itself wrong) | Medium | The adjudication view (H-label); report the share of label errors |
| **The tool looks like "a dashboard"** | Medium | Tie every view to a research question (§7) and evaluate with simulated inspection; add a 1–2 person think-aloud if time allows |
| **Measurement pitfalls** | Known | Report κ *and* log-OR, budget curves *with ceilings*, accuracies next to every number |
| **Scoop** (RBQE group, Pape lab) | Low | Manual Google Scholar "Cited by" check before Oct 20 and Nov 3 (`spike_results/K7_scoop_check.md`) |
| **License** (NeurIPS22 is ND) | Low | Evaluate on it, but never show its pixels or masks in the public tool; use LIVECell or mCellSeg images for the demo |

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

> *"Biologists increasingly segment cells with generalist models like Cellpose-SAM and micro-SAM, and without ground truth they check results by seeing where several models agree. Tools like BISCUIT assume those models make independent errors. A reviewer asked whether that holds for related models, and it was never answered. I want to build a visual-analytics tool that shows, per cell, where segmenters disagree, how confident each model is, and which cells a biologist should check first, and then evaluate against ground truth when agreement actually reveals errors. I've already run pre-registered pilots on four datasets. Three things held up: copies of a model fail on the same cells, so agreement between them is useless; averaging several unrelated references beats any single one; and, surprisingly, cross-model agreement is only about as good as the model's own confidence signal. One thing didn't hold up: whether 'related' models share errors depends on their training data, not on a general rule. The tool would let users see all of this per cell, and adjudicate the 'silent failures' where every model agrees but may be wrong. It's in the spirit of Visagreement's disagreement–error conjecture, tested against ground truth."*

**Questions to ask him:**
- Is this topic close enough to the class?
- Is building on Visagreement's framing welcome?
- Is the lab's planned image/text extension of Visagreement in progress? (It matters less for D5 than for D4.)
- Can you confirm solo status?
- **Would he advise a paper extension after Dec 14** if the result is strong? (You're targeting Fall 2028 PhD applications.)

---

## 11. Proposal blueprint (4 pages, section by section)

A suggested structure and page budget. It follows a conference-proposal shape, which suits the IEEE VIS final format.

**Working title:** *"When Does Agreement Reveal Segmentation Errors? A Visual Analytics Tool and a Pre-Registered Evaluation of Ground-Truth-Free QC for Cell Segmentation."*

| Section | Pages | What to write | Draw from |
|---|---|---|---|
| **1. Introduction and motivation** | 0.5 | Generalist segmenters; no GT in practice; agreement-based QC (BISCUIT) and its independence assumption; Bankhead's unanswered question; silent failures; your RQ in one sentence; 3 contributions (the tool, the pre-registered evaluation, the measurement lessons) | §1, §3.1; reading #1, #11 |
| **2. Related work** | 0.75 | Four paragraphs: (i) error consistency in classification and LLMs (Gontijo-Lopes, Geirhos, Kim 2025); (ii) segmentation failure detection (Zenk, Kirscher, RBQE: "closest work; we differ: per cell, several references, against the model's own confidence, microscopy"); (iii) agreement-based bio-image QC (BISCUIT, SEG, MARC); (iv) VA for model disagreement and uncertainty (Visagreement, Calibrate). End with the gap statement | §3; `spike_results/K7_scoop_check.md` |
| **3. Research questions and pre-registered hypotheses** | 0.5 | RQ1–RQ4 with H1–H5 and H-label; thresholds; the held-out set and how it was chosen (scale check). One sentence pointing to the earlier pre-registered rounds | §5 |
| **4. Data and models** | 0.35 | LIVECell, NeurIPS22 Public-Test, the new held-out set; the leakage table; the roster; licenses | §6.1–6.3 |
| **5. Methods** | 0.5 | Matching and error taxonomy; κ, κ/κ_max **and log-OR**; QC signals; within-image AUROC, AURC, budget recall **with ceilings**; image bootstrap; Holm | §2, §6.4 |
| **6. Visual analytics design** | 0.6 | Views 1–5, one sentence each on the question it answers; a mock-up of view 1 or 2 (LIVECell image, not NeurIPS22) | §7 |
| **7. Preliminary results** | 0.4 | One figure (`spike_results/fig_kappa_by_level_v2.png`) and 3–4 sentences: copies share errors; mean-of-references > single; agreement ≈ own confidence; the family effect depends on data. State plainly that earlier headlines failed pre-registered tests | §4 |
| **8. Timeline, risks, deliverables** | 0.4 | The milestone table; top 3 risks; the scope tiers | §8, §9 |
| References | (extra) | About 15–20 | §3.3 |

**Tips:**
- **Lead with the question, then the tool that answers it.** "When does agreement reveal errors?" is the question; the tool is how a user (and you) see the answer per cell.
- **Put the preliminary results in.** Few proposals have pre-registered pilots on four datasets.
- **Be upfront about what failed.** One sentence: "two earlier hypotheses about model relatedness failed pre-registered tests on new data; we report them and build on what replicated." Reviewers trust this.
- **Make one clear, labelled figure:** `fig_kappa_by_level_v2.png`, or a mock-up of the triage view.
- Add an **AI-use disclosure** (the course requires it).

---

## 12. Open decisions, things to verify, and where everything lives

**Decisions for you:**
1. **The new held-out set** for RQ3 (core tier). Scale-check candidates from GT first. The D0 runner-up was the Xiong murine set (licence unclear).
2. **Which views to build first.** Recommended: view 1 (overlay + adjudication) and view 2 (triage), since the demo and H5 depend on them.
3. **Include CellSAM?** Only on datasets where its error rate is ≤ 0.6 (it fails on LIVECell).
4. **Think-aloud with 1–2 users?** Optional; strengthens the VA section.

**Verify yourself (quick):**
- Read BISCUIT's open reviews and confirm Bankhead's comment is still unanswered.
- **Scoop check (manual, ~20–30 min):** in Google Scholar, open "Cited by" for RBQE, BISCUIT, Gontijo-Lopes 2022 and Geirhos 2020; filter to 2026; skim titles for segmentation / cells / agreement. Before Oct 20 and again before Nov 3. First automated pass: `spike_results/K7_scoop_check.md`.
- Check whether micro-SAM's 1,151 NeurIPS22 training images include Public-Test (D6 open question).
- Confirm cyto3's training data includes LIVECell. It is secondhand so far, and it affects the L4 interpretation.

**Where everything lives:**

| What | Where |
|---|---|
| This onboarding doc | `D5_onboarding.md` |
| **Pre-registration history** (Part A: round 3; Part B: round 4; amendments) | `PREREG_D5.md` |
| Round-3 and round-4 results and verdicts | `spike_results/C1–C5`, `D0–D6` (verdicts: end of `C1_heldout_replication.md`, `D6_verdict.md`) |
| Proposal figures | `spike_results/fig_kappa_by_level_v2.png`, `fig_reference_tradeoff.png` |
| Round-3 cluster brief (kill tests, held-out replication, pitch figure) | `NEXT_SESSION_TASKS_3.md` |
| Round-4 cluster brief (reference-QC question, new held-out set N1, ViT-L contrast, controlled shift) | `NEXT_SESSION_TASKS_4.md`, `PREREG_D5.md` Part B |
| Scoop check, first pass | `spike_results/K7_scoop_check.md` |
| The D5 deep dive (full paper table, leakage quotes, detailed design, hour budget) | `lit_notes_open/deep_5_microscopy_qc.md` |
| Novelty check for the hierarchy framing (must-cites, experiment sketch) | `lit_notes_open/check_d5_error_hierarchy.md` |
| Spike write-ups | `spike_results/A1_microscopy_seg.md`, `spike_results/B1_d5_hierarchy.md` |
| Summary and rankings across all candidates | `research_gap_review.md` (short); `research_gap_details.md` (full) |
| Code, envs, models, outputs | Cluster: `~/vis4ml_spikes/` (`work/a1`, `b1`, `c1`, `c4`, `r4`; envs `d5`, the `d5c3` and `dinov3` overlays; fine-tuned seeds in `work/b1/models/`); aggregate tables in `spike_results/r4_tables/` |

*AI-use note: this onboarding doc was produced with Claude from the project's research notes and spike results. The research design choices are yours to make and defend.*
