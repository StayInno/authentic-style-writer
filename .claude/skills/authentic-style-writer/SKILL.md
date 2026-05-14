---
name: authentic-style-writer
description: Analyze text for AI-like writing signals, score them, then rewrite the text in an authentic human voice. Use when the user wants to reduce generic assistant-like patterns, make AI-generated text sound natural, or get a writing quality diagnosis. Trigger phrases: "authentic-style-writer", "make this less AI", "rewrite in my style", "check if this sounds AI-written", "reduce AI patterns".
argument-hint: [draft text] or ===MY WRITING=== ... ===DRAFT=== ...
---

# Authentic Style Writer

You are running a writing pipeline with explicit data hand-offs between stages. Each stage consumes the previous stage's output as a **constraint**, not a reference. Do not skip any stage. Do not produce a stage that ignores its required inputs.

## Core principle — no hallucinated style

You may only use stylistic moves that are either:
- (a) present in the author's corpus (when corpus is provided), or
- (b) explicitly chosen to remove an AI signal flagged in Analysis.

You may **not** introduce a punctuation mark, sentence shape, opening, closing, or structural device just because it "sounds good." If a corpus exists and a device is not in it, the device is forbidden unless removing an AI signal demands it (and that demand is stated in the Replacement Map).

---

## Input parsing

Check `$ARGUMENTS` for one of two formats:

**Format A — draft only:**
The entire argument is the text to process. Skip LEARN_STYLE and STYLE_BRIEF. Target = general human writing norms.

**Format B — corpus + draft:**
```
===MY WRITING===
[one or more author samples]
===DRAFT===
[text to analyze and rewrite]
```

When Format B is detected, count the samples and run LEARN_STYLE → STYLE_BRIEF before ANALYZE.

If `$ARGUMENTS` is empty, ask the user to paste the text.

---

## Stage 1 — LEARN_STYLE  *(Format B only)*

Output under heading `## Style Card`. Build it from **evidence only**. Every trait must carry a count.

### Evidence rule

Let `N` = total number of author samples and `W` = total words across the corpus.

A trait is **stable** only when all three conditions hold:
1. Trait appears in ≥ ⌈N/2⌉ samples and ≥ 2 samples total.
2. `N ≥ 3`.
3. `W ≥ 2000` words.

Stylometry community baseline for reliable authorship signal is 2,000–5,000 words (Patel et al., STYLL; Eder 2015). Below that, individual style is not separable from topic noise.

If any condition fails, every trait goes into `occasional_traits`, and the failure is named in `limitations` (e.g. "N=2 samples, W=1,100 words — traits treated as occasional"). When `W < 2000`, the Stage 4 Style match score is capped at 7/10 and the cap is stated in the Evaluation.

### Required counts before writing the card

Before filling in the JSON, count and write a short evidence table:

```
Samples (N): ___
Total words (W): ___
Avg sentence length (words): ___
Sentence-length stdDev (burstiness): ___
Avg paragraph length (sentences): ___

Burstiness reference: human writing typically shows sentence-length stdDev ≥ 5 (Tang et al., 2024). Default LLM output clusters at 0.2–0.4. stdDev < 1.5 reads as machine-paced. This number is the target the rewrite must hit.

Punctuation per 1000 words across corpus:
- em-dash (—):
- en-dash (–):
- ellipsis (…):
- semicolon (;):
- colon (:):
- parentheses ( ):
- exclamation (!):
- question (?):

Structural patterns (count of samples where this appears, out of N):
- opens with name/product/person hook: x/N
- opens with question: x/N
- opens with numbered fact: x/N
- ends with call-to-action: x/N
- ends with question: x/N
- ends with personal sign-off: x/N
- uses bulleted lists: x/N
- uses subheadings: x/N
```

You must produce this table **before** the JSON. If you cannot count a category, write `n/a` — never guess.

### Lexical inventory

List up to 30 **actual phrases** from the corpus that the author uses repeatedly or that carry their voice: connectors, fillers, idioms, dialectisms, signature expressions. Quote them verbatim. These are the words you can **reuse** in the rewrite.

```
Lexical inventory (verbatim from corpus):
- connectors: [exact phrases]
- dialectisms / regional forms: [exact phrases]
- signature openers: [exact phrases]
- signature closers / CTAs: [exact phrases]
- repeated nouns / objects of attention: [exact phrases]
```

If a category is empty in the corpus, write `(none observed)`.

### Style Card JSON

```json
{
  "samples_count": 0,
  "corpus_word_count": 0,
  "burstiness_stdev": 0.0,
  "stable_traits": {
    "lexical": { "value": "", "evidence_count": 0 },
    "sentence_length": { "value": "", "evidence_count": 0 },
    "rhythm": { "value": "", "evidence_count": 0 },
    "syntax_patterns": { "value": "", "evidence_count": 0 },
    "paragraph_density": { "value": "", "evidence_count": 0 },
    "punctuation_habits": { "value": "", "evidence_count": 0 },
    "transitions": { "value": "", "evidence_count": 0 },
    "rhetorical_moves": { "value": "", "evidence_count": 0 },
    "openings": { "value": "", "evidence_count": 0 },
    "closings": { "value": "", "evidence_count": 0 },
    "specificity_level": { "value": "", "evidence_count": 0 }
  },
  "structural_invariants": [
    { "pattern": "", "evidence_count": 0 }
  ],
  "imitate_constructions": [
    { "template": "", "example_from_corpus": "", "evidence_count": 0 }
  ],
  "occasional_traits": [],
  "avoid_patterns": [],
  "limitations": ""
}
```

**Rules:**
- `evidence_count` is the number of samples (out of `samples_count`) that show the trait. Stable requires ≥ ⌈N/2⌉ AND `N ≥ 3` AND `W ≥ 2000`.
- `corpus_word_count` (W): total words across all samples. Below 2,000 every trait is demoted to `occasional_traits`.
- `burstiness_stdev`: standard deviation of sentence length (in words) across the corpus. Becomes the target pacing for the rewrite.
- `structural_invariants` are recurring shapes the author always uses: e.g. "ends with CTA", "headline names a person", "opens with a number". These must be carried into the rewrite if their count meets the stable threshold.
- `imitate_constructions` are concrete syntactic templates the author uses repeatedly (e.g. "short past-tense opener: «Купив X»", "two-clause negation: «не X, а Y»", "one-word emphatic close"). List 4–8 with a literal example from the corpus. These are what the rewrite must reach toward — removing AI signals is not the same as moving toward the target author's style (Patel et al., STYLL, 2024).
- `avoid_patterns` are constructions the author **never** uses but the draft might (corporate clichés, em-dash if the corpus has none, etc.).
- `limitations`: flag short or homogeneous corpora explicitly. Record `W < 2000` here when it occurs.

---

## Stage 1.5 — STYLE_BRIEF  *(Format B only)*

This is the **bridge from card to rewrite**. Convert the Style Card into 5–10 numbered editing rules and a forbidden list. The rewrite MUST consume this brief. Output under `## Style Brief`.

Format:

```
APPLY (each rule cites the trait it comes from):
1. [rule] — from stable_traits.X (count Y/N)
2. ...

IMITATE (syntactic templates to actively copy — at least ⌈len/2⌉ must appear in rewrite):
- [template]: «literal example from corpus» — count Y/N
- ...

CARRY FORWARD (structural invariants that must appear in rewrite):
- [invariant] — count Y/N

TARGET BURSTINESS:
- corpus sentence-length stdDev: ___ (rewrite must land within ±1.5)

DO NOT INTRODUCE (patterns not in corpus):
- em-dash (—) [if corpus count is 0 or near-0]
- semicolons [if corpus count is 0]
- bulleted lists [if 0/N samples use them]
- subheadings [if 0/N samples use them]
- [other devices absent from corpus]

DO NOT USE (avoid_patterns from card):
- [pattern]
```

Each APPLY rule must be **actionable** ("use connector «ну і», «короче» — 4/6 samples") not vague ("conversational tone"). Each IMITATE entry must quote a literal example from the corpus, so the rewrite has a concrete template to copy. Each DO NOT INTRODUCE entry must reference a count of 0 or near-0 in the corpus.

**Why IMITATE exists:** removing AI signals reliably "moves away" from machine style but does not reliably "move toward" the target author (Patel et al., STYLL, 2024). Without explicit syntactic templates the rewrite drifts to generic-human writing instead of *this* author. IMITATE forces the rewrite to copy actual sentence shapes from the corpus.

---

## Stage 2 — ANALYZE

Output under heading `## Analysis`.

### 2a. AI-likeness score

`AI-likeness score: X/10` (calibration in [ai-signals.md](ai-signals.md))
`Draft burstiness stdDev: X.X` (target: corpus stdDev in Format B, or ≥ 5 in Format A; < 1.5 is machine-paced)
One-sentence verdict.

### 2b. Signal table

| # | Signal type | Excerpt from draft | Why it's a problem |
|---|-------------|--------------------|--------------------|

Signal types: see [ai-signals.md](ai-signals.md) — `hedge-cluster`, `over-explanation`, `even-pacing`, `abstract-filler`, `assistant-opener`, `comprehensiveness-creep`, `passive-excess`, `transition-cliche`, `symmetry-forced`, `enthusiasm-flatten`.

### 2c. Style mismatch  *(Format B only)*

For each stable trait or structural invariant in the Style Card, check the draft. List divergences as:
- `trait → what the draft does instead → quote from draft`

---

## Stage 2.5 — REPLACEMENT_MAP

This is the **bridge from analysis to rewrite**. For every row in the signal table AND every row in the style mismatch list, produce a concrete replacement. Output under `## Replacement Map`.

| # | Source | Excerpt (from draft) | Replacement (concrete words) | Justification |
|---|--------|----------------------|------------------------------|---------------|

- `Source` = signal type or `mismatch: <trait>`.
- `Replacement` must be **actual proposed wording**, not a description. If a corpus exists, prefer phrases from the Lexical inventory.
- `Justification` cites either Style Brief rule # or the AI signal removed.

If a signal has no clean replacement (e.g. you'd need to delete the sentence), say `DELETE` and explain.

The rewrite is built from this map. If the map is empty, the rewrite changes nothing.

---

## Stage 3 — REWRITE

Output under heading `## Rewritten Version`.

Execution order:
1. Apply every row of the Replacement Map. Use the exact `Replacement` text wherever possible.
2. Inject IMITATE constructions from the Style Brief — at least ⌈len/2⌉ of the listed templates must appear in the rewrite. Use the corpus example as a *shape*, not a verbatim copy.
3. Enforce every `CARRY FORWARD` invariant from the Style Brief (e.g. if the brief says "ends with CTA, 6/6", the rewrite must end with a CTA).
4. Match TARGET BURSTINESS: the rewrite's sentence-length stdDev must land within ±1.5 of the corpus stdDev (or ≥ 5 in Format A). If too even, break up a long sentence with a short punchy one or merge two short ones into a long one.
5. Enforce every `DO NOT INTRODUCE` constraint. Re-read the rewrite once and remove any forbidden device that slipped in.
6. Preserve meaning. No new claims, no removed claims.

Hard rules:
- Do not add an em-dash, semicolon, ellipsis, bulleted list, or subheading if it is on the `DO NOT INTRODUCE` list.
- Do not "polish" by introducing connectors, hedges, or transitions not in the Lexical inventory.
- If you cannot apply a Replacement Map row, write a one-line note under the rewrite explaining why — do not silently skip it.

---

## Stage 3.5 — TRACE

This is the **bridge from rewrite to evaluation**. Output under `## Trace`.

### 3.5a. Brief application check  *(Format B only)*

| Style Brief rule | Applied? | Where in rewrite (quote) |
|------------------|----------|--------------------------|

Every numbered APPLY rule, every IMITATE template, and every CARRY FORWARD invariant must appear in this table. `Applied?` is `yes` / `no` / `partial`. If `no` or `partial`, quote what is there instead. IMITATE coverage below ⌈len/2⌉ means the rewrite is incomplete — return to Stage 3.

### 3.5b. Burstiness check

Measure the rewrite's sentence-length stdDev. Record:

| Metric | Target | Rewrite | Within ±1.5? |
|--------|--------|---------|--------------|
| Sentence-length stdDev | (corpus stdDev, or ≥ 5 in Format A) | ___ | yes / no |

If `no`, return to Stage 3 and adjust at least two sentences to widen or tighten the spread.

### 3.5c. Forbidden-pattern audit

For each item on `DO NOT INTRODUCE` and `DO NOT USE`: did it appear in the rewrite? Count occurrences in the new text.

| Forbidden item | Count in rewrite | Status |
|----------------|------------------|--------|

If any count > 0, the rewrite is **not done** — go back to Stage 3 and fix it before continuing.

### 3.5d. Replacement Map coverage

| Map row # | Applied as written? | If not, what was used |
|-----------|---------------------|-----------------------|

Coverage below 80% means the rewrite is incomplete — return to Stage 3.

---

## Stage 4 — EVALUATE

Output under heading `## Evaluation`. Scores must be derived from the Trace, not from feeling.

| Dimension | Score | Derivation |
|-----------|-------|------------|
| Genericity reduction | /10 | (signals removed in Replacement Map) ÷ (signals detected) × 10 |
| Meaning preservation | /10 | 10 minus 1 point per claim added, removed, or distorted |
| Style match | /10 | (APPLY rules + IMITATE templates + invariants applied) ÷ (total) × 10 — N/A if no corpus. Capped at 7 when `W < 2000`. |
| Burstiness match | /10 | 10 if rewrite stdDev within ±1.5 of target; subtract 2 per additional unit of drift. Floor 0. |

Then:

**Changed (style):**
- [bullet list of actual moves made — must be traceable to Replacement Map or Style Brief]

**Preserved (meaning):**
- [bullet list confirming each major claim from the original survived]

**Hallucinated style (must be empty):**
- [any device used in rewrite that was not justified by Replacement Map or Style Brief — if non-empty, this is a defect, not a feature]

If `Meaning preservation < 8`: `⚠ Meaning risk: [what shifted]`.
If `Style match < 7` (Format B): `⚠ Style miss: [which brief rules failed]` and offer to re-run Stage 3 with stricter adherence.

---

## Output order

1. Style Card  *(Format B only)*
2. Style Brief  *(Format B only)*
3. Analysis
4. Replacement Map
5. Rewritten Version
6. Trace
7. Evaluation

Keep each section under its heading. Do not collapse or reorder.
