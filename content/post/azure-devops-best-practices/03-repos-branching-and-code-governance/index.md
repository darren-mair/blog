---
title: 3) Repos, Branching, and Code Governance
description: Branching models, PR controls, and repository policies for Azure Repos.
date: 2026-05-05
lastmod: 2026-05-05
weight: 3
guide: azure-devops
build:
    list: never
    render: always
---

## Branching Strategy Choices

Prefer simple strategies unless your release model requires complexity.

### Recommended default (most teams)

- Main branch as stable integration branch
- Short-lived feature branches
- Pull request merge only
- Release branches only when needed for parallel support windows

### When to add release branches

Use release branches if:

- you support multiple versions simultaneously
- you need staged hardening before release
- hotfixes must be isolated from ongoing feature flow

Use `cherry-pick` to port fixes between release and mainline to avoid accidental drift.

## Branch Naming Convention

Use explicit naming for traceability:

- `feature/<workitem>-<short-description>`
- `bugfix/<workitem>-<short-description>`
- `hotfix/<release>-<short-description>`
- `release/<yyyy.mm>` or `release/<major.minor>`

## Protected Branch Baseline

Apply to `main` and active `release/*` branches:

- Require PR for merge
- Minimum 2 reviewers (or 1 for small teams)
- Prohibit most recent pusher approval for critical repos
- Require linked work items
- Require comment resolution
- Require build validation
- Restrict merge types (usually squash or rebase+ff)

## Pull Request Standard

Each PR should include:

- linked work item(s)
- problem statement and scope
- test evidence
- rollback considerations for high-risk changes
- security notes when changing auth, secrets, data access, or networking

## CODEOWNERS and Review Coverage

Use required reviewers by path (or equivalent reviewer policies) for:

- security-sensitive directories
- infrastructure and IaC
- pipeline templates
- shared libraries

## Repository Permissions Baseline

- Remove broad contributor/admin assignments where not needed.
- Avoid granting bypass permissions except emergency responders.
- Scope bypass permissions to specific branch/repo and monitor every use.

## Merge Strategy Guidance

- Squash merge: clean history, easiest default for product repos.
- Rebase/FF: clean linear history, useful for disciplined teams.
- Merge commit: use when preserving branch context is important.

Consistency matters more than preference. Standardize by repo category.

## Example Policy Matrix

| Repo Type | Reviewers | Build Validation | Bypass Allowed | Merge Type |
| --- | --- | --- | --- | --- |
| Product service | 2 | Required | Emergency only | Squash |
| Shared platform lib | 2 + path owners | Required | Emergency only | Rebase/FF |
| IaC production | 2 + security review | Required + policy checks | Named responders only | Squash |
| Experiment/sandbox | 1 | Optional | Team lead | Squash |

## Anti-Patterns

- Long-lived feature branches with no rebase cadence
- Direct commits to protected branches
- Optional build validation on critical repositories
- Excessive bypass rights inherited at project level

## Next Step

Implement pipelines and environment gates that enforce the same controls at deployment time.
