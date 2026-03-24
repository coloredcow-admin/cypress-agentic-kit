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

### 1.2 Detect Cypress location

Determine where `cypress.config.ts` (or `.js`) lives — this decides whether the workflow needs a `working-directory`.

1. Check the **project root** for `cypress.config.ts` or `cypress.config.js`.
2. If not found at root → search subdirectories (one level deep):
   ```bash
   find . -maxdepth 2 -name "cypress.config.*" -not -path "*/node_modules/*"
   ```
3. If found in a subdirectory (e.g., `cypress-tests/`) → that directory is the **Cypress root**. All Cypress-related paths (lockfile, config, tests, screenshots) are relative to it, and the workflow must use `working-directory`.

| `cypress.config` location | Cypress root | `working-directory` |
|---|---|---|
| `./cypress.config.ts` | Project root | Not needed |
| `./cypress-tests/cypress.config.ts` | `cypress-tests/` | `cypress-tests` |

If `cypress.config` is not found anywhere → stop. Direct the user to run the setup agent first (`prompts/bootstrap.md`).

### 1.3 Tests exist

Check that at least one test file exists inside the **Cypress root**:

```
<cypress-root>/cypress/e2e/**/*.cy.{js,ts}
```

- If no test files exist → stop. Direct the user to write tests first (`prompts/test-writer.md`).

### 1.4 Detect CI platform

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

### 1.5 Check for existing Cypress workflow

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

> **IMPORTANT:** The template below is a **reference structure**, not a copy-paste workflow. Every `<placeholder>` must be replaced with values detected from the project. Do not use the template as-is — always resolve all placeholders first using the detection rules in Section 2.3.

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
          node-version: <node-version>

      # --- Include this section ONLY for non-Node projects (Section 2.4) ---
      # <app-setup-steps>

      - name: Cache Cypress binary
        uses: actions/cache@v4
        with:
          path: ~/.cache/Cypress
          key: cypress-${{ runner.os }}-${{ hashFiles('<lockfile-path>') }}

      - name: Install dependencies
        # Include 'working-directory' ONLY if Cypress is in a subdirectory
        # working-directory: <cypress-root>
        run: <install-command>

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          # Include these two ONLY if Cypress is in a subdirectory
          # install: false
          # working-directory: <cypress-root>
          start: <dev-server-start-command>
          wait-on: '<base-url>'
          wait-on-timeout: 120

      - name: Upload screenshots on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-screenshots
          path: <cypress-root>/cypress/screenshots/
          retention-days: 7
```

> **Subdirectory setup:** When Cypress lives in a subdirectory (e.g., `cypress-tests/`), uncomment the `working-directory` and `install: false` lines and replace `<cypress-root>` with the subdirectory name. The `install: false` is needed because dependencies are already installed in the separate "Install dependencies" step with `working-directory`.

### 2.3 Workflow configuration notes

The agent must adapt the workflow based on the project:

| Placeholder | How to determine |
|-------------|-----------------|
| `<node-version>` | Check project root for `.nvmrc` or `.node-version`. If not found, check `engines` field in `./package.json`. Default: `20` |
| `<cypress-root>` | Detected in Pre-flight Section 1.2. If `cypress.config` is at project root → use `./` (omit `working-directory`). If in a subdirectory → use that directory name (e.g., `cypress-tests`) |
| `<lockfile-path>` | Look inside `<cypress-root>` for: `package-lock.json` (npm), `yarn.lock` (yarn), `pnpm-lock.yaml` (pnpm). If `<cypress-root>` is a subdirectory, prefix the path (e.g., `cypress-tests/package-lock.json`) |
| `<install-command>` | `npm ci` / `yarn install --frozen-lockfile` / `pnpm install --frozen-lockfile` — detect from lockfile found in `<cypress-root>` |
| `<dev-server-start-command>` | Read the **project root** `package.json` scripts — use the dev server script (`start`, `dev`, `serve`). If no dev server script is found (common for non-Node projects like Python, Java, .NET), ask the user: "How do you start your application locally?" |
| `<base-url>` | Read `baseUrl` from `<cypress-root>/cypress.config.ts` (or `.js`). If it is still the default `http://localhost:3000`, ask the user to confirm |
| `<app-setup-steps>` | Only needed for non-Node projects. See Section 2.4 |
| `wait-on-timeout` | Default `120` seconds. Increase if the user reports slow startup |

### 2.4 Project-specific variants

#### Package manager variants

If the project uses **yarn**:

```yaml
      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          start: yarn <dev-server-script>
          wait-on: '<base-url>'
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
          start: pnpm <dev-server-script>
          wait-on: '<base-url>'
          wait-on-timeout: 120
```

#### Non-Node project adjustments

For projects where the main application is not Node-based (Python, Java, .NET, etc.), Cypress typically lives in a subdirectory (e.g., `cypress-tests/`). The workflow needs additional steps to set up and start the application.

Replace `<app-setup-steps>` in the template with steps specific to the project's tech stack. Ask the user what commands are needed. Common examples:

**Python:**
```yaml
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '<python-version>'

      - name: Install app dependencies
        run: pip install -r requirements.txt

      - name: Start application
        run: <python-start-command> &
```

**Java (Maven):**
```yaml
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '<java-version>'

      - name: Build and start application
        run: <java-start-command> &
```

> **Key points for non-Node projects:**
> - The app start command is NOT in any `package.json` — always ask the user.
> - The `&` at the end runs the app in the background so the workflow can proceed to Cypress.
> - The `start` field in cypress-io/github-action should be omitted — the app is already started in a prior step.
> - `working-directory` and `install: false` are required on the Cypress action step.

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

## 2.6 Code Coverage Reporting on PR

If code coverage is set up in the project (check for `@cypress/code-coverage` in `package.json` devDependencies), add coverage reporting to the CI workflow.

### How it works

1. When Cypress runs with instrumentation enabled, `@cypress/code-coverage` generates coverage reports in `coverage/`.
2. The workflow parses the coverage percentage from `coverage/coverage-summary.json`.
3. A sticky PR comment is posted with the coverage summary (same comment updates on each push).

### Add to the workflow

Add these steps **after** the "Run Cypress tests" step in the workflow:

```yaml
      - name: Extract coverage percentage
        if: always()
        id: coverage
        run: |
          if [ -f coverage/coverage-summary.json ]; then
            COVERAGE=$(node -e "const c = require('./coverage/coverage-summary.json'); console.log(c.total.lines.pct)")
            echo "percentage=$COVERAGE" >> $GITHUB_OUTPUT
            echo "Coverage: $COVERAGE%"

            # Determine emoji based on threshold
            if [ "$(echo "$COVERAGE >= 90" | bc -l)" -eq 1 ]; then
              echo "emoji=✅" >> $GITHUB_OUTPUT
            elif [ "$(echo "$COVERAGE >= 70" | bc -l)" -eq 1 ]; then
              echo "emoji=⚠️" >> $GITHUB_OUTPUT
            else
              echo "emoji=🔴" >> $GITHUB_OUTPUT
            fi

            # Generate text report for PR comment
            COVERAGE_TEXT=$(npx nyc report --reporter=text 2>/dev/null || echo "See coverage artifact for full details.")
            # Write multiline output using delimiter
            echo "coverage_text<<EOF" >> $GITHUB_OUTPUT
            echo "$COVERAGE_TEXT" >> $GITHUB_OUTPUT
            echo "EOF" >> $GITHUB_OUTPUT
          else
            echo "percentage=N/A" >> $GITHUB_OUTPUT
            echo "No coverage report found"
          fi
        working-directory: ${{ env.CYPRESS_ROOT || '.' }}

      - name: Post coverage comment on PR
        if: always() && steps.coverage.outputs.percentage != 'N/A'
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          header: cypress-coverage
          message: |
            ### ${{ steps.coverage.outputs.emoji }} Cypress Test Coverage: ${{ steps.coverage.outputs.percentage }}%

            <details>
            <summary>Coverage Details</summary>

            ```text
            ${{ steps.coverage.outputs.coverage_text }}
            ```

            </details>

            <!-- Sticky Pull Request Comment -->

      - name: Upload coverage report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-coverage-report
          path: ${{ env.CYPRESS_ROOT || '.' }}/coverage/
          retention-days: 14
```

### Coverage thresholds

The PR comment uses color-coded indicators:

| Coverage | Indicator | Meaning |
|----------|-----------|---------|
| >= 90% | ✅ | Good coverage |
| >= 70% | ⚠️ | Acceptable, room for improvement |
| < 70% | 🔴 | Low coverage, needs attention |

These thresholds are defaults. The user can adjust them in the workflow.

### If coverage is NOT set up

If `@cypress/code-coverage` is not in the project's devDependencies, skip all coverage steps. The workflow runs Cypress tests without coverage — pass/fail only. Do not add coverage steps to the workflow.

### Subdirectory support

If Cypress is in a subdirectory, set the `CYPRESS_ROOT` env variable at the job level:

```yaml
jobs:
  cypress-run:
    env:
      CYPRESS_ROOT: cypress-tests
```

The coverage steps use `${{ env.CYPRESS_ROOT || '.' }}` to resolve paths correctly.

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
Coverage in CI:   PASS / SKIPPED (not set up) / N/A
----------------------------------------
Overall Status:   SUCCESS / PARTIAL / FAILED
========================================

How to use:
1. Push your branch and create a PR.
2. When ready to test, add the "ready-to-test" label to the PR.
3. Cypress tests will run automatically.
4. Check the Actions tab for results.
5. On failure, download screenshots from the workflow artifacts.

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
