# Maintainer Handoff

Use this guide when returning after a break or joining with a new coding agent.

## Purpose

This repository owns reusable agentic engineering guidance and versioned GitHub Actions workflows. It is not an application repository. Consumer repositories own source code, tests, project configuration, and metrics history.

## Automation Boundary

The current system is human-supervised. GitHub automates issue/project status,
CI, PR/CI reconciliation, merge-gate evaluation, backlog synchronization, and
acceptance metrics. Humans still start the Architect, Developer, and Reviewer
agents, assess review findings, approve the coordinator transition, and merge.

The next level requires an authenticated agent runner with scoped credentials,
isolated workspaces, durable agent outputs, policy checks, escalation and
rollback behavior, and audit logs. Fully autonomous coding additionally needs a
well-defined no-human-merge policy and strong controls around secrets, tests,
review, and failure recovery. Do not describe the current system as fully
autonomous.

## Current Releases

| Capability | Release |
| --- | --- |
| CI | v1.1.1 |
| Acceptance Metrics | v1.2.0 |
| Coordinator | v1.3.0 |
| PR/CI reconciliation | v1.4.0 |
| Merge gate | v1.5.0 |
| Backlog sync | v1.6.0 |

Greeter is the reference consumer and has completed an end-to-end run using all six versioned wrappers.

## Learning Loop

The workflow does not end at merge. After a PR closes, Acceptance Metrics records the outcome and prints feedback. A maintainer reviews that evidence, updates or creates an experiment in `metrics/experiments.json` when a reusable lesson emerges, and applies the resulting guidance, test, or workflow change to subsequent issues.

The loop is:

```text
metrics -> experiment -> guidance/test/workflow change -> next issue
```

Experiment creation and interpretation are intentionally human-controlled in the current system.

## Start Here

1. Read README.md and docs/project-contract.md.
2. Inspect git status and preserve unrelated .vscode changes.
3. Check the consumer repository contract and wrapper release versions.
4. Validate workflow YAML and the consumer configured commands.
5. Change one workflow slice at a time.
6. Publish a release when reusable workflow behavior changes.
7. Upgrade one consumer and run an integration check before the next migration.

## Validation

For workflow YAML:

```bash
python - <<'PY'
from pathlib import Path
import yaml
for path in Path(".github/workflows").glob("*.yml"):
    yaml.safe_load(path.read_text())
print("workflow YAML valid")
PY
git diff --check
```

For a consumer, run the commands configured in its agentic-project.json. Do not assume Calculator package names or commands.

## Boundaries

Keep consumer behavior and Project IDs in consumer repositories. Keep reusable workflow logic here. Pin consumers to release tags or commit SHAs, not main.

## Next Work

The reusable workflow extraction and first consumer migration are complete. The next useful work is onboarding a third consumer, improving reusable workflow tests and release automation, or adding a workflow capability required by a consumer.
