# Fathom — Horizon Sub-Skill v1.0

Generate the "Did You Know" historical factoid and the Knowledge Frontier map.
Two outputs: intellectual lineage (where this came from) + learning map (where it goes).

---

## Part A — Did You Know

### Step 1 — Research in parallel

**Brave search — history and breakthrough:**
```
mcp__brave-search__brave_web_search: "[subject] history origin who invented breakthrough discovery"
mcp__brave-search__brave_web_search: "[subject] foundational paper milestone first implementation"
```

**arXiv — foundational papers (technical/scientific topics):**
```
WebFetch: https://export.arxiv.org/api/query?search_query=all:[subject]&max_results=3&sortBy=relevance&sortOrder=descending
```
Prompt: "Extract the title, authors, year, and one-sentence summary of the most relevant paper."

Use the arXiv result if it surfaces a genuinely foundational paper (not just a recent survey).
Skip arXiv for non-technical topics (philosophy, history, language) — Brave is sufficient.

**GitHub MCP — for open-source projects:**
```
mcp__github__search_repositories: query="[subject] original"
```
Surface the original repo if relevant — first commit date, creator, founding context.

### Step 2 — Synthesise the factoid

Write 2–4 sentences in UK English, Housel × Gladwell style:
- The person, moment, or insight that made this possible
- Something non-obvious or surprising about the origin
- Why it connects to what we're learning today
- Short, specific, concrete — dinner conversation, not encyclopedia lede

Format:
```markdown
## Did You Know

[2–4 sentences.]

*Source: [Title, Author/Year — or URL]*
```

Quality bar: if it doesn't make you want to share it with someone, rewrite it.
If research returns nothing useful, synthesise from knowledge — but flag as "no external source found."

---

## Part B — Knowledge Frontier

### Step 1 — Prerequisite analysis

Reason about what [subject] assumes the learner already knows.
Assess each against Kelvin's profile: self-taught in CS, philosophy, physics — strong in risk/insurance, Stoicism/Daoism, languages, pattern recognition. Formal gaps may exist in: CS fundamentals, maths beyond calculus, formal logic, electrical engineering.

Output a table:

```markdown
### Prerequisite Gaps

| Concept | Likely status for Kelvin | If gap: entry point |
|---------|--------------------------|---------------------|
| [Concept] | Probably solid / Possibly shaky / Likely gap | [Suggested /fathom or resource] |
```

Flag the 1–2 most critical gaps explicitly. Don't list everything — only what genuinely blocks understanding.

### Step 2 — Adjacency and unlocks

Reason about the learning graph around [subject]:

```markdown
### What This Unlocks

- **[Concept A]** — [you can now understand this because...]
- **[Concept B]** — [this pattern recurs here...]

### Adjacent Territory Worth Exploring

- **[Topic X]** — [1 sentence: why it's nearby and relevant]
- **[Topic Y]** — [1 sentence]

### Unknown Unknowns

Things you didn't know to ask about, but that connect to [subject]:
- **[Concept Z]** — [brief explanation of the connection]
- **[Concept W]** — [brief explanation]
```

### Step 3 — Recent developments (active fields only)

If [subject] is an active area (ML, distributed systems, philosophy of mind, quantum computing, etc.):
Run: `/last30days [subject]`
Surface 1–2 recent developments that connect to the frontier — what's changing right now.

### Step 4 — Next fathom recommendation

Identify the single best next `/fathom` call based on:
- The most critical prerequisite gap (if any)
- OR the most interesting adjacency (if no major gaps)

```markdown
### Recommended Next Fathom

`/fathom [topic]` — [1 sentence: why this is the natural next step]
```

---

## Output Format

Return to the main skill as two labelled sections ready to paste into the vault note:

```
## Did You Know
[factoid + source]

---

## Knowledge Frontier
[prerequisite gaps table]
[unlocks]
[adjacent territory]
[unknown unknowns]
[recommended next fathom]
```

---

*fathom-horizon sub-skill v1.0 — 2026-03-27 — Claude Code*
