---
title: 4) Pipelines, Environments, and Approval Gates
description: Secure CI/CD architecture, environment checks, and deployment governance.
date: 2026-05-05
lastmod: 2026-05-05
weight: 4
guide: azure-devops
build:
  list: never
  render: always
---

## Pipeline Architecture Principles

- YAML pipelines only (disable classic creation where possible).
- Reusable templates for build, test, scan, and deployment patterns.
- Separate CI, CD, and compliance concerns by stage.
- Keep PR validation fast; run deep checks in post-merge/nightly as needed.

## Reference Stage Model

1. Validate: lint, unit tests, static checks
2. Build: package and sign artifacts
3. Security: code/dependency/secret checks
4. Deploy Non-Prod: auto-deploy with checks
5. Verify: smoke/integration checks
6. Deploy Prod: gated deployment with approvals/checks

## Environment Strategy

Use explicit environment resources:

- `dev`
- `test`
- `staging`
- `prod`

Benefits:

- deployment history per environment
- traceability of commits/work items to environment
- resource-level security and checks

## Approval and Check Baseline

Apply checks to environments and critical service connections.

Recommended production checks:

- Manual approval (separate approver group)
- Branch control (`refs/heads/main` and approved release branches)
- Business hours (if required by operations)
- Required template (for service connections/resource access)
- Optional external check (REST/Azure Function) for CAB, risk, or policy engine

## Approval Design Patterns

### Pattern A: Human gate only for production

- Non-prod: automated checks
- Prod: manual approval + branch control + template enforcement

### Pattern B: Risk-based dynamic gate

- Low-risk changes: automated progression
- High-risk changes: manual + external risk system check

### Pattern C: Compliance-heavy

- Manual approval + change ticket check + business hours + exclusive lock

## Use Exclusive Locks Carefully

If many deployments target a shared production resource, use exclusive lock with:

- `runLatest` for superseding behavior
- `sequential` when every run must deploy in order

## Pipeline Hardening Checklist

- [ ] Pipeline references approved `extends` template
- [ ] Protected resources require checks
- [ ] Service connections scoped to minimum resource set
- [ ] Job authorization scope is project-level
- [ ] Self-hosted agents segmented by trust boundary
- [ ] Production deployment requires explicit environment checks

## Minimal Deployment Job Example

```yaml
stages:
- stage: Deploy_Prod
  displayName: Deploy to production
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    displayName: Deploy web app
    environment: prod
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
          - script: echo "Deploying release $(Build.BuildNumber) to prod"
```

## Template Extension Pattern

```yaml
# azure-pipelines.yml
resources:
  repositories:
  - repository: templates
    type: git
    name: Platform/PipelineTemplates
    ref: refs/tags/v1.0.0

extends:
  template: secure-entry.yml@templates
  parameters:
    serviceName: my-service
    runIntegrationTests: true
```

Use required template checks on protected resources to enforce this model.

## Common CI/CD Anti-Patterns

- huge monolithic pipelines with no modular templates
- production checks implemented only in YAML (editable by any committer)
- service connections shared broadly across unrelated projects
- unrestricted scripts with user-provided arguments

## Next Step

Add secure-by-design controls for permissions, identity, secret handling, and scanning.
