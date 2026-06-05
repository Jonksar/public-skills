# Research Dispatch Prompt Template

Use this template when dispatching a subagent to investigate code, gather context, or answer questions.

```
Task tool:
  description: "Research: [topic]"
  prompt: |
    ## Questions

    Answer these specific questions:
    1. [Question]
    2. [Question]

    ## Where to Look

    Start with: [file paths, directories, or patterns]
    Also check: [related areas if primary doesn't answer]

    ## What I Already Know

    [Any context that narrows the search — don't make the subagent
     re-discover what you already know]

    ## Report Back

    For each question:
    - **Answer**: [concise finding]
    - **Evidence**: [file:line references]
    - **Confidence**: High / Medium / Low

    Also flag anything unexpected or relevant you noticed
    that I didn't ask about.

    Keep answers concise — summaries, not full file contents.
```
