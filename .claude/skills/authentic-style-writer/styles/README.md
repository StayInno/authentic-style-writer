# Saved styles

Stage 1.6 (STYLE_PERSIST) writes one file per saved style here:

```
<slug>.md
```

`INDEX.md` is an auto-maintained list of all saved slugs with a one-line description each.

## File format

Every `<slug>.md` is a markdown file with YAML frontmatter:

```markdown
---
name: <slug>
created: <YYYY-MM-DD>
samples_count: <N>
corpus_word_count: <W>
burstiness_stdev: <X.X>
corpus_ttr: <X.XX>
corpus_simpsons_d: <X.XX>
limitations: "<one-line summary; empty string if none>"
---

## Style Card

<Stage 1 output verbatim — evidence table, Lexical inventory, Style Card JSON>

## Style Brief

<Stage 1.5 output verbatim — APPLY / IMITATE / CARRY FORWARD / TARGET STYLOMETRY / DO NOT INTRODUCE / DO NOT USE>
```

## Reusing a saved style

```
/authentic-style-writer
===USE STYLE=== <slug>
===DRAFT===
[text to rewrite]
```

The skill loads the file, re-emits the Style Card and Style Brief under loaded-headings, and jumps directly to ANALYZE — LEARN_STYLE / STYLE_BRIEF / STYLE_PERSIST are skipped because the corpus has already been distilled.

## Portability between Claude Code and claude.ai

These files are plain markdown. To move a style from Claude Code (this directory) to claude.ai:

1. Open `<slug>.md` and copy its entire contents (including frontmatter).
2. In claude.ai, paste it back at runtime as Format D:
   ```
   ===STYLE CARD===
   <paste contents>
   ===DRAFT===
   [text]
   ```
   Or, for persistent reuse in claude.ai, add the same contents to a Project's Instructions.

## Overwriting

Saving with a slug that already exists overwrites the file (after a one-line `Overwriting existing style ...` notice). The skill does not auto-merge corpora — to combine new samples with an old style, re-run Format B with the union of samples and re-save under the same slug.
