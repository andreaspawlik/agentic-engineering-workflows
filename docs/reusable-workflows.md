# Reusable Workflows

Consumers call versioned workflows from this repository through thin local
wrappers. The wrapper owns the consumer trigger; the reusable workflow owns the
implementation.

## CI Example

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    uses: andreaspawlik/agentic-engineering-workflows/.github/workflows/ci.yml@v1.1.1
    with:
      workflow_ref: v1.1.1
    permissions:
      contents: read
```

The reusable CI workflow checks out both the caller repository and the pinned
workflow repository. It reads the caller's `agentic-project.json` and executes
its configured install and coverage commands. The caller keeps its source,
tests, package metadata, and project contract.

## Inputs And Permissions

## Acceptance Metrics Example

```yaml
name: Acceptance metrics

on:
  pull_request:
    types: [closed]

jobs:
  record:
    uses: andreaspawlik/agentic-engineering-workflows/.github/workflows/acceptance-metrics.yml@v1.2.0
    with:
      workflow_ref: v1.2.0
    secrets: inherit
    permissions:
      contents: write
      issues: read
      repository-projects: write
```

This workflow writes acceptance metrics in the caller repository and uses the
caller's `agentic-project.json` to mark merged issues Done.

## Coordinator Example

```yaml
name: Level 5 coordinator

on:
  workflow_dispatch:
    inputs:
      issue_number:
        required: false
        type: string
      max_repair_iterations:
        required: false
        default: "2"
        type: string
      run_id:
        required: false
        type: string

jobs:
  coordinate:
    uses: andreaspawlik/agentic-engineering-workflows/.github/workflows/level-5-coordinator.yml@v1.3.0
    with:
      workflow_ref: v1.3.0
      issue_number: ${{ inputs.issue_number }}
      max_repair_iterations: ${{ inputs.max_repair_iterations }}
      run_id: ${{ inputs.run_id }}
    secrets: inherit
    permissions:
      contents: read
      issues: write
      pull-requests: read
      repository-projects: write
```

The reusable coordinator checks out the caller repository, reads its project
contract, updates its configured Project status, and posts the initial state
comment. `PROJECT_TOKEN` remains a secret in the caller repository.

Pin a release tag or commit instead of `main`. Consumers should grant only the
permissions required by the workflow and keep `PROJECT_TOKEN` in the consumer
repository's secrets when Project writes are needed.

The coordinator, reconciliation, merge-gate, backlog-sync, and Acceptance Metrics workflows are now available as versioned reusable entrypoints. See `README.md` or `docs/maintainer-handoff.md` for the current release map.
