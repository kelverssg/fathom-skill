---
name: fathom
description: Pedagogical skill suite to build genuine comprehension — not summaries, but grokked knowledge. Handles codebases, topics, and documents. Context-inferring and role-aware. Orchestrates Jupyter notebooks, flashcards (via notebooklm), multi-perspective exploration (via council), and comprehension quizzes. Use when asked to "fathom X", "learn X", "grok X", "build a notebook for X", "make flashcards for X", "quiz me on X", or "explain X deeply".
---

# Fathom Skill

Rapidly build genuine understanding — not summaries, but real comprehension. Codebase tours, topic deep-dives, Jupyter notebooks, flashcards, quizzes, and executive briefs. The skill infers what you need based on context.

---

## Entry Points

| Command | What it does |
|---------|-------------|
| `/fathom` | Auto-mode — scans context, infers role, picks outputs |
| `/fathom codebase [path]` | Codebase understanding: GitHub recon → Jupyter notebook + component map |
| `/fathom topic [topic]` | Knowledge domain: council perspectives + vault note |
| `/fathom jupyter [topic/path]` | Build an instructive Jupyter notebook only |
| `/fathom flashcard [topic]` | Generate flashcards via notebooklm |
| `/fathom quiz [topic]` | Comprehension quiz — 10 questions, scored |
| `/fathom brief [topic]` | Executive synthesis — under 400 words, role-aware |

---

## Step 1 — Parse Arguments

From the user's message, extract:

- **mode**: `codebase` | `topic` | `jupyter` | `flashcard` | `quiz` | `brief` | (blank = `auto`)
- **subject**: everything after the mode keyword — file path, topic name, or concept

If mode is blank (`/fathom` alone with no argument), proceed to Step 2.
If mode is specified, skip Step 2 and go directly to Step 3.

---

## Step 2 — Context Scan (auto-mode only)

Run in parallel:

```bash
# Scan the current directory
ls -la .

# Git history if present
git log --oneline -10 2>/dev/null || echo "not a git repo"

# Project instructions if present
cat CLAUDE.md 2>/dev/null | head -30 || echo "no CLAUDE.md"
```

From the output, infer:
- **Language and framework** (Python, TypeScript, Go, shell scripts, etc.)
- **Domain** (infra, product, finance, philosophy, science, etc.)
- **Scale** (single file / focused module / large codebase / abstract concept)

If the context strongly points to code → set mode to `codebase`.
If the context points to a knowledge domain → set mode to `topic`.
If unclear, ask one question: *"Are you looking to understand the code structure, the underlying concepts, or both?"*

---

## Step 3 — Role Inference and Asset Selection

Infer the user's current mode from context clues in the subject or session:

| If context suggests... | Role | Outputs |
|------------------------|------|---------|
| Source code, file paths, repo | Technical / Tech Lead | Jupyter notebook + component map |
| Strategy, decisions, trade-offs | Executive / PM / Business | Brief + mermaid diagram + flashcards |
| Theory, philosophy, or learning domain | Learner | Flashcards + quiz + council perspectives |
| Mixed or no clear signal | Ask once, then proceed | |

**Asset selection matrix:**

```
Role = technical   → Jupyter notebook (fathom-jupyter)
Role = executive   → Brief (inline) + mermaid diagram + flashcards (notebooklm)
Role = learner     → Council perspectives + flashcards (notebooklm) + quiz (fathom-quiz)
Explicit mode set  → Run only the specified output
```

Announce the plan before proceeding:
`Fathom: [subject] | Mode: [mode] | Outputs: [list]`

---

## Step 4 — GitHub Reconnaissance (codebase mode only)

Before generating codebase assets, search for existing tools to stand on:

Search GitHub using WebSearch with queries such as:
- `"[language] codebase explainer jupyter notebook site:github.com"`
- `"code-to-flow" OR "codebase-tour" OR "explain-codebase" site:github.com stars:>100`
- `"[framework] architecture diagram generator open source"`

Surface the top 2–3 candidates:
- Name, GitHub URL
- What it does (one line)
- Whether it can be adapted or referenced

Note findings in the vault output note. Proceed with asset generation regardless — recon is informational, not blocking.

---

## Step 5 — Generate Assets

Route to the appropriate sub-skill or inline action:

### A — Jupyter Notebook

Load and follow sub-skill instructions from:
`~/.claude/skills/fathom/sub-skills/fathom-jupyter.md`

Subject: the topic or file path.
Output: `Learning/Fathom/Notebooks/[Topic].ipynb`

### B — Flashcards via notebooklm

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
DATE=$(date +%Y-%m-%d)

# Create a NotebookLM notebook for the subject
notebooklm create "Fathom: [subject]"

# Add vault file as source if a relevant file exists; otherwise research
# For a vault file:
# notebooklm source add "$VAULT/[path]"
# For a concept:
notebooklm source add-research "[subject]"

notebooklm generate flashcards
notebooklm download --output "$VAULT/Technology/NotebookLM/$DATE - [Subject] Flashcards.pdf"
```

### C — Council Perspectives (topic mode)

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
python3 "$VAULT/System/tools/council.py" \
  "Explain [subject]: its essence, history, surprising implications, and what a practitioner should know that a textbook won't say" \
  --advisors 6
```

Council output is saved to `Personal/Councils/` automatically. Link to it from the vault note.

### D — Quiz

Load and follow sub-skill instructions from:
`~/.claude/skills/fathom/sub-skills/fathom-quiz.md`

Subject: the topic.

### E — Brief (executive mode, inline)

Generate directly — no sub-skill. Under 400 words, structured as:

```markdown
## [Topic] — Executive Brief

**What it is:** [one sentence]

**Why it matters:**
- [bullet 1]
- [bullet 2]
- [bullet 3]

**Key trade-offs or decision it implies:**
[1–2 sentences]

**Mermaid diagram (if applicable):**
[flow or architecture diagram]
```

---

## Step 6 — Write Vault Note

Always write a learning note at:
`Learning/Fathom/[YYYY-MM-DD] - [Topic].md`

Use the template at `~/.claude/skills/fathom/templates/vault-note-template.md`.

**Critical:** Include frontmatter `type: learning` and `tags: [fathom]` — this makes the note auto-surface in `System/Bases/Learning.base`.

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
DATE=$(date +%Y-%m-%d)
# Write the note to Learning/Fathom/
```

Populate the note with:
- **What I Grokked** — 3–5 bullet insights (not summaries, genuine understanding)
- **Key Concepts** — condensed conceptual map (can be a table or nested bullets)
- **Assets Generated** — wikilinks to Jupyter, flashcards, council output
- **Open Questions** — what still needs understanding (makes this useful for next session)

---

## Step 7 — Report

| Item | Value |
|------|-------|
| Subject | [subject] |
| Mode | [mode] |
| Role inferred | [role] |
| Assets generated | [list with paths] |
| Vault note | `Learning/Fathom/[filename]` |
| Notebook | `Learning/Fathom/Notebooks/[name].ipynb` (if generated) |
| Open questions | N (see vault note) |

---

*Fathom Skill v1.0 — 2026-03-26 — Claude Code (claude-sonnet-4-6)*
