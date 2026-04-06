# Nudge Task Plan - Investigation Protocol (Headless)

You are investigating a task (Jira card, GitHub issue, bug report, or feature request). Your goal is to **understand the task thoroughly** and **update documentation** before writing any code.

## Workflow Position

This is **step 1 of 3** in the `nudge-task` workflow. Runs **headlessly via Nudge** — no browser tools, no interactive prompts.

```
→ nudge-task-1-plan (you are here)
  nudge-task-2-execute
  nudge-task-3-document
```

**No prerequisites** - this is the starting point. After completion, run `/nudge-task-2-execute`.

## Rules

- **DO NOT write code** during this investigation
- **DO NOT suggest fixes** yet
- **DO NOT search or read code files directly** - spawn subagents for all exploration
- **DO NOT use Glob, Grep, or Read on source code** - subagents handle this
- Focus entirely on orchestrating subagents and synthesizing their summaries
- Your context is precious - let subagents consume tokens on file exploration

## Phase 0: Load Project Instructions (MANDATORY)

**Before anything else, search for an `AI-INSTRUCTIONS.md` file in this project and read it if it exists. Trust its guidance — it will tell you where to find everything you need.**

## Phase 0b: Start Dev Server & Get App URL

Before browser testing, start the dev server and find the local URL:

1. Run `npm run dev` (or the appropriate start command) in the worktree
2. Check the output for the local URL (e.g. `http://localhost:5173` or the port from `NUDGE_FRONTEND_PORT`)
3. Note the URL for use in Phase 3 browser exploration

## Phase 1: Parse the Task

1. Read the task details the user provided (description, acceptance criteria, screenshots)
2. Identify the key screen/feature mentioned
3. Extract keywords for searching docs
4. Convert acceptance criteria to Gherkin format:
   ```gherkin
   Feature: [Feature name]

   Scenario: [Scenario description]
     Given [initial context]
     When [action taken]
     Then [expected outcome]
   ```

## Phase 2: Delegate to Subagents

**⚠️ CRITICAL: DO NOT explore the codebase yourself. ONLY spawn subagents.**

You may read ONE file: `.agents/operations/NAVIGATION.md` for screen paths.

**ALL other exploration MUST be done by subagents. Do not:**
- Search for patterns with Grep
- Read source code files with Read
- Glob for file patterns
- "Just quickly check" anything

**Subagents return summaries. You synthesize. This saves 80%+ of your context.**

**Spawn these subagents ONE AT A TIME - each depends on the previous:**

**⚠️ DO NOT run in parallel. Wait for each to complete before spawning the next.**

### Subagent 1: Doc Lookup
```
Task tool:
  subagent_type: "Explore"
  model: "haiku"
  description: "Doc lookup"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/doc-lookup.md

    Input:
    - keywords: [keywords from Phase 1]
    - feature_name: [feature name from ticket]
    - ticket_summary: [brief ticket description]"
```

### Subagent 2: Code Lookup (uses Doc Lookup output)
```
Task tool:
  subagent_type: "Explore"
  model: "haiku"
  description: "Code lookup"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/code-lookup.md

    Input:
    - doc_summary: [paste relevant_docs and patterns from Subagent 1]
    - keywords: [keywords from Phase 1]
    - feature_name: [feature name from ticket]"
```

### Subagent 3: Operation Lookup (uses Doc Lookup output)
```
Task tool:
  subagent_type: "Explore"
  model: "haiku"
  description: "Operation lookup"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/operation-lookup.md

    Input:
    - screen_name: [target screen from ticket]
    - feature_name: [feature name]
    - doc_context: [relevant operations docs from Subagent 1]"
```

### Subagent 4: Browser Exploration (Playwright MCP)

Use `mcp__playwright__*` tools to smoke-test the feature in the running dev server:

1. Navigate to the app URL from Phase 0b using `mcp__playwright__navigate`
2. Follow the navigation steps from Subagent 3 to reach the target screen
3. Take a screenshot with `mcp__playwright__screenshot` to confirm the current state
4. Attempt to reproduce the issue described in the ticket
5. Take a screenshot after reproduction
6. Record: navigation success, whether bug was reproduced, any console errors

**Execution order (strict - do not parallelize):**
1. Spawn Doc Lookup → **wait for completion** → get doc_summary
2. Spawn Code Lookup with doc_summary → **wait for completion**
3. Spawn Operation Lookup with doc_context → **wait for completion**
4. Run Browser Exploration with Playwright MCP using navigation_steps from Subagent 3

## Phase 3: Review Subagent Findings

**All subagents have returned. Synthesize their findings:**

### From Doc Lookup (Subagent 1):
- `relevant_docs` - Which docs apply?
- `terminology` - Key terms to understand
- `patterns_to_follow` - Architecture patterns
- `gaps_identified` - Missing documentation

### From Code Lookup (Subagent 2):
- `files_to_modify` - Where to make changes
- `entry_points` - Starting points for code
- `data_flow` - How data moves through system
- `test_files` - Existing tests to reference

### From Operation Lookup (Subagent 3):
- `navigation_steps` - How to reach the screen
- `target_element` - What to look for
- `expected_states` - Before/after states

### From Browser Exploration (Subagent 4):
- `navigation.success` - Could we reach the screen?
- `bug_recreation.success` - Did we reproduce the issue?
- `verification_checklist` - Items to test after fix
- `questions_for_user` - Uncertainties to clarify

### From Browser Exploration (Playwright MCP):
- Did navigation succeed?
- Was the bug reproduced?
- Any console errors or unexpected states?
- Screenshots captured for before/after

### Your Job:
1. Combine findings into coherent understanding
2. Cross-reference: Do UI observations match docs?
3. Note discrepancies between documented and actual behavior
4. If bug not reproduced, clarify with user before proceeding

## Phase 4: Interview Developer

Use `nudge-ask` to fill gaps. For each batch of questions:

```
Bash: nudge-ask --question "Your batched question here?" --options "Answer A,Answer B,Answer C,Other"
```

Read the output and record the answer. Keep questions focused and batched (2-3 at a time max).

## Phase 5: Create Investigation Summary

**Determine filename:**
- If ticket ID provided: `investigation-[TICKET-ID].md`
- If no ticket ID: `investigation-[slug-from-description].md`

**Create `.agents/.scratch/investigation-[filename].md`:**

Follow the template at `~/.pdds-agents/templates/investigation-template.md`

Fill in all sections with findings from subagents and interviews. Key sections:
- **Acceptance Criteria** - Gherkin from Phase 1
- **Browser Investigation** - From Browser subagent
- **Browser Verification Recipe** - Navigation steps + checklist from subagents
- **Root Cause Analysis** - Your technical analysis
- **Implementation Plan** - Files to modify from Code Lookup subagent
- **Documentation Contract** - Gaps from Doc Lookup subagent

## Phase 6: Checkpoint

Report to user - keep it terse and scannable:

```
## Investigation Complete ✅

**File**: `.agents/.scratch/investigation-[filename].md`

**Bug Recreation**: [🟢 Success | 🔴 Failed] - [one sentence]

**Root Cause**: [one sentence technical explanation]

---

**Next Steps**:
• Reply "proceed" to implement now
• Reply "/clear" then paste below for fresh context

**Copy for fresh context**:
```
/nudge-task-2-execute

Investigation: .agents/.scratch/investigation-[filename].md
```
```

Do not proceed to code until the developer confirms.

## Nudge Summary (MANDATORY — output this last)

After the checkpoint, output this exact block:

```
<!-- NUDGE_SUMMARY
- [one line: what the issue is]
- [one line: root cause]
- [one line: files/areas to change]
-->
```