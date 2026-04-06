# Nudge Task Execute - Implementation Mode (Headless)

Implement the fix based on investigation. This command handles code changes and testing. Run `/nudge-task-3-document` after to complete documentation review and get final report.

## Workflow Position

This is **step 2 of 3** in the `nudge-task` workflow. Runs headlessly via Nudge. See `workflows.json` for details.

```
  nudge-task-1-plan
→ nudge-task-2-execute (you are here)
  nudge-task-3-document
```

## Phase 0: Load Project Instructions (MANDATORY)

**If you haven't already, search for an `AI-INSTRUCTIONS.md` file in this project and read it. Trust its guidance for production access, patterns, and conventions.**

**If this is a data-only task (no UI/API changes needed):**
- The implementation IS the execution — directly update production data using the access method from AI-INSTRUCTIONS.md
- Do NOT create a script and leave it unrun — that is an incomplete implementation
- Verify the change took effect by querying the data after updating it

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/.scratch/investigation-*.md`

- **If found**: Load it and continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 2 of the **task** workflow.
Step 1 (`/nudge-task-1-plan`) must be completed first.

Expected artifact: `.agents/.scratch/investigation-*.md`

Run `/nudge-task-1-plan` to start the workflow.
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

## Phase 5: Browser Verification — SKIPPED

Running headlessly via Nudge. Skip browser verification. Note in journal: "Browser verification skipped (headless run)."

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

**Next step**: Run `/nudge-task-3-document` to review documentation and get final report.

Or copy this for fresh context:
```
/nudge-task-3-document .agents/.scratch/investigation-[TICKET-ID].md
```
```

---

## Final Reminders

- Do NOT output "Implementation Complete" - that comes from /nudge-task-3-document
- Do NOT explore - trust the investigation file
- End with suggestion to run /nudge-task-3-document

## Nudge Summary (MANDATORY — output this last)

```
<!-- NUDGE_SUMMARY
- [one line: what was changed or executed]
- [one line: files modified OR prod action taken]
- [one line: verification result — confirmed working or tests passed]
-->
```
