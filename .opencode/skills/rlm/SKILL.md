---
name: rlm
description: Run a Recursive Language Model-style loop for scientific literature analysis. Loads a corpus of PDF papers into the persistent Python REPL, chunks the content, and uses the rlm-subcall subagent for paper-by-paper extraction and synthesis.
---

# rlm (Scientific Literature RLM Workflow)

Use this Skill when:
- The user provides a folder of scientific PDF papers to analyse
- You need to answer a research question across a large corpus
- The combined text of the papers exceeds the LLM context window
- You need to extract evidence, citations, or findings systematically

## Mental Model

- Main OpenCode conversation = the root LM (orchestrator + synthesiser)
- Persistent Python REPL (`rlm_repl.py`) = environment for PDF extraction, state, and corpus management
- Subagent `rlm-subcall` = the sub-LM used for chunk-by-chunk analysis

```
┌─────────────────────────────────────────────────────────┐
│                 Root LLM (research-write)                │
│             Orchestration & Synthesis                    │
└──────────────────────┬──────────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
┌─────────────────┐         ┌─────────────────┐
│  Python REPL    │         │  rlm-subcall    │
│  (Environment)  │         │  (Sub-LM)       │
│                 │         │                 │
│ - Extract PDFs  │         │ - Analyse chunk │
│ - Manage corpus │         │ - Extract claims│
│ - Store results │         │ - Return JSON   │
└─────────────────┘         └─────────────────┘
```

## Inputs

This Skill accepts:
- `corpus=<path>` — path to a directory of PDFs (e.g. `corpus=./context/`)
- `paper=<path>` — path to a single PDF for deep-dive mode
- `query=<question>` — what the user wants to know
- Optional: `chunk_chars=<int>` (default 150000), `overlap_chars=<int>` (default 2000)

If arguments were not supplied, ask for:
1. The corpus directory or single paper path
2. The research question or analysis goal

## Step-by-Step Procedure

> ⚠️ **Critical: YOU (the root agent) run all bash/REPL steps directly.**
> Do NOT spawn a generic `Task` or subagent to run the `/rlm` command or REPL commands —
> that bypasses the REPL entirely and produces answers from the LLM's own knowledge, not the corpus.
> Only delegate to `@rlm-subcall` when you have a specific *chunk file path* to pass for content extraction.

### Mode A: Full Corpus (multiple PDFs)

1. **Load the corpus into the REPL**
   (Automatically resumes if interrupted — just re-run the same command)
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py load-corpus ./context/
   python .opencode/skills/rlm/scripts/rlm_repl.py status
   ```
   Per-paper chunk files are written to `.opencode/rlm_state/chunks/` automatically during this step.
   Each file is named `<bibtex_key>_chunk_NNNN.txt` (overlapping, 150 000 chars max by default).

2. **Orient yourself**
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(list_papers())"
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(stats())"
   ```

3. **Find relevant papers**
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(find_papers('climate'))"
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(cite('Rockström 2009'))"
   ```

4. **For broad queries — use pre-written chunk files and delegate to @rlm-subcall**

   Chunk files are already in `.opencode/rlm_state/chunks/` from step 1.
   List available chunks:
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "import os; print(sorted(os.listdir('.opencode/rlm_state/chunks'))[:20])"
   ```
   For each relevant chunk file, invoke `@rlm-subcall` with the file path + query.
   (You can filter to only chunks whose `bibtex_key` prefix matches papers found in step 3.)

   If chunks are missing (e.g. after a `reset`), regenerate them:
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "paths = write_chunks('.opencode/rlm_state/chunks'); print(len(paths), 'chunks')"
   ```

5. **Store intermediate results**
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "add_buffer(json_result)"
   ```

6. **Synthesise**
   Collect subagent JSON outputs and synthesise:
   - Group findings by theme
   - Identify consensus and disagreement
   - Draft LaTeX section with verified citations
   - Generate BibTeX entries from paper metadata

### Mode B: Single Paper Deep-Dive

1. **Load the paper**
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py pdf ./context/paper.pdf
   python .opencode/skills/rlm/scripts/rlm_repl.py status
   ```

2. **Inspect the paper**
   ```bash
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(peek(0, 3000))"
   python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(stats())"
   ```

3. **Use @paper-analyzer for structured extraction**
   Invoke `@paper-analyzer` with the extracted text or file path.

## Research-Specific REPL Helpers

```python
list_papers()              # List all loaded papers with indices
find_papers('keyword')     # Search by author/title keyword
cite('Smith 2020')         # Get passages mentioning a specific reference
search_claim('X causes Y') # Find evidence for/against a claim
extract_references()       # Parse reference lists from papers
get_paper(0)               # Get full text of paper at index 0
stats()                    # Corpus overview: papers, chars, top terms
```

## Guardrails

- Do NOT paste large raw chunks into the main chat
- Use the REPL to locate exact passages; quote only what you need
- Subagents cannot spawn other subagents
- NEVER fabricate citations — only report what the REPL returns
- Keep scratch files under `.opencode/rlm_state/`
- Clean up chunk files after analysis: `python rlm_repl.py reset`
