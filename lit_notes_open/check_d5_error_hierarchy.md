# Novelty check: the "D5 + Rashomon" error-correlation hierarchy

**Date:** 2026-09-29 · **Searches:** 15 WebSearch, ~10 WebFetch, arXiv API, OpenAlex (noisy, no hits); Semantic Scholar gave HTTP 429.
**Labels:** FACT / AUTHOR CLAIM / SYNTHESIS / SPECULATION. Evidence levels: FULL TEXT / ABSTRACT / SECONDHAND. "FULL TEXT (summarized)" means WebFetch read the full HTML and a small model summarized it. Quotes in those rows were returned by that summarizer, so check them before citing.

**Claim tested.** *"No prior work measures, for instance/cell segmentation (or segmentation generally), how error correlation between models changes along a hierarchy of model difference (seed, then fine-tuned variant, then architecture/lineage, then training data), and uses this to judge when agreement-based QC is trustworthy."*

---

## 1. Verdict: **PARTLY FALSIFIED**

| Part of the claim | Status | Why |
|---|---|---|
| The hierarchy itself (seed, then hyperparameters, then architecture, then objective, then data), with error correlation falling at each step | **Falsified for classification** | Gontijo-Lopes et al., ICLR 2022 (FULL TEXT) do exactly this on ImageNet with 82 models. |
| "The source of diversity decides whether agreement-based estimates work" | **Falsified for classification, at dataset level** | Saxena et al., NeurIPS 2024 (ABSTRACT); Jiang et al. 2021 (ABSTRACT). |
| "…for segmentation generally" | **Partly falsified, at image level** | RBQE (arXiv 2609.10495; FULL TEXT, summarized) compares a same-architecture seed referee, cross-architecture referees and a SAM-based referee for per-image agreement QC on polyps. Kirscher et al. 2026 (ABSTRACT) compare seed ensembles with CV-fold ensembles for failure detection. Neither *measures error correlation*. Neither goes past two or three levels. |
| "…for instance / cell segmentation, per-object errors, shared-SAM-backbone generalists, linked to GT-free QC" | **Open (not found)** | No paper measures per-instance error consistency across a controlled hierarchy in any instance-segmentation domain. Absence is unproven (~15 searches). |

**Bottom line (SYNTHESIS).** "Errors decorrelate as models diverge" is Gontijo-Lopes' headline result, not a finding of ours. What survives is the **transfer and its consequence**: whether the ordering holds for **per-object** instance-segmentation errors, where **shared-SAM-backbone** models fall on it, and what that means for per-cell agreement QC (AUROC, silent failures). The interesting results are the *deviations* from the monotone ordering.

---

## 2. Evidence, general ML

**Gontijo-Lopes, Dauphin, Cubuk. "No One Representation to Rule Them All: Overlapping Features of Training Methods." ICLR 2022. arXiv 2110.12899. FULL TEXT (PDF text extracted locally).**
- FACT: five categories of model pair relative to a ResNet-50 base: "Reinits: identical models, just different in reinitialization"; Hyper-parameters; Architectures ("same framework and dataset"); Frameworks (e.g., SimCLR); Datasets (e.g., CLIP, BiT).
- FACT: contribution 1, verbatim: "Model pairs that diverge more in training methodology (in order: reinitializations → hyperparameters → architectures → frameworks → datasets) produce increasingly uncorrelated errors."
- FACT: the metric is **error inconsistency**, "the fraction of examples where only one model in the pair makes a correct prediction". They call it the complement of Geirhos' error consistency. It is **not chance-corrected**. Error inconsistency is "linearly correlated with the ensemble" gain.
- FACT: ImageNet classification only; the use is ensemble accuracy. **No failure detection, QC or segmentation.**
- SYNTHESIS: the direct precedent. Gaps: the metric is confounded by accuracy, and nothing tests agreement as an *error detector* per level.

**Geirhos, Meding, Wichmann. "Beyond accuracy: … error consistency." NeurIPS 2020. arXiv 2006.16736. FULL TEXT (summarized).**
- FACT: they define κ with expected overlap c_exp = p_i·p_j + (1−p_i)(1−p_j).
- FACT: model-to-model consistency across CNN families "is generally very high". The highest was κ = 0.793, for DenseNet-121 vs ResNet-18.
- FACT: seeds are not tested. The authors say it "remains an open question" why seed instances differ internally while different architectures behave alike.
- Relevance: the metric; and architecture changes alone may **not** decorrelate errors.

**Klein, Meyen, Brendel, Wichmann, Meding. "Quantifying Uncertainty in Error Consistency." arXiv 2507.06645 (2025). ABSTRACT.**
- FACT: EC values are noisy. They give bootstrap confidence intervals and a "copying" model for power analysis, and find "many reported differences between deep vision models are statistically insignificant."
- Relevance: we need CIs for every κ. Our 8-image κ = 0.52 has none.

**Saxena, Kim, Mehra, Baek, Kolter, Raghunathan. "Predicting the Performance of Foundation Models via Agreement-on-the-Line." NeurIPS 2024. arXiv 2404.01542. ABSTRACT.**
- FACT: when several runs are fine-tuned from *one* foundation model, "the choice of randomness … (linear head initialization, data ordering, and data subsetting) can lead to drastically different levels of agreement-on-the-line". "Only random head initialization" works reliably.
- FACT: ensembles of FMs "pretrained on different datasets but finetuned on the same task" also show agreement-on-the-line.
- Relevance: the closest *QC-consequence* precedent (dataset-level accuracy estimation). It warns that fine-tuning `cpsam` with different seeds varies only data order and augmentation, which they found insufficient.

**Jiang, Nagarajan, Baek, Kolter. "Assessing Generalization of SGD via Disagreement." arXiv 2106.13799. ABSTRACT.**
- FACT: the disagreement between two SGD runs of the *same* architecture on the same data estimates test error. They trace this to ensemble calibration.
- Relevance: seed-level disagreement can already be informative in-distribution.

**Kim et al. "Correlated Errors in Large Language Models." ICML 2025. arXiv 2506.07962. ABSTRACT.**
- FACT: over 350 LLMs. "models agree 60% of the time when both models err"; shared architecture and provider drive correlation; "larger and more accurate models have highly correlated errors, even with distinct architectures and providers". Downstream effect shown for LLM-as-judge.
- Relevance: accurate segmenters may correlate *because* they are accurate; read κ against accuracy.

**Also (ABSTRACT):**
- Goel et al. 2025 (2502.04313): CAPA, a chance-adjusted agreement on mistakes; "model mistakes are becoming more similar with increasing capabilities".
- Mania et al. 2019 (1905.12580): ImageNet models agree "well beyond" what accuracy implies.
- Bommasani et al. 2022 (2211.13972): the "component-sharing hypothesis". Toups et al. 2023 (2307.05862): "systemic failure" across deployed models. Together they give a name to the shared-SAM-backbone silent failures.
- Fort et al. 2019 (1912.02757): seed diversity. Abe et al. 2022 (2202.06985): diversity "does not meaningfully contribute" to OOD uncertainty quantification, a caution for the QC-benefit hypothesis. D'Amour et al. 2020 (2011.03395): underspecification.
- A paper titled "Do ImageNet classifiers make the same mistakes?" was not located. Do not cite it.

---

## 3. Evidence, segmentation

**Gupta and Singla. "Cross-Model Agreement as a Deployment-Time Reliability Signal for Automatic Polyp Segmentation" (RBQE). arXiv 2609.10495, 9 Sep 2026. FULL TEXT (summarized).**
- FACT: primary YOLOv8n-Seg; referees are a re-initialized YOLOv8n-Seg (seed control), SegFormer-B0, UNet++ (ImageNet-pretrained), and MedSAM **prompted from the primary's output**. Trained on Kvasir-SEG; tested on 4 external polyp sets (1,223 images).
- FACT: per-image failure-detection ROC-AUC: SegFormer 0.960, UNet++ 0.938, same-architecture YOLO 0.923, MedSAM 0.863. SegFormer beats the seed control by 0.037 (DeLong p<0.001). On the non-degenerate subset: 0.876 vs 0.783.
- AUTHOR CLAIM: "Independent training alone provides a useful reliability signal, while architectural diversity can provide additional discriminative strength". Also "diversity without output-level independence fails" (MedSAM).
- FACT (per the summary): **no direct error-correlation measure**; per image, semantic; data and pretraining held fixed; the one foundation model is prompt-coupled.
- Relevance: **partly falsifies "segmentation generally"** with a two-level hierarchy (seed vs architecture) linked to QC AUROC. The closest precedent.

**Kirscher et al. (DKFZ, Isensee, Maier-Hein). "Lost in the Folds: When Cross-Validation Is Not a Deep Ensemble for Uncertainty Estimation." arXiv 2605.18329, May 2026. ABSTRACT.**
- FACT: CV-fold ensembles (different data subsets) vs 5-seed deep ensembles on the same nnU-Net. The deep ensembles "improv[e] calibration and failure detection", while the CV ensembles track inter-rater variability better. Semantic medical segmentation.
- Relevance: varies the diversity *source* (seed vs data subset) for failure detection. No architecture level; no error-correlation metric.

**Zenk et al., MedIA 2024. arXiv 2406.03323. FULL TEXT (summarized; also FULL TEXT in deep_5).**
- FACT: the ensemble is "five networks with different experimental seeds" (U-Nets, 5 seeds × 5 folds); no heterogeneous ensembles. Silent failures occur where members agree on a wrong region (deep_5).

**Other items:**
- Åkesson et al., CBM 2024: seed-only models differ in performance. SECONDHAND; it covers metric variance, not per-case overlap.
- Siddiqui et al. 2023 (2309.10513, ABSTRACT): per-instance StarDist certainty, within one architecture.
- Miao et al. 2026 (deep_5, FULL TEXT): "three SAM-based models … showed the highest degree of agreement with each other". Agreement only, not linked to GT errors.
- BISCUIT and the Bankhead review (deep_5, FULL TEXT): the untested "uncorrelated errors" assumption.
- SAM-family "monoculture" searches found only per-model failure papers.

---

## 4. The surviving novelty statement (SYNTHESIS)

> For **cell instance segmentation**, we measure **per-object, chance-corrected error consistency** (Cohen's κ with image-cluster bootstrap CIs, reported with accuracy-matched bounds) between model pairs at controlled levels of difference:
> 1. seed;
> 2. fine-tuned variant from a shared checkpoint;
> 3. **shared SAM backbone** with a different decoder, training set and lineage;
> 4. non-SAM architecture;
> 5. same architecture on different training data.
>
> We show how each level changes **per-cell agreement as a GT-free QC signal** (within-image AUROC, AURC) and the **silent-failure mass**, testing the "uncorrelated errors" assumption behind agreement-based bio-image QC for SAM-based generalists.

**New vs** Gontijo-Lopes: segmentation, per-object, chance-corrected κ, a QC consequence, a backbone level. **vs** RBQE: instances, κ measured, five levels, silent failures. **vs** Saxena: per-object error detection, not dataset-level accuracy.

**The two testable deviations that make it a paper, not a replication (SPECULATION):**
- **H-mono:** SAM-sharing pairs (Cellpose-SAM ↔ micro-SAM) have κ closer to the seed level than to the non-SAM level, even though they differ in decoder, objective and data. The shared pretrained encoder would then cancel much of that diversity. The spike's κ = 0.52 is suggestive, but it comes from 8 images and has no CI.
- **H-data > H-arch:** a same-architecture pair trained on different data (e.g., Cellpose3 `cyto3` vs the dataset-specific `livecell_cp3`, if that model is still distributed; verify) decorrelates errors more than cross-architecture pairs trained on the same data. That would put data *before* architecture for instances, reversing Gontijo-Lopes' order.

A clean monotone transfer is a less exciting but reportable null: a quantified answer to Bankhead.

---

## 5. Minimal experiment (about 15–25 student hours; GPUs abundant)

**Data**
- **Primary:** LIVECell test, 100–200 stratified images. In-distribution for all models (own-trained, `cpsam` and `vit_b_lm` all saw LIVECell train).
- **Shift probe:** leave-one-cell-type-out. Own-seed models train on 7 cell types and test on the held-out one (e.g., SH-SY5Y, the spike's hardest).
- **Optional:** the NeurIPS22 Tuning or Public-Test set, restricted to its phase-contrast/brightfield subset.

**Model pairs (each level, at least 2–3 pairs)**

| Level | Pairs | How obtained | GPU |
|---|---|---|---|
| L0a reinit seed | Cellpose U-Net (cyto3 architecture) **from scratch** on LIVECell train, seeds s1–s4 → 6 pairs | `cellpose<4` env, `train_seg` | ~1–2 h each |
| L0b fine-tune seed | `cpsam` fine-tuned on a LIVECell-train subset, seeds s1–s3 → 3 pairs | cellpose 4.x `train_seg` | ~1–2 h each (ViT-L) |
| L1 checkpoint variant | `cpsam` off-the-shelf vs `cpsam`-FT; micro-SAM `vit_b_lm` vs `vit_l_lm` (vs `vit_b` generalist) | off-the-shelf | inference only |
| L2 shared SAM backbone, different lineage | Cellpose-SAM ↔ micro-SAM AIS ↔ CellSAM | spike env (done) | done |
| L3 non-SAM architecture | each SAM model ↔ `cyto3` | off-the-shelf | inference only |
| L4 same architecture, different data | `cyto3` (generalist) ↔ own-trained LIVECell U-Net ↔ `livecell_cp3` (verify availability) | mix | — |

**Per-cell error indicator.** e_m(c) = 1 if GT cell c is not matched at IoU **> 0.5** by model m. This fixes the spike's ≥ 0.5 caveat. Keep the merge / split / miss labels for a stratified secondary analysis.

**Metrics per pair**
1. **Error consistency κ** (Geirhos), with **95% CIs from an image-level cluster bootstrap** (following Klein et al.), because cells within one image are not independent.
2. **κ / κ_max(p_a, p_b)**, and κ restricted to accuracy-matched pairs or competent cell types. κ is bounded when accuracies differ, which the spike already showed for CellSAM.
3. **Error-overlap Jaccard** |E_a∩E_b| / |E_a∪E_b|, and P(b wrong | a wrong).
4. **Silent-failure rate:** cells where both models are wrong *and* agree with each other at IoU > 0.5, per 1,000 cells.
5. **QC utility:** within-image AUROC of agree_{a,b}(c), the IoU between a's and b's instances at c, for detecting e_a(c). Also AURC (Zenk's protocol), and a comparison against model a's internal signal (Cellpose flow error, micro-SAM foreground).

**Analysis**
- **Fig. 1:** κ (with CI) per pair, grouped by level. Rank order L0 > L1 > L2 > L3/L4 means the hierarchy transfers.
- **Fig. 2:** QC AUROC vs κ across pairs, with a bootstrap-CI slope. Predicted: negative.
- **Fig. 3:** silent-failure rate vs level. Predicted: a non-zero floor even at L3/L4 (systemic failure).
- Repeat on the held-out cell type to test stability under shift.

**Hour budget (student hours; GPU jobs overnight)**

| Task | Hours |
|---|---|
| `cellpose<4` env for cyto3 / `livecell_cp3` | 1.5 |
| Two training scripts (U-Net from scratch; cpsam fine-tune) plus sbatch arrays | 4 |
| Inference over about 12 models × 2 test splits | 2 |
| Extend `match.py` to IoU > 0.5; pairwise metrics; cluster bootstrap | 4 |
| Three figures | 3 |
| Debugging buffer | 3–5 |
| Write-up section | 3 |
| **Total** | **≈20–22 h** |

**Known confounds to state up front**
- Off-the-shelf levels (L2, L3) change decoder, objective and data at once. Only the own-trained levels are controlled.
- LIVECell GT noise (CellSAM's authors graded it): some "silent failures" may be label errors. Inspect 50.
- A cpsam fine-tune may give near-zero diversity (the Saxena warning). That is itself an L0b data point.

---

## 6. Must-cite list

1. **Gontijo-Lopes et al., ICLR 2022**: the classification hierarchy; the framing precedent. FULL TEXT.
2. **Geirhos et al., NeurIPS 2020**: the κ metric; high cross-architecture κ; seeds left open. FULL TEXT (summarized).
3. **Klein et al. 2025**: bootstrap CIs for error consistency. ABSTRACT.
4. **Saxena et al., NeurIPS 2024**: the source of diversity decides whether agreement-based estimation works for foundation models. ABSTRACT.
5. **RBQE, arXiv 2609.10495**: closest segmentation precedent, per image. FULL TEXT (summarized).
6. **Kirscher et al. 2026**: seed vs CV-fold ensembles for segmentation failure detection. ABSTRACT.
7. **Zenk et al., MedIA 2024**: the AURC protocol; seed-only ensembles; silent failures. FULL TEXT.
8. **Kim et al., ICML 2025**: correlated errors rise with accuracy, across architectures. ABSTRACT.
9. **Bommasani et al. 2022 / Toups et al. 2023**: component sharing and systemic failure. ABSTRACT.
10. **BISCUIT and the Bankhead review**: the assumption being tested. FULL TEXT (in deep_5).

---

## 7. Residual scoop risk

Low–Medium (SYNTHESIS).
- RBQE (Sep 2026) and "Lost in the Folds" (May 2026) show that segmentation groups are *actively* moving toward "which diversity makes agreement QC work". DKFZ in particular has the nnU-Net ensemble infrastructure to add an architecture level quickly.
- The microscopy, per-instance, SAM-lineage version is not visible in any of the three communities checked: medical failure detection, bio-image tooling, and model papers.
- **Re-check before Nov 3:** arXiv "error consistency segmentation"; new citations of RBQE and of Gontijo-Lopes that involve segmentation.

---

## Addendum (2026-09-30): cross-check by 3 external LLM runs (user-run), verified by the lead reviewer
- **All 3 runs agree:** the claim is **appears open / partly done**, the same verdict as this note. The surviving wording should say "in cell instance segmentation" and "as a function of model relatedness".
- **New papers they raised, checked (ABSTRACT level):**
  - **"In search of truth: Evaluating concordance of AI-based anatomy segmentation models", arXiv 2512.15921.** Six CT anatomy segmenters (TotalSegmentator v1.5 and v2.6, Auto3DSeg, MOOSE, MultiTalent, CADS) are compared without GT, with structure-wise agreement. No error-correlation or relatedness hierarchy; semantic segmentation. **Not a threat. Cite it** as a precedent for GT-free concordance.
  - **"Segmentation quality assessment by automated detection of erroneous surface regions", Comput. Biol. Med. 2023** (PMC10563140). It explicitly handles ensemble base models that "largely agree on mistakes". **Not a threat, but the "agree-and-wrong" phenomenon is already known.** D5 must frame silent failures as *quantified across the hierarchy*, not as a discovery. Cite it.
  - "Detecting Silent Failures in Rare Tumor Segmentation" (Springer 2026): single-model QC. Tangential; cite for terminology.
- **Caution:** one run labeled Kirscher 2026 as MICCAI 2026. We only verified it as arXiv. Treat the venue as unverified.
