---
name: revise-citations
description: Use when scanning scientific text for citation gaps — places where a claim is made without a supporting reference. Flags missing citations and suggests up to three candidate references when found. Invocable standalone or dispatched by the revise skill. Requires a file path argument.
---

# Revise: Citations

Scan the scientific text at the provided file path for citation gaps. A citation gap is any factual, quantitative, or interpretive claim that lacks a supporting reference and where a reader would reasonably expect one.

**Core principle:** Flag that a citation is missing. Do not fabricate a reference to fill the gap. Stating uncertainty is always preferable to inventing a source.

## Invocation

```
/revise-citations <file-path> [focus instruction]
```

- **file-path** (required): path to the `.md` file to review.
- **focus instruction** (optional): narrows attention but does not replace full coverage.

## When to use

Use this sub-skill when:
- You want to check for missing citations only.
- You are a sub-agent dispatched by the `revise` skill to handle the citations dimension.

Do not use this skill to evaluate whether existing citations are scientifically accurate or well-matched to the claim they support — that is the scope of `revise-grounding`.

## Workflow

> Skip this section if dispatched by `revise` — the orchestrating skill tracks phases and runs its own self-review.

If invoked standalone:
1. Announce: "Using **revise-citations** to review `<filename>` for citation gaps."
2. Use **TaskCreate** to create one task per phase. Mark each task complete before moving to the next.

- Phase 1 — Silent read
- Phase 3 — Annotate *(Phase 2 applies only when dispatched)*
- Phase 4 — Self-review

## Phase 1 — Silent read

Read the full text at the provided file path. Do not produce any output yet.

Then look for a `references/` folder in the same directory as the file.

- **Folder exists:** read all `.md` files silently. Use them as the first source when searching for candidate references to fill gaps.
- **Folder absent:** stop and ask the user whether they want to provide a `references/` folder or links to relevant literature. Do not proceed until the user responds.
  - If the user provides context: use it and proceed.
  - If the user provides nothing: proceed using prior knowledge as the sole source for candidate references.

## Phase 2 — If dispatched by `revise`

If dispatched by `revise`, you will receive approved citation items from the revision plan. Address only those items. Skip this phase if invoked standalone.

## Phase 3 — Annotate

Make a single pass through the text. For each citation gap, insert a footnote marker immediately after the span that needs a reference. If dispatched by `revise`, start numbering from the offset provided. If invoked standalone, start from `[^r1]`.

Each footnote must contain:

1. `**[citations]**` tag
2. A brief statement that a reference is missing and why one is expected here
3. Candidate references, if any were found — up to three, ordered by relevance. Prefer to state fewer if uncertain about fit. If no candidates were found, state that explicitly.

**Hallucination rule:** Never cite a source you cannot verify is real and directly relevant. When in doubt, name the type of source that would be needed (e.g. "a clinical trial reporting remission rates in early RA") rather than inventing an author and year.

**Example — candidate found in provided literature:**

```
Precise definitions of the treatment targets are paramount to the success
of the T2T strategy[^r1]: overly lenient definitions risk undertreating
patients with active disease, while overly strict definitions risk
overtreating patients whose disease is in remission.

---

[^r1]: **[citations]** This claim about the consequences of mis-calibrated treatment targets needs a supporting reference. Candidate from provided literature: Felson et al. (2011) — directly addresses the trade-off between overly lenient and overly strict remission definitions.
```

**Example — no candidate found:**

```
composite scores discard information from the individual component
variables[^r1]

---

[^r1]: **[citations]** This methodological claim about composite scores lacks a citation. No candidate found in the provided literature. A reference to a paper discussing the properties of summative disease activity scores (e.g. DAS28 or SDAI construction) would be appropriate here.
```

## Citation gap criteria

Flag spans where:

- A factual claim is stated without any reference (e.g. "studies have shown", "it has been demonstrated")
- A quantitative figure (percentage, odds ratio, p-value) appears without a source
- A clinical guideline or recommendation is cited informally without a formal reference
- A named tool, score, or index is introduced without a methodological reference
- A causal or interpretive claim is made that rests on empirical evidence the reader cannot verify from the text

Do not flag:
- Spans that already carry a citation marker
- Claims that are self-evident or definitional (e.g. "RA is a chronic inflammatory disease")
- The author's own stated aims or hypotheses

## Searching for candidates

Search in this order:

1. `.md` files in `references/` — prefer these; cite by author and year as given in the summary
2. Prior knowledge — only if the provided literature does not contain a suitable candidate. Verify the source is real before citing. Include author(s), year, and publication venue.

State up to three candidates. If uncertain about fit, state one or none and explain why you are uncertain.

## Output file

Write the annotated text to `<original-name>-reviewed-citations.md` in the same directory. If dispatched by `revise`, write to the intermediate file path provided by the orchestrating skill.

## Phase 4 — Self-review (standalone only)

> Skip if dispatched by `revise`.

After writing the output file, read it with fresh eyes:

1. **Coverage:** Did every citation gap you spotted during the read get a footnote? If you silently dropped one, add it now.
2. **Marker integrity:** Every `[^rN]` marker in the body has a matching definition at the bottom, and vice versa. No orphans in either direction.
3. **Hallucination check:** Every candidate reference in a footnote was verified as real before being suggested. If any candidate is uncertain, add a caveat or replace it with a description of the type of source needed.
4. **Footnote quality:** Each footnote has a `**[citations]**` tag, a brief statement of why a reference is expected, and either at least one candidate or an explicit "no candidate found." Add any missing element.

Fix issues inline. No need to re-review after fixing.
