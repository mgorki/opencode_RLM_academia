# Scientific Research RLM Usage Guide

Ready-to-use examples for the scientific literature RLM workflow in OpenCode.

---

## Setup

Install the required Python dependency:

```bash
pip install pdfplumber
```

Place your PDF papers in the `context/` folder (any PDFs — any scientific field).

---

## Example 1: Load a Full Corpus and Ask a Research Question

**In OpenCode** (with `research-write` agent):

```
/rlm corpus=./context/ query=What are the main definitions of sustainability in the literature?
```

**Or manually:**

```bash
# 1. Load all PDFs
python .opencode/skills/rlm/scripts/rlm_repl.py load-corpus ./context/

# 2. Check status
python .opencode/skills/rlm/scripts/rlm_repl.py status

# 3. List papers
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(list_papers())"
```

Then in OpenCode:
> "Using the loaded corpus, what are the main definitions of sustainability?"

---

## Example 2: Write a Literature Review Section in LaTeX

```
/write-section the concept of planetary boundaries and Earth-system tipping points
```

Expected output:
```latex
\section{Planetary Boundaries}

The planetary boundaries framework, introduced by \textcite{rockstrom2009safe},
proposes a set of nine Earth-system processes within which humanity can safely
operate. \parencite{rockstrom2009planetary} define these boundaries as...

\printbibliography
```

```bibtex
@article{rockstrom2009safe,
  author  = {Rockström, Johan and Steffen, Will and ...},
  year    = {2009},
  title   = {A safe operating space for humanity},
  journal = {Nature},
  volume  = {461},
  pages   = {472--475},
  doi     = {10.1038/461472a},
}
```

---

## Example 3: Find Papers on a Specific Topic

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "
import json
results = find_papers('consensus')
print(json.dumps(results, indent=2))
"
```

---

## Example 4: Retrieve Passages About a Specific Citation

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "
results = cite('Brundtland 1987')
for r in results[:3]:
    print('---')
    print(r['context'][:400])
"
```

---

## Example 5: Search for Evidence Supporting a Claim

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "
results = search_claim('scientific consensus on climate change exceeds 97 percent')
for r in results[:3]:
    print('---')
    print(r['context'][:400])
"
```

---

## Example 6: Deep-Dive on a Single Paper

```
/deep-dive ./context/Rockström et al. - 2009 - A safe operating space for humanity.pdf
```

Or load it directly:

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py pdf "./context/Rockström et al. - 2009 - A safe operating space for humanity.pdf"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(peek(0, 3000))"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(stats())"
```

---

## Example 7: Generate BibTeX for Cited Papers

```
/bibtex Cook et al. 2013, Rockström et al. 2009, Brundtland 1987
```

Or use the REPL to extract reference lists:

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "
refs = extract_references(paper_index=0)  # from first paper
for r in refs[:10]:
    print(r)
"
```

---

## Example 8: Check Citations for Hallucinations

After writing a section, verify every citation:

```
/cite-check \parencite{cook2013quantifying} found that 97\% of climate scientists...
\textcite{rockstrom2009safe} proposed nine planetary boundaries...
\parencite{brundtland1987our} defined sustainable development as...
```

The agent will check each key against the loaded corpus and report:
- `VERIFIED` — paper found and claim traceable
- `NOT FOUND` — paper not in corpus
- `UNVERIFIABLE` — paper found but claim cannot be located

---

## Example 9: Large Corpus — Chunk and Delegate

For corpora too large for direct analysis:

```bash
# Create chunks
python .opencode/skills/rlm/scripts/rlm_repl.py exec <<'PY'
paths = write_chunks('.opencode/rlm_state/chunks', size=150000, overlap=2000)
print(f"Created {len(paths)} chunks")
for p in paths[:5]:
    print(p)
PY
```

Then in OpenCode, invoke `@rlm-subcall` for each chunk:
> "@rlm-subcall: Read .opencode/rlm_state/chunks/chunk_0000.txt and extract all definitions of sustainability with direct quotes and author attributions."

Collect results and synthesise:

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "
add_buffer('''
{
  \"chunk\": 0,
  \"findings\": [...]
}
''')
"
```

---

## Example 10: Export Synthesis Buffers

After collecting chunk analysis results:

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py export-buffers .opencode/rlm_state/synthesis.txt
```

Then ask the agent to synthesise from the file:
> "Read .opencode/rlm_state/synthesis.txt and write a 500-word literature review on definitions of sustainability in LaTeX with BibTeX."

---

## REPL Quick Reference

```bash
# Load corpus
python .opencode/skills/rlm/scripts/rlm_repl.py load-corpus ./context/
python .opencode/skills/rlm/scripts/rlm_repl.py pdf ./context/paper.pdf

# Status
python .opencode/skills/rlm/scripts/rlm_repl.py status

# Helpers
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(list_papers())"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(stats())"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(find_papers('keyword'))"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(cite('Author Year'))"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(search_claim('your claim here'))"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(get_paper(0))"
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(extract_references(0))"

# Chunking
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "paths = write_chunks('.opencode/rlm_state/chunks'); print(len(paths), 'chunks')"

# Cleanup
python .opencode/skills/rlm/scripts/rlm_repl.py reset
```

---

## Agent Quick Reference

| Agent | Use for |
|-------|---------|
| `research-write` | Writing, synthesis, LaTeX output, citation generation |
| `research-read` | Safe exploration, theme mapping, gap analysis (read-only) |
| `@rlm-subcall` | Chunk-level extraction — called by root LLM in RLM loops |
| `@paper-analyzer` | Deep single-paper analysis |

## Command Quick Reference

| Command | Purpose |
|---------|---------|
| `/rlm` | Load PDF corpus and run RLM workflow |
| `/research` | Answer a research question from corpus |
| `/write-section` | Draft a LaTeX section with BibTeX |
| `/cite-check` | Verify citations against loaded corpus |
| `/bibtex` | Generate BibTeX entries from corpus metadata |
| `/deep-dive` | Full analysis of a single paper |
