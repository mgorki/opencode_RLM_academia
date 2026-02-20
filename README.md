# OpenCode RLM - Scientific Literature Edition *(Version: 0.2)*

An OpenCode implementation of the Recursive Language Model (RLM) pattern, specialized for analyzing scientific paper corpora and producing LaTeX drafts with verified BibTeX citations for academic publications. ***Everything still experimental! Double check everything and use at your own risk!***

## About

This repository enables OpenCode to analyze large collections of PDF research papers that exceed typical LLM context windows. A root language model orchestrates sub-LLM calls over chunked paper text, synthesizes findings, and produces grounded academic writing — with strong safeguards against citation hallucination.

**Use cases:**
- Analyzing and comparing large PDF paper corpora
- Drafting literature review sections with verified citations
- Extracting claims, evidence, and metadata from academic papers
- Generating BibTeX entries from corpus data
- Mapping research consensus, debates, and open questions

## Architecture

| RLM Concept | Implementation | Component |
|-------------|----------------|-----------|
| Root LLM | Main OpenCode conversation | `research-write` / `research-read` agent |
| Sub-LLM (`llm_query`) | `rlm-subcall` subagent | OpenCode subagent |
| External Environment | Persistent Python REPL (`rlm_repl.py`) | Python 3 + pdfplumber |

```
┌─────────────────────────────────────────────────────────┐
│                      Root LLM                            │
│             research-write / research-read               │
│          (Orchestration, Synthesis, Writing)             │
└──────────────────────┬──────────────────────────────────┘
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
┌─────────────────┐         ┌─────────────────┐
│  Python REPL    │         │  rlm-subcall    │
│  (rlm_repl.py)  │         │  (Subagent)     │
│                 │         │                 │
│ - Load PDFs     │         │ - Analyze chunk │
│ - Extract text  │         │ - Extract claims│
│ - Chunk & store │         │ - Return JSON   │
│ - REPL helpers  │         │ - No fabrication│
└─────────────────┘         └─────────────────┘
```

**Model-agnostic in principle** — should work (more or less well) with any model supported by OpenCode.

## Prerequisites

- **OpenCode** — [Install OpenCode](https://opencode.ai/docs/)
- **Python 3** — For the persistent REPL environment
- **pdfplumber** — PDF text extraction (`pip install pdfplumber`)
- **API keys** — Configured for your model in OpenCode

Install the Python dependency:
```bash
pip install -r requirements.txt
```

## Quick Start

1. **Place your PDF papers in the `context/` directory**

   Papers should follow Zotero-style naming for best metadata extraction:
   ```
   context/Cook et al. - 2013 - Quantifying the consensus on anthropogenic....pdf
   context/Rockstrom et al. - 2009 - A safe operating space for humanity.pdf
   ```

2. **Start OpenCode**
   ```bash
   opencode
   ```

3. **Load your corpus and run the RLM workflow**
   ```
   /rlm corpus=./context/ query="What definitions of sustainability appear across these papers?"
   ```

4. **Write a section or check citations**
   ```
   /write-section planetary boundaries and Earth-system tipping points
   /cite-check "97% of climate scientists agree on anthropogenic warming"
   ```

## Available Agents

### Primary Agents (Tab to switch)

| Agent | Description | Use Case |
|-------|-------------|----------|
| `research-write` | Full access — edit, bash, web | Drafting LaTeX, generating BibTeX, writing literature reviews |
| `research-read` | Read-only analysis mode | Safe corpus exploration, mapping, planning |

### Subagents (@mention to invoke)

| Agent | Description | Use Case |
|-------|-------------|----------|
| `rlm-subcall` | Chunk-level paper analyzer | RLM workflow chunk processing; returns structured JSON |
| `paper-analyzer` | Deep-dive subagent | Full structured extraction from a single paper |

## Custom Commands

| Command | Description |
|---------|-------------|
| `/rlm` | Load corpus and run the RLM analysis workflow |
| `/research` | Answer a research question grounded in loaded corpus |
| `/write-section` | Draft a LaTeX section with in-text citations |
| `/cite-check` | Verify a claim or citation against the corpus |
| `/bibtex` | Generate BibTeX entries from corpus metadata |
| `/deep-dive` | Run full structured extraction on a single paper |

## RLM REPL Commands

The persistent Python REPL (`rlm_repl.py`) provides these helpers after loading a corpus:

```python
# Corpus inspection
list_papers()                   # List all loaded papers with keys
find_papers('keyword')          # Search by author, title, or keyword
stats()                         # Corpus size and composition

# Citation and retrieval
cite('Author Year')             # Look up a paper and its BibTeX key
search_claim('claim text')      # Find papers supporting a claim
peek(start, end)                # View a slice of a chunk
grep(pattern, max_matches)      # Regex search across corpus text

# Chunking
chunk_indices(size, overlap)    # Get chunk boundaries
write_chunks(out_dir, size)     # Write chunks to files

# State management
add_buffer(text)                # Store intermediate results across sub-calls
```

REPL can also be invoked directly from the command line:

```bash
python .opencode/skills/rlm/scripts/rlm_repl.py load-corpus ./context/
python .opencode/skills/rlm/scripts/rlm_repl.py status
python .opencode/skills/rlm/scripts/rlm_repl.py exec -c "print(list_papers())"
python .opencode/skills/rlm/scripts/rlm_repl.py reset
```

## Output Formats

### LaTeX with BibTeX

```latex
\documentclass{article}
\usepackage[style=apa,backend=biber]{biblatex}
\addbibresource{refs.bib}

\begin{document}
\textcite{cook2013quantifying} found that among abstracts expressing a position
on anthropogenic global warming, 97\% endorsed the consensus.
Planetary boundaries were proposed by \textcite{rockstrom2009safe} as a
framework for defining a safe operating space for humanity.
\printbibliography
\end{document}
```

### BibTeX entries

```bibtex
@article{cook2013quantifying,
  author  = {Cook, John and Nuccitelli, Dana and others},
  year    = {2013},
  title   = {Quantifying the consensus on anthropogenic global warming},
  journal = {Environmental Research Letters},
  volume  = {8},
  pages   = {024024},
  doi     = {10.1088/1748-9326/8/2/024024},
}
```

### Structured JSON (from subagents)

Each chunk analyzed by `rlm-subcall` returns:

```json
{
  "chunk_id": "cook2013_chunk_0001.txt",
  "chunk_summary": "Methods and consensus calculation section",
  "paper_metadata": {
    "authors_visible": ["Cook, John", "Nuccitelli, Dana"],
    "year_visible": "2013",
    "title_visible": "Quantifying the consensus on anthropogenic global warming"
  },
  "relevant": [
    {
      "claim": "97% of climate scientists endorse the consensus on AGW",
      "quote": "Among abstracts expressing a position on AGW...",
      "authors": "Cook et al.",
      "year": "2013",
      "confidence": "high",
      "category": "finding"
    }
  ],
  "references_mentioned": ["IPCC (2007)", "Doran & Zimmerman (2009)"],
  "missing": ["Sample size details"],
  "suggested_next_queries": ["What methodological criticisms exist?"]
}
```

## Repository Structure

```
opencode_RLM/
├── AGENTS.md                          # Main agent instructions (research focus)
├── opencode.json                      # OpenCode configuration
├── README.md                          # This file
├── requirements.txt                   # Python dependencies (pdfplumber)
├── .opencode/
│   ├── agents/
│   │   └── rlm-subcall.md            # Sub-LM agent definition
│   ├── skills/
│   │   └── rlm/
│   │       ├── SKILL.md              # RLM skill definition
│   │       └── scripts/
│   │           └── rlm_repl.py       # Persistent Python REPL (PDF extraction, chunking)
│   └── commands/                      # Custom slash command definitions
├── prompts/
│   ├── research-write.md             # Write agent prompt (LaTeX, BibTeX, citations)
│   ├── research-read.md              # Read-only agent prompt
│   └── paper-analyzer.md             # Deep-dive subagent prompt
├── knowledge/
│   └── research-guidelines.md        # Anti-hallucination rules, BibTeX templates, LaTeX standards
├── examples/
│   └── RLM_USAGE_GUIDE.md            # Practical usage examples
└── context/                           # Place your PDF papers here
```

## Example Workflows

### Analyzing a Paper Corpus

```
# Start OpenCode, then load corpus and query
/rlm corpus=./context/ query="What are the main definitions of sustainability across these papers?"
```

The root LM loads all PDFs, extracts and chunks text, then delegates chunk analysis to `@rlm-subcall` for any papers too large to fit in context — synthesizing a grounded answer at the end.

### Writing a Literature Review Section

```
/write-section planetary boundaries and Earth-system tipping points
```

Produces a LaTeX subsection with `\textcite{}` and `\parencite{}` commands, drawing only on papers present in the loaded corpus. Unsupported claims are flagged.

### Verifying a Citation

```
/cite-check "97% of climate scientists agree on anthropogenic warming"
```

Searches the corpus for supporting evidence and reports which paper(s) back the claim, with direct quotes and BibTeX keys.

### Generating BibTeX

```
/bibtex Cook 2013
```

Returns a formatted BibTeX entry derived from the PDF metadata and Zotero-style filename.

### Deep-Diving a Single Paper

```
/deep-dive context/Rockstrom et al. - 2009 - A safe operating space for humanity.pdf
```

Uses `@paper-analyzer` to extract a fully structured breakdown: research questions, methodology, findings, limitations, and all references mentioned.

## Anti-Hallucination Safeguards

- All claims in LaTeX output must be grounded in a paper present in the loaded corpus
- `rlm-subcall` is explicitly forbidden from fabricating authors, years, quotes, or titles
- `cite-check` verifies citations against actual corpus text before confirming them
- BibTeX keys are generated deterministically from metadata (`<firstauthor><year><firsttitleword>`)
- The `research-read` agent is strictly read-only — no file writes during exploration

## State Persistence

Corpus state is saved automatically to `.opencode/rlm_state/state.pkl` after each PDF is loaded. If loading a large corpus is interrupted, re-running the load command resumes from the last saved state. Chunks are written to `.opencode/rlm_state/chunks/`.

## Based On

> **Recursive Language Models**
> Alex L. Zhang, Tim Kraska, Omar Khattab
> MIT CSAIL
> [arXiv:2512.24601](https://arxiv.org/abs/2512.24601)

Adapted from [claude_code_RLM](https://github.com/Brainqub3/claude_code_RLM) for OpenCode by Gavin Yap (maclarensg; https://github.com/maclarensg/opencode_RLM).
Specialized for scientific PDF corpus analysis and academic writing by Michael Gorki.

## License

MIT License — See LICENSE for details.
