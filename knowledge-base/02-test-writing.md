# Cypress Test Writing Standards — Runbook

Follow these sections sequentially. Each section must pass before proceeding to the next.

---

## Section 1: Pre-flight Checks

Before writing any test, verify the environment is ready and understand what already exists.

### Step 1.0: Verify node_modules is Installed

Before running any Cypress commands, confirm that project dependencies are installed.

Check whether the `node_modules` directory exists and is non-empty:
```
ls node_modules
```

- **If `node_modules` exists and contains packages:** Proceed to Step 1.1.
- **If `node_modules` is missing or empty:** This is a **hard blocker**. Stop immediately and inform the user:
  > "`node_modules` is missing or empty. Dependencies are not installed. Please run `npm install` (or `yarn install` / `pnpm install`) and then re-run the agent."

  Do not proceed further. Do not attempt to run `npx cypress` or any other npm-based command — they will fail with `Cannot find module 'cypress'` and produce misleading errors.

---

### Step 1.1: Verify Cypress Installation

Run:
```
npx cypress --version
```

- **If Cypress is installed:** Report version, proceed to Step 1.2.
- **If Cypress is NOT installed:** Stop. Inform the user:
  > "Cypress is not installed in this project. Please run the setup agent first (`cypress-setup-agent`) or install Cypress manually before writing tests."

  Do not proceed further.

### Step 1.2: Read Cypress Configuration

Look for `cypress.config.ts` or `cypress.config.js` in the project root.

- **If found:** Read the file and extract:
  - `baseUrl` — used for `cy.visit()` relative paths
  - `specPattern` — determines where test files must be placed
  - `supportFile` — path to the support file (needed for custom commands check)
  - `viewportWidth` / `viewportHeight` — for reference
  - Any other custom config (env variables, retries, timeouts)

- **If not found:** Stop. Inform the user:
  > "No `cypress.config.ts` or `cypress.config.js` found. Please run the setup agent first to scaffold the Cypress configuration."

  Do not proceed further.

### Step 1.3: Check for Existing Custom Commands

Read the support file path from the config (default: `cypress/support/e2e.ts` or `cypress/support/e2e.js`).

If the support file exists, also check for:
- `cypress/support/commands.ts` (or `.js`)
- Any file imported inside the support file

Read these files and note all existing custom commands (e.g., `Cypress.Commands.add('login', ...)`).

**Purpose:** The agent must reuse existing custom commands instead of duplicating them. If a `login` command already exists, the agent uses `cy.login()` — it does not write the login steps inline.

- **If custom commands found:** List them internally for reference during test writing.
- **If no custom commands found:** Proceed — agent may create new ones later if needed (see Section 8).

### Step 1.4: Scan Existing Tests

Check if any test files exist in the `cypress/e2e/` directory (or the path defined by `specPattern`).

- **If existing tests found:**
  - Note the folder structure (are tests organized into subdirectories like `smoke/`, `regression/`?)
  - Note the naming convention (e.g., `feature.cy.ts` vs `feature.spec.ts`)
  - Note the code patterns (describe/context/it structure, selector style, assertion style)
  - **Rule:** Follow the existing conventions. Do not introduce a different style unless the user explicitly asks for it.

- **If no existing tests found:** Use KB defaults from Sections 3–9.

### Pre-flight Summary

After completing Steps 1.0–1.4, the agent must have:

| Item | Status |
|------|--------|
| node_modules | Exists and non-empty — confirmed |
| Cypress version | Detected |
| Config file | Read — baseUrl, specPattern, supportFile extracted |
| Custom commands | Listed (or none found) |
| Existing test conventions | Learned (or using KB defaults) |

Only proceed to Section 2 when all items are confirmed.

---

## Section 2: Project Structure Awareness

The agent needs to understand the project's structure to write meaningful tests — what framework is used, where pages/components live, how routing works, etc. Instead of scanning the entire project on every run, the agent finds or creates a project structure file once and reuses it.

### Step 2.1: Check for Existing Project Structure File

Look for a `project-structure-file:` pointer in `agents/CLAUDE.md` first, then in the project's root `CLAUDE.md`:

```
## Project Structure Reference
project-structure-file: <path-to-file>
```

- **If pointer found and file exists:** Read that file. Skip to Step 2.4.
- **If pointer found but file is missing:** Pointer is stale. Proceed to Step 2.2.
- **If no pointer found in either file:** Proceed to Step 2.2.

### Step 2.2: Scan for Existing Structure Files

Search for `.md` files in these locations that contain project structure information:
- Project root
- `.claude/`
- `knowledge-base/`
- `docs/`

Read any `.md` file whose content contains keywords like: "project structure", "architecture", "directory layout", "tech stack", "folder structure", "project overview".

- **If a matching file is found:** Use it as the project structure reference. Add the pointer to `agents/CLAUDE.md`:
  ```
  ## Project Structure Reference
  project-structure-file: <path-to-found-file>
  ```
  Skip to Step 2.4.

- **If no matching file found:** Proceed to Step 2.3.

### Step 2.3: Generate Project Structure File

Scan the project once and create `knowledge-base/project-structure.md` containing:

**Project type:**
- Frontend-only / Backend-only / Full-stack / Monorepo
- Detected by: presence of `src/`, `pages/`, `api/`, `server/`, `packages/`, workspace config

**Framework:**
- Detect from `package.json` dependencies: React, Angular, Vue, Next.js, Nuxt, Svelte, etc.
- Detect from non-Node markers: `composer.json` (Laravel), `requirements.txt` (Django), `Gemfile` (Rails)

**Key directories:**
- Where pages/views/screens live (e.g., `src/pages/`, `app/`, `views/`)
- Where components live (e.g., `src/components/`)
- Where API routes/endpoints are defined (e.g., `src/api/`, `routes/`, `server/`)
- Where static assets live (e.g., `public/`, `static/`)

**Routing:**
- File-based routing (Next.js, Nuxt) — detect from directory structure
- Config-based routing (React Router, Vue Router) — detect from router config file
- Framework routing (Laravel, Rails) — detect from routes file

**Auth mechanism (if detectable):**
- Login page path
- Auth middleware or guards
- Token/session patterns

**Environment:**
- Dev server URL (from `package.json` scripts, `.env` files, or framework defaults)
- Any `.env` pattern detected (`.env.local`, `.env.development`)

After generating the file, add the pointer to `agents/CLAUDE.md`:
```
## Project Structure Reference
project-structure-file: knowledge-base/project-structure.md
```

### Step 2.4: Confirm Structure is Loaded

The agent must have answers for:

| Item | Required |
|------|----------|
| Project type (frontend/backend/full-stack/monorepo) | Yes |
| Framework | Yes |
| Pages/views directory | Yes |
| Components directory | If frontend |
| API routes directory | If applicable |
| Routing type | Yes |
| Dev server URL | Yes (needed for baseUrl context) |

If any required item is missing from the structure file, ask the user to fill in the gaps. Do not guess.

### Step 2.5: Resolve the App Entry Point URL

Before writing any `cy.visit()` call, confirm what URL actually loads the application. Do not assume `cy.visit('/')` loads the app — verify it.

**Read the `start` (or `dev`) script from `package.json` and apply this logic:**

| Server tool | Check | Risk |
|-------------|-------|------|
| `serve`, `http-server`, `live-server` | Does `index.html` exist in the served root directory? | If no `index.html`, `/` serves a directory listing — not the app |
| `vite`, `webpack-dev-server`, `next dev`, `react-scripts start` | These always serve the app at `/` | No risk — `cy.visit('/')` loads the app |

**Resolution steps:**

1. Identify the server tool from the `start`/`dev` script
2. If the tool is `serve`, `http-server`, or `live-server`:
   - Check if `index.html` exists in the directory being served
   - If `index.html` is missing, look at the project structure (Step 2.1–2.3) to find the actual app HTML file (e.g., `app.html`, `temp-design.html`)
   - Derive the correct visit path: `cy.visit('/temp-design.html')` — not `cy.visit('/')`
3. Note this entry point path. Use it in every `cy.visit()` call in the tests.

**Rule:** Never write `cy.visit('/')` without first verifying it serves the app. The entry point is a fact to derive from the project — not an assumption.

Only proceed to Section 3 when the project structure is confirmed.

---

## Section 3: Test Categorization & Folder Structure

Tests must be organized by category. The agent must know where to place each test file based on what it tests.

### 3.1: Default Folder Structure

```
cypress/e2e/
├── smoke/           ← critical happy paths, runs first, fast
├── regression/      ← detailed coverage, edge cases, validations
├── integration/     ← API + UI interaction tests
└── functional/      ← feature-specific, all scenarios for one feature
```

### 3.2: What Goes Where

Use the **purpose** of each category to decide placement. The examples below are illustrative, not exhaustive — apply the category definition to whatever the user asks for.

| Category | Purpose | Example scenarios (for reference only) |
|----------|---------|---------------------------------------|
| `smoke/` | Critical happy-path flows that must never break. Fast, minimal, high-value tests. | Login happy path, signup happy path, checkout completion, primary navigation |
| `regression/` | Detailed coverage of existing features — edge cases, error states, boundary values, validations. | Form validation errors, empty states, permission denied flows, pagination boundaries |
| `integration/` | Tests that verify the interaction between UI and API — requests, responses, error handling from backend. | Verifying API responses render correctly, network error handling, data submission and confirmation |
| `functional/` | Complete coverage of a single feature — all scenarios (happy + edge) grouped in one suite. | Full test suite for search functionality, complete profile page coverage |

### 3.3: How the Agent Decides the Category

Follow this order:

1. **User specifies the category** → use it, no questions asked.
2. **User does not specify** → agent decides based on the **purpose definitions** in 3.2. Ask:
   - Is this a core happy path that should always pass? → `smoke/`
   - Is this testing edge cases, validations, or error states of an existing feature? → `regression/`
   - Is this testing how the UI interacts with an API? → `integration/`
   - Is this a complete suite covering all scenarios for one feature? → `functional/`

3. **Still ambiguous** → ask the user:
   > "Should this be a smoke test (core happy path) or regression test (detailed edge cases)?"

### 3.4: Folder Creation

Before placing a test file, check if the target folder exists.

- **If `cypress/e2e/<category>/` exists:** Place the test file there.
- **If it does not exist:** Create the folder, then place the test file.

### 3.5: Respecting Existing Structure

If pre-flight (Section 1, Step 1.4) detected that existing tests use a different folder structure (e.g., tests organized by page like `cypress/e2e/auth/`, `cypress/e2e/dashboard/`), **follow the existing structure**.

Do not force the default category folders if the project already has its own organization. Instead:
- Understand the existing pattern
- Place new tests consistently with what's already there
- If the user wants to switch to category-based folders, they must explicitly ask for it

Only proceed to Section 4 when the test category and target folder are determined.

---

## Section 4: File & Naming Conventions

### 4.1: Test File Naming

| Rule | Convention |
|------|-----------|
| Format | `<feature-name>.cy.ts` (use `.cy.js` if the project does not use TypeScript) |
| Casing | kebab-case — `user-profile.cy.ts`, not `userProfile.cy.ts` or `user_profile.cy.ts` |
| Name must describe the feature | `login.cy.ts`, `checkout.cy.ts`, `search-filters.cy.ts` — never `test1.cy.ts` or `new-test.cy.ts` |
| One file per scope | One test file covers one feature or one specific aspect of a feature |

### 4.2: File Placement

Test files go in the category folder determined in Section 3:

```
cypress/e2e/<category>/<feature-name>.cy.ts
```

If the project uses a non-category structure (detected in Section 1, Step 1.4), follow that structure instead.

### 4.3: Fixture File Naming

When a test needs static test data (mock API responses, form data, user records):

| Rule | Convention |
|------|-----------|
| Location | `cypress/fixtures/` |
| Format | `<feature-name>.json` or `<feature-name>/<specific-data>.json` for multiple fixtures per feature |
| Name matches the feature it serves | `users.json`, `login/valid-credentials.json`, `products/search-results.json` |

Only create fixture files when the test actually needs static data. Do not create empty or placeholder fixtures.

### 4.4: Follow Existing Conventions

If pre-flight (Section 1, Step 1.4) detected existing tests with a different convention:

- **Different file extension** (e.g., `.spec.ts` instead of `.cy.ts`) → use the existing extension
- **Different casing** (e.g., `camelCase` instead of `kebab-case`) → use the existing casing
- **Different structure** (e.g., tests grouped by page instead of category) → follow existing structure

The KB defaults apply only when no existing convention is detected. Consistency with the existing codebase always takes priority.

Only proceed to Section 5 when the file name and path are determined.

---

## Section 5: Test Structure Standards

This section defines how to organize code inside a test file.

### 5.1: Hierarchy

Every test file follows this structure. The code examples throughout this section illustrate the **pattern and format only** — do not copy them literally. Adapt the structure, names, selectors, and assertions to match the actual feature being tested.

```typescript
describe('Feature Name', () => {

  beforeEach(() => {
    // setup that runs before every test — navigation, intercepts, fixtures
  })

  context('Scenario Group', () => {

    it('should do one specific thing', () => {
      // arrange → act → assert
    })

    it('should do another specific thing', () => {
      // arrange → act → assert
    })

  })

  context('Another Scenario Group', () => {
    // ...
  })

})
```

**Rules:**

| Block | Purpose | Rule |
|-------|---------|------|
| `describe` | The feature being tested | One per file. Name matches the feature: `'Login Page'`, `'Checkout Flow'`, `'Search Filters'` |
| `context` | A scenario group within the feature | Groups related tests: `'with valid credentials'`, `'with empty cart'`, `'when user is not logged in'` |
| `it` | One specific behavior | Tests exactly one thing. Starts with `'should'`: `'should redirect to dashboard'`, `'should show error message'` |
| `beforeEach` | Shared setup | Runs before every `it` in its scope. Used for navigation, intercepts, common state. Never for assertions. |

### 5.2: The Arrange → Act → Assert Pattern

Every `it` block follows this flow:

```typescript
it('should show error for invalid email', () => {
  // Arrange — set up the state
  cy.get('[data-cy="email-input"]').clear()

  // Act — perform the action
  cy.get('[data-cy="email-input"]').type('not-an-email')
  cy.get('[data-cy="submit-btn"]').click()

  // Assert — verify the outcome
  cy.get('[data-cy="email-error"]').should('be.visible')
  cy.get('[data-cy="email-error"]').should('contain.text', 'valid email')
})
```

The comments (`// Arrange`, `// Act`, `// Assert`) are optional — the pattern is what matters, not the labels. Do not add these comments unless the test is complex enough to benefit from them. The example above is for reference — adapt the flow to the actual test scenario.

### 5.3: Test Isolation

Each `it` block must be completely independent.

- **No shared state between tests.** One test must never depend on the outcome of another test.
- **No order dependency.** Tests must pass in any order — Cypress can randomize test order.
- **Setup goes in `beforeEach`.** If multiple tests need the same starting state, put it in `beforeEach`, not in a prior `it` block.

The following wrong/right examples illustrate the **principle of test isolation** — they are not templates to copy.

**Wrong:**
```typescript
// DO NOT do this — test 2 depends on test 1
it('should create a user', () => {
  cy.get('[data-cy="name"]').type('John')
  cy.get('[data-cy="submit"]').click()
})

it('should show the created user in the list', () => {
  // This fails if the above test didn't run first
  cy.get('[data-cy="user-list"]').should('contain.text', 'John')
})
```

**Right:**
```typescript
beforeEach(() => {
  // Each test starts from the same known state
  cy.visit('/users')
})

it('should create a user', () => {
  cy.get('[data-cy="name"]').type('John')
  cy.get('[data-cy="submit"]').click()
  cy.get('[data-cy="success-message"]').should('be.visible')
})

it('should show existing users in the list', () => {
  // Uses its own setup — does not depend on the test above
  cy.intercept('GET', '/api/users', { fixture: 'users.json' }).as('getUsers')
  cy.wait('@getUsers')
  cy.get('[data-cy="user-list"]').should('contain.text', 'John')
})
```

### 5.4: When to Use `context` vs Flat `it` Blocks

- **Use `context`** when there are multiple scenarios for the same feature (e.g., valid vs invalid input, logged in vs logged out, empty vs populated state).
- **Skip `context`** when the test file has only 2–3 simple tests that don't need grouping. Flat `it` blocks inside `describe` are fine for simple cases.

Do not add `context` blocks just for the sake of structure. They should make the test file easier to read, not add unnecessary nesting.

Only proceed to Section 6 when the test structure is planned.

---

## Section 6: Selector Strategy

The agent must identify elements in the application to interact with them in tests. This section defines how to select elements and how to add test-specific attributes when they are missing.

### 6.1: Selector Priority

When choosing a selector for an element, follow this priority order:

| Priority | Selector Type | How it looks in test | When to use |
|----------|--------------|---------------------|-------------|
| 1 | `data-cy` | `cy.get('[data-cy="submit-btn"]')` | Preferred — dedicated test attribute, does not break on UI changes |
| 2 | `data-testid` | `cy.get('[data-testid="submit"]')` | If project already uses this convention |
| 3 | `aria-` roles/labels | `cy.get('[role="dialog"]')`, `cy.get('[aria-label="Close"]')` | For accessible elements where `data-cy` is not yet added |
| 4 | Stable ID or class | `cy.get('#login-form')`, `cy.get('.nav-menu')` | Only if options 1–3 are not available |

### 6.2: Selectors to Never Use

- **Tag chains:** `div > ul > li:nth-child(3)` — breaks on any DOM change
- **Dynamic/generated classes:** CSS module hashes, Tailwind JIT classes, styled-component classes
- **Text content:** `cy.contains('Submit')` as primary selector — breaks with i18n or copy changes (acceptable only as a secondary assertion, not for element targeting)
- **Positional selectors:** `.eq(index)`, `.first()`, `.last()` — these depend on DOM order, which changes with layout updates, async rendering, or different data. If you need a specific element among siblings, use a `data-cy` attribute to identify it directly
- **XPath:** not supported natively in Cypress, do not use

### 6.3: Read the Implementation Before Writing Tests

Before writing any test, the agent must read the source code of the feature being tested (use project structure from Section 2 to locate the file). This is the source of truth — do not guess how a component behaves.

**Read the implementation to understand:**

1. **What events the component listens to** — e.g., `onClick`, `onMouseOver`, `onChange`, `onSubmit`. Use the exact event the component expects in the test (e.g., if the component uses `onMouseOver`, trigger `mouseover` in the test — not `mouseenter`).
2. **What third-party libraries/components are used** — e.g., MUI, Ant Design, React Select. This determines how to interact with complex widgets (date pickers, dropdowns, tooltips, modals).
3. **How data flows** — what API response fields are mapped to which UI elements. This tells you what to stub in fixtures and what the UI should display.
4. **What CSS classes or conditional rendering logic controls visibility, styles, and states** — so you can assert on classes or attributes instead of computed styles.
5. **All elements the test will interact with** — buttons, inputs, links, containers, error messages, etc. — for adding `data-cy` attributes.

If the implementation is unclear or uses a pattern you don't understand, **ask the user**. Do not guess.

### 6.3.1: Adding `data-cy` Attributes to Application Code

After reading the implementation, add `data-cy` attributes to the application source code for elements the test needs.

**Process:**

1. From the implementation read in 6.3, identify all elements the test will interact with
2. Add `data-cy` attributes directly to those elements in the application source code
3. Use the added `data-cy` attributes in the test file

**Rules for adding `data-cy` attributes:**

- **Do not overwrite** existing `data-cy` or `data-testid` attributes — reuse them as-is
- **Do not add to every element** — only add to elements the test actually needs
- **Do not change any other code** in the component — only add the `data-cy` attribute to the relevant HTML/JSX elements

### 6.4: `data-cy` Naming Convention

Format: `<feature>-<element>-<type>`

| Part | What it describes | Examples |
|------|-------------------|----------|
| `feature` | The feature or page this element belongs to | `login`, `checkout`, `search`, `nav` |
| `element` | What this specific element is | `email`, `submit`, `error`, `title`, `list` |
| `type` | The element type (optional, add when it improves clarity) | `btn`, `input`, `link`, `msg`, `container` |

**Examples (for reference — adapt naming to the actual feature):**

```
data-cy="login-email-input"
data-cy="login-submit-btn"
data-cy="login-error-msg"
data-cy="nav-profile-link"
data-cy="search-results-container"
data-cy="checkout-total"
```

**Rules:**
- kebab-case only
- Keep names short but descriptive
- Be consistent within a feature — if one element is `login-email-input`, don't name another `loginPasswordField`

### 6.5: Framework-Specific Attribute Placement

Where to add `data-cy` depends on the framework detected in Section 2.

**React / Next.js (JSX/TSX):**
```jsx
<button data-cy="login-submit-btn" onClick={handleSubmit}>Login</button>
```

**Angular (HTML templates):**
```html
<button data-cy="login-submit-btn" (click)="onSubmit()">Login</button>
```

**Vue (SFC templates):**
```html
<button data-cy="login-submit-btn" @click="submit">Login</button>
```

**Plain HTML:**
```html
<button data-cy="login-submit-btn" onclick="submit()">Login</button>
```

The above are format references — adapt to the actual element and framework in use.

### 6.6: When the Agent Cannot Modify Application Code

In some cases the agent may not have access to the source files (e.g., testing a third-party app, or the user restricts app modifications). If the agent cannot add `data-cy` attributes:

1. Fall back to the selector priority in 6.1 — use `data-testid`, `aria-`, or stable IDs if available
2. If no stable selectors exist, inform the user:
   > "The element `<description>` does not have a stable test selector. I recommend adding `data-cy="<suggested-name>"` to `<file-path>`. Would you like me to add it, or should I use an alternative selector?"
3. Only use a lower-priority selector (class, ID) if the user explicitly approves

Only proceed to Section 7 when selectors are determined for all elements the test needs.

---

## Section 7: Cypress Command Patterns

This section defines the correct usage of common Cypress commands. All examples illustrate the **pattern only** — adapt commands, selectors, URLs, and values to the actual test being written.

### 7.1: Navigation

```typescript
// Relative path — uses baseUrl from cypress.config
cy.visit('/login')

// With custom timeout for slow-loading pages
cy.visit('/dashboard', { timeout: 10000 })
```

Always use relative paths with `cy.visit()`. The `baseUrl` from `cypress.config.*` handles the domain. Never hardcode full URLs in tests.

### 7.2: Element Interaction

```typescript
// Typing
cy.get('[data-cy="email-input"]').type('user@example.com')

// Clearing then typing
cy.get('[data-cy="email-input"]').clear().type('new@example.com')

// Clicking
cy.get('[data-cy="submit-btn"]').click()

// Selecting from dropdown
cy.get('[data-cy="country-select"]').select('India')

// Checkbox
cy.get('[data-cy="terms-checkbox"]').check()
cy.get('[data-cy="terms-checkbox"]').uncheck()

// File upload
cy.get('[data-cy="file-input"]').selectFile('cypress/fixtures/document.pdf')
```

### 7.3: Assertions

```typescript
// Visibility
cy.get('[data-cy="welcome-msg"]').should('be.visible')
cy.get('[data-cy="error-msg"]').should('not.exist')

// Text content
cy.get('[data-cy="title"]').should('contain.text', 'Dashboard')
cy.get('[data-cy="title"]').should('have.text', 'Welcome to Dashboard')

// Element count
cy.get('[data-cy="list-item"]').should('have.length', 5)

// Value (for inputs)
cy.get('[data-cy="email-input"]').should('have.value', 'user@example.com')

// URL
cy.url().should('include', '/dashboard')
cy.url().should('eq', 'http://localhost:3000/dashboard')

// CSS / attributes
cy.get('[data-cy="submit-btn"]').should('be.disabled')
cy.get('[data-cy="submit-btn"]').should('have.attr', 'type', 'submit')
cy.get('[data-cy="alert"]').should('have.class', 'alert-danger')

// Chained assertions
cy.get('[data-cy="user-card"]')
  .should('be.visible')
  .and('contain.text', 'John')
```

**Choosing the right assertion:**
- Use `should('not.exist')` when the element should not be in the DOM at all
- Use `should('not.be.visible')` when the element is in the DOM but hidden
- Use `should('contain.text', ...)` for partial text match
- Use `should('have.text', ...)` for exact text match

### 7.4: Waiting — Network-Based Only

**Never use time-based waits:**
```typescript
// WRONG — never do this
cy.wait(2000)
cy.wait(5000)
```

**Always use intercepts for network-dependent actions:**
```typescript
// Set up intercept BEFORE the action that triggers the request
cy.intercept('POST', '/api/login').as('loginRequest')

// Perform the action
cy.get('[data-cy="submit-btn"]').click()

// Wait for the specific request
cy.wait('@loginRequest')

// Then assert on the result
cy.url().should('include', '/dashboard')
```

### 7.5: API Intercepts

Intercepts are used for two purposes: **stubbing** (fake responses) and **spying** (observing real requests).

**Stubbing — replace API response with test data:**
```typescript
cy.intercept('GET', '/api/users', { fixture: 'users.json' }).as('getUsers')

// Or with inline data
cy.intercept('GET', '/api/users', {
  statusCode: 200,
  body: [{ id: 1, name: 'John' }]
}).as('getUsers')

// Stub an error response
cy.intercept('POST', '/api/login', {
  statusCode: 401,
  body: { message: 'Invalid credentials' }
}).as('loginFailed')
```

**Spying — let the request pass through, observe it:**
```typescript
cy.intercept('POST', '/api/login').as('loginRequest')
cy.get('[data-cy="submit-btn"]').click()

// Assert on the request that was sent
cy.wait('@loginRequest').then((interception) => {
  expect(interception.request.body).to.have.property('email')
})

// Assert on the response that came back
cy.wait('@loginRequest').its('response.statusCode').should('eq', 200)
```

**Rules for intercepts:**
- Always set up `cy.intercept()` **before** the action that triggers the request
- Always give intercepts an alias with `.as('name')` — use descriptive names
- Use stubbing when you want to control test data or test error scenarios
- Use spying when you want to verify the app sends correct requests
- **Only intercept APIs your test is directly asserting on.** Do not intercept every API the page calls. Unrelated intercepts add noise, and if any unrelated API is slow or fails, it will timeout or break the test for reasons outside what you're testing.

### 7.5.1: Intercept Timing with Auth Flows

If the test requires login or authentication before testing the actual feature, **set up test-specific intercepts after the auth step completes — not before.**

Login and auth flows trigger their own API calls (token requests, user profile fetches, session validation, etc.). If you set up intercepts before login, those auth-related calls can match and consume your intercepts, leaving the actual feature's API calls unmatched.

**Wrong — intercepts before login:**
```typescript
beforeEach(() => {
  // Intercept set up first
  cy.intercept('GET', '/api/users').as('getUsers')

  // Login triggers its own API calls — may consume the intercept above
  cy.login('user@example.com', 'password123')
  cy.visit('/dashboard')
})

it('shows the user list', () => {
  // This wait may fail — the intercept was already consumed during login
  cy.wait('@getUsers')
  cy.get('[data-cy="user-list"]').should('be.visible')
})
```

**Correct — intercepts after login:**
```typescript
beforeEach(() => {
  // Login first — let all auth API calls finish
  cy.login('user@example.com', 'password123')
  cy.visit('/dashboard')
})

it('shows the user list', () => {
  // Set up intercept AFTER auth is done, BEFORE the action that triggers it
  cy.intercept('GET', '/api/users').as('getUsers')
  cy.get('[data-cy="refresh-btn"]').click()

  cy.wait('@getUsers')
  cy.get('[data-cy="user-list"]').should('be.visible')
})
```

**Rule:** Auth/login is setup. Intercepts are part of the test. Keep them separate. If a page loads data on visit (not triggered by a user action), set up the intercept *after* login but *before* `cy.visit()`:

### 7.6: Working with `cy.then()` and Cypress Chains

Cypress commands are asynchronous and chainable. Do not mix them with synchronous JavaScript logic incorrectly.

**Correct — staying in the Cypress chain:**
```typescript
cy.get('[data-cy="user-count"]').invoke('text').then((text) => {
  const count = parseInt(text)
  expect(count).to.be.greaterThan(0)
})
```

**Wrong — breaking the chain with async/await or raw Promises:**
```typescript
// DO NOT do this — Cypress commands are not Promises
const text = await cy.get('[data-cy="user-count"]').invoke('text')
```

### 7.7: Version-Specific Pattern Compatibility

The Cypress version was read during pre-flight (Section 1, Step 1.1). Use it here. Before writing any test that uses the patterns below, cross-check the installed version against this table. If a pattern is marked broken, use the listed alternative instead.

| Pattern | Broken in | Why | Use instead |
|---------|-----------|-----|-------------|
| `cy.spy()` or `cy.stub()` inside `onBeforeLoad` option of `cy.visit()` | Cypress 13+ | `onBeforeLoad` runs before the page's JS executes. Spying on `window` methods before they exist is unreliable — the spy may be set up on a stale window object that gets replaced when the page loads. | Use `cy.intercept()` to observe/stub network calls. For window method stubs, use `cy.window().then(win => cy.stub(win, 'methodName'))` after visit. |
| `cy.route()` | Cypress 6+ | Removed. | Use `cy.intercept()`. |
| `cy.server()` | Cypress 6+ | Removed along with `cy.route()`. | Not needed — `cy.intercept()` works without it. |

**Rule:** If the installed Cypress version is 13 or higher, never write `cy.spy()` or `cy.stub()` inside `onBeforeLoad`. If the installed version is 6 or higher, never use `cy.route()` or `cy.server()`.

Only proceed to Section 8 when command patterns are understood.

---

## Section 8: Custom Commands

Custom commands encapsulate reusable sequences of Cypress actions. This section defines when to create them, how to register them, and how to avoid duplication.

### 8.1: When to Create a Custom Command

Create a custom command **only** when:
- The same sequence of actions is used in **3 or more tests**
- The sequence involves multiple steps (not a single `cy.get().click()`)

**Do not create a custom command for:**
- A one-off action used in a single test
- A simple single-step action (wrapping `cy.get('[data-cy="btn"]').click()` in a command adds no value)
- Something that already exists — check the custom commands list from pre-flight (Section 1, Step 1.3) first

### 8.2: Where to Register

| File | Purpose |
|------|---------|
| `cypress/support/commands.ts` (or `.js`) | Define all custom commands here |
| `cypress/support/e2e.ts` (or `.js`) | Must import `commands.ts` — verify this import exists |

If `commands.ts` does not exist, create it and add the import in `e2e.ts`:
```typescript
// cypress/support/e2e.ts
import './commands'
```

If `commands.ts` already exists, append new commands to it. Do not create separate files per command.

### 8.3: Command Pattern

The code below illustrates the structure — adapt the command name, parameters, and steps to the actual reusable action.

```typescript
// cypress/support/commands.ts

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.visit('/login')
  cy.get('[data-cy="email-input"]').type(email)
  cy.get('[data-cy="password-input"]').type(password)
  cy.get('[data-cy="submit-btn"]').click()
  cy.url().should('include', '/dashboard')
})
```

**Rules:**
- Command name should be descriptive: `login`, `createUser`, `addToCart` — not `doAction` or `helper1`
- Accept parameters for values that change between uses (email, password, product name, etc.)
- Include a final assertion inside the command to confirm the action succeeded
- Keep commands focused — one command does one thing

### 8.4: TypeScript Declarations

If the project uses TypeScript, add type declarations so the command is recognized by the compiler and IDE.

Add to `cypress/support/commands.ts` (at the top) or a separate `cypress.d.ts` file:

```typescript
declare namespace Cypress {
  interface Chainable {
    login(email: string, password: string): Chainable<void>
  }
}
```

If the project does not use TypeScript, skip this step.

### 8.5: Using Custom Commands in Tests

```typescript
// In any test file
cy.login('user@example.com', 'password123')

// Then continue with test-specific actions
cy.get('[data-cy="dashboard-title"]').should('be.visible')
```

### 8.6: Before Creating — Check for Duplicates

Before adding a new custom command, the agent must:

1. Review the custom commands list from pre-flight (Section 1, Step 1.3)
2. Check if a similar command already exists — same purpose, different name, or partial overlap
3. If a similar command exists:
   - **Exact match:** Use the existing command as-is
   - **Partial overlap:** Extend the existing command with optional parameters instead of creating a new one — but only if the extension is backward-compatible
   - **Different purpose:** Create the new command

Never create a command that duplicates an existing one.

Only proceed to Section 9 when custom commands (if needed) are planned.

---

## Section 9: Anti-Patterns

This section lists common mistakes that lead to flaky, slow, or unmaintainable tests. The agent must never introduce these patterns. If any are detected in existing tests, do not replicate them in new tests.

### 9.1: Time-Based Waiting

**Wrong:**
```typescript
cy.wait(2000)
cy.wait(5000)
```

**Why it's bad:** Slows down tests unnecessarily on fast environments. Still flaky on slow environments because the wait may not be long enough.

**Instead:** Use `cy.intercept()` + `cy.wait('@alias')` for network requests. Use `.should()` assertions for DOM changes — Cypress automatically retries assertions until they pass or timeout.

### 9.2: Shared State Between Tests

**Wrong:** One `it` block creates data that another `it` block reads.

**Why it's bad:** Tests become order-dependent. If one test fails or is skipped, all downstream tests break.

**Instead:** Each `it` block sets up its own state via `beforeEach`, fixtures, or API intercepts. See Section 5.3.

### 9.3: Brittle Selectors

**Wrong:**
```typescript
cy.get('div > form > div:nth-child(2) > input')
cy.get('.sc-bdfBwQ.iKqMTH')
```

**Why it's bad:** Breaks on any DOM restructure or style change.

**Instead:** Use `data-cy` attributes. See Section 6.

### 9.4: Hardcoded URLs

**Wrong:**
```typescript
cy.visit('http://localhost:3000/login')
cy.request('http://localhost:8080/api/users')
```

**Why it's bad:** Breaks when running against different environments (staging, CI, production).

**Instead:** Use relative paths with `cy.visit('/login')` — `baseUrl` from config handles the domain. For API calls, use relative paths or environment variables.

### 9.5: Conditional Testing

**Wrong (non-deterministic state):**
```typescript
cy.get('body').then(($body) => {
  if ($body.find('[data-cy="popup"]').length) {
    cy.get('[data-cy="popup-close"]').click()
  }
})
```

**Why it's bad:** Tests must be deterministic. If the popup sometimes appears and sometimes doesn't, the test is not reliably verifying anything.

**Instead:** Control the state so the test knows exactly what to expect. Use intercepts to control API responses. Use `beforeEach` to set up a known starting state.

**Wrong (if/else where one branch always passes):**
```typescript
cy.get('[data-cy="hero"]').then(($el) => {
  const bg = $el.css('background-image')
  if (bg && bg !== 'none') {
    // check image URL pattern
    expect(bg).to.match(/url\(.*\.jpg\)/)
  } else {
    // check fallback color
    expect($el.css('background-color')).to.eq('rgb(0, 0, 0)')
  }
})
```

**Why it's bad:** One of the two branches will always pass. If the image URL is malformed but the fallback color is correct (or vice versa), the test still passes because it only checks the branch that matches. The test can never fail — it is not protecting against anything.

**Instead:** Write separate, deterministic tests — each with a single assertion path that can independently fail:
```typescript
it('displays the background image', () => {
  cy.get('[data-cy="hero"]')
    .should('have.css', 'background-image')
    .and('match', /url\(.*\.jpg\)/)
})

it('has a fallback background color when image fails to load', () => {
  cy.intercept('**/*.jpg', { statusCode: 404 })
  cy.visit('/')
  cy.get('[data-cy="hero"]')
    .should('have.css', 'background-color', 'rgb(0, 0, 0)')
})
```

**Rule:** Every test must have a single, deterministic assertion path. If a test contains if/else logic where one branch always passes, split it into separate tests that each verify one scenario.

### 9.6: Testing Implementation Details

**Wrong:** Asserting on internal component state, Redux store values, or private function return values.

**Why it's bad:** Breaks on any refactor even if user-visible behavior is unchanged.

**Instead:** Test what the user sees — visible text, navigation, element states (enabled/disabled/visible/hidden), URL changes.

### 9.7: One Giant Test File

**Wrong:** A single test file with 30+ `it` blocks covering every feature.

**Why it's bad:** Slow to run, hard to debug, impossible to run a subset of tests.

**Instead:** One file per feature or feature-aspect. Keep files focused — see Section 4.

### 9.8: Assertions Inside `beforeEach`

**Wrong:**
```typescript
beforeEach(() => {
  cy.visit('/dashboard')
  cy.get('[data-cy="title"]').should('have.text', 'Dashboard')  // assertion in setup
})
```

**Why it's bad:** `beforeEach` is for setup, not verification. If this assertion fails, the error message is misleading — it looks like a setup failure, not a test failure.

**Instead:** Keep `beforeEach` for navigation and state setup only. Put assertions in `it` blocks.

### 9.9: Breaking the Cypress Chain

**Wrong:**
```typescript
const text = await cy.get('[data-cy="title"]').invoke('text')
```

**Why it's bad:** Cypress commands are not standard Promises. Using `async/await` with them breaks the command queue.

**Instead:** Use `.then()` to access values. See Section 7.6.

### 9.10: Unnecessary Cleanup in `afterEach`

**Wrong:**
```typescript
afterEach(() => {
  cy.get('[data-cy="logout-btn"]').click()
  cy.clearCookies()
  cy.clearLocalStorage()
})
```

**Why it's bad:** Cypress automatically clears cookies, local storage, and session storage between tests. Manual cleanup is redundant and slows down the suite.

**Instead:** Let Cypress handle cleanup. Only use `afterEach` if you need to clean up external state (e.g., delete test data via API) that Cypress doesn't handle automatically.

### 9.11: Blanket API Interception

**Wrong:**
```typescript
beforeEach(() => {
  cy.intercept('GET', '/api/users').as('getUsers')
  cy.intercept('GET', '/api/notifications').as('getNotifications')
  cy.intercept('GET', '/api/settings').as('getSettings')
  cy.intercept('GET', '/api/analytics').as('getAnalytics')
  cy.visit('/dashboard')
})

it('shows the user list', () => {
  // Waiting on ALL intercepted APIs — test only cares about users
  cy.wait(['@getUsers', '@getNotifications', '@getSettings', '@getAnalytics'])
  cy.get('[data-cy="user-list"]').should('be.visible')
})
```

**Why it's bad:** The test is about the user list, but it waits on every API the page calls. If the analytics or notifications API is slow, the test times out — for a reason completely unrelated to what it's testing. This is a major source of flaky tests.

**Instead:** Only intercept and wait on the API that your test is asserting on.
```typescript
beforeEach(() => {
  cy.visit('/dashboard')
})

it('shows the user list', () => {
  cy.intercept('GET', '/api/users').as('getUsers')
  cy.get('[data-cy="refresh-btn"]').click()

  cy.wait('@getUsers')
  cy.get('[data-cy="user-list"]').should('be.visible')
})
```

**Rule:** One test, one concern. Intercept only what the test needs to verify. Let other APIs resolve on their own.

### 9.12: Hardcoded Values Against Live API Responses

**Wrong:**
```typescript
// API is NOT stubbed — real data comes back
cy.intercept('GET', '/api/kpis').as('getKpis')
cy.visit('/dashboard')
cy.wait('@getKpis')

// Hardcoded value that assumes specific live data
cy.get('[data-cy="kpi-value"]').should('have.text', '42.5')
```

**Why it's bad:** Live API data changes independently of code — new records, recalculations, different date ranges. The test passes today and fails tomorrow even though the code is perfectly fine.

**Instead — spy on the real API, capture the response, assert the UI reflects it:**
```typescript
cy.intercept('GET', '/api/kpis').as('getKpis')
cy.visit('/dashboard')

cy.wait('@getKpis').then((interception) => {
  const value = interception.response.body.data.energy
  cy.get('[data-cy="kpi-value"]').should('contain.text', String(value))
})
```

**When to stub instead:** Use stubs (fixtures) when testing error states, empty states, edge case data, or when the API is unreliable/slow in the test environment.

**Rule:** Default to spy + capture + assert. Use stubs for controlled scenarios. Never hardcode expected values against a live unstubbed API.

### 9.13: Computed Style Assertions

**Wrong:**
```typescript
cy.get('p').filter((index, el) => {
  const style = window.getComputedStyle(el)
  return style.fontWeight === '700' && style.fontSize === '28px'
})
```

**Why it's bad:** Computed style values vary across browsers (`'700'` vs `'bold'`), operating systems, DPI/zoom levels, and CI environments. Tests pass locally but fail in CI — or vice versa.

**Instead:** Assert on CSS classes, data attributes, or semantic properties that the component applies:
```typescript
// Assert on the class the component uses
cy.get('[data-cy="kpi-title"]').should('have.class', 'kpi-title--bold')

// Or use Cypress have.css only for truly stable properties
cy.get('[data-cy="kpi-title"]').should('have.css', 'font-weight', '700')
```

**Rule:** Never use `getComputedStyle()` to find or filter elements. Read the implementation (Section 6.3) to understand what classes or attributes control styles, and assert on those instead.

### 9.14: Guessing Third-Party Component Behavior

**Wrong:** The agent does not know how a date picker, tooltip, or dropdown works, but writes test interaction code anyway — guessing DOM events, internal class names, or multi-step sequences.

**Why it's bad:** Third-party components (MUI, Ant Design, React Select, etc.) have specific interaction contracts — which events they listen to, how their internal DOM is structured, what animations or transitions they use. Guessing leads to tests that work accidentally in one environment and fail in another.

**Instead:**
1. **Read the implementation** (Section 6.3) — check which third-party library is used and how the component is configured
2. **Derive the interaction from the code** — if the component has `onMouseOver`, use `trigger('mouseover')`. If it uses Ant Design's `DatePicker`, understand its DOM structure from the import
3. **If still unclear, ask the user** — "The component uses `<library>` for `<widget>`. How should the test interact with it?"

**Rule:** Never guess how a third-party component works. Read the code first. Ask the user if the code doesn't make it clear.

Only proceed to Section 10 when anti-patterns are understood.

---

## Section 10: Output & Agent Behavior

This section defines how the agent communicates with the user before, during, and after writing tests.

### 10.1: Before Writing Any Test

The agent must present the following to the user before creating any files:

1. **What feature/flow** is being tested
2. **Test category** and why (smoke/regression/integration/functional)
3. **File path** where the test will be placed
4. **Selectors needed** — list of `data-cy` attributes that will be added to the application code (if any)
5. **Complete test code** — show the full test file content

Wait for user confirmation before creating the test file or modifying application code.

**Exception:** If the user explicitly asks the agent to write tests without confirmation (e.g., "just write it", "go ahead"), skip the confirmation step.

### 10.2: While Writing

When creating or modifying files, the agent must:

- **Show each file change separately** — test file creation, `data-cy` attribute additions, custom command additions, fixture file creation
- **Explain what each change does** in one line — not a paragraph, just enough context
- **Never silently modify files** — every change must be visible to the user
- **After editing any file, re-read it and verify the change is complete** — confirm that every affected occurrence was updated, not just the first one. Do not move on until the verification is done.

### 10.3: After Writing

Once the test is created, the agent must provide:

**1. Summary of what was created/modified:**

```
Files created:
  - cypress/e2e/smoke/login.cy.ts (test file)
  - cypress/fixtures/login/valid-credentials.json (fixture)

Files modified:
  - src/components/LoginForm.tsx (added data-cy attributes)
  - cypress/support/commands.ts (added login command)
```

**2. How to run the test:**

```
Run this specific test:
  npx cypress run --spec "cypress/e2e/smoke/login.cy.ts"

Run in interactive mode:
  npx cypress open
```

**3. Missing `data-cy` attributes (if agent could not add them):**

If Section 6.6 applied and the agent could not modify application code, list the attributes the user needs to add manually:

```
Manual action needed — add these data-cy attributes:
  - <input> email field in src/components/LoginForm.tsx → data-cy="login-email-input"
  - <button> submit in src/components/LoginForm.tsx → data-cy="login-submit-btn"
```

If all attributes were added by the agent, skip this.

### 10.4: When the User's Request is Unclear

If the user gives a vague or incomplete request, the agent must ask clarifying questions before writing. Do not guess.

**Ask when:**
- The feature to test is not clear → "Which feature or page should I write tests for?"
- The scope is ambiguous → "Should I cover only the happy path, or edge cases and error states too?"
- The category is unclear → "Should this be a smoke test or a regression test?"
- The application URL or route is unknown → "What URL or route does this feature live on?"

**Do not ask when:**
- The request is clear and specific enough to proceed
- The answer can be determined from the project structure file (Section 2) or existing tests (Section 1)
- The user has already provided the information in their prompt

Keep questions minimal — ask only what's needed, not a checklist of every possible detail.

### 10.5: Handling Multiple Tests in One Request

If the user asks for tests covering multiple features or a full suite:

1. List all test files that will be created with their paths and categories
2. Get user confirmation on the plan before writing any files
3. Create tests one at a time — show each test as it's written
4. Provide a combined summary at the end

Do not dump all test files at once without explanation.
