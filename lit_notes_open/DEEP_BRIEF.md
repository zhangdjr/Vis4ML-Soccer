# Deep-dive brief (shared by all deep-dive agents)

Read `SCAN_BRIEF.md` in this folder for the student's constraints and the lessons learned. Also read the scan file(s) named in your task. You are now doing the **deep, falsification-first review** of ONE candidate area. **Today is 2026-09-28.**

## Your job
Decide, with evidence, whether this area contains a **defensible research question** that a solo student can execute in about 40 hours, to get (1) a good course grade and (2) a plausible workshop or short paper.

## Requirements
1. **Try hard to falsify the gap.** Search both domain terms and general ML/VIS terms, on arXiv, OpenAlex, Semantic Scholar (rate-limited, so try once, then fall back), ACM/IEEE (VIS 2023–2026 programs, VIS workshop programs), CHI/UIST/IUI, and NeurIPS/ICLR/ICML workshops.
   - For the closest 3–5 papers: **read the full text** (arXiv HTML/PDF via WebFetch, or download with curl + `pdftotext` into `lit_notes_open/pdfs/`). Record their stated limitations and future work.
   - Check papers that **cite** them (OpenAlex `cites:` filter works: `api.openalex.org/works?filter=cites:W...`), plus 2025–2026 follow-ups.
   - Report the queries that returned nothing.
2. **Check the instructor lab exhaustively** for your area: Silva, Nonato, Miranda, Xenopoulos, Rulff, Guardieiro, Solunke, P. Silva, Barr, Bertini (ex-NYU). Use the OpenAlex author search: `api.openalex.org/authors?search=...`, then `works?filter=author.id:...`.
3. **Verify the reproduction target.** Check that the repo exists and is maintained, its license, its dependencies (read `requirements.txt` / `pyproject.toml` and flag stale pins), and whether pretrained weights or data are downloadable (use `curl -I`). **Do not** install large packages or download big files.
4. **Design the project concretely:**
   - The precise research question
   - Hypotheses
   - A minimum viable project and a stronger version
   - The dataset, with its exact subset and size
   - Models
   - Visualization: which views, and why they are scientifically necessary
   - Evaluation: metrics, baselines, controls, and the success/failure criteria
   - An hour budget summing to about 40 h, including write-up
   - The main failure modes, with mitigations
   - The publication path: what must be added after the course
5. **Label claims** FACT / AUTHOR CLAIM / SYNTHESIS / SPECULATION, and give each source an evidence level (FULL TEXT / ABSTRACT / SECONDHAND). Never fabricate. Write "unverified" when needed.
6. **WebSearch budget:** at most about 80 calls. Prefer APIs and WebFetch.

## Output
- **Where:** your file path is given in your task. Aim for 4,000–6,000 words.
- **Paper table columns:** Paper | Year | Venue | Status | RQ | Data (open?) | Method | Viz | Evaluation | Main finding | Limitation / future work | Code? | URL | Evidence level
- **Also include:**
  - A "gap verdict" section: open / partly addressed / already done, with evidence.
  - A "novelty confidence" explanation in words.
  - A 12b-style row: course fit | research upside | technical risk | data risk | visualization burden | evaluation clarity | best suited if…, each with a 1-line reason.
  - 3–5 "verify this yourself" checks.
- **Final message:** a summary of about 250 words, including your honest verdict and the single biggest risk.
