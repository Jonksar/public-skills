# Debug Investigation Prompt Template

Use this template when dispatching a subagent to diagnose a failure.

```
Task tool:
  description: "Debug: [failure summary]"
  prompt: |
    ## Failure

    [What failed — error message, test name, unexpected behavior]

    ## Failure Output

    [Paste the relevant output — error traces, test results, logs.
     Only what's needed, not the entire terminal.]

    ## Hypothesis (if any)

    [Your best guess at root cause, or "no hypothesis" if investigating blind]

    ## Where to Look

    Start with: [likely files/areas]
    Related: [dependencies, callers, config]

    ## Your Job

    1. Reproduce or confirm the failure
    2. Identify root cause (not just symptoms)
    3. Propose a fix (don't implement unless told to)

    Do NOT just increase timeouts, add retries, or suppress errors.
    Find the real issue.

    ## Report Back

    - **Root cause**: [one-line summary]
    - **Evidence**: [file:line references, what you observed]
    - **Fix hypothesis**: [what should change and why]
    - **Confidence**: High / Medium / Low
    - **Side effects**: [anything the fix might break]
```
