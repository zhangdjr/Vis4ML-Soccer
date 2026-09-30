# K7 scoop check, first pass (2026-09-30, Mac session, web search only)

**Verdict: no scoop found.** Nobody reports per-cell error consistency across a model-relatedness hierarchy for cell segmentation, or how reference choice affects agreement-based QC in microscopy.

This pass used Google-style web search only. It still needs a **manual Google Scholar pass**, because Scholar's "Cited by" pages block automated access. The user should do that before Oct 20 and again before Nov 3.

| Paper | What it does | Overlap with D5 | Action |
|---|---|---|---|
| **RBQE**, arXiv 2609.10495 (polyp segmentation) | Primary model + an independently trained "referee"; image-level agreement AUROC. Finds "independence alone is sufficient" (same-architecture new-seed referee, AUROC 0.923), and cross-architecture is better (SegFormer 0.960) | **Closest.** But it is **image-level**, reports **no referee accuracy and no accuracy–independence trade-off**, has no κ/OR, and is not microscopy | Cite as the main precedent. Our round-3 finding (a *more accurate, same-family* reference triages best at top 5%) partly contradicts its "more diverse is better" story. Good hook |
| **Archit & Pape**, MIDL 2026, arXiv 2603.17845 | Benchmarks CellPoseSAM, CellSAM, µSAM and SAM 1/2/3; new APG prompting | Accuracy only, no error consistency. Already on the reading list | Use for model lineage details and possible extra models (SAM2/3, APG) |
| **Correlated Errors in LLMs** (Kim et al., ICML 2025) | Across many LLMs, "more accurate models have more correlated errors" | Same phenomenon in another domain; supports the accuracy–independence tension | Cite in related work |
| **Harnessing Disagreement / "correlated agreement blindness"**, arXiv 2607.19899 (intrusion detection) | Stronger base learners → more error correlation → disagreement-triggered escalation misses failures | Same mechanism as our reference trade-off, tabular domain | Cite; our contribution is the per-object segmentation version with a VA tool |
| **Cross-Model Disagreement as a Label-Free Correctness Signal**, arXiv 2603.25450 (LLMs) | A verifier model's perplexity flags errors; AUROC 0.75 vs 0.59 for own entropy | LLM domain; cross-model > self-signal. In cells, round 3 found cross-model ≈ self-signal (H4) | Cite as a contrast |
| **Chen et al., NeurIPS 2021** ("Detecting errors and estimating accuracy on unlabeled data with self-training ensembles") | Disagreement works when the reference agrees where the target is right and disagrees where it is wrong | Theory for the trade-off | Cite; check whether their conditions give a closed-form prediction we can test |
| BISCUIT (F1000Research 14:1277) | The motivating tool; mean model disagreement | Motivation (Bankhead's review) | Already central |
| Cellpose-SAM **v2** (June 2026 release, ViT-L, fix for low-contrast regions) | New checkpoint | A new L1-style pair, and a model to include | Added to round 4 |

**Search terms used:**
- "error consistency" + segmentation + foundation models;
- per-cell Cohen's kappa + generalist cell segmentation;
- agreement-based QC + reference model choice;
- Cellpose-SAM / micro-SAM / CellSAM + disagreement + QC;
- BISCUIT + consensus;
- reference-model disagreement + correlated errors.

**For the manual pass (Google Scholar, "Cited by"):**
- RBQE;
- Geirhos 2020 (error consistency), restricted to 2026 + "segmentation";
- Gontijo-Lopes ICLR 2022;
- BISCUIT;
- Cellpose-SAM + "disagreement".
