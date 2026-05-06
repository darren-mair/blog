---
title: 1) Operating Model and Target State
description: Define the Azure DevOps operating model before tooling details.
date: 2026-05-05
lastmod: 2026-05-05
weight: 1
guide: azure-devops
build:
    list: never
    render: always
---

## Why Start Here

Most Azure DevOps issues are not tooling issues. They are operating model issues:

- unclear ownership
- inconsistent standards across teams
- weak approval and exception handling
- security controls added too late

Define the operating model first, then encode it into Boards, Repos, and Pipelines.

## Recommended Team Topology

Use a federated model:

- Product teams own backlog, code, tests, and service operation.
- Platform/DevOps team owns reusable templates, shared agents, environment controls, and secure defaults.
- Security team defines policy requirements and validates evidence.
- Architecture/engineering leadership governs standards and exceptions.

## Platform Principles

1. Golden path first: secure defaults with low-friction templates.
2. Policy as code: branch policies, required templates, and checks.
3. Least privilege everywhere: users, pipelines, service connections, agents.
4. Traceability by default: work item -> PR -> build -> deploy -> environment.
5. Exceptions are explicit, time-bound, and auditable.

## Target State Blueprint

### Portfolio and project structure

- One Azure DevOps project per product/domain or bounded context.
- Avoid large shared projects unless there is a strong reason.
- Keep environments and service connections project-scoped where possible.

### Repository structure

- One service/application per repo (preferred).
- Shared libraries in dedicated repos with their own policy.
- Infrastructure code (IaC) in separate repos with stricter deployment controls.

### Pipeline architecture

- YAML pipelines only.
- Mandatory `extends` template for all production pipelines.
- Environment-based deployments with approvals/checks at environment or service connection level.

### Governance cadence

- Weekly delivery health review (flow, failures, lead time).
- Weekly security triage (critical/high findings).
- Monthly permissions review and policy exception review.
- Quarterly maturity assessment and control uplift.

## RACI (Example)

| Control Area | Product Team | Platform Team | Security Team | Leadership |
| --- | --- | --- | --- | --- |
| Branch policies | R | A | C | I |
| Pipeline templates | C | A/R | C | I |
| Environment approvals | C | A/R | C | I |
| GHAS scanning standards | C | R | A | I |
| Access reviews | C | R | A | I |
| Exception approval | C | C | R | A |

R = Responsible, A = Accountable, C = Consulted, I = Informed.

## Non-Negotiable Baseline

Set these as mandatory for all production repos:

- PR required to merge into protected branches.
- Minimum reviewers and build validation required.
- Linked work item required for PR completion.
- YAML pipeline required; classic pipelines disabled.
- Protected environments with checks for production.
- Secret scanning enabled on all active repos.
- Code/dependency scanning enabled for internet-facing or critical services.

## Key Metrics

Track both delivery and security outcomes.

Delivery:

- Lead time for changes
- Deployment frequency
- Change failure rate
- Mean time to restore

Security/compliance:

- % repos with required branch policies
- % pipelines extending approved template
- % production deploys with approval and branch control checks
- Secret leak events detected/prevented
- Time to remediate critical findings

## Anti-Patterns to Avoid

- One project for everything, everyone admin.
- Direct pushes to main in the name of speed.
- Shared long-lived branches with no clear release strategy.
- Service connections with subscription-wide contributor access.
- Inline pipeline scripts with unrestricted runtime inputs.
- Permanent bypass permissions without monitoring.

## Next Step

Continue with Boards governance to make sure planning data quality supports traceability and reporting.
