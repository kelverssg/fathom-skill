# Fathom — Flashcards Sub-Skill

Generate a flashcard deck from a topic. Two output forms — identical viewer code, different data loading.

- **Form A (deck):** JSON file only. Drag into `~/Desktop/ea-flashcards.html` to study.
- **Form B (self-contained):** Single HTML file. Open directly in any browser, no companion files needed.

**Output paths:**
- Form A JSON: `Learning/Fathom/[YYYY-MM-DD]-[Topic]-flashcards.json`
- Form B HTML: `Learning/Fathom/[YYYY-MM-DD]-[Topic]-Flashcards.html`

---

## Step 1 — Source the Field Map

If `/fathom flashcards` follows a full or map run in this session, use the Field Map already generated — component names become the `topic` field, threshold concepts are flagged `"threshold": true`.

If running standalone (e.g. `/fathom flashcards [topic]` fresh):
1. Check `Learning/Fathom/` for an existing vault note on this topic — read it for Field Map and Threshold Concepts
2. If no note exists: generate a quick internal Field Map (5–7 components) first — use it only to organise cards, don't print it

---

## Step 2 — Generate Cards

**Target: 15–25 cards total.** Spread across components in proportion to their importance. Threshold components (🔑) get 1 extra card each.

**Card types per component (mix freely):**
| Type | Front | Back |
|------|-------|------|
| Term | The term or concept name | Definition in plain language |
| Question | A "what is" or "why does" question | Answer, 1–3 sentences |
| Contrast | "What is the difference between X and Y?" | Crisp comparative answer |
| Threshold | The core question for a 🔑 concept | The portal insight — the thing that clicks |
| Analogy | "[Concept] is like what?" | The analogy (from Kelvin's domains if applicable) |

**Card content rules:**
- `front`: 1 line ideally, 2 max — concise prompt
- `back`: 2–5 lines — enough to self-grade against; not an essay
- Use `<br>` for line breaks, `<strong>` for key terms, `<em>` for technical names or foreign terms — nothing else
- `$...$` for inline LaTeX, `$$...$$` for display LaTeX (KaTeX renders both)
- `threshold: true` for any card that covers a 🔑 concept

**Produce as a JSON structure (in memory — goes into the builder in Step 4):**

```json
{
  "title": "[Topic]",
  "date": "[YYYY-MM-DD]",
  "cards": [
    {
      "id": "1",
      "topic": "[Component Name]",
      "front": "What is X?",
      "back": "X is ... [2–3 sentences]",
      "threshold": false
    },
    {
      "id": "2",
      "topic": "[Component Name]",
      "front": "Why does X matter for Y?",
      "back": "Because ... <br><strong>Key insight:</strong> ...",
      "threshold": true
    }
  ]
}
```

---

## Step 3 — Form A: Write JSON deck

Write the `cards` array (not the wrapper object) as a flat JSON file.
The ea-flashcards.html viewer expects a top-level array of `{id, topic, front, back}` objects.
Include `threshold` as an extra field — the viewer ignores unknown fields.

```python
import json, os
VAULT = "/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
path = f'{VAULT}/Learning/Fathom/{DATE}-{SLUG}-flashcards.json'
with open(path, 'w', encoding='utf-8') as f:
    json.dump(cards_list, f, ensure_ascii=False, indent=2)
print(f'Form A deck written: {path}')
```

Always write Form A. Form B is written additionally if the user asked for self-contained.

---

## Step 4 — Form B: Build self-contained HTML

Write `/tmp/build_flashcards.py` with the full card data. **Fill all `[PLACEHOLDER]` values before running.**

```python
#!/usr/bin/env python3
"""Build a self-contained Fathom flashcards HTML file."""
import json, os
from datetime import datetime

VAULT = "/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"

TITLE = "[TOPIC TITLE]"
DATE  = "[YYYY-MM-DD]"
SLUG  = "[topic-slug]"

# --- Card data (paste generated cards list here) ---
CARDS = [PASTE_CARDS_LIST_HERE]

# --- HTML template ---
HTML = """<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Flashcards: {title}</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css" crossorigin="anonymous">
<style>
*,*::before,*::after{{box-sizing:border-box;margin:0;padding:0}}
:root{{
  --bg:#F0F2F5;--surface:#FFFFFF;--border:#E5E7EB;
  --text-primary:#111827;--text-sec:#6B7280;--text-muted:#9CA3AF;
  --accent:#2563EB;--accent-h:#1D4ED8;
  --shadow-card:0 4px 24px rgba(0,0,0,0.09),0 1px 4px rgba(0,0,0,0.06);
  --shadow-toast:0 8px 32px rgba(0,0,0,0.18);
  --r:16px;--r-sm:9px;
  --font:'Segoe UI',system-ui,-apple-system,sans-serif
}}
body{{font-family:var(--font);background:var(--bg);color:var(--text-primary);min-height:100vh;display:flex;flex-direction:column;align-items:center;padding-bottom:60px}}
header{{width:100%;max-width:780px;padding:22px 20px 0;display:flex;align-items:center;justify-content:space-between}}
.logo{{font-size:1.125rem;font-weight:700;letter-spacing:-0.02em}}
.logo em{{color:var(--accent);font-style:normal}}
.card-count{{font-size:0.8rem;color:var(--text-muted);background:#E9EBF0;padding:3px 12px;border-radius:99px}}
#progress-area{{width:100%;max-width:780px;padding:16px 20px 0}}
.progress-meta{{display:flex;justify-content:space-between;font-size:0.8125rem;margin-bottom:7px;color:var(--text-sec)}}
.progress-meta strong{{color:var(--text-primary)}}
.progress-track{{height:4px;background:var(--border);border-radius:99px;overflow:hidden}}
.progress-fill{{height:100%;background:var(--accent);border-radius:99px;transition:width 0.35s ease}}
/* Topic tabs */
#tabs{{width:100%;max-width:780px;padding:14px 20px 0;display:flex;gap:8px;flex-wrap:wrap}}
.tab-btn{{padding:5px 14px;border:1.5px solid var(--border);border-radius:99px;background:var(--surface);color:var(--text-sec);font-size:0.78rem;font-weight:500;cursor:pointer;transition:all 0.15s;font-family:inherit}}
.tab-btn:hover{{border-color:var(--accent);color:var(--accent)}}
.tab-btn.active{{background:var(--accent);border-color:var(--accent);color:#fff;font-weight:600}}
/* Card scene */
#card-area{{width:100%;max-width:780px;padding:20px 20px 0;perspective:1400px}}
.card-scene{{width:100%;height:380px;position:relative;transform-style:preserve-3d;transition:transform 0.48s cubic-bezier(0.4,0,0.2,1);cursor:pointer;user-select:none}}
.card-face{{position:absolute;inset:0;background:var(--surface);border-radius:var(--r);box-shadow:var(--shadow-card);border:1px solid var(--border);padding:24px 28px 20px;backface-visibility:hidden;-webkit-backface-visibility:hidden;display:flex;flex-direction:column}}
.card-back-face{{transform:rotateY(180deg)}}
.face-header{{display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:16px}}
.topic-badge{{display:inline-flex;align-items:center;padding:3px 11px;border-radius:99px;font-size:0.7rem;font-weight:700;letter-spacing:0.04em;text-transform:uppercase;border:1.5px solid transparent}}
.threshold-badge{{font-size:0.85rem;line-height:1;cursor:default;title:'Threshold concept'}}
.face-label{{font-size:0.65rem;font-weight:700;text-transform:uppercase;letter-spacing:0.1em;color:var(--text-muted)}}
.card-body{{flex:1;display:flex;align-items:center;justify-content:center;overflow-y:auto;padding:4px 0}}
.card-body .inner{{width:100%;text-align:center;font-size:1.05rem;line-height:1.75}}
.card-body .katex-display{{margin:0.6em 0}}
.card-hint{{margin-top:12px;text-align:center;font-size:0.75rem;color:var(--text-muted)}}
kbd{{display:inline-block;padding:1px 6px;background:#F3F4F6;border:1px solid #D1D5DB;border-bottom-width:2px;border-radius:4px;font-size:0.7rem;font-family:monospace;color:var(--text-sec)}}
#controls{{display:flex;width:100%;max-width:780px;padding:18px 20px 0;gap:10px;justify-content:center}}
.btn{{padding:11px 26px;border-radius:var(--r-sm);font-size:0.9375rem;font-weight:600;cursor:pointer;border:1.5px solid;transition:all 0.15s;display:inline-flex;align-items:center;gap:7px;font-family:inherit}}
.btn-ghost{{background:var(--surface);border-color:var(--border);color:var(--text-primary)}}
.btn-ghost:hover{{border-color:var(--accent);color:var(--accent)}}
.btn-primary{{background:var(--accent);border-color:var(--accent);color:#fff}}
.btn-primary:hover{{background:var(--accent-h);border-color:var(--accent-h)}}
#toast{{position:fixed;bottom:28px;left:50%;transform:translateX(-50%) translateY(90px);background:#1F2937;color:#fff;padding:10px 22px;border-radius:var(--r-sm);font-size:0.875rem;font-weight:500;box-shadow:var(--shadow-toast);transition:transform 0.32s cubic-bezier(0.34,1.56,0.64,1);white-space:nowrap;z-index:999;pointer-events:none}}
#toast.show{{transform:translateX(-50%) translateY(0)}}
</style>
</head>
<body>

<header>
  <div class="logo">{title} <em>Flashcards</em></div>
  <span class="card-count">{n} cards</span>
</header>

<div id="progress-area">
  <div class="progress-meta">
    <span><strong id="progress-label">Card 1 of {n}</strong></span>
    <span id="progress-count">{n_minus_1} remaining</span>
  </div>
  <div class="progress-track"><div class="progress-fill" id="progress-fill" style="width:0%"></div></div>
</div>

<div id="tabs"></div>

<div id="card-area">
  <div class="card-scene" id="card-scene" onclick="flipCard()">
    <div class="card-face card-front-face">
      <div class="face-header">
        <span class="topic-badge" id="badge-front"></span>
        <div style="display:flex;align-items:center;gap:8px">
          <span class="threshold-badge" id="threshold-badge" style="display:none" title="Threshold concept">🔑</span>
          <span class="face-label">Prompt</span>
        </div>
      </div>
      <div class="card-body"><div class="inner" id="front-content"></div></div>
      <div class="card-hint">Click or press <kbd>Space</kbd> to reveal</div>
    </div>
    <div class="card-face card-back-face">
      <div class="face-header">
        <span class="topic-badge" id="badge-back"></span>
        <span class="face-label">Answer</span>
      </div>
      <div class="card-body"><div class="inner" id="back-content"></div></div>
      <div class="card-hint">Press <kbd>&#8594;</kbd> or <kbd>Enter</kbd> for next</div>
    </div>
  </div>
</div>

<div id="controls">
  <button class="btn btn-ghost" onclick="flipCard()">&#8635; Flip</button>
  <button class="btn btn-primary" onclick="nextCard()">Next &#8594;</button>
</div>

<div id="toast"></div>

<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js" crossorigin="anonymous"></script>
<script>
'use strict';

const CARDS = {cards_json};

/* ── Topic colours (auto-assigned) ── */
const PALETTE = [
  {{bg:'#EFF6FF',fg:'#1D4ED8',bd:'#BFDBFE'}},
  {{bg:'#F5F3FF',fg:'#6D28D9',bd:'#DDD6FE'}},
  {{bg:'#ECFDF5',fg:'#065F46',bd:'#A7F3D0'}},
  {{bg:'#FFF7ED',fg:'#C2410C',bd:'#FED7AA'}},
  {{bg:'#F0FDFA',fg:'#0F766E',bd:'#99F6E4'}},
  {{bg:'#FDF4FF',fg:'#7E22CE',bd:'#E9D5FF'}},
  {{bg:'#EEF2FF',fg:'#3730A3',bd:'#C7D2FE'}},
  {{bg:'#FFF1F2',fg:'#BE123C',bd:'#FECDD3'}},
];
const DEFAULT_COLOR = {{bg:'#F1F5F9',fg:'#334155',bd:'#CBD5E1'}};
const topicList = [...new Set(CARDS.map(c => c.topic || 'General'))];
const COLORS = {{}};
topicList.forEach((t, i) => {{ COLORS[t] = PALETTE[i % PALETTE.length]; }});

/* ── State ── */
let activeTopic = 'all';
let filteredDeck = [];
let remaining = [];
let current = null;
let isFlipped = false;

/* ── Build topic tabs ── */
function buildTabs() {{
  const el = document.getElementById('tabs');
  if (topicList.length <= 1) {{ el.style.display = 'none'; return; }}
  const allBtn = `<button class="tab-btn active" data-topic="all" onclick="setTopic('all',this)">All ({{CARDS.length}})</button>`;
  const topicBtns = topicList.map(t => {{
    const c = COLORS[t] || DEFAULT_COLOR;
    const n = CARDS.filter(c2 => c2.topic === t).length;
    return `<button class="tab-btn" data-topic="${{t}}" onclick="setTopic('${{t}}',this)">${{t}} (${{n}})</button>`;
  }}).join('');
  el.innerHTML = allBtn + topicBtns;
}}

function setTopic(topic, btn) {{
  activeTopic = topic;
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  filteredDeck = topic === 'all' ? [...CARDS] : CARDS.filter(c => c.topic === topic);
  remaining = shuffle(filteredDeck);
  drawCard();
  updateProgress();
}}

/* ── Shuffle ── */
function shuffle(arr) {{
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {{
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }}
  return a;
}}

/* ── Draw next card ── */
function drawCard() {{
  if (remaining.length === 0) {{
    remaining = shuffle(filteredDeck);
    toast('🔁 Deck complete — reshuffling…');
  }}
  current = remaining.pop();
  isFlipped = false;
  renderCard();
  updateProgress();
}}

/* ── Flip ── */
function flipCard() {{
  if (!current) return;
  isFlipped = !isFlipped;
  const scene = document.getElementById('card-scene');
  scene.style.transition = 'transform 0.48s cubic-bezier(0.4,0,0.2,1)';
  scene.style.transform = isFlipped ? 'rotateY(180deg)' : 'rotateY(0deg)';
}}

/* ── Render card ── */
function renderCard() {{
  if (!current) return;
  const scene = document.getElementById('card-scene');
  scene.style.transition = 'none';
  scene.style.transform = 'rotateY(0deg)';
  void scene.offsetWidth;

  document.getElementById('front-content').innerHTML = current.front;
  document.getElementById('back-content').innerHTML  = current.back;

  const topic = current.topic || 'General';
  const c = COLORS[topic] || DEFAULT_COLOR;
  ['badge-front','badge-back'].forEach(id => {{
    const el = document.getElementById(id);
    el.textContent = topic;
    el.style.background = c.bg;
    el.style.color = c.fg;
    el.style.borderColor = c.bd;
  }});

  const badge = document.getElementById('threshold-badge');
  badge.style.display = current.threshold ? 'inline' : 'none';

  renderMath(document.querySelector('.card-front-face'));
  renderMath(document.querySelector('.card-back-face'));
}}

/* ── KaTeX ── */
function renderMath(el) {{
  if (typeof renderMathInElement === 'function') {{
    renderMathInElement(el, {{
      delimiters: [
        {{left:'$$',right:'$$',display:true}},
        {{left:'$',right:'$',display:false}},
      ],
      throwOnError: false,
    }});
  }}
}}

/* ── Progress ── */
function updateProgress() {{
  const total = filteredDeck.length;
  const drawn = total - remaining.length;
  document.getElementById('progress-label').textContent = `Card ${{drawn}} of ${{total}}`;
  document.getElementById('progress-count').textContent = `${{remaining.length}} remaining`;
  document.getElementById('progress-fill').style.width = total > 0 ? (drawn/total*100)+'%' : '0%';
}}

/* ── Next card ── */
function nextCard() {{ drawCard(); }}

/* ── Toast ── */
let toastTimer = null;
function toast(msg) {{
  const el = document.getElementById('toast');
  el.textContent = msg;
  el.classList.add('show');
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => el.classList.remove('show'), 3200);
}}

/* ── Keyboard ── */
document.addEventListener('keydown', e => {{
  if (e.key === ' ' || e.key === 'Enter') {{ e.preventDefault(); isFlipped ? nextCard() : flipCard(); }}
  if (e.key === 'ArrowRight') nextCard();
  if (e.key === 'ArrowLeft')  {{ isFlipped = false; renderCard(); }}
}});

/* ── Init ── */
buildTabs();
filteredDeck = [...CARDS];
remaining = shuffle(filteredDeck);
drawCard();
</script>
</body>
</html>"""

# ── Build ──
n       = len(CARDS)
cards_j = json.dumps(CARDS, ensure_ascii=False)
out = HTML.format(
    title    = TITLE,
    n        = n,
    n_minus_1 = n - 1,
    cards_json = cards_j,
    date     = DATE,
)

path = f'{VAULT}/Learning/Fathom/{DATE}-{SLUG}-Flashcards.html'
os.makedirs(os.path.dirname(path), exist_ok=True)
with open(path, 'w', encoding='utf-8') as f:
    f.write(out)
print(f'Form B written: {path}  ({n} cards, {len(out):,} bytes)')
```

Run: `python3 /tmp/build_flashcards.py`

---

## Step 5 — Report

| Item | Value |
|------|-------|
| Cards generated | N total · M threshold (🔑) |
| Topics (tabs) | Component names |
| Form A (JSON) | `Learning/Fathom/[YYYY-MM-DD]-[slug]-flashcards.json` |
| Form B (HTML) | `Learning/Fathom/[YYYY-MM-DD]-[slug]-Flashcards.html` |
| Load in viewer | Drag Form A JSON into `~/Desktop/ea-flashcards.html` |

Both files written — Form B opens standalone, Form A is loadable into any ea-flashcards-compatible viewer.

---

## Notes

**Keeping the two forms in sync:** Form A is the JSON source of truth. Form B embeds the same JSON. If you regenerate, overwrite both. Do not hand-edit the HTML.

**ea-flashcards.html compatibility:** The `threshold` field in Form A is ignored by the loader (it only reads `front` and `back`). No action needed.

**Card front/back can contain:** `<br>`, `<strong>`, `<em>`, `<code>`, `$math$`, `$$math$$`. No other HTML.
