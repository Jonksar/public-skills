---
name: code-review
description: Use when asked to review code, review a PR, inspect a diff before merge, find bugs or regressions, check CLAUDE.md or AGENTS.md compliance, or produce high-confidence actionable code review findings
---

# Code Review

## Overview

Review changed code for high-confidence correctness, regression, and guideline issues. This skill adapts Anthropic's Code Review plugin pattern: independent review angles, confidence filtering, and concise actionable findings.

**Core principle:** Review for bugs and violations a senior engineer would act on. Filter noise aggressively.

## Scope Selection

Choose the narrowest review target:

1. If the user gives a PR number or URL, review that PR.
2. If there is a local branch with an upstream base, review `git diff origin/main...HEAD` or the repo's default base.
3. If there are uncommitted changes, review both:

```bash
git diff --no-color
git diff --cached --no-color
```

4. If no diff exists, use explicit files or symbols mentioned by the user. If none exist, ask for scope.

Preserve unrelated user changes. Do not fix code unless the user explicitly asks for fixes.

## Review Lanes

Use independent read-only lanes when available; otherwise perform each lane yourself and keep findings separated:

| Lane | Check |
|---|---|
| Eligibility | Skip closed, draft, trivial automated, or already-reviewed PRs. |
| Guideline compliance | Read relevant `CLAUDE.md`, `AGENTS.md`, and local guidelines. Flag only newly introduced, specific violations. |
| Bug scan | Read changed files and hunks for logic errors, broken edge cases, data loss, security issues, and regressions. |
| Historical context | Inspect git blame/history for modified areas to catch intent or contract violations. |
| Prior review context | Search prior PRs/comments touching the same files for recurring issues. |
| Code comment consistency | Verify changed code still honors nearby comments and documented invariants. |

## Confidence Filter

Score each possible issue before reporting:

| Score | Meaning |
|---|---|
| 0 | False positive or pre-existing issue. |
| 25 | Plausible but unverified. |
| 50 | Real but minor, rare, or not important for this PR. |
| 75 | Likely real and important. |
| 100 | Definitely real, directly evidenced, likely to occur. |

Report only issues scoring **80+** unless the user explicitly asks for speculative findings.

Do not report issues that linters, typecheckers, compilers, or formatters will catch unless the user asked for that category.

## GitHub PR Reviews

Use `gh` for PR data when available:

```bash
gh pr view <pr> --json number,title,state,isDraft,headRefOid,baseRefName,files,body
gh pr diff <pr> --patch
gh pr list --search "<file or topic>"
```

If posting comments, cite links with the repository name, full commit SHA, file path, and line range:

```text
https://github.com/owner/repo/blob/<full-sha>/path/file.ext#L10-L15
```

## Output Format

Lead with findings. Keep summaries short.

```markdown
### Findings

1. [severity] `file:line` Brief issue title
   Why it matters. Evidence. Suggested fix.

### Open Questions

- Any assumptions that affect the review.

### Review Summary

- Scope reviewed.
- Checks not run or unavailable.
```

If no high-confidence issues are found, say that clearly and mention residual risk or checks not run.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Reporting nitpicks as bugs | Filter below 80 confidence. |
| Reviewing code outside the diff | Mention only when needed to explain a changed-line issue. |
| Flagging pre-existing issues | Report only newly introduced or newly exposed problems. |
| Ignoring repo guidance | Read relevant `CLAUDE.md`, `AGENTS.md`, and guidelines first. |
| Posting broken links | Use full SHA and `#Lstart-Lend` GitHub links. |
