# Task Dispatch Prompt Template

Use this template when dispatching a subagent to implement, fix, or modify something.

```
Task tool:
  description: "[3-5 word summary]"
  prompt: |
    ## Task

    [What to accomplish — specific and measurable]

    ## Context

    [Where this fits: dependencies, architecture, relevant decisions.
     Paste relevant content directly — don't make the subagent read files.]

    ## Scope

    Work in: [directory/files]
    Do NOT modify: [boundaries]

    ## Before You Begin

    If anything is unclear about requirements, approach, or assumptions
    — ask now. Don't guess.

    ## Your Job

    1. [Specific step]
    2. [Specific step]
    3. Verify your work
    4. Commit changes

    ## When You're Stuck

    It's always OK to stop and escalate. Bad work is worse than no work.

    STOP and report BLOCKED or NEEDS_CONTEXT when:
    - Task requires architectural decisions with multiple valid approaches
    - You need context beyond what was provided
    - You feel uncertain about correctness
    - You've been exploring without progress

    ## Report Back

    - **Status:** DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
    - What you did (or attempted)
    - Files changed
    - Test results (if applicable)
    - Concerns or issues
```
