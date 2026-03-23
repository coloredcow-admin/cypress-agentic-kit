# Cypress Test Scope & Coverage — Runbook

Follow these sections sequentially. The scope step must be completed before writing any Cypress tests.

---

## Section 1: Scope — Decide What to Automate

Before writing any Cypress test, the agent must determine which scenarios from the manual test cases are in scope for automation and which are not.

### Step 1.1: Gather Inputs

The agent requires:

1. **Manual test cases** (mandatory) — the test case document for the feature (from `test-cases/` directory or user-provided path)
2. **Figma design reference** (optional) — for UI context, component names, visual states

If manual test cases are not available, ask the user to provide them or run the `manual-test-generator-agent` first.

### Step 1.2: Extract Scenarios

Read the manual test cases and extract every individual test scenario. Each scenario should be identified by:

- **TC ID** — the test case identifier (e.g., TC-12)
- **Scenario name** — what is being tested (e.g., "Login with valid credentials")
- **Category** — the type of test (e.g., smoke, regression, functional)
- **Flow type** — what kind of interaction it involves (see scope rules below)

### Step 1.3: Categorize Each Scenario Against Scope Rules

For each extracted scenario, apply the following rules in order. The first matching rule wins.

---

#### Category 1: ALWAYS Automate

If the scenario involves any of the following, it is **IN SCOPE**:

| # | Flow Type | Examples |
|---|-----------|---------|
| 1 | Login / logout / session management | Login with valid credentials, session expiry, logout redirect |
| 2 | Core CRUD operations | Create a record, view details, edit fields, delete entry |
| 3 | Form validations on critical flows | Required field empty, invalid email format, boundary values |
| 4 | Navigation and routing | Can user reach all main pages, breadcrumb links work |
| 5 | Role-based access control | Admin sees admin panel, regular user does not |
| 6 | API response handling | Success response renders data, error response shows message, empty state displayed |
| 7 | Protected route redirects | Unauthenticated user redirected to login |
| 8 | Search / filter / sort on data-heavy pages | Search returns correct results, filter narrows list, sort changes order |

**Decision:** IN SCOPE — automate this scenario.

---

#### Category 2: AUTOMATE IF Critical + Stable

If the scenario involves any of the following, it is **CONDITIONALLY IN SCOPE**. The agent proposes it and flags it for user validation.

| # | Flow Type | Condition |
|---|-----------|-----------|
| 1 | Multi-step workflows (checkout, onboarding, wizards) | Only if the flow is stable — not actively being redesigned |
| 2 | Negative / error cases on non-critical flows | Only on flows the team considers important |
| 3 | Pagination, infinite scroll, lazy loading | Only if it is a core user experience |
| 4 | File upload — trigger and success validation | Can validate the trigger and API response, not file contents |
| 5 | Notification / toast / alert messages | Only if they confirm critical actions (payment success, delete confirmation) |
| 6 | State persistence (page refresh, browser back) | Only for critical flows where data loss means bad UX |

**Decision:** PROPOSE — include in scope proposal with a flag, ask user to confirm or skip.

---

#### Category 3: NEVER Automate

If the scenario involves any of the following, it is **OUT OF SCOPE**:

| # | Flow Type | Reason |
|---|-----------|--------|
| 1 | CAPTCHA / reCAPTCHA | Third-party widget, designed to block automation |
| 2 | Third-party authentication (Google SSO, Facebook login) | Cross-origin iframe restrictions, outside project control |
| 3 | Third-party payment iframes (Stripe, PayPal) | Cross-origin, cannot interact reliably |
| 4 | Email / SMS / OTP verification | Lives outside the browser, Cypress cannot access |
| 5 | Visual pixel-level correctness | Cypress checks DOM structure, not visual rendering |
| 6 | Performance / load testing | Cypress is not built for performance measurement |
| 7 | Mobile native behavior (push notifications, camera, GPS) | Cypress runs in a browser, not on native devices |
| 8 | PDF / file content validation (beyond download trigger) | Can verify download triggers, cannot verify file contents visually |
| 9 | Full accessibility audit | Basic checks possible via plugins, but full audit needs specialized tools and human review |

**Decision:** OUT OF SCOPE — mark as "manual only" with the reason.

---

#### Category 4: SKIP for Now (Conditional)

If the scenario involves any of the following, it is **OUT OF SCOPE temporarily**:

| # | Flow Type | When to Revisit |
|---|-----------|-----------------|
| 1 | Features actively being redesigned | When UI stabilizes |
| 2 | Experimental / beta features | When promoted to stable |
| 3 | Flows with frequently changing test data | When data strategy is defined |
| 4 | Rarely used admin-only tools | When team decides it is worth the maintenance cost |

**Decision:** SKIP FOR NOW — mark as "deferred" with the reason and revisit condition.

---

#### No Match

If the scenario does not match any of the above categories:

**Decision:** PROPOSE — include in scope proposal with reasoning, ask user to decide.

---

## Section 2: Scope Proposal — Present to User for Validation

After categorizing all scenarios, present the scope proposal to the user in the following format:

### Scope Proposal Format

```
## Scope Proposal: [Feature Name]

**Source:** [Manual test case document path]
**Total scenarios:** [count]
**In scope:** [count] | **Proposed (needs confirmation):** [count] | **Out of scope:** [count]

### IN SCOPE — Will be automated

| TC ID | Scenario | Rule Applied |
|-------|----------|-------------|
| TC-01 | Login with valid credentials | ALWAYS: Login/session flow |
| TC-03 | Create new appointment | ALWAYS: Core CRUD |
| ... | ... | ... |

### PROPOSED — Needs your confirmation

| TC ID | Scenario | Rule Applied | Why flagged |
|-------|----------|-------------|-------------|
| TC-07 | Complete onboarding wizard | CONDITIONAL: Multi-step workflow | Confirm if this flow is stable |
| ... | ... | ... | ... |

### OUT OF SCOPE — Manual only

| TC ID | Scenario | Rule Applied | Reason |
|-------|----------|-------------|--------|
| TC-10 | Verify OTP on login | NEVER: Email/SMS/OTP | Lives outside browser |
| TC-15 | Visual layout on mobile | NEVER: Visual pixel correctness | Cypress checks DOM, not pixels |
| ... | ... | ... | ... |

### DEFERRED — Skipped for now

| TC ID | Scenario | Rule Applied | Revisit When |
|-------|----------|-------------|-------------|
| TC-20 | New dashboard v2 filters | SKIP: Feature being redesigned | When dashboard v2 UI stabilizes |
| ... | ... | ... | ... |
```

### Validation Rules

- Wait for the user to review and respond before proceeding.
- If the user moves a scenario from OUT OF SCOPE to IN SCOPE, accept it — user has final say.
- If the user moves a scenario from IN SCOPE to OUT OF SCOPE, accept it — record the reason.
- If the user approves PROPOSED items, move them to IN SCOPE.
- If the user skips PROPOSED items, move them to OUT OF SCOPE with reason "user decision."
- Once the user confirms, the scope is **finalized**. Proceed to test writing only for finalized IN SCOPE scenarios.

---

## Section 3: Coverage — Track Completeness

After writing Cypress tests for the finalized scope, the agent must check and report coverage.

### Step 3.1: Self-Check

Compare the finalized scope (IN SCOPE scenarios) against the tests actually written:

- For each IN SCOPE scenario, verify a corresponding test exists.
- If a test could not be written (unexpected blocker), note it as a gap with the reason.

### Step 3.2: Update Coverage Report

Update (or create) the coverage report file at `cypress/coverage-report.md` in the project.

The report tracks coverage **per feature**. Each feature entry follows this format:

```markdown
## [Feature Name]

- **Last updated:** [date]
- **Source:** [manual test case document path]
- **Scope decided by:** agent + [user name if available]

| Metric | Count |
|--------|-------|
| Total manual test cases | [count] |
| In scope (approved for automation) | [count] |
| Tests written | [count] |
| Gaps (in scope but not written) | [count] |
| Out of scope (manual only) | [count] |
| Deferred (skipped for now) | [count] |
| **Coverage** | **[written / in scope]%** |

### Automated Scenarios

| TC ID | Scenario | Test File | Status |
|-------|----------|-----------|--------|
| TC-01 | Login with valid credentials | cypress/e2e/smoke/login.cy.ts | Written |
| TC-03 | Create new appointment | cypress/e2e/functional/appointment-create.cy.ts | Written |
| ... | ... | ... | ... |

### Gaps (In Scope but Not Written)

| TC ID | Scenario | Reason |
|-------|----------|--------|
| TC-08 | Bulk delete records | Blocked: bulk API endpoint not yet implemented |

### Out of Scope (Manual Only)

| TC ID | Scenario | Reason |
|-------|----------|--------|
| TC-10 | Verify OTP on login | Lives outside browser |

### Deferred

| TC ID | Scenario | Revisit When |
|-------|----------|-------------|
| TC-20 | New dashboard v2 filters | When dashboard v2 UI stabilizes |
```

### Step 3.3: Commit Report with Tests

The coverage report must be committed in the same PR as the test files. It is not a separate step — it is part of the test writing output.

### Step 3.4: Report Summary to User

After updating the coverage report, present a summary to the user:

```
Coverage Summary: [Feature Name]
- In scope: [X] scenarios
- Tests written: [Y]
- Gaps: [Z] (reasons listed above)
- Coverage: [Y/X]%
```

If coverage is 100%, confirm completion. If there are gaps, explain each gap and ask the user if they want to address them now or defer.

---

## Section 4: Edge Cases

### 4.1: Manual Test Cases Updated After Scope Was Finalized

If the user asks the agent to write tests for a feature that already has a coverage report entry:

1. Read the existing coverage report entry for that feature.
2. Compare the current manual test cases against the scenarios listed in the report.
3. If new scenarios were added to the manual test cases — run scope categorization on only the new scenarios and present an updated scope proposal.
4. If existing scenarios were removed or modified — flag them to the user and ask how to handle (update tests, remove tests, or leave as-is).
5. Update the coverage report after any changes.

### 4.2: User Overrides Agent Scope Decision

If the user moves a scenario from OUT OF SCOPE to IN SCOPE (or vice versa):

- Accept the override — the user has final say.
- Record the override reason in the coverage report under the scenario entry.
- The agent does not re-challenge user overrides in future runs for the same feature. The coverage report serves as the record of decisions.

### 4.3: Scenario Cannot Be Automated Despite Being In Scope

If during test writing the agent discovers a scenario cannot be automated (e.g., depends on a third-party iframe not known at scope time):

1. Do not silently skip it.
2. Move it to Gaps in the coverage report with the reason.
3. Inform the user and ask if they want to move it to OUT OF SCOPE or keep it as a gap to revisit.

### 4.4: Feature Has No Manual Test Cases

If manual test cases do not exist for the feature:

- Do not attempt to define scope without them.
- Point the user to `manual-test-generator-agent` to generate manual test cases first.
- Stop and wait. Do not proceed to test writing.
