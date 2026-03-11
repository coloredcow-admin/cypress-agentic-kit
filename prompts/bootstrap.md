# Bootstrap Prompt — Install Cypress Agentic Kit

Paste this entire prompt into a Claude session inside your project to install the Cypress knowledge base and setup agent.

---

## Prompt

```
I want to set up Cypress in this project using the Cypress Agentic Kit.

Please do the following:

1. Clone or fetch the knowledge base and agent files from the repository:
   https://github.com/abhilashabattu97/cypress-agentic-kit

2. Add the kit files into this project. **Merge, do not replace:**
   - If `knowledge-base/` already exists (from an existing agentic system or prior setup), only add the kit's specific files (`00-master.md`, `01-setup-run.md`, `02-test-writing.md`, `03-manual-test-cases.md`) into it. Do not overwrite or remove any existing files in the directory.
   - If `knowledge-base/` does not exist, create it and copy all kit KB files into it.
   - Same for `agents/` — only add the kit's agent files (`cypress-setup-agent.md`, `cypress-test-writer-agent.md`, `manual-test-generator-agent.md`). Do not overwrite or remove existing agent files.
   - If any of the kit's specific files already exist in the project (e.g., from a previous install), skip them. To update existing files, use `prompts/sync.md` instead.

3. Check if a `CLAUDE.md` file exists in the project root:
   - If it exists: append only the "Knowledge Base Reading Rules" and "Agent-to-Section Mapping" sections from the kit's `CLAUDE.md`. Do not duplicate entries that already exist. Do not modify or remove any existing content in the file.
   - If it does not exist: copy the kit's `CLAUDE.md` to the project root.

4. After copying the files, read `knowledge-base/00-master.md` and confirm what was installed.

5. Then assume the role of `cypress-setup-agent` by reading `agents/cypress-setup-agent.md` and execute the Cypress setup by following the runbook in `knowledge-base/01-setup-run.md`.

Important:
- Do not guess any project-specific values. Ask me if you are unsure.
- If this is not a Node project, ask before creating a package.json.
- If Node.js is not installed, ask before installing it.
- Show me every command before running it.
- Give me a final report at the end.
```

---

## What Happens When You Run This

1. The knowledge base files and agent files are copied into your project.
2. Your project's `CLAUDE.md` is updated with KB reading rules.
3. The setup agent runs automatically and:
   - Checks if Node.js is installed (installs with your consent if not)
   - Detects your package manager
   - Installs Cypress
   - Scaffolds config and support files
   - Validates the setup
   - Shows a final report

## After Setup

- Add your first test in `cypress/e2e/`
- Run tests with `npx cypress open` (interactive) or `npx cypress run` (headless)
- The knowledge base stays in your project for future agents

## Next Steps

- **Write tests:** Paste the prompt from `prompts/test-writer.md` to start writing Cypress tests using the test writer agent.
- **Update kit:** If a new version of the kit is released, paste the prompt from `prompts/sync.md` to pull the latest KB and agent files without re-running setup.
