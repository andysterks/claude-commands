# Task Execute - Implementation Mode

Implement the fix based on investigation. This command handles code changes and testing. Run `/task-3-document` after to complete documentation review and get final report.

## Workflow Position

This is **step 2 of 3** in the `task` workflow. See `workflows.json` for details.

```
  task-1-plan
→ task-2-execute (you are here)
  task-3-document
```

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/.scratch/investigation-*.md`

- **If found**: Load it and continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 2 of the **task** workflow.
Step 1 (`/task-1-plan`) must be completed first.

Expected artifact: `.agents/.scratch/investigation-*.md`

Run `/task-1-plan` to start the workflow.
```

**Do NOT proceed without the investigation file.**

## Rules

- **DO NOT explore the codebase** - the investigation file already contains everything
- **DO NOT search for patterns** - they're in the investigation's Implementation Plan
- **DO NOT "understand the codebase first"** - that work is done, trust it
- **ONLY read files listed in the investigation's Implementation Plan**
- Your job: read investigation → write tests → write code → run tests

## Phase 1: Load Investigation & Initialize Journal

Read the investigation file: `.agents/.scratch/investigation-[TICKET-ID].md`

If no path provided, ask user for the investigation file path.

**Extract from investigation (DO NOT re-search for these):**
- `Implementation Plan > Files to Modify` - the exact files to change
- `Implementation Plan > Approach` - the steps to follow
- `Testing Strategy` - what tests to write
- `Browser Verification Recipe` - for UI verification later

**Create journal** at `.agents/.scratch/impl-journal-[TICKET-ID].md` using template: `~/.pdds-agents/templates/impl-journal-template.md`

## Phase 2: Read ONLY Listed Files

**⚠️ DO NOT search or explore. Read ONLY files from the Implementation Plan.**

For each file in `Files to Modify`:
1. Read it
2. Identify the exact location for changes

If a file path in the investigation is wrong, note it in journal and ask user.

**Journal checkpoint:** Update Phase 2 section in journal.

## Phase 3: Write Failing Tests

Write tests FIRST. Priority: E2E → Integration → Unit.

**Use test files from investigation's `Testing Strategy` section - DO NOT search for other examples.**

Tests should:
- Fail initially (proving they test the bug/feature)
- Cover acceptance criteria from investigation's Gherkin scenarios
- Follow patterns in the test files already listed in the investigation

**Journal checkpoint:** Update Phase 3 section in journal.

## Phase 4: Implement & Iterate

**Iteration 1 (you):** Write implementation, run tests.

**Iterations 2-3 (if needed):** Spawn Test Fix Subagent:
```
Task tool:
  subagent_type: "general-purpose"
  model: "haiku"
  description: "Fix failing tests"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/test-fix.md

    Input:
    - test_failures: [paste test output]
    - files_modified: [list of files you changed]
    - investigation_path: .agents/.scratch/investigation-[TICKET-ID].md"
```

Stop after 3 iterations if still failing - ask user for guidance.

**Journal checkpoint:** Update Phase 4 section in journal.

## Phase 5: Browser Verification (UI Only)

Skip if backend-only. Ask user: "Should I verify in browser?"

If yes, **prepare context before spawning** the subagent:

1. Read `.agents/operations/browser-automation.md` (if it exists) and `.agents/operations/login.md`
2. Extract the `Browser Investigation` and `Reproduction Method` sections from the investigation file

Then spawn Browser Verification Subagent with all context inlined:
```
Task tool:
  subagent_type: "general-purpose"
  model: "sonnet"
  description: "Browser verification"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/browser-verification.md

    Input:
    - navigation_steps: [copy from investigation file Browser Verification Recipe]
    - verifications: [copy verification checklist table with Type column]
    - expected_outcomes: {
        primary: '[expected behavior after fix]'
      }
    - reproduction_method: |
        [copy the entire Browser Investigation + Reproduction Method sections
         from the investigation file — includes exact JS commands, store paths,
         and workarounds that were already proven to work during investigation]
    - project_browser_guide: |
        [paste contents of .agents/operations/browser-automation.md here]
    - login_credentials: |
        [paste contents of .agents/operations/login.md here]"
```

**Why inline?** Subagents start cold with no project context. Telling them to "read file X" wastes turns and risks them skimming or missing key details. Inlining gives them proven commands immediately.

**After subagent returns:**
1. Log results: "[X] passed, [Y] failed, [Z] uncertain"
2. If `questions_for_user` returned, ask them via AskUserQuestion
3. Update Phase 5 section in journal

## Phase 6: Update Investigation File

Append implementation results to the investigation file using template: `~/.pdds-agents/templates/impl-results-template.md`

---

## Output

**Do NOT output "Implementation Complete". Output this instead:**

```
## Implementation Done - Documentation Review Required

**Tests**: ✅ [N] unit, [M] contract/integration passed

**Files Modified**:
| File | Change |
|------|--------|
| [file] | [description] |

**Investigation**: `.agents/.scratch/investigation-[TICKET-ID].md`
**Journal**: `.agents/.scratch/impl-journal-[TICKET-ID].md`

---

**Next step**: Run `/task-3-document` to review documentation and get final report.

Or copy this for fresh context:
```
/task-3-document .agents/.scratch/investigation-[TICKET-ID].md
```
```

---

## Final Reminders

- Do NOT output "Implementation Complete" - that comes from /task-3-document
- Do NOT explore - trust the investigation file
- End with suggestion to run /task-3-document
