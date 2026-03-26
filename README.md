# Fathom: Pedagogical Skill Suite for AI Agents

**Fathom** is a specialized skill suite designed for AI coding assistants (like Claude Code, Antigravity, and others) to build genuine comprehension of codebases and complex topics—not just summaries, but deep, "grokked" knowledge.

## Core Capabilities

- **Codebase Deep Dives:** Automated GitHub reconnaissance and instructive Jupyter notebook generation.
- **Conceptual Grokking:** Multi-perspective exploration via "AI Councils" and structured vault notes.
- **Comprehension Quizzes:** Interactive, evaluation-driven loops to verify understanding.
- **Executive Synthesis:** Role-aware briefs with Mermaid.js architecture diagrams.

## Installation for AI Agents

To add Fathom to your agent's skill directory:

1. Clone this repository into your skills or commands folder.
2. Link the main `fathom.md` to your agent's execution entry point.
3. Ensure the `sub-skills/` and `templates/` directories are accessible relative to the main skill.

## Synchronization (Multi-Agent Setup)

If you use multiple agents, it is recommended to keep a master copy in your central configuration (e.g., `~/.claude/skills/fathom/`) and symlink it to other agents' specific skill paths.

---
*Created by [kelverssg](https://github.com/kelverssg) — March 2026*
