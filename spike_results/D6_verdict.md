# D6: Blunt verdict on round 4

## R1–R4 (confirmatory, Holm m = 4)

| | Verdict | One line |
|---|---|---|
| **R1** {f, o} beats κ at predicting recall@5% | **Not supported.** Untestable on native N1; **inconclusive** on N1-rescaled (+0.05 [−0.02, 0.17], bar 0.10) | Mixed on every dataset tried: PT supported, LC200 falsified, N2 inconclusive, N1-rescaled inconclusive. Raw recall@5% saturates at its ceiling whenever the target's error rate is ≫ 5% |
| **R2** family effect is margin-free | **Falsified** on N1-rescaled (−0.06 [−0.28, 0.15]); untestable on native N1 | Null under every control: accuracy-matched, matched operating point (R7), external-consensus strata (R8), v1. Held on LC200, N2 and PT (κ) |
| **R3** shared SAM ViT-L checkpoint raises dependence | **Not testable**: µSAM ViT-L excluded on N1-rescaled (error 0.604); Cellpose-SAM excluded on native N1 | Exploratory indications go the *other* way: accuracy-matched −0.39 [−0.72, −0.10]; matched operating point −0.26 [−0.40, −0.11] |
| **R4** shift decorrelates cross-architecture > seed pairs | **Falsified** (−0.57 [−0.72, −0.43]) | Same sign in all 3 folds, including SH-SY5Y, the only genuinely harder held-out type. Shift makes *seeds* less alike; cross-architecture dependence barely moves |

**Plainly: round 4 supported none of its four confirmatory hypotheses.**
- Two were falsified (R2, R4) and one could not be tested (R3).
- The fourth (R1) was untestable at native scale and inconclusive after rescaling.
- The biggest single cause was a **design failure in D0**: N1's cells are 50–327 px across, outside Cellpose's 7.5–120 px training range. The pre-registered "no rescaling" rule then excluded the main model.
  - D0 flagged this risk, but it chose not to gate on it.
  - Lesson for the proposal: the inclusion criteria for a held-out set must include a model-agnostic **scale check**, e.g. median GT diameter within the models' documented range. This can be checked from GT alone, before any model is run.

## R5–R8 (secondary, one line each)
- **R5** (agreement + flow-error combination ≥ 0.02 over each alone): **not supported.** On N1-rescaled (fallback reference) and on N2, the combination ≈ Cellpose-SAM's own flow error (+0.007 and +0.002 AUROC). The model's own signal is as good as anything a reference adds, which repeats round 3's H4.
- **R6** (mean of ≥ 2 references beats the best single one): **supported.** On N2 as pre-registered: +0.028 AUROC, +0.033 recall@5%. On N1-rescaled as an exploratory fallback: +0.048 AUROC. A small but consistent practical win.
- **R7** (operating-point matching): done. `cellprob_threshold` 0.5 matches µSAM's error rate within 0.01. R2 stays null (−0.08 [−0.32, 0.13]).
- **R8** (external-consensus difficulty strata): R2 null on N1-rescaled (+0.14 [−0.09, 0.37]). On PT round-3 data the family effect also vanished under R8 (log-OR +0.06 [−0.40, 0.64]), while on LIVECell it survived.

## Does "what makes a good reference" survive as the headline?
**Not as stated.** The confirmatory R1 (raw recall@5%) failed to come out positive on any new data. But one piece of it is robust and should be kept, in this form:
- **Robust (secondary, but consistent across PT, LC200, N2 and N1-rescaled; within targets too).** "Reference accuracy and independence (f, o) predict how well agreement *ranks* a model's errors (AUROC) far better than error consistency κ does":
  - Δρ = +0.84, +0.36, +0.29 and +1.18;
  - within-target: +0.73, +0.29, +0.17 and +1.03.
- **Robust caveat.** "At a fixed inspection budget, reference choice stops mattering once the target's error rate exceeds the budget; there, κ ranks references as well as {f, o}."

**What the better-supported headline is (SYNTHESIS, built from confirmatory nulls plus consistent exploratory patterns):**
> *"Shared failures between cell segmenters are driven by shared training data and recipe, not by a shared foundation backbone. Agreement-based QC works through reference accuracy and independence, but the model's own flow signal is as informative, and budgeted triage saturates when models are weak."*

The evidence for each part:
1. **Same recipe, different backbone stays dependent** (exploratory, D3 §3). Cellpose recipe with DINOv3 instead of SAM: log-OR 4.95 (N2) and 4.55 (N1-rescaled), against 5.9/5.0 for Cellpose-SAM v1↔v2 and 2.65–3.2 for different recipes (cyto3, µSAM).
2. **Family effect gone on N1.** R2 was falsified on data neither model trained on. The effect lived where both models shared training data (LIVECell).
3. **No shared-checkpoint effect** (R3 exploratory, negative). **Shift does not separate architectures** (R4 falsified).
4. **Flow error ≈ best combination** (R5, H4).

For the proposal, pre-register (1) as the primary hypothesis on a new, scale-checked held-out set:

| Pair | Changes |
|---|---|
| Cellpose-SAM vs CellposeDINO | backbone only |
| µSAM ViT-B vs ViT-L | encoder size only |
| Cellpose-SAM vs cyto3 | recipe/data |

Prediction: log-OR(backbone-only) > log-OR(recipe/data) on accuracy-matched data.

## Surprises
1. **Round 3's "Cellpose-SAM" was v2 all along.** cellpose 4.2.1.1 defaults to `cpsam_v2`, so the "add v2" task was a no-op and v1 was the new model. Round-3 prose citing the v1 paper describes a model we did not run. v1 and v2 are near-copies anyway: log-OR 5.9 (N2) and 5.0 (N1).
2. **Rescaling hurt micro-SAM** (0.48 → 0.40 on N1), the one model that was scale-robust at native resolution. A single "fair" preprocessing does not exist across these models.
3. **CellposeDINO ≈ Cellpose-SAM in its errors**, despite a completely different foundation model (D3 §3).
4. **Shift makes seeds diverge, not architectures** (D4). Margin-free cross-architecture dependence is flat under shift.
5. **κ can invert the conclusion when error rates differ** (D4, BV2 fold): κ for cross-architecture pairs halves (0.52 → 0.25) while log-OR rises (2.94 → 3.48). Making log-OR co-primary was the right call.
6. **The family effect was data, not lineage.** It held strongly on LIVECell (OR 92 vs 23) and was null on mCellSeg.

## New oversights found (like round 3's ViT-L/ViT-B catch)
- **Cellpose-SAM version mix-up** (above). Fixed in Amendment 5.
- **Leakage map:** Cellpose-SAM used **504** NeurIPS22 training images, not 616 (616 is LynSec). Corrected in Amendment 5 and in the onboarding doc.
- **micro-SAM v3/v4 (the weights micro_sam 1.8.x downloads) use NeurIPS22 Tuning as *validation*.** The pre-registered fallback N1 would have been model-selection-exposed (D0).
- **micro-SAM's paper count of 1,151 NeurIPS images** (Supp. Table 1) may include Public-Test. That is unconfirmed. If true, round-3's "held-out test split" label for micro-SAM on Public-Test is too generous. Check before the proposal.
- **recall@5% has a hard ceiling of 0.05 / e_t.** Pooled across targets it measures the target's error rate more than reference quality. No one noticed this before the D1 dry run; Amendment 6 added the normalised and within-target variants before N1.
- **The pre-registered R5/R6 choice rule had no fallback** for "the N2-chosen reference is excluded on N1". Handled as exploratory here; the next pre-registration needs a rule.

## Remaining risks for the proposal (due Oct 20)
1. **Thin confirmatory record.** Round 4 confirmed nothing. The proposal must be framed around hypotheses that are pre-registered *now* (the recipe-vs-backbone contrast) and a new held-out set chosen with a GT-only scale check. It must not imply that R1–R3 were confirmed.
2. **Held-out data is running out.** NeurIPS22 Public-Test and LIVECell are used, and mCellSeg is used and scale-problematic. Candidates: the Xiong murine set (D0 runner-up; sparse, licence ambiguity) or rescaled mCellSeg with a *new* pre-registered contrast. The latter is defensible only for hypotheses not yet looked at on it.
3. **CellposeDINO is exploratory and its training data is undocumented**, like `cpsam_v2`'s. A backbone-swap claim rests on the assumption that both share the Cellpose data mix. The authors' benchmark scripts suggest so, but this is SPECULATION.
4. **R4's folds are weak shifts** (2 of 3 held-out types are easier). A stronger controlled shift needs other modalities or a more different held-out type.
5. **The scoop check (K7) still needs the manual Google Scholar "Cited by" pass.** RBQE (arXiv 2609.10495) is the closest prior work.
6. **Compute is fine.** This round used about 27 GPU-hours, including about 7.5 h lost to L4 timeouts last night. Everything runs in under 1 h per job on L40S.

---

## Lead-reviewer notes (2026-09-30, Mac session)

**1. The "robust secondary" result ({f, o} predicts AUROC far better than κ) is close to true by definition.**
- If agreement were a binary flag ("disagree / agree"), then for target t:
  - TPR = P(disagree | t wrong) ≈ 1 − o;
  - TNR = P(agree | t right) ≈ 1 − f;
  - so AUROC = (TPR + TNR)/2 ≈ **(2 − o − f)/2**.

  The real signal is continuous IoU agreement scored within images, which adds about 0.16–0.26 on top. But the ranking is mostly the identity.
- FACT (from `r4_tables/*_directed.csv`, confirmatory pairs): Spearman(AUROC, (2 − o − f)/2) is

  | | PT | LC200 | N2 | N1s |
  |---|---|---|---|---|
  | Spearman | 0.94 | 0.74 | 0.58 | 0.85 |
  | Pairs | 56 | 42 | 72 | 42 |

- So {f, o} "beating κ" for AUROC restates how AUROC is built. It is a useful **explanation** (it tells a user why one reference beats another), not an empirical **finding**. Do not headline it.
- What would still be a finding: the part of AUROC *not* explained by (2 − o − f)/2. That part comes from the continuous IoU agreement and the within-image scoring. It is small and unexamined.

**2. The pattern across rounds is the real warning sign.**
- Each round's exploratory data produced a new headline:
  - B1: family > encoder;
  - round 3: the reference trade-off;
  - round 4: recipe > backbone.
- Each confirmatory test on new data then failed or became untestable.
- That is the "garden of forking paths": with ~10 models and ~4 dependence measures there is always *some* interesting contrast in the exploratory data.
- **The DINO lead (recipe > backbone) has the same shape.**
  - It rests on 2 exploratory models with undocumented training data.
  - On N2, CellposeDINO-B (4.51) ≈ cyto3 (4.53), so the separation appears only on rescaled N1.
  - Treat its prior as low. **Do not commission round 5 to chase it before the proposal.**

**3. What is solid (pre-registered or replicated on ≥ 3 datasets):**
- seed/fine-tune copies are near-identical in errors (log-OR 5–8) everywhere;
- agreement-based QC ≈ the model's own flow-error signal (H4, R5: 4 datasets);
- averaging ≥ 2 references beats the best single one (R6: pre-registered on N2, plus N1s);
- **measurement pitfalls**, each with a concrete demonstration:
  - κ inverts conclusions under error-rate mismatch (BV2: κ halves while log-OR rises);
  - recall@budget has a ceiling of budget/error rate, and pooled comparisons mostly measure the target's error rate;
  - held-out sets need a scale check;
  - the version mix-ups (v1/v2, 504 vs 616).
- the family effect is real in-distribution and on the NeurIPS22 split, and **absent** on genuinely new data. That is a clean, honest finding about where "related models fail together" applies.

**Reviewer verdict.** Stop the hypothesis rounds. The project is now best framed around the **visual-analytics tool plus a pre-registered evaluation** of when agreement does and does not reveal segmentation errors, with the results above as its evidence base. See the onboarding §4.4 note.
