# Section 04 — CI Integration

This section covers setting up a CI workflow that runs Cypress tests on a pull request when a specific label is added.

---

## 1. Pre-flight

Before creating any CI workflow, verify the following:

### 1.1 Cypress is installed

```bash
npx cypress --version
```

- If Cypress is not installed → stop. Direct the user to run the setup agent first (`prompts/bootstrap.md`).

### 1.2 Tests exist

Check that at least one test file exists:

```
cypress/e2e/**/*.cy.{js,ts}
```

- If no test files exist → stop. Direct the user to write tests first (`prompts/test-writer.md`).

### 1.3 Detect CI platform

Check the project root for existing CI configuration:

| File / Directory | CI Platform |
|------------------|-------------|
| `.github/workflows/` | GitHub Actions |
| `.gitlab-ci.yml` | GitLab CI |
| `Jenkinsfile` | Jenkins |
| `.circleci/config.yml` | CircleCI |
| `bitbucket-pipelines.yml` | Bitbucket Pipelines |
| `azure-pipelines.yml` | Azure DevOps |

- If a CI platform is detected → use that platform.
- If multiple are detected → ask the user which one to use.
- If none is detected → ask the user which CI platform they use.

**This runbook provides workflow templates for GitHub Actions.** For other platforms, adapt the workflow structure accordingly.

### 1.4 Check for existing Cypress workflow

If a CI workflow file already exists that runs Cypress (e.g., `.github/workflows/cypress.yml`):

- Read it and check if it already triggers on label events.
- If it does → skip workflow creation, report as already configured.
- If it runs on push/PR but not on label → ask the user if they want to replace or keep both.

---

## 2. Cypress CI Workflow

### 2.1 Trigger: Label-based execution

The workflow triggers when a specific label is added to a pull request. The label name is `ready-to-test`.

**Why label-based?** Not every PR push should trigger a full Cypress run. The developer decides when the PR is ready for testing by adding the label. This avoids wasting CI minutes on work-in-progress PRs.

### 2.2 GitHub Actions workflow

Create the file `.github/workflows/cypress.yml`:

```yaml
name: Cypress Tests

on:
  pull_request:
    types: [labeled]

jobs:
  cypress-run:
    if: github.event.label.name == 'ready-to-test'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          start: npm start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120

      - name: Upload screenshots on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-screenshots
          path: cypress/screenshots/
          retention-days: 7

      - name: Upload videos
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-videos
          path: cypress/videos/
          retention-days: 7
```

### 2.3 Workflow configuration notes

The agent must adapt the workflow based on the project:

| Setting | How to determine |
|---------|-----------------|
| `node-version` | Read from `.nvmrc`, `.node-version`, or `package.json` engines field. Default: `18` |
| `npm ci` vs `yarn install --frozen-lockfile` vs `pnpm install --frozen-lockfile` | Detect from lockfile (same logic as setup agent, Step 5 in `01-setup-run.md`) |
| `npm start` | Read `package.json` scripts — use the dev server script (`start`, `dev`, `serve`). Ask the user if unclear |
| `wait-on` URL | Use the `baseUrl` from `cypress.config.ts`. If it is still the default `http://localhost:3000`, ask the user to confirm |
| `wait-on-timeout` | Default `120` seconds. Increase if the user reports slow startup |

### 2.4 Package manager variants

If the project uses **yarn**:

```yaml
      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          start: yarn start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
```

If the project uses **pnpm**:

```yaml
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          start: pnpm start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
```

### 2.5 Label creation

The `ready-to-test` label must exist in the GitHub repository. The agent should:

1. Check if the label already exists:
   ```bash
   gh label list | grep "ready-to-test"
   ```

2. If it does not exist, create it:
   ```bash
   gh label create "ready-to-test" --description "Triggers Cypress CI run on this PR" --color "0E8A16"
   ```

3. If `gh` CLI is not available or not authenticated, provide manual instructions:
   > Go to GitHub → Repository → Issues → Labels → New Label
   > - Name: `ready-to-test`
   > - Description: Triggers Cypress CI run on this PR
   > - Color: `#0E8A16` (green)

---

## 3. Final Report

After completing the setup, show the following report:

```
========================================
  Cypress CI Setup — Final Report
========================================
CI Platform:      <detected platform>
Workflow File:    <path to workflow file>
Trigger:          Label "ready-to-test" added to PR
Package Manager:  <npm/yarn/pnpm>
Dev Server:       <start command>
Base URL:         <wait-on URL>
----------------------------------------
Preflight:        PASS / FAIL
Workflow Created: PASS / SKIPPED (already exists)
Label Created:    PASS / SKIPPED (already exists)
----------------------------------------
Overall Status:   SUCCESS / PARTIAL / FAILED
========================================

How to use:
1. Push your branch and create a PR.
2. When ready to test, add the "ready-to-test" label to the PR.
3. Cypress tests will run automatically.
4. Check the Actions tab for results.
5. On failure, download screenshots/videos from the workflow artifacts.

Action Required:
→ Verify that the dev server command and base URL are correct for your project.
→ If your project needs environment variables in CI, add them as GitHub repository secrets.
========================================
```

### Status classification

| Status | Meaning |
|--------|---------|
| SUCCESS | Workflow file created, label exists, all checks passed |
| PARTIAL | Workflow created but label could not be created (e.g., no `gh` CLI), or user needs to confirm settings |
| FAILED | Pre-flight failed (no Cypress, no tests) — agent stopped early |
