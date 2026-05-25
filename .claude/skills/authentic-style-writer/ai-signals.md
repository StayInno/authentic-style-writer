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

## low-lexical-diversity

**Definition:** Звужений вокабуляр з повторами тих самих слів і коренів. LLM схильні переюзовувати лексику в межах одного відрізка тексту. Це другий за силою кількісний сигнал AI-тексту (після burstiness).

**Quantitative measures** (compute on **content words only** — strip stopwords like `the/a/of/is/are` first, інакше короткі тексти штучно занижуються):

- **Type-Token Ratio (TTR)** = `unique_content_words / total_content_words`. Lower = less diverse.
- **Simpson's Diversity Index (D)** = `Σ (n_i / N)²` де `n_i` — частота content-word `i`, `N` — total content words. Інтерпретація: ймовірність, що два випадково обрані слова однакові. **Higher = less diverse** (more repetition).

**Thresholds:**

- **Format B (corpus provided):** порівнювати TTR і Simpson's D рерайту з відповідними метриками корпусу автора (записаними в Style Card як `corpus_ttr`, `corpus_simpsons_d`). Рерайт має лягти **в ±15% відносно corpus TTR** і **в ±20% відносно corpus Simpson's D**. Вихід за межі — сигнал у Stage 2 і причина повернутися на Stage 3.
- **Format A (no corpus):** обчислити і повідомити TTR + Simpson's D у Stage 2a поруч із burstiness. **Не виставляти жорсткого порогу** — TTR сильно залежить від довжини тексту, тому без baseline це орієнтир, а не вирок. Прапор підіймати тільки якщо повтори видно очима (≥ 3 повтори того ж content-word на короткому відрізку, або кластер абстрактних іменників на кшталт `approach/framework/process`).

**Detection cues:** ті самі content-words повторюються в сусідніх реченнях; вокабуляр зосереджений на загальних абстрактних іменниках (`approach`, `framework`, `process`, `solution`, `system`); відсутні синоніми попри явні приводи їх вжити.

**Citation:** Petryshak T.V., Rybchak Z.L. 2025. *Stylometric Classification of AI-Generated Texts: Comparative Evaluation of Machine Learning Models.* Таврійський науковий вісник №2, pp. 135–147. DOI: [10.32782/tnv-tech.2025.2.15](https://doi.org/10.32782/tnv-tech.2025.2.15). На 30 000-семпловому датасеті (Human / ChatGPT / Deepseek) Simpson's D показано як **топ-1 ознаку** (feature importance 0.309 у Gradient Boosting, 0.184 у Random Forest), TTR — топ-2 (0.191 / 0.142). Binary Human-vs-AI Random Forest досягає macro F1 0.86 саме на цьому стилометричному наборі.

---

## redundancy

**Definition:** Повторне формулювання тієї самої інформації різними словами в межах одного абзацу. Це не повтор слів (це `low-lexical-diversity`), а повтор **пропозицій / змісту** — кожне наступне речення дає нову форму, але не нову інформацію.

**Examples:**
- "The team finished the project on time. They completed the work as scheduled. The deliverables were submitted before the deadline." (три формулювання одного факту)
- "This approach is effective. It produces good results. It works well in practice." (три майже-синонімічні речення)

**Detection cues:** Сусідні речення, у яких друге/третє не несе нової претензії, лише перефразовує. Тест: якщо речення можна видалити без втрати інформації — це redundancy. Прапор піднімається, коли ≥ 30% речень у абзаці видаляються без втрати змісту. Відрізняється від `low-lexical-diversity`: там повторюються слова, тут повторюються **твердження**, навіть якщо лексика різна.

**Citation:** Mitrović, S., Andreoletti, D., Ayoub, O. 2023. *ChatGPT or Human? Detect and Explain.* arXiv:2301.13852. SHAP-аналіз виділяє надлишковість серед топ-інтерпретованих ознак, що відрізняють короткий ChatGPT-текст від людського.

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

## pronoun-deficit

**Definition:** Перевикористання власних назв і конкретних іменників замість анафоричних займенників (`він/вона/воно/it/they/this/that`). LLM схильні повторювати назву суб'єкта в кожному реченні замість підхопити її займенником — це дає неприродну іменну щільність і відчуття «тексту без короткої памʼяті».

**Examples:**
- "Marie went to the store. Marie bought apples. Marie returned home." (замість "Marie went to the store, bought apples, and returned home.")
- "The framework provides benefits. The framework is widely used. The framework supports..." (замість чергування з `it` / `this approach`)

**Detection cues:** Назва суб'єкта (власна назва або повторювана іменникова група) зустрічається 3+ рази в межах 4 сусідніх речень, тоді як одного анафоричного займенника було б достатньо. Співвідношення `proper-noun + повторений-іменник : анафоричний-займенник` > 4:1 на абзац — машинний підпис.

**Citation:** Mitrović, Andreoletti, Ayoub 2023 (arXiv:2301.13852) — SHAP-аналіз показує дисбаланс власних назв і займенників серед топ-інтерпретованих ознак ChatGPT-тексту в коротких відгуках.

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

## syntactic-repetition

**Definition:** Повтор тієї самої синтаксичної конструкції в кількох послідовних реченнях без риторичної потреби (на відміну від навмисного паралелізму). Відрізняється від `symmetry-forced` тим, що тут немає вимушеного списку — повторюється сам **порядок слів і скелет речення** в проз-блоці.

**Examples:**
- "X provides Y. A enables B. P facilitates Q." (три SVO-клаузи з ідентичною generic-verb структурою)
- "By doing X, you can achieve Y. By using Z, you can ensure W. By applying V, you can deliver U." (три однакові `By -ing, you can [verb]` клаузи)
- "It is important to note that... It is worth mentioning that... It is essential to understand that..." (повтор скелета `It is X to Y that...`)

**Detection cues:** ≥ 3 послідовних речення з тим самим opening-pattern (`By X, ...`, `It is X that...`, `[Noun] is a [Adjective] [Noun] that...`) або з однаковим SVO-скелетом, у якому міняються лише іменники. Людський автор варіює синтаксичні рамки; LLM їх переюзовує.

**Citation:** Mitrović et al. 2023 (як вище). Додатково: Kumarage, T., Garland, J., Bhattacharjee, A. et al. 2023. *Stylometric Detection of AI-Generated Text in Twitter Timelines.* arXiv:2303.03697 — phraseology-фічі включають повтор синтаксичних n-gram-патернів як один із найсильніших дискримінаторів.

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
