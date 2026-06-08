---
name: benchmarking-implementations
description: Use when writing implementation plans, proposing technical approaches, finishing implementation tasks, or about to commit to a solution without checking how established codebases implement similar behavior, APIs, naming, state, tests, or edge cases
---

# Benchmarking Implementations

## Overview

Search GitHub and mature codebases for implementation patterns before locking in a design.

**Core principle:** Your first design is only one possible design. Established codebases reveal naming conventions, data models, API shapes, tests, edge cases, and simpler implementation patterns you may miss.

This skill answers: **"How do good codebases build this?"**

**REQUIRED COMPANION:** Use `finding-existing-solutions` in a separate subagent for libraries, managed services, internal platforms, gateways, workflow engines, and other build-vs-adopt options.

## Planning Subagent

When planning a feature, dispatch a dedicated read-only subagent for this lane. Give it the feature goal and ask it to:

- Decompose the feature into searchable implementation subproblems.
- Search GitHub and code examples for each subproblem.
- Report naming conventions, data models, API boundaries, tests, edge cases, and reusable patterns.
- Stay in the code-pattern lane. Do not evaluate managed services or internal platforms; that belongs to `finding-existing-solutions`.

## When to Search

**MUST search at three points:**

1. **Brainstorming** — before proposing approaches, check how mature codebases model the behavior.
2. **Planning** — before writing tasks, confirm file structure, API shape, and test strategy against real examples.
3. **Post-implementation** — after code works, compare against established patterns and simplify if needed.

## Decompose Before Searching

Search for reusable subproblems, not only the full feature phrase.

Example: "accountant editing purchase-order invoices directly in a web UI" may not have a perfect reference. Decompose it into:

| Feature slice | Search target |
|---|---|
| Line-item editing | embedded spreadsheet, editable grid, invoice line editor |
| Validation | cross-field validation, tax/currency validation, tolerance checks |
| State transitions | approval workflow, draft/posted invoice states |
| Concurrency | optimistic updates, row locking, conflict resolution |
| Auditability | audit log, change history, event sourcing |
| Document context | side-by-side PDF viewer, document annotation |

## How to Search

1. **Formulate 2-3 code-pattern queries** from the decomposed slices.
2. **Execute them** — listing queries you "would" run does not count.
3. **Read top examples** — inspect actual code, not just README claims.
4. **Extract patterns** — names, types, data flow, test cases, edge cases, and boundaries.
5. **Document findings** — during planning: "Approach informed by [repo]'s [specific pattern]."

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "The exact feature is too domain-specific" | Search the subproblems: grids, validation, workflows, audit logs, concurrency. |
| "I already know how to implement this" | You know one way. Search reveals better names, states, tests, and edge cases. |
| "I checked our codebase" | Internal exploration is useful, but not benchmarking against established projects. |
| "GitHub did not find the whole product workflow" | Decompose and search the reusable implementation slices. |
| "Service discovery covers this" | Service discovery decides build-vs-adopt. This skill studies code patterns for the parts you still build. |

## Red Flags - STOP and Search

- About to write a plan without checking code patterns.
- Designing data models, state machines, or API names from scratch.
- Building editable tables, workflow state, validation, audit logs, permissions, or concurrency logic.
- Writing 50+ lines for something that mature repos likely solve.
- Saying "I would search" instead of executing searches.

## Quick Reference

```bash
gh search repos "<subproblem>" --language <lang> --sort stars --limit 5
gh search code "<pattern>" --language <lang> --limit 10
gh search code "<pattern>" --repo owner/repo --limit 10
gh api repos/owner/repo/contents/path/to/file | jq -r '.content' | base64 -d
```
