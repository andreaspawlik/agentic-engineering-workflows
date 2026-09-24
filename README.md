# Agentic Engineering Workflows

Reusable GitHub and VS Code workflow support for developing software one small
GitHub Issue at a time. This repository is infrastructure, not an application:
your project keeps its own source code, tests, build commands, GitHub Project,
and metrics history.

## Background

The design is loosely inspired by the [AI Codebase Maturity Model](https://arxiv.org/pdf/2604.09388), an arXiv paper describing how teams can move from basic AI assistance toward more autonomous engineering through progressively stronger instructions, tests, metrics, and feedback loops. This repository is an independent engineering implementation inspired by those ideas, not an official implementation or companion project of the paper.

The paper provides the maturity-model framing; this repository turns selected concepts into practical GitHub Issues, VS Code agents, GitHub Actions, Project status transitions, review gates, and acceptance metrics.

## Transparency

The workflow code, templates, documentation, and supporting automation in this repository were developed with assistance from [GitHub Copilot](https://github.com/features/copilot). This repository is provided as-is; anyone is free to use it, but at their own responsibility. Users are responsible for reviewing, testing, and validating the design decisions, configuration, and releases before adopting them in their own projects. Copilot assistance is disclosed here so users can understand how this generative-AI-supported engineering workflow was created.

## What You Get

The workflow supports this path:

```text
Issue -> Architect -> Developer -> CI -> Reviewer -> merge -> metrics -> experiment -> guidance -> next issue
```

It provides:

- Architect, Developer, and Reviewer agents for VS Code;
- issue and pull-request templates;
- coordinator, CI reconciliation, merge-gate, backlog-sync, and acceptance
  metrics workflows;
- project-specific configuration through `agentic-project.json`; and
- versioned reusable GitHub Actions releases.

## Automation Maturity

This repository is a **bounded, human-supervised Level 5-style workflow**, not a fully autonomous coding system. The surrounding engineering loop is automated, while agent invocation and high-impact decisions remain human-controlled.

After each closed PR, the Acceptance Metrics workflow records the outcome and prints feedback. A maintainer then reviews the metrics, updates or creates an experiment in `metrics/experiments.json` when the result reveals a reusable lesson, and applies the resulting instruction, test, or workflow improvement to future issues. This closes the learning loop.

| Workflow step | Current state |
| --- | --- |
| Issue selection and Project status | Automated by coordinator and Project workflows |
| Architect validation | Agent-assisted; a human starts the Architect and the agent records the transition |
| Developer implementation | Human starts the Developer agent; the agent changes code, tests, and opens a PR |
| CI and coverage | Automated through reusable workflows |
| PR and CI state reconciliation | Automated by commit/PR matching |
| Code review | Human starts Reviewer; findings are reported, not silently applied |
| Category-label consistency | Reviewer checks it as a minor metrics finding |
| Merge decision | Human-controlled after the merge gate passes |
| Acceptance metrics and Project Done status | Automated after PR closure |

The next maturity step is authenticated agent orchestration: a trusted service
would invoke Architect, Developer, and Reviewer with scoped credentials, persist
agent outputs, enforce approvals, and safely resume or stop runs. Fully
autonomous code development would additionally require reliable agent execution,
workspace isolation, secret and permission boundaries, test and review policies,
rollback/escalation behavior, audit logs, and an explicit policy for when
merging is allowed without a human. Those capabilities are not implemented
here.

## Workflow Reference

The reusable workflows are the GitHub automation layer. A consumer usually has
a thin wrapper for each workflow under `.github/workflows/`. The wrappers define
consumer triggers; the reusable workflows execute the shared implementation.

| Workflow file | Trigger | Responsibility | Main consumer inputs |
| --- | --- | --- | --- |
| `ci.yml` | Pull request and push to `main` | Checks out the consumer, runs its configured install and coverage commands, and uploads coverage | `agentic-project.json` |
| `acceptance-metrics.yml` | Closed pull request | Records acceptance outcome, prints feedback, commits metrics, and marks merged issues Done | `agentic-project.json`, `PROJECT_TOKEN` when Project writes are needed |
| `level-5-coordinator.yml` | Manual workflow dispatch | Selects an eligible issue, creates coordinator state, moves it to In Progress, and posts the Architect handoff | Issue number, repair limit, run ID, `agentic-project.json`, `PROJECT_TOKEN` |
| `level-5-pr-ci-reconciliation.yml` | Pull request and successful CI workflow-run events | Records Developer PR creation and CI success in coordinator state | Linked issue and coordinator comments |
| `level-5-merge-gate.yml` | Pull request, review, and CI completion events | Evaluates whether coordinated PR state is eligible to merge | Linked issue state and PR number |
| `project-backlog-sync.yml` | Issue opened, reopened, or labeled | Adds issues with the configured backlog label to the consumer Project | `agentic-project.json`, `PROJECT_TOKEN` |

The copy-mode template also includes
`templates/.github/workflows/level-5-coordinator-continue.yml`. It handles
trusted issue comments such as coordinator transitions. Consumers using the
versioned workflow path should keep an equivalent local continuation wrapper
until that event-driven entrypoint is released as a reusable workflow.

The normal lifecycle is:

```text
backlog sync -> coordinator -> Architect -> Developer PR -> CI reconciliation
-> Reviewer -> merge gate -> manual merge -> acceptance metrics -> experiment loop
```

See [docs/reusable-workflows.md](docs/reusable-workflows.md) for wrapper
examples, inputs, permissions, secrets, and release pinning.

## Adopt This In Your Project

If you want to use this workflow in a new or existing project, set up the
consumer repository first, then run the delivery loop. The consumer repository
keeps its own source code, tests, `agentic-project.json`, GitHub Project,
secrets, and metrics history. This repository supplies the reusable agents,
templates, scripts, and workflow implementations.

### 1. Prepare the consumer application

Your consumer repository needs:

- application source code;
- automated tests;
- a build or package file;
- install, test, and coverage commands; and
- a default branch named `main`.

Create a user-owned GitHub Project with a Status field containing Todo, In
Progress, and Done. Create the repository backlog label you plan to use, such
as `work-item`, and create the matching category labels used by your project.

### 2. Choose one setup path

#### Path A: copy the templates

Use this when you want the fastest setup or want to customize workflow files
inside the consumer repository. Copy the full template tree:

```text
templates/.github/       -> .github/
templates/scripts/       -> scripts/
templates/docs/          -> docs/
templates/agentic-project.example.json -> agentic-project.json
templates/metrics/       -> metrics/
```

This path is simple, but future fixes and upgrades must be copied into every
consumer repository.

#### Path B: call versioned reusable workflows

Use this when you want workflow fixes and upgrades to come from this repository.
Copy only the consumer-local support files:

```text
templates/.github/agents/
templates/.github/copilot-instructions.md
templates/.github/ISSUE_TEMPLATE/
templates/.github/pull_request_template.md
templates/scripts/project_config.py
templates/docs/
templates/agentic-project.example.json -> agentic-project.json
templates/metrics/
```

Then create thin wrappers in `.github/workflows/` that call pinned releases from
this repository. Example CI wrapper:

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

Use the release map below for other wrappers. Use `secrets: inherit` for
workflows that write Project or metrics data. Pin release tags or commit SHAs;
do not call `main` in production.

| Capability | Release |
| --- | --- |
| CI | `v1.1.1` |
| Acceptance Metrics | `v1.2.0` |
| Coordinator | `v1.3.0` |
| PR/CI reconciliation | `v1.4.0` |
| Merge gate | `v1.5.0` |
| Backlog sync | `v1.6.0` |

### 3. Configure `agentic-project.json`

If you are starting a new project, use GitHub Copilot in VS Code to walk through
this step. Open the consumer repository, select Agent mode, and use:

```text
Inspect this project and help configure agentic-project.json. Infer the project
name, language, install command, test command, coverage command, coverage
threshold, and backlog label from the repository. Validate every inferred
command locally. Do not invent GitHub Project IDs: identify which values require
a Project to exist, show me how to retrieve them, and update the file only after
verifying the live GitHub values.
```

Copilot can usually infer and test the application commands from files such as
`pyproject.toml`, `package.json`, or existing CI setup. GitHub generates the
Project number, Project ID, Status field ID, and status option IDs after the
Project exists. Never copy those IDs from another consumer repository.

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

Validate locally:

```bash
python -m json.tool agentic-project.json
python scripts/project_config.py
python scripts/project_config.py run project.test_command
python scripts/project_config.py run project.coverage_command
```

### 4. Configure GitHub

In the consumer repository:

1. Set the issue-template `labels:` value to the configured backlog label.
2. Add repository secret `PROJECT_TOKEN` under **Settings -> Secrets and
   variables -> Actions**. The token needs write access to the user-owned
   Project.
3. Enable Actions for the repository.
4. Create exactly one primary category label for each issue category.

The issue body and GitHub label must agree, for example:

```text
Primary category: architecture
Labels: work-item, architecture
```

At this point setup is complete. Continue with the delivery loop below for each
new issue.

## Run The Delivery And Learning Loop

Project setup is a one-time activity. The following loop is repeated for each
small backlog issue after setup is complete.

```text
prepare issue -> deliver change -> merge and measure -> experiment and improve -> next issue
```

### 1. Prepare and start an issue

1. Create an issue using the backlog template.
2. Set exactly one primary category in the issue body.
3. Apply the configured backlog label and the matching category label.
4. Add the issue to the configured Project and set Status to Todo.
5. Open **Actions -> Level 5 coordinator -> Run workflow**.
6. Choose branch `main` and enter the issue number, repair limit `2`, and a
   unique run ID such as `project-issue-1-001`.
7. Confirm the issue moves Todo -> In Progress.

### 2. Deliver the change

In VS Code, open the consumer repository and select the requested agent.

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

#### Single-maintainer review

GitHub does not allow a pull-request author to approve their own PR. A token or
repository setting cannot turn self-review into a genuine GitHub approval. For
single-maintainer and demonstration repositories, use one of these practical
policies:

1. **Explicit coordinator self-review.** Run the Reviewer agent, assess its
   findings, and record the repository's documented `self_reviewed` coordinator
   transition. This records `review_source: self` and may allow the merge gate to
   pass, but it must not be represented as an independent GitHub approval.
2. **CI plus merge gate without required GitHub approval.** Do not require a
   GitHub approval in branch protection. Require CI and the Level 5 merge gate,
   run Reviewer as an advisory check, and keep the final merge manual. This is
   the recommended policy for a single-maintainer consumer.

Production repositories should prefer an independent human reviewer or a
separately authenticated GitHub App operating under an explicit review policy.

### 3. Merge and measure

After CI and the merge gate pass, merge the PR manually. Then verify:

- the issue is closed;
- Project status is Done;
- Acceptance Metrics completed successfully;
- `metrics/acceptance-rate.json` contains the PR with its category; and
- the engineering feedback report was printed.

### 4. Experiment and improve

Metrics collection is automatic; learning is currently human-controlled. After
each closed PR:

1. Review the Acceptance Metrics and engineering feedback workflow output.
2. Compare the result with active entries in `metrics/experiments.json`.
3. Add the PR as follow-up evidence when it tests an existing experiment.
4. Create a new experiment only when a recurring failure or weak outcome leads
   to a measurable instruction, test, or workflow change.
5. Record the baseline, proposed change, success criteria, observation period,
   and result.
6. Keep, revise, or revert the change based on evidence.
7. Apply the resulting guidance, test, or workflow improvement to the next
   issue.

Use this prompt with GitHub Copilot when reviewing the loop:

```text
Review metrics/acceptance-rate.json and metrics/experiments.json after the latest
closed PR. Identify whether it provides evidence for an existing experiment or
a recurring failure worth a new experiment. Do not create an experiment from a
single isolated result without explaining the hypothesis, baseline, measurable
success criteria, and future comparison.
```

The loop is complete only when delivery evidence has either informed an
experiment or been explicitly judged not to require one.

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

## Maintainer Handoff

See [docs/maintainer-handoff.md](docs/maintainer-handoff.md) for the current release map, validation commands, safe change procedure, consumer boundaries, and next work.

## Versioning

This repository publishes versioned releases so consumers can upgrade
intentionally. Start with the releases listed above, validate in a non-critical
consumer, then upgrade one release at a time.
