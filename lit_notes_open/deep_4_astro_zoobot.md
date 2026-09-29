# Deep dive D4: Zoobot / Galaxy Zoo uncertainty and explanations vs. volunteer disagreement

Written 2026-09-29. This is a falsification-first review following `DEEP_BRIEF.md` and `SCAN_BRIEF.md` (including the round-2 lessons). It builds on `scan_8_astronomy.md`, area card (b).

**Labels.** FACT means I checked it this session (API, file or full text). AUTHOR CLAIM means the paper says it and I did not re-check. SYNTHESIS means my inference across sources. SPECULATION means a guess.

**Evidence levels.** FULL TEXT means I downloaded the PDF, ran `pdftotext` into `lit_notes_open/pdfs/d4_*.txt`, and read the relevant sections (method, results, limitations). ABSTRACT means arXiv, Crossref or OpenAlex metadata only. SECONDHAND means a search snippet or a citation inside another paper.

**Tool budget used.** 12 WebSearch calls (limit 60). The rest went through the arXiv API (about 30 queries), Semantic Scholar (worked about 5 times, with 429s in between), Crossref, the Zenodo, Hugging Face and GitHub APIs, `curl -I`, and one GZ3D FITS file plus the GZ3D metadata table (7.5 MB) that I parsed locally. **OpenAlex failed:** its daily budget for this IP was exhausted ("Insufficient budget", resets at midnight UTC), so I could not run the OpenAlex `cites:` or author sweeps. I used Semantic Scholar citation lists instead. **dblp timed out** from this machine.

---

## 0. Bottom line (read this first)

1. **The grade-safe RQ as first written ("is Zoobot's uncertainty aligned with volunteer disagreement beyond the counting-noise floor?") is PARTLY ALREADY DONE by the Zoobot authors.** Three pieces exist:
   - **Noise floor:** the GZ DESI paper already computes a Beta-posterior "minimum error that even a perfect model would experience" for each question. It reports the model is "as accurate as asking ~15 volunteers" (about 20 for smooth/edge-on, 6–10 for spiral/bar). FACT, FULL TEXT.
   - **Volunteer truncation:** GZ DECaLS truncated the votes of 387 galaxies that had more than 75 votes. It found the model equivalent to about 10 volunteers. FACT, FULL TEXT.
   - **Calibration:** GZ DECaLS shows posterior-coverage calibration curves, but **only for 2 binary questions**. Walmsley 2020 reported coverage calibration for smooth and bar. FACT, FULL TEXT.

   The general-ML side is also covered. **Soft-label-trained models are known to track annotator entropy** (Singh et al. 2025: +61% entropy correlation versus hard labels). **Hard-label models do not track it** (Singh and Pakrashi, Sept 2026, ρ = 0.24–0.55). So a plain "does it align?" study is not novel.
2. **What survives (SYNTHESIS, moderate confidence):** a *decomposed, small-N-aware* audit, which nobody has done for Galaxy Zoo. Its parts:
   - (a) Instance-level calibration-to-disagreement metrics (Baan et al. 2022's DistCE/EntCE) **corrected for small, heteroskedastic annotator counts (N = 5–40)**. Baan et al. name this as an open problem in their Limitations: "a better understanding of how many annotations are required for good estimates of human uncertainty". FACT, FULL TEXT.
   - (b) Separating **aleatoric** from **epistemic** uncertainty and asking which one tracks human disagreement. Aleatoric uncertainty is ambiguity the model predicts is inherent in the image (a predicted vote fraction near 0.5). Epistemic uncertainty is the model's own uncertainty about its parameters (MC-dropout or ensemble spread).
   - (c) Slices by question depth in the decision tree, magnitude, redshift, size, and imaging region.
   - (d) Linked views that expose the slices.

   This is a workshop-grade methods/audit paper, not a main-venue one.
3. **The higher-upside RQ (attributions vs. volunteer ambiguity, validated on GZ3D masks) is OPEN as far as I can find, and the data is verified available.**
   - GZ3D is live on SDSS DR17 SAS. It has 29,813 per-galaxy FITS files, each with a 525×525 SDSS image plus four 525×525 volunteer-count mask layers (FACT, parsed). About 9,800 galaxies have ≥3 bar drawers and about 10,000 have ≥3 spiral drawers (FACT, from the metadata table).
   - No paper evaluates galaxy-CNN attributions against GZ3D masks, or relates attribution agreement to volunteer disagreement. The closest papers:
     - Bhambra 2022 validates SmoothGrad against *bar lengths*, not masks. Its future work explicitly suggests trying uncertainty-aware or pretrained Zoobot-style models to get "cleaner saliency maps".
     - Sankar et al. 2024 relate machine attention to volunteer confusion for *Jupiter vortices*, not galaxies.
     - Bhatt et al. show that ensemble uncertainty inflates explanation variance, in general ML.
   - A 2026 Zoobot-family segmentation model (ZooBot:3D, trained on GZ3D) now exists. It gives a second reference, but it is also the most likely source of a scoop.
4. **Recommendation.** Run one project with two layers.
   - **MVP (grade-safe):** reproduce GZ DESI Fig. 5 (the noise floor) and the GZ DECaLS Fig. 19 calibration, then extend with the small-N-corrected aleatoric/epistemic audit and a Calibrate-style linked view. Calibrate is an in-lab TVCG paper.
   - **Stronger version:** the GZ3D attribution study on about 1,500 MaNGA galaxies. It asks whether *human* ambiguity or *model* (epistemic) uncertainty predicts explanation unreliability. It uses a mandatory brightness baseline.
5. **Single biggest risk: the brightness confound in RQ2.** Bars are central and bright, so any attribution that follows surface brightness will "hit" bar masks. If attributions do not beat a light-profile baseline, the plausibility result is empty. The secondary risk is a scoop by the Walmsley/Masters/Spindler group, who own every dataset involved.

---

## 1. Refined research questions (after falsification)

### RQ-A (grade-safe): small-N-aware, decomposed calibration of Zoobot to volunteer disagreement

> For a Zoobot decision-tree model fine-tuned on GZ DESI, is each galaxy's **Dirichlet-multinomial posterior predictive** calibrated to the *observed* volunteer vote counts, per question and per stratum, once finite-N counting noise is accounted for?
>
> Which component tracks human disagreement: **predicted ambiguity** (the entropy of the expected vote fraction, i.e. aleatoric) or **model uncertainty** (MC-dropout and 5-seed ensemble spread, i.e. epistemic)?

Why this is not already done:
- GZ DESI reports only an *aggregate* per-question error against the noise floor.
- GZ DECaLS shows calibration for two binary questions only.
- Neither decomposes aleatoric from epistemic uncertainty.
- Neither computes instance-level distributional calibration with a finite-N correction.

(FACT for what they do. SYNTHESIS for the absence.)

### RQ-B (higher upside): is human ambiguity a predictor of explanation unreliability?

> For Zoobot's "bar?" and "spiral arms?" outputs on MaNGA/GZ3D galaxies, do attribution methods (Integrated Gradients, Grad-CAM, SmoothGrad, occlusion):
> - (i) agree with each other more on volunteer-unambiguous galaxies than on ambiguous ones; and
> - (ii) put more relevance mass inside the GZ3D volunteer masks than a surface-brightness baseline does?
>
> And after controlling for model epistemic uncertainty and image covariates, does human ambiguity still explain attribution disagreement?

**Definitions (first use):**
- **Integrated Gradients (IG):** it integrates the gradients along a path from a baseline image to the input.
- **Grad-CAM:** it weights the last convolutional feature maps by their pooled gradients.
- **Occlusion:** it measures the output drop when a patch is masked.
- **Relevance mass accuracy:** the fraction of total positive attribution that falls inside the ground-truth mask.

---

## 2. Paper table (closest work)

| Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Walmsley et al., "GZ: Probabilistic morphology through Bayesian CNNs and active learning" | 2020 | MNRAS | peer-rev. | Predict vote-fraction posteriors with uncertainty and pick informative galaxies | GZ2 (open) | Binomial likelihood, MC dropout, BALD active learning | Posterior plots, calibration plots | Coverage error: bar 22.6% → 11.8% with MC dropout (0.2 max error) | Approximately calibrated after MC dropout | "Remain slightly overconfident"; dropout rate arbitrary (0.5); suggests deep ensembles; model "reluctant to predict extreme ρ" | yes (`mwalmsley/galaxy-zoo-bayesian-cnn`) | arxiv.org/abs/1905.07424 | FULL TEXT |
| Walmsley et al., "Galaxy Zoo DECaLS" | 2022 | MNRAS 509:3966 | peer-rev. | Release volunteer and ML morphology for 314k galaxies | Zenodo 4573248, CC-BY-4.0 (FACT) | Dirichlet-multinomial loss, 5-model ensemble × dropout | Posterior galleries, entropy-binned galleries, 2 calibration curves | Accuracy on confident galaxies ≈99%; vote-fraction MAE; 387 galaxies with >75 votes truncated → model ≈10 volunteers; coverage curves for 'edge-on' and 'has spiral arms' | "Classifier is correctly uncertain" | Calibration shown only for 2 binary questions; ambiguous galaxies hard to evaluate | yes (zoobot) | arxiv.org/abs/2102.08414 | FULL TEXT |
| Walmsley et al., "Galaxy Zoo DESI" | 2023 | MNRAS | peer-rev. | 8.7M-galaxy catalogue | Zenodo 7786416, CC-BY-4.0 (FACT); HF `mwalmsley/gz_desi` CC-BY-NC-SA-4.0 (FACT) | Multi-campaign Dirichlet loss, 5-model ensemble + MC dropout | Fig. 5: model error vs Beta-posterior noise floor per question | Model ≈15 volunteers overall; 6–10 for spiral/bar; ~20 for smooth/edge-on | Near noise floor on easy questions | **Catalogue models trained on all labelled galaxies, no test set reserved** (FACT). Released predictions are in-sample for voted galaxies, so do not use them for calibration | yes | arxiv.org/abs/2309.11425 | FULL TEXT |
| Walmsley et al., "Scaling laws for galaxy images" | 2024 | arXiv / workshop (status unverified) | preprint | Supervised pretraining scaling | GZ Evo | Many timm backbones | Scaling curves | Fixed canonical test set (AUTHOR CLAIM) | Domain pretraining helps downstream | No calibration or uncertainty analysis | yes | arxiv.org/abs/2404.02973 | FULL TEXT (limitations + setup) |
| Walmsley et al., "Galaxy Zoo Evo" | 2025 | NeurIPS D&B submission, **rejected by AC** (FACT, arXiv comment) | preprint | 823k images / 104M labels as a CV benchmark | HF, CC-BY-NC-SA-4.0 (AUTHOR CLAIM) | Multinomial-loss baselines | Class distributions | Accuracy, RMSE, multinomial loss | Names "learning under uncertainty from crowdsourced labels" as a target research topic | "Inter-annotator disagreement is expected"; deeper questions get fewer votes, giving "heteroskedastic uncertainty" | yes | arxiv.org/abs/2512.23691 | FULL TEXT |
| Bhambra, Joachimi, Lahav, "Explaining deep learning of galaxy morphology with saliency mapping" | 2022 | MNRAS | peer-rev. | Can SmoothGrad extract bars? | GZ2 Kaggle (open) | CNN + SmoothGrad → bar length | Saliency maps | Correlation with human bar lengths 0.76 (vs 0.59 for direct regression) | XAI beats direct regression for bar length | Future work: train only on strong-consensus galaxies, **or use uncertainty-aware (Walmsley 2020) or pretrained (Zoobot) models → "cleaner saliency maps"** (FACT) | "on acceptance" (unverified) | arxiv.org/abs/2110.08288 | FULL TEXT |
| Masters et al., "Galaxy Zoo: 3D" | 2021 | MNRAS 507:3923 | peer-rev. | Crowd pixel masks for bars/spirals in MaNGA targets | SDSS DR17 VAC, open (FACT, SAS listing) | 15 volunteers draw polygons; per-pixel counts | Mask overlays | Validation vs other catalogues (AUTHOR CLAIM) | 29,831 galaxies; masks "may provide a useful training set" | Mask threshold choice is non-trivial (per the ZooBot:3D paper) | FITS | arxiv.org/abs/2108.02065 | ABSTRACT + FITS parsed |
| Spindler, Walmsley, Masters et al., "Deep learning segmentation of spiral arms and bars for 600,000 galaxies in DESI" (ZooBot:3D) | 2026 (Jun) | submitted MNRAS | preprint | U-net soft segmentation trained on GZ3D | Zenodo (DOI not found in text; unverified) | U-net predicting the per-pixel volunteer fraction | Soft masks, threshold percentiles | MAE vs volunteer maps; emergent ring detection | 639,636 soft maps; MaNGA/SAMI cross-matches | Thresholding is non-trivial; volunteers draw arms of variable extent | `mwalmsley/zoobot-3d` (**no license**, pushed 2024-10; FACT) | arxiv.org/abs/2606.16507 | FULL TEXT |
| Walls, Barry, Mohan, Scaife, "MC conformal prediction ... radio galaxy classification under ambiguous ground truth" | 2026 | RASTI (accepted) | peer-rev. | Does MCCP set size track annotator ambiguity? | MiraBest + new soft labels from 8 annotators | Fine-tuned radio FM + MC conformal (Stutz 2023) vs HMC BNN entropy | UMAP coloured by set size | Pearson ρ = 0.15–0.30 | **"Only a weak correlation"**; annotator behaviour (labelling faint sources FRII) compresses entropy | Soft labels are expensive; annotator bias breaks coverage | not stated | arxiv.org/abs/2603.20000 | FULL TEXT |
| Singh & Pakrashi, "Does model uncertainty track human ambiguity?" | 2026 (Sep) | arXiv | preprint | Same question, general vision | CIFAR-10H, FER+ | 8 hard-label CNNs; entropy, inter-model disagreement | Density scatter | Spearman ρ 0.24–0.57 | Weak alignment; "hard-label training gives models no signal about annotation ambiguity" | Only hard-label models; no finite-N correction | not stated | arxiv.org/abs/2609.34506 | FULL TEXT |
| Baan, Aziz, Plank, Fernández, "Stop measuring calibration when humans disagree" | 2022 | EMNLP | peer-rev. | Calibration against human distributions | ChaosNLI (100 annotators) | DistCE (TVD), EntCE, RankCS | Histograms | Instance-level metrics | ECE vs majority is misleading under disagreement | **Needs a reliable human distribution; calls for work on "how many annotations are required"** (FACT) | yes (`jsbaan/calibration-on-disagreement-data`) | arxiv.org/abs/2210.16133 | FULL TEXT (method + limitations) |
| Sankar, Mantha, ..., Fortson et al., "Understanding Confusion ... consensus from volunteer labels" | 2024 | Citizen Science: Theory & Practice | peer-rev. | Can a NN explain low volunteer consensus? | Jovian Vortex Hunter (Zooniverse) | Latent space + attention | Latent-space plots, attention maps | Qualitative/semi-quantitative | Latent space separates sources of low consensus; attention highlights class features | Jupiter, not galaxies; no attribution faithfulness | unverified | doi.org/10.5334/cstp.731 | ABSTRACT |
| Cheng, Wang, Luo, "UEGMC: Estimating uncertainty in galaxy morphology classification" | 2026 (Aug) | arXiv | preprint | Post-hoc uncertainty typing (epistemic, data, reference, boundary) | Galaxy10 DECaLS, GalaxyMNIST (**hard labels**) | Heads on frozen FM features | Example grids | Misclassification-detection metrics | Beats EDL-style baselines | Uses no volunteer vote distributions (FACT: no "vote/volunteer" in text) | not found | arxiv.org/abs/2608.08398 | FULL TEXT (skim) |
| Prakash, Desai, Srijith, "Uncertainty modeling and OOD detection" | 2026 | Astronomy & Computing (accepted) | peer-rev. | OOD detection + calibration | GZ DECaLS 9-class | IsoMaxPlus + MC dropout | — | ECE 0.0095 → 0.0026 | Better OOD and ECE | Hard 9-class labels; no vote distributions | not stated | arxiv.org/abs/2608.16654 | ABSTRACT |
| Wu & Walmsley, SAEs on Euclid morphology | 2025 | NeurIPS ML4PS | workshop | Interpretable features beyond the GZ tree | Euclid Q1 + GZ | SAE on Zoobot/MAE features | Feature galleries | Alignment of SAE features with vote fractions | Features beyond the decision tree | Some features uninterpretable | yes | arxiv.org/abs/2510.23749 | FULL TEXT (grep) |
| Xenopoulos, Rulff, Nonato, Barr, Silva, "Calibrate" | 2023 | TVCG (VIS 2022) | peer-rev. | Interactive reliability diagrams | Tabular | Learned reliability diagram, subgroup brushing | Linked views | Think-aloud study | Better than binned reliability diagrams | **"We do not directly tackle multiclass calibration"** (one-vs-rest only); hard labels only | code not found (see §5) | arxiv.org/abs/2207.13770 | FULL TEXT (grep) |
| Bhatt et al., "Effects of uncertainty on the quality of feature importance explanations" | ~2021 | AAAI workshop (unverified) | workshop | Does predictive uncertainty propagate to explanations? | Tabular/image | Ensembles + explainers | — | Explanation variance | Ensemble disagreement → higher attribution variance | Epistemic only, no human labels | — | umangsbhatt.github.io/reports/AAAI_XAI_QB.pdf | SECONDHAND |

Other relevant items (ABSTRACT unless noted):
- Stutz et al. 2023, conformal prediction under ambiguous ground truth (arXiv 2307.09302).
- Peterson et al. 2019, CIFAR-10H, "Human uncertainty makes classification more robust" (ICCV).
- Collins, Bhatt, Weller 2022, soft labels from every annotator (HCOMP).
- Tao et al. 2026, ambiguity-aware post-hoc calibrators, including a "Dirichlet-Soft" calibrator (arXiv 2603.22879).
- Singh et al. 2025, "Distributions in, distributions out" (arXiv 2511.14117).
- Khurana et al. 2024, Crowd-Calibrator (COLM).
- Krishna et al., "The disagreement problem in XAI" (TMLR).
- Rair et al. 2025, Mapper on annotator ambiguity in text embeddings (EMNLP).
- Cavanagh, Bekki, Groves 2024, U-Net bar/spiral segmentation for bar lengths (MNRAS 530).
- Walmsley & Spindler 2023, the ML4PS segmentation precursor.
- Marín Díaz et al. 2026, *AI* (MDPI), a fuzzy/XAI framework comparing GZ2 vote-fraction clusters with physical clusters. It uses SHAP on tabular SDSS features plus an "interactive visualization layer". It is tabular, not images, so the overlap is low.

---

## 3. What is already known (so novelty is framed right)

**Established in astronomy (FACT, FULL TEXT):**
- Zoobot-style models trained on vote counts with a Dirichlet-multinomial loss predict vote-fraction posteriors whose *aggregate* error is close to the finite-volunteer noise floor on easy questions. Error is worse on spiral/bar and on deep-tree questions (GZ DESI Fig. 5).
- Posteriors are well calibrated for 'edge-on' and 'has spiral arms' (GZ DECaLS Fig. 19). MC dropout improves the coverage error (Walmsley 2020).
- Errors concentrate where volunteer vote fractions are 0.2–0.8 (GZ DECaLS). The same pattern is echoed in the Cosmic Dawn / Euclid papers (SECONDHAND via search).
- Human labels carry *systematic* biases, not only noise:
  - redshift/brightness "featured" bias, hence the `_debiased` columns (FACT, GZ DECaLS schema);
  - the spiral winding-direction bias in GZ1 (Hayes et al. 2016; ABSTRACT);
  - annotators defaulting to FRII in MiraBest (Walls 2026, FULL TEXT).

**Established in general ML (FACT/ABSTRACT):**
- Soft-label training makes per-sample entropy track annotator entropy. Hard-label training does not.
- ECE against majority labels is misleading when humans disagree; use distributional instance-level metrics (Baan).
- Conformal coverage breaks under ambiguous labels unless you sample from the label distribution (Stutz; Walls).
- Epistemic uncertainty inflates attribution variance (Bhatt; SECONDHAND).
- Attribution methods disagree with each other (Krishna).

**What this means for novelty (SYNTHESIS):**
- A result like "Zoobot's predicted entropy correlates with volunteer entropy" is *expected* from theory, because the model is trained on the distribution. On its own it would be rejected as a finding.
- The findable things are:
  - *where* it fails, beyond finite-N noise;
  - *which* uncertainty component carries the human-disagreement signal;
  - whether explanations degrade with human ambiguity *or* with model uncertainty.

  These are distinct hypotheses, and each outcome is informative.

---

## 4. Gap verdict

| Sub-question | Verdict | Evidence |
|---|---|---|
| Aggregate model error vs volunteer noise floor | **Already done** | GZ DESI Fig. 5 (Beta-posterior floor); GZ DECaLS truncation of 387 high-N galaxies |
| Posterior calibration, all questions and strata, with finite-N correction | **Partly addressed** | Only 2 binary questions calibrated (GZ DECaLS). Walmsley 2020 did smooth/bar on GZ2. No per-stratum PIT found. Baan et al. flag the small-N annotator problem as open |
| Aleatoric vs epistemic, and which tracks human disagreement, for a soft-label astro model | **Open (weak negative evidence)** | UEGMC types uncertainty but on hard labels. Walls/Mohan did radio with hard or small soft labels. Singh & Pakrashi used hard-label general models. arXiv queries "galaxy morphology aleatoric" and "galaxy morphology epistemic" returned 1–2 unrelated hits |
| Attribution vs GZ3D masks (physical plausibility) | **Open** | arXiv: `abs:"Galaxy Zoo: 3D"` gives 4 hits, none XAI. `abs:MaNGA AND abs:saliency` gives 0 relevant. `galaxy AND "pointing game"` gives 0. `galaxy AND deletion AND saliency` gives 0. WebSearch for GZ3D + saliency/attribution found only segmentation (U-Nets), no attribution evaluation |
| Attribution agreement vs volunteer disagreement | **Open in astronomy; adjacent in citizen science** | Sankar 2024 (Jupiter vortices, attention vs consensus) is the nearest. General ML has no direct study (WebSearch: "I could not locate ... a specific published study"). Bhatt covers epistemic uncertainty only |
| VA tool for model uncertainty vs human disagreement in astronomy | **Thin** | No VIS/CHI paper found. Calibrate (in-lab) is hard-label and binary/one-vs-rest. Marín Díaz 2026 is tabular. Stein's similarity-search app is a Streamlit app last pushed 2022 (FACT), with no uncertainty view |

Queries that returned nothing relevant (FACT):
- arXiv:
  - `abs:"galaxy zoo" AND abs:"deep ensemble"` (0)
  - `abs:GZ3D` (0)
  - `abs:galaxy AND abs:"pointing game"` (0)
  - `abs:galaxy AND abs:deletion AND abs:saliency` (0)
  - `abs:galaxy AND abs:"integrated gradients"` (2 unrelated)
  - `abs:"galaxy zoo" AND abs:conformal` (1 unrelated)
  - `abs:galaxy AND abs:"label distribution"` (4; none optical GZ)
  - `all:Visagreement` (0; the paper is not on arXiv)
  - `au:Nonato AND galaxy/astronomy` (0)
- Semantic Scholar citers of Walmsley 2020 (107), GZ DECaLS (136), GZ DESI (50) and Scaling Laws (9), keyword-filtered for uncertainty/calibration/explanation/attribution/crowd: the only new, relevant hits were Prakash 2026, Sankar 2024, Katachi (Wu 2024, interpretable CNN for star-formation history) and "Explainable galaxy interaction prediction" (2026). None tests calibration-to-disagreement or attribution vs masks.
- Citers of Bhambra 2022 (24): Cao 2025 (counterfactuals, ML4PS), a spiral-arm-count CNN with Grad-CAM++/SmoothGrad (2412.11696, qualitative), COSMOS2025, reviews. None uses masks or votes.

---

## 5. Instructor-lab overlap (VIDA / Silva / Nonato / Miranda)

- **No galaxy-ML paper** from the lab.
  - Silva homepage: the only astronomy item is **OpenSpace** (NASA SMD, 2017–2025, planetarium visualization). FACT.
  - Silva publication page and the course-site repo searched for "galaxy / zoobot / telescope": no hits. "astronomy" appears only as a suggested InfoVis project theme. FACT, `gh search code --repo ctsilva/ctsilva.github.io`.
  - arXiv `au:Nonato` + astro terms gives 0.
  - Scan 8's OpenAlex sweep (Silva 2023+) found none. I could not re-run OpenAlex today (budget exhausted), so treat this as FACT as of scan 8.
- **Direct method overlap (useful, must cite):**
  - **Calibrate** (Xenopoulos, Rulff, Nonato, Barr, Silva; TVCG 2023). It is *taught in this course* (the Calibrate slides are in `2024-VisML-CDS` and `2025-VisML-CSE`, FACT). Its stated limitations are multiclass calibration and hard labels only. **Extending Calibrate's learned reliability diagram to Dirichlet-multinomial vote-count targets is a natural in-lab "reproduce + extend" hook.** Code: I found no public repo (gh searches for Calibrate / xenopoulos returned nothing). Plan to re-implement: it is a kernel/learned smoother over (predicted, observed), a few dozen lines.
  - **MOUNTAINEER** (Solunke, Guardieiro, Rulff, ... Silva, Nonato; TVCG 2024) and **Visagreement** (P. Silva, Guardieiro, Barr, C. Silva, Nonato; TVCG 2025). Both are VA tools for disagreement among explanation methods. The Visagreement repo `priscylla/visagreement` exists: Python/captum, tabular `X_train`, no license, pushed 2023-10 (FACT). **RQ-B is "Visagreement's question, on images, with a built-in human-ambiguity axis and physical masks."** Reuse its agreement metrics and cite it.
  - "Exploring the relationship between feature attribution methods and model performance" (lab, 2024–25, per scan 8). It is related to RQ-B's "attribution quality vs model state".
- **Prior cohort:** `gh search repos` for "zoobot", "galaxy zoo", "galaxy morphology uncertainty", "galaxy zoo visualization", "galaxy saliency" and "gz desi" (created 2025-08 to 2026-09) found no DS-GA 3001/VisML repo. There was one unrelated course project, `abdelrhmanfathi-commits/StarGPT-Galaxy-Morphology` (ECCE 635, 2025-11, Zoobot classification). FACT. Absence is weak evidence.

---

## 6. Reproduction target verification

| Item | Status | Details |
|---|---|---|
| `mwalmsley/zoobot` | FACT | GPL-3.0; not archived; 127 stars; last push 2025-10-26; last commit 2025-08-19; PyPI 2.9.0 (2025-07-25), Python ≥3.9 |
| Dependencies | FACT (`setup.py`) | `torch>=2.7.0`, `lightning>=2.2.5`, `timm>=1.0.15`, `galaxy-datasets>=0.0.25`, `wandb`, `webdataset`. Stale pins exist only in extras: `python-dateutil==2.8.1` (utilities) and `docutils<0.18` (docs). Nothing stale in the core. torch ≥2.7 works on M1 (MPS) and CUDA |
| Tree fine-tuning | FACT (source) | `FinetuneableZoobotTree` has a Dirichlet head and Dirichlet-multinomial loss, plus a `schema` for the decision tree and `head_dropout_prob=0.5`. It supports MC dropout at test time |
| Pretrained weights | FACT (HF API) | Encoders only: `zoobot-encoder-convnext_{pico,nano,tiny,small,base,large}`, maxvit, efficientnet, resnet. **Apache-2.0**. convnext_nano is the most downloaded. Docs: "All models are encoder-only", trained on GZ Evo (GZ2, UKIDSS, Hubble, CANDELS, DECaLS/DESI, Cosmic Dawn). **No released GZ-DESI decision-tree head**, so you must fine-tune one (about 1–3 GPU-h). `baseline-tree-regression-convnext_base` (GPL-3.0, a Lightning ckpt) may be a tree model, but its contents are unverified |
| HF `mwalmsley/gz_desi` | FACT | CC-BY-NC-SA-4.0, with an extra condition: "all models trained on these datasets to be released as source code by publication". Configs: `default` = 319,530 train / 79,883 test (17.5 GB, 28 + 8 parquet shards of ~437 MB each, ~10k galaxies/shard); `tiny` = 3,195 / 798 (175 MB). The tiny test parquet downloads without login (`curl -I` → 200, 35 MB). Columns: image (424×424), `id_str`, `dataset_name`, `ra`, `dec`, and per-campaign (dr12/dr5/dr8) counts, fractions and `total-votes` for 10 questions. **No photometric covariates** |
| Covariates | FACT | Zenodo 4573248 (GZ DECaLS, CC-BY-4.0): `gz_decals_volunteers_5.parquet` (40.5 MB) and `_1_and_2.parquet` (18.7 MB), with NSA `redshift`, `mag_r`, `petro_th50/90`, `elpetro_absmag_r`, `wrong_size_warning`, and `_debiased` fractions. Zenodo 7786416 (GZ DESI v1.0.1, CC-BY-4.0): `external_catalog.parquet` (1.6 GB; photo-z and masses per the description), volunteer GZD-8 core (6.4 MB) |
| Leakage risk | AUTHOR CLAIM / unverified | The Evo encoders were pretrained on GZ DESI labels. The scaling-laws paper says it keeps a "fixed canonical test set", and the HF card says each dataset "has a random fixed train/test split". Whether the HF `test` split equals the Evo canonical test set is **unverified** (verify check 1) |
| GZ3D | FACT | `data.sdss.org/sas/dr17/manga/morphology/galaxyzoo3d/v4_0_0/` lists 29,813 `gz3d_*.fits.gz` files (~350 KB each) plus `gz3d_metadata.fits`. One file parsed: HDU0 is a 3×525×525 uint8 SDSS image with TAN WCS (2.75e-5 deg ≈ 0.099″/px); HDUs 1–4 are 525×525 float64 layers (per Masters 2021: centre, star, spiral and bar counts; **HDU-to-layer order unverified**); HDUs 5–7 are metadata/centre/star tables. Metadata: `GZ_total_classifications` (mean 48), `GZ_bar_votes` (≥3 for 9,779 galaxies), `GZ_spiral_votes` (≥3 for 9,966) |
| DESI cutouts at GZ3D WCS | FACT | `legacysurvey.org/viewer/cutout.jpg?ra=..&dec=..&layer=ls-dr10&pixscale=0.099&size=525` returns 200. So masks can be matched pixel-for-pixel to a DESI image (same TAN centre and scale, after a vertical flip; verify) |
| ZooBot:3D maps | AUTHOR CLAIM | "Publicly available via Zenodo" (DOI not in the extracted text); the repo has no license |
| Compute | SYNTHESIS | GZ DESI: 5 MC-dropout passes ≈ 15 ms/galaxy on an A100 for a larger model (AUTHOR CLAIM). A convnext_nano fine-tune on ~40k images at 224 px takes about 5 min/epoch on an 11 GB RTX (estimate), so 5 seeds × 10 epochs is about 4 GPU-h. Attributions for 1,500 galaxies × 2 targets × (IG 32 steps + SmoothGrad 16 + ~200 occlusion windows + Grad-CAM) is about 1.5M forward-equivalents, roughly 1–2 GPU-h. M1 is fine for analysis and the app, but slow for occlusion |

---

## 7. Project design

### 7.1 Hypotheses

**RQ-A**
- **H-A0 (reproduction).** A fine-tuned convnext_nano tree model on the GZ DESI HF split matches GZ DESI Fig. 5's "equivalent number of volunteers" within ±30% per question (about 20 for smooth/edge-on, 6–10 for spiral/bar). It also reproduces near-diagonal coverage curves for 'edge-on' and 'spiral arms' (GZ DECaLS Fig. 19).
- **H-A1 (strata).** Randomized PIT is a probability integral transform: the observed count's CDF value under the predicted Dirichlet-multinomial, randomized for discrete counts, which should be Uniform(0,1) if the model is calibrated.
  - Overall, the PIT is near-uniform.
  - It departs significantly from uniform in predictable strata:
    - (i) extreme predicted ρ, where Walmsley 2020 says the model is "reluctant to predict extreme ρ", so it should *over*-predict ambiguity;
    - (ii) deep-tree questions with small effective N (bar, winding, arm count);
    - (iii) faint, small or high-z galaxies;
    - (iv) imaging region (DECaLS vs BASS/MzLS).
- **H-A2 (small-N correction matters).** Naive instance-level EntCE (the entropy of the prediction minus the entropy of the observed vote fraction) is biased *negative* at small N, because the plug-in entropy of a 5-vote fraction underestimates the true entropy. Comparing each galaxy's EntCE to the distribution a *perfect* model would produce (simulate votes from the model's own posterior at the same N) removes most of the apparent "overconfidence". This is a general-ML methods point that answers Baan et al.'s stated open problem.
- **H-A3 (decomposition).** Predicted aleatoric entropy carries most of the correlation with observed volunteer disagreement. **Epistemic spread (MC dropout / 5-seed ensemble variance of E[ρ]) has partial correlation ≤ 0.1 with observed disagreement** after conditioning on predicted aleatoric entropy and N. If it is larger, the model is conflating "humans disagree" with "I don't know", and the linked views should show where.
- **H-A4 (ceiling).** The correlation between the model's predicted entropy and the observed entropy reaches ≥80% of the volunteer split-half reliability ceiling. Split-half reliability is computed by hypergeometric subsampling of each galaxy's vote multiset into two halves. This assumes exchangeable volunteers, because individual votes are not released.

**RQ-B**
- **H-B1 (plausibility beyond brightness).** For "spiral arms?", relevance mass inside the GZ3D spiral mask (weighted by the per-pixel volunteer count) exceeds a **surface-brightness baseline** (attribution = r-band flux) and a centred 2-D Gaussian baseline (matched to the galaxy size), for IG, SmoothGrad and occlusion. For "bar?", I expect *no* significant gain over brightness (SPECULATION); that would itself be a reportable null.
- **H-B2 (agreement vs ambiguity).** Inter-method agreement is lower on volunteer-ambiguous galaxies (bar vote fraction 0.3–0.7) than on unambiguous ones (<0.1 or >0.9). Agreement is measured as Spearman ρ on 28×28 downsampled maps and top-10% pixel IoU.
- **H-B3 (the discriminating test).** In a regression of attribution agreement and plausibility on human ambiguity, model aleatoric entropy, model epistemic spread, size, magnitude and inclination, *epistemic spread* dominates and human ambiguity adds little. The explanation is that attribution instability is a property of the model's parameter uncertainty (Bhatt et al.), not of the label distribution it has learned to predict. Either sign is publishable, because the question is precise.
- **H-B4 (spatial human agreement).** Plausibility is higher where GZ3D masks are spatially "sharp", i.e. a larger fraction of mask pixels were drawn by ≥ 8 of 15 volunteers.

### 7.2 Minimum viable project (grade-safe; fits about 26 h)

1. Fine-tune `FinetuneableZoobotTree` from `zoobot-encoder-convnext_nano`.
   - Data: 4 HF train shards (~40k galaxies) plus the full HF test split's dr5 rows, or 2–3 test shards (~20–30k galaxies). Restrict to the GZD-5 (dr5) campaign so N ≈ 40 for pre-active-learning galaxies. Join the NSA covariates from `gz_decals_volunteers_5.parquet` on `iauname`/`id_str` (verify the join).
   - Train 5 seeds and use MC dropout (5 passes each), so the 25-member predictive gives the epistemic spread.
2. Reproduce GZ DESI Fig. 5 (noise floor, equivalent-volunteers curves) and GZ DECaLS Fig. 19 (coverage).
3. Per question and per stratum:
   - randomized PIT histograms and coverage curves;
   - Baan DistCE/EntCE, both raw and "perfect-model-simulated" at matched N;
   - the split-half ceiling;
   - partial correlations for H-A3, with bootstrap CIs and Benjamini–Hochberg correction across questions × strata. BH is a procedure that controls the false discovery rate over many tests.
4. **Linked-view app** (Streamlit + Plotly):
   - (V1) a **Calibrate-style learned reliability / PIT view per question**, needed because the aggregate hides the stratum failures;
   - (V2) an **observed-vs-predicted disagreement scatter with a finite-N noise band**, needed to tell real divergence from counting noise;
   - (V3) **covariate brushing** (mag_r, redshift, petro_th50, N votes, tree depth, imaging region), needed because H-A1 is stratum-specific;
   - (V4) an **image grid with vote bars, predicted Dirichlet and epistemic spread**, needed because astronomers judge failure modes visually;
   - (V5) a UMAP of Zoobot features coloured by the PIT z-score, to find clusters of miscalibration beyond the named covariates.

   A sky map is optional. It is justified only if the region effect (DECaLS vs BASS/MzLS) shows up.

### 7.3 Stronger version (adds about 12 h)

5. **GZ3D subset.**
   - Selection: cross-match GZ3D metadata to GZ DECaLS dr5 by RA/Dec (1″) so that ambiguity comes from *independent* GZ DECaLS volunteers, not from GZ3D drawers. This matters because GZ3D volunteers only draw if they already see a bar, so ambiguous galaxies have thin masks, which is a selection effect.
   - Take about 1,500 face-on disks: the "featured" fraction > 0.5, "edge-on no" > 0.7, and not in the model's training set.
   - Fetch DESI JPEG cutouts at the GZ3D WCS. Check the alignment by overlay on 20 galaxies.
6. Attributions with Captum on the expected vote fraction for 'bar: strong+weak' and 'spiral: yes' (Dirichlet mean = α_answer / Σα): IG (black and blurred baselines), SmoothGrad, Grad-CAM (last ConvNeXt stage) and occlusion (16 px).
   - **Sanity checks:** the model-parameter randomization test (Adebayo 2018; a map that does not change when the weights are randomized is not explaining the model) and deletion/insertion curves.
7. Metrics:
   - relevance mass accuracy (hard mask at ≥3/15, and soft count-weighted);
   - pointing game;
   - pixel-level AUROC of attribution vs mask;
   - pairwise method agreement.

   Baselines: brightness, a centred Gaussian, random, and (secondary) the ZooBot:3D soft masks as an alternative "reference" (flagged as the same model family).
8. Add a V6 view: GZ3D mask overlay versus the four attribution maps, brushable by human ambiguity versus epistemic spread. This is the Visagreement idea applied to images.

### 7.4 Dataset (exact)

- **RQ-A:**
  - HF `mwalmsley/gz_desi` default: 4 train shards (~40k) + 3 test shards (~30k), about 3 GB on disk.
  - Zenodo 4573248 `gz_decals_volunteers_5.parquet` (40.5 MB) for covariates and the ~40-vote galaxies.
  - Optionally the 387 >75-vote galaxies for a direct high-N check (identify them via `total` > 75 in the volunteer catalogue).
- **RQ-B:**
  - About 1,500 GZ3D FITS (~0.5 GB) plus 1,500 DESI cutouts (~80 MB).
  - Optionally the ZooBot:3D MaNGA cross-match file (Zenodo, size unverified).
- Do not start with the `tiny` config for the science: its 798 test galaxies are too few to stratify. It is fine for pipeline debugging.

### 7.5 Models

- Primary: `zoobot-encoder-convnext_nano` + tree head (5 seeds × MC dropout).
- Control for leakage and pretraining effects: the same architecture from ImageNet-12k (`timm/convnext_nano.in12k`) fine-tuned on the same data. It shows whether Evo pretraining (which saw GZ labels) changes calibration.
- Optional second family for generality: `zoobot-encoder-maxvit_tiny_rw_224`.

### 7.6 Evaluation: success and failure criteria

| Claim | Metric | Success | Failure → what it means |
|---|---|---|---|
| H-A0 | Equivalent-volunteer N per question; coverage curve deviation | Within ±30% of GZ DESI Fig. 5; max coverage deviation ≤0.05 on 2 binary questions | A >30% gap means a fine-tuning or leakage issue. Report it and use the ImageNet control |
| H-A1 | KS statistic of randomized PIT per stratum; coverage error | ≥1 stratum with BH-adjusted p<0.01 **and** coverage error >0.05 | "Calibrated at fine granularity" is a positive but low-novelty finding. Pivot the weight to RQ-B |
| H-A2 | Mean EntCE, raw vs perfect-model-simulated | Raw mean EntCE < 0 at N≤10, and simulated-corrected EntCE within the ±CI of 0 | If there is no difference, finite-N is negligible at these N (still a useful negative) |
| H-A3 | Partial Spearman(epistemic, observed H), conditioned on predicted H and N | ≤0.1 means "clean decomposition"; >0.2 means "conflation", then characterise it in the views | — |
| H-A4 | Model–observed entropy ρ / split-half ceiling | ≥0.8 | <0.5 shows real headroom |
| H-B1 | Relevance mass (soft) minus the brightness baseline, paired bootstrap | Spiral: ≥+0.10 for ≥2 methods | ≤0 for all methods means the attributions are "brightness detectors" (a key negative) |
| H-B2 | Agreement, ambiguous vs unambiguous (Mann-Whitney, effect size) | Cliff's δ ≥0.2 | ≈0: ambiguity doesn't affect agreement |
| H-B3 | Standardised coefficients / ΔR² from adding human ambiguity | ΔR² for human ambiguity <0.02 while epistemic >0.05 (or the reverse, which is also reportable) | Both ≈0: agreement is driven by image covariates |
| Sanity | Randomization test | Maps change (SSIM <0.5) under weight randomization | Otherwise drop that method |

### 7.7 Hour budget (about 40 h)

| Block | Hours |
|---|---|
| Proposal (4 pp; due Oct 20) and 1-page update | 3 |
| Data: HF shards, NSA join, sanity plots; GZ3D subset + DESI cutouts + alignment overlay | 6 |
| Fine-tune 5 seeds (+ ImageNet control, 1 seed) and MC-dropout inference | 5 |
| Reproduction: Fig. 5 noise floor, coverage curves | 3 |
| RQ-A analysis: PIT/coverage by stratum, Baan metrics with finite-N simulation, split-half ceiling, partial correlations, BH | 5 |
| RQ-B: attributions, sanity checks, metrics, baselines, regression | 8 |
| Streamlit/Plotly linked views V1–V6 | 5 |
| Final 8-page report + slides + demo | 5 |
| **Total** | **40** |

To cut to the MVP only, drop RQ-B (−8 h) and V6. That frees buffer for debugging.

### 7.8 Failure modes and mitigations

1. **Brightness confound (RQ-B).** This is the biggest risk. Mitigate with the brightness and Gaussian baselines. Report *excess* relevance mass. Stress spiral arms, which are extended and faint, over bars. Add a "mask minus bulge" region (exclude the inner 0.2 R90).
2. **Encoder leakage.** The Evo encoder saw GZ DESI train labels, and possibly the test labels if the splits differ. Mitigate with verify check 1, the ImageNet-init control, and by restricting to galaxies absent from Evo train if a list can be obtained.
3. **GZ3D selection effect.** Masks exist mainly where volunteers see features. Mitigate by taking ambiguity from independent GZ DECaLS votes and by stratifying on the GZ3D drawer count.
4. **WCS/orientation misalignment** between SDSS masks and DESI cutouts. Mitigate by overlaying 20 galaxies. Fallback: run the model on the GZ3D SDSS image itself (in-domain for Evo's GZ2 portion), accepting a survey shift.
5. **The 424→224 resize and field-of-view mismatch.** GZ DESI cutouts use a size-scaled field; the GZ3D fields are fixed. Mitigate by choosing the cutout `pixscale` with the GZ DESI rule (from `petro_th50`) and reprojecting the masks with astropy WCS.
6. **Null results everywhere in RQ-A.** Mitigate by treating the fine-grained confirmation plus the finite-N correction as the methods deliverable, and by putting the novelty weight on RQ-B.
7. **Time sink in the front end.** Streamlit only; no D3.
8. **Scoop** by the Walmsley/Masters/Spindler group (they own GZ3D, ZooBot:3D and Zoobot). Mitigate by moving fast to an ML4PS-style short paper and framing it as a VIS/XAI-evaluation contribution (Visagreement/Calibrate lineage), which is not their focus.
9. **License obligations.** The HF gz_desi license requires the source code of trained models to be released at publication (FACT). zoobot-3d has no license, so read it but do not copy it.

### 7.9 Publication path (after the course)

- **Add:**
  - a second architecture family (MaxViT or a ViT MAE) and a second survey (GZ Evo HSC/Euclid) to show generality;
  - expert bar labels (e.g. Nair & Abraham 2010, availability unverified) as a third, non-crowd reference;
  - a small case study with 1–2 astronomers using the tool;
  - the finite-N-corrected Baan metrics as a stand-alone contribution, also tested on CIFAR-10H subsampled to 5–10 annotators, where the ground truth at 51 annotators is known. This last one is cheap and makes the methods claim general.
- **Venues:**
  - NeurIPS ML4PS workshop (Zoobot's authors publish there);
  - ICML EIML (the epistemic-intelligence workshop ran in 2026, FACT from 2605.20642);
  - IEEE VIS short paper, or a VIS workshop such as UncertaintyVis (VIS 2025 had an "Uncertainty Visualization ... AI" workshop, FACT from the program URL);
  - RASTI or Astronomy & Computing for the astro audience.

---

## 8. Novelty confidence (in words)

- **RQ-A: low-to-moderate.** Its core ("does Zoobot's uncertainty match volunteer disagreement") is largely answered by the Zoobot authors at the aggregate level. It is also *theoretically expected*, because the model is trained on vote counts.
  - The defensible increments are real but incremental:
    - finite-N-corrected instance-level metrics, a named open problem in Baan et al.;
    - the aleatoric/epistemic decomposition against human disagreement;
    - stratum-level PIT across all questions.
  - It is a safe course project with a clean reproduction target. As a paper it is a workshop short at best, unless the finite-N correction is generalised beyond astronomy (e.g. CIFAR-10H subsampling).
- **RQ-B: moderate.** I searched arXiv (8 targeted queries), Semantic Scholar citation lists of four anchor papers, and 4 WebSearches. None found attributions evaluated against GZ3D masks or conditioned on volunteer ambiguity. The closest analogues are in other domains (Jupiter vortices) or general ML (epistemic → explanation variance).
  - The ingredients are all public and recently refreshed (the ZooBot:3D paper, June 2026), which is exactly why the owning group could do it next.
  - I could not run OpenAlex, so 2025–26 non-arXiv venues (e.g. MDPI, Springer astro-informatics) are under-searched. One such item (Marín Díaz 2026) surfaced only via WebSearch.
  - Confidence that RQ-B is unpublished as of today: about 70% (SPECULATION, calibrated by how many adjacent items appeared in the last 6 months).

---

## 9. 12b-style row

| Course fit | Research upside | Technical risk | Data risk | Visualization burden | Evaluation clarity | Best suited if… |
|---|---|---|---|---|---|---|
| **High.** It hits model assessment/calibration (Sept 22; Calibrate is taught), black-box attribution (Oct 6), DL vis (Oct 27) and DR (Oct 20), and has in-lab anchors (Calibrate, Visagreement, MOUNTAINEER) | **Medium.** RQ-A is workshop-level; RQ-B is a plausible ML4PS/VIS-short paper with a clear yes/no hypothesis (H-B3) | **Medium.** Fine-tuning is routine, but mask alignment, field-of-view scaling and the brightness confound need care | **Low.** All sources verified live and open (HF CC-BY-NC-SA, Zenodo CC-BY, SDSS DR17); only GBs needed | **Low-Med.** Streamlit/Plotly linked views; no custom D3 | **High for RQ-A** (PIT/coverage, pre-registered thresholds); **Med-High for RQ-B** (objective masks, but baseline choice matters) | …the student wants a CV/XAI project with an objective human reference (masks and vote counts, not a user study), is comfortable with PyTorch fine-tuning, and accepts that the "safe" half is incremental |

---

## 10. "Verify this yourself" checks

1. **Split leakage.** Load `mwalmsley/gz_desi` (`tiny` first) and check that `id_str` for `dataset_name == dr5` matches `iauname` in `gz_decals_volunteers_5.parquet`. Then check in `galaxy-datasets` (`galaxy_datasets/pytorch/galaxy_dataset.py` mentions "canonical ... train/test split") or by emailing Walmsley that the HF test split is the one held out from GZ Evo encoder pretraining.
2. **GZ3D layer order and alignment.** Download 10 `gz3d_*.fits.gz`, read them with astropy, and confirm which of HDUs 1–4 is spiral vs bar (Masters 2021 datamodel). Fetch `cutout.jpg?...&pixscale=0.099&size=525`, flip it vertically, and overlay. Confirm the arms line up.
3. **Bar schema per campaign.** In `zoobot.shared.schemas` / `label_metadata`, confirm the answers (dr12: yes/no; dr5/dr8: strong/weak/no) and which campaign's N ≈ 40.
4. **Timing.** Run `FinetuneableZoobotTree` for 1 epoch on `tiny` on the 11 GB GPU, and time Captum IG (32 steps) per image. Rescale the §7.7 budget if it is more than 2× my estimates.
5. **Re-run the searches I could not.** Once the OpenAlex budget resets, run `works?filter=cites:<GZ3D W-id>` and `cites:<Bhambra W-id>` for 2025–26, and an OpenAlex author sweep for Nonato, Miranda, Guardieiro and P. Silva with "galaxy|astronomy". Also search the ZooBot:3D Zenodo record for its license.

---

## 11. Sources consulted (key URLs)

- Full texts: `lit_notes_open/pdfs/d4_{1905.07424, 2102.08414, 2309.11425, 2404.02973, 2512.23691, 2110.08288, 2606.16507, 2603.20000, 2609.34506, 2210.16133, 2608.08398, 2510.23749}.txt`, plus `d1_calibrate_2207.13770.txt`.
- Data: huggingface.co/datasets/mwalmsley/gz_desi · zenodo.org/records/4573248 · zenodo.org/records/7786416 · zenodo.org/records/5536996 · data.sdss.org/sas/dr17/manga/morphology/galaxyzoo3d/v4_0_0/ · legacysurvey.org/viewer (cutout service).
- Code: github.com/mwalmsley/zoobot · github.com/mwalmsley/galaxy-datasets · github.com/mwalmsley/zoobot-3d · github.com/priscylla/visagreement · github.com/jsbaan/calibration-on-disagreement-data · zoobot.readthedocs.io/en/latest/pretrained_models.html.
