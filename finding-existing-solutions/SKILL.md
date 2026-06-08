---
name: finding-existing-solutions
description: Use when planning or proposing a feature where a library, SDK, managed service, internal platform, gateway, workflow engine, infrastructure product, or vendor API might solve the capability
---

# Finding Existing Solutions

## Overview

Search beyond code examples before designing custom infrastructure.

**Core principle:** Many hard problems are better adopted than rebuilt. Look for libraries, managed services, internal platforms, gateways, workflow engines, and vendor APIs before committing to custom code.

This skill answers: **"Should we build this ourselves, or use an existing capability?"**

**REQUIRED COMPANION:** Use `benchmarking-implementations` in a separate subagent for GitHub/code-pattern research on the parts you still build.

## Planning Subagent

When planning a feature, dispatch a dedicated read-only subagent for this lane. Give it the capability and ask it to:

- Name the capability, not the intended implementation.
- Search libraries, SDKs, managed services, vendor docs, internal platform catalogs, and infrastructure products.
- Identify build-vs-adopt options, operational ownership, integration cost, risks, and what custom code remains.
- Stay in the solution-surface lane. Do not deep-dive code patterns; that belongs to `benchmarking-implementations`.

## Capability First

Describe the capability without assuming the implementation:

| Intended build | Capability name |
|---|---|
| Write token counter tables | LLM usage metering and budget enforcement |
| Build invoice editor workflow | Accounts-payable invoice editing, matching, and approval |
| Implement retry scheduler | Durable workflow orchestration |
| Add custom auth checks | Authorization policy enforcement |

## Solution Surface Checklist

Search at least three relevant surfaces:

| Surface | Look for | Examples |
|---|---|---|
| Libraries and SDKs | Drop-in APIs, adapters, CLIs | package registries, official SDK docs |
| Managed services | Hosted capability with operational ownership | gateways, queues, billing, observability |
| Internal platforms | Company-owned reusable services | Bifrost-style LLM gateways, platform catalogs |
| Infrastructure engines | Durable workflow/state machinery | Temporal-style workflows, schedulers, queues |
| Vendor APIs | Provider-native primitives | token APIs, quota APIs, billing exports |

## Infrastructure-Shaped Problems

GitHub-only benchmarking is incomplete when the problem involves:

- Orchestration, retries, background jobs, or scheduling.
- Metering, billing, quotas, budgets, or cost attribution.
- Auth, permissions, audit logs, compliance, or policy.
- Observability, tracing, evaluation, or analytics.
- LLM gateways, routing, provider abstraction, or token accounting.
- Queues, storage, caching, search, or workflow state.

For these, explicitly check services and platforms before proposing a custom build.

## How to Search

1. **Search current docs and catalogs** — public docs for vendor/service options; internal docs/service catalogs if available.
2. **Search package registries** — npm, PyPI, crates, Maven, etc. for maintained libraries.
3. **Search vendor/provider docs** — official APIs often expose primitives you were about to recreate.
4. **Ask when access is missing** — if internal platform catalogs are unavailable, ask the human whether a service like Bifrost, Temporal, billing, quota, or observability already exists.
5. **Document the decision** — adopt, integrate, thin custom wrapper, or custom build with reasons.

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "We can wire it ourselves quickly" | That is the not-invented-here trap. Check existing capabilities first. |
| "GitHub search found patterns" | Code patterns do not answer build-vs-adopt. Search services and platforms too. |
| "A service might be overkill" | Maybe. You still need to know it exists and reject it deliberately. |
| "Internal platforms are hard to find" | Ask or search catalogs/docs. Missing access is a finding, not a reason to skip. |
| "We only need a small version" | Small custom infrastructure often grows into unsupported billing, workflow, or observability systems. |

## Quick Reference

```text
"<capability>" library <language/framework>
"<capability>" SDK
"<capability>" managed service
"<capability>" gateway platform
"<capability>" vendor API
"<capability>" Temporal workflow OR queue OR scheduler
"<capability>" internal platform OR service catalog
```
