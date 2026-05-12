# authentic-style-writer

A Claude Code skill that analyzes text for AI-like writing patterns, scores them, and rewrites the text in an authentic human voice.

## What it does

- Detects 10 types of AI-typical signals (hedge clusters, assistant openers, abstract filler, forced symmetry, etc.)
- Scores AI-likeness on a 0–10 scale
- Rewrites toward authentic, specific, human-sounding prose
- Optionally learns your personal style from writing samples
- Evaluates rewrite quality across three separate dimensions

## Install

**Option 1 — clone the repo into your project:**

```bash
git clone https://github.com/StayInno/authentic-style-writer.git
cp -r authentic-style-writer/.claude/skills/authentic-style-writer .claude/skills/
```

**Option 2 — copy the skill folder directly:**

Copy `.claude/skills/authentic-style-writer/` into your project's `.claude/skills/` directory. Claude Code picks it up on the next session start.

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
