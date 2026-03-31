---
name: fathom
description: Pedagogical skill for deep comprehension — not summaries, but grokked knowledge. Runs a 5-phase framework: Field Map (architecture + threshold concepts + analogy bridges) → Primers (plain language) → Deep Dives (Jupyter per component) → Quiz (threshold-weighted) → Horizon (Did You Know + Knowledge Frontier). Handles codebases, topics, documents. Identifies what you don't know you don't know. Use when asked to "fathom X", "learn X", "grok X", "explain X deeply", "map X", "quiz me on X", "what should I learn next about X".
---

# Fathom Skill v2.0

Five-phase learning framework for genuine comprehension. Especially useful for self-taught learners — surfaces the architecture, the threshold concepts (the portals that change everything downstream), prerequisite gaps, and what comes next.

---

## Entry Points

| Command | Phases run |
|---------|-----------|
| `/fathom [topic]` | All 5 phases with interactive pauses |
| `/fathom map [topic]` | Phases 1+2 only — landscape + primers |
| `/fathom dive [topic] [N or name]` | Phase 3 for one component only |
| `/fathom horizon [topic]` | Phase 5 only — Did You Know + frontier |
| `/fathom quiz [topic]` | Phase 4 only — markdown quiz in chat |
| `/fathom quizmaker [topic]` | Phase 4 as HTML file (browser, AI grading) |
| `/fathom flashcards [topic]` | Flashcard deck — Form B (self-contained HTML) + Form A (JSON for ea-flashcards viewer) |
| `/fathom flashcard-deck [topic]` | Form A only — JSON deck file (drag into `~/Desktop/ea-flashcards.html`) |
| `/fathom jupyter [topic]` | Phase 3 full notebook |
| `/fathom brief [topic]` | Executive brief inline (<400 words) |

---

## Step 1 — Parse & Scan

Extract from the user's message:
- **mode**: map | dive | horizon | quiz | jupyter | brief | (blank = full)
- **subject**: the topic, file path, or concept
- **component**: for `dive` mode — number or name from Field Map

If full mode and no subject given, scan context:
```bash
ls -la . && git log --oneline -5 2>/dev/null || true && cat CLAUDE.md 2>/dev/null | head -20 || true
```
Infer subject. If still unclear, ask once: *"What would you like to fathom?"*

Announce before proceeding:
`Fathom v2.0 — [subject] | Mode: [mode]`

---

## Step 2 — Field Map (Phases 1 + 2)

*Skip entirely if mode is: dive / quiz / jupyter / brief / horizon.*

### 2a — Background research (run in parallel)

**If subject is an open-source project, library, or named codebase:**
Use GitHub MCP to fetch the repo README and key source files:
```
mcp__github__get_file_contents: {owner: "[org]", repo: "[repo]", path: "README.md"}
```
Also search: `mcp__github__search_repositories query:"[subject]"` if repo path is unknown.

**If subject is an active research or technical field:**
Fetch recent arXiv papers for grounding:
```
WebFetch: https://export.arxiv.org/api/query?search_query=all:[subject]&max_results=3&sortBy=relevance
```
Prompt: "List the title, authors, year, and one-sentence summary of each paper found."

**If subject has recent developments worth knowing:**
Run the last30days skill: `/last30days [subject]`

Use findings to sharpen the Field Map — especially for threshold concept identification and prerequisite gaps.

### 2b — Generate the Field Map

Output as structured Markdown:

```markdown
## Field Map: [Subject]

### Architecture

| # | Component | What it does | |
|---|-----------|-------------|--|
| 1 | [Name]    | [One line]  | 🔑 |
| 2 | [Name]    | [One line]  |    |
| 3 | [Name]    | [One line]  | ⚠️ |
| 4 | [Name]    | [One line]  | 🔑 |
| 5 | [Name]    | [One line]  |    |

🔑 = Threshold concept — understanding this transforms everything downstream
⚠️ = Prerequisite gap — this topic assumes you know this; flag if there may be a gap

### Dependency Order
[A → B means: understand A before B makes sense]
e.g. 1 → 2 → 4, 3 → 4, 4 → 5

### Critical Path
Minimum to understand first (in order): [list — usually 3–4 components]
```

Rules for the Field Map:
- 5–7 components maximum — resist listing everything, find the load-bearing ones
- Flag 1–3 threshold concepts (the portals — missing them causes snowball confusion)
- Flag prerequisite gaps honestly — what a self-taught learner in CS/philosophy/physics might not have
- Dependency order must reflect actual learning sequence, not alphabetical or arbitrary

### 2c — Primers

For each component, one block:

```
**[N]. [Component Name]** [🔑 if threshold] [⚠️ if prerequisite gap]

[2–3 sentences: what it does, how it fits in the architecture, plain language — no jargon without definition]

*Analogy:* [Bridge to one of Kelvin's domains: risk/insurance, Stoicism/Daoism/philosophy, 
martial arts, Japanese/Chinese language learning, physics — whichever fits naturally. 
If none fit naturally, skip rather than force one.]

[Threshold concepts only] *Why this is a portal:* [1 sentence — what becomes legible 
once you understand this that was opaque before]

[Prerequisite gap only] *If this feels unfamiliar:* [1 sentence on where to start — 
a /fathom call, a concept to look up, or a simpler entry point]
```

### 2d — Interactive Pause

After presenting the Field Map and all primers, pause and present:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Field Map complete
[N] components · [M] threshold concepts 🔑 · [P] prerequisite gaps ⚠️

What next?
A  Full run — notebook + horizon + quiz offer
B  Double-click a component — type its number
C  Horizon only — Did You Know + what to learn next
D  Quiz on threshold concepts now
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Wait for the user's response. Do not proceed until they choose.

**If user types a number** → run `dive` mode for that component (Step 3, single component).
**If user types A** → continue to Steps 3 → 5 → offer quiz.
**If user types B + number** → single component dive.
**If user types C** → skip to Step 5.
**If user types D** → skip to Step 4.

---

## Step 3 — Deep Dives (Jupyter Notebooks)

*Runs on: full mode A, `/fathom jupyter`, `/fathom dive [N]`.*

Load and follow: `~/.claude/skills/fathom/sub-skills/fathom-jupyter.md`

**v2.0 constraint:** Structure the notebook around Field Map component order.
Each major notebook section = one Field Map component, in dependency order (not arbitrary).
Threshold concepts (🔑) get more depth — an extra worked example or a "why this matters" cell.

For `dive` mode (single component): generate a focused notebook for that component only.
The notebook filename should reflect the component: `[Topic] — [Component Name].ipynb`

---

## Step 4 — Quiz

*Runs on: full mode (if user opts in at end), `/fathom quiz`, `/fathom quizmaker`.*

**`/fathom quiz`** — markdown quiz presented in chat. Load and follow: `~/.claude/skills/fathom/sub-skills/fathom-quiz.md`

**`/fathom quizmaker`** — generates a self-contained HTML file (browser UI, AI grading for short answers). Load and follow: `~/.claude/skills/fathom/sub-skills/fathom-quizmaker.md`

**`/fathom flashcards`** — generates flashcard deck. Load and follow: `~/.claude/skills/fathom/sub-skills/fathom-flashcards.md`

Both Form A (JSON, loadable into `~/Desktop/ea-flashcards.html`) and Form B (self-contained HTML, opens standalone) are written unless user specifies `/fathom flashcard-deck` (Form A only).

**v2.0 constraint:** Cards are organised by Field Map component — each component becomes a filterable tab in Form B. Threshold concepts (🔑) are flagged on the card front face.

**v2.0 constraint:** At least 4 of 10 questions must target threshold concepts (🔑).
If no Field Map has been generated yet, generate a quick one first (internal, not shown) to identify threshold concepts before writing questions.

---

## Step 5 — Horizon

*Runs on: full mode, `/fathom horizon`.*

Load and follow: `~/.claude/skills/fathom/sub-skills/fathom-horizon.md`

Returns two sections:
- **Did You Know** — historical/inspirational factoid, cited
- **Knowledge Frontier** — prerequisites table, adjacency map, recommended next fathom

---

## Step 6 — Vault Note

Always write (unless mode is quiz or brief):
`Learning/Fathom/[YYYY-MM-DD] - [Topic].md`

Use template: `~/.claude/skills/fathom/templates/vault-note-template.md`

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
DATE=$(date +%Y-%m-%d)
```

v2.0 sections added to the note: Field Map summary, Threshold Concepts, Prerequisite Gaps, Did You Know, Knowledge Frontier.

---

## Step 7 — Report

| Item | Value |
|------|-------|
| Subject | [subject] |
| Mode | [mode] |
| Field Map | [N] components · [M] threshold concepts · [P] gaps |
| Critical path | [ordered list] |
| Prerequisite gaps | [list, or "none identified"] |
| Notebook | [path, or "not generated"] |
| Did You Know | [1-line summary of factoid] |
| Next fathom | `/fathom [suggested topic]` |
| Vault note | `Learning/Fathom/[filename].md` |

---

*Fathom Skill v2.0 — 2026-03-27 — Claude Code (claude-sonnet-4-6)*
