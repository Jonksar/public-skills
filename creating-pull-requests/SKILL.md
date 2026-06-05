---
name: creating-pull-requests
description: Use when creating a pull request, writing a PR description, or preparing branches for review
---

# Creating Pull Requests

## Overview

A good PR description is a narrative for the reviewer, not a diff summary. Write for someone who has zero context: explain what each piece does and **why it exists**.

## Step 0: Find the design file

Before writing anything, look for a matching spec or plan in `../docs/superpowers/`:

```bash
ls ../docs/superpowers/specs/
ls ../docs/superpowers/plans/
```

Match by feature name or branch name (e.g. branch `merit-credential-provisioning` → `2026-04-09-merit-credential-provisioning-design.md`).

**If a design file exists, mine it for:**

| Design file section | Use it for |
|---------------------|------------|
| `**Goal:**` / `## Goal` | Opening line of `## Summary` |
| `**Architecture:**` | Design rationale bullets in `## Summary` |
| `## File Structure` table | Component-by-component bullets |
| `## Guideline Exceptions` table | Add a `### Guideline exceptions` subsection |
| Task checkboxes (completed `[x]`) | Populate `## Test plan` |

If no design file exists, derive the summary from `git log` and the diff.

## Step 1: Check for conflicts against remote master

Before creating the PR, always fetch and merge the latest remote master to ensure the branch is conflict-free:

```bash
git fetch origin master
git merge origin/master
```

If conflicts arise, resolve them and commit the merge before proceeding. A PR with merge conflicts cannot be reviewed — catching them here avoids a wasted review cycle.

## Step 2: Pre-PR code review via subagent

Before writing the PR description, spawn a **read-only review subagent** to catch issues that reviewers would flag. The subagent must:

1. Run `git diff origin/master...HEAD` and `git log origin/master...HEAD --oneline` to get the full changeset.
2. Read every changed file in full.
3. Check against all applicable guidelines (`guidelines/` if present, AGENTS.md coding conventions, linting rules).
4. Identify concrete issues only — no praise, no style nitpicks unless enforced by a linter. Focus on:
   - Correctness bugs and logic errors
   - Violations of project coding conventions (e.g. wrong data class library, missing type hints, forbidden imports)
   - Test gaps (untested branches, missing edge cases)
   - Security or data-integrity issues
5. Return a **structured issue list** — one item per finding, each with: file path + line range, severity (`must-fix` / `consider`), and a one-sentence description.

Spawn the subagent like this (using the Task tool with `subagent_type: "generalPurpose"`, `readonly: true`):

```
Prompt: "Review the diff on this branch against [repo]'s coding conventions and guidelines.
Run: git diff origin/master...HEAD
Read every changed file.
Return a structured list of issues: file, line range, severity (must-fix or consider), description.
Do NOT fix anything. Do NOT praise. Only report concrete problems."
```

If the review returns **zero must-fix issues**, skip Step 3 and proceed to Step 4.

## Step 3: Fix issues via subagent

For each `must-fix` issue returned by the review subagent, spawn a **fix subagent** (write-enabled, `readonly: false`). Pass it:

- The full issue list from Step 2
- The relevant file paths and line ranges
- The project conventions to follow

The fix subagent must:

1. Fix all `must-fix` issues.
2. Optionally address `consider` items when the fix is unambiguous and low-risk.
3. Run the project's verification suite after fixes (tests, linting, type-check — whatever applies).
4. Commit all fixes in a single commit: `fix(scope): address pre-PR review findings`.

After the fix subagent completes, re-run the review subagent (Step 2) on the updated diff to confirm all `must-fix` items are resolved. Iterate until clean.

## Step 4: Write the PR description and open the PR

With the branch clean and the review subagent sign-off in hand, write the PR body using the structure below and open the PR.

## Body Structure

```markdown
## Summary

- **ComponentA**: what it does and why it was added
  - Sub-bullet for each major sub-component when the component has multiple parts
- **ComponentB**: what it does and why it was added

[Optional named subsections — add only when they materially help reviewers]

### Prerequisite: <name>
[If this PR depends on another PR or has a companion PR, describe the dependency and what it enables.]

### Design references
[Spec file references and any guideline exceptions.]

### What the integration tests exercise
[Table of test modules, assertions, and pass/fail/skip counts — use when integration tests are non-trivial.]

## Test plan

- [x] N unit tests pass
  ```
  ===== N passed in Xs =====
  ```
- [x] Specific verification step (e.g. "ruff/pyright/typecheck clean")
- [ ] Post-merge step (acceptable to leave unchecked)

### Why N tests are skipped
[Explain skip conditions in order — missing infra, services unavailable, missing fixtures.]
```

### Summary section rules

- One bullet per logical component — bold the name, explain the role and design intent
- Use **nested sub-bullets** when a component has multiple enumerable parts (e.g. pipeline stages, file-by-file breakdown)
- Add named subsections (`### Prerequisite: …`, `### Design references`, `### What the integration tests exercise`, `### Example`) only when they materially help
- Include code snippets or tables when the structure is non-obvious
- **No auto-generated content** — delete any `<!-- CURSOR_SUMMARY -->` / Cursor Bugbot blocks before opening the PR

### Test plan rules

- Every item you can verify locally must be checked `[x]` before the PR is opened
- Unchecked `[ ]` is acceptable only for post-merge steps (e.g. CI that requires master credentials)
- Name the specific command or outcome, not just "tests pass"
- **Attach proof** — paste a short terminal snippet under each non-trivial item:

```markdown
- [x] 120 unit tests pass
  ```
  ===== 120 passed in 4.31s =====
  ```
- [x] Pre-commit hooks pass
  ```
  ruff....................................................................Passed
  ruff-format.............................................................Passed
  trim trailing whitespace................................................Passed
  ```
- [ ] After merge: verify guideline step posts inline comment on a test PR
```

Keep snippets to the last 2–5 lines of output. Omit verbose tracebacks — only include them if a test was intentionally skipped or a known failure is being called out.

**When integration tests have non-trivial skip conditions**, add a `### Why N tests are skipped` subsection listing each skip trigger in order (missing sibling skill → service unavailable → missing fixture data). This turns a confusing "3 skipped" into actionable context.

**When integration tests span multiple modules**, use a table inside `### What the integration tests exercise`:

```markdown
| Module | What it asserts | Passed | Failed | Skipped | Accuracy |
|--------|-----------------|--------|--------|---------|----------|
| `test_stage_1` | validates X against live data | 2 | 0 | 0 | **100%** (2/2) |
| **Total** | `uv run pytest … -m integration` | **N** | **0** | **M** | **100% of executed** |
```

**When the PR includes an eval suite**, add it as a subsection at the bottom of the test plan:

```markdown
### Eval suite (`@pytest.mark.eval`)

| Layer | Module | Cases | How "accuracy" is judged |
|-------|--------|-------|--------------------------|
| **L1** | `test_reasoning.py` | N | LLM judge (Haiku) on natural-language criteria |
| **L2** | `test_flow.py` | N | Agent must narrate all stages; same criteria + judge pattern |
| **L3** | `test_posting.py` | N | Real API post; judge + GET assertion on totals |

**Requirements:** list env vars needed.

**Run:** paste the pytest command(s).

- [x] Eval suite green locally (paste short summary)
```

## Title

Use sentence case, not a commit-message prefix:

```
✅  Add eval framework with assistant service smoke test
✅  Code execution runtime credentials and dependencies
❌  feat: Add eval framework   ← commit-message style
❌  add eval framework         ← lowercase
```

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org):

```
feat(scope): short description under 72 chars
fix(scope): short description under 72 chars
refactor(scope): short description under 72 chars
```

- Subject line ≤ 72 characters — GitHub truncates beyond that
- Add a body paragraph when the commit is non-obvious (what + why)
- No commit messages like `(fix) Lint fix` or `PR fixes` — they belong in the branch history, not the PR

## Full Example

```markdown
## Summary

- **EvalCase model**: `msgspec.Struct` with `case_id`, `prompt`, `criteria`, and optional `expected_skill`
  for parametrized eval tests
- **LLM judge**: Haiku-based evaluator that scores assistant responses against criteria and returns
  structured pass/fail verdicts
- **Generic assertions**: reusable helpers for checking skill loading, code execution, and non-empty
  responses on `FullMessage` objects
- **Smoke test**: sends a greeting through the full assistant pipeline (real Anthropic API,
  fake persistence) and judges the response
- **Infrastructure**: `eval` pytest marker, `NoMcpToolsProvider` test double, gitignore for eval
  artifacts and skill `_config.json`

The `assistant_service` fixture reuses `Container` from `dependencies.py` with fake
manager/publisher, avoiding Bus construction duplication.

### Example: adding an eval case

\`\`\`python
_SMOKE_CASES: tuple[EvalCase, ...] = (
    EvalCase(
        case_id="greeting",
        prompt="Hello, what can you help me with?",
        criteria=(
            "Response is a helpful greeting that describes the assistant's capabilities.",
            "Response is not a refusal or error message.",
        ),
    ),
)
\`\`\`

Each case is parametrized and judged by an LLM against its criteria — add new cases by
appending to the tuple.

### What the integration tests exercise

| Module | What it asserts | Passed | Failed | Skipped | Accuracy |
|--------|-----------------|--------|--------|---------|----------|
| **`test_smoke`** | Full pipeline via live API, greeting criteria | 1 | 0 | 0 | **100%** (1/1) |
| **Total** | `uv run pytest … -m eval` | **1** | **0** | **0** | **100% of executed** |

## Test plan

- [x] 120 unit tests pass
  ```
  ===== 120 passed in 4.31s =====
  ```
- [x] Eval smoke test passes with live Anthropic API (10.8s)
  ```
  PASSED tests/ai_assistant/eval/test_smoke.py::test_eval_case[greeting] (10.8s)
  ===== 1 passed in 10.84s =====
  ```
- [x] Pre-commit hooks pass
  ```
  ruff....................................................................Passed
  ruff-format.............................................................Passed
  trim trailing whitespace................................................Passed
  ```
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Leaving Cursor Bugbot `<!-- CURSOR_SUMMARY -->` in the body | Delete it — it's auto-generated noise |
| `[ ]` items in test plan when PR is ready | Run the verification; check the box |
| Commit message subject > 72 chars | Shorten the subject; move detail to the body |
| Commit message like `(fix) Lint fix` | Use `fix(scope): descriptive reason` |
| Title like `feat: Add X` | Drop the prefix; use sentence case |
| Bullets that only say WHAT | Add WHY: design intent, constraint, or tradeoff |
| `[x]` with no output | Paste 2–5 lines of terminal proof under each non-trivial item |
| "3 skipped" with no context | Add `### Why N tests are skipped` listing each skip trigger in order |
| Flat bullets for a multi-stage component | Use nested sub-bullets for each enumerable part (e.g. pipeline stages) |
| Prerequisite PR mentioned only in passing | Add a named `### Prerequisite: …` subsection explaining what it enables |
| Integration test results as prose | Use the accuracy table format: Module / What it asserts / Passed / Failed / Skipped / Accuracy |
| PR opened with merge conflicts against master | Always `git fetch origin master && git merge origin/master` before creating the PR |
| Skipping the review subagent to save time | The review subagent catches issues reviewers will flag anyway — running it costs less than a review round-trip |
| Fix subagent commits with a vague message | Use `fix(scope): address pre-PR review findings` so the fix is traceable |
| Opening PR before re-running review after fixes | Always re-run the review subagent after the fix subagent to confirm all `must-fix` items are resolved |
