# Scan 8: Astronomy / astrophysics ML + visual analytics (slice S8)

Scan date 2026-09-29. WebSearch calls used: 8. Everything else via OpenAlex, arXiv abstract fetches, HF/Zenodo/GitHub APIs. Labels: FACT (I checked it in an API or page), AUTHOR CLAIM, SYNTHESIS (my inference), SPECULATION, "unverified".

## 0. Cross-cutting checks

**Instructor-lab overlap (FACT unless noted)**
- OpenAlex author sweep for Claudio T. Silva (A5003584200, 356 works), 2023 onward: zero astronomy/galaxy/astrophysics titles. His 2023-26 output is urban/AR/XAI: MOUNTAINEER, TopoMap++, T-Explainer, "Exploring the Relationship Between Feature Attribution Methods and Model Performance", TiVy (time-series visual summary), POEM, BDIViz, ARGUS.
- A web search on "Claudio Silva" + astronomy returned only a general profile: real-time visualization of NASA missions and astronomical data with AMNH, Linkoping, Utah (this is the older OpenSpace line, AUTHOR CLAIM from his bio, not checked at paper level). One search snippet says he "advised a student who wanted to apply AI to astrophysics ... galaxy formation history with GNNs" (unverified, second-hand; do not cite).
- Reusable in-lab tools (SYNTHESIS): MOUNTAINEER/Visagreement (compare attributions), TopoMap++ (topology-guaranteed projection, which speaks directly to "are embedding neighbourhoods trustworthy"), TiVy (time-series summaries). Astronomy is not a lab area, so no direct overlap. A proposal that uses TopoMap on galaxy embeddings would be readable as "lab-adjacent".
- Prior cohort: `gh search repos` for "DS-GA 3001", "galaxy zoo visualization", "visualization for machine learning astronomy" found no VisML project (only unrelated DS-GA 3001 course repos, and an unrelated Oct-2025 beta-VAE Galaxy Zoo repo `stuti-bot/Galaxy-Clustering-VAE`, which does not mention VisML or Silva). No evidence of a prior cohort astronomy project (search is coarse; absence is weak evidence).
- NYU/Flatiron/Polymathic: AstroCLIP (Polymathic, code MIT, `PolymathicAI/AstroCLIP`, pushed 2025-12), Multimodal Universe (NeurIPS D&B 2024; last author John F. Wu; repo `MultimodalUniverse/MultimodalUniverse`, MIT, pushed 2026-06). Their work is ML, not VA. I found no Polymathic/Flatiron visual-analytics or interpretability paper on AstroCLIP embeddings (queries: "AstroCLIP interpretability", "interactive embedding explorer ... AstroCLIP OR Zoobot OR AstroPT"). Not a proof of absence.

**Compute reality (SYNTHESIS)**: Only small slices are needed. Do not touch the 100 TB Multimodal Universe (MMU) whole. Per-subset HF sizes (FACT, `usedStorage` from HF API): gz10 2.7 GB, plasticc 17 MB, legacysurvey 71.6 TB (no), ssl_legacysurvey 28 GB, desi 7.7 GB, btsbot 4.8 GB. Zoobot encoders are small (convnext_nano checkpoint 180 MB, Apache-2.0).

---

## 1. Area card (a): Foundation-model embeddings, anomaly discovery, embedding-space exploration

**Instructor-lab overlap:** none on astronomy (see 0). Topical overlap: TopoMap++ (projection trust), MOUNTAINEER (neighbourhood/attribution comparison).

**Key 2023-2026 papers**
1. Lanusse et al., AstroCLIP, MNRAS 2024 (arXiv 2310.03024). Peer-reviewed. Contrastive image-spectrum embedding; similarity search, photo-z. Code MIT, weights on HF (`polymathic-ai/astroclip`, `astrodino`, `specformer`, MIT for specformer; FACT).
2. Angeloudi et al., The Multimodal Universe, NeurIPS D&B 2024 (arXiv 2412.02527). Peer-reviewed. ~100 TB, CC-BY-4.0 (AUTHOR CLAIM from abstract page), HF org has 24 subsets (FACT).
3. Smith et al., AstroPT, arXiv 2405.14930. Preprint at my check (publication status unverified). Autoregressive "observation model"; code AGPL-3.0 (FACT).
4. Duraphe, Kumar, Smith, Sourav (UniverseTBD), "What AstroPT knows about galaxies, and what that can teach us about LLMs", arXiv 2608.22614 (Aug 2026). Preprint. Linear probes across checkpoints/layers/sizes; properties emerge in an order matching difficulty (magnitudes early/shallow; redshift, sSFR late/deep). Code on GitHub. FACT (abstract fetch). This is the closest "interpretability of astro FMs" item and is very fresh.
5. Wu and Walmsley, "Re-envisioning Euclid Galaxy Morphology: ... Sparse Autoencoders", arXiv 2510.23749, NeurIPS ML4PS workshop (accepted). SAEs on Zoobot and MAE features; features beyond the Galaxy Zoo decision tree; MAE on HF, demo + code. Active/saturating for SAE-on-galaxy.
6. Euclid Q1 multimodal FM (AstroPT), A&A 2026, arXiv 2503.15312. Peer-reviewed. UMAP coloured by physical properties, LOF/isolation forest anomalies; the reported anomalies "consist of images with multiple objects in cutouts" (seen in search snippet, i.e. many top anomalies are pipeline artefacts).
7. Mohale and Lochner, "Enabling unsupervised discovery in astronomical images through self-supervised representations" (BYOL on Galaxy Zoo DECaLS + Astronomaly), MNRAS 2024 (arXiv 2311.14157). Peer-reviewed. Code availability unverified.
8. Stein et al., "Self-supervised similarity search for large scientific datasets" (arXiv 2110.13151) and "Mining for strong gravitational lenses with self-supervised learning" (ApJ 2022, arXiv 2110.00023). 42M/76M Legacy Survey cutouts; released a public web similarity-search tool (AUTHOR CLAIM). Tool liveness not checked.
9. Ortiz Manrique and Boquien, "Interpreting anomaly detection of SDSS spectra", arXiv 2510.05235 (A&A manuscript). VAE anomaly scores + adapted LIME + clustering; separates artefacts from astrophysical outliers. Code public (AUTHOR CLAIM).
10. AnomalyMatch (arXiv 2505.03509), semi-supervised + active learning for rare objects; "Making Euclid VIS Imaging AI-Ready" (arXiv 2609.32753, unverified content); Astronomaly-on-KiDS (arXiv 2609.00154, unverified content). Preprints.

**Saturation verdict:** ML side active-to-saturating (SAEs, probes, anomaly + active learning all have 2025-26 papers). *Visual-analytics / faithfulness side is thin* (SYNTHESIS): I found no paper that quantifies how stable embedding-neighbourhood "discoveries" are under projection choice, encoder choice, or preprocessing. Evidence: queries below returned only tool/ML papers.

**Open data (FACT via API unless noted)**
- HF `mwalmsley/gz_desi`: CC-BY-NC-SA-4.0, 52 GB total but has a `tiny` config; vote counts/fractions per question. Non-commercial license is fine for coursework.
- HF `MultimodalUniverse/gz10` (2.7 GB), `.../ssl_legacysurvey` (28 GB), `.../desi` (7.7 GB) (licence field empty on card; parent MMU stated CC-BY-4.0 by authors).
- Zenodo 5536996 "Galaxy Zoo DECaLS: Trained Representations" (CC-BY-4.0, 5.1 GB): precomputed embeddings from Walmsley 2022. Useful for a fast start.
- Weights: Zoobot pip (GPL-3.0 code; encoders Apache-2.0 on HF), AstroCLIP HF.

**Candidate RQs**
- *Grade-safe RQ1a: "How stable are embedding-space neighbourhoods and top-k anomalies across encoders and projections?"*
  - Reproduction target: AstroCLIP similarity search (public code/weights) or Mohale and Lochner-style anomaly ranking on Galaxy Zoo DECaLS with Zoobot embeddings.
  - Minimal extension: compute neighbourhood-preservation metrics (trustworthiness, continuity, k-NN Jaccard between Zoobot vs AstroDINO vs AstroCLIP), plus rank-correlation of top-100 anomaly lists; use GZ vote fractions as an external "physical/human" reference. Plotly/Streamlit linked views: UMAP + TopoMap-style projection + neighbour panels.
  - Evaluation: agreement of anomaly lists with GZ "artifact/merger/odd" vote fractions; a small manual audit of 100 objects (subjective, so keep the audit second-order).
  - Falsification check: queries run: "anomaly detection galaxy images foundation model embeddings" (OpenAlex), "similarity search Legacy Survey embeddings", "interactive embedding explorer UMAP galaxy ... AstroCLIP OR Zoobot OR AstroPT", arXiv fetches. Found tools and ML anomaly papers, but no stability/projection-audit study. Weak negative; likely partly done inside individual papers as robustness sections (SPECULATION).
- *Higher-upside RQ1b: "Do AstroCLIP/Zoobot SAE features (Wu and Walmsley) predict which top anomalies are artefacts vs real, and can a VA workflow with feature-level explanations reduce human vetting?"*
  - Extra risk: depends on their SAE code releasing usable checkpoints; vetting labels are subjective; near a very active group (Walmsley/Wu).

**Feasibility (40 h):** Med-High. Data small; embeddings cheap on M1 (10-50k images). Front-end: Streamlit/Plotly only, fine.
**Publication path:** NeurIPS ML4PS workshop, ICLR/ICML AI4Science workshops, IEEE VIS short/poster, VisXAI. Astro-side venue: RNAAS/Astronomy and Computing.
**Red flags:** Fresh competition (SAE, AstroPT probes); "discovery" claims cannot be verified without astronomers (ground truth is scarce); anomaly ground truth is subjective.

---

## 2. Area card (b): Galaxy morphology (GZ DECaLS/GZ2/Zoobot): XAI faithfulness, vote-fraction uncertainty

**Instructor-lab overlap:** none astro. Strongly overlaps method-wise with MOUNTAINEER/Visagreement (comparing local explanations) and the lab's "attribution vs model performance" paper. This is a plus: a natural "reproduce lab's tool idea in a new domain" story, but flag that generic "apply to new domain" novelty is weak (per the brief).

**Key papers**
1. Walmsley et al., Galaxy Zoo DECaLS, MNRAS 2022 (DOI via OUP 509/3/3966). Peer-reviewed. Dirichlet-multinomial loss, 314k galaxies; data on Zenodo 4573248 (CC-BY-4.0, ~108 GB total).
2. Walmsley et al., Bayesian CNNs + active learning, MNRAS 2020 (arXiv 1905.07424). Peer-reviewed; code public (`mwalmsley/galaxy-zoo-bayesian-cnn`). Reports calibration in vote-fraction coverage (e.g. 11.8% coverage error within 0.2 for bars; AUTHOR CLAIM).
3. Walmsley et al., Zoobot, JOSS 2023; repo GPL-3.0, pushed 2025-10 (FACT). Pip-installable.
4. Walmsley et al., Galaxy Zoo DESI (arXiv 2404.02973), MNRAS 2024; HF dataset above.
5. Galaxy Zoo: Cosmic Dawn (arXiv 2509.22311) and Euclid morphology paper (2402.10187). Newer labels and Euclid transfer.
6. Wu and Walmsley SAE paper (above), and "Insights into Galaxy Evolution from Interpretable Sparse Feature Networks", ApJ 2025 (hypothesises Zoobot features are polysemantic; DOI 10.3847/1538-4357/adadec).
7. Cavanagh and Bekki 2022: SmoothGrad saliency for bars (from search snippet; venue unverified).
8. Prakash, Desai, Srijith, "Galaxy Morphology Classification: Uncertainty Modeling and OOD Detection", arXiv 2608.16654. Preprint; ResNet-34 + IsoMaxPlus + MC dropout on GZ DECaLS 9-class; ECE cut 65-73% (AUTHOR CLAIM); does NOT use GZ vote distributions (FACT from abstract fetch); code not mentioned.
9. Domain adaptation GZ DECaLS to BASS/MzLS (arXiv 2412.15533; Zenodo 10579386).

**Saturation verdict:** Classification and uncertainty: saturated on the ML side. **XAI faithfulness against physically meaningful features: thin-active.** Saliency papers exist (bars) and SAEs exist; I found none that (i) tests saliency/attribution faithfulness with deletion/insertion metrics against known physical structures (bar/spiral arm/bulge masks), or (ii) compares attribution methods with GZ **vote-disagreement** as a proxy for ambiguity (SYNTHESIS; queries: "saliency explainability galaxy morphology classification", "Galaxy Zoo Zoobot saliency OR feature attribution ... faithfulness", "Galaxy Zoo uncertainty saliency" (arXiv API returned empty, so not fully covered)). Note the Galaxy Zoo decision tree also gives a built-in concept hierarchy (a plus for concept-based XAI).

**Open data:** GZ DESI on HF (tiny config feasible); Galaxy10 DECaLS via MMU `gz10` (17,736 galaxies, 10 classes, 2.7 GB; FACT card); Zenodo GZ DECaLS (CC-BY-4.0). GZ2 Kaggle not checked (GZ2 and Galaxy10 SDSS also public via astroNN; unverified).

**Candidate RQs**
- *Grade-safe RQ2a:* "Are Zoobot predictive Dirichlet uncertainty and MC/ensemble spread aligned with volunteer disagreement, and where do they diverge?" Reproduce: Walmsley 2022 calibration (Zoobot pip + released weights). Extension: linked-view calibration explorer (reliability diagrams per question, stratified by total-votes and by vote-fraction entropy, with image drill-down). Evaluation: calibration error vs a held-out volunteer-vote resampling baseline (the irreducible noise floor: how well do subsamples of volunteers predict the rest). Falsification: no paper found comparing model-uncertainty with the volunteer-resampling noise floor (queries above). Risk: Zoobot's own paper covers much calibration (so novelty is the VA plus noise floor). Feasibility High.
- *Higher-upside RQ2b:* "Do attribution methods (IG, GradCAM, SmoothGrad, SAE features) for Zoobot agree with each other and with physical masks (bar/arm/bulge segmentation) more where volunteers agree than where they disagree?" This reuses MOUNTAINEER/Visagreement ideas (cite them, use their disagreement metrics), giving lab-relevant framing and "reproduce (Visagreement idea) + extend (new domain with built-in human-disagreement axis)". Extra risk: physical masks need labels (use photometric decomposition or GZ bar-length catalogs, unverified availability), attribution on a ViT/ConvNeXt is fiddly, and generic-XAI-in-new-domain novelty is modest.

**Feasibility:** High for 2a; Med for 2b (11 GB GPU is fine for Zoobot convnext_nano/ResNet fine-tuning).
**Publication path:** NeurIPS ML4PS, VisXAI workshop, Astronomy and Computing; VIS short is a stretch unless a genuinely new visual design emerges.
**Red flags:** Zoobot is GPL-3.0 (irrelevant for coursework); "physical faithfulness" ground truth is the hard part; ImageNet-style XAI evaluation critiques apply.

---

## 3. Area card (c): Time-domain (PLAsTiCC/ELAsTiCC) and photo-z PDFs

**Instructor-lab overlap:** No astro papers, but TiVy (time-series summaries, 2025) and Nov 17 course topic on time series. mTSeer (CHI 2021, Silva and Bertini; from the brief) is time-series-model VA. Reproduction/extension could sit on that lineage.

**Key papers**
1. PLAsTiCC results paper, ApJS 2023 (DOI 10.3847/1538-4365/accd6a). Peer-reviewed. Kaggle challenge; results data on Zenodo 2539456, CC-BY-4.0, ~9 GB test lightcurves (FACT; train set 20 MB).
2. Zhou, Malz, et al., "Beyond the Final Label: ... Classification Histories", arXiv 2604.23792. Preprint. RNN + attention over ELAsTiCC classifier probability histories; Wasserstein-based stability/early-classification metrics; code not stated (FACT abstract). Close to a "VA for probabilistic classifier trajectories" idea, hence overlap risk with Malz (who works on calibration).
3. ORACLE (ApJ 2025, DOI 10.3847/1538-4357/ae1130; Zenodo 15328166, MIT). Hierarchical LSST classifier; ELAsTiCC data on Zenodo (e.g. 20150662, CC-BY-4.0, 450 MB).
4. Fink transient classifiers (A&A 2024), ATAT (2023), Tiny Time-series Transformer (2303.08951), NightLANP (2026, preprint, unverified).
5. Photo-z: RAIL (arXiv 2505.02928) provides PIT/QQ diagnostics; LADaR (2205.14568) instance-wise calibration; CLAP I (A&A 2024, arXiv 2410.19390); photo-z PDFs for Legacy Surveys via NN classification (arXiv 2602.01548); Mantis Shrimp with CalPIT (2501.09112). All open-ish; mostly code-public.
6. Malanchev/Fink-type anomaly work and Pruzhinskaya-type live anomaly detection (search results, details unverified).

**Saturation verdict:** Active on metrics/calibration (PIT-based diagnostics are mature and standardised in RAIL); **thin on interactive visualisation of PDF calibration or classifier PMF trajectories**, though the Zhou et al. preprint occupies part of the trajectory idea. Evidence: the visual-analytics query returned only ML/broker papers, no VA tool.

**Open data:** PLAsTiCC (Zenodo, CC-BY-4.0; HF MMU `plasticc` 17 MB train-only), ELAsTiCC (Zenodo, various CC-BY), SDSS/Legacy photo-z catalogs. Kaggle PLAsTiCC needs a Kaggle login; use Zenodo instead.

**Candidate RQs**
- *Grade-safe RQ3a:* "Where are PLAsTiCC classifier probability outputs miscalibrated, by class, redshift, phase and cadence?" Reproduce: a public PLAsTiCC top-solution or ORACLE/ELAsTiCC baseline. Extension: linked-view calibration dashboard (class-conditional reliability diagrams + PIT-style views + drill-down to light curves). Evaluation: ECE and calibration slices vs the published class-specific metrics. Falsification: RAIL/LADaR cover photo-z, and Zhou et al. cover histories; none provide an interactive light-curve-linked calibration slice explorer as far as I found (queries: "photometric redshift PDF calibration ...", "PLAsTiCC ELAsTiCC light curve classification transformer", VA query). Risk: needs a trained classifier (LightGBM on hand-crafted features is fine); the PLAsTiCC test set is huge (use the subset).
- *Higher-upside RQ3b:* "Do calibration failures of photo-z PDFs cluster in feature space (LADaR-style local diagnostics) and can a projection-based view expose them?" Extra risk: photo-z needs training data prep (spec-z crossmatch) and astro-domain knowledge; direct competition with LADaR/CalPIT authors.

**Feasibility:** Med (domain jargon and data plumbing). Compute is trivial for tabular/LightGBM; 1D CNN/transformer on subsets is fine.
**Publication path:** VisXAI/ML4PS workshop, Astronomy and Computing; IEEE VIS unlikely.
**Red flags:** Large simulated data with heavy class imbalance (evaluation choices dominate); domain knowledge onboarding; simulated not real data (limits scientific credibility).

---

## 4. Triage table

| Sub-area | Best RQ | Novelty evidence | Feasibility | Grade-safety | Pub upside | Deep dive? | Why |
|---|---|---|---|---|---|---|---|
| (a) FM embeddings / anomalies | RQ1a: stability of neighbourhoods + top-k anomalies across encoders and projections (Zoobot/AstroCLIP/AstroDINO), GZ votes as reference | No stability/projection-audit study found; but SAE, probes, anomaly + active learning all have 2025-26 papers | Med-High | Med-High | Med | maybe (yes if paired with b) | Fits "Dim. reduction" and "Clustering" weeks; TopoMap++ hook; risk from fast-moving competitors |
| (b) Morphology / vote-uncertainty / XAI | RQ2a: model uncertainty vs volunteer disagreement with a volunteer-resampling noise floor; RQ2b: attribution agreement gated by disagreement | Uncertainty side saturated; disagreement-gated attribution faithfulness not found | High (2a), Med (2b) | High | Med | **yes** | Cleanest reproduction target (Zoobot, public weights/data), built-in human-disagreement labels, directly tied to MOUNTAINEER/Visagreement |
| (c) Time-domain / photo-z | RQ3a: calibration slice explorer for PLAsTiCC/ELAsTiCC classifiers | PIT tools mature; VA layer thin; Zhou et al. partial overlap | Med | Med | Low-Med | maybe | Time-series week fits, but domain overhead and overlap with Malz-group work |

## 5. Bottom line

Recommend a deep dive on (b), possibly merged with (a) into one project: "Zoobot / AstroCLIP embeddings and attributions audited against Galaxy Zoo volunteer disagreement." Biggest red flag: novelty vs speed of the astro-ML community (Walmsley/Wu SAE work, UniverseTBD AstroPT probes appeared in the last 12 months), and physical-faithfulness ground truth is hard to obtain without an astronomer collaborator. Unverified/not covered: Silva lab's older astronomy visualization work (OpenSpace) at paper level, Stein's tool liveness, GZ2 Kaggle status, ELAsTiCC Kaggle status.
