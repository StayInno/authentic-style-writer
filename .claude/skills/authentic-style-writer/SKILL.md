---
name: authentic-style-writer
description: Analyze text for AI-like writing signals, score them, then rewrite the text in an authentic human voice. Use when the user wants to reduce generic assistant-like patterns, make AI-generated text sound natural, or get a writing quality diagnosis. Trigger phrases: "authentic-style-writer", "make this less AI", "rewrite in my style", "check if this sounds AI-written", "reduce AI patterns".
argument-hint: [draft text] or ===MY WRITING=== ... ===DRAFT=== ...
---

# Authentic Style Writer

You are running a 4-stage writing pipeline. Work through each stage in order. Do not skip stages.

## Input parsing

Check `$ARGUMENTS` for one of two formats:

**Format A — draft only:**
The entire argument is the text to process. No style card will be built; use general human writing norms as the target.

**Format B — corpus + draft:**
The argument contains both author samples and a draft, separated by markers:
```
===MY WRITING===
[one or more author text samples]
===DRAFT===
[the text to analyze and rewrite]
```
When Format B is detected, run LEARN_STYLE before ANALYZE. If `$ARGUMENTS` is empty, ask the user to paste the text they want processed.

---

## Stage 1 — LEARN_STYLE (only when author corpus is provided)

Extract a compact style card from the author samples. Output it as a JSON block under the heading `## Style Card`.

```json
{
  "stable_traits": {
    "lexical": "",
    "sentence_length": "",
    "rhythm": "",
    "syntax_patterns": "",
    "paragraph_density": "",
    "punctuation_habits": "",
    "transitions": "",
    "rhetorical_moves": "",
    "openings_closings": "",
    "specificity_level": ""
  },
  "weak_traits": [],
  "avoid_patterns": [],
  "limitations": ""
}
```

**Rules:**
- `stable_traits`: patterns that repeat across multiple samples
- `weak_traits`: things that appear once and may be topic or platform noise, not style
- `avoid_patterns`: clichés, corporate phrasing, or constructions the author never uses
- `limitations`: note if the corpus is too short or too uniform to draw reliable conclusions

---

## Stage 2 — ANALYZE

Diagnose the draft. Output under the heading `## Analysis`.

**2a. AI-likeness score**

Give a score from 0–10 where:
- 0–2: reads naturally, minimal generic patterns
- 3–4: a few smooth or templated moments, mostly fine
- 5–6: noticeably assistant-like in several places
- 7–8: heavily templated, generic structure, little personality
- 9–10: almost entirely AI-typical writing

State it as: `AI-likeness score: X/10`

Follow with a one-sentence verdict.

**2b. Signal table**

List every AI-like signal found. See [ai-signals.md](ai-signals.md) for the full signal taxonomy. Format as a markdown table:

| # | Signal type | Excerpt | Why it's a problem |
|---|-------------|---------|-------------------|

Signal types to detect:
- `hedge-cluster` — stacked hedges and qualifiers ("it's worth noting that", "it's important to consider")
- `over-explanation` — restating the obvious, narrating what you're about to do
- `even-pacing` — every sentence near the same length, no rhythm variation
- `abstract-filler` — vague noun phrases that say nothing ("leverage synergies", "holistic approach")
- `assistant-opener` — starts with "Certainly!", "Great question!", "Of course", "Absolutely"
- `comprehensiveness-creep` — covering every angle even when not asked
- `passive-excess` — excessive passive voice that buries agency
- `transition-cliche` — "Furthermore", "Moreover", "It is worth noting", "In conclusion"
- `symmetry-forced` — artificial parallel lists when prose would be more natural
- `enthusiasm-flatten` — uniform positivity, no edge, no skepticism

**2c. Style mismatch** (only if style card was built)

List specific places where the draft diverges from the author's stable traits.

---

## Stage 3 — REWRITE

Rewrite the draft under the heading `## Rewritten Version`.

Apply edits in this priority order:
1. **Preserve meaning** — never change what is being said, only how it is said
2. **Move toward author style** — if a style card exists, apply stable traits directly
3. **Move away from generic assistant style** — remove every signal flagged in the table

**Editing rules:**
- Prefer targeted edits over full regeneration
- Change sentence openings, rhythm, pacing, and transitions first
- Replace abstract noun phrases with concrete or specific language
- Break or vary uniform sentence lengths
- Remove qualifiers and hedges that add no information
- Cut over-explanation; trust the reader
- If no style card: aim for direct, specific, uneven-rhythmed prose with personality

Do not add new content or change the structure unless the structure itself is a signal.

---

## Stage 4 — EVALUATE

Output under the heading `## Evaluation`.

Score three dimensions separately on a 1–10 scale with a one-sentence explanation each:

| Dimension | Score | Note |
|-----------|-------|------|
| Genericity reduction | /10 | How much less assistant-like does it feel? |
| Meaning preservation | /10 | Is everything from the original still present and accurate? |
| Style match | /10 | How closely does it match the author's stable traits? (N/A if no corpus) |

Then list **what changed** and **what stayed constant**:

**Changed (style):**
- [bullet list of the main stylistic moves made]

**Preserved (meaning):**
- [bullet list confirming core content was kept intact]

If meaning preservation is below 8, flag it explicitly: `⚠ Meaning risk: [describe what may have shifted]`

---

## Output order

Always output in this order:
1. Style Card (if corpus provided)
2. Analysis
3. Rewritten Version
4. Evaluation

Keep each section under its heading. Do not mix them.
