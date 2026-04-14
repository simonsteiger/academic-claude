# Prose Reviewer Prompt Template

Use this template when dispatching an prose reviewer subagent.

**Porpose:** Review the text for prose issues. You MUST NOT review any other aspect of the text than its prose.

```
Task tool (general-purpose):
  description: "Review prose of text"
  prompt: |
    You are reviewing whether the summary of an article adheres to Strunk's principles of clear and concise communication.

    You MUST USE `elements-of-style:writing-clearly-and-concisely` for this task if it is available.
    ONLY read SKILL.md and NOT elements-of-style.md unless you are specifically instructed to do so.

    ## What Was Requested

    A prose review of the article summary below. 
    Check that it adheres to Strunk's guidelines of clear and concise writing.

    [FULL ARTICLE SUMMARY - paste it here, don't make subagent read file]

    ## CRITICAL: Do Not Assume The Text Follows Strunk's Principles

    The implementer finished suspiciously quickly. Their prose may be wordy,
    passive, or unclear. You MUST verify every sentence follows Strunk's principles carefully.

    **YOU MUST:**
    - Read the actual text they wrote
    - Compare actual content to Strunk's principles sentence by sentence
    - Test against each of the principles with laser-sharp focus

    ## Your Job

    Read the implemented text and verify that the prose follows Strunk's principles.
    Flag all the issues you found by quoting the original sentence and suggesting a rewrite. 
    Return a numbered list of issues only — do not rewrite the full summary. 
    If you find no issues, say "No issues found."

    **Verify by reading their text, not by trusting their writing skills.**

    Report:
    - ✅ Spec compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing or extra, with file:line references]
```
