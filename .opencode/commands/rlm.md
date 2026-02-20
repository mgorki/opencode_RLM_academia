---
description: Run RLM workflow for scientific PDF corpus analysis
agent: research-write
---

Load the `rlm` skill to process a scientific PDF corpus using the Recursive Language Model workflow.

$ARGUMENTS

If no arguments were provided, ask for:
1. The path to the corpus directory (e.g. `./context/`) or a single PDF path
2. The research question or analysis goal

Then follow the RLM skill procedure to:
1. Load the PDF corpus into the REPL (using `load-corpus` for a directory or `pdf` for a single file)
2. Orient yourself with `list_papers()` and `stats()`
3. Find relevant papers using `find_papers()` and `cite()`
4. For large corpora: chunk the content and delegate analysis to `@rlm-subcall`
5. Synthesise findings with explicit source attribution
6. Produce LaTeX + BibTeX output if writing is requested
