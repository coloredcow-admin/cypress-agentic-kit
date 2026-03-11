# Manual Test Generator Prompt — Generate Manual Test Cases

Paste this prompt into a Claude session inside your project to generate manual test cases for a feature.

**Prerequisite:** None — this agent works with any project. It does not require Cypress to be installed.

---

## Prompt

```
I want to generate manual test cases for a feature in this project using the Cypress Agentic Kit.

Please do the following:

1. Read `knowledge-base/00-master.md` to understand the knowledge base structure.

2. Assume the role of `manual-test-generator-agent` by reading `agents/manual-test-generator-agent.md`.

3. Execute the test case generation runbook from `knowledge-base/03-manual-test-cases.md`:
   - Run pre-flight checks (understand project structure, check for existing test case docs)
   - Ask me for the feature name and description
   - Generate test cases across all 4 categories (Functional, UI, Interdependency, Edge Cases)

4. When generating test cases:
   - Show me the complete document before creating any files
   - Follow the standard format (Title, Preconditions, Steps, Expected Result, Test Data)
   - Place the file in `test-cases/` directory
   - Give me a summary with case counts per category

Important:
- Do not guess what feature to test — ask me.
- Do not modify any application code — only generate test case documents.
- Show me the complete document before writing any file.
- Give me a summary after generating.
```

---

## What Happens When You Run This

1. The agent checks for project structure and existing test case documents.
2. It asks you for the feature name and description (requirement doc, user story, or brief explanation).
3. It generates a structured test case document with 4 sections:
   - Functional Test Cases
   - UI Test Cases
   - Interdependency Test Cases
   - Edge Cases
4. It shows you the complete document for review.
5. After your confirmation, it creates the file and gives you a summary.

## After Generating

- Review the test cases and share with your QA team for manual testing.
- Later, use the test writer agent (`prompts/test-writer.md`) to automate selected cases — the manual cases are written to be automation-ready.
- To generate cases for another feature, run this prompt again.
