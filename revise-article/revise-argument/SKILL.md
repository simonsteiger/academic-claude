---
name: revise-argument
description: Use when reviewing argument structure in scientific text — logical gaps, paragraph coherence, and ordering of ideas. Invocable standalone or dispatched by the revise skill. Requires a file path argument. Produces a -reviewed.md file with inline footnote comments.
---

# Revise: Argument

Review the scientific text at the provided file path for argument structure. Act as a domain-aware collaborator: flag logical gaps and structural problems grounded in the text's specific scientific context.

## Invocation

```
/revise-argument <file-path> [focus instruction]
```

- **file-path** (required): path to the `.md` file to review.
- **focus instruction** (optional): narrows attention but does not replace full coverage.

## When to use

Use this sub-skill when:
- You want argument structure feedback only.
- You are a sub-agent dispatched by the `revise` skill to handle the argument dimension.

Do not use this skill to review prose quality or scientific grounding — those are handled by `revise-prose` and `revise-grounding`.

## Workflow

> Skip this section if dispatched by `revise` — the orchestrating skill tracks phases and runs its own self-review.

If invoked standalone:
1. Announce: "Using **revise-argument** to review `<filename>` for argument structure."
2. Use **TaskCreate** to create one task per phase. Mark each task complete before moving to the next.

- Phase 1 — Silent read
- Phase 3 — Annotate *(Phase 2 applies only when dispatched)*
- Phase 4 — Self-review

## Phase 1 — Silent read

Read the full text at the provided file path. Do not produce any output yet.

Then look for a `references/` folder in the same directory as the file.

- **Folder exists:** read all `.md` files silently. Understanding the scientific argument requires knowing what the literature establishes and what it does not.
- **Folder absent:** stop and ask the user whether they want to provide a `references/` folder or links to relevant literature. Do not proceed until the user responds.
  - If the user provides context: use it and proceed.
  - If the user provides nothing: proceed using prior knowledge as the primary source. This is the only case in which prior knowledge becomes the primary source.

## Phase 2 — If dispatched by `revise`

If dispatched by `revise`, you will receive approved argument items from the revision plan. Address only those items. Skip this phase if invoked standalone.

## Phase 3 — Annotate

Make a single pass through the text. For each argument problem, insert a footnote marker immediately after the affected span. If dispatched by `revise`, start numbering from the offset provided. If invoked standalone, start from `[^r1]`.

At the end of the file, add footnote definitions. Each footnote must contain:

1. `**[argument]**` tag
2. Brief explanation of the logical or structural problem
3. Suggested rewrite of the affected sentence or passage (or a bridging sentence to add)
4. Reference to supporting source where applicable

**Example:**

```
In this context, it would be valuable to be able to stratify patients[^r1].

---

[^r1]: **[argument]** The paragraph introduces patient stratification without connecting it to the dual-target strategy discussed in the preceding paragraph. A bridging sentence is needed. Suggested addition before this sentence: "If these two targets represent distinct clinical needs, classification schemes that separate them may predict treatment outcomes more precisely than composite scores."
```

## Argument criteria

Flag any of the following:

**Logical gaps**
- A conclusion that does not follow from the preceding sentences
- An implication that the author treats as established but has not argued for
- A leap from a specific finding to a broad claim

**Paragraph coherence**
- A paragraph whose topic sentence does not match its body
- A paragraph that introduces a concept but does not follow through on it
- A paragraph that ends without connecting back to the paper's main argument

**Ordering of ideas**
- A concept introduced before the reader has the background to understand it
- A key claim buried in the middle of a paragraph rather than placed at the end (the position of emphasis)
- An argument thread raised early and then dropped without resolution

**Transitions**
- An abrupt shift between paragraphs with no signal to the reader
- A transition word that misrepresents the logical relationship (e.g. "however" where there is no contrast)

## Output file

Write the annotated text to `<original-name>-reviewed-argument.md` in the same directory. If dispatched by `revise`, write to the intermediate file path provided by the orchestrating skill.

## Phase 4 — Self-review (standalone only)

> Skip if dispatched by `revise`.

After writing the output file, read it with fresh eyes:

1. **Coverage:** Did every argument problem you identified during the read get an annotation? If you silently dropped an issue, add it now.
2. **Marker integrity:** Every `[^rN]` marker in the body has a matching definition at the bottom, and vice versa. No orphans in either direction.
3. **Footnote quality:** Each footnote has an `**[argument]**` tag, a one-sentence explanation of the logical or structural problem, and a suggested rewrite or bridging sentence. Add any missing element.

Fix issues inline. No need to re-review after fixing.
