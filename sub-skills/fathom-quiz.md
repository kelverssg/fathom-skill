# Fathom — Quiz Sub-Skill

Generate a 10-question comprehension quiz on a topic, score it interactively, and identify gaps for follow-up.

---

## Step 1 — Research the Topic

Read relevant vault files or source material before writing questions.

For a concept topic:
```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
# Check if a relevant vault note exists
ls "$VAULT/Philosophy/" | grep -i "[topic]"
ls "$VAULT/Learning/" | grep -i "[topic]"
```

For a codebase:
```bash
# Read key files to understand the concepts to test
cat [key files]
```

Do not write questions from memory alone — ground them in real source material.

---

## Step 2 — Write 10 Questions

Distribute across three difficulty levels:

| Level | Count | Format |
|-------|-------|--------|
| Recall (L1) | 3 | Multiple choice (4 options) |
| Application (L2) | 4 | Short answer or "what would happen if..." |
| Synthesis (L3) | 3 | "Explain the difference between...", trace-through, or scenario |

**Question quality rules:**
- One right answer per multiple choice — no "all of the above"
- Application questions must require *doing* something (not just recalling)
- Synthesis questions should reveal whether the concept is actually understood
- No trick questions — test understanding, not memory of phrasing

**Format:**

```markdown
## Quiz: [Topic]

*10 questions — answer before reading the answers. No peeking.*

---

**Q1 (Recall):** [Question text]

a) [Option]
b) [Option]
c) [Correct option]
d) [Option]

---

**Q2 (Recall):** [Question text]

a) [Option]
b) [Correct option]
c) [Option]
d) [Option]

---

**Q3 (Recall):** True or False — [Statement]

---

**Q4 (Application):** [Scenario or "what happens when..."]

---

**Q5 (Application):** Given the following code/situation, what is the output/result?

```[language]
[code or scenario]
```

---

**Q6 (Application):** [Another application question]

---

**Q7 (Application):** [Another application question]

---

**Q8 (Synthesis):** Explain the difference between [concept A] and [concept B].

---

**Q9 (Synthesis):** [Trace-through or causal chain question]

---

**Q10 (Synthesis):** You are building [scenario]. Which approach would you choose and why: [option A] or [option B]?

---

<details>
<summary>Answer Key</summary>

1. c — [brief reason]
2. b — [brief reason]
3. True/False — [reason]
4. [Expected answer]
5. [Expected output]
6. [Expected answer]
7. [Expected answer]
8. [Model answer — 2–3 sentences]
9. [Model answer]
10. [Model answer — both options are valid in different contexts, or one is clearly better and why]

**Score interpretation:**
- 9–10: Strong grasp. Proceed to application.
- 6–8: Solid foundation. Review the questions you missed.
- 3–5: Gaps in core concepts — revisit the Jupyter notebook or flashcards.
- 0–2: Start with the overview again. The fundamentals need more work.

</details>
```

---

## Step 3 — Present Interactively

Present the quiz to the user in one of two modes:

**Mode A — Print and self-score (default)**
Output all 10 questions at once. Ask the user to respond with their answers. When they do, score each answer and explain what they got right or wrong.

**Mode B — One at a time (if user requests "interactive quiz")**
Present each question individually. Wait for the user's answer. Respond immediately: "Correct — [reason]" or "Not quite — [reason, correct answer]". Track score. Summarise at the end.

---

## Step 4 — Identify Gaps

After scoring, identify which questions were missed and what concept gap they represent:

```
Gaps identified:
- Q3 (Recall): [concept] — recommend reviewing [specific section of notebook or flashcard]
- Q8 (Synthesis): [concept pair] — recommend revisiting [[vault note]] or running /fathom topic [concept]
```

---

## Step 5 — Append to Vault Note

If a fathom vault note exists for this topic, append a Quiz Results section:

```markdown
## Quiz Results — [date]

Score: N/10
Gaps: [list of missed concepts]
Re-test: [suggested date or "after reviewing X"]
```

---

*fathom-quiz sub-skill v1.0 — 2026-03-26*
