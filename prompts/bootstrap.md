# Bootstrap Prompt — Install Cypress Agentic Kit

Paste this entire prompt into a Claude session inside your project to install the Cypress knowledge base and setup agent.

---

## Prompt

```
I want to set up Cypress in this project using the Cypress Agentic Kit.

Please do the following:

1. Clone or fetch the knowledge base and agent files from the repository:
   https://github.com/abhilashabattu97/cypress-agentic-kit

2. Add only the **ongoing-use** kit files into this project. Setup and CI files stay in the kit repo — they are one-time use and should be referenced from the repo directly when needed. **Merge, do not replace:**
   - `knowledge-base/` — only add `00-master.md`, `02-test-writing.md`, `03-manual-test-cases.md`. Do NOT copy `01-setup-run.md` or `04-ci-integration.md` (these stay in the kit repo). Do not overwrite or remove any existing files in the directory.
   - If `knowledge-base/` does not exist, create it and copy the above KB files into it.
   - `agents/` — only add `CLAUDE.md`, `cypress-test-writer-agent.md`, `manual-test-generator-agent.md`. Do NOT copy `cypress-setup-agent.md` or `cypress-ci-agent.md` (these stay in the kit repo). Do not overwrite or remove existing agent files.
   - `prompts/` — only add `test-writer.md`, `manual-test-generator.md`. Do NOT copy `bootstrap.md`, `ci-setup.md`, or `sync.md` (these stay in the kit repo). Do not overwrite or remove existing prompt files.
   - If any of the kit's specific files already exist in the project (e.g., from a previous install), skip them. To update existing files, use the sync prompt from the kit repo (`prompts/sync.md`) instead.

3. The kit's agent rules are in `agents/CLAUDE.md` — this file is included with the agent files copied in step 2. Do NOT modify the project's root `CLAUDE.md`.

4. After copying the files, read `knowledge-base/00-master.md` and confirm what was installed.

5. Then assume the role of `cypress-setup-agent` by reading the agent file from the kit repo (`agents/cypress-setup-agent.md`) and execute the Cypress setup by following the runbook from the kit repo (`knowledge-base/01-setup-run.md`). These files are NOT copied to the project — read them directly from the cloned/fetched kit repo.

Important:
- Do not guess any project-specific values. Ask me if you are unsure.
- If this is not a Node project, ask before creating a package.json.
- If Node.js is not installed, ask before installing it.
- Show me every command before running it.
- Give me a final report at the end.
```

---

## What Happens When You Run This

1. Only the ongoing-use files are copied into your project (test writing + manual test case KB, agents, and prompts).
2. Setup and CI files stay in the kit repo — they are used once and not needed afterward.
3. Agent rules are placed in `agents/CLAUDE.md` — your project's root `CLAUDE.md` is not touched.
4. The setup agent runs directly from the kit repo and:
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

- **Write tests:** Paste the prompt from `prompts/test-writer.md` (now in your project) to start writing Cypress tests using the test writer agent.
- **Generate manual test cases:** Paste the prompt from `prompts/manual-test-generator.md` (now in your project) to generate structured test case documents.
- **Set up CI:** Paste the prompt from `prompts/ci-setup.md` in the kit repo — this is not copied to your project since it's a one-time operation.
- **Update kit:** If a new version of the kit is released, paste the prompt from `prompts/sync.md` in the kit repo to pull the latest ongoing-use files.
