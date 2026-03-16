# Cypress Agentic Kit — Agent Rules

## Knowledge Base Reading Rules

1. **Always read `knowledge-base/00-master.md` first.** This is the only mandatory read. It is a lightweight index listing all sections and agent-to-section mappings.
2. **Load only your mapped sections.** Each agent has a defined set of KB sections. Do not read sections outside your mapping.
3. **No Assumptions rule.** If a section file contains unfilled `TODO:` placeholders or is clearly incomplete, skip that section entirely. Do not infer, guess, or fabricate context from partial data.
4. **No Disturbance rule.** The knowledge base is read-only for all agents. No consuming agent writes to it. If the user provides explicit context in their prompt that differs from what the KB says, the user's prompt always takes precedence.
5. **Graceful degradation.** If `knowledge-base/` does not exist, or `00-master.md` is missing or empty, proceed exactly as you would without a knowledge base. No error. No assumption.

## Agent-to-Section Mapping

| Agent                    | Sections to Load         |
|--------------------------|--------------------------|
| cypress-setup-agent      | `01-setup-run.md`        |
| cypress-test-writer-agent | `02-test-writing.md`     |
| manual-test-generator-agent | `03-manual-test-cases.md` |
| cypress-ci-agent           | `04-ci-integration.md`   |

## Execution Rules

- Agents must **execute** the steps from their KB sections, not just restate them.
- Agents must be **idempotent** — do not redo what is already done.
- Agents must **show every command** before running it.
- Agents must **report pass/fail** at each step and give a final status report.
- If blocked, agents must state the **exact next action needed from the user**.
