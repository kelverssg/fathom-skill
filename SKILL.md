---
name: fathom
description: Integrated pedagogical skill suite for building genuine comprehension. Orchestrates Jupyter notebooks, flashcards, quizzes, and executive briefs. Bridged from the Claude Code master version to ensure synchronization.
---

# Fathom (Antigravity Bridge)

Use this skill to help the user "grok", "learn", or "deeply understand" a topic or codebase. This skill is a bridge to the master Fathom suite located in the `Claude/` subdirectory.

## Capabilities

- **Codebase Deep Dives:** GitHub recon + instructive Jupyter notebooks (Python 3.14 / Node v24).
- **Conceptual Grokking:** Council perspectives + vault notes with frontmatter surfacing.
- **Comprehension Quizzes:** 10-question evaluation loops.
- **Executive Synthesis:** 400-word briefs with Mermaid diagrams.

## Usage Guide

1. **Invoke the Bridge:** When the user says "fathom X", read the master instruction file at:
   `./Claude/fathom.md`
2. **Execute Sub-Skills:** Delegate to the specific sub-skills as needed:
   - `jupyter`: `./Claude/sub-skills/fathom-jupyter.md`
   - `quiz`: `./Claude/sub-skills/fathom-quiz.md`
   - `templates`: `./Claude/templates/`
3. **Paths:** Always use the vault-standard output paths:
   - Notes: `Learning/Fathom/[YYYY-MM-DD] - [Topic].md`
   - Notebooks: `Learning/Fathom/Notebooks/[Topic].ipynb`

## Synchronization

This skill is symlinked to `~/.claude/skills/fathom/`. Any updates made by the Claude agent to the master version will be automatically available to Antigravity.

---
*Fathom Bridge v1.0 — 2026-03-26 — Antigravity (Mac ARM64)*
