# Spike A3: D3 LLM judges, MT-Bench agreement reproduction

**Date:** 2026-09-29 · **Machine:** BC Andromeda, CPU job on `short` (c039), 2 CPUs · **Wall time:** 1 s compute, about 10 min total.
**Verdict: step 1 PASS. Step 2 (Ollama throughput) PENDING:** it needs the GPU box, per the 2026-09-29 brief update.

## Step 1: exact reproduction
- Data: HF `lmsys/mt_bench_human_judgments` (CC-BY-4.0), 2 parquet files. `human` has 3,355 votes (judges `expert_*`, `author_*`); `gpt4_pair` has 2,400 votes (judge `gpt4_pair`).
- Code: `~/vis4ml_spikes/work/a3/a3_mtbench.py`, SLURM job 3073098. It is a pandas port of FastChat `fastchat/llm_judge/compute_agreement.py`, with the logic copied verbatim. FastChat was **not** installed.
  - One adaptation: on HF, the GPT-4 judge field is the string `"gpt4_pair"`, not FastChat's `["gpt-4", "pair-v2"]` list. `get_judge_name` accepts both.
  - As in FastChat, `author_*` votes are not counted as "human".
- S1 = ties count as votes; S2 = ties excluded.

| Judges | Turn | Setup | Agree / total | Ours | Paper | Δ (pp) | ±1 pp |
|---|---|---|---|---|---|---|---|
| GPT-4 pair vs human | 1 | S1 | 886 / 1343 | 66.0% | 66% | −0.03 | ✅ |
| GPT-4 pair vs human | 1 | S2 | 727 / 859 | 84.6% | 85% | −0.37 | ✅ |
| human vs human | 1 | S1 | 454 / 721 | 63.0% | 63% | −0.03 | ✅ |
| human vs human | 1 | S2 | 388 / 479 | 81.0% | 81% | 0.00 | ✅ |
| GPT-4 pair vs human | 2 | S1 / S2 | 871/1325, 731/864 | 65.7% / 84.6% | – | – | – |
| human vs human | 2 | S1 / S2 | 471/707, 388/474 | 66.6% / 81.9% | – | – | – |

FACT: the paper's 66/85/63/81 numbers are **turn-1** numbers; turn 2 is shown for completeness.

## Step 2a: throughput on the lab server cscigpu08 (qwen3.6:27b only; gpt-oss:20b not installed there)
- Server: the lab Ollama server at **`http://cscigpu08.bc.edu:11434`** (Ollama 0.32.1). It is reachable directly from the Andromeda login node, which is inside the BC network, so no VPN is needed from the cluster. The calls are just HTTP; all compute runs on cscigpu08.
- Installed models: `qwen3.6:27b` (17.4 GB, Q4_K_M), `qwen3:4b`, `qwen3.5:4b`, `qwen3-embedding:8b`. **`gpt-oss:20b` is not installed.** The brief's "qwen3.8:27b" is really `qwen3.6:27b` (the name used in the user's PyTC chatbot).
- Prompt: FastChat `pair-v2` (system prompt + template from `fastchat/llm_judge/data/judge_prompts.jsonl`), filled with 2 real turn-1 MT-Bench cells whose prompts are about 1,500 tokens. Each cell ran in both positions.
- Settings: `/api/chat`, `think: false`, temperature 0, `num_ctx` 8192. Script: `~/vis4ml_spikes/work/a3/a3_throughput.py`; output `work/a3/out/throughput.csv`.

| Cell | Position | Prompt tokens | Output tokens | Prompt eval tok/s | Generation tok/s | Load (s) | Total (s) | Verdict (GPT-4 said) |
|---|---|---|---|---|---|---|---|---|
| q81 gpt-4 vs claude-v1 | orig | 1427 | 313 | 622 | 25.7 | **67.0** (cold) | 81.5 | A (A) |
| q81 | swap | 1427 | 238 | 629 | 25.4 | 0.9 | 12.9 | B (A), consistent |
| q145 llama-13b vs alpaca-13b | orig | 1230 | 582 | 626 | 25.3 | 0.8 | 26.0 | A (B) |
| q145 | swap | 1230 | 1075 | 758 | 25.0 | 0.8 | 45.7 | B (B), position-consistent, disagrees with GPT-4 |

- Resident size (`/api/ps`): 18.4 GB, all in VRAM, at ctx 8192.
- I could not run `nvidia-smi` on cscigpu08 (no shell there), so whether both judges fit at once is unverified. SYNTHESIS: 18.4 GB plus about 14 GB for gpt-oss:20b is about 32 GB, which plausibly fits in the box's 4×11 GB if nobody else is using it.
- **Warm cost is set by output length:** about 25 tok/s generation, while prompt eval is about 0.2 s per 100 tokens.

## Step 2b: throughput on an Andromeda L40S, user-space Ollama (both judges) — DONE
- Setup (SLURM 3073319, CPU):
  - Ollama **v0.34.4** user-space tarball (`~/vis4ml_spikes/ollama/`: bin + lib 2.1 GB).
  - `ollama pull qwen3.6:27b` (17 GB) and `gpt-oss:20b` (13 GB) into `ollama/models`. Total 32 GB on /projects; the pull took about 6 min.
- Timing (SLURM 3073320): `gtml` grs001, **L40S 46 GB**. Settings: `OLLAMA_NUM_PARALLEL=4`, `MAX_LOADED_MODELS=2`, `KEEP_ALIVE=-1`, ctx 8192. Script `work/a3/a3_throughput_l40s.py`; outputs `work/a3/out/throughput_l40s.{csv,_summary.json}`.
  - Phase 1: the same 2 cells × 2 positions as the cscigpu08 run, sequential.
  - Phase 2: 8 cells × 2 positions = 16 calls with 4 in flight.
- **Both judges resident at once: yes.** nvidia-smi showed 32.4 / 46.1 GB used.

| | qwen3.6:27b | gpt-oss:20b (`think: "low"`) |
|---|---|---|
| Prompt eval | about 1,500–8,500 tok/s (prefix cache inflates it) | about 7,000+ tok/s |
| Generation, one request | **about 70 tok/s** (2.8× cscigpu08's 25) | 66–86 tok/s |
| Parallel requests | **not supported**: Ollama logs "model architecture does not currently support parallel requests" (arch `qwen35`, a hybrid SSM / linear-attention model), so it runs serially | works: about 38 tok/s each, **113 tok/s aggregate** at 4 in flight |
| Output tokens per call (16 calls), median (max) | 332 (1,700) | 67 (208) |
| Effective s per call (phase 2) | **8.0** | **0.67** |
| Hours for 1,922 calls (one judge's share of 3,844) | **4.3 h** | **0.4 h** |

- **Minimum run (3,844 calls, 2 judges × position swap): about 4.7 h on one L40S**, against about 30 h for Qwen alone on cscigpu08. It fits in a single `gtml` job, which has no time limit, and would need your OK for a job over 1 h.
- Findings to keep (FACT, n = 2 cells):
  - Both judges were position-consistent on both cells.
  - On q145, both disagreed with GPT-4, but n = 2 means nothing yet.
  - **Qwen's output for the same prompt differs across Ollama versions and hardware**: q145 gave 1,700 tokens here vs 582 / 1,075 on cscigpu08, with the same verdicts. So **pin the Ollama version, quantization and hardware for the real run**, and cap `num_predict` at about 1,024 to bound runaway rationales.
  - gpt-oss's rationales are much terser (median 67 tokens). Judge "effort" differs by model; note this when decomposing disagreement.

## Step 3: hour estimate
- **Recommended setup: one L40S running user-space Ollama with both judges, about 4.7 h** for the 3,844-call minimum run (Step 2b). The earlier cscigpu08 numbers are kept below for reference.
- **qwen3.6:27b alone:** warm mean 28.2 s/call → 3,844 calls ≈ **30 h sequential**, for one judge.
- **Two-judge minimum run** (1,922 calls each): about 15 h for Qwen, plus gpt-oss:20b, whose speed is unmeasured (likely faster, as a MoE with about 3.6B active parameters; SPECULATION). Expect about 20–25 h in total on the shared box.
- Ways to cut it (SYNTHESIS):
  - (a) Cap `num_predict` at about 300 and ask for a one-sentence rationale. Generation is about 90% of the time, so this gives roughly 2–3× faster calls, but it changes the judge protocol.
  - (b) Run parallel requests (needs `OLLAMA_NUM_PARALLEL` on the server; not ours to change).
  - (c) Serve the models with vLLM on an Andromeda L40S (46 GB) under SLURM. That is batched and likely much faster, but it needs about 30 GB of weights plus approval.
