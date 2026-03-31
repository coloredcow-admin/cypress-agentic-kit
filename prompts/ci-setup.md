# CI Setup Prompt — Set Up Cypress CI Workflow

Paste this prompt into a Claude session inside your project to set up a CI workflow that runs Cypress tests on pull requests.

**Prerequisite:** Cypress must be installed and at least one test file must exist. Run `prompts/bootstrap.md` and `prompts/test-writer.md` first if needed.

---

## Prompt

```
I want to set up CI for Cypress in this project using the Cypress Agentic Kit.

Please do the following:

1. Read `knowledge-base/00-master.md` to understand the knowledge base structure.

2. Assume the role of `cypress-ci-agent` by reading `agents/cypress-ci-agent.md`.

3. Execute the CI integration runbook from `knowledge-base/04-ci-integration.md`:
   - Run pre-flight checks (Cypress installed, tests exist, detect CI platform)
   - Detect my package manager, dev server command, and base URL
   - Create the CI workflow file triggered by the "ready to test" label
   - Cross-check that the "ready to test" label exists in the repository

4. When creating the workflow:
   - Adapt it for my project's package manager and dev server
   - Include artifact uploads for screenshots and videos on failure
   - Confirm any values you're not sure about before writing the file

Important:
- Do not create a workflow if no Cypress tests exist — tell me to write tests first.
- Do not overwrite existing CI workflow files without asking me.
- Do not guess the dev server command or base URL — ask me if unclear.
- Show me every command before running it.
- Give me a final report at the end.
```

---

## What Happens When You Run This

1. The agent checks that Cypress is installed and test files exist.
2. It detects your CI platform, package manager, dev server command, and base URL.
3. It creates a workflow file (e.g., `.github/workflows/cypress.yml`) that:
   - Triggers when the `ready to test` label is added to a PR
   - Installs dependencies and starts the dev server
   - Runs the full Cypress test suite
   - Uploads screenshots and videos as artifacts on failure
4. It checks that the `ready to test` label exists in the repository.
5. You get a final report with the setup status.

## How to Use After Setup

1. Push your branch and create a pull request.
2. When you're ready to run tests, add the `ready to test` label to the PR.
3. The CI workflow runs Cypress automatically.
4. Check the Actions tab on GitHub for pass/fail results.
5. On failure, download screenshots and videos from the workflow artifacts.

## Next Steps

- **Need to add environment variables?** Add them as GitHub repository secrets and reference them in the workflow file.
- **Want to update the kit?** Run `prompts/sync.md` to pull the latest files.
