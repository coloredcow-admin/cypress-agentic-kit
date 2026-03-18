## Cypress Agentic Kit — How to Use It in Your Project

### What is this?

An AI-assisted testing toolkit. You paste prompts into Claude Code, and AI agents set up Cypress, generate test cases, write automated tests, and configure CI — all inside your project.

**Your only job is to paste the prompt. The agent will ask you for any input it needs.**

---

### The Flow

```
┌─────────────────────────────────────────────────────────────┐
│                       YOUR PROJECT                          │
│                                                             │
│  Step 1-3 │ BOOTSTRAP + CYPRESS SETUP                       │
│           │ Cypress gets installed & configured             │
│           │ Prompt: prompts/bootstrap.md                    │
│           │ Input: None — agent handles everything          │
│           ↓                                                 │
│  ┌──────────────────── TESTING ────────────────────────┐    │
│  │                                                     │    │
│  │  Step 4a │ MANUAL TEST CASES                        │    │
│  │          │ AI generates structured test case docs   │    │
│  │          │ Prompt: manual-test-generator.md         │    │
│  │          │ Input: Requirements doc / feature spec   │    │
│  │          │        (agent will ask you for this)     │    │
│  │          ↓                                          │    │
│  │  Step 4b │ AUTOMATED TEST WRITING                   │    │
│  │          │ AI writes Cypress E2E tests              │    │
│  │          │ Prompt: test-writer.md                   │    │
│  │          │ Input: Manual test cases from Step 4a    │    │
│  │          │   + app source code (read by agent)      │    │
│  │          │        (agent will ask which feature)    │    │
│  │                                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│           ↓                                                 │
│  Step 5   │ CI SETUP                                        │
│           │ CI pipeline gets created                        │
│           │ Prompt: prompts/ci-setup.md                     │
│           │ Input: None — auto-detects your CI platform     │
│           ↓                                                 │
│  Step 6   │ RUN IN CI                                       │
│           │ Add "ready-to-test" label on PR → tests run     │
│           │ Input: Just the label                           │
│                                                             │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│  SYNC     │ Update kit to latest version anytime            │
│           │ Prompt: prompts/sync.md                         │
└─────────────────────────────────────────────────────────────┘
```

---

### What Each Step Does

#### Steps 1–3: Bootstrap + Cypress Setup (One-Time)

> **Prompt:** `prompts/bootstrap.md` · **Input:** None

Paste the prompt. The agent installs kit files into your project, detects your environment, installs Cypress, scaffolds the folder structure, and verifies everything works.

#### Step 4a: Manual Test Case Generation

> **Prompt:** `prompts/manual-test-generator.md` · **Input:** Requirements doc / feature spec *(agent will ask you)*

Paste the prompt. The agent asks you for your feature's requirements or acceptance criteria. It then generates a structured test case document covering Functional, UI, Interdependency, and Edge Case scenarios — saved to `test-cases/`.

#### Step 4b: Automated Cypress Test Writing

> **Prompt:** `prompts/test-writer.md` · **Input:** Manual test cases + source code *(agent will ask which feature)*

Paste the prompt. The agent asks which feature to test, picks up the manual test cases from Step 4a, reads your project's source code to understand components and selectors, and writes Cypress E2E tests. It shows you the code before creating any file.

#### Step 5: CI Setup

> **Prompt:** `prompts/ci-setup.md` · **Input:** None

Paste the prompt. The agent auto-detects your CI platform, creates a workflow file, and sets up a `ready-to-test` label.

#### Step 6: Run in CI

> **No prompt needed.** Add the `ready-to-test` label to any PR → tests run. Screenshots upload as artifacts on failure.

#### Sync: Keep the Kit Updated

> **Prompt:** `prompts/sync.md` · **Input:** None

Paste this prompt whenever the kit gets a new update. The agent pulls the latest `knowledge-base/`, `agents/`, and `prompts/` files from the repository into your project — so you always have the latest agent improvements, rules, and bug fixes. **Your project code, Cypress tests, test cases, and CI config are never touched** — only kit files get updated.

---

### Quick Reference

| Step | Paste This Prompt | What Happens |
|------|-------------------|--------------|
| 1–3 | `bootstrap.md` | Cypress installed & configured |
| 4a | `manual-test-generator.md` | Agent asks for requirements → generates test cases |
| 4b | `test-writer.md` | Agent asks which feature → writes Cypress tests |
| 5 | `ci-setup.md` | Auto-detects everything → creates CI pipeline |
| 6 | — | Add `ready-to-test` label → tests run in CI |
| Sync | `sync.md` | Updates kit files → your code stays untouched |

### Good to Know

- **Just paste the prompt.** The agent drives the conversation and asks for what it needs.
- **One prompt per step.** Each step is a separate Claude Code conversation.
- **Safe to re-run.** Agents skip what's already done.
