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
    - Annotate with footnotes labelled `[^p1]`, `[^p2]`, … — the `p` prefix is yours and keeps your labels from colliding with the other reviewers'. Do not rewrite the full summary.

    **This task REQUIRES you to thoroughly read their text.**

    ## Output

    Write your footnotes to the output path the controller substitutes (`<filename>-prose.md`), as a list in this EXACT format so the controller can place each one mechanically:

    ## Footnotes

    ### [^p1]
    **Anchor:** "verbatim quoted sentence from the source that this marker attaches to"
    **Note:** the issue, the explanation, and your suggested rewrite

    ### [^p2]
    **Anchor:** "..."
    **Note:** ...

    Use the `p` prefix and number sequentially within it (`[^p1]`, `[^p2]`, …). Copy each Anchor verbatim from the source so it can be located. Do NOT return the annotations in your reply.

    Return ONLY:
    - The verdict: ✅ No issues found (if none within this review's scope) or ❌ Issues found
    - The path you wrote
    - The footnote count
```
