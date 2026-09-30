# Round-4 frozen data specs (committed before any model is run on N2 or trained on N3)

- `N2_file_list.json`: 200 LIVECell **test** images, 25 per cell type, sampled with `numpy.random.default_rng(20260930)` from test images not in LC200 (which contains E40) and not among the duplicate files (Amendment 2). 65,865 annotations. Code: `~/vis4ml_spikes/work/r4/n2_data.py`.
- `N3_folds.json`: leave-one-cell-type-out folds for R4. Held-out types **SH-SY5Y, BV2, SKOV3** (chosen to span morphology: small neuronal-like, small round microglia, large flat epithelial). Each fold trains on 200 LIVECell **train** images of the other 7 types (4 types × 29 + 3 × 28), drawn from B1's train200 ∪ round-3 trainB, plus 14 validation images (2/type) for micro-SAM early stopping. Code: `work/r4/n3_folds.py`.
- Evaluation for R4: N2 images of the held-out type (25) vs N2 images of the 7 training types (175).
- LIVECell is CC BY-NC 4.0; only file names are committed here.
- `N2_choices_for_R5_R6.json` (added 2026-09-30, **before N1 was chosen or downloaded**): the best single reference, the top-2 references and the best flow-error variant for Cellpose-SAM, chosen on N2 by within-image AUROC (PREREG Amendment 5 item 5). R5/R6 on N1 use these choices.
