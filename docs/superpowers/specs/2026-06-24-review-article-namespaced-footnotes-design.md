# review-article: namespaced footnote labels (drop manual renumbering)

**Date:** 2026-06-24
**Status:** Approved design, pending implementation plan
**Builds on:** `2026-06-24-review-article-file-handoffs-design.md` (same branch, file handoffs already shipped)

## Problem

After the file-handoff flip, the merge step still has the controller **manually renumber** every footnote into a single `[^r1], [^r2], …` sequence in document order. This is the one genuinely LLM-error-prone step in the workflow: off-by-one, duplicate label, dropped marker. The Step 5 self-review (orphan check) exists as a net for exactly this error class — catching mistakes after the fact rather than preventing them.

A second, related defect surfaced in end-to-end testing: the three reviewers emit footnotes in **three different ad-hoc styles** (`[1]`, `[^1]`, `**[1]**`). Their numbering namespaces collide (prose `[^1]` and grounding `[^1]` are different footnotes), which is precisely why the controller must renumber.

## Goal

Remove manual renumbering and cross-reviewer collisions by construction, staying in Markdown (source and reviewed output are both `.md`).

## Non-goals

- No Typst / `.typ` output. It would force a format change away from the Markdown the author writes, plus footnote-content escaping fragility, for a benefit the namespaced-label approach gets without changing format.
- No change to file handoffs, parallel dispatch, model selection, or the `<filename>-reviewed.md` output target.

## Design

### Namespaced labels per reviewer

Each reviewer labels its footnotes with its own prefix, so labels are unique across reviewers by construction:

- prose → `[^p1]`, `[^p2]`, …
- argument → `[^a1]`, `[^a2]`, …
- grounding → `[^g1]`, `[^g2]`, …

### Standardized reviewer output format

Each reviewer writes its file as a list of footnotes in this exact shape, so the controller can place each one mechanically:

```markdown
## Footnotes

### [^p1]
**Anchor:** "verbatim quoted sentence from the source that the marker attaches to"
**Note:** <issue + explanation + suggested rewrite/flag>

### [^p2]
**Anchor:** "..."
**Note:** ...
```

The `Anchor` is copied verbatim from the source so the controller can locate the insertion point. The label uses the reviewer's namespace.

### Merge: single ordered pass, no renumbering

The controller reads the three files plus `<filename>.md` once, then walks the source in document order. At each `Anchor`, it appends that footnote's label-marker after the anchor sentence in the body **and** appends `<label>: <Note>` to a running definition list. Because both the markers and the definitions are emitted in document order, the result renders as sequential 1, 2, 3… under any Markdown renderer (Pandoc and GitHub/CommonMark number footnotes by order of appearance and treat the label text as an internal anchor only). The controller never assigns or reassigns numbers.

The validity check and the re-review loop for malformed footnotes are unchanged.

### Self-review

Keep the orphan check, retargeted to namespaced labels: every `[^pN]`/`[^aN]`/`[^gN]` marker in the body has a matching definition, and every definition has a matching marker. Collisions and renumber errors are now impossible by construction; the check remains a cheap net for a dropped marker.

## Files touched

- `skills/review-article/prose-reviewer-prompt.md` — namespaced label (`[^pN]`) + standardized Anchor/Note output format.
- `skills/review-article/argument-reviewer-prompt.md` — namespaced label (`[^aN]`) + same format.
- `skills/review-article/grounding-reviewer-prompt.md` — namespaced label (`[^gN]`) + same format.
- `skills/review-article/SKILL.md` — merge diagram (drop the "Renumber sequentially" node), merge prose, and Step 5 self-review wording.

## Verification

Re-run `review-article` on a flawed sample `.md`:

- Each reviewer's file uses only its own namespace (`[^pN]` / `[^aN]` / `[^gN]`) in the standardized Anchor/Note format.
- `<filename>-reviewed.md` contains the source with namespaced markers placed after their anchor sentences and a matching definition for each — no orphans, no duplicate labels.
- The controller performed no renumbering (no `[^rN]` relabeling step).
