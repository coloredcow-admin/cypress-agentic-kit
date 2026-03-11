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

2. Update only the Cypress Agentic Kit files in this project. **Only touch kit-specific files, leave everything else untouched:**
   - `knowledge-base/` — only add or update kit files (`00-master.md`, `01-setup-run.md`, `02-test-writing.md`, `03-manual-test-cases.md`). Do not touch any other files in this directory (they may belong to the project's own agentic system or other tools). Do not overwrite kit files that have local modifications — if the file exists and has been customized, skip it.
   - `agents/` — only add or update kit agent files (`cypress-setup-agent.md`, `cypress-test-writer-agent.md`, `manual-test-generator-agent.md`). Do not touch any other agent files in this directory.
   - `prompts/` — only add or update kit prompt files (`bootstrap.md`, `test-writer.md`, `manual-test-generator.md`, `sync.md`). Do not touch any other prompt files.

3. Update `knowledge-base/00-master.md`:
   - Add any new Cypress kit section entries that don't already exist.
   - Add any new Cypress kit agent-to-section mappings that don't already exist.
   - Do not remove or modify any existing entries — they may belong to other systems.

4. Update `CLAUDE.md`:
   - Add any new Cypress kit agent-to-section mappings that don't already exist.
   - Do not remove or modify any existing rules, mappings, or content — they may belong to the project's own configuration.

5. Report what was added or updated. List each file with its status:
   - Added (new file)
   - Updated (new entries merged)
   - Skipped (already up to date)

Important:
- Do NOT run any agent after syncing. Only update files.
- Do NOT overwrite files that the user may have customized.
- Do NOT remove any existing entries from CLAUDE.md or 00-master.md.
```

---

## What Happens When You Run This

1. New knowledge base sections are added (e.g., `02-test-writing.md`).
2. New agent prompts are added (e.g., `cypress-test-writer-agent.md`).
3. New bootstrap/invocation prompts are added (e.g., `test-writer.md`).
4. Index files (`00-master.md`, `CLAUDE.md`) are updated with new mappings.
5. No agent is executed — this is a file sync only.

## After Syncing

Run the relevant agent prompt to use the new capabilities:
- Test writer: paste the prompt from `prompts/test-writer.md`
- Setup (if not done): paste the prompt from `prompts/bootstrap.md`
