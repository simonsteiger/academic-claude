# Accuracy Reviewer Prompt Template

Use this template when dispatching an accuracy reviewer subagent.

**Porpose:** Veryify implementer wrote about what was requested in the task requirements (summary spec) – nothing more, nothing less.

```
Task tool (general-purpose):
  description: "Review accuracy of text"
  prompt: |
    You are reviewing whether the summary of an article accuractely represents the original article.

    ## What Was Requested

    An accuracy review of the summary below. Check dilligently that the summary contains ONLY information from the original article, and is free of misrepresentations and / or misinterpretations. Additional fabricated content is STRICTLY disallowed and must be flagged.

    ## Article Summary and Original Article

    ### Article Summary

    [FULL ARTICLE SUMMARY - paste it here, don't make subagent read file]
    
    ### Original Article

    [FULL ORIGINAL ARTICLE - paste it here, don't make subagent read file]

    ## What Implementer Claims They Built

    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer finished suspiciously quickly. Their report may be incomplete,
    inaccurate, or incorrect. You MUST verify everything independently.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust their claims about completeness or accuracy
    - Accept their interpretation of requirements

    **DO:**
    - Read the actual article summary and original article
    - Compare actual content to requirements line by line
    - Look for fabricated or inaccurate content they didn't mention

    ## Your Job

    Read the implemented text and verify:

    **Fabricated or inaccurate content:**
    - Did they write about things that weren't in the original article?
    - Did they add overinterpret or add unnecessary detail?

    **Misunderstandings:**
    - Did they write about a different topic?
    - Did they summarise the right points but contextualise incorrectly?

    **Verify by reading their text, not by trusting report.**

    Report:
    - ✅ Spec compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing or extra, with file:line references]
```
