# authentic-style-writer

A Claude Code skill that analyzes text for AI-like writing patterns, scores them, and rewrites the text in an authentic human voice.

## What it does

- Detects 16 types of AI-typical signals (hedge clusters, assistant openers, abstract filler, forced symmetry, redundancy, pronoun-deficit, syntactic-repetition, list-cram, register-mix, etc.)
- Scores AI-likeness on a 0–10 scale
- Rewrites toward authentic, specific, human-sounding prose
- Optionally learns your personal style from writing samples
- Optionally flags possible mixed authorship within a single draft (Stage 1.7 Boundary Detection, for drafts ≥ 400 words)
- Evaluates rewrite quality across four dimensions (genericity reduction, meaning preservation, style match, stylometry match) with a detection-resistance reference anchored to the StyloAI / AuTexTification baseline

## Scientific grounding

The skill is built on published stylometric research:

- **Sadasivan et al. 2023** — paraphrasing attacks drop detector accuracy from 97% to 57–80%, validating the entire humanization approach.
- **Mindner & Schaaff 2024** — detector F1 falls from > 96% on raw ChatGPT to 78% on paraphrased text.
- **Alshammari 2025** — humanization brings GPTZero, Copyleaks, QuillBot down to 52–71% on DeepSeek-generated text.
- **Opara 2024 (StyloAI)** — F1 = 0.81 benchmark on AuTexTification, used as detection-resistance anchor in Stage 4.
- **Petryshak & Rybchak 2025** — Simpson's D and TTR are top-2 features for Human/ChatGPT/DeepSeek separation, used as Stage 3 rewrite targets.
- **Mitrović, Andreoletti & Ayoub 2023** — SHAP-identified signals (`redundancy`, `pronoun-deficit`, `syntactic-repetition`) added to the taxonomy.
- **Kumarage et al. 2023** — change-point detection inspired the Stage 1.7 Boundary Detection.

For an external validator with ~120 stylometric vectors and Ukrainian-language support, see **StyloMetrix** (Okulska, Stetsenko et al. 2023).

## Install

Just ask Claude Code to install it — e.g.:

> Install the authentic-style-writer skill from github.com/StayInno/authentic-style-writer

It will fetch the skill and place it in your project's `.claude/skills/` directory for you.

Prefer to do it by hand? Copy `.claude/skills/authentic-style-writer/` into your project's `.claude/skills/` directory. Claude Code picks it up on the next session start.

## Usage

```
/authentic-style-writer [your draft text]
```

With your own writing samples (for style-matched rewriting):

```
/authentic-style-writer
===MY WRITING===
[3–10 samples of your own writing, 600+ words total]
===DRAFT===
[text to rewrite]
```

See [`.claude/skills/authentic-style-writer/README.md`](.claude/skills/authentic-style-writer/README.md) for the full usage guide.

## Files

```
.claude/skills/authentic-style-writer/
├── SKILL.md          # Pipeline instructions for Claude
├── ai-signals.md     # Signal taxonomy with examples
└── README.md         # Usage guide
```
