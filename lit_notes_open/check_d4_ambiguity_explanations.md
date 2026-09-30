# Check: does human label ambiguity predict attribution unreliability? (D4 RQ-B, general-ML level)

Written 2026-09-29. This is a falsification-first check of the D4 RQ-B novelty claim, looking beyond galaxies. It builds on `deep_4_astro_zoobot.md` §0–1, which already covered the galaxy papers (Bhambra 2022, ZooBot:3D, Sankar 2024, Bhatt et al.). It also uses `spike_results/A4_zoobot_gz.md`, which found GZ3D masks align to about 1 px and that bar-mask flux is about 30× the cutout mean.

**Labels.**
- FACT: I checked it this session.
- AUTHOR CLAIM: the paper says it; I did not re-check.
- SYNTHESIS: my inference across sources.
- SPECULATION: a guess.

**Evidence levels.**
- FULL TEXT: I downloaded the PDF or HTML and read or grepped the relevant sections. "(WebFetch)" means a summarizer read the page, which is slightly weaker.
- ABSTRACT: the arXiv abstract only.
- SECONDHAND: a search snippet, or my prior knowledge that I did not re-check.

**Tool budget used.**
- 19 WebSearch calls (limit 40).
- About 25 arXiv API queries.
- 4 OpenAlex keyword searches. These were too noisy to be useful: they only rediscovered Saporta 2022.
- Semantic Scholar returned HTTP 429 (rate limit) on every try.
- Full texts read: 10.

---

## 0. Verdict: PARTLY FALSIFIED

The claim has two halves, and they come out differently.

**Half 1: "model uncertainty predicts explanation unreliability." Already done, several times.** Every study I found uses a model-side signal. None uses human labels.
- **Shaikhina, Bhatt et al. (AAAI XAI workshop, 2021).** Uncertainty across a set of candidate models "propagates to uncertainty in the feature importance explanations". Explanation quality (variance, complexity, monotonicity, efficiency, faithfulness) is "much poorer" for uncertain/out-of-distribution samples. The work is mostly tabular (Adult and similar datasets). FACT, FULL TEXT.
- **Mikriukov, Montavon & Höhne (arXiv 2603.29915, Mar 2026).** They propose "epistemic uncertainty as a low-cost proxy for explanation reliability". They find a "strong negative correlation between epistemic uncertainty and explanation stability", and that epistemic uncertainty also separates faithful from unfaithful explanations.
  - 4 tabular datasets plus a small image check (PlantVillage, 100 images, noise-*induced* uncertainty, ρ across noise levels, not instances). No aleatoric uncertainty, no human labels. FACT, FULL TEXT.
- **Saporta et al. (Nature MI, 2022; CheXlocalize).**
  - "For all pathologies the model confidence was positively correlated with the IoU saliency method pipeline performance" (Grad-CAM vs radiologist masks). Inter-rater variability appears only as a per-pathology human benchmark, never as a per-image stratifier. FACT, FULL TEXT.
- **Jukić, Tutek & Šnajder (ACL Findings, 2023).** They measure saliency-method agreement by *dataset-cartography* group. Dataset cartography sorts instances as easy, hard or ambiguous using the model's confidence and variability across training epochs. Their surprise: "easy-to-learn instances exhibit low average agreement, while ambiguous instances have a high average agreement" (NLP: SST-2, IMDB and others). "Ambiguous" here is model-defined, not human-defined. FACT, FULL TEXT.
- **In-lab:** P. Silva, C. Silva & Nonato (arXiv 2405.13957, 2024; ABSTRACT) find a "very strong correlation" between agreement across 9 explainers and model performance (education, tabular). Visagreement's Case Study 2 only conjectures the disagreement–error link (FACT per `research_gap_details.md`).

**Half 2: "human label ambiguity predicts explanation unreliability, contrasted with model uncertainty." No direct test found.** I checked vision, medical imaging, NLP and citizen science. The nearest misses are in §2. The strongest evidence of absence:
- Singh & Pakrashi (arXiv 2609.34506, Sept 2026) is the newest study of human ambiguity against model uncertainty (CIFAR-10H and FER+). It computes **no** saliency or attribution and cites none. FACT, FULL TEXT (WebFetch).
- Muscato, Chen, …, Plank, Giannotti (arXiv 2605.31563, May 2026) evaluate rationale plausibility on HateXplain under soft labels, but **do not stratify by per-instance annotator agreement**. They use attention-based attributions only and list gradient methods as future work. FACT, FULL TEXT (WebFetch).

So the combined claim, human ambiguity **versus** model uncertainty as competing predictors, is **OPEN at the general-ML level.** This rests on weak negative evidence: 19 web searches plus about 25 arXiv queries, with Semantic Scholar and OpenAlex citation sweeps unavailable. The claim must be reworded so it does not imply the model-uncertainty half is new.

**Direction warning (SYNTHESIS).** Jukić found higher explanation agreement on model-ambiguous instances. The hypothesis "more ambiguity → less agreement" is therefore **not** safe to pre-register as one-sided. Test it two-sided and treat either sign as a finding.

---

## 1. The surviving novelty statement

**General-ML version (use this wording):**

> Prior work links post-hoc attribution reliability to *model* uncertainty (ensemble or epistemic variance, confidence, training dynamics). No study we found tests whether *human* label ambiguity (annotator vote entropy) predicts attribution unreliability **after controlling for** the model's aleatoric and epistemic uncertainty. Unreliability here means lower cross-method or cross-seed agreement, lower deletion faithfulness, or lower localization against human-drawn masks beyond a brightness baseline.

**Galaxy/GZ3D-specific angle.** This is what makes GZ3D the right testbed rather than a domain port. SYNTHESIS, based on the dataset facts in the deep dive and spike.
1. **Two kinds of human disagreement on the same images.**
   - *Label* ambiguity: "is there a bar?", from GZ DESI/DECaLS vote fractions.
   - *Spatial* ambiguity: "where is the bar?", from GZ3D per-pixel vote counts over about 15 drawers.

   CheXlocalize has masks but no per-image vote distribution; CIFAR-10H and FER+ have vote distributions but no masks. LIDC-IDRI and Gleason19 have both, though I found no attribution-vs-agreement study on them. So GZ3D is **rare, not unique**; name LIDC-IDRI as the follow-up for generality.
2. **A model whose aleatoric head is trained on the human vote distribution.** Zoobot's Dirichlet-multinomial head is fit to vote counts, so its predicted ambiguity is built to mimic human ambiguity. This makes it the hardest and cleanest test of whether *human* ambiguity carries explanation-relevant information beyond the model's own learned ambiguity. General studies use hard-label models, where human and model uncertainty barely correlate (ρ = 0.24–0.55).
3. **A physically motivated null: surface brightness.** Bars are bright and central (the spike measured bar-mask flux at 30× the cutout mean). A light-profile baseline is therefore a *domain-given* control for the "saliency ≈ intensity" failure. CheXlocalize and the radiologist-gaze study do not control for it.
4. **Finite-N volunteer noise (N ≈ 5–40)**, which requires a posterior entropy estimate. This ties RQ-B to the small-N issue in RQ-A.
5. **Scoop status.** No galaxy XAI paper uses GZ3D masks (the deep dive's queries; FACT). The two new 2025–26 galaxy ML papers found here are not XAI-versus-masks:
   - Sucar et al. (arXiv 2609.06316): Galaxy Zoo 1, training under different annotator-agreement levels, no attributions. FACT, FULL TEXT grep.
   - Wu & Walmsley (arXiv 2510.23749): sparse autoencoders on Euclid Zoobot features, no masks. ABSTRACT.

---

## 2. Nearest misses (why each is not the claim)

| Paper | What it does | Why it does not falsify | Evidence |
|---|---|---|---|
| Mikriukov, Montavon, Höhne 2026 (2603.29915) | Epistemic uncertainty ↔ explanation stability and faithfulness | Model-side only; noise-induced; images n = 100 | FACT, FULL TEXT |
| Shaikhina, Bhatt et al. 2021 | Candidate-model uncertainty → worse feature-importance quality | Model-side; tabular | FACT, FULL TEXT |
| Saporta et al. 2022, Nat MI | Grad-CAM vs radiologist masks; confidence ↔ IoU | Inter-rater only as a per-pathology benchmark | FACT, FULL TEXT |
| Jukić et al. 2023, ACL Findings | Saliency agreement by cartography group | "Ambiguous" is model-defined; NLP | FACT, FULL TEXT |
| Singh & Pakrashi 2026 (2609.34506) | Human ambiguity vs model uncertainty, CIFAR-10H/FER+ | No explanations at all | FACT, FULL TEXT (WebFetch) |
| Muscato et al. 2026 (2605.31563) | Soft label and rationale spaces, HateXplain plausibility | Not stratified by agreement; attention only | FACT, FULL TEXT (WebFetch) |
| Bigolin Lanfredi et al. 2021 (2112.11716) | Grad-CAM vs 5 radiologists' gaze; interobserver upper bound | Subsets are only normal vs abnormal | FACT, FULL TEXT grep |
| Raghu et al. 2019, ICML (1807.01771) | Predicts doctor disagreement; SmoothGrad/IG shown in the appendix | Saliency *of* a disagreement predictor, qualitative | FACT, FULL TEXT grep |
| GleasonXAI 2025, Nat Comms | Soft-label pathology concept segmentation; Fleiss' κ | Inherently interpretable; no post-hoc attribution test | FACT, FULL TEXT |
| Fel et al. 2022, Harmonization | DNN vs ClickMe maps, normalized by human inter-rater alignment | Spatial ceiling only; not label ambiguity | SECONDHAND |
| Inoshita & Ueno 2026 (2606.22725); Wang et al. 2026 (2609.17130) | Aleatoric ↔ annotator disagreement in facial expression recognition (ρ = 0.66; 0.52) | Uncertainty only, no explanations; useful as ρ reference points | ABSTRACT |

---

## 3. Must-cite papers (9)

1. **Mikriukov, Montavon & Höhne 2026** (arXiv 2603.29915). The direct model-side precedent. Frame RQ-B as "adds the human-ambiguity predictor and a mask ground truth".
2. **Shaikhina, Bhatt et al. 2021** (AAAI XAI workshop). The original "uncertainty degrades explanation quality" result, and the deep dive's "Bhatt et al."
3. **Saporta et al. 2022** (Nature MI). The template for localization against expert masks (mIoU, hit rate / pointing game). It shows confidence ↔ IoU and leaves inter-rater variability aggregate.
4. **Jukić, Tutek & Šnajder 2023** (ACL Findings). Agreement is *higher* on model-ambiguous instances. It argues for Pearson-r over rank correlation, and it motivates the two-sided test.
5. **Singh & Pakrashi 2026** (arXiv 2609.34506). Human ambiguity is weakly tracked by model uncertainty. Explanations are absent, so this is where your gap starts.
6. **Singh et al. 2025** (arXiv 2511.14117). Soft-label training makes model entropy track annotator entropy. It explains why the Zoobot contrast is stricter.
7. **Krishna et al. 2022**, "The Disagreement Problem in XAI" (arXiv 2202.01602). The agreement metrics: feature, rank and sign agreement. SECONDHAND content; citation standard.
8. **P. Silva, C. Silva & Nonato 2024** (arXiv 2405.13957) and **Visagreement** (TVCG 2025). In-lab: explainer agreement ↔ performance. RQ-B is the ground-truth-anchored, human-ambiguity version.
9. **Bhambra et al. 2022** plus **Peterson et al. 2019** (CIFAR-10H). The galaxy-saliency precedent (bar lengths, not masks) and the canonical soft-label dataset.

---

## 4. Minimal RQ-B experiment (about 20 student hours; GPUs are not a constraint)

**Definitions (first use):**
- **Partial correlation:** the correlation between two variables after regressing out a set of controls from both.
- **Relevance mass accuracy (RMA):** the share of positive attribution inside the mask.
- **Deletion AUC:** the area under the model-output curve as the most-attributed pixels are removed. Lower means more faithful.
- **Pointing game:** a hit if the attribution's argmax pixel lies in the mask.

**Data**
- Galaxies: the intersection of MaNGA/GZ3D with GZ DESI (or GZ DECaLS) vote counts for *bar* (strong + weak vs none) and *spiral arms*.
- Keep galaxies with **≥10 DESI/DECaLS votes** on the bar question (for label entropy) and **≥10 GZ3D classifiers** (for mask reliability).
- Target **n ≈ 1,500**. The deep dive counted about 9,800 GZ3D galaxies with ≥3 bar drawers. Also include galaxies with *low* bar votes so the ambiguity range is not truncated.
- Input: the Legacy Survey cutout, 224 px; hold out every MaNGA galaxy from fine-tuning. GZ3D masks are resampled through the WCS, as in the spike. Do **not** use GZ3D HDU0, which has the IFU hexagon drawn on it.

**Model and uncertainty**
- Zoobot ConvNeXt-nano encoder with a Dirichlet-multinomial head on the bar question. Train **5 seeds** as a deep ensemble. This needs the full GZ DESI train split, which is the 17.5 GB download and needs your approval.
- **Aleatoric** = the entropy of the ensemble-mean predicted vote fraction (the expected Beta/Dirichlet entropy).
- **Epistemic** = the variance of the predicted fraction across seeds. MC dropout is an optional second estimate.
- **Human ambiguity** H_h = the entropy of the Beta posterior-mean vote fraction, Beta(1+k, 1+N−k). It shrinks small-N galaxies toward the middle. Keep N as a covariate.
- **Spatial human ambiguity** = 1 − (pixels with ≥50% of drawers ÷ pixels with ≥1 drawer).

**Attribution** (target: logit of the predicted bar fraction)
- Integrated Gradients (50 steps). Use a *blurred-image* baseline, because a black baseline is close to real sky and is uninformative.
- SmoothGrad (n = 25, σ = 0.1).
- Grad-CAM on the last ConvNeXt stage.
- Occlusion (16 px patch, stride 8, blur fill).

That gives 4 methods × 5 seeds × 1,500 galaxies ≈ 30k maps. SPECULATION: a few GPU-hours at the spike's 1.5 ms per forward pass; occlusion dominates.

**Unreliability outcomes, per galaxy**
1. *Cross-method agreement:* the mean pairwise **Pearson r** of |attribution| on 28×28-pooled maps (Jukić's argument). Report top-5% IoU as a robustness check.
2. *Cross-seed agreement:* the same metric, one method across the 5 seeds. This is a cheap Rashomon signal.
3. *Faithfulness:* deletion AUC with blur replacement, normalized by a random-order deletion AUC.
4. *Localization beyond brightness* (the key outcome):
   - Fit a per-galaxy pixel-level logistic model predicting the soft GZ3D bar mask from **log r-band flux + elliptical radius**. This is the light-profile baseline.
   - Add the attribution as a predictor and take **ΔAUPRC**, the attribution's gain over the baseline.
   - Also report raw RMA and pointing game next to the same metrics computed for the flux map and a fitted 2D Sérsic model (photutils). Readers can then see that brightness alone "localizes" well.

**Analysis**
- For each outcome, fit an OLS or rank regression on standardized predictors, with bootstrap 95% CIs:
  > outcome ~ H_h + aleatoric + epistemic + covariates

  Covariates: magnitude, half-light radius, redshift, axis ratio, mask area, number of GZ3D drawers.
- The primary quantity is the **H_h coefficient and partial correlation**, **two-sided**.
- The secondary quantity is the contrast: ΔR² of H_h against ΔR² of epistemic, compared with dominance analysis.
- **Collinearity check.** H_h and aleatoric will correlate strongly by design. Report the VIF, the variance inflation factor. If it is above about 5, also report the residualized H_h ("human ambiguity the model does not predict") as the predictor. That residual is the most defensible novel quantity.
- Visualization: a 3×3 grid of tercile(H_h) × tercile(epistemic) cells, showing mean ΔAUPRC and agreement, with example galaxies. This is the Vis4ML deliverable.
- **Sample size.** Partial r = 0.10 at α = 0.05, power 0.8 needs about 780 galaxies; about 1,100 after Holm correction over 2 questions × 4 outcomes. n = 1,500 is adequate.

**Controls:** an Adebayo-style randomization check (last stage re-initialized; agreement and ΔAUPRC should collapse); a spiral-arm replication on GZ3D spiral masks; *smooth-or-featured* for the agreement outcomes only (no mask).

**Confounds to state up front (SPECULATION, check them)**
1. GZ3D mask quality may itself fall with ambiguity: fewer or less-consistent drawers on borderline bars. Control for the number of drawers and the spatial-ambiguity score, or the localization outcome is circular.
2. Low-vote-fraction galaxies may have *no* bar mask, so localization is only defined when some volunteers saw a bar. Restrict outcome 4 to galaxies with at least 3 drawers, and say so.
3. Brightness is confounded with surface-brightness-dependent vote bias (the `_debiased` issue). Include magnitude and redshift.

**Hours (about 21):** data join and mask resampling at scale 4 (spike code exists); ensemble fine-tune 3; attribution pipeline 4; light-profile baselines and ΔAUPRC 3; regression, bootstrap and power 4; figures and the 3×3 view 3.

**Success and failure criteria**
- *Informative positive:* H_h (or residualized H_h) has a partial coefficient whose CI excludes 0 for ΔAUPRC or agreement, after the controls.
- *Informative negative:* epistemic dominates and H_h adds nothing ("reliability is a model property, not a data property").
- *Empty:* ΔAUPRC ≈ 0; then the brightness finding is the main result.

---

## Addendum (2026-09-30 00:10 UTC): retry after the rate limits reset

**Why the first check was weak:**
- Semantic Scholar returned HTTP 429 on every call.
- The OpenAlex daily quota was exhausted, and its keyword search was noisy.
- dblp was blocked.
- So citation chaining, the strongest test here, was not done.

**Retry, done directly by the lead reviewer:**

1. **OpenAlex citation chaining** (quota reset at midnight UTC). I scanned **every citer** of 6 seed works and kept those whose title or abstract mentions explanation terms (saliency, attribution, Grad-CAM, explanation, interpretability) **and** ambiguity terms (disagree, ambiguous, soft label, annotator, inter-rater, vote, crowd). About 850 citers in total:

   | Seed work | Citers scanned |
   |---|---|
   | Peterson 2019 (CIFAR-10H) | 187 |
   | Saporta 2022 (CheXlocalize) | 243 |
   | Bhatt 2021 | 224 |
   | Slack 2021 | 23 |
   | Collins 2022 (soft labels from every annotator) | 28 |
   | Uma 2021 (learning from disagreement) | 140 |

   Raw output: `check_d4_citation_chain_raw.txt`.

   **FACT: none of the 22 flagged papers tests whether human label ambiguity predicts post-hoc explanation reliability.** The closest are:
   - Collins et al., AIES 2023, "Human Uncertainty in Concept-Based AI Systems": concept-bottleneck models, not post-hoc attributions.
   - "Confidence Contours" (HCOMP 2023): annotation, not explanation.
   - SHAP-RC 2025: *explains* annotator disagreement; it does not test explanation reliability.

   Two seeds (Jukić 2023; Mikriukov 2026, titled "Uncertainty Gating for Cost-Aware XAI") were mismatched or rate-limited. They are covered by web search below.
2. **6 targeted web searches:** soft labels + saliency reliability; CIFAR-10H + attribution; inter-rater variability + Grad-CAM; human label uncertainty + attribution disagreement; Galaxy Zoo + saliency; follow-ups to Jukić 2023. None found a direct test.
3. **Two new September 2026 preprints were found and checked (abstract level):**
   - **Singh & Pakrashi, arXiv 2609.34506 (28 Sep 2026):** model uncertainty vs human ambiguity on CIFAR-10H and FER+ (ρ = 0.24–0.55). **No explanations.** It is relevant to D4's *safe* half (RQ-A) as a general-ML reference; cite it.
   - **Schmid et al., arXiv 2609.17753:** uncertainty mapping of label ambiguity (medical, Fazekas score). **No explanation or saliency test.**

**Updated verdict (SYNTHESIS):** the D4 RQ-B claim ("human vote entropy predicts attribution unreliability beyond model uncertainty") **remains open**. Evidence strength goes from *weak* to **moderate**. Still not covered: Semantic Scholar and Google Scholar citation graphs.
- **Suggested user check (about 10 min):** on Google Scholar, open "Cited by" for CIFAR-10H (Peterson 2019) and for Jukić 2023, and search within the citing articles for "saliency" or "attribution".
