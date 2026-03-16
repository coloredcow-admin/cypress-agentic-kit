# Cypress Agentic Kit — Knowledge Base Index

This is the entry point for all agents. Read this file first to determine which sections to load.

## Sections

| Section | File | Description |
|---------|------|-------------|
| Setup & Run | `01-setup-run.md` | Install Node (if missing), install Cypress, scaffold config, verify setup |
| Test Writing Standards | `02-test-writing.md` | Project structure detection, test categorization, writing standards, selector strategy, command patterns, anti-patterns |
| Manual Test Cases | `03-manual-test-cases.md` | Manual test case generation — format, categories, writing rules, automation-readiness, output behavior |
| CI Integration | `04-ci-integration.md` | CI workflow setup — label-triggered Cypress runs on PRs, artifact uploads, final report |

## Agent-to-Section Mapping

| Agent | Sections |
|-------|----------|
| `cypress-setup-agent` | `01-setup-run.md` |
| `cypress-test-writer-agent` | `02-test-writing.md` |
| `manual-test-generator-agent` | `03-manual-test-cases.md` |
| `cypress-ci-agent` | `04-ci-integration.md` |

## Rules

- Load only the sections mapped to your agent.
- If a section contains `TODO:` placeholders, skip it entirely.
- If the user's prompt conflicts with KB content, follow the user's prompt.
