# Cypress Setup & Run — Runbook

Follow these steps sequentially. Report pass/fail after each step.

---

## Step 1: Detect Operating System

Identify the OS the agent is running on:
- Windows
- macOS
- Linux

This determines Node installation method and shell commands used in later steps.

---

## Step 2: Check if Node.js is Installed

Run:
```
node -v
```

- **If Node.js is found:** Report the version and proceed to Step 4.
- **If Node.js is NOT found:** Inform the user and proceed to Step 3.

Minimum required version: Node 18 or higher.
If an older version is found, treat it as "not installed" and proceed to Step 3.

---

## Step 3: Install Node.js (Auto-Install with User Consent)

Before installing, ask the user for consent:
> "Node.js is not installed (or below version 18). I need to install it to proceed with Cypress setup. May I install Node.js?"

- **If user consents:** Install based on OS detected in Step 1.
- **If user declines:** Stop and explain that Cypress requires Node.js 18+. Provide manual install instructions and exit.

### Windows
Download and run the official Node.js LTS installer:
```
curl -o node-installer.msi https://nodejs.org/dist/v22.14.0/node-v22.14.0-x64.msi
msiexec /i node-installer.msi /qn
```
After install, restart the shell and verify:
```
node -v
npm -v
```
Clean up:
```
rm node-installer.msi
```

### macOS / Linux
Install via nvm:
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```
Load nvm in current shell:
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```
Install Node LTS:
```
nvm install --lts
nvm use --lts
```
Verify:
```
node -v
npm -v
```

### If install fails
- Report the exact error message.
- Suggest the user install Node.js manually from https://nodejs.org.
- Stop execution — do not proceed without Node.js.

---

## Step 4: Check Network Connectivity

Verify the npm registry is reachable:
```
npm ping
```

- **If reachable:** Proceed.
- **If not reachable:** Warn the user. Suggest checking proxy/firewall settings. Stop if npm commands will fail.

---

## Step 5: Detect Package Manager

Look for lockfiles in the project root:

| Lockfile | Package Manager |
|----------|-----------------|
| `package-lock.json` | npm |
| `yarn.lock` | yarn |
| `pnpm-lock.yaml` | pnpm |
| None found | Default to npm |

Report which package manager will be used.

---

## Step 6: Ensure package.json Exists

Check if `package.json` exists in the project root.

- **If it exists:** Proceed to Step 7.
- **If it does NOT exist:** This is not a Node project. Inform the user:
  > "This project does not have a package.json. I'll create a minimal one just for Cypress testing. This won't affect your existing project setup."

  Run:
  ```
  npm init -y
  ```

  If the user wants Cypress isolated from the main project, create a subdirectory:
  ```
  mkdir cypress-tests
  cd cypress-tests
  npm init -y
  ```
  All subsequent steps run inside `cypress-tests/`.

---

## Step 7: Install Cypress

First, check if Cypress is already installed:
```
npx cypress --version
```

- **If already installed:** Report version, skip to Step 8.
- **If not installed:** Install as a dev dependency using the detected package manager:

| Package Manager | Command |
|-----------------|---------|
| npm | `npm install cypress --save-dev` |
| yarn | `yarn add cypress --dev` |
| pnpm | `pnpm add cypress --save-dev` |

### If install fails

| Error | Recovery |
|-------|----------|
| `EACCES` / `EPERM` | Ask for elevated permission, retry once |
| Network timeout | Check Step 4 connectivity, retry once |
| Disk space | Report error, ask user to free space |
| Other | Report exact error, suggest `npm install cypress --save-dev` manually |

---

## Step 8: Verify Cypress Binary

Run:
```
npx cypress verify
```

- **If verification passes:** Proceed to Step 9.
- **If binary is missing:** Run `npx cypress install` then verify again.

### Windows 11 smoke test issue

If `npx cypress verify` fails with `bad option: --smoke-test`, this is a **known compatibility issue** between Cypress 15.x's Electron binary and Windows 11 (build 26200+). Cypress is still installed correctly.

**Fallback verification:**
```
npx cypress open
```
- If the Cypress GUI opens successfully → Cypress is working. Report **PASS** with a note: "Smoke test skipped due to Windows 11 compatibility — verified via Cypress GUI."
- If `npx cypress open` also fails → Report as FAIL with the exact error.

Do **not** treat the smoke test failure as a blocker on Windows 11.

---

## Step 9: Scaffold Cypress Files (First Time Only)

**Skip this entire step** if both `cypress/` folder and `cypress.config.*` already exist.

### 9.1 Create Cypress config

Create `cypress.config.ts` (use `.js` if the project does not use TypeScript):

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: 'cypress/e2e/**/*.cy.{js,ts}',
    supportFile: 'cypress/support/e2e.{js,ts}',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
  },
});
```

Note: `baseUrl` is set to `http://localhost:3000` as a default. The user must update this in `cypress.config.ts` to match their actual dev server URL. For running tests against other environments (UAT, production), use: `npx cypress run --config baseUrl=https://your-environment-url.com`

### 9.2 Create support file

Create `cypress/support/e2e.ts` (or `.js`):

```typescript
// Cypress support file
// Add custom commands and global configuration here
```

### 9.3 Create directory structure

```
cypress/
├── e2e/          ← test files go here
├── fixtures/     ← test data (JSON files)
└── support/
    └── e2e.ts    ← support file (created above)
```

Create empty directories:
```
mkdir -p cypress/e2e cypress/fixtures
```

### 9.4 Update .gitignore

Append these lines to `.gitignore` (create the file if it doesn't exist):

```
# Cypress artifacts
cypress/videos/
cypress/screenshots/
```

Do not duplicate entries if they already exist in `.gitignore`.

---

## Step 10: Validate Setup

Run a quick validation to confirm everything works.

### If test files exist in `cypress/e2e/`:
```
npx cypress run --headless
```

### If no test files exist:
```
npx cypress verify
```
This confirms Cypress is installed and the binary works without needing test files.

### Fallback chain (if validation fails):
1. If the error is `bad option: --smoke-test` on Windows 11 → skip directly to step 4 (this is the known Windows 11 issue, retrying with Chrome won't help).
2. Retry with Chrome: `npx cypress run --headless --browser chrome`
3. Retry in headed mode: `npx cypress run --headed`
4. Run `npx cypress open` — if the GUI opens and loads correctly, Cypress is working. Report as **PASS** with a note.
5. If all fail: Report the exact error and stop.

---

## Step 11: Final Report

Show the user a summary:

```
========================================
  Cypress Setup — Final Report
========================================
OS:               <detected OS>
Node.js:          <version>
Package Manager:  <npm/yarn/pnpm>
Cypress:          <version>
----------------------------------------
Preflight:        PASS / FAIL
Install:          PASS / SKIPPED / FAIL
Binary Verify:    PASS / FAIL / PASS (via GUI)
Scaffold:         PASS / SKIPPED / FAIL
Validation:       PASS / FAIL / PASS (via GUI)
----------------------------------------
Notes:            <any relevant notes>
Overall Status:   SUCCESS / PARTIAL / FAILED
========================================

Action Required:
→ Update `baseUrl` in cypress.config.ts to your dev server URL.
→ To run tests against other environments:
  npx cypress run --config baseUrl=https://your-uat-url.com
========================================
```

### Status classification:

- **SUCCESS:** All steps passed. If smoke test failed but `npx cypress open` worked, this is still SUCCESS — add a note: "Smoke test skipped due to Windows 11 compatibility — verified via Cypress GUI."
- **PARTIAL:** Some steps failed but Cypress may still be usable. List what succeeded and what failed. Provide exact next steps.
- **FAILED:** A critical step failed and Cypress is not usable. Show the step that failed, the exact error, and what the user needs to do.

---

## Step 12: Code Coverage Setup (Optional)

This step sets up code coverage instrumentation so that when Cypress tests run, they report which lines/branches/functions of the application source code were executed.

**Ask the user before proceeding:**
> "Would you like to set up code coverage for Cypress? This instruments your app code so test runs generate coverage reports showing which lines were exercised."

- **If user agrees:** Proceed with Step 12.
- **If user declines:** Skip to Final Report. Coverage can be added later.

### 12.1 Detect Build Tool

Examine the project to determine the build tool / framework:

| Check | Build Tool | Instrumentation Package |
|-------|-----------|------------------------|
| `vite.config.ts` or `vite.config.js` exists | Vite | `vite-plugin-istanbul` |
| `react-scripts` in `package.json` dependencies | Create React App (CRA) | `@cypress/instrument-cra` |
| `next.config.js` or `next.config.mjs` exists | Next.js | `babel-plugin-istanbul` (requires forcing Babel over SWC) |
| `angular.json` exists | Angular | `istanbul-instrumenter-loader` via `@angular-builders/custom-webpack` |
| `webpack.config.js` or `webpack.config.ts` exists | Webpack (generic) | `istanbul-instrumenter-loader` |
| None of the above | Plain HTML / other | `nyc instrument` CLI (manual instrumentation) |

Report the detected build tool to the user before proceeding.

### 12.2 Install Coverage Packages

**Always install (all build tools):**

| Package Manager | Command |
|-----------------|---------|
| npm | `npm install --save-dev @cypress/code-coverage nyc` |
| yarn | `yarn add --dev @cypress/code-coverage nyc` |
| pnpm | `pnpm add --save-dev @cypress/code-coverage nyc` |

**Then install the build-tool-specific instrumentation package:**

| Build Tool | npm Command |
|-----------|-------------|
| Vite | `npm install --save-dev vite-plugin-istanbul` |
| CRA | `npm install --save-dev @cypress/instrument-cra` |
| Next.js | `npm install --save-dev babel-plugin-istanbul` |
| Angular | `npm install --save-dev @angular-builders/custom-webpack istanbul-instrumenter-loader` |
| Webpack | `npm install --save-dev istanbul-instrumenter-loader` |
| Plain HTML | No additional package — uses `nyc instrument` CLI |

Adapt the command for the detected package manager (yarn/pnpm).

### 12.3 Configure Instrumentation

#### Vite

Add the istanbul plugin to `vite.config.ts` (or `.js`):

```typescript
import istanbul from 'vite-plugin-istanbul';

export default defineConfig({
  plugins: [
    // ...existing plugins
    istanbul({
      include: 'src/*',
      exclude: ['node_modules', 'cypress'],
      extension: ['.js', '.ts', '.jsx', '.tsx', '.vue', '.svelte'],
      cypress: true,
      requireEnv: false,
    }),
  ],
  build: {
    sourcemap: true,
  },
});
```

> **Note:** `requireEnv: false` instruments code in all environments. To instrument only when running Cypress, set `requireEnv: true` and run with `VITE_COVERAGE=true`.

#### Create React App (CRA)

No build config changes needed. Modify the dev server start script in `package.json`:

```json
{
  "scripts": {
    "start:coverage": "react-scripts -r @cypress/instrument-cra start"
  }
}
```

Use `start:coverage` instead of `start` when running Cypress with coverage.

#### Next.js

Next.js uses SWC by default, which does not support Istanbul. You must force Babel by creating a `.babelrc` in the project root:

```json
{
  "presets": ["next/babel"],
  "plugins": ["istanbul"]
}
```

> **Warning:** This disables SWC and uses Babel instead, which is slower. Inform the user of this trade-off.

#### Angular

Update `angular.json` to use custom webpack builder and create a custom webpack config that includes the istanbul instrumenter loader. This is project-specific — ask the user for guidance on their Angular build setup before modifying.

#### Webpack (generic)

Add `istanbul-instrumenter-loader` as a rule in the webpack config:

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.[jt]sx?$/,
        exclude: /node_modules|cypress/,
        enforce: 'post',
        use: {
          loader: 'istanbul-instrumenter-loader',
          options: { esModules: true },
        },
      },
    ],
  },
  devtool: 'source-map',
};
```

#### Plain HTML / Other

Instrument the source files manually before serving:

```bash
npx nyc instrument --compact=false src instrumented
```

Then serve the `instrumented/` directory instead of `src/`. Ask the user how they serve their application and adapt accordingly.

### 12.4 Register Coverage Plugin in Cypress

#### Update `cypress/support/e2e.ts` (or `.js`)

Add at the top of the file:

```typescript
import '@cypress/code-coverage/support';
```

#### Update `cypress.config.ts` (or `.js`)

Add the coverage task registration:

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    // ...existing config
    setupNodeEvents(on, config) {
      require('@cypress/code-coverage/task')(on, config);
      // IMPORTANT: return the config object
      return config;
    },
  },
});
```

> **If `setupNodeEvents` already exists:** Merge the coverage task into the existing function. Do not overwrite existing event handlers.

### 12.5 Configure NYC Reporters

Add NYC configuration to `package.json`:

```json
{
  "nyc": {
    "reporter": ["text", "lcov", "json-summary"],
    "report-dir": "coverage",
    "all": true,
    "include": ["src/**/*.{js,ts,jsx,tsx,vue,svelte}"],
    "exclude": [
      "node_modules",
      "cypress",
      "**/*.spec.*",
      "**/*.test.*",
      "**/*.cy.*"
    ]
  }
}
```

| Reporter | Purpose |
|----------|---------|
| `text` | Prints summary table to console |
| `lcov` | Generates HTML report at `coverage/lcov-report/index.html` and `lcov.info` for CI uploads |
| `json-summary` | Machine-readable summary at `coverage/coverage-summary.json` — used by CI to parse coverage % |

### 12.6 Update .gitignore

Append coverage output directories to `.gitignore`:

```
# Coverage reports
coverage/
.nyc_output/
```

Do not duplicate entries if they already exist.

### 12.7 Validate Coverage Setup

Run a quick test to verify coverage is working:

```bash
npx cypress run --headless
```

After the run, check:

1. `.nyc_output/` directory exists and contains JSON files
2. `coverage/` directory exists
3. `coverage/lcov-report/index.html` exists (open in browser to verify)
4. Console output includes a coverage summary table

If coverage output is missing:
- Verify `window.__coverage__` exists in the browser: add a test with `cy.window().then(win => expect(win.__coverage__).to.exist)`
- Check that instrumentation is configured for the correct source directory
- Check that source maps are enabled in the build config

### 12.8 Update Final Report

If coverage setup was performed, add these lines to the Final Report:

```
Coverage Setup:   PASS / FAIL
Build Tool:       <detected build tool>
Coverage Report:  coverage/lcov-report/index.html
```

And add to Action Required:

```
→ Run `npx cypress run` and open coverage/lcov-report/index.html to view the coverage report.
→ To disable coverage temporarily: set CYPRESS_COVERAGE=false
```
