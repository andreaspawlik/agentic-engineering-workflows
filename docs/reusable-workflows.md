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
    uses: andreaspawlik/agentic-engineering-workflows/.github/workflows/ci.yml@v1.1.0
    with:
      workflow_ref: v1.1.0
    permissions:
      contents: read
```

The reusable CI workflow checks out both the caller repository and the pinned
workflow repository. It reads the caller's `agentic-project.json` and executes
its configured install and coverage commands. The caller keeps its source,
tests, package metadata, and project contract.

## Inputs And Permissions

Pin a release tag or commit instead of `main`. Consumers should grant only the
permissions required by the workflow and keep `PROJECT_TOKEN` in the consumer
repository's secrets when Project writes are needed.

The remaining coordinator, reconciliation, merge-gate, backlog-sync, and
acceptance workflows will migrate to the same pattern in subsequent slices.
