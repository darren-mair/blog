---
title: 6) Implementation Scripts and Templates
description: Starter scripts and YAML templates to accelerate Azure DevOps standardization.
date: 2026-05-05
lastmod: 2026-05-05
weight: 6
guide: azure-devops
build:
  list: never
  render: always
---

These examples are implementation starters. Validate in a non-production project first.

## Prerequisites

- Azure CLI with Azure DevOps extension
- Appropriate Azure DevOps permissions
- Standardized naming for projects/repos/branches

```powershell
az extension add --name azure-devops
az devops configure --defaults organization=https://dev.azure.com/<org> project=<project>
```

## Script: Create Core Branch Policies (PowerShell + Azure CLI)

```powershell
param(
    [Parameter(Mandatory = $true)]
    [string]$RepositoryId,

    [Parameter(Mandatory = $false)]
    [string]$BranchName = "main",

    [Parameter(Mandatory = $false)]
    [int]$MinimumReviewers = 2,

    [Parameter(Mandatory = $false)]
    [string]$BuildDefinitionId
)

$branchRef = "refs/heads/$BranchName"

# Minimum reviewers
az repos policy approver-count create `
  --repository-id $RepositoryId `
  --branch $branchRef `
  --minimum-approver-count $MinimumReviewers `
  --creator-vote-counts false `
  --allow-downvotes false `
  --blocking true `
  --enabled true

# Linked work items required
az repos policy work-item-linking create `
  --repository-id $RepositoryId `
  --branch $branchRef `
  --blocking true `
  --enabled true

# Comment resolution required
az repos policy comment-required create `
  --repository-id $RepositoryId `
  --branch $branchRef `
  --blocking true `
  --enabled true

if ($BuildDefinitionId) {
  # Build validation
  az repos policy build create `
    --repository-id $RepositoryId `
    --branch $branchRef `
    --build-definition-id $BuildDefinitionId `
    --display-name "PR Validation" `
    --queue-on-source-update-only true `
    --manual-queue-only false `
    --blocking true `
    --enabled true
}
```

## Script: Policy-As-Code via Configuration File

Use a source-controlled policy configuration and apply it repeatedly.

```json
{
  "isEnabled": true,
  "isBlocking": true,
  "type": {
    "id": "fa4e907d-c16b-4a4c-9dfa-4906e5d171dd"
  },
  "settings": {
    "minimumApproverCount": 2,
    "creatorVoteCounts": false,
    "scope": [
      {
        "repositoryId": "<repo-guid>",
        "refName": "refs/heads/main",
        "matchKind": "exact"
      }
    ]
  }
}
```

```powershell
az repos policy create --config policy.minimum-reviewers.json
```

## Template: Secure Pipeline Entry (`extends`)

```yaml
# secure-entry.yml
parameters:
- name: serviceName
  type: string
- name: vmImage
  type: string
  default: ubuntu-latest
  values:
  - ubuntu-latest
  - windows-latest
- name: runIntegrationTests
  type: boolean
  default: false

stages:
- stage: Validate
  jobs:
  - job: Validate
    pool:
      vmImage: ${{ parameters.vmImage }}
    steps:
    - checkout: self
    - script: echo "Validate ${{ parameters.serviceName }}"

- stage: Build
  dependsOn: Validate
  jobs:
  - job: Build
    steps:
    - script: echo "Build artifact"

- ${{ if eq(parameters.runIntegrationTests, true) }}:
  - stage: IntegrationTests
    dependsOn: Build
    jobs:
    - job: Integration
      steps:
      - script: echo "Run integration tests"
```

## Template: CodeQL Scanning Starter

```yaml
# codeql-scan.yml
steps:
- task: AdvancedSecurity-Codeql-Init@1
  inputs:
    languages: 'csharp,javascript,python'

# Add build steps for compiled languages here

- task: AdvancedSecurity-Codeql-Analyze@1
```

## Template: Dependency Scanning Starter

```yaml
# dependency-scan.yml
steps:
- task: AdvancedSecurity-Dependency-Scanning@1
```

## Example: Deployment with Environment Gate

```yaml
stages:
- stage: DeployProd
  displayName: Deploy to Prod
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: prod
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to prod"
```

Configure approvals/checks in the environment UI:

- Approvals
- Branch control
- Required template
- Business hours (optional)
- REST/Azure Function check (optional)

## Script: Find Repositories Missing Branch Policy Baseline

```powershell
param(
    [Parameter(Mandatory = $true)]
    [string]$ProjectName
)

$repos = az repos list --project $ProjectName --output json | ConvertFrom-Json

foreach ($repo in $repos) {
    $policies = az repos policy list --repository-id $repo.id --output json | ConvertFrom-Json

    $hasMinReviewers = $policies | Where-Object { $_.type.displayName -eq "Minimum number of reviewers" }
    $hasBuild = $policies | Where-Object { $_.type.displayName -eq "Build" }
    $hasLinkedWorkItems = $policies | Where-Object { $_.type.displayName -eq "Work item linking" }

    if (-not $hasMinReviewers -or -not $hasBuild -or -not $hasLinkedWorkItems) {
        Write-Output "NON-COMPLIANT: $($repo.name)"
    }
}
```

## Script: Enumerate Projects and Group Membership (Audit Starter)

```powershell
# List projects
az devops project list --output table

# List users in organization
az devops user list --output table

# List security groups (graph)
az devops security group list --output table
```

## Implementation Notes

- Keep scripts in a dedicated platform repo and version them.
- Treat policy scripts as idempotent and rerunnable.
- Add CI checks that fail when required templates are not referenced.
- Add reporting dashboards for branch policy and scanning coverage.
