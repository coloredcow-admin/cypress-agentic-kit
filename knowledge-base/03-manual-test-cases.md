# Manual Test Case Generation — Runbook

Follow these sections sequentially. Each section must pass before proceeding to the next.

---

## Section 1: Pre-flight

Before generating any test cases, the agent must understand the project and the feature to be tested.

### Step 1.1: Project Structure Awareness

Follow the same project structure detection process defined in `02-test-writing.md`, Section 2:

1. Check if `CLAUDE.md` contains a `project-structure-file:` pointer — if yes, read that file.
2. If no pointer, scan the project for existing structure files (`architecture.md`, `project-structure.md`, `structure.md`, or similar).
3. If no structure file exists, generate one by scanning the project and save it for reuse.

**Purpose:** The agent needs to understand what components, pages, APIs, and features exist in the project so it can generate relevant and complete test cases.

### Step 1.2: Check for Existing Test Case Documents

Scan the project for existing manual test case documents:

- Check `test-cases/` directory if it exists
- Check for markdown files matching patterns like `*-test-cases.md`, `*-tests.md`, `test-plan-*.md`
- Check any path the user specifies

If existing test case documents are found:

- Read them to understand the conventions already in use (naming, numbering, format, structure)
- Follow existing conventions where possible — consistency across the project matters more than KB defaults
- Note the TC numbering to avoid ID conflicts (e.g., if existing docs use TC-1 through TC-45, start new cases from TC-46)

If no existing documents are found:

- Use the KB defaults defined in Section 2
- Proceed without convention constraints

### Step 1.3: Gather Feature Information from the User

Ask the user to provide the following. Present this as a clear checklist:

> To generate test cases, I need the following details:
>
> 1. **Feature name** — e.g., "Energy Consumption KPIs", "User Login", "Search Filters"
> 2. **Requirements or description** — a requirement document, user story, ticket link, or a brief explanation of what the feature does
> 3. **Design reference** (optional) — Figma link, screenshots, or design spec
> 4. **Roles involved** (optional) — if the feature has role-based access, which roles exist and what each can do

**Rules:**
- Do not proceed without at least the feature name and a description
- If the user provides a vague description ("test the dashboard"), ask for specifics — what components are on the dashboard, what data does it show, what interactions are possible
- If the user points to a source file or component instead of a description, read the source code and extract the feature behavior from it — then confirm your understanding with the user before generating cases
- Never guess feature behavior — confirm with the user when unclear

Only proceed to Section 2 when the project structure is understood and feature details are gathered.

---

## Section 2: Test Case Format

This section defines the standard format for every test case and the overall document structure.

### 2.1: Document Structure

Every test case document is organized into 4 sections, each representing a test category:

```
<Feature Name>

1. Functional Test Cases
   1. <Sub-feature / Area>
      (test cases here)

2. UI Test Cases
   1. <Sub-feature / Area>
      (test cases here)

3. Interdependency Test Cases
   1. <Sub-feature / Area>
      (test cases here)

4. Edge Cases
   1. <Sub-feature / Area>
      (test cases here)
```

The above is the structural pattern only — adapt section names and sub-feature groupings to the actual feature being tested.

**Rules:**
- All 4 sections must be present in every document, even if a section has fewer cases
- Sub-features group related test cases within a section — use them to keep the document scannable
- The category of a test case is determined by which section it lives under — do not add a separate "Category" field to individual cases

### 2.2: Individual Test Case Format

Every test case follows this structure:

```
**TC-<number>: <Title>**
- Preconditions: <what must be true before this test>
- Steps:
  1. <concrete action>
  2. <concrete action>
  3. <concrete action>
- Expected Result: <the single outcome this test verifies>
- Test Data: <specific values to use, if any>
```

**Field definitions:**

| Field | What it contains | Required |
|-------|-----------------|----------|
| **Title** | A short, clear description of what is being tested | Yes |
| **Preconditions** | What must be set up or true before running — logged-in state, data loaded, specific page open, role in use | Yes |
| **Steps** | Numbered, concrete actions to perform — each step is a single user action | Yes |
| **Expected Result** | One outcome that determines pass or fail for this test case | Yes |
| **Test Data** | Specific input values, filter selections, or data needed — omit if the test needs no specific data | Only if applicable |

### 2.3: TC Numbering

- Number test cases sequentially across the entire document: TC-1, TC-2, TC-3, ...
- Do not restart numbering per section — a document with 30 cases goes from TC-1 to TC-30
- If existing test case documents exist in the project, continue numbering from where they left off (see Step 1.2)

### 2.4: Writing Quality Rules for Each Field

**Title:**
- Must describe what is being verified, not how
- Good: "KPIs update on time period change"
- Bad: "Click date picker and check values" (this is a step, not a title)

**Preconditions:**
- Must be specific enough that someone unfamiliar with the feature can set up the test
- Include: role/user type, page/screen to be on, data that must exist, any prior actions needed
- Good: "User is logged in as Admin, KPI dashboard is loaded with default filters"
- Bad: "App is open" (too vague)

**Steps:**
- Each step is one user action — click, type, select, navigate, hover, scroll
- Use concrete language: "Click the 'Apply' button" not "Apply the filter"
- Reference UI elements clearly: "Select 'India' from the Country dropdown" not "Choose a country"
- Do not combine multiple actions into one step
- Do not include verifications in steps — those belong in Expected Result

**Expected Result:**
- One clear outcome per test case — this is what determines pass or fail
- Must be observable and verifiable — something a tester can see or measure
- Good: "All KPI values update to reflect data for the selected time period only"
- Bad: "It works correctly" (not verifiable)

**Test Data:**
- Use realistic values, not placeholders like "test123" or "abc@xyz.com"
- If the test needs specific values to be meaningful, list them explicitly
- If the test works with any valid input, omit the Test Data field entirely

Only proceed to Section 3 when the format is understood.

---

## Section 3: Category Definitions & Guidance

This section defines what belongs in each of the 4 test case categories and provides guiding questions to help the agent think through coverage for each.

### 3.1: Functional Test Cases

**What belongs here:** Tests that verify the feature works correctly — inputs produce the right outputs, business logic behaves as specified, data flows as expected.

**Guiding questions to generate cases:**
- Does each input produce the correct output?
- Do filters, search, sorting, and pagination work as specified?
- Does CRUD (create, read, update, delete) work for all entities involved?
- Are calculations and data transformations correct?
- Does the feature respect access control — what can each role do and not do?
- Do default values load correctly on first visit?
- Does the feature handle all specified states (active, inactive, pending, etc.)?

**Sub-feature grouping examples (for reference — adapt to the actual feature):**
- Feature Availability & Access Control
- Filter Behavior
- Data Rendering & Calculations
- CRUD Operations
- State Handling

### 3.2: UI Test Cases

**What belongs here:** Tests that verify the visual presentation, layout, responsiveness, and user-facing behavior of the feature — how it looks and feels, not what it computes.

**Guiding questions to generate cases:**
- Does the layout match the design spec / Figma?
- Are labels, titles, and text content correct?
- Do loading states show while data is being fetched?
- Do empty states show when there is no data?
- Are error messages clear and user-friendly when something fails?
- Is the feature usable on different screen sizes (if responsive)?
- Do interactive elements give visual feedback (hover, active, disabled states)?
- Are tooltips, modals, and popovers displayed correctly?
- Is scrolling behavior correct when content overflows?

**Sub-feature grouping examples:**
- Layout & Visual Consistency
- Loading & Empty States
- Error Handling UI
- Responsive Behavior
- Interactive Feedback

### 3.3: Interdependency Test Cases

**What belongs here:** Tests that verify how this feature interacts with other features or components — shared state, cascading effects, navigation, and data consistency across the application.

**Guiding questions to generate cases:**
- When this feature changes state (e.g., a filter is applied), what other components should update?
- What other components should NOT be affected?
- Does navigation away and back preserve the feature's state?
- Do shared filters or global state changes propagate correctly?
- Does data stay consistent between this feature and related features (e.g., a list page and its detail page)?
- Does switching tabs, sections, or views within the feature maintain state?

**Sub-feature grouping examples:**
- Global Filter Consistency
- Cross-Component Impact
- Navigation & State Retention
- Data Consistency Across Views

### 3.4: Edge Cases

**What belongs here:** Tests that verify the feature handles unusual, extreme, or unexpected conditions — the scenarios that typically break in production.

**Guiding questions to generate cases:**
- What happens with empty data (no records, no results, blank inputs)?
- What happens with large data (thousands of records, very long text, maximum values)?
- What happens with boundary values (minimum, maximum, zero, negative)?
- What happens with special characters in inputs?
- What happens when APIs fail or return errors?
- What happens with rapid repeated actions (double-click, fast filter switching)?
- What happens with concurrent actions (multiple tabs, simultaneous updates)?
- What happens when required data is missing or null?
- What happens when the user does things in an unexpected order?

**Sub-feature grouping examples:**
- Empty & No-Data States
- Boundary Values
- Error & Failure Handling
- Concurrency & Rapid Actions
- Unexpected User Behavior

### 3.5: Category Decision Rules

When deciding which category a test case belongs to:

- If it verifies **business logic, data correctness, or feature behavior** → Functional
- If it verifies **how something looks, reads, or is presented** → UI
- If it verifies **how this feature affects or is affected by other features** → Interdependency
- If it verifies **what happens under unusual or extreme conditions** → Edge Cases

If a test case could fit in multiple categories, place it in the **most specific** one. For example: "Error message appears when API fails" could be UI (error display) or Edge Cases (API failure). Place it in Edge Cases — the trigger condition (API failure) is the defining characteristic, not the error display.

Do not duplicate test cases across sections. One case, one section.

Only proceed to Section 4 when categories are understood and the agent knows what kind of cases to generate for each.

---

## Section 4: Writing Rules

This section defines rules the agent must follow when generating test cases. These rules prevent common quality problems — vague cases, duplicated coverage, out-of-scope cases, and bloated documents.

### 4.1: Scope Control

- Only generate test cases for the feature the user described — do not wander into testing adjacent features
- If the feature depends on another feature (e.g., a filter that affects multiple pages), test the interaction in the Interdependency section but do not write full test cases for the other feature
- If the user asks for cases for multiple features, generate a separate document for each feature

### 4.2: No Duplication

- Each scenario is tested once, in one section
- If a similar verification appears in Functional and UI, decide which category owns it using the rules in Section 3.5
- If the same precondition + steps + expected result would appear in two cases, they are the same case — merge them

### 4.3: Coverage Completeness

- Walk through every guiding question in Section 3 for each category — do not skip categories
- For each sub-feature the user described, generate cases across all 4 categories where applicable
- If a category has no applicable cases for the feature (rare but possible), include the section header with a note: "No applicable cases for this feature"
- Aim for thorough coverage, not exhaustive coverage — 30 well-written cases are better than 80 padded ones

### 4.4: Consistency

- Use the same terminology the user used in their feature description — do not rename UI elements, labels, or concepts
- If the feature calls it a "depot," the test case calls it a "depot" — not "location" or "site"
- Keep the same level of detail across all test cases — do not write 5-step detailed cases for one area and vague one-liners for another

### 4.5: One Test, One Purpose

- Each test case verifies exactly one thing
- If a test case has steps that verify two independent behaviors, split it into two cases
- Exception: if two checks are inseparable parts of the same flow (e.g., "submit form" and "success message appears"), they belong in one case

### 4.6: Do Not Generate

- Test cases for features the user did not describe
- Test cases that require backend/database verification (unless the user explicitly includes API testing in scope)
- Performance benchmarks (e.g., "page should load in under 3 seconds") unless the user specifically asks for them
- Security test cases (e.g., SQL injection, XSS) unless the user specifically asks for them
- Test cases that are impossible to verify manually (e.g., "verify memory usage stays below 100MB")

Only proceed to Section 5 when writing rules are understood.

---

## Section 5: Automation-Readiness

These test cases will later be consumed by the `cypress-test-writer-agent` (Phase 2) to generate automated Cypress tests. This section defines rules that make the manual cases easy to convert to automation — without making them harder for a manual tester to use.

### 5.1: Why This Matters

The manual test case document serves two consumers:
1. **Manual QA** — executes cases during feature testing
2. **Automation agent** — reads cases later to generate Cypress e2e tests

Writing cases with automation in mind upfront means the automation agent can pick them up without guessing or asking for clarification. This saves a full round of re-interpretation later.

### 5.2: Steps Must Reference UI Elements Clearly

Steps should name the UI element being interacted with — not describe the action abstractly.

| Automation-ready | Not automation-ready |
|-----------------|---------------------|
| Click the 'Apply' button | Apply the filter |
| Type "admin@company.com" in the Email input field | Enter email |
| Select 'India' from the Country dropdown | Choose a country |
| Hover over the location filter label | Check the tooltip |

The automation agent translates steps into Cypress commands (`cy.get`, `cy.click`, `cy.type`). If the step says "Apply the filter," the agent doesn't know which element to target. If it says "Click the 'Apply' button," the agent knows exactly what to look for.

**Rule:** Every step that involves a UI interaction must name the element (button, input, dropdown, link, checkbox, etc.) and its label or identifier.

### 5.3: Expected Results Must Be Assertable

Expected results should describe something a test can verify programmatically — visible text, element state, URL change, element presence/absence.

| Assertable | Not assertable |
|-----------|---------------|
| Success message "Changes saved" is displayed below the form | It saves correctly |
| KPI card shows value "1,250 kWh" | KPI shows correct value |
| URL changes to `/dashboard/reports` | User is redirected |
| Submit button is disabled | Button state changes |
| Error banner with text "Invalid date range" is visible | Error is shown |

**Rule:** If a manual tester can point to it on screen, it's assertable. If it requires interpretation or subjective judgment ("page looks correct," "performance is acceptable"), it's not automation-ready — and that's fine for purely manual cases, but flag them clearly.

### 5.4: Preconditions Must State Data and Auth Explicitly

Automation needs to set up state before running the test. Preconditions should include:

- **Auth state:** which user role is logged in (e.g., "Logged in as Admin")
- **Page state:** which page/screen is loaded (e.g., "On the KPI dashboard page")
- **Data state:** what data must exist (e.g., "At least 3 locations with KPI data exist for the selected time period")

Vague preconditions like "App is open" force the automation agent to guess the setup. Explicit preconditions translate directly to `beforeEach` blocks, fixtures, and API seeds.

### 5.5: Cases That Are Not Automatable

Some manual test cases are inherently visual or subjective and cannot be automated with Cypress. This is expected. Examples:

- "Layout matches Figma design"
- "Typography and spacing are consistent"
- "Animation feels smooth"

Do not avoid writing these cases — they are valid manual tests. But do not force them into an automation-friendly format either. Write them naturally. The automation agent will skip cases it cannot convert and will not attempt to automate subjective checks.

Only proceed to Section 6 when automation-readiness rules are understood.

---

## Section 6: Output & Agent Behavior

This section defines how the agent presents, delivers, and manages the generated test cases.

### 6.1: Show Before Writing

The agent must present the complete test case document to the user before creating any file.

**Process:**
1. Generate the full document with all 4 sections
2. Show the complete document in the conversation
3. Wait for user confirmation ("looks good," "go ahead," "approved")
4. Only then create the file

**Exception:** If the user explicitly says "just generate it" or "skip review," create the file directly.

### 6.2: File Placement

Save the generated test case document as a markdown file:

- **Directory:** `test-cases/` in the project root (create if it doesn't exist)
- **File name:** `<feature-name>-test-cases.md` in kebab-case
- **Examples:** `energy-consumption-kpis-test-cases.md`, `user-login-test-cases.md`, `search-filters-test-cases.md`

If the user specifies a different location or naming convention, follow their preference.

### 6.3: Summary After Generation

After the test case document is created, provide a summary:

> **Test Case Summary — <Feature Name>**
>
> | Category | Count |
> |----------|-------|
> | Functional | X |
> | UI | X |
> | Interdependency | X |
> | Edge Cases | X |
> | **Total** | **X** |
>
> File: `test-cases/<feature-name>-test-cases.md`

### 6.4: Handling Multiple Features

If the user asks for test cases for multiple features in one session:

1. Confirm the list of features with the user
2. Generate one document per feature — do not combine features into a single document
3. Process features one at a time — show, confirm, create, then move to the next
4. Provide a combined summary at the end listing all documents created

### 6.5: Updating Existing Test Cases

If a test case document already exists for the feature:

1. Read the existing document first
2. Ask the user what they want:
   - **Add new cases** — append new cases to the existing document, continue TC numbering from where it left off
   - **Regenerate** — replace the entire document with a fresh set of cases
   - **Update specific section** — modify only the section the user specifies
3. Do not silently overwrite an existing document — always ask

### 6.6: Idempotent Behavior

- If the user asks for test cases for a feature that already has a document, do not create a duplicate file — follow Step 6.5
- If the user runs the agent again without specifying a feature, ask what they want to do — do not regenerate everything
- Never redo what is already done

### 6.7: What the Agent Must NOT Do

- Do not generate test cases without understanding the feature first (Section 1 must complete)
- Do not create files without showing the document to the user first (unless they opt out)
- Do not modify application code — this agent only produces test case documents
- Do not run any Cypress commands — this agent writes documentation, not automation
- Do not guess feature behavior — ask when unclear
- Do not combine multiple features into one document
