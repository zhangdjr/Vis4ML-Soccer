# Next-session tasks: feasibility spikes + verification checks

**For:** a fresh Claude Code session running **on the GPU box (4× RTX 11 GB) or on the HPC cluster.** It has no access to the previous session's memory, so everything it needs is in this file.
**Context to read first:**
- `research_gap_review.md`, §1, §5.1–5.3, §10 and §11.
- The deep dives are optional background: `lit_notes_open/deep_5_microscopy_qc.md`, `deep_2_tsfm_va.md`, `deep_4_astro_zoobot.md`, `deep_3_llm_judges.md`.

## ⚠️ Update 2026-09-29: running on the HPC cluster (GPU box unreachable)
- **Login node:** installs, downloads, CPU-only work (A2, A3 step 1, all of Part B). Keep it light; no heavy compute on the login node.
- **GPU work (A1, A4) runs inside a SLURM job,** e.g. `srun --gres=gpu:1 --time=2:00:00 --pty bash`. Ask the user for partition and account names. First test whether compute nodes have internet (`curl -I https://huggingface.co` from inside a job). If not, pre-download everything on the login node.
- **Skip A3 step 2 (Ollama throughput)** until the GPU box is reachable. Just note it as pending.
- Check how much GPU memory the assigned card has (`nvidia-smi`), and adjust batch or tile sizes to fit.

## Who the user is (short)
- NYU MSDS student doing a **solo** DS-GA 3001 (Visualization for ML, Prof. Claudio Silva) project.
- Proposal is due **2026-10-20**. The project discussion lecture is **2026-10-06**.
- Strong ML, PyTorch, CV and segmentation; moderate NLP.
- **Silva confirmed JS/D3 is not required**; Streamlit/Plotly is fine.
- Priorities: a good grade first, publication second. Uses **open data only.**
- Communication style: explain terms briefly the first time they come up. Be skeptical. Label claims FACT / AUTHOR CLAIM / SYNTHESIS / SPECULATION.

## Ground rules for this session
- **Work in one folder:** `~/vis4ml_spikes/`, with its own conda env(s). **Disk budget: about 10 GB** (weights plus data subsets). Report the final disk usage.
- **HPC only:** compute nodes may lack internet. Download on the login node, then run GPU jobs via SLURM (`srun --gres=gpu:1 ...`). Ask the user for partition and account names if needed.
- **Never download full datasets.** Use the subsets named below.
- **Ask the user before:** anything over 2 GB in a single download, installing system packages, or submitting SLURM jobs longer than 1 h.
- **Write results to `spike_results/*.md`** in the repo: one file per spike, with the commands run, numbers, pass/fail, and time spent. At the end, add a summary to `research_gap_review.md` §12 as a "Spike results" subsection, and update §10 if a spike fails.
- **Do not commit or push** unless the user asks.
- **Credits:** the user wants to use their remaining credits well. Prefer doing work directly over spawning many sub-agents. Use at most 2 sub-agents, and only for the literature checks in Part B.

---

## Part A: feasibility spikes (priority order)

### A1. D5 microscopy: can we run 2 generalist segmenters and score per-cell errors? (GPU box preferred)
1. Create env `d5`. `pip install cellpose` (v4, which includes Cellpose-SAM) and `micro_sam`. micro-SAM is often installed via conda-forge: `conda install -c conda-forge micro_sam`. Check its docs.
2. Get **5–10 LIVECell test images plus their annotations.** Do NOT pull the whole archive. LIVECell is on AWS S3 (`s3://livecell-dataset/`), licensed CC BY-NC 4.0. Find the test-split COCO json and fetch individual images if possible. Fallback: NeurIPS22 CellSeg **Public-Test** (Zenodo, CC-BY-NC-ND). Check its size first.
3. Run Cellpose-SAM and micro-SAM (automatic instance segmentation) on those images. Record wall time per image and GPU memory.
4. Match predicted instances to ground truth (GT) instances by IoU ≥ 0.5, using Hungarian matching or a greedy match. Compute per-cell IoU and counts of merge / split / miss / false positive (FP).
5. Compute **cross-model instance agreement.** For each GT cell, take the IoU between the two models' matched masks. Report the Spearman correlation between agreement and each model's per-cell IoU on these few images. This is just a smoke test, not a result.
6. **Pass criteria:** both models run; matching works; per-cell table produced; under 60 s per image on GPU.
7. **Also note:**
   - Does Cellpose expose flow-error or cell-probability per instance?
   - Does micro-SAM expose a predicted-IoU score per instance?
   - Is CellSAM usable? It needs a free **DeepCell API key**. If there isn't one, skip it and tell the user they would need to register.

### A2. D2 time-series foundation models: do the released outputs work, and is the row ordering right? (either machine; CPU is fine)
1. Env `d2` (Python 3.11): pandas, numpy, huggingface_hub.
2. From HF dataset `Real-TSF/TIME-Output`, download **only** two model folders: `seasonal_naive` and `chronos2` (names may differ; list the repo first). Also download the `features/` folder.
3. Read `github.com/zqiao11/TIME`: the saver and the feature scripts. **Verify how the series and variate indices in `metrics.npz` map to rows of `features/*.csv`.** The previous session *assumed* the orders matched, and that assumption is unverified.
4. Reproduce one of TIME's pattern-level numbers. For example, the Chronos-2 MASE ratio for high vs. low seasonal strength. Compare it against the paper (arXiv 2602.12147) or the HF leaderboard space `Real-TSF/TIME-leaderboard`.
5. Re-run the pilot with **CRPS as well as MASE**. The pilot asked two things:
   - What share of window-level log relative error variance lies within series?
   - On what % of windows do two TSFMs differ by more than 25%?

   Previous provisional values: about 66%, and about 32% (Chronos-2 vs TimesFM-2.5).
6. **Pass criteria:** row ordering confirmed (or corrected); one TIME number reproduced within about 2%; pilot numbers recomputed.

### A3. D3 LLM judges (GPU box only, where Ollama lives)
1. **Exact reproduction** (any machine): download HF `lmsys/mt_bench_human_judgments` (CC-BY-4.0, about 1.4 MB). Port FastChat's `fastchat/llm_judge/compute_agreement.py` logic (numpy only; do NOT install FastChat's `llm_judge` extra, which pins `openai<1`). **Pass:** GPT-4-pair vs human agreement 66% (setup S1) / 85% (setup S2), and human–human 63% / 81%, each within ±1 pp.
2. **Throughput:** with Ollama, run one real MT-Bench pairwise judge prompt (about 1,500 tokens) through `gpt-oss:20b` and `qwen3.8:27b`. Use `ollama run <model> --verbose`, or the API with timing. Turn thinking off for Qwen. Record prompt-eval tok/s, eval tok/s, and seconds per call. Check `nvidia-smi` to see whether both models can be resident at once.
3. Re-estimate hours for the minimum run: 961 cells × 2 judges × 1 prompt × position swap = 3,844 calls.

### A4. D4 astronomy (GPU box preferred): only if time and credits remain
1. Env `d4`: `pip install zoobot` (GPL-3.0; check the torch version). Load a pretrained encoder from HF, e.g. `mwalmsley/zoobot-encoder-convnext_nano`.
2. Download HF `mwalmsley/gz_desi` in its **small or tiny config only** (CC-BY-NC-SA-4.0, which requires releasing trained-model code if published).
3. Fine-tune a head for **one** question (e.g. "smooth or featured") for 1–2 epochs. Report the loss curve and time per epoch.
4. Download **one** GZ3D FITS file from `https://data.sdss.org/sas/dr17/manga/morphology/galaxyzoo3d/`, plus the matching Legacy Survey cutout by RA/Dec. Check that the bar/spiral mask aligns with the cutout: same center, and a sensible scale once pixel sizes are converted.
5. **Pass:** fine-tune runs, and the mask aligns.

### A5. D1 Tile2Net: only if the user asks
- Needs CUDA. Install `VIDA-NYU/tile2net` and run `examples/example.sh` (Boston).
- Check whether any `*_prob.png` files are written. The previous code reading says no.
- Estimate the patch needed to dump the softmax.

---

## Part B: verification checks (literature and data; any machine with internet)
Label findings FACT / SYNTHESIS. Append them to `research_gap_review.md` §11 as "Verified on <date>".

1. **dblp sweeps**, 2023–2026, for Claudio T. Silva, Luis Gustavo Nonato, Fabio Miranda, Brian Barr, Enrico Bertini. Try `https://dblp.org/search/publ/api?q=author%3A<Name>%3A&h=1000&format=json` via curl with a normal User-Agent (the web UI blocked bots). Grep the results for: cell, microscopy, segmentation, galaxy, astronomy, forecast, time series, judge, LLM, calibration. **Report any overlap** with D5, D2, D4 or D3.
2. **D5 scoop check.** Citers of BISCUIT (doi 10.12688/f1000research.171889.1) and MARC (arXiv 2609.13665), and any 2025–26 paper testing "correlated errors" between generalist segmenters. Also: was NeurIPS22 CellSeg *Tuning* used as micro-SAM's validation set? Check the micro-SAM paper and repo.
3. **D4 scoop check.** The ZooBot:3D (2026) paper: does it analyze attributions vs. masks? Also Walmsley / Masters / Spindler 2025–26 papers, including the SAE-on-Zoobot work.
4. **D2 scoop check.** Citers of TIME (arXiv 2602.12147) and Jander et al. (arXiv 2608.24303): has anyone done window-level failure analysis or in-vivo validation?
5. **License recheck** for anything the project would publish under: LIVECell, NeurIPS22 CellSeg, GZ DESI (its code-release clause), TIME data.

## Things only the user can do (list them at the end if still open)
- ~~Visagreement full text~~: done 2026-09-29. Takeaways are in `research_gap_review.md` §13.2.
- ~~Get Cefis & Carpita 2024~~: done 2026-09-29. Global SHAP/RGE rank concordance only; C1's local gap survives (`research_gap_review.md` §11).
- **Ask Silva or the TA, only if D1 (Tile2Net) is chosen:** which NYC areas Tile2Net was trained on. Evaluating calibration on training areas would look falsely good.
- **Create a DeepCell API key**, if CellSAM is wanted for D5.
