# D2: New models (micro-SAM `vit_l_lm`, Cellpose-SAM v1 vs v2, CellposeDINO), encoder fact-check, runs on N1 and N2

## 1. The Cellpose-SAM version (FACT; the main oversight this round)
- The brief said "add Cellpose-SAM v2 (the June 2026 release)". The installed code shows **it was already there**:
  - cellpose **4.2.1.1** has been installed in `envs/d5` since round 3;
  - `cellpose/models.py` sets `MODEL_NAMES = ["cpsam_v2", "cpdino", "cpdino-vitb", "cpsam"]` and `CellposeModel(pretrained_model="cpsam_v2")`.
- So **every round-3 result labelled "Cellpose-SAM" (`cpsam`) is v2.** The April 2025 v1 is the checkpoint named `cpsam`. In round 4 it runs as `cpsam_v1`.
- The two checkpoints differ: 1,233,586,851 B vs 1,233,587,898 B; the sha256 of the first 64 MiB is `8c23ee11…` (v2) vs `7bb51d78…` (v1).
- v2's only documented change is "a fix in the training for low contrast regions" (cellpose docs). Its training data is undocumented; see D0 §3.2.
- **Consequences:**
  - round-3 prose that cites the Cellpose-SAM v1 paper for "Cellpose-SAM" is describing a model we did not run;
  - PREREG Amendment 5 fixes "Cellpose-SAM" = v2 in Part B, with v1 as a sensitivity analysis;
  - `envs/d5` did not need changing.
- **CellposeDINO** needed the `dinov3` package, and only its code: the weights are inside the cellpose checkpoint and `pretrained=False` is used.
  - It is installed with `--no-deps` into a **separate overlay**, `envs/dinov3_overlay`, from facebookresearch/dinov3 commit `6876159a`. `envs/d5` is untouched.
  - It is **exploratory** (Amendment 5, item 2).

## 2. Encoder fact-check (FACT, from the installed code)
Script: `~/vis4ml_spikes/work/r4/d2/d2_encoders.py`. Output: `work/r4/d2/encoders_{cp4,cp3,msam}.json`. Each model was instantiated on CPU. The encoder class, width and depth were read from the module itself; the init-weights source is taken from the code or the download URL.

| Model (code name) | Package / version | Encoder (class) | Width × depth | Patch / stride | Encoder params | Pretrained init of the encoder | Checkpoint |
|---|---|---|---|---|---|---|---|
| Cellpose-SAM **v2** (`cpsam`) | cellpose 4.2.1.1, `CPSAM` | SAM `ImageEncoderViT` (**ViT-L**) | 1024 × 24 | 8 / 8 (re-strided from SAM's 16) | 304.5 M | SAM ViT-L `sam_vit_l_0b3195.pth` (SA-1B; named in `vit.py`) | `cpsam_v2`, 1.23 GB |
| Cellpose-SAM **v1** (`cpsam_v1`) | same | same (**ViT-L**) | 1024 × 24 | 8 / 8 | 304.5 M | same | `cpsam`, 1.23 GB |
| CellposeDINO-L (`cpdino`, exploratory) | cellpose 4.2.1.1 + dinov3 | `DinoVisionTransformer` (**DINOv3 ViT-L/16**) | 1024 × 24 | 16 / **8** | 303.2 M | DINOv3 `dinov3_vitl16_pretrain_lvd1689m` (LVD-1689M natural images; named in `vit.py`) | `cpdino`, 1.21 GB |
| CellposeDINO-B (`cpdino_b`, exploratory) | same | DINOv3 **ViT-B/16** | 768 × 12 | 16 / 8 | 85.7 M | `dinov3_vitb16_pretrain_lvd1689m` | `cpdino-vitb`, 0.34 GB |
| cyto3 | cellpose 3 overlay, `CPnet` | **CNN U-Net**, no attention layers | nbase [2, 32, 64, 128, 256] | – | 6.6 M (whole net) | none (trained from scratch on 9 datasets) | 26.6 MB |
| livecell_cp3 | same | CNN U-Net | same | – | 6.6 M | none | 26.6 MB |
| micro-SAM `vit_b_lm` (`msam_ais`) | micro_sam 1.8.14 | SAM `ImageEncoderViT` (**ViT-B**) | 768 × 12 | 16 / 16 | 89.7 M | SAM ViT-B, fine-tuned on the 14-dataset LM v4 mix | BioImage.IO `diplomatic-bug/1.2` (v4), 375 MB |
| micro-SAM `vit_l_lm` (`msam_ais_l`) | micro_sam 1.8.14 | SAM `ImageEncoderViT` (**ViT-L**) | 1024 × 24 | 16 / 16 | 308.3 M | SAM ViT-L, fine-tuned on the same mix | `idealistic-rat/1.2` (v4), 1.25 GB |
| CellSAM (`cellsam`) | cellSAM 0.0.dev1, weights v1.2 | SAM ViT-B (`sam_model_registry["vit_b"]` is the only call in `sam_inference.py`) + AnchorDETR prompter (6 encoder / 6 decoder layers, hidden 256) | 768 × 12 | 16 / 16 | – | SAM ViT-B, fine-tuned | `cellsam_general.pt` (gated) |
| **R4 init:** vanilla SAM `vit_b` (`msamv_*`) | micro_sam 1.8.14 `model_type="vit_b"` | SAM ViT-B | 768 × 12 | 16 / 16 | 89.7 M | **SA-1B only.** URL `dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth`; no microscopy, so no LIVECell | 375 MB |

**R3's contrast, checked:**
- Cellpose-SAM and micro-SAM `vit_l_lm` both start from **SAM ViT-L** (same architecture, same SA-1B checkpoint family). Cellpose-SAM re-strides the patch embedding to 8 and discards SAM's decoder.
- Cellpose-SAM and micro-SAM `vit_b_lm` share only the SAM pretraining family (ViT-L vs ViT-B).
- So R3 isolates "same encoder checkpoint family" on the encoder side. The two models still differ in decoder, objective and fine-tuning data (SYNTHESIS).

**R4's clean second architecture, checked:**
- The micro-SAM training was initialised from the vanilla SAM ViT-B URL above (FACT, `r4_msam_train.py`: `model_type="vit_b"`), not from a `*_lm` checkpoint.
- SA-1B consists of natural images (SAM paper). SYNTHESIS: no LIVECell exposure.

## 3. Runs on N2 (FACT)
Mean instances per image and the median seconds per image are in `work/c1/out/n2/timing_*.json`. Accuracy is the share of N2 GT cells matched at IoU > 0.5, over 200 images and 65,243 cells.

| Model | Role | N2 accuracy | Mean instances / image | Median s / image (L40S/A10/L4) | Images with 0 predictions |
|---|---|---|---|---|---|
| `cpsam` | roster (Cellpose-SAM v2) | 0.722 | 263.1 | 0.517 | 0 |
| `cpsam_v1` | roster (new; v1) | 0.721 | 261.5 | 0.516 | 0 |
| `cpsam_ft_s1` | roster | 0.729 | 268.0 | 0.471 | 0 |
| `cpsam_ft_s2` | roster | 0.719 | 262.1 | 0.473 | 0 |
| `cpsam_ft_s3` | roster | 0.726 | 266.1 | 0.473 | 0 |
| `cpsam_ft0` | QC-signal variant (flow_threshold=0) | 0.796 | 328.1 | 0.47 | 0 |
| `cyto3` | roster | 0.694 | 250.3 | 0.945 | 0 |
| `cyto3_gtd` | K3 variant | 0.706 | 255.0 | 0.327 | 0 |
| `livecell_cp3` | roster | 0.705 | 252.6 | 0.318 | 0 |
| `livecell_cp3_gtd` | K3 variant | 0.713 | 256.0 | 0.334 | 0 |
| `msam_ais` | roster | 0.549 | 226.3 | 0.275 | 0 |
| `msam_ais_l` | roster (new; R3) | 0.578 | 232.1 | 0.336 | 0 |
| `cellsam` | roster; **excluded on N2** (error 0.72) | 0.278 | 118.2 | 0.919 | 1 |
| `cpdino` | exploratory (new) | 0.716 | 260.3 | 0.537 | 0 |
| `cpdino_b` | exploratory (new) | 0.703 | 257.8 | 0.249 | 0 |

GT: 326 cells per image on average (65,243 / 200). The 18 R4 fold models are in D4.

## 4. Runs on N1 (mCellSeg, 198 images, 15,975 cells) (FACT)
| Model | Native accuracy | Excluded (native)? | Rescaled accuracy (Amendment 7) | Excluded (rescaled)? | Mean instances / image (native) |
|---|---|---|---|---|---|
| `cpsam` | 0.348 | yes | 0.442 | no | 44.9 |
| `cpsam_v1` | 0.357 | yes | 0.471 | no | 45.9 |
| `cpsam_ft_s1` | 0.388 | yes | 0.528 | no | 51.6 |
| `cpsam_ft_s2` | 0.367 | yes | 0.507 | no | 50.5 |
| `cpsam_ft_s3` | 0.351 | yes | 0.502 | no | 47.8 |
| `cpsam_ft0` | 0.363 | yes | 0.461 | no | 162.9 |
| `cyto3` | 0.239 | yes | 0.409 | no | 59.8 |
| `cyto3_gtd` | 0.398 | yes | 0.454 | no | 44.2 |
| `livecell_cp3` | 0.104 | yes | 0.160 | yes | 13.0 |
| `livecell_cp3_gtd` | 0.182 | yes | 0.177 | yes | 18.3 |
| `msam_ais` | 0.482 | no | 0.405 | no | 70.2 |
| `msam_ais_l` | 0.527 | no | 0.396 | yes | 70.9 |
| `cellsam` | 0.333 | yes | 0.337 | yes | 62.9 |
| `cpdino` | 0.412 | no | 0.506 | no | 51.6 |
| `cpdino_b` | 0.353 | yes | 0.475 | no | 53.1 |
| `msam_amg` | – | – | 0.348 | yes | – |

GT: 80.7 cells per image on average. `msam_amg` was not run at native scale: it exceeded 2 h on the 2796² images, it is not in Part B, and the job was cancelled; it ran on the rescaled images. The `_gtd`, `ft0` and AMG rows are variants used only in their named tests. Native-scale CellSAM ran separately after that cancellation, with identical code. The full analysis of the scale failure is in D3 §1.
