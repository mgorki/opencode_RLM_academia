## v0.2-beta — Scientific Literature Edition

> ⚠️ **Experimental release** — Everything is still in active development. Double-check all outputs and use at your own risk.

### What is this?

OpenCode RLM is an implementation of the [Recursive Language Model (RLM) pattern](https://arxiv.org/abs/2512.24601) for [OpenCode](https://opencode.ai), specialized for **scientific literature analysis and academic writing**. It enables OpenCode to analyze PDF paper corpora that exceed typical LLM context windows and produce LaTeX drafts with verified BibTeX citations.

---

### ✨ What's in this release

**Core functionality:**
- 📄 **PDF corpus loading** via `pdfplumber` — load individual PDFs or entire directories at once
- 🔁 **Resumable ingestion** — corpus loading saves state after each PDF; interrupted runs pick up where they left off
- 🧩 **Automatic chunking** — papers are split into overlapping 150k-character chunks for sub-LM delegation
- 🔑 **Automatic BibTeX key generation** — deterministic keys from Zotero-style filenames (`cook2013quantifying`)
- 📝 **LaTeX + BibTeX output** — produces `\textcite{}` / `\parencite{}` citations with matching `.bib` entries
- 🚫 **Anti-hallucination safeguards** — `rlm-subcall` is explicitly forbidden from fabricating authors, years, quotes, or titles

**Agents:**
- `research-write` — full access agent for drafting LaTeX sections, generating BibTeX, writing literature reviews
- `research-read` — read-only agent for safe corpus exploration and planning
- `rlm-subcall` — chunk-level subagent; returns structured JSON (claims, evidence, metadata)
- `paper-analyzer` — deep-dive subagent for full structured extraction from a single paper

**Custom commands:**
- `/rlm` — load corpus and run the RLM analysis workflow
- `/research` — answer a research question grounded in the loaded corpus
- `/write-section` — draft a LaTeX section with in-text citations
- `/cite-check` — verify a claim or citation against the corpus
- `/bibtex` — generate BibTeX entries from corpus metadata
- `/deep-dive` — full structured extraction of a single paper

**REPL helpers** (via `rlm_repl.py`):
`list_papers()`, `find_papers()`, `cite()`, `search_claim()`, `extract_references()`, `get_paper()`, `stats()`, `grep()`, `add_buffer()`, and more

---

### 📦 Installation

```bash
pip install -r requirements.txt   # pdfplumber>=0.11.0
```

Then place your PDFs in `context/` (Zotero-style naming recommended) and start OpenCode.

---

### ⚙️ Requirements

- [OpenCode](https://opencode.ai/docs/) with a configured API key
- Python 3
- `pdfplumber >= 0.11.0`

---

### 🙏 Credits

Based on the [Recursive Language Models paper](https://arxiv.org/abs/2512.24601) (Zhang, Kraska, Khattab — MIT CSAIL).
Adapted from [claude_code_RLM](https://github.com/Brainqub3/claude_code_RLM) for OpenCode by [Gavin Yap (maclarensg)](https://github.com/maclarensg/opencode_RLM).
Specialized for scientific PDF corpus analysis and academic writing by Michael Gorki.

---

*Model-agnostic in principle — should work (more or less well) with any model supported by OpenCode.*
