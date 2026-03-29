---
name: revise-grounding
description: Use when checking scientific claims in text against provided literature summaries — flags unsupported, overclaimed, or inconsistent statements. Invocable standalone or dispatched by the revise skill. Requires a file path argument.
---

# Revise: Grounding

Review the scientific text at the provided file path for claim grounding. Check each factual and interpretive claim against the provided literature. Flag anything unsupported, overclaimed, or inconsistent with the evidence.

## Invocation

```
/revise-grounding <file-path> [focus instruction]
```

- **file-path** (required): path to the `.md` file to review.
- **focus instruction** (optional): narrows attention but does not replace full coverage.

## When to use

Use this sub-skill when:
- You want grounding feedback only.
- You are a sub-agent dispatched by the `revise` skill to handle the grounding dimension.

Do not use this skill to review prose quality or argument structure.

## Workflow

> Skip this section if dispatched by `revise` — the orchestrating skill tracks phases and runs its own self-review.

If invoked standalone:
1. Announce: "Using **revise-grounding** to review `<filename>` for claim grounding."
2. Use **TaskCreate** to create one task per phase. Mark each task complete before moving to the next.

- Phase 1 — Silent read
- Phase 3 — Annotate *(Phase 2 applies only when dispatched)*
- Phase 4 — Self-review

## Phase 1 — Silent read

Read the full text at the provided file path. Do not produce any output yet.

Then look for a `references/` folder in the same directory as the file.

- **Folder exists:** read all `.md` files in full. These summaries are the primary — and preferred — source for all grounding feedback. Any comment should, where possible, cite a specific summary by author and year.
- **Folder absent:** stop and ask the user whether they want to provide a `references/` folder or links to relevant literature. Do not proceed until the user responds.
  - If the user provides context: use it and proceed.
  - If the user provides nothing: proceed using prior knowledge as the primary source. This is the only case in which prior knowledge becomes primary.

## Prior knowledge policy

When drawing on prior knowledge (not from the provided summaries):
- You must support any grounding or argument claim with a specific, real scientific reference.
- **Fabricated or unverified citations are strictly prohibited.** If you are not certain a source is real and directly relevant to the specific claim, flag the uncertainty explicitly rather than cite.
- Prior knowledge citations must include author(s), year, and publication venue.

## Phase 2 — If dispatched by `revise`

If dispatched by `revise`, you will receive approved grounding items from the revision plan. Address only those items. Skip this phase if invoked standalone.

## Phase 3 — Annotate

Make a single pass through the text. For each grounding problem, insert a footnote marker immediately after the affected span. If dispatched by `revise`, start numbering from the offset provided. If invoked standalone, start from `[^r1]`.

Each footnote must contain:

1. `**[grounding]**` tag
2. Brief explanation of the grounding problem (unsupported, overclaimed, or inconsistent)
3. Suggested rewrite that aligns the claim with what the literature actually supports
4. Citation to the relevant summary (or a verified prior-knowledge citation)

**Example:**

```
studies have shown that a three-variable (3V) version of Boolean remission
without patient's global assessment (PGA) better predicts good radiographic
outcome[^r1]

---

[^r1]: **[grounding]** "Studies have shown" overstates the evidence base. The claim rests on a single meta-analysis. Suggested rewrite: "A meta-analysis of individual patient data found that 3V Boolean remission without PGA better predicted good radiographic outcome than 4V Boolean remission or SDAI." cf. Ferreira et al. (2020a).
```

## Grounding criteria

Flag any of the following:

**Unsupported claims**
- A factual statement with no basis in the provided literature or verifiable prior knowledge
- "Studies have shown…" or "It is well established…" without a specific reference
- A quantitative claim (percentage, effect size) stated without a source

**Overclaimed conclusions**
- A finding from one study generalised to all patients or all contexts
- A correlation presented as causation
- A result from a specific population (e.g. RCT patients) applied to a broader one

**Inconsistency with provided literature**
- A claim that contradicts or misrepresents a finding in the provided summaries
- A citation used to support a claim it does not actually support
- A nuance from the literature dropped in a way that makes the claim stronger than warranted

**Scope of this sub-skill**
The focus is on whether the claims that are made are scientifically defensible given the provided context. Citation gaps (missing references) are handled by `revise-citations`, not this skill.

## Output file

Write the annotated text to `<original-name>-reviewed-grounding.md` in the same directory. If dispatched by `revise`, write to the intermediate file path provided by the orchestrating skill.

## Phase 4 — Self-review (standalone only)

> Skip if dispatched by `revise`.

After writing the output file, read it with fresh eyes:

1. **Coverage:** Did every grounding problem you identified during the read get an annotation? If you silently dropped an issue, add it now.
2. **Marker integrity:** Every `[^rN]` marker in the body has a matching definition at the bottom, and vice versa. No orphans in either direction.
3. **Grounding honesty:** Every claim drawn from prior knowledge is either flagged as uncertain or backed by a verifiable citation with author, year, and venue. No fabricated sources — if uncertain, flag it rather than cite.
4. **Footnote quality:** Each footnote has a `**[grounding]**` tag, an explanation of the grounding problem, a suggested rewrite, and a citation where one was available. Add any missing element.

Fix issues inline. No need to re-review after fixing.
