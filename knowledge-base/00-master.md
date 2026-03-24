# Cypress Agentic Kit — Knowledge Base Index

This is the entry point for all agents. Read this file first to determine which sections to load.

## Sections

| Section | File | Description |
|---------|------|-------------|
| Test Writing Standards | `02-test-writing.md` | Project structure detection, test categorization, writing standards, selector strategy, command patterns, anti-patterns |
| Manual Test Cases | `03-manual-test-cases.md` | Manual test case generation — format, categories, writing rules, automation-readiness, output behavior |

> **Note:** Setup (`01-setup-run.md`) and CI (`04-ci-integration.md`) knowledge base files are available in the kit repo only. They are one-time use and not copied to the project.

## Agent-to-Section Mapping

| Agent | Sections |
|-------|----------|
| `cypress-test-writer-agent` | `02-test-writing.md` |
| `manual-test-generator-agent` | `03-manual-test-cases.md` |

> **Note:** `cypress-setup-agent` and `cypress-ci-agent` run directly from the kit repo and are not installed in the project.

## Rules

- Load only the sections mapped to your agent.
- If a section contains `TODO:` placeholders, skip it entirely.
- If the user's prompt conflicts with KB content, follow the user's prompt.
