# Fathom — Quizmaker Sub-Skill

Generate a self-contained HTML+CSS+JS quiz file with multiple choice, true/false, and short answer questions. Opens directly in any browser. Short answer questions optionally use the Anthropic API for semantic grading.

**Output:** `Learning/Fathom/Quizzes/[YYYY-MM-DD]-[Topic]-quiz.html`

---

## Step 1 — Research the Topic

Same as fathom-quiz Step 1. Read relevant vault files or source material before writing questions.

```bash
VAULT="/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"
ls "$VAULT/Learning/Fathom/" | grep -i "[topic]"
```

If a fathom vault note exists for this topic, read it — the Field Map and Threshold Concepts are the primary source for question generation.

Do not write questions from memory alone.

---

## Step 2 — Generate 10 Questions as JSON

Produce the quiz as a JSON structure (do not write this to a file yet — it goes into the builder script in Step 3).

**Distribution:**
| Type | Count | Level |
|------|-------|-------|
| Multiple choice (4 options) | 4–5 | recall or application |
| True/False | 2–3 | recall only |
| Short answer | 2–3 | application or synthesis |

**Threshold constraint:** At least 4 of 10 must have `"threshold": true` — these test 🔑 portal concepts.

**JSON schema:**
```json
{
  "topic": "[Topic name]",
  "date": "[YYYY-MM-DD]",
  "questions": [
    {
      "id": 1,
      "type": "mc",
      "level": "recall",
      "threshold": false,
      "text": "Question text here.",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "answer": 2,
      "explanation": "Why this answer is correct — 1-2 sentences."
    },
    {
      "id": 2,
      "type": "tf",
      "level": "recall",
      "threshold": true,
      "text": "Statement that is true or false.",
      "answer": true,
      "explanation": "Why true/false — 1-2 sentences."
    },
    {
      "id": 3,
      "type": "short",
      "level": "synthesis",
      "threshold": true,
      "text": "Explain the difference between X and Y.",
      "model_answer": "A complete model answer — 2-4 sentences covering the key points.",
      "grading_rubric": "Key points: (1) X does A, (2) Y does B, (3) the distinction matters because C."
    }
  ]
}
```

**Question quality rules (same as fathom-quiz.md):**
- One right answer per MC — no "all of the above"
- T/F statements must be unambiguously true or false
- Short answer questions should require *doing* something, not just recalling
- Synthesis questions reveal whether the concept is actually understood
- `answer` for MC is a 0-based index into `options`
- `model_answer` should be thorough enough to self-grade against

---

## Step 3 — Build the HTML

Write `/tmp/build_quiz.py` with the quiz JSON embedded. Fill ALL `[PLACEHOLDER]` values before running.

```python
#!/usr/bin/env python3
"""Build a fathom quiz HTML file."""
import json, os, sys
from datetime import datetime

VAULT = "/Users/macbookair2022/Library/Mobile Documents/iCloud~md~obsidian/Documents/Kelvers"

# --- Quiz data (fill this in) ---
QUIZ = [PASTE_QUIZ_JSON_HERE]

# --- HTML template ---
HTML = """<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Quiz: {topic}</title>
<style>
*{{box-sizing:border-box;margin:0;padding:0}}
body{{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;background:#f1f5f9;color:#1a1a2e;min-height:100vh}}
.container{{max-width:720px;margin:0 auto;padding:24px 16px 60px}}
.quiz-header{{background:#fff;border-radius:12px;padding:24px;margin-bottom:20px;box-shadow:0 1px 3px rgba(0,0,0,.08)}}
h1{{font-size:1.4rem;font-weight:700;color:#1a1a2e;margin-bottom:4px}}
.subtitle{{font-size:.85rem;color:#6b7280}}
.controls{{display:flex;align-items:center;justify-content:space-between;margin-top:16px;flex-wrap:wrap;gap:8px}}
.toggle-group{{display:flex;background:#f3f4f6;border-radius:8px;padding:3px}}
.toggle-btn{{padding:6px 14px;border:none;background:transparent;border-radius:6px;font-size:.8rem;cursor:pointer;color:#6b7280;transition:all .15s}}
.toggle-btn.active{{background:#fff;color:#4f46e5;font-weight:600;box-shadow:0 1px 2px rgba(0,0,0,.1)}}
.right-controls{{display:flex;gap:8px;align-items:center}}
.settings-btn{{padding:6px 12px;border:1px solid #e5e7eb;border-radius:8px;background:#fff;cursor:pointer;font-size:.8rem;color:#6b7280}}
.settings-btn:hover{{background:#f9fafb}}
.ai-badge{{font-size:.72rem;padding:3px 8px;border-radius:20px;background:#ecfdf5;color:#059669;border:1px solid #a7f3d0;display:none}}
.ai-badge.show{{display:inline-block}}
.progress-bar{{height:4px;background:#e5e7eb;border-radius:2px;margin-top:16px}}
.progress-fill{{height:100%;background:#4f46e5;border-radius:2px;transition:width .3s ease}}
.progress-label{{font-size:.72rem;color:#9ca3af;margin-top:5px}}
.question-card{{background:#fff;border-radius:12px;padding:24px 28px;margin-bottom:14px;box-shadow:0 1px 3px rgba(0,0,0,.08);border-left:4px solid transparent;transition:border-color .2s}}
.question-card.answered-correct{{border-left-color:#10b981}}
.question-card.answered-incorrect{{border-left-color:#ef4444}}
.question-card.answered-partial{{border-left-color:#f59e0b}}
.q-meta{{display:flex;align-items:center;gap:8px;margin-bottom:12px;flex-wrap:wrap}}
.q-num{{font-size:.72rem;font-weight:600;color:#9ca3af}}
.q-level{{font-size:.68rem;padding:2px 8px;border-radius:20px;font-weight:500}}
.level-recall{{background:#eff6ff;color:#3b82f6}}
.level-application{{background:#fdf4ff;color:#a855f7}}
.level-synthesis{{background:#fff7ed;color:#f97316}}
.q-text{{font-size:.975rem;line-height:1.65;color:#1a1a2e;margin-bottom:18px;font-weight:500}}
code{{font-family:'SF Mono','Fira Code',monospace;background:#f3f4f6;padding:1px 5px;border-radius:4px;font-size:.875em}}
pre{{background:#1e293b;border-radius:8px;padding:14px;margin:12px 0;overflow-x:auto}}
pre code{{background:transparent;padding:0;color:#e2e8f0;font-size:.85em;line-height:1.6}}
.options{{display:flex;flex-direction:column;gap:9px}}
.option{{padding:11px 15px;border:2px solid #e5e7eb;border-radius:8px;cursor:pointer;transition:all .15s;display:flex;align-items:flex-start;gap:10px;font-size:.9rem;line-height:1.5}}
.option:hover:not(.disabled){{border-color:#a5b4fc;background:#eef2ff}}
.option.selected{{border-color:#4f46e5;background:#eef2ff}}
.option.correct{{border-color:#10b981;background:#ecfdf5;color:#065f46}}
.option.incorrect{{border-color:#ef4444;background:#fef2f2;color:#991b1b}}
.option.reveal-correct{{border-color:#10b981;background:#ecfdf5}}
.option.disabled{{cursor:default}}
.opt-label{{font-weight:700;font-size:.78rem;min-width:18px;color:#9ca3af;padding-top:1px}}
.option.selected .opt-label{{color:#4f46e5}}
.option.correct .opt-label,.option.reveal-correct .opt-label{{color:#10b981}}
.option.incorrect .opt-label{{color:#ef4444}}
.tf-options{{display:flex;gap:12px}}
.tf-btn{{flex:1;padding:13px;border:2px solid #e5e7eb;border-radius:8px;cursor:pointer;font-weight:600;font-size:.9rem;text-align:center;transition:all .15s;background:#fff}}
.tf-btn:hover:not(.disabled){{border-color:#a5b4fc;background:#eef2ff}}
.tf-btn.selected{{border-color:#4f46e5;background:#eef2ff;color:#4f46e5}}
.tf-btn.correct{{border-color:#10b981;background:#ecfdf5;color:#065f46}}
.tf-btn.incorrect{{border-color:#ef4444;background:#fef2f2;color:#991b1b}}
.tf-btn.reveal-correct{{border-color:#10b981;background:#ecfdf5;color:#065f46}}
.tf-btn.disabled{{cursor:default}}
.short-area textarea{{width:100%;padding:11px 13px;border:2px solid #e5e7eb;border-radius:8px;font-size:.9rem;font-family:inherit;resize:vertical;min-height:80px;transition:border-color .15s;line-height:1.5}}
.short-area textarea:focus{{outline:none;border-color:#a5b4fc}}
.short-area textarea:disabled{{background:#f9fafb;color:#6b7280}}
.btn{{padding:9px 18px;border-radius:8px;border:none;cursor:pointer;font-size:.85rem;font-weight:500;transition:all .15s}}
.btn-primary{{background:#4f46e5;color:#fff}}
.btn-primary:hover{{background:#4338ca}}
.btn-sm{{padding:6px 13px;font-size:.78rem}}
.btn-got-it{{background:#ecfdf5;color:#065f46;border:1px solid #6ee7b7}}
.btn-got-it:hover{{background:#d1fae5}}
.btn-partial{{background:#fffbeb;color:#92400e;border:1px solid #fcd34d}}
.btn-partial:hover{{background:#fef3c7}}
.btn-missed{{background:#fef2f2;color:#991b1b;border:1px solid #fca5a5}}
.btn-missed:hover{{background:#fee2e2}}
.btn-ai{{background:#f0fdf4;color:#166534;border:1px solid #86efac;font-size:.76rem;padding:5px 11px}}
.btn-ai:hover:not(:disabled){{background:#dcfce7}}
.btn-ai:disabled{{opacity:.5;cursor:not-allowed}}
.action-row{{display:flex;gap:9px;margin-top:13px;align-items:center;flex-wrap:wrap}}
.explanation{{margin-top:15px;padding:13px 15px;background:#f8fafc;border-radius:8px;border-left:3px solid #94a3b8;font-size:.865rem;line-height:1.65;color:#475569;display:none}}
.explanation.show{{display:block}}
.explanation strong{{color:#1a1a2e}}
.model-answer-label{{font-size:.7rem;font-weight:600;color:#6b7280;text-transform:uppercase;letter-spacing:.05em;margin-bottom:5px;margin-top:10px}}
.model-answer-box{{padding:11px;background:#fff;border:1px solid #e5e7eb;border-radius:6px;font-size:.875rem;line-height:1.55}}
.ai-feedback{{margin-top:10px;padding:11px 13px;background:#f0fdf4;border-radius:6px;border:1px solid #86efac;font-size:.845rem;line-height:1.5;color:#166534;display:none}}
.ai-feedback.show{{display:block}}
.ai-score-tag{{font-weight:700;font-size:.8rem;padding:2px 7px;border-radius:4px;margin-right:6px}}
.score-0{{background:#fef2f2;color:#991b1b}}
.score-1{{background:#fffbeb;color:#92400e}}
.score-2{{background:#ecfdf5;color:#065f46}}
.end-screen{{display:none}}
.end-screen.show{{display:block}}
.score-card{{background:#fff;border-radius:12px;padding:36px 28px;text-align:center;box-shadow:0 1px 3px rgba(0,0,0,.08);margin-bottom:18px}}
.score-big{{font-size:4rem;font-weight:800;color:#4f46e5;line-height:1}}
.score-denom{{font-size:2rem;color:#9ca3af}}
.score-label{{font-size:.95rem;color:#6b7280;margin-top:8px}}
.score-interp{{margin-top:18px;padding:13px 18px;background:#f8fafc;border-radius:8px;font-size:.875rem;color:#374151;font-style:italic}}
.breakdown{{margin-top:24px;display:grid;grid-template-columns:repeat(3,1fr);gap:10px;text-align:center}}
.bd-item{{padding:13px 8px;background:#f8f9fa;border-radius:8px}}
.bd-num{{font-size:1.4rem;font-weight:700;color:#1a1a2e}}
.bd-label{{font-size:.7rem;color:#6b7280;margin-top:2px}}
.threshold-row{{margin-top:14px;font-size:.85rem;color:#6b7280}}
.threshold-row strong{{color:#1a1a2e}}
.gaps-card{{background:#fff;border-radius:12px;padding:22px 24px;box-shadow:0 1px 3px rgba(0,0,0,.08)}}
.gaps-title{{font-size:.95rem;font-weight:700;margin-bottom:14px}}
.gap-item{{padding:11px 13px;border-left:3px solid #ef4444;background:#fef2f2;border-radius:0 6px 6px 0;margin-bottom:9px;font-size:.855rem;color:#374151;line-height:1.5}}
.gap-item.partial{{border-left-color:#f59e0b;background:#fffbeb}}
.gap-q-num{{font-size:.7rem;font-weight:600;color:#9ca3af;margin-bottom:3px}}
.no-gaps{{text-align:center;padding:20px;color:#6b7280;font-size:.9rem}}
.end-actions{{display:flex;gap:12px;justify-content:center;margin-top:20px;flex-wrap:wrap}}
.modal-overlay{{position:fixed;inset:0;background:rgba(0,0,0,.4);display:flex;align-items:center;justify-content:center;z-index:100;opacity:0;pointer-events:none;transition:opacity .2s}}
.modal-overlay.show{{opacity:1;pointer-events:auto}}
.modal{{background:#fff;border-radius:12px;padding:26px;max-width:420px;width:90%;transform:scale(.95);transition:transform .2s}}
.modal-overlay.show .modal{{transform:scale(1)}}
.modal h3{{margin-bottom:7px;font-size:1rem}}
.modal p{{font-size:.83rem;color:#6b7280;margin-bottom:14px;line-height:1.55}}
.modal input{{width:100%;padding:9px 12px;border:2px solid #e5e7eb;border-radius:8px;font-size:.85rem;font-family:monospace;margin-bottom:14px}}
.modal input:focus{{outline:none;border-color:#a5b4fc}}
.modal-actions{{display:flex;gap:8px;justify-content:flex-end}}
.hidden{{display:none!important}}
@media(max-width:500px){{
  .breakdown{{grid-template-columns:repeat(3,1fr);gap:6px}}
  .score-big{{font-size:3rem}}
  .tf-btn{{padding:11px 6px;font-size:.82rem}}
}}
</style>
</head>
<body>
<div class="container">

  <!-- Header -->
  <div class="quiz-header">
    <h1>Quiz: {topic}</h1>
    <div class="subtitle">{date} &nbsp;·&nbsp; {n} questions &nbsp;·&nbsp; Fathom v2.0</div>
    <div class="controls">
      <div class="toggle-group">
        <button class="toggle-btn active" id="btn-after" onclick="setMode('after')">After each</button>
        <button class="toggle-btn" id="btn-end" onclick="setMode('end')">At the end</button>
      </div>
      <div class="right-controls">
        <span class="ai-badge" id="ai-badge">✦ AI grading on</span>
        <button class="settings-btn" onclick="openSettings()">⚙ API key</button>
      </div>
    </div>
    <div class="progress-bar"><div class="progress-fill" id="progress" style="width:0%"></div></div>
    <div class="progress-label" id="progress-label">0 / {n} answered</div>
  </div>

  <!-- Questions -->
  <div id="questions-area">
    {questions_html}
  </div>

  <!-- Submit all (end mode) -->
  <div id="submit-all-row" class="hidden" style="text-align:center;margin-bottom:20px">
    <button class="btn btn-primary" onclick="revealAll()">Show results</button>
  </div>

  <!-- End screen -->
  <div class="end-screen" id="end-screen">
    <div class="score-card">
      <div class="score-big" id="final-score">0</div>
      <div class="score-label">out of {n} points</div>
      <div class="score-interp" id="score-interp"></div>
      <div class="breakdown">
        <div class="bd-item"><div class="bd-num" id="bd-mc">—</div><div class="bd-label">Multiple choice</div></div>
        <div class="bd-item"><div class="bd-num" id="bd-tf">—</div><div class="bd-label">True / False</div></div>
        <div class="bd-item"><div class="bd-num" id="bd-sh">—</div><div class="bd-label">Short answer</div></div>
      </div>
      <div class="threshold-row">Threshold concepts 🔑: <strong id="threshold-score">—</strong></div>
    </div>
    <div class="gaps-card">
      <div class="gaps-title">Gaps to revisit</div>
      <div id="gaps-list"></div>
    </div>
    <div class="end-actions">
      <button class="btn btn-primary" onclick="window.location.reload()">Retake quiz</button>
    </div>
  </div>

</div>

<!-- Settings modal -->
<div class="modal-overlay" id="modal" onclick="closeSettings(event)">
  <div class="modal">
    <h3>OpenRouter API key</h3>
    <p>Used only for short answer semantic grading. Stored in your browser (localStorage). Never sent anywhere except openrouter.ai.</p>
    <input type="password" id="api-key-input" placeholder="sk-or-..." />
    <div class="modal-actions">
      <button class="btn btn-sm" style="background:#f3f4f6;border:1px solid #e5e7eb" onclick="clearKey()">Clear</button>
      <button class="btn btn-sm btn-primary" onclick="saveKey()">Save</button>
    </div>
  </div>
</div>

<script>
const QUIZ = {quiz_json};

// State
let feedbackMode = localStorage.getItem('fathom_quiz_mode') || 'after';
let answers = {{}};
let shortScores = {{}};
let answered = new Set();

// Init
(function init() {{
  // Mode
  if (feedbackMode === 'end') {{
    document.getElementById('btn-after').classList.remove('active');
    document.getElementById('btn-end').classList.add('active');
    document.getElementById('submit-all-row').classList.remove('hidden');
  }}
  // API key badge
  if (localStorage.getItem('openrouter_key')) {{
    document.getElementById('ai-badge').classList.add('show');
    document.querySelectorAll('.btn-ai').forEach(b => b.style.display = 'inline-block');
  }} else {{
    document.querySelectorAll('.btn-ai').forEach(b => b.style.display = 'none');
  }}
}})();

function setMode(m) {{
  feedbackMode = m;
  localStorage.setItem('fathom_quiz_mode', m);
  document.getElementById('btn-after').classList.toggle('active', m === 'after');
  document.getElementById('btn-end').classList.toggle('active', m === 'end');
  document.getElementById('submit-all-row').classList.toggle('hidden', m !== 'end');
}}

// Progress
function updateProgress() {{
  const n = QUIZ.questions.length;
  const done = answered.size;
  document.getElementById('progress').style.width = (done/n*100) + '%';
  document.getElementById('progress-label').textContent = done + ' / ' + n + ' answered';
  if (done === n) setTimeout(showEndScreen, feedbackMode === 'after' ? 600 : 0);
}}

// MC
function selectOption(qid, idx) {{
  const q = QUIZ.questions.find(q => q.id === qid);
  if (answered.has(qid)) return;
  if (feedbackMode === 'end') {{
    answers[qid] = idx;
    const card = document.getElementById('q-' + qid);
    card.querySelectorAll('.option').forEach((o, i) => {{
      o.classList.toggle('selected', i === idx);
    }});
    markAnswered(qid, null);
    return;
  }}
  answers[qid] = idx;
  const correct = idx === q.answer;
  const card = document.getElementById('q-' + qid);
  card.querySelectorAll('.option').forEach((o, i) => {{
    o.classList.add('disabled');
    if (i === idx) o.classList.add(correct ? 'correct' : 'incorrect');
    if (!correct && i === q.answer) o.classList.add('reveal-correct');
  }});
  markAnswered(qid, correct ? 2 : 0);
  showExplanation(qid);
}}

// T/F
function selectTF(qid, val) {{
  const q = QUIZ.questions.find(q => q.id === qid);
  if (answered.has(qid)) return;
  if (feedbackMode === 'end') {{
    answers[qid] = val;
    const card = document.getElementById('q-' + qid);
    card.querySelectorAll('.tf-btn').forEach(b => {{
      b.classList.toggle('selected', b.dataset.val === String(val));
    }});
    markAnswered(qid, null);
    return;
  }}
  answers[qid] = val;
  const correct = val === q.answer;
  const card = document.getElementById('q-' + qid);
  card.querySelectorAll('.tf-btn').forEach(b => {{
    b.classList.add('disabled');
    const bval = b.dataset.val === 'true';
    if (bval === val) b.classList.add(correct ? 'correct' : 'incorrect');
    if (!correct && bval === q.answer) b.classList.add('reveal-correct');
  }});
  markAnswered(qid, correct ? 2 : 0);
  showExplanation(qid);
}}

// Short answer submit
function submitShort(qid) {{
  if (answered.has(qid)) return;
  const ta = document.getElementById('ta-' + qid);
  const text = ta.value.trim();
  if (!text) return;
  ta.disabled = true;
  document.getElementById('submit-' + qid).style.display = 'none';
  showExplanation(qid);
  answers[qid] = text;
  markAnswered(qid, null);  // score set by self-grade
  // Show AI button if key exists
  if (localStorage.getItem('openrouter_key')) {{
    document.getElementById('ai-btn-' + qid).style.display = 'inline-block';
  }}
}}

function selfGrade(qid, score) {{
  shortScores[qid] = score;
  const card = document.getElementById('q-' + qid);
  card.classList.remove('answered-partial', 'answered-correct', 'answered-incorrect');
  if (score === 2) card.classList.add('answered-correct');
  else if (score === 1) card.classList.add('answered-partial');
  else card.classList.add('answered-incorrect');
  // Disable grade buttons
  card.querySelectorAll('.grade-btn').forEach(b => b.disabled = true);
  updateProgress();
}}

// AI grading
async function getAIFeedback(qid) {{
  const q = QUIZ.questions.find(q => q.id === qid);
  const key = localStorage.getItem('openrouter_key');
  if (!key) return;
  const btn = document.getElementById('ai-btn-' + qid);
  btn.disabled = true;
  btn.textContent = 'Grading…';
  const fb = document.getElementById('ai-fb-' + qid);
  fb.classList.add('show');
  fb.textContent = 'Asking Claude…';
  try {{
    const resp = await fetch('https://openrouter.ai/api/v1/chat/completions', {{
      method: 'POST',
      headers: {{
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + key,
        'HTTP-Referer': 'https://fathom.ai',
        'X-Title': 'Fathom Quizmaker'
      }},
      body: JSON.stringify({{
        model: 'google/gemini-2.0-flash-001',
        max_tokens: 400,
        messages: [{{
          role: 'user',
          content: `Grade this short answer on a scale of 0-2. Be concise.

Question: ${{q.text}}
Model answer: ${{q.model_answer}}
Grading rubric: ${{q.grading_rubric}}
Student answer: ${{answers[qid]}}

Respond in exactly this format:
SCORE: [0/1/2]
FEEDBACK: [1-2 sentences explaining what they got right or wrong]`
        }}]
      }})
    }});
    const data = await resp.json();
    if (data.error || !data.choices || !data.choices[0]) {{
      throw new Error(data.error?.message || data.error?.code || 'API error — check key');
    }}
    const raw = data.choices[0].message.content || data.choices[0].message.refusal || '';
    if (!raw) throw new Error('Model returned empty content');
    const sm = raw.match(/SCORE:\s*(\d)/);
    const fm = raw.match(/FEEDBACK:\s*(.+)/s);
    const aiScore = sm ? parseInt(sm[1]) : null;
    const aiText = fm ? fm[1].trim() : raw;
    let scoreTag = '';
    if (aiScore !== null) {{
      scoreTag = `<span class="ai-score-tag score-${{aiScore}}">${{['Missed','Partial','Got it'][aiScore]}}</span>`;
      // Auto-apply self-grade from AI
      selfGrade(qid, aiScore);
    }}
    fb.innerHTML = scoreTag + aiText;
    btn.textContent = '✓ Graded';
  }} catch(e) {{
    fb.textContent = 'Error: ' + (e.message || 'check API key');
    btn.disabled = false;
    btn.textContent = 'Get AI feedback';
  }}
}}

function showExplanation(qid) {{
  document.getElementById('expl-' + qid).classList.add('show');
}}

function markAnswered(qid, score) {{
  answered.add(qid);
  const card = document.getElementById('q-' + qid);
  if (score !== null) {{
    card.classList.add(score > 0 ? 'answered-correct' : 'answered-incorrect');
  }}
  if (feedbackMode === 'after') updateProgress();
}}

// End mode: reveal all
function revealAll() {{
  QUIZ.questions.forEach(q => {{
    if (answered.has(q.id)) return;
    if (q.type === 'mc') {{
      const sel = answers[q.id];
      if (sel !== undefined) {{
        const correct = sel === q.answer;
        const card = document.getElementById('q-' + q.id);
        card.querySelectorAll('.option').forEach((o, i) => {{
          o.classList.add('disabled');
          if (i === sel) o.classList.add(correct ? 'correct' : 'incorrect');
          if (!correct && i === q.answer) o.classList.add('reveal-correct');
        }});
        markAnswered(q.id, correct ? 2 : 0);
        showExplanation(q.id);
      }}
    }} else if (q.type === 'tf') {{
      const sel = answers[q.id];
      if (sel !== undefined) {{
        const correct = sel === q.answer;
        const card = document.getElementById('q-' + q.id);
        card.querySelectorAll('.tf-btn').forEach(b => {{
          b.classList.add('disabled');
          const bval = b.dataset.val === 'true';
          if (bval === sel) b.classList.add(correct ? 'correct' : 'incorrect');
          if (!correct && bval === q.answer) b.classList.add('reveal-correct');
        }});
        markAnswered(q.id, correct ? 2 : 0);
        showExplanation(q.id);
      }}
    }}
  }});
  document.getElementById('submit-all-row').classList.add('hidden');
  answered.forEach(qid => {{}}); // trigger progress
  updateProgress();
}}

function showEndScreen() {{
  // Calculate scores
  let total = 0, maxTotal = 0;
  let mcRight = 0, mcTotal = 0;
  let tfRight = 0, tfTotal = 0;
  let shTotal = 0, shRight = 0;
  let threshRight = 0, threshTotal = 0;
  const gaps = [];
  QUIZ.questions.forEach(q => {{
    let pts = 0, maxPts = 2;
    if (q.type === 'mc' || q.type === 'tf') {{
      const correct = (q.type === 'mc') ? answers[q.id] === q.answer : answers[q.id] === q.answer;
      pts = correct ? 2 : 0;
      if (q.type === 'mc') {{ mcRight += correct ? 1 : 0; mcTotal++; }}
      else {{ tfRight += correct ? 1 : 0; tfTotal++; }}
      if (!correct) gaps.push({{id: q.id, level: q.level, text: q.text, partial: false, threshold: q.threshold}});
    }} else {{
      pts = shortScores[q.id] ?? 0;
      shTotal++;
      shRight += pts;
      if (pts < 2) gaps.push({{id: q.id, level: q.level, text: q.text, partial: pts === 1, threshold: q.threshold}});
    }}
    total += pts;
    maxTotal += maxPts;
    if (q.threshold) {{ threshRight += pts === 2 ? 1 : 0; threshTotal++; }}
  }});
  const pct = Math.round(total / maxTotal * 10);
  document.getElementById('final-score').textContent = pct;
  document.getElementById('bd-mc').textContent = mcTotal ? mcRight + '/' + mcTotal : '—';
  document.getElementById('bd-tf').textContent = tfTotal ? tfRight + '/' + tfTotal : '—';
  document.getElementById('bd-sh').textContent = shTotal ? shRight + '/' + (shTotal * 2) + ' pts' : '—';
  document.getElementById('threshold-score').textContent = threshRight + ' / ' + threshTotal + ' 🔑';
  const interps = [
    [9,10,'Strong grasp. Proceed to application.'],
    [6,8,'Solid foundation. Review the questions you missed.'],
    [3,5,'Gaps in core concepts — revisit the Jupyter notebook or flashcards.'],
    [0,2,'Start with the overview again. The fundamentals need more work.']
  ];
  const interp = interps.find(([lo,hi]) => pct >= lo && pct <= hi);
  document.getElementById('score-interp').textContent = interp ? interp[2] : '';
  const gapsEl = document.getElementById('gaps-list');
  if (gaps.length === 0) {{
    gapsEl.innerHTML = '<div class="no-gaps">No gaps — perfect score! 🎉</div>';
  }} else {{
    gapsEl.innerHTML = gaps.map(g =>
      `<div class="gap-item${{g.partial ? ' partial' : ''}}"><div class="gap-q-num">Q${{g.id}}${{g.threshold ? ' 🔑' : ''}} · ${{g.level}}</div>${{g.text}}</div>`
    ).join('');
  }}
  document.querySelector('.end-screen').classList.add('show');
  document.querySelector('.end-screen').scrollIntoView({{behavior:'smooth'}});
}}

// Settings modal
function openSettings() {{
  const key = localStorage.getItem('openrouter_key') || '';
  document.getElementById('api-key-input').value = key;
  document.getElementById('modal').classList.add('show');
}}
function closeSettings(e) {{
  if (e.target === document.getElementById('modal')) document.getElementById('modal').classList.remove('show');
}}
function saveKey() {{
  const k = document.getElementById('api-key-input').value.trim();
  if (k) {{
    localStorage.setItem('openrouter_key', k);
    document.getElementById('ai-badge').classList.add('show');
    document.querySelectorAll('.btn-ai').forEach(b => b.style.display = 'inline-block');
  }}
  document.getElementById('modal').classList.remove('show');
}}
function clearKey() {{
  localStorage.removeItem('openrouter_key');
  document.getElementById('ai-badge').classList.remove('show');
  document.querySelectorAll('.btn-ai').forEach(b => b.style.display = 'none');
  document.getElementById('api-key-input').value = '';
}}
</script>
</body>
</html>"""

LEVEL_CLASS = {"recall": "level-recall", "application": "level-application", "synthesis": "level-synthesis"}
LABELS = ["A", "B", "C", "D", "E"]

def render_question(q):
    qid = q["id"]
    level = q.get("level", "recall")
    threshold = "🔑 " if q.get("threshold") else ""
    meta = (f'<div class="q-meta">'
            f'<span class="q-num">Q{qid}</span>'
            f'<span class="q-level {LEVEL_CLASS.get(level, "")}">{level.title()}</span>'
            f'<span class="q-threshold">{threshold}</span>'
            f'</div>')
    text = f'<div class="q-text">{q["text"]}</div>'

    if q["type"] == "mc":
        opts = "".join(
            f'<div class="option" onclick="selectOption({qid},{i})">'
            f'<span class="opt-label">{LABELS[i]}</span>{o}</div>'
            for i, o in enumerate(q["options"])
        )
        body = f'<div class="options">{opts}</div>'
        expl = f'<div class="explanation" id="expl-{qid}"><strong>Answer: {LABELS[q["answer"]]}</strong> — {q.get("explanation","")}</div>'

    elif q["type"] == "tf":
        body = (f'<div class="tf-options">'
                f'<button class="tf-btn" data-val="true" onclick="selectTF({qid},true)">True</button>'
                f'<button class="tf-btn" data-val="false" onclick="selectTF({qid},false)">False</button>'
                f'</div>')
        ans_str = "True" if q["answer"] else "False"
        expl = f'<div class="explanation" id="expl-{qid}"><strong>Answer: {ans_str}</strong> — {q.get("explanation","")}</div>'

    elif q["type"] == "short":
        body = (f'<div class="short-area">'
                f'<textarea id="ta-{qid}" placeholder="Type your answer…" rows="3"></textarea>'
                f'<div class="action-row">'
                f'<button class="btn btn-primary btn-sm" id="submit-{qid}" onclick="submitShort({qid})">Submit</button>'
                f'<button class="btn btn-ai btn-sm" id="ai-btn-{qid}" onclick="getAIFeedback({qid})" style="display:none">✦ Get AI feedback</button>'
                f'</div>'
                f'</div>')
        model = q.get("model_answer","")
        expl = (f'<div class="explanation" id="expl-{qid}">'
                f'<div class="model-answer-label">Model answer</div>'
                f'<div class="model-answer-box">{model}</div>'
                f'<div class="action-row" style="margin-top:10px">'
                f'<button class="btn btn-sm btn-got-it grade-btn" onclick="selfGrade({qid},2)">✓ Got it</button>'
                f'<button class="btn btn-sm btn-partial grade-btn" onclick="selfGrade({qid},1)">~ Partial</button>'
                f'<button class="btn btn-sm btn-missed grade-btn" onclick="selfGrade({qid},0)">✗ Missed</button>'
                f'</div>'
                f'<div class="ai-feedback" id="ai-fb-{qid}"></div>'
                f'</div>')
    else:
        body = ""
        expl = ""

    return (f'<div class="question-card" id="q-{qid}">'
            f'{meta}{text}{body}{expl}'
            f'</div>')

questions_html = "\n".join(render_question(q) for q in QUIZ["questions"])
topic = QUIZ["topic"]
date = QUIZ.get("date", datetime.now().strftime("%Y-%m-%d"))
n = len(QUIZ["questions"])
quiz_json = json.dumps(QUIZ)

html = HTML.format(
    topic=topic,
    date=date,
    n=n,
    questions_html=questions_html,
    quiz_json=quiz_json,
)

output_dir = os.path.join(VAULT, "Learning/Fathom/Quizzes")
os.makedirs(output_dir, exist_ok=True)
slug = topic.lower().replace(" ", "-").replace("/", "-").replace(".", "-")
filename = f"{date}-{slug}-quiz.html"
output_path = os.path.join(output_dir, filename)

with open(output_path, "w") as f:
    f.write(html)

print(f"Quiz written: {output_path}")
print(f"Questions: {n}")
print(f"Open: open '{output_path}'")
```

**Before running:** Replace `[PASTE_QUIZ_JSON_HERE]` with the actual quiz dict (not a string — the raw Python dict literal, or `json.loads("""...""")` wrapping the JSON string).

Then execute:
```bash
python3 /tmp/build_quiz.py
```

---

## Step 4 — Verify

```bash
python3 -c "
import json, re
with open('[OUTPUT_PATH]') as f:
    html = f.read()
match = re.search(r'const QUIZ = (\{.*?\});', html, re.DOTALL)
quiz = json.loads(match.group(1))
print(f'Questions: {len(quiz[\"questions\"])}')
types = [q[\"type\"] for q in quiz[\"questions\"]]
threshold = sum(1 for q in quiz[\"questions\"] if q.get(\"threshold\"))
print(f'Types: MC={types.count(\"mc\")} TF={types.count(\"tf\")} Short={types.count(\"short\")}')
print(f'Threshold: {threshold}/10')
assert threshold >= 4, f'Need ≥4 threshold questions, got {threshold}'
print('Validation passed.')
"
```

Then open:
```bash
open "[OUTPUT_PATH]"
```

---

## Step 5 — Report

| Item | Value |
|------|-------|
| Quiz file | `Learning/Fathom/Quizzes/[filename].html` |
| Questions | 10 |
| MC / TF / Short | N / N / N |
| Threshold 🔑 | N/10 |
| AI grading | Available (set API key via ⚙ button) |
| Open | `open "[path]"` |

---

## Quality Rules

- All questions grounded in source material or Field Map — no free-floating trivia
- Short answer `model_answer` must be thorough enough to self-grade against (2–4 sentences)
- `grading_rubric` must list the specific key points (not just restate the question)
- `explanation` for MC/TF must state the correct answer AND briefly explain why — not just restate it
- At least one short answer question at synthesis level
- Threshold questions distributed across types (not all MC)

---

*fathom-quizmaker sub-skill v1.0 — 2026-03-27*
