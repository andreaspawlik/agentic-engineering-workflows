# Agentic Engineering Workflows

Reusable GitHub and VS Code workflow support for developing software one small
GitHub Issue at a time. This repository is infrastructure, not an application:
your project keeps its own source code, tests, build commands, GitHub Project,
and metrics history.

## Background

The design is loosely inspired by the [AI Codebase Maturity Model](https://arxiv.org/pdf/2604.09388), an arXiv paper describing how teams can move from basic AI assistance toward more autonomous engineering through progressively stronger instructions, tests, metrics, and feedback loops. This repository is an independent engineering implementation inspired by those ideas, not an official implementation or companion project of the paper.

The paper provides the maturity-model framing; this repository turns selected concepts into practical GitHub Issues, VS Code agents, GitHub Actions, Project status transitions, review gates, and acceptance metrics.

## What You Get

The workflow supports this path:

```text
Issue -> Architect -> Developer -> CI -> Reviewer -> merge -> metrics
```

It provides:

- Architect, Developer, and Reviewer agents for VS Code;
- issue and pull-request templates;
- coordinator, CI reconciliation, merge-gate, backlog-sync, and acceptance
  metrics workflows;
- project-specific configuration through `agentic-project.json`; and
- versioned reusable GitHub Actions releases.

## Choose An Integration Mode

### Fastest: copy the templates

Copy the contents of `templates/` into a new project. This is the simplest
starting point and is useful while experimenting. Replace the example contract
with the new project's values.

### Recommended: call versioned workflows

Keep application-specific files in the consumer repository and use thin local
workflow wrappers that call releases from this repository. The current releases
are:

| Capability | Release |
| --- | --- |
| CI | `v1.1.1` |
| Acceptance Metrics | `v1.2.0` |
| Coordinator | `v1.3.0` |
| PR/CI reconciliation | `v1.4.0` |
| Merge gate | `v1.5.0` |
| Backlog sync | `v1.6.0` |

Pin release tags or commit SHAs. Do not call `main` in production.

## New Project Setup

Follow the common prerequisites, choose exactly one integration path, then
complete the shared Project and first-issue steps.

### 1. Common prerequisites

Your consumer repository needs:

- application source code;
- automated tests;
- a build or package file;
- install, test, and coverage commands; and
- a default branch named `main`.

Create a user-owned GitHub Project with a Status field containing Todo, In
Progress, and Done. Create the repository backlog label you plan to use, such
as `work-item`, and create the matching category labels used by your project.

### 2A. Path A: copy the templates

Use this path when you want the quickest setup or need to customize the
workflow files locally. Copy the contents of `templates/` into the consumer
repository:

```text
templates/.github/       -> .github/
templates/scripts/       -> scripts/
templates/docs/          -> docs/
templates/agentic-project.example.json -> agentic-project.json
templates/metrics/       -> metrics/
```

Keep the copied workflow implementations locally. Update the issue-template
`labels:` value to your backlog label. This path makes local workflow edits
simple, but fixes and upgrades must be copied into each consumer.

### 2B. Path B: call versioned workflows

Use this path when you want workflow fixes and upgrades to come from this
repository. Copy only the consumer-local support files:

```text
templates/.github/agents/
templates/.github/copilot-instructions.md
templates/.github/ISSUE_TEMPLATE/
templates/.github/pull_request_template.md
templates/scripts/project_config.py
templates/docs/
templates/agentic-project.example.json -> agentic-project.json
```

Then create thin workflow wrappers in `.github/workflows/` that call the
versioned releases listed above. For example:

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

Use the corresponding release for each other workflow. Use `secrets: inherit`
for workflows that write Project or metrics data. Pin a release tag or commit
SHA; do not call `main` in production.

### 3. Configure `agentic-project.json`

Copy `templates/agentic-project.example.json` and replace every placeholder:

```json
{
  "project": {
    "name": "your-project",
    "language": "python",
    "install_command": "pip install -e \".[dev]\"",
    "test_command": "pytest -q",
    "coverage_command": "pytest --cov=your_package --cov-report=xml -q",
    "coverage_threshold": 90
  },
  "backlog": {
    "label": "work-item",
    "project_name": "Your Backlog"
  },
  "github": {
    "project_owner": "YOUR_GITHUB_USER",
    "project_number": 2,
    "project_id": "PVT_...",
    "status_field_id": "PVTSSF_...",
    "status_options": {
      "todo": "...",
      "in_progress": "...",
      "done": "..."
    }
  }
}
```

Use the exact IDs from your user-owned GitHub Project. Validate locally:

```bash
python -m json.tool agentic-project.json
python scripts/project_config.py
python scripts/project_config.py run project.test_command
python scripts/project_config.py run project.coverage_command
```

### 4. Configure GitHub in the UI

In the consumer repository:

1. Set the issue-template `labels:` value to the configured backlog label.
2. Add repository secret `PROJECT_TOKEN` under **Settings -> Secrets and
   variables -> Actions**. The token needs write access to the user-owned
   Project.
3. Enable Actions for the repository.

Create exactly one primary category label for each issue. The issue body and
GitHub label must agree, for example:

```text
Primary category: architecture
Labels: work-item, architecture
```

### 5. Create and run the first issue

1. Create an issue using the backlog template.
2. Set its primary category in the body.
3. Apply the configured backlog label and matching category label.
4. Add it to the Project and set Status to Todo.
5. Open **Actions -> Level 5 coordinator -> Run workflow**.
6. Choose branch `main` and enter the issue number, repair limit `2`, and a
   unique run ID such as `project-issue-1-001`.
7. Confirm the issue moves Todo -> In Progress.

### 6. Run the agent workflow

In VS Code, open the consumer repository and select the requested agent:

Architect:

```text
Prepare GitHub issue #N for implementation. Validate its scope, dependencies,
acceptance criteria, category, project membership, and edge cases. Record the
validation transition automatically on issue #N.
```

Developer:

```text
Implement GitHub issue #N, run the configured tests, and open a pull request.
Include Closes #N in the pull request description.
```

Reviewer:

```text
Review pull request #PR_NUMBER against GitHub issue #N and report any blocking findings.
```

Architect validation, PR creation, CI success, and review approval update the
coordinator state. Reviewer also checks that the PR category label matches the
issue category; a missing label is a minor metrics finding, not a code blocker.

### 7. Merge and verify

After CI and the merge gate pass, merge the PR manually. Then verify:

- the issue is closed;
- Project status is Done;
- Acceptance Metrics completed successfully;
- `metrics/acceptance-rate.json` contains the PR with its category; and
- the feedback report was printed.

## Troubleshooting

- **Issue is not added to the Project:** check the exact backlog label,
  `PROJECT_TOKEN`, Project ID, and Status field ID. Re-run the issue label event
  after correcting configuration.
- **Coordinator cannot select an issue:** confirm the issue is open, has the
  configured backlog label, has acceptance criteria, and is not an epic.
- **Merge gate fails:** inspect coordinator state. The required sequence is
  Architect validated, Developer PR recorded, CI passed, Reviewer approved, and
  `next_action: merge`.
- **Metrics say `uncategorized`:** add the issue's primary category label to
  the PR before it closes.
- **Reusable workflow cannot start:** check that the consumer wrapper uses a
  published release tag and that its permissions match the workflow's needs.

## Versioning

This repository publishes versioned releases so consumers can upgrade
intentionally. Start with the releases listed above, validate in a non-critical
consumer, then upgrade one release at a time.
