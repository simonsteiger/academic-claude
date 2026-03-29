# Accuracy Reviewer Prompt Template

Use this template when dispatching an accuracy reviewer subagent.

**Porpose:** Veryify implementer wrote about what was requested in the task requirements (summary spec) – nothing more, nothing less.

```
Task tool (general-purpose):
  description: "Review spec compliance of text"
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## What Was Requested

    [FULL TEXT of task requirements]

    ## What Implementer Claims They Built

    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer finished suspiciously quickly. Their report may be incomplete,
    inaccurate, or optimistic. You MUST verify everything independently.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust their claims about completeness
    - Accept their interpretation of requirements

    **DO:**
    - Read the actual text they wrote
    - Compare actual content to requirements line by line
    - Check for missing pieces they claimed to implement
    - Look for extra features they didn't mention

    ## Your Job

    Read the implemented text and verify:

    **Missing requirements:**
    - Did they implement everything that was requested?
    - Are there requirements they skipped or missed?
    - Did they claim a bullet point is included but didn't actually write about it?

    **Extra/unneeded work:**
    - Did they write about things that weren't requested?
    - Did they add overinterpret or add unnecessary detail?
    - Did they add "nice to haves" that weren't in spec?

    **Misunderstandings:**
    - Did they interpret requirements differently than intended?
    - Did they write about a different topic?
    - Did they summarise the right points but contextualise incorrectly?

    **Verify by reading their text, not by trusting report.**

    Report:
    - ✅ Spec compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing or extra, with file:line references]
```
