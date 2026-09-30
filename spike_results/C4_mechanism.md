# Spike C4: mechanism, untangling "family" (H5 decoding, H6 data vs architecture)

**Compute:**
- Five cellpose-3 U-Nets trained **from scratch**, 600 epochs each. Four ran 16–17 min on an L40S; one ran 39 min on an L4. Total about 1.8 GPU-hours.
- Every job was under 1 h, so no approval was needed. The time was measured first with a 10-epoch test (2.7 s/epoch).
- micro-SAM AMG was tuned on LIVECell **train** only (40 images of subset A; 5 × 5 grid). Best: pred_iou 0.7, stability 0.8, train F1@0.5 = 0.40.

**Verdict:**
- **H5: partly supported.** Shared decoding is not enough to explain shared merge/split errors.
- **H6: supported on LIVECell, but the effect is small.** A disjoint data subset from the same distribution barely decorrelates errors.

## H5: decoding and error type
Per-type κ is computed from the IoU > 0.5 taxonomy. The miss indicator is status = miss; the merge/split indicator is status ∈ {merge, split}.

| Pair | What differs | PT κ miss | PT κ merge/split | LC200 κ miss | LC200 κ merge/split |
|---|---|---|---|---|---|
| FT seed pair | only data order | 0.88 | 0.88 | 0.87 | 0.88 |
| cpsam vs cyto3 | encoder (same flow decoding) | 0.29 | **0.25** [0.16, 0.34] | 0.70 | **0.68** |
| cyto3 vs livecell_cp3 | data | 0.01 | 0.39 | 0.76 | 0.75 |
| **micro-SAM AIS vs AMG** | **decoding only** (same encoder, same weights) | 0.13 | **0.35** [0.22, 0.44] | 0.18 | 0.22 (AMG excluded: accuracy 0.28) |
| cpsam vs micro-SAM AIS | encoder weights, decoder, data | 0.19 | **0.09** [0.06, 0.13] | 0.42 | 0.27 |
| cyto3 vs micro-SAM AIS | everything | 0.14 | 0.12 | 0.47 | 0.31 |

Pre-registered contrasts for merge/split (Public-Test):
- same-decoding (cpsam vs cyto3) − cross-family (cpsam vs micro-SAM): **+0.15 [0.06, 0.26]**. Supported.
- same-decoding (cpsam vs cyto3) − same-weights/different-decoding (AIS vs AMG): **−0.10 [−0.22, 0.06]**. Not supported: AIS vs AMG is, if anything, *more* consistent.

SYNTHESIS:
- Merge/split sharing is predicted by sharing **either** the decoder **or** the full weights, not by the decoding objective alone.
- On overall κ, micro-SAM AIS vs AMG (0.40 on Public-Test, κ/κ_max 0.68) is **more** consistent than Cellpose-SAM vs cyto3 (0.33). Sharing the *same fine-tuned weights* matters more than sharing a decoder.
- The shared *pretrained SAM backbone* with different fine-tuned weights (Cellpose-SAM vs micro-SAM) matters least.
- A "number of shared components" reading fits better than any single-component story.

## H6: data vs architecture, on our own from-scratch U-Nets (LIVECell)
**Design:**
- Subset A = B1's 200 train images; subset B = 200 disjoint train images. Both have 25 per cell type.
- Seeds set the torch initialisation and offset cellpose's per-epoch `np.random.seed`. Initial losses differ (2.68 / 2.73 / 2.77), which confirms different inits.
- U-Nets are evaluated with the GT-median diameter.
- LC200 accuracy: 0.711–0.718, comparable to cyto3 (0.724).

**κ on LC200:**

| Level | Pairs | κ |
|---|---|---|
| L0b fine-tune seeds (Cellpose-SAM, for reference) | 3 | 0.91 |
| **L0a from-scratch seeds**, same data | 4 | **0.856–0.863** (mean 0.860) |
| **L4c same architecture, disjoint data** | 6 | **0.823–0.828** (mean 0.825) |
| own U-Net vs cyto3 / livecell_cp3 (same architecture, much more data) | 10 | 0.80 |
| own U-Net vs Cellpose-SAM (same objective and decoding, different encoder) | 5 | 0.75 |
| own U-Net vs micro-SAM | 5 | 0.57 |

- **H6 supported:** same-data seeds − disjoint-data pairs = **+0.035 [0.030, 0.041]**.
- The "different data" level sits **above** every cross-architecture level (0.825 vs 0.75–0.80).
- From-scratch seeds (0.86) are less consistent than fine-tune seeds (0.91). Random initialisation adds real diversity, which matches Saxena's warning about data-order-only seeds.

**Public-Test** (descriptive; the U-Nets are LIVECell-only, accuracy 0.38–0.46):
- They fail *together*: κ 0.77–0.89 among themselves, and **0.73–0.84 with livecell_cp3**, the other LIVECell-only Cellpose-3 model.
- κ ≈ 0 with every generalist.
- Same-data vs disjoint-data: +0.004 [−0.006, 0.015]. No difference, because both subsets come from the same distribution.

SYNTHESIS:
- In-distribution, resampling the training data from the same distribution hardly changes *which* cells fail. Architecture and recipe change it more.
- Out of distribution, **the training distribution dominates**: all LIVECell-only models fail on the same cells.
- So "data vs architecture" depends on whether "different data" means a different sample or a different distribution. The full study should vary the *distribution*, e.g. train on 7 LIVECell cell types and hold out the 8th, as in `D5_onboarding.md` §6.1.
