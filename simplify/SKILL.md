---
name: simplify
description: Use when asked to simplify, refine, clean up, or reduce complexity in changed code while preserving behavior, especially when reviewing a diff for maintainability, reuse, performance, dead code, comments, or unnecessary abstractions
---

# Simplify

## Overview

Simplify scoped code by using parallel read-only review lanes, then make targeted cleanup fixes yourself.

**Core principle:** Improve clarity, reuse, and efficiency without changing behavior. Preserve all original features, outputs, and externally visible behavior.

## Scope Selection

1. If the user provides an explicit scope after `/simplify` or in plain language (paths, symbols, a diff, or an area), use that scope.
2. Otherwise inspect both unstaged and staged diffs so staged work is not missed:

```bash
git diff --no-color
git diff --cached --no-color
```

Treat the combined non-empty output as the scope.

3. If there is no local diff, fall back to concrete files, symbols, or changes mentioned in the conversation.
4. If that also does not exist, fall back to the current HEAD commit:

```bash
git show --stat --patch --no-color HEAD
```

Preserve unrelated user changes. Do not broaden the scope beyond the selected diff or mentioned files unless needed to understand existing patterns.

## Read-Only Review Lanes

Launch these three read-only lanes in parallel when subagents are available. They must only report findings: no edits, no formatters, no worktrees, no commits. Pass the full combined diff when possible; if too large, pass the file list, relevant hunks, and scope summary.

| Lane | Look for |
|---|---|
| Code quality | Low-information comments, one-off helpers, nullable state proliferation, catch-all error handling, unnecessary abstraction, weak type escape hatches, duplicated/derived state, dead compatibility code, nested ternaries, dense one-liners |
| Performance | Blocking hot-path operations, uncached expensive work, busy waits, repeated string concatenation, N+1 I/O, chatty logging or telemetry |
| Reuse | Existing helpers, local patterns, shared components, utilities, or abstractions already present in the codebase or diff |

If subagents are unavailable, do the three lanes yourself and keep their findings separated.

## Fixing

Aggregate the findings. Make targeted fixes that reduce complexity or reuse existing patterns while preserving exact behavior.

Skip recommendations that need more user context or require a much larger refactor than the scoped diff. Include skipped recommendations in the final summary.

After editing, run the most relevant lightweight checks for the touched files when practical. If checks are skipped or unavailable, say so.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Changing behavior to make code cleaner | Preserve behavior; propose behavior changes separately. |
| Broadening into unrelated refactors | Stay inside the selected scope. |
| Running formatters over untouched files | Only touch files needed for the simplification. |
| Trusting reviewer findings blindly | Apply judgment; skip risky or oversized recommendations. |
| Reporting only what changed | Also report useful recommendations you skipped and why. |
