# Sync Prompt — Update Cypress Agentic Kit

Paste this prompt into a Claude session inside your project to update the knowledge base and agent files to the latest version. This does not re-run any agent — it only syncs files.

**Use this when:** A new phase has been released (e.g., test writer agent added) and you want to pull the latest KB and agent files into your existing project.

---

## Prompt

```
I want to update the Cypress Agentic Kit files in this project to the latest version.

Please do the following:

1. Fetch the latest files from the repository:
   https://github.com/abhilashabattu97/cypress-agentic-kit

2. Update only the **ongoing-use** Cypress Agentic Kit files in this project. Setup and CI files are not part of the project — they stay in the kit repo. **Only touch kit-specific files, leave everything else untouched:**
   - `knowledge-base/` — only add or update `00-master.md`, `02-test-writing.md`, `03-manual-test-cases.md`. Do NOT add `01-setup-run.md` or `04-ci-integration.md` (these stay in the kit repo). Do not touch any other files in this directory. Do not overwrite kit files that have local modifications — if the file exists and has been customized, skip it.
   - `agents/` — only add or update `CLAUDE.md`, `cypress-test-writer-agent.md`, `manual-test-generator-agent.md`. Do NOT add `cypress-setup-agent.md` or `cypress-ci-agent.md` (these stay in the kit repo). Do not touch any other agent files in this directory.
   - `prompts/` — only add or update `test-writer.md`, `manual-test-generator.md`. Do NOT add `bootstrap.md`, `ci-setup.md`, or `sync.md` (these stay in the kit repo). Do not touch any other prompt files.

3. Update `knowledge-base/00-master.md`:
   - Add any new Cypress kit section entries that don't already exist.
   - Add any new Cypress kit agent-to-section mappings that don't already exist.
   - Do not remove or modify any existing entries — they may belong to other systems.

4. Do NOT modify the project's root `CLAUDE.md`. The kit's agent rules live in `agents/CLAUDE.md` — this file is synced as part of the `agents/` directory in step 2.

5. Report what was added or updated. List each file with its status:
   - Added (new file)
   - Updated (new entries merged)
   - Skipped (already up to date)

Important:
- Do NOT run any agent after syncing. Only update files.
- Do NOT overwrite files that the user may have customized.
- Do NOT modify the project's root CLAUDE.md.
- Do NOT remove any existing entries from 00-master.md.
```

---

## What Happens When You Run This

1. Ongoing-use knowledge base files are added or updated (`02-test-writing.md`, `03-manual-test-cases.md`).
2. Ongoing-use agent files are added or updated (`cypress-test-writer-agent.md`, `manual-test-generator-agent.md`).
3. Ongoing-use prompt files are added or updated (`test-writer.md`, `manual-test-generator.md`).
4. Index file (`00-master.md`) is updated with new mappings. Agent rules in `agents/CLAUDE.md` are synced.
5. No agent is executed — this is a file sync only.
6. Setup and CI files are NOT synced — they stay in the kit repo for one-time use.

## After Syncing

Run the relevant agent prompt to use the new capabilities:
- Test writer: paste the prompt from `prompts/test-writer.md` (in your project)
- Manual test cases: paste the prompt from `prompts/manual-test-generator.md` (in your project)
- CI setup: paste the prompt from `prompts/ci-setup.md` (in the kit repo)
- Setup (if not done): paste the prompt from `prompts/bootstrap.md` (in the kit repo)
