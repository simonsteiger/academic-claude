
### Prior knowledge policy

When drawing on prior knowledge rather than the provided summaries, support any grounding or argument claim with a specific scientific reference. **Fabricated or unverified citations are strictly prohibited.** When uncertain whether a source is real and directly relevant, flag the uncertainty explicitly rather than cite.

## References

Then look for a `references/` folder in the same directory as the file. This folder contains `.md` summaries of relevant research articles.

- **Folder exists:** read all `.md` files silently. These summaries are the primary source for grounding and argument feedback. Any comment on grounding or argument should, where possible, cite the relevant summary explicitly.
- **Folder absent:** stop and ask the user whether they want to provide a `references/` folder or links to relevant literature. Do not proceed until the user responds.
  - If the user provides context: use it and proceed.
  - If the user provides nothing: proceed using prior knowledge as the primary source. This is the only case in which prior knowledge becomes the primary source.

## Self-review

3. **Grounding honesty:** Scan the footnotes for any factual claim or citation that was drawn from prior knowledge. Confirm each such claim is appropriately flagged as uncertain, or is supported by a verifiable reference. If a footnote asserts something as fact without a source and without a caveat, add the caveat.

4. **Footnote quality:** Each footnote should contain a dimension tag, an explanation of the issue, and a suggested rewrite. If any footnote is missing one of these elements, add it.