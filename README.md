# Cypress Agentic Kit

An AI-driven kit that automates Cypress setup in any project — Node or non-Node — using a knowledge base and agent architecture.

## What This Does

A single prompt installs a Cypress knowledge base and setup agent into your project. The agent reads the knowledge base and automatically:

- Checks if Node.js is installed (installs it with your consent if not)
- Detects your package manager (npm / yarn / pnpm)
- Handles non-Node projects (Laravel, Django, Rails, etc.) by creating a minimal Node setup
- Installs Cypress as a dev dependency
- Scaffolds config, support files, and directory structure
- Validates the setup and gives you a final report

No scripts to run. No dependencies to install. Just one prompt.

## How to Use

### 1. Open your project in Claude

Open a Claude session (Claude Code, Cursor, or any Claude-integrated IDE) inside the project where you want Cypress.

### 2. Paste the bootstrap prompt

Copy the prompt from [`prompts/bootstrap.md`](prompts/bootstrap.md) and paste it into your Claude session.

### 3. Follow the agent

The agent will walk through the setup step by step, asking for your input only when needed (e.g., Node installation consent, project-specific config).

### 4. Done

You'll get a final report showing what was installed and the status of each step. Start writing tests in `cypress/e2e/`.

## Project Structure

```
cypress-agentic-kit/
├── CLAUDE.md                          # Rules for how agents read the KB
├── README.md                          # This file
├── knowledge-base/
│   ├── 00-master.md                   # Index — lists all sections + agent mappings
│   ├── 01-setup-run.md               # Setup runbook (11 steps)
│   ├── 02-test-writing.md            # Test writing standards (10 sections)
│   ├── 03-manual-test-cases.md       # Manual test case generation (6 sections)
│   └── 04-ci-integration.md          # CI workflow setup (3 sections)
├── agents/
│   ├── cypress-setup-agent.md         # Setup agent prompt
│   ├── cypress-test-writer-agent.md   # Test writer agent prompt
│   ├── manual-test-generator-agent.md # Manual test case generator agent prompt
│   └── cypress-ci-agent.md            # CI setup agent prompt
└── prompts/
    ├── bootstrap.md                   # One prompt to install everything
    ├── test-writer.md                 # One prompt to start writing tests
    ├── manual-test-generator.md       # One prompt to generate manual test cases
    ├── ci-setup.md                    # One prompt to set up CI workflow
    └── sync.md                        # One prompt to update kit files
```

## Works With Any Project

| Project Type | What Happens |
|-------------|--------------|
| Node project (React, Next.js, Angular, Vue) | Detects package manager, installs Cypress |
| Non-Node project (Laravel, Django, Rails, WordPress) | Creates minimal package.json, installs Cypress |
| No Node.js on machine | Installs Node.js with your consent, then proceeds |

## Writing Tests

After Cypress is set up, use the test writer agent to generate tests:

1. Paste the prompt from [`prompts/test-writer.md`](prompts/test-writer.md) into your Claude session.
2. The agent runs pre-flight checks and reads your project structure.
3. Tell it what feature or flow you want to test.
4. It writes the test, adds `data-cy` selectors to your app code, and gives you a summary.

Already have the kit installed from an earlier version? Run [`prompts/sync.md`](prompts/sync.md) first to pull the latest files.

## Generating Manual Test Cases

Generate structured manual test cases for any feature — before or during development:

1. Paste the prompt from [`prompts/manual-test-generator.md`](prompts/manual-test-generator.md) into your Claude session.
2. The agent asks you for the feature name and description.
3. It generates test cases across 4 categories: Functional, UI, Interdependency, and Edge Cases.
4. Review and share with your QA team. Later, use the test writer agent to automate selected cases.

## Setting Up CI

Set up a CI workflow that runs Cypress tests when a pull request is labeled:

1. Paste the prompt from [`prompts/ci-setup.md`](prompts/ci-setup.md) into your Claude session.
2. The agent detects your CI platform, package manager, and dev server.
3. It creates a workflow file triggered by the `ready-to-test` label on PRs.
4. When you add the label to a PR, Cypress runs automatically in CI.
5. Screenshots and videos are uploaded as artifacts on failure.

## What's Coming Next

| Phase | What | Status |
|-------|------|--------|
| Phase 1 | Cypress setup automation | Done |
| Phase 2 | Test writing standards KB + test writer agent | Done |
| Phase 3 | Manual test case generator agent | Done |
| Phase 4 | CI integration KB + CI agent | Planned |

## Design Principles

- **One prompt installs everything** — no manual file creation
- **Agents read only what they need** — minimal token usage, no hallucination from context overload
- **Idempotent** — safe to run multiple times, skips what's already done
- **Works without the KB** — if knowledge base is missing, agents degrade gracefully
- **User prompt always wins** — KB is supplementary context, never an override
