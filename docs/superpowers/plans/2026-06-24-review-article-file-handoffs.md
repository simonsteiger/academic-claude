# review-article File-Handoff Flip Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Flip the `review-article` skill from pasting source text inline to file handoffs — reviewers Read the source and write footnotes to per-reviewer files — and fix existing bugs in the prompts and merge step.

**Architecture:** Three Markdown prompt templates and one `SKILL.md` are edited so the controller passes file paths instead of file contents, reviewers Read the source themselves and write footnotes to dedicated files, and the controller reads the source once at merge to place markers. No code, no scripts, no tests framework — verification is grep-based assertions plus one manual end-to-end run.

**Tech Stack:** Markdown skill files; `grep`/`rg` for verification; the `review-article` skill itself for the manual run.

## Global Constraints

- Pass **paths**, never file contents, into reviewer prompts. No `[FULL TEXT ... paste it here]` blocks may remain.
- Reviewer output filenames are exact: `<filename>-prose.md`, `<filename>-argument.md`, `<filename>-grounding.md`.
- There are **three** intermediate files, never "four" — fix every occurrence.
- Reviewers stay **parallel** and **read-only on the source** (they only write their own footnote file). Do not serialize them.
- No bash scripts, no progress ledger, no structural rewrite of `SKILL.md` section layout.
- Each reviewer returns only: verdict (✅/❌) + the path it wrote + footnote count. Annotations never go into the reply.
- The grounding reviewer keeps reading the literature-summary directory.
- All edits are to files under `skills/review-article/`.

---

### Task 1: Rewrite the three reviewer prompt templates

**Files:**
- Modify: `skills/review-article/prose-reviewer-prompt.md`
- Modify: `skills/review-article/argument-reviewer-prompt.md`
- Modify: `skills/review-article/grounding-reviewer-prompt.md`

**Interfaces:**
- Consumes: nothing from earlier tasks.
- Produces: three prompt templates that expect the controller to substitute `<filename>.md` (source path), an output path (`<filename>-prose.md` / `-argument.md` / `-grounding.md`), and — grounding only — a literature directory. `SKILL.md` (Task 2) references these output filenames; they MUST match exactly.

- [ ] **Step 1: Replace `prose-reviewer-prompt.md` with the file-handoff version**

Overwrite the whole file with:

````markdown
# Prose Reviewer Prompt Template

Use this template when dispatching a prose reviewer subagent.

**Purpose:** Verify the author wrote clearly and concisely. You MUST NOT review any aspect of the text other than its prose.

```
Task tool (general-purpose):
  description: "Review prose of text"
  prompt: |
    You are reviewing whether the summary of an article adheres to Strunk's principles of clear and concise communication.

    You MUST USE `elements-of-style:writing-clearly-and-concisely` for this task if it is available.
    ONLY read SKILL.md and NOT elements-of-style.md unless you are specifically instructed to do so.

    ## Text for review

    Read the article at `<filename>.md` — substituted by the controller as the source path. Read it yourself; it is not pasted here.

    ## What Was Requested

    Footnotes addressing prose issues against Strunk's guidelines for clear and concise writing.

    ## CRITICAL: Do Not Assume The Text Follows Strunk's Principles

    Assume that the author of the text has questionable taste and is unaware of Strunk's principles. Their prose may be wordy, passive, or unclear. You MUST verify every sentence follows Strunk's principles carefully.

    **YOU MUST:**
    - Read the actual text they wrote
    - Compare actual content to Strunk's principles sentence by sentence
    - Test against each of the principles with laser-sharp focus

    ## Your Job

    You are a domain-aware collaborator:
    - Read the text and verify that the prose follows Strunk's principles.
    - Flag all the issues you found by quoting the original sentence and suggesting a rewrite.
    - Annotate the text with numbered footnotes — do not rewrite the full summary.

    **This task REQUIRES you to thoroughly read their text.**

    ## Output

    Write your footnotes to the output path the controller substitutes (`<filename>-prose.md`). Each footnote is a definition with enough quoted sentence context for the controller to locate it in the source. Do NOT return the annotations in your reply.

    Return ONLY:
    - The verdict: ✅ No issues found (if none within this review's scope) or ❌ Issues found
    - The path you wrote
    - The footnote count
```
````

- [ ] **Step 2: Verify the prose template flipped**

Run:
```bash
cd skills/review-article && \
  grep -n "paste it here\|FULL TEXT\|Porpose\|in the to Strunk" prose-reviewer-prompt.md; \
  echo "---present---"; \
  grep -c "Read the article at\|Write your footnotes to" prose-reviewer-prompt.md
```
Expected: the first grep prints nothing (no leftover paste blocks or typos); the count after `---present---` is `2`.

- [ ] **Step 3: Replace `argument-reviewer-prompt.md` with the file-handoff version**

Overwrite the whole file with:

````markdown
# Argument Reviewer Prompt Template

Use this template when dispatching an argument reviewer subagent.

**Purpose:** Review the logical structure, paragraph order, and transitions between sections.

```
Task tool (general-purpose):
  description: "Review argument quality of text"
  prompt: |
    You are reviewing whether the argumentative structure of a text is logical and links different units of content coherently.

    ## What Was Requested

    A review of the argumentative structure of the text. Additional fabricated content is STRICTLY disallowed and must be flagged.

    ## Text for review

    Read the article at `<filename>.md` — substituted by the controller as the source path. Read it yourself; it is not pasted here.

    ## CRITICAL: Do NOT assume the text is well-structured

    The text may be an early draft, or the author a poor writer.
    Their argumentation may be poorly structured, incoherent, with poor transitions and isolated units of content.

    **DO NOT:**
    - Assume that the text will be clear to other readers if it is confusing to you
    - Assume that the author makes a reasonable argument
    - Comfort the author by being lenient when the argument is of bad quality

    **DO:**
    - Read the actual text
    - Verify that the author's argument is logical and coherent
    - Question their argumentative logic and how it is delivered

    ## Your Job

    You are a domain-aware collaborator:
    - Review the text for argument structure.
    - Flag logical gaps and structural problems grounded in the text's specific scientific context.
    - Annotate the text with numbered footnotes — do not rewrite the full summary.

    **This task REQUIRES you to thoroughly read their text.**

    ## Output

    Write your footnotes to the output path the controller substitutes (`<filename>-argument.md`). Each footnote is a definition with enough quoted sentence context for the controller to locate it in the source. Do NOT return the annotations in your reply.

    Return ONLY:
    - The verdict: ✅ No issues found (if none within this review's scope) or ❌ Issues found
    - The path you wrote
    - The footnote count
```
````

- [ ] **Step 4: Replace `grounding-reviewer-prompt.md` with the file-handoff version**

Overwrite the whole file with:

````markdown
# Grounding Reviewer Prompt Template

Use this template when dispatching a grounding reviewer subagent.

**Purpose:** Review the scientific claims in the text against provided literature summaries and flag unsupported, overclaimed, or inconsistent statements.

```
Task tool (general-purpose):
  description: "Review grounding honesty of text"
  prompt: |
    You are reviewing whether the text is grounded in the provided literature.

    ## What Was Requested

    A review of the grounding honesty of the text.

    ## Text for review

    Read the article at `<filename>.md` — substituted by the controller as the source path. Read it yourself; it is not pasted here.

    ## Literature summaries

    Read ALL literature summaries in the directory the controller substitutes.
    If no literature was provided, skip this step and rely on your prior knowledge.

    [Literature summary directory – substituted by the controller]

    ## CRITICAL: Do NOT assume the text is grounded in the literature

    The author may have only read abstracts, or confused articles.
    Their references may be incorrect, or they may overclaim findings to support their own research agenda.

    **DO NOT:**
    - Skip reading any of the provided literature summaries
    - Assume that your prior knowledge suffices for reviewing the grounding honesty
    - Fabricate citations – these are STRICTLY prohibited

    **DO:**
    - Read ALL provided literature summaries
    - Prioritise information from provided literature over your prior knowledge
    - Support claims with a specific, real scientific reference
    - Flag uncertainties whenever you cannot verify an unsupported claim based on the provided literature or your prior knowledge; acknowledging uncertainty is CRUCIAL and COMMENDABLE in academic work!

    ## Your Job

    You are a domain-aware collaborator:
    - Review the scientific text for claim grounding.
    - Check each factual and interpretive claim against the provided literature.
    - Flag anything unsupported, overclaimed, or inconsistent with the evidence.
    - Annotate the text with numbered footnotes — do not rewrite the full summary.

    **This task REQUIRES you to thoroughly read their text.**

    ## Output

    Write your footnotes to the output path the controller substitutes (`<filename>-grounding.md`). Each footnote is a definition with enough quoted sentence context for the controller to locate it in the source. Do NOT return the annotations in your reply.

    Return ONLY:
    - The verdict: ✅ No issues found (if none within this review's scope) or ❌ Issues found
    - The path you wrote
    - The footnote count
```
````

- [ ] **Step 5: Verify all three templates flipped, no paste blocks remain**

Run:
```bash
cd skills/review-article && \
  echo "leftovers (want empty):"; \
  grep -rn "paste it here\|FULL TEXT\|don't make subagent read file" *-reviewer-prompt.md; \
  echo "read+write present (want 6):"; \
  grep -c "Read the article at\|Write your footnotes to" prose-reviewer-prompt.md argument-reviewer-prompt.md grounding-reviewer-prompt.md | awk -F: '{s+=$2} END{print s}'; \
  echo "grounding still reads lit dir (want >=1):"; \
  grep -c "literature summaries in the directory" grounding-reviewer-prompt.md
```
Expected: the leftovers grep prints nothing; the read+write sum is `6`; the lit-dir count is `1`.

- [ ] **Step 6: Commit**

```bash
git add skills/review-article/prose-reviewer-prompt.md skills/review-article/argument-reviewer-prompt.md skills/review-article/grounding-reviewer-prompt.md
git commit -m "feat: flip reviewer prompts to file handoffs

Reviewers Read the source path and write footnotes to per-reviewer
files; reply carries only verdict + path + count. Fix prose typos
(Porpose, broken Strunk sentence) and supply prose its missing text
source.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 2: Update `SKILL.md` — handoff wording, model warning, file-count fix, robustness note

**Files:**
- Modify: `skills/review-article/SKILL.md`

**Interfaces:**
- Consumes: the output filenames defined in Task 1 (`<filename>-prose.md`, `-argument.md`, `-grounding.md`).
- Produces: nothing later tasks consume in code; Task 3 manually exercises the result.

- [ ] **Step 1: Rewrite the Prompt Templates section**

Replace this block:
```markdown
## Prompt Templates

Before dispatching, read both the output file and `<filename>.md` into context. Construct each prompt string by substituting the literal file text inline — subagents do not read files themselves.

- `./prose-reviewer-prompt.md` — prose reviewer subagent prompt
- `./argument-reviewer-prompt.md` — argument reviewer subagent prompt
- `./grounding-reviewer-prompt.md` — grounding reviewer subagent prompt
```
with:
```markdown
## Prompt Templates

Do NOT read `<filename>.md` into your own context before dispatching — each reviewer Reads the source itself. For each reviewer, substitute the source path (`<filename>.md`) and the reviewer's output path into its template; the grounding reviewer also gets the literature-summary directory path. Pass paths, never file contents: pasted text stays resident in your context for the whole session and is re-read on every turn.

Output paths:
- prose → `<filename>-prose.md`
- argument → `<filename>-argument.md`
- grounding → `<filename>-grounding.md`

- `./prose-reviewer-prompt.md` — prose reviewer subagent prompt
- `./argument-reviewer-prompt.md` — argument reviewer subagent prompt
- `./grounding-reviewer-prompt.md` — grounding reviewer subagent prompt
```

- [ ] **Step 2: Add the model-discipline warning under Model Selection**

Immediately after the Model Selection table (the line ending `| Grounding  | Sonnet |`), add a blank line then:
```markdown
**Always specify the model explicitly when dispatching a subagent.** An omitted model inherits your session's model — often the most capable and most expensive.
```

- [ ] **Step 3: Fix the file-count bug in the merge diagram**

In the `digraph footnote_merge` block, change:
- `collect [label="1. Collect footnotes\nfrom all four intermediate files"]` → `collect [label="1. Collect footnotes\nfrom all three intermediate files"]`
- `delete  [label="5. Delete the four\nintermediate files"]` → `delete  [label="5. Delete the three\nintermediate files"]`

- [ ] **Step 4: State that the controller reads the source once at merge**

Immediately before the `### Step 4 — Merge` diagram's closing prose (the line beginning `The final `<filename>-reviewed.md` contains`), insert a new paragraph:
```markdown
Read `<filename>.md` once here — you need the source body to place the `[^rN]` markers. You deliberately did not load it at dispatch time, so this is the only point the full source enters your context.
```

- [ ] **Step 5: Add the robustness note**

After the `### Step 5 — Self-review` section (after its final paragraph "Fix any issues inline. No need to re-review after fixing — just correct and move on."), add:
```markdown
### Robustness

The three intermediate files (`<filename>-prose.md`, `-argument.md`, `-grounding.md`) persist on disk until the merge step deletes them. If your context is compacted mid-run, recover by re-reading whichever of these files already exist — do not re-dispatch a reviewer whose output file is already present.
```

- [ ] **Step 6: Verify SKILL.md is internally consistent**

Run:
```bash
cd skills/review-article && \
  echo "no word 'four' anywhere (want empty):"; \
  grep -niw four SKILL.md; \
  echo "no inline-paste instruction (want empty):"; \
  grep -n "substituting the literal file text inline\|subagents do not read files" SKILL.md; \
  echo "model warning present (want 1):"; \
  grep -c "Always specify the model explicitly" SKILL.md; \
  echo "robustness section present (want 1):"; \
  grep -c "^### Robustness" SKILL.md
```
Expected: the `four` grep prints nothing (both label fixes landed); the inline-paste grep prints nothing; model-warning count is `1`; robustness count is `1`. If `grep -niw four` surfaces an unrelated "four", confirm by eye it is not one of the two intermediate-file labels and proceed.

- [ ] **Step 7: Commit**

```bash
git add skills/review-article/SKILL.md
git commit -m "feat: file-handoff wording, model warning, three-file fix in review-article SKILL

Controller passes paths not contents, reads source once at merge; add
model-selection warning; fix four-vs-three intermediate-file count; add
robustness note for compaction recovery.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 3: End-to-end manual verification

**Files:**
- Create (temporary): `skills/review-article/.claude/notes/sample.md` (gitignored scratch — not committed)

**Interfaces:**
- Consumes: the flipped prompts (Task 1) and updated `SKILL.md` (Task 2).
- Produces: confirmation the skill runs end-to-end; no committed artifacts.

- [ ] **Step 1: Create a small sample article in gitignored scratch**

Write `skills/review-article/.claude/notes/sample.md` with ~3 short paragraphs of deliberately flawed scientific prose (a wordy passive sentence, one weak transition, one unsupported claim) so each reviewer has something to flag.

- [ ] **Step 2: Run the skill on the sample**

Invoke `review-article` on `.claude/notes/sample.md` (decline the literature step, or point it at an empty dir). Let it dispatch all three reviewers and merge.

- [ ] **Step 3: Verify the run honored the file-handoff contract**

Confirm by observation:
- No reviewer dispatch prompt contained the article body (only the path).
- Each reviewer's reply was only verdict + path + count.
- The three `sample-prose.md` / `sample-argument.md` / `sample-grounding.md` files were created during the run, then deleted after merge.
- `sample-reviewed.md` contains the full source with `[^rN]` markers and a matching definition for every marker — no orphan markers and no orphan definitions (this is the existing Step 5 self-review check).

If any check fails, fix the responsible template or `SKILL.md` and re-run before proceeding.

- [ ] **Step 4: Clean up scratch**

```bash
rm -f skills/review-article/.claude/notes/sample.md skills/review-article/.claude/notes/sample-reviewed.md
```
(The `.claude/notes/` directory is gitignored; nothing to commit.)

---

## Notes for the implementer

- These are documentation/skill files — there is no test runner. The "tests" are the grep assertions in each task; run them exactly and check the expected output.
- Do not introduce scripts or a progress ledger; the spec's non-goals forbid them.
- Keep the reviewers parallel and read-only on the source. The only thing a reviewer writes is its own footnote file.
