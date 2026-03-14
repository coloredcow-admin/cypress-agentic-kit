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
