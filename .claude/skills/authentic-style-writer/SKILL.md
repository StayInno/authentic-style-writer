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

Lexical diversity reference: human short-form prose typically shows TTR ≈ 0.5–0.7 and Simpson's D ≈ 0.02–0.08 (content-words only, stopwords removed). LLMs cluster at lower TTR and higher Simpson's D — see Petryshak & Rybchak (2025), where Simpson's D is the top-ranked feature for distinguishing AI from human writing. Corpus values below become the rewrite targets.

Lexical diversity (content-words only, stopwords removed):
- Type-Token Ratio (TTR): ___
- Simpson's Diversity Index (D): ___

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

Optional grammar-level metrics (compute if `W ≥ 2000`; otherwise mark `n/a`):
- stopword ratio (stopwords / total tokens): ___ (LLM English cluster: 0.35–0.45; human varies more widely 0.30–0.55. Source: Opara 2024, StyloAI, 31 stylometric features.)
- noun ratio (nouns / total content words): ___
- verb ratio: ___
- adjective ratio: ___
- pronoun-to-proper-noun ratio: ___ (see `pronoun-deficit` in [ai-signals.md](ai-signals.md); < 0.25 reads as machine-paced naming)

These four ratios are **optional reference points**. They are reported in the Style Card but do not become hard rewrite targets. They exist so that Stage 2 can flag a draft whose grammar distribution diverges sharply from the corpus (e.g. corpus noun-ratio = 0.30 vs draft noun-ratio = 0.55).
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
  "corpus_ttr": 0.0,
  "corpus_simpsons_d": 0.0,
  "optional_grammar_metrics": {
    "stopword_ratio": 0.0,
    "noun_ratio": 0.0,
    "verb_ratio": 0.0,
    "adjective_ratio": 0.0,
    "pronoun_to_proper_noun_ratio": 0.0
  },
  "stable_traits": {
    "lexical": { "value": "", "evidence_count": 0 },
    "lexical_diversity": { "value": "", "evidence_count": 0 },
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
- `corpus_ttr` and `corpus_simpsons_d`: Type-Token Ratio and Simpson's Diversity Index across the corpus, computed on **content words only** (stopwords removed). Both become rewrite targets in Stage 3 (±15% for TTR, ±20% for Simpson's D). See [ai-signals.md](ai-signals.md) `low-lexical-diversity` for formulas.
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

TARGET STYLOMETRY:
- corpus sentence-length stdDev: ___ (rewrite must land within ±1.5)
- corpus TTR: ___ (rewrite must land within ±15%)
- corpus Simpson's D: ___ (rewrite must land within ±20%)

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

## Stage 1.7 — BOUNDARY_DETECTION  *(optional, both formats)*

Skip this stage if the draft is **< 400 words** or has **fewer than 3 paragraphs**. Otherwise output under `## Boundary Detection`.

This stage detects whether the draft is internally homogeneous or shows signs of mixed authorship (e.g. user-written text spliced with LLM-generated paragraphs). Stylometric profiles vary within a single document when sources differ.

### Procedure

For each paragraph in the draft, compute:
- sentence-length stdDev
- TTR (content-words only, stopwords removed)
- a paragraph-local AI-likeness score 0–10 using the [ai-signals.md](ai-signals.md) taxonomy

Then compute the document-mean of stdDev and TTR.

### Output table

| Paragraph # | stdDev | TTR | AI score | Flag |
|------------:|-------:|----:|---------:|------|

Flag a paragraph when **any** holds:
- stdDev deviates by > 2 absolute units from doc mean
- TTR deviates by > 20% relative from doc mean
- paragraph AI score is ≥ 3 points higher than the lowest-scoring paragraph in the doc

### Interpretation

- **0–1 paragraphs flagged** → write `Profile: homogeneous. Treat the draft as a single block in Stage 2.`
- **2+ paragraphs flagged with notably higher AI scores** → write `Profile: possible mixed authorship — paragraphs N, M show distinct stylometric profile and may have a different source than the rest.`

This flag is **informational**. It does not split the rewrite or change Stage 3 execution — Stage 3 still produces one Final Rewrite for the whole draft. But Stage 2.5 should weight Replacement Map entries from flagged paragraphs more aggressively (they are the likeliest LLM-origin segments).

**Citation:** Kumarage, T., Garland, J., Bhattacharjee, A., Trapeznikov, K., Ruston, S., Liu, H. 2023. *Stylometric Detection of AI-Generated Text in Twitter Timelines.* arXiv:2303.03697. Introduces change-point detection for stylometric boundaries within a single document — same logic adapted here for paragraph-level granularity.

---

## Stage 2 — ANALYZE

Output under heading `## Analysis`.

### 2a. AI-likeness score

`AI-likeness score: X/10` (calibration in [ai-signals.md](ai-signals.md))
`Draft burstiness stdDev: X.X` (target: corpus stdDev in Format B, or ≥ 5 in Format A; < 1.5 is machine-paced)
`Draft TTR: X.XX | Draft Simpson's D: X.XX` (content-words only, stopwords removed. Format B: targets = `corpus_ttr` / `corpus_simpsons_d` from Style Card. Format A: report only, no hard threshold — see [ai-signals.md](ai-signals.md) `low-lexical-diversity`.)
One-sentence verdict.

### 2b. Signal table

| # | Signal type | Excerpt from draft | Why it's a problem |
|---|-------------|--------------------|--------------------|

Signal types: see [ai-signals.md](ai-signals.md) — `hedge-cluster`, `over-explanation`, `even-pacing`, `low-lexical-diversity`, `redundancy`, `abstract-filler`, `assistant-opener`, `comprehensiveness-creep`, `passive-excess`, `pronoun-deficit`, `transition-cliche`, `symmetry-forced`, `syntactic-repetition`, `enthusiasm-flatten`.

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
4. Match TARGET STYLOMETRY:
   - Sentence-length stdDev within ±1.5 of corpus stdDev (or ≥ 5 in Format A). If too even, break up a long sentence with a short punchy one or merge two short ones into a long one.
   - TTR within ±15% of `corpus_ttr` (Format B only).
   - Simpson's D within ±20% of `corpus_simpsons_d` (Format B only).
   If TTR too low or Simpson's D too high: replace repeated content-words with synonyms or restructure to remove the repetition. Prefer vocabulary from the Lexical inventory before reaching for outside synonyms.
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

### 3.5b. Stylometry check

Measure the rewrite's sentence-length stdDev, TTR, and Simpson's D (content-words only). Record:

| Metric | Target | Rewrite | Within tolerance? |
|--------|--------|---------|-------------------|
| Sentence-length stdDev | (corpus stdDev, or ≥ 5 in Format A) | ___ | yes / no (±1.5) |
| TTR | (`corpus_ttr`, Format B only) | ___ | yes / no (±15%) |
| Simpson's D | (`corpus_simpsons_d`, Format B only) | ___ | yes / no (±20%) |

If any row is `no`: return to Stage 3 and fix the offending dimension before continuing. For stdDev, vary sentence length. For TTR / Simpson's D, replace repeated content-words with synonyms (prefer Lexical inventory).

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

## Stage 3.7 — REFINE_LOOP

Take the rewrite that passed Stage 3.5 (call it `rewrite_v1`) and refine it through **up to 3 additional iterations**. Goal: catch AI signals that survived the first pass — or new ones the rewrite itself introduced. Output under `## Refinement Log`.

### Iteration procedure

For each iteration `N` ∈ {v2, v3, v4} (v1 already exists):

1. **Re-analyze `rewrite_v(N-1)`** with the same instruments as Stage 2a:
   - `AI-likeness score: X/10` (calibration in [ai-signals.md](ai-signals.md))
   - `stdDev | TTR | Simpson's D` (2 decimals; TTR / Simpson's D on content-words only)
   - Compact signal table: only rows for signal types from [ai-signals.md](ai-signals.md) that still appear. Empty = clean.

2. **Early-exit conditions** — stop the loop if any one holds:
   - (a) `AI-likeness score ≤ 2/10` AND signal table is empty.
   - (b) The new signal set is identical to the previous iteration's — refinement has plateaued and another pass will not help.
   - (c) All three stylometry metrics are within tolerance AND no signal type appears more than once.

3. **Mini Replacement Map** — for each residual signal produce a concrete replacement (same format as Stage 2.5). Address only signals that newly appeared or survived from the previous pass. If a corpus exists, prefer phrases from the Lexical inventory.

4. **Mini Rewrite** — apply the mini Replacement Map to `rewrite_v(N-1)` to produce `rewrite_v(N)`. All Stage 3 hard rules still apply (`DO NOT INTRODUCE`, no hallucinated style, `CARRY FORWARD` invariants preserved). Run a compact stylometry check (3.5b only — full Trace already passed for v1).

   **Surgical edit rule (hard):** change only the exact spans identified in the Mini Replacement Map. Every sentence not touched by the map must be copied verbatim from `rewrite_v(N-1)` — character for character. Do not rephrase, smooth, or "improve" surrounding sentences. Do not rewrite a paragraph to make an edit flow better. If an edit cannot be applied without touching adjacent text, note it as `skipped — would require surrounding rewrite` and leave the original. Introducing a new device (em-dash, semicolon, subheading, list) in an untouched sentence is a defect equal to introducing it in Stage 3.

### Refinement Log table

Record every iteration that actually ran. For iterations that did not run (early-exit fired), write a single row stating the exit condition.

| Iteration | AI score | stdDev | TTR | Simpson's D | Residual signals | Action taken |
|-----------|---------:|-------:|----:|------------:|------------------|--------------|
| v1 (Stage 3 output) | _ | _ | _ | _ | _ | — |
| v2 | _ | _ | _ | _ | _ | _ |
| v3 | _ | _ | _ | _ | _ | _ |
| v4 | _ | _ | _ | _ | _ | _ |

If exit fires after v2: drop the v3/v4 rows and add a one-liner `(exited after v2: <condition a/b/c>)`.

### Final pick

Choose the version with the **lowest AI-likeness score**. Tie-breaker: prefer the later version (carries more refinement). Output the chosen text under `## Final Rewrite` with a one-line note: `Picked vN (score X/10).` Stage 4 evaluates the **Final Rewrite**, not v1.

Hard cap: never produce more than v4. If after v4 the score has not improved over v1, output v1 as the Final Rewrite and note `Refinement did not converge — kept v1.`

---

## Stage 4 — EVALUATE

Output under heading `## Evaluation`. **Evaluate the Final Rewrite picked in Stage 3.7, not the v1 rewrite.** Scores must be derived from the Trace and Refinement Log, not from feeling.

| Dimension | Score | Derivation |
|-----------|-------|------------|
| Genericity reduction | /10 | (signals removed in Replacement Map) ÷ (signals detected) × 10 |
| Meaning preservation | /10 | 10 minus 1 point per claim added, removed, or distorted |
| Style match | /10 | (APPLY rules + IMITATE templates + invariants applied) ÷ (total) × 10 — N/A if no corpus. Capped at 7 when `W < 2000`. |
| Stylometry match | /10 | 10 if all three metrics (stdDev, TTR, Simpson's D) within tolerance. −2 per metric out of tolerance. −1 per additional unit of drift beyond. Floor 0. In Format A only stdDev is scored — TTR and Simpson's D are reported but not penalized. |

Then:

**Changed (style):**
- [bullet list of actual moves made — must be traceable to Replacement Map or Style Brief]

**Preserved (meaning):**
- [bullet list confirming each major claim from the original survived]

**Hallucinated style (must be empty):**
- [any device used in rewrite that was not justified by Replacement Map or Style Brief — if non-empty, this is a defect, not a feature]

If `Meaning preservation < 8`: `⚠ Meaning risk: [what shifted]`.
If `Style match < 7` (Format B): `⚠ Style miss: [which brief rules failed]` and offer to re-run Stage 3 with stricter adherence.

### Detection-resistance reference

These mappings are heuristic, not measured against a specific detector. They anchor scale, not predict outcome:

| Final AI-likeness score | Expected behavior under feature-based detectors |
|------------------------:|--------------------------------------------------|
| 0–2 / 10 | Likely **below** F1 0.81 — the StyloAI baseline on AuTexTification (Opara 2024). Rewrite would read as human to most stylometric classifiers. |
| 3–5 / 10 | Boundary zone. Some signals still trigger; detectors trained on humanization-attacked corpora (Alshammari 2025, Mindner & Schaaff 2024) may still flag. |
| 6+ / 10 | Likely classified as AI by feature-based detectors. Return to Stage 3 or run another Refinement Loop. |

**Citations:**
- Opara, C. 2024. *StyloAI: Distinguishing AI-Generated Content with Stylometric Analysis.* AIED 2024, Springer CCIS 2151. arXiv:2405.10129. F1 = 0.81 on the AuTexTification benchmark (160k+ texts, EN/ES, 5 domains, 6 generator models).
- Sarvazyan, A. M. et al. 2023. *Overview of AuTexTification at IberLEF 2023.* *Procesamiento del Lenguaje Natural* 71, 275–288. The reference dataset against which stylometric detectors are scored.

---

## Output order

1. Style Card  *(Format B only)*
2. Style Brief  *(Format B only)*
3. Boundary Detection  *(both formats, only when draft ≥ 400 words and ≥ 3 paragraphs)*
4. Analysis
5. Replacement Map
6. Rewritten Version  *(v1 — output of Stage 3)*
7. Trace
8. Refinement Log  *(iterations v2–v4 with early-exit notes)*
9. Final Rewrite  *(the picked version; this is what Stage 4 evaluates)*
10. Evaluation

Keep each section under its heading. Do not collapse or reorder.
