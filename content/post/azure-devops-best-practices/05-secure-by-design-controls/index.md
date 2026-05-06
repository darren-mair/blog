---
title: "5) Secure by Design: Permissions, Secrets, and Scanning"
description: Practical security controls for Azure DevOps identity, access, pipelines, and repositories.
date: 2026-05-05
lastmod: 2026-05-05
weight: 5
guide: azure-devops
build:
    list: never
    render: always
---

## Security Model Summary

Design security through layered controls:

- identity and authentication
- authorization and least privilege
- protected resources and approvals/checks
- secure coding and dependency posture
- auditability and incident response

## Identity and Authentication

Recommended order of preference:

1. Microsoft Entra ID backed identities
2. Managed identity/service principal with federation for automation
3. PAT only for constrained, temporary, low-risk scenarios

Guidance:

- Reduce PAT usage over time.
- Enforce conditional access where available.
- Use group-based identity management.

## Authorization and Permissions

### Principles

- Group-based assignment over user-level permissions.
- Prefer `Not set` over `Deny` unless explicitly blocking.
- Minimize members in Project Collection Administrators.
- Use project-level admins for project scope tasks.

### Pipeline/resource permissions

- Restrict pipeline permissions on environments and service connections.
- Do not keep open access enabled on production resources.
- Use resource ownership model (platform/security as resource owners).

## Service Connection Security

- Scope connections to resource group/service boundary, not whole subscription when avoidable.
- Prefer workload identity federation over secret-based authentication.
- Attach branch control checks to sensitive service connections.

## Secrets and Variables

- Do not store plaintext secrets in YAML.
- Prefer secretless auth (federation/managed identity).
- If secrets are required, store in Key Vault or protected variable groups.
- Rotate secrets regularly and remove unused secrets.
- Never pass secrets in command-line arguments when avoidable.

## Secure Pipeline Input Handling

- Use typed runtime parameters for constrained inputs.
- Limit queue-time variables.
- Enable shell task argument validation settings.
- Use `settableVariables` restrictions where relevant.

## Code, Dependency, and Secret Scanning

Use GitHub Advanced Security for Azure DevOps where licensed.

### Secret protection

- Enable push protection
- Enable repository secret scanning
- Monitor and triage secret alerts

### Code security

- Enable code scanning (CodeQL tasks in pipelines)
- Enable dependency scanning tasks
- Use PR annotations with build validation policies

### Rollout recommendation

1. Enable for internet-facing and critical repos first.
2. Add scanning to default branch + PR validation.
3. Expand to all active production repositories.
4. Add organization coverage/risk reporting in security overview.

## Agent and Infrastructure Security

- Prefer Microsoft-hosted agents when possible.
- Segment self-hosted pools by trust boundary.
- Run agents with low-privilege identities.
- Keep agent versions and base images patched.
- For containerized jobs, use read-only mounts and trusted images.

## Bypass and Break-Glass Governance

If bypass is needed:

- limit to named responders
- require reason text and ticket reference
- log and review within 24 hours
- remove emergency elevation immediately after incident

## Security Operations Metrics

- % repos with secret scanning enabled
- % critical repos with code + dependency scanning
- median time to remediate high/critical findings
- count of bypass events per month
- % production resources with restricted pipeline permissions

## Quick Security Checklist

- [ ] Entra-backed identities and group-based access
- [ ] PAT reduction strategy published
- [ ] Protected branches with required PR policies
- [ ] Required template checks on protected resources
- [ ] Production environment checks enforced
- [ ] Scanning baseline enabled and triage SLA defined
- [ ] Audit and exception review cadence established

## Next Step

Use the scripts/templates section to bootstrap implementation at scale.
