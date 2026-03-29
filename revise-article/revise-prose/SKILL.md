---
name: revise-prose
description: Use when revising prose quality in scientific text — clarity, concision, active voice, and sentence flow. Invocable standalone or dispatched by the revise skill. Requires a file path argument. Produces a -reviewed.md file with inline footnote comments.
---

# Revise: Prose

> **For agentic workers:** REQUIRED SUB-SKILL: Use `elements-of-style:writing-clearly-and-concisely` to guide prose evaluation and rewrite suggestions. If this skill is not available on your system, state that it was not found and suggest the user to install it from https://github.com/obra/the-elements-of-style/

Review the scientific text at the provided file path for prose quality. Act as a domain-aware collaborator, not a copy-editor: flag problems and suggest rewrites grounded in the text's own argument.

## Invocation

```
/revise-prose <file-path> [focus instruction]
```

- **file-path** (required): path to the `.md` file to review.
- **focus instruction** (optional): e.g. "pay particular attention to paragraph 3". Narrows attention but does not replace full coverage.

## When to use

Use this sub-skill when:
- You want prose feedback only, without argument or grounding review.
- You are a sub-agent dispatched by the `revise` skill to handle the prose dimension.

Do not use this skill to review argument structure or scientific grounding — those are handled by `revise-argument` and `revise-grounding`.

## Workflow

> Skip this section if dispatched by `revise` — the orchestrating skill tracks phases and runs its own self-review.

If invoked standalone:
1. Announce: "Using **revise-prose** to review `<filename>` for prose quality."
2. Use **TaskCreate** to create one task per phase. Mark each task complete before moving to the next.

- Phase 1 — Silent read
- Phase 3 — Annotate *(Phase 2 applies only when dispatched)*
- Phase 4 — Self-review

## Phase 1 — Silent read

Read the full text at the provided file path. Do not produce any output yet.

Then look for a `references/` folder in the same directory as the file. This folder contains `.md` summaries of relevant research articles.

- **Folder exists:** read all `.md` files silently. Use them to understand the argument and domain context before evaluating prose.
- **Folder absent:** stop and ask the user whether they want to provide a `references/` folder or links to relevant literature. Do not proceed until the user responds.
  - If the user provides context: use it and proceed.
  - If the user provides nothing: proceed using prior knowledge as the primary source. This is the only case in which prior knowledge becomes the primary source.

## Phase 2 — If dispatched by `revise`

If you were dispatched by the `revise` skill, you will receive a list of approved prose items from the revision plan. Only address those items. Skip this phase if invoked standalone.

## Phase 3 — Annotate

Make a single pass through the text. For each prose problem, insert a footnote marker (`[^r1]`, `[^r2]`, …) immediately after the affected span. If dispatched by `revise`, start numbering from the offset provided. If invoked standalone, start from `[^r1]`.

At the end of the file, add the footnote definitions. Each footnote must contain:

1. `**[prose]**` tag
2. Brief explanation of the issue (one sentence)
3. Suggested rewrite of the affected sentence or passage
4. Reference to supporting source where applicable

**Example:**

```
The treatment of RA has evolved dramatically[^r1] over the course of the last decades.

---

[^r1]: **[prose]** "Evolved dramatically" is vague. Suggested rewrite: "New biologics and the treat-to-target strategy have transformed RA management over the past two decades."
```

## Prose criteria

Flag any of the following:

**Clarity and concision**
- Vague or abstract language where concrete language is available ("has evolved dramatically", "various studies have shown")
- Redundant phrases ("in order to" → "to", "due to the fact that" → "because")
- Nominalizations that obscure the verb ("make a decision" → "decide", "provide a description" → "describe")

**Active voice**
- Passive constructions where an active form is available and the agent is known ("it has been shown that" → "Ferreira et al. showed that")
- Impersonal constructions that distance the author from the claim

**Flow and transitions**
- Abrupt topic shifts between sentences with no transition
- Paragraphs that open with a sentence that does not preview their content
- Sentences placed in weak positions (topic buried at end, emphasis buried in the middle)

**Strunk's principles to apply throughout**
- Rule 10: Use active voice
- Rule 11: Put statements in positive form
- Rule 12: Use definite, specific, concrete language
- Rule 13: Omit needless words
- Rule 18: Place emphatic words at end of sentence

## Output file

Write the annotated text to `<original-name>-reviewed-prose.md` in the same directory as the source. If dispatched by `revise`, write to the intermediate file path provided by the orchestrating skill.

## Phase 4 — Self-review (standalone only)

> Skip if dispatched by `revise`.

After writing the output file, read it with fresh eyes:

1. **Coverage:** Did every prose problem you noticed during the read get an annotation? If you silently dropped an issue, add it now.
2. **Marker integrity:** Every `[^rN]` marker in the body has a matching definition at the bottom, and vice versa. No orphans in either direction.
3. **Footnote quality:** Each footnote has a `**[prose]**` tag, a one-sentence explanation of the issue, and a suggested rewrite. Add any missing element.

Fix issues inline. No need to re-review after fixing.
