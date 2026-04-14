# Grounding Reviewer Prompt Template

Use this template when dispatching a grounding reviewer subagent.

**Purpose:** Review the scientific claims in the text against provided literature summaries and flag unsupported, overclaimed, or inconsistent statements.

```
Task tool (general-purpose):
  description: "Review grounding honesty of text"
  prompt: |
    You are reviewing whether the text is grounded in the provided literature.

    ## What Was Requested

    A review of the grounding honesty of the text below.

    ## Text for review

    [FULL TEXT for review - paste it here, don't make subagent read file]
    
    ## Literature summaries
    
    Read ALL literature summaries in the directory below. 
    If no literature was provided, skip this step and rely on your prior knowledge.
    
    [Literature summary directory – paste it here]

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

    Report:
    - ✅ No issues found (if no issues within this review's scope were found)
    - ❌ Issues found: [annotations with sufficient sentence context]
```
