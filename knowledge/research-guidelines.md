# Research Guidelines

## Anti-Hallucination Protocol

This is the most critical rule in this system:

> **You MUST NOT cite, attribute, or quote any source that is not present in the loaded corpus.**

### What counts as hallucination (forbidden)
- Writing `(Smith & Jones, 2019)` when that paper is not in the corpus
- Attributing a specific statistic or finding to an author you are not certain made that claim
- Inventing a journal name, volume, or DOI
- Using a plausible-sounding citation as a placeholder
- Saying "as many scholars have noted" without being able to name them from the corpus

### What to do instead
- Use `list_papers()` to verify a paper is loaded before citing it
- Use `find_papers(author)` to check spelling and exact year
- Write "This claim is not supported by a source in the loaded corpus" when you cannot find evidence
- Mark knowledge gaps explicitly: [SOURCE NEEDED — not found in corpus]

---

## APA 7 Quick Reference

### In-Text Citations

| Situation | Format |
|-----------|--------|
| One author | (Smith, 2020) |
| Two authors | (Smith & Jones, 2020) |
| Three or more | (Smith et al., 2020) |
| Running text (one) | Smith (2020) argued that... |
| Running text (two) | Smith and Jones (2020) found... |
| Running text (three+) | Smith et al. (2020) showed... |
| Direct quote | (Smith, 2020, p. 45) |
| Secondary source | (Jones, 2015, as cited in Smith, 2020) — use sparingly |

### Reference List Format

**Journal article:**
Author, A. A., & Author, B. B. (Year). Title of article in sentence case. *Journal Title in Title Case*, *volume*(issue), start–end. https://doi.org/xxxxx

**Book:**
Author, A. A. (Year). *Title of book in sentence case* (Edition if not first). Publisher. https://doi.org/xxxxx

**Book chapter:**
Author, A. A. (Year). Title of chapter. In E. E. Editor (Ed.), *Title of book* (pp. xx–xx). Publisher. https://doi.org/xxxxx

**Report:**
Author, A. A. (Year). *Title of report*. Organisation. URL

---

## BibTeX Templates

### Journal Article
```bibtex
@article{keyYEARword,
  author    = {Last, First and Last2, First2},
  year      = {YYYY},
  title     = {Title in sentence case},
  journal   = {Journal Title},
  volume    = {X},
  number    = {Y},
  pages     = {start--end},
  doi       = {10.xxxx/xxxxx},
}
```

### Book
```bibtex
@book{keyYEARword,
  author    = {Last, First},
  year      = {YYYY},
  title     = {Title of Book},
  edition   = {2nd},
  publisher = {Publisher Name},
  address   = {City},
  doi       = {10.xxxx/xxxxx},
}
```

### Book Chapter
```bibtex
@incollection{keyYEARword,
  author    = {Last, First},
  year      = {YYYY},
  title     = {Chapter title},
  booktitle = {Book Title},
  editor    = {Last, First},
  pages     = {start--end},
  publisher = {Publisher},
  address   = {City},
}
```

### Report / Grey Literature
```bibtex
@techreport{keyYEARword,
  author      = {Last, First},
  year        = {YYYY},
  title       = {Title of Report},
  institution = {Organisation Name},
  url         = {https://...},
}
```

### BibTeX Key Convention
Format: `firstauthorlastname` + `year` + `firstmeaningfulword`
Example: `cook2013quantifying`, `rockstrom2009planetary`, `ipcc2021sixth`

---

## LaTeX / Overleaf Setup

### Recommended Preamble (APA 7 with biblatex)
```latex
\documentclass[12pt, a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[style=apa, backend=biber]{biblatex}
\addbibresource{refs.bib}
\usepackage{csquotes}
\usepackage{hyperref}

\begin{document}
\section{Introduction}
\parencite{cook2013quantifying} demonstrated...
\textcite{rockstrom2009planetary} argued...

\printbibliography
\end{document}
```

### Citation Commands (biblatex-apa)
| Command | Output |
|---------|--------|
| `\parencite{key}` | (Author, Year) |
| `\textcite{key}` | Author (Year) |
| `\parencite[p.~45]{key}` | (Author, Year, p. 45) |
| `\parencite{key1,key2}` | (Author1, Year1; Author2, Year2) |

---

## Scientific Writing Standards

### Hedged Language
Prefer cautious formulations that match the strength of evidence:
- Strong: "demonstrates", "establishes", "confirms"
- Moderate: "suggests", "indicates", "finds", "reports"
- Weak: "proposes", "argues", "contends", "speculates"

Match your hedge to the evidence. A single study "suggests"; a meta-analysis may "demonstrate".

### Claim–Evidence Structure
Every empirical claim should be followed (or preceded) by its citation:
> The global scientific consensus on anthropogenic climate change exceeds 97% \parencite{cook2013quantifying}.

### Distinguishing Claim Types
- Author's claim: "Smith (2020) argues that X"
- Established fact: "X has been demonstrated across multiple studies \parencite{...}"
- Your synthesis: "Taken together, these findings suggest X"
- Contested: "While Smith (2020) contends X, Jones (2019) disputes this on the grounds that Y"

### Avoiding Plagiarism in Paraphrase
When paraphrasing, change both the wording AND the sentence structure. When in doubt, use a direct quote with quotation marks and page number.
