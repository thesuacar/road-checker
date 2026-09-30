# 5ARE0 Assignment 1 — LaTeX report starter

This is an **editable scaffold**, not the official TU/e LaTeX style and not a completed report. It follows the **CRISP-DM headings and up-to-8-page report requirement** specified in the assignment PDF. Replace every `[Insert: ...]` placeholder with your own group's findings and verified figures. The PDF's generic health/clinical wording appears inconsistent with its bicycle-lane problem: make the substance relevant to the bicycle-lane assignment.

## Contents

```
main.tex
references.bib
sections/
  01_objective.tex
  02_data_understanding.tex
  03_data_preparation.tex
  04_modeling.tex
  05_evaluation.tex
  06_deployment.tex
figures/
  README.md
```

## Zotero workflow

1. Create a Zotero collection, e.g. `TUe / 5ARE0 / Assignment 1 — Bicycle Lane Quality`.
2. Add the sources actually consulted (course material as appropriate, methods and acquisition/physics references); verify metadata and author/year fields.
3. Export the collection as **BibLaTeX** to replace `references.bib` (BibTeX is also generally supported by `biblatex`). You can repeat the export after adding literature.
4. Cite within your sections with `\cite{your-zotero-key}`. Do not use uncited fake entries. Stable citation keys make re-export safer; Better BibTeX is optional, not required.

## Compile

From this folder:

```
latexmk -pdf -interaction=nonstopmode main.tex
```

The build uses `biblatex` + `biber` when citations exist. Or upload the entire folder to Overleaf and set the main document to `main.tex`.

To start fresh after changing the bibliography: `latexmk -C` then rebuild.

## Required handoff checks

- Keep the report **at or below 8 pages** and use your own completed factual prose.
- Fill in all participant contributions, sampling exclusions, features, model hyperparameters, results, error analysis and independent external deployment.
- The report PDF submitted in the assignment ZIP should have the required group-specific name, e.g. `group5_Report.pdf` if the instructor confirms that convention.
- Assignment PDF gives conflicting whole-ZIP filenames (`groupX_submission.zip` and `groupX_HAR_submission.zip`). Verify Canvas's live submission instructions.
- The *executed notebook* and bundled raw data, not this LaTeX source, must satisfy the notebook's no-extra-`.py`-imports requirement.
- Do not portray the pilot model run as a final result; freeze model choice before opening the test set.
