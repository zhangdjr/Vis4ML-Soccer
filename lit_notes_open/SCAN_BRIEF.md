# Broad-scan brief (shared by all scan agents)

You are doing a **cheap, skeptical broad scan** of one slice of possible research areas for a course project. **Today is 2026-09-28.** Your job is triage, not a full review.

## The student and constraints
- NYU MS Data Science student, **solo**.
- Course: DS-GA 3001 *Visualization for Machine Learning*, taught by Prof. **Claudio Silva** (NYU VIDA; frequent co-authors are L. G. Nonato and Fabio Miranda).
- **About 40 hours total** (3–4 h/week).
- Milestones: 4-page proposal due **Oct 20, 2026**; 1-page update Nov 3; final 8-page report plus presentation Dec 1–14.
- ~~The course requires the student to **reproduce prior work AND extend it**~~. **Corrected 2026-09-29:** the syllabus says "reproduce prior work **or** implement a proposed research idea", then "demonstrate both the prior work, and your final research project". Reproduction is optional; a small one is still recommended as validation.
- Priorities, in order:
  1. A good grade, meaning feasible and low-risk.
  2. A plausible path to a workshop or conference paper for PhD applications.
- **Skills:** strong ML, PyTorch and CV (they have done segmentation before); moderate NLP/LLM and statistics; **weak JS/D3 and frontend**. Prefer Jupyter, Streamlit or Plotly linked views.
- **Compute:**
  - MacBook M1 Pro, 32 GB
  - SLURM HPC cluster
  - A machine with 4× RTX GPUs of 11 GB each (Ollama only so far; can run a ~27B Qwen model as a black box; activation-level work is realistic only for ≤2–4B models)
  - Local `gpt-oss-20b`
  - Maybe no paid LLM API
- **Data must be openly downloadable.** No credentialed or NDA data.

## Course topics (lecture dates)
| Date | Topic |
|---|---|
| Sept 22 | Model assessment |
| Sept 29 | White-box |
| Oct 6 | Black-box interpretation |
| Oct 13 | Clustering |
| Oct 20 | Dimensionality reduction |
| Oct 27 | DL visualization |
| Nov 3 | NLP/LLM visualization |
| Nov 10 | Topological data analysis |
| Nov 17 | Time series |
| Nov 24 | Interpretable ML and fairness |

## Lessons from a previous (soccer) review. Apply these.
- **The instructor's own lab had already published the obvious idea.** MOUNTAINEER (TVCG) and Visagreement (TVCG 2025) are VA tools for comparing and evaluating feature attributions. SUBPLEX and Calibrate are also theirs.
  - **For every area, first check Silva, Nonato, Miranda and VIDA-NYU publications for overlap**: Google Scholar, dblp and ctsilva.github.io/publications.
  - Overlap is not fatal. It can make a great reproduction target. But it must be known.
- Generic "apply known XAI/DR method to a new domain" has weak novelty. Look for a **specific open question stated or implied by a named recent paper**.
- Tools:
  - Semantic Scholar rate-limits heavily. Use OpenAlex (`api.openalex.org/works?search=...`), the arXiv API (`export.arxiv.org/api/query?search_query=...`) and Crossref as fallbacks.
  - **The WebSearch budget is shared by all agents. Use at most about 45 WebSearch calls.** Prefer WebFetch on APIs.
  - Use Bash (`gh api repos/...`, `curl -I`) to verify that repos and datasets exist.
- Never fabricate papers, authors, venues, numbers or URLs. If unverified, write "unverified".

## Extra lessons from round 2 (apply these too)

- **Search the instructor's publication list first, directly.** The broad scan missed two things that later changed conclusions:
  - mTSeer (CHI 2021, Silva and Bertini) in time series;
  - a prior student project in the same course.

  So for your domain, check both sources: OpenAlex author sweeps (`api.openalex.org/authors?search=Claudio Silva`, then `works?filter=author.id:...,from_publication_date:2019-01-01`), and a web search for "Claudio Silva" together with the domain terms.
- **Check for prior cohorts of this course.** Run `gh search repos <domain keywords>` and look for student projects created around **Dec 2025** (last year's final deadline) that mention VisML, DS-GA 3001, or Silva.
- **Give two RQs per sub-area:**
  - a **grade-safe** RQ: low risk, with a clean reproduction target;
  - a **higher-upside** RQ: what would make it a main-venue or strong-workshop paper, and what extra risk it adds.

## What to produce
Write one markdown file (path given in your task), **at most about 3,000 words**. Include:

1. **One "area card" per sub-area** in your slice, with:
   - **Instructor-lab overlap:** list any Silva/Nonato/Miranda/VIDA papers (FACT/unverified).
   - **Key 2023–2026 papers:** 5–10, one line each: citation, venue, status (peer-reviewed / preprint), what it does, and whether code/data are open.
   - **Saturation verdict:** saturated / active / thin, with the evidence.
   - **Open data:** verified dataset names, license, size, URL, and how you verified it.
   - **1–3 candidate research questions.** Each needs a *reproduction target* (a specific paper with public code), a *minimal extension*, an *evaluation plan*, and a **falsification check**: did someone already do it? Name the queries you ran.
   - **Feasibility in about 40 h** for this student: data, compute, and front-end burden.
   - **Publication path:** plausible venues (e.g. VIS short paper, a workshop).
   - **Red flags:** saturation, subjective evaluation, heavy engineering, or dependence on non-open data.
2. **A final triage table:** sub-area | best RQ | novelty evidence (1 line) | feasibility (Low/Med/High) | grade-safety (Low/Med/High) | publication upside (Low/Med/High) | recommend deep dive? (yes/maybe/no) | why.
3. **Labels:** tag claims FACT / AUTHOR CLAIM / SYNTHESIS / SPECULATION where it matters.

Your final message should be a summary of about 200 words. Say which 1–2 sub-areas deserve a deep dive and why, and give the single biggest red flag.
