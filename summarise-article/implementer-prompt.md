# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent.

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    You MUST USE `elements-of-style:writing-clearly-and-concisely` for this task if it is available.
    ONLY read SKILL.md and NOT elements-of-style.md unless you are specifically instructed to do so.

    ## Task Description

    [FULL ARTICLE and SUMMARY SPEC - paste it here, don't make subagent read file]

    ## Context

    Your task is to write a summary of a scientific article (the FULL ARTICLE pasted above)
    covering ONLY the points featured in the SUMMARY SPEC. 

    ## Before You Begin

    If you have questions about:
    - The requirements or acceptance criteria
    - The approach or implementation strategy
    - Anything unclear in the task description

    **Ask them now.** Raise any concerns before starting work.

    ## Your Job

    Once you're clear on requirements:
    1. Implement exactly what the task specifies
    2. Verify the implementation is complete
    3. Self-review (see below)
    4. Report back

    Work from: [directory]

    **While you work:** If you encounter something unexpected or unclear, **ask questions**.
    It's always OK to pause and clarify. Don't guess or make assumptions.

    ## When You're in Over Your Head

    It is always OK to stop and say "I don't understand the matter at hand." 
    Bad work is worse than no work. You will not be penalized for escalating.

    **STOP and escalate when:**
    - You need to understand information beyond what was provided and can't find clarity
    - You feel uncertain about whether your approach is correct
    - You think that the summary spec contradicts information found in the full text

    **How to escalate:** Report back with status BLOCKED or NEEDS_CONTEXT. Describe
    specifically what you're stuck on, what you've tried, and what kind of help you need.
    The controller can provide more context, re-dispatch with a more capable model,
    or break the task into smaller pieces.

    ## Before Reporting Back: Self-Review

    Review your work with fresh eyes. Ask yourself:

    **Completeness:**
    - Did I fully cover everything in the summary spec?
    - Did I miss any important points addressed in the spec?

    **Quality:**
    - Is this my best work?
    - Is the text I wrote concise, clear, and in active voice where possible?
    - Is the code clean and maintainable?

    **Discipline:**
    - Did I avoid overly complicated terms?
    - Did I only summarise what was requested?
    - Did I follow Strunk's principles for clear writing?

    If you find issues during self-review, fix them now before reporting.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or what you attempted, if blocked)
    - Files changed
    - Self-review findings (if any)
    - Any issues or concerns

    Use DONE_WITH_CONCERNS if you completed the work but have doubts about correctness.
    Use BLOCKED if you cannot complete the task. Use NEEDS_CONTEXT if you need
    information that wasn't provided. Never silently produce work you're unsure about.
```
