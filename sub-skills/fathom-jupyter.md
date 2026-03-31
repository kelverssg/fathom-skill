# Fathom — Jupyter Sub-Skill

Generate an instructive, fully runnable Jupyter notebook (``.ipynb``) that builds genuine comprehension of a codebase, library, or concept.

**Output:** `Learning/Fathom/Notebooks/[Topic].ipynb`
**Runtime:** Python 3 (available at `/Library/Frameworks/Python.framework/Versions/3.10/bin/python3`)
**Validator:** `python3 -c "import nbformat; nbformat.validate(nbformat.read(open('[file]'), as_version=4))"`

---

## Step 1 — Understand the Subject

**For a file path or codebase:**
```bash
# Read the target — understand what it does before writing a cell
cat [path]
# or for a directory
find [path] -name "*.py" | head -20
# Read the key files
```

Identify:
- What the code does at a high level
- The top 3–5 distinct concepts or components to teach
- External dependencies (what to `pip install`)
- One or two working examples that demonstrate the whole thing

**For a concept or topic:**
- Identify 3–5 sub-concepts that build on each other (scaffold the learning)
- Find or construct simple runnable examples for each
- Note any standard library or pip package that lets you demonstrate it in code

---

## Step 2 — Plan the Notebook Structure

Before writing any cells, plan the sequence:

```
1. Overview cell (markdown)       — What we're learning and why it matters
2. Dependency cell (code)         — pip install block, imports
3. For each concept (2–5 total):
   a. Concept cell (markdown)     — Explain the concept, one idea per cell
   b. Demo cell (code)            — Show it running, output visible
   [For 🔑 threshold concepts: add a second markdown cell after the demo
    — "Why this is a portal" — explaining what changes once you grasp it]
4. Worked example (code)          — End-to-end realistic scenario
5. Challenge cell (markdown)      — "Your turn: try X" prompt
6. Quiz cells (markdown)          — 3 multiple-choice or fill-in questions
7. Summary cell (markdown)        — Key takeaways + wikilinks to vault
```

**Alignment with Field Map:** Sequence concepts in the same order they appear in the Field Map's critical path. If the Field Map hasn't been built yet (running fathom-jupyter standalone), derive the critical path from the source material.

**Threshold concepts get extra depth:** Any concept flagged 🔑 in the Field Map gets a third cell — a short "portal" markdown cell that explicitly states: what the common misconception is before grasping it, what shifts once you cross it, and what becomes accessible downstream.

Limit to 3–5 concepts total. A tight notebook that runs clean beats a comprehensive one that confuses.

---

## Step 3 — Build the Notebook via Python

Write a Python script to `/tmp/build_notebook.py` that uses `nbformat` to assemble the notebook:

```python
import nbformat
import json
import os

VAULT = "/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"

# --- Cell content (fill in based on subject) ---

overview_md = """# [Topic]

*Fathom notebook — [date]*

## What we're learning

[1–2 sentence explanation of the topic and why it matters]

## What you'll be able to do after this notebook

- [Outcome 1]
- [Outcome 2]
- [Outcome 3]
"""

dependency_code = """# Install dependencies (run this cell first)
# !pip install [package1] [package2]

# Imports
import [module]
"""

# Concept cells — repeat pattern for each concept
concept_1_md = """## Concept 1: [Name]

[Explanation — plain English, one idea, under 100 words]

**Key point:** [the thing to remember]
"""

demo_1_code = """# Demonstrate Concept 1
[runnable code that outputs something visible]
"""

# ... add more concept pairs ...

worked_example_md = """## Worked Example: [Realistic Scenario]

[Setup — what problem we're solving]
"""

worked_example_code = """# End-to-end example
[self-contained, realistic code]
"""

challenge_md = """## Challenge

Try the following:

1. [Specific task that requires applying Concept 1]
2. [Specific task that requires applying Concept 2]
3. Stretch: [Harder variation]

<details>
<summary>Hints</summary>

- [Hint for task 1]
- [Hint for task 2]

</details>
"""

quiz_md = """## Quick Quiz

**Q1:** [Question about Concept 1]

a) [Option]  b) [Option]  c) [Correct option]  d) [Option]

---

**Q2:** What does `[function]` return when called with `[arg]`?

*(Fill in the blank)*

---

**Q3:** True or False: [Statement about Concept 2]

<details>
<summary>Answers</summary>

1. c  2. [answer]  3. [True/False — reason]

</details>
"""

summary_md = """## Summary

### What you learned

| Concept | Key Point |
|---------|-----------|
| [Concept 1] | [One sentence] |
| [Concept 2] | [One sentence] |

### Further reading (vault)

- [[Learning/Fathom/[date] - [Topic]]] — session note
- [[Philosophy/[related]]] or [[Technology/[related]]] — related vault content

### What to explore next

- [Logical next topic]
- [Practical application]
"""

# --- Assemble the notebook ---
nb = nbformat.v4.new_notebook()
nb.metadata = {
    "kernelspec": {
        "display_name": "Python 3",
        "language": "python",
        "name": "python3"
    },
    "language_info": {
        "name": "python",
        "version": "3.10.0"
    }
}

nb.cells = [
    nbformat.v4.new_markdown_cell(overview_md),
    nbformat.v4.new_code_cell(dependency_code),
    nbformat.v4.new_markdown_cell(concept_1_md),
    nbformat.v4.new_code_cell(demo_1_code),
    # ... more concept pairs ...
    nbformat.v4.new_markdown_cell(worked_example_md),
    nbformat.v4.new_code_cell(worked_example_code),
    nbformat.v4.new_markdown_cell(challenge_md),
    nbformat.v4.new_markdown_cell(quiz_md),
    nbformat.v4.new_markdown_cell(summary_md),
]

# --- Write and validate ---
output_dir = os.path.join(VAULT, "Learning/Fathom/Notebooks")
os.makedirs(output_dir, exist_ok=True)
output_path = os.path.join(output_dir, "[Topic].ipynb")

with open(output_path, "w") as f:
    nbformat.write(nb, f)

# Validate
with open(output_path) as f:
    nb_check = nbformat.read(f, as_version=4)
nbformat.validate(nb_check)

print(f"Notebook written and validated: {output_path}")
print(f"Cells: {len(nb.cells)}")
```

Fill in all `[placeholder]` values based on the actual subject matter before running.

Then execute:
```bash
python3 /tmp/build_notebook.py
```

---

## Step 4 — Verify the Output

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
python3 -c "
import nbformat, json
with open('$VAULT/Learning/Fathom/Notebooks/[Topic].ipynb') as f:
    nb = nbformat.read(f, as_version=4)
nbformat.validate(nb)
print(f'Valid — {len(nb.cells)} cells')
for i, c in enumerate(nb.cells):
    print(f'  Cell {i+1}: [{c.cell_type}] {c.source[:60].strip()!r}')
"
```

If validation fails, inspect the error and fix the cell that caused it (usually a malformed metadata dict or missing `source` field).

---

## Step 5 — Report

Return to the main fathom skill:

| Item | Value |
|------|-------|
| Notebook | `Learning/Fathom/Notebooks/[Topic].ipynb` |
| Cell count | N |
| Concepts covered | [list] |
| Validation | Passed / Failed (with error) |
| Open in Jupyter | `jupyter notebook "[path]"` |

The notebook path becomes an asset link in the vault note (Step 6 of main skill).

---

## Quality Rules

- **Every code cell must run standalone** — no hidden state from previous cells except imports
- **Dependency cell first** — always `pip install` before first import
- **One concept per markdown cell** — split stacked ideas
- **Output must be visible** — end each demo cell with a `print()` or expression that shows output
- **Challenge before quiz** — doing precedes testing
- **No wall-of-code cells** — if a demo needs >20 lines, split it

---

*fathom-jupyter sub-skill v1.0 — 2026-03-26*
