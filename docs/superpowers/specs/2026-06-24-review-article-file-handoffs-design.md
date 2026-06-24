# review-article: flip to file handoffs

**Date:** 2026-06-24
**Status:** Approved design, pending implementation plan

## Problem

`review-article` was built by adopting the pre-6.0 subagent-driven-development (SDD) "best practice" of pasting source text inline into subagent prompts. SDD reversed this in v6.0: the central efficiency lesson is that **everything pasted into a dispatch prompt stays resident in the controller's context and is re-read on every later turn**, so SDD now hands artifacts over as files.

`review-article` still does the opposite, and inconsistently:

- The Prompt Templates section instructs the controller to read `<filename>.md` into context and substitute the literal text inline. With three reviewers, the article is held in controller context **3×** and stays resident for the whole session.
- Reviewers return full footnote annotations **into chat**, so the controller also holds three complete annotation sets in context to merge them.
- Yet the merge step's diagram already assumes file outputs exist ("collect footnotes from all four intermediate files", "delete the four intermediate files") — an internal contradiction. It also says **four** files for **three** reviewers (a counting bug).
- The prose prompt has typos (`Porpose`, "addressing prose issues in the to Strunk's") and, critically, **never supplies the text to review** — it tells the subagent to "read the actual text they wrote" but no text block is included. The argument and grounding prompts paste the text inline.

The inline pattern was cargo-culted from old SDD, not a considered choice. SDD itself flipped, so flipping `review-article` is safe and well-precedented.

## Goals

1. **Token/context efficiency** — stop pasting the article 3× and holding three annotation sets in controller context.
2. **Review quality** — fix the broken prompts; add SDD's model-selection discipline.
3. **Robustness** — survive compaction mid-run for free.

## Non-goals

- No bash scripts. SDD ships `review-package`/`task-brief` because assembling a git diff is fiddly; `review-article` has no fiddly handoff step (input is already a file path, output is "write to this path"). Scripts would be ceremony.
- No progress ledger. The on-disk intermediate files already provide recovery; a separate ledger duplicates that.
- No structural rewrite to mirror SDD's section layout. Out of scope.
- Reviewers keep running **in parallel** (read-only, no write conflicts) — unlike SDD's serial implementers. This is correct for the domain and stays.

## Design

### S1 — Input handoff: reviewers Read, controller does not

- Remove from `SKILL.md`: "Before dispatching, read both the output file and `<filename>.md` into context" and "Construct each prompt string by substituting the literal file text inline — subagents do not read files themselves."
- Replace with: the controller passes each reviewer the **path** to `<filename>.md`; each reviewer Reads it. The grounding reviewer additionally reads the literature-summary directory (unchanged behavior).
- In all three prompt templates, remove the inline `[FULL TEXT ... paste it here]` blocks and replace with an instruction to Read `<filename>.md` at a path the controller substitutes.
- Effect: the article text never enters a dispatch prompt; the controller does not load the article at dispatch time.

### S2 — Output handoff: reviewers write files, return status only

- Each reviewer writes its footnotes to a dedicated file alongside the source:
  - prose → `<filename>-prose.md`
  - argument → `<filename>-argument.md`
  - grounding → `<filename>-grounding.md`
- Each file contains footnote definitions with **quoted sentence context** (enough for the controller to locate each insertion point in the source body), matching the current "annotations with sufficient sentence context" contract.
- The reviewer's returned message contains **only**: the verdict (✅ no issues / ❌ issues found), the file path it wrote, and the footnote count. The annotations themselves never enter chat.
- Effect: the controller never holds full annotation sets in context.

### S3 — Merge: controller reads source once

- The controller reads the three footnote files plus `<filename>.md` **once**, at merge time (not 3×, not resident all session).
- Per-footnote validity check (context clear + explanation + suggested rewrite); invalid footnotes go back to the originating reviewer via the existing re-review loop.
- Insert `[^rN]` markers into the source body at the quoted-context locations, collect definitions at the bottom, renumber sequentially in document order.
- Write `<filename>-reviewed.md`; delete the three intermediate files.
- Fix the counting bug: **three** intermediate files, not four, everywhere it appears (merge diagram and prose).

### S4 — Quality fixes

- Model Selection: keep the existing table (Prose=Haiku, Argument/Grounding=Sonnet); add SDD's warning — always specify the model explicitly when dispatching a subagent, because an omitted model silently inherits the session's most expensive model.
- Prose prompt: fix `Porpose` → `Purpose` and the broken sentence "Footnotes addressing prose issues in the to Strunk's guidelines"; ensure it now sources the text via the S1 Read instruction.

### S5 — Robustness (free, no ledger)

- Add one note to `SKILL.md`: the three intermediate footnote files persist on disk until the merge completes and deletes them. After a compaction mid-run, the controller recovers by re-reading whichever files already exist rather than re-dispatching reviewers.

## Net effect

- Article text in controller context: **1× at merge** (was 3× in prompts + resident all session).
- Reviewer annotations in chat: **0×** (was three full sets).
- Existing bugs fixed: missing text source in prose prompt, prose typos, the four-vs-three file-count contradiction, and the inline-vs-file-output inconsistency.

## Files touched

- `skills/review-article/SKILL.md` — Prompt Templates section, Model Selection, merge diagram/text, robustness note.
- `skills/review-article/prose-reviewer-prompt.md` — Read-the-file instruction, write-to-file output, typo fixes.
- `skills/review-article/argument-reviewer-prompt.md` — Read-the-file instruction, write-to-file output.
- `skills/review-article/grounding-reviewer-prompt.md` — Read-the-file instruction (keep lit-dir read), write-to-file output.

## Verification

The skill has no automated tests. Verify by running `review-article` on a sample `.md` article (e.g., from `.claude/notes/`) and confirming:

- No reviewer prompt contains the article body.
- Each reviewer returns only verdict + path + count.
- The three `-prose/-argument/-grounding` files are created, then deleted after merge.
- `<filename>-reviewed.md` contains the full source with `[^rN]` markers and matching definitions, no orphans in either direction.
