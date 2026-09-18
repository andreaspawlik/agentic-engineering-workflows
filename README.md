# Agentic Engineering Workflows

Reusable agentic engineering guidance and GitHub Actions templates for
issue-driven development. Consumer repositories provide their own application
code, tests, Project configuration, and metrics history.

## What This Repository Provides

The reusable template tree contains:

- role-focused Architect, Developer, and Reviewer agents;
- issue and pull-request templates;
- Level 5 coordinator, reconciliation, merge-gate, and metrics workflows;
- coordinator and metrics support scripts; and
- the project adapter contract documentation.

## Onboard A Consumer

1. Copy `templates/.github`, `templates/scripts`, and `templates/docs` into the
   consumer repository.
2. Copy `templates/agentic-project.example.json` to
   `agentic-project.json`.
3. Set the consumer's install, test, coverage, backlog, and GitHub Project
   values in `agentic-project.json`.
4. Replace the issue-template `labels` value with the consumer backlog label.
5. Add `PROJECT_TOKEN` as a repository secret with Project write access.
6. Run the configured commands before pushing workflow changes:

   ```bash
   python scripts/project_config.py
   python scripts/project_config.py run project.test_command
   python scripts/project_config.py run project.coverage_command
   ```

7. Create a labeled issue, add it to the consumer Project as Todo, and run the
   Level 5 coordinator from the Actions UI.
8. Verify Todo -> In Progress -> Done, PR/CI reconciliation, merge-gate
   evaluation, and acceptance metrics after the first merged PR.

The detailed consumer checklist is documented in the source project's README
and `templates/docs/project-contract.md`.

## Versioning

This repository is the extraction point for reusable workflow assets. The first
consumer migration is Greeter. Tag a release after the template tree is
validated, then pin future consumers to that tag or commit instead of copying
from a moving branch.
