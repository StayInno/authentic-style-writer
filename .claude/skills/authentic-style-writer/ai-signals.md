# AI Signal Taxonomy

Reference for the ANALYZE stage. Each signal type includes a definition, typical examples, and what makes it detectable.

---

## hedge-cluster

**Definition:** Two or more hedges or qualifiers stacked in a short span, often used by LLMs to sound cautious without saying anything concrete.

**Examples:**
- "It's worth noting that it may potentially be important to consider..."
- "While there are many factors to consider, it's generally advisable to..."
- "In many cases, this approach can often be seen as..."

**Detection cues:** Watch for `worth noting`, `it is important`, `keep in mind`, `it may`, `it can`, `generally`, `often`, `typically` appearing within the same sentence or adjacent sentences.

---

## over-explanation

**Definition:** Narrating what you are about to do instead of doing it. Restating what the reader already knows. Explaining conclusions before stating them.

**Examples:**
- "To answer your question, I will first explain X, then discuss Y, and finally summarize Z."
- "As we can see from the above, it is clear that..."
- "This section will cover the main points of the topic."

**Detection cues:** Phrases like "to address this", "let me explain", "as I mentioned", "as we discussed", "as you can see", "in summary", opening sentences that describe the paragraph rather than advance it.

---

## even-pacing

**Definition:** Every sentence in a paragraph is roughly the same length, producing a metronomic, monotonous rhythm. Human writers naturally vary sentence length for emphasis and flow. This is the "burstiness" axis.

**Quantitative thresholds** (sentence-length stdDev in words across the passage):
- ≥ 5 — human-typical bursty writing
- 1.5–5 — moderate; flag only when other signals cluster
- < 1.5 — machine-paced; almost always AI
- 0.2–0.4 — GPT-4 default cluster (Tang et al., 2024 and follow-up detector work)

**Detection cues:** Compute the stdDev of sentence length (in words) across any block of 4+ sentences. Report the measured number explicitly in Stage 2a — do not flag on impression alone. A signal table row for `even-pacing` must cite the stdDev value.

---

## abstract-filler

**Definition:** Vague noun phrases or buzzword constructions that gesture at meaning without delivering it.

**Examples:**
- "leverage synergies across the ecosystem"
- "a holistic approach to the challenges"
- "drive meaningful outcomes for stakeholders"
- "foster innovation and collaboration"
- "a comprehensive framework for success"

**Detection cues:** Nominalizations (`optimization`, `utilization`, `implementation`) combined with generic verbs (`ensure`, `provide`, `enable`, `facilitate`, `leverage`). Phrases that could be removed without changing what is communicated.

---

## assistant-opener

**Definition:** Opening a response or sentence with affirmative filler that acknowledges the request instead of responding to it.

**Examples:**
- "Certainly! I'd be happy to help with that."
- "Great question! Let me explain..."
- "Of course! Here's what you need to know:"
- "Absolutely, that's a great point."

**Detection cues:** `Certainly`, `Of course`, `Absolutely`, `Great`, `Sure`, `Happy to`, `I'd be glad`, `I'd be happy`, at sentence or paragraph start. Also: second-person openers that mirror the user's request back at them.

---

## comprehensiveness-creep

**Definition:** Covering every possible angle, caveat, and exception even when the request was specific. LLMs default to complete coverage; humans answer the question asked.

**Examples:**
- A question about one tool answered with a comparison of five tools
- A paragraph that lists "on one hand... on the other hand... however... that said... ultimately..." without landing anywhere
- Adding "of course, it depends on your specific situation" to every recommendation

**Detection cues:** Excessive `however`, `that said`, `on the other hand`, `it depends`, `in some cases`. Length that clearly exceeds what the question required. Balanced structures where one clear position would do.

---

## passive-excess

**Definition:** Overuse of passive voice that hides the actor and weakens directness. A few passives are fine; a paragraph dominated by them reads as evasive.

**Examples:**
- "The decision was made to implement the changes that had been previously discussed."
- "It is believed that the results can be improved by utilizing the recommended approach."

**Detection cues:** `was/were [verb]ed`, `is/are [verb]ed`, `has been [verb]ed` in dense sequence. More than 40% passive constructions in a paragraph is a flag.

---

## transition-cliche

**Definition:** Formulaic connective phrases that signal structure instead of creating it organically.

**Examples:**
- "Furthermore, it is important to consider..."
- "Moreover, this approach offers..."
- "In addition to the above..."
- "It is worth noting that..."
- "In conclusion, we have seen that..."
- "To summarize the key points..."

**Detection cues:** `Furthermore`, `Moreover`, `Additionally`, `In addition`, `It is worth noting`, `As previously mentioned`, `In conclusion`, `To summarize`, `Last but not least` at sentence start.

---

## symmetry-forced

**Definition:** Artificially parallel lists or structures imposed on content that would flow better as prose. LLMs default to bullet points and numbered lists even when the content is continuous reasoning.

**Examples:**
- Breaking a 3-sentence observation into a bulleted list of 3 items
- "There are three key factors: (1) X, (2) Y, and (3) Z." when the sentence could just say it
- Headers on every paragraph in a short response

**Detection cues:** Lists where prose would work. Three-part structures that feel imposed. Numbered items that are not actually steps or enumerable things.

---

## enthusiasm-flatten

**Definition:** Uniform positivity or constant endorsement across the entire text. No skepticism, no rough edges, no genuine personality. Everything is "great", "important", "valuable", "exciting".

**Examples:**
- "This is a fantastic approach that offers numerous benefits and exciting possibilities."
- "It's a great opportunity to leverage these powerful tools to achieve meaningful results."

**Detection cues:** `great`, `fantastic`, `exciting`, `powerful`, `amazing`, `valuable`, `important`, `meaningful`, `significant` used more than once or twice per 200 words. Absence of any critical or skeptical voice when the topic warrants one.

---

## Score calibration reference

| Score | Description |
|-------|-------------|
| 0–1 | No detectable AI signals. Reads naturally. |
| 2–3 | One or two minor signals; overall voice is human. |
| 4–5 | Several signals visible; text feels smooth but generic. |
| 6–7 | Multiple signal types present; clearly templated in places. |
| 8–9 | Dominated by AI patterns; little individual voice. |
| 10 | Nearly every sentence triggers one or more signals. |

A text can score high on individual signals while still reading naturally if the signals are sparse or not clustered. Weight clusters more heavily than isolated occurrences.
