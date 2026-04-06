# Nudge Task Document - Documentation Review & Final Report (Headless)

Complete documentation reconciliation and output final implementation report.

**Argument**: Investigation file path (e.g., `.agents/.scratch/investigation-[TICKET-ID].md`)

## Workflow Position

This is **step 3 of 3** in the `nudge-task` workflow. Runs headlessly via Nudge. See `workflows.json` for details.

```
  nudge-task-1-plan
  nudge-task-2-execute
→ nudge-task-3-document (you are here)
```

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/.scratch/impl-journal-*.md`

- **If found**: Continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 3 of the **task** workflow.
Step 2 (`/nudge-task-2-execute`) must be completed first.

Expected artifact: `.agents/.scratch/impl-journal-*.md`

Workflow: /task-1-plan → /task-2-execute → /task-3-document
```

**Do NOT proceed without the implementation journal.**

## Phase 1: Load Context

Read:
1. Investigation file (from argument or ask user)
2. Implementation journal: `.agents/.scratch/impl-journal-[TICKET-ID].md`

**Extract the Documentation Contract section. It contains:**
- **Docs to Consult** table - each row needs validation
- **Documentation Gaps** table - each row needs a proposal
- **Domain Knowledge** - questions that were answered
- **Post-Implementation Requirements** - checklist items

**You MUST process every row in these tables. Do not skip any.**

Example contract items you might see:
```
| `.agents/operations/locations.md` | Verify implementation matches |
| `.agents/reference/INDEX.md` | Add missing terminology |
```

Each of these becomes a proposal in Phase 3.

## Phase 2: Discovery - Be Curious About New Concepts

**Your job is to EXTRACT domain knowledge from the user. Be genuinely curious.**

Review the implementation and journal. Look for concepts you don't fully understand:
- Domain terms (e.g., "Capture Station", "Site", business-specific nouns)
- Entity relationships (how things connect)
- Business rules (why something works this way)
- States/workflows (what are the lifecycle stages)

**For EACH significant concept you encountered, ask 3-6 probing questions:**

Example for "Capture Station":
```
nudge-ask (batch up to 4 questions — send as a single combined question):

Bash: nudge-ask --question "About Capture Stations: What is it physically? How does it relate to a Site? What states can it be in?" --options "Hardware only,Software only,Both hardware and software,Other"
```

**Follow-up questions to consider:**
- "Are there business rules around [X] I should document?"
- "What happens when [edge case]?"
- "Is [my inference] correct, or is there more nuance?"
- "What's the history/context behind this design?"

**Don't rush this phase.** The user's answers become high-quality reference documentation. If you encountered something interesting in the code, ASK ABOUT IT.

Record all answers - these feed directly into Phase 4 proposals.

## Phase 3: Validate Docs (No Approval Needed)

**For each "Docs to Consult" row**: Read the doc and verify implementation followed it.

**Report validations silently** - no user approval needed:
```
### Docs Validated
| Doc | Result |
|-----|--------|
| .agents/codebase/testing.md | ✅ Followed test patterns |
| .agents/codebase/code-standards.md | ✅ Used BadRequestException correctly |
```

**Only flag discrepancies** - if implementation doesn't match a doc:
- Either the doc needs updating (proceed to Phase 4)
- Or implementation is wrong (flag to user)

## Phase 4: Propose Documentation Changes (Approval Required)

**Create proposals from:**
1. **Answers from Phase 2** - New concepts the user explained
2. **Documentation Gaps** from contract - Missing content to create
3. **Discrepancies** - Docs that need updating to match implementation

**If no changes needed**: Skip to Phase 6 and report "No documentation changes required."

---

**⚠️ CRITICAL: You MUST show the FULL proposed content BEFORE asking for approval.**

**For EACH change, output this FIRST (before AskUserQuestion):**

```
---
## Proposal [N/M]: [Brief description]

**File**: `.agents/[section]/[filename].md`
**Action**: [Create new file | Add section | Update existing]

### Proposed Content:

[Write out the COMPLETE markdown that will be written.
Do not summarize. Show every heading, every paragraph,
every table row. The user must see exactly what they're approving.]

### Rationale:
[Why this change is needed - reference Phase 2 answers or contract]

---
```

**THEN ask for approval via nudge-ask, passing the full proposed content as --context:**

```
Bash: nudge-ask --question "Approve documentation for [filename]?" --options "Approve as written,Approve with my edits,Skip this change,Discuss first" --context "[paste the complete proposed markdown content here]"
```

Read the output and handle accordingly.

**Handle responses:**
- **Approve** → Apply exactly what you showed above
- **With edits** → Use nudge-ask freetext to ask what to change: `nudge-ask --question "What changes would you like?" --freetext` then revise and re-ask for approval
- **Skip** → Record reason, continue to next
- **Discuss** → Use nudge-ask freetext: `nudge-ask --question "What would you like to discuss?" --freetext` then address their concern and re-show the proposal

**Continue until ALL proposals processed.**

## Phase 5: INDEX.md Updates

For new/updated docs, propose INDEX.md keyword entries using same approval flow:

```
**INDEX.md Update**
**File**: .agents/[section]/INDEX.md

**Add**:
**Keywords**: [keyword1], [keyword2]
- [Doc Name](./doc.md) - [one-line description]
```

## Phase 6: Run Lints

```bash
node .agents/scripts/lint-docs.js
```

If lint fails, ask via nudge-ask:
```
Bash: nudge-ask --question "[file] exceeds token limit. How to fix?" --options "Summarize - make more concise,Split - break into multiple files,Override - accept overage for now"
```

## Phase 7: Record Results

Append to investigation file:

```markdown
---
## Documentation Reconciliation

**Date**: [YYYY-MM-DD]
**Reviewer**: User

### Proposals: [N] total
| # | File | Action | Status |
|---|------|--------|--------|
| 1 | [path] | [Create/Update] | [Approved/Skipped] |

### Interview Answers
| Question | Answer | Documented In |
|----------|--------|---------------|
| [Q] | [A] | [file path] |

### Changes Applied
- Created: [list]
- Updated: [list]

### Lint: [Pass/Fail]
```

## Phase 8: Final Report

**NOW output the final report:**

```
## Implementation Complete ✅

**Investigation**: [filename]

**Files Modified**:
| File | Change |
|------|--------|
| [file] | [description] |

**Tests**: ✅ [N] unit, [M] contract/integration passed

**Browser** (if applicable): [Pass/Fail/N/A]

**Documentation**:
- Proposals: [N] presented, [M] approved, [P] skipped
- Created: [list or "None"]
- Updated: [list or "None"]

**Status**: 🟢 Ready to commit

**Next**: Review changes and commit, close ticket
```

---

## Rules

- **Be curious**: If you encountered something new, ASK about it (3-6 questions per concept)
- **Show full content**: ALWAYS display the complete proposed markdown BEFORE asking for approval
- **Never approve blindly**: User must see exactly what they're approving
- Present proposals one at a time, not batched
- Record why user skipped any proposals
- Lint must pass (or user explicitly overrides)
- Only THIS command outputs "Implementation Complete"

## Nudge Summary (MANDATORY — output this last)

```
<!-- NUDGE_SUMMARY
- [one line: docs created/updated or "No doc changes"]
- [one line: lint result]
- [one line: ready to merge or any concerns]
-->
```
