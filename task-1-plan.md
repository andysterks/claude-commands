# Task Plan - Investigation Protocol

You are investigating a task (Jira card, GitHub issue, bug report, or feature request). Your goal is to **understand the task thoroughly** and **update documentation** before writing any code.

## Workflow Position

This is **step 1 of 3** in the `task` workflow. See `workflows.json` for details.

```
→ task-1-plan (you are here)
  task-2-execute
  task-3-document
```

**No prerequisites** - this is the starting point. After completion, run `/task-2-execute`.

## Rules

- **DO NOT write code** during this investigation
- **DO NOT suggest fixes** yet
- **DO NOT search or read code files directly** - spawn subagents for all exploration
- **DO NOT use Glob, Grep, or Read on source code** - subagents handle this
- Focus entirely on orchestrating subagents and synthesizing their summaries
- Your context is precious - let subagents consume tokens on file exploration

## Phase 0: Verify Browser Tools (MANDATORY PREREQUISITE)

**⚠️ CRITICAL: This MUST be the first thing you do. Do not proceed to Phase 1 without completing this step.**

**Test browser tools immediately using this exact sequence:**

1. **Test Claude in Chrome first:**
   ```
   Call: mcp__claude-in-chrome__tabs_context_mcp with createIfEmpty=true
   ```
   - If successful → Browser tools are ready, proceed to Phase 1
   - If error → Continue to step 2

2. **Test Playwright as fallback:**
   ```
   Call: mcp__playwright__browser_snapshot
   ```
   - If successful → Browser tools are ready, proceed to Phase 1
   - If error → Continue to step 3

3. **Both tools failed - STOP IMMEDIATELY:**

   **Do NOT proceed. Output this exact message:**

   ```
   ❌ BROWSER TOOLS NOT AVAILABLE

   I cannot proceed with the task investigation because browser tools are required to observe and recreate the issue.

   Tested:
   - Claude in Chrome: [error message]
   - Playwright: [error message]

   To continue, please either:
   1. Fix the browser tool connection and tell me to retry
   2. Provide screenshots/recordings of the issue as an alternative
   3. Explicitly tell me to skip browser observation

   I will wait for your response before proceeding.
   ```

   **STOP HERE. Do not read documentation, do not parse the task, do not do anything else until the user responds.**

**Why this matters:**
- UI observation catches issues code analysis misses
- Recreating the bug ensures we're solving the right problem
- Screenshots document the actual vs expected behavior
- Skipping this step leads to incomplete understanding

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

### Subagent 4: Browser Exploration (uses Operation Lookup output)
```
Task tool:
  subagent_type: "general-purpose"
  model: "sonnet"
  description: "Browser exploration"
  prompt: "Follow the template at ~/.pdds-agents/templates/subagents/browser-verification.md

    Input:
    - navigation_steps: [steps from Subagent 3]
    - verifications: [
        {type: 'A-Navigation', check: 'Can reach target screen'},
        {type: 'B-State', check: 'Recreate the bug - [description]'},
        {type: 'B-State', check: 'Observe error state'}
      ]
    - expected_outcomes: {
        bug_behavior: '[what the bug looks like]'
      }"
```

**Execution order (strict - do not parallelize):**
1. Spawn Doc Lookup → **wait for completion** → get doc_summary
2. Spawn Code Lookup with doc_summary → **wait for completion**
3. Spawn Operation Lookup with doc_context → **wait for completion**
4. Spawn Browser Exploration with navigation_steps

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

### Handle Browser Subagent Questions

**If browser subagent returned `questions_for_user`, ask them now:**

```
AskUserQuestion:
Question: "[question from subagent]"
Header: "Browser"
Options: [suggested_options from subagent]
```

Prefix: "During browser exploration, I observed: [context]"

Record answers for the investigation.

### Your Job:
1. Combine findings into coherent understanding
2. Cross-reference: Do UI observations match docs?
3. Note discrepancies between documented and actual behavior
4. If bug not reproduced, clarify with user before proceeding

## Phase 4: Interview Developer

Use AskUserQuestion to fill gaps. Ask about:
- Unclear acceptance criteria
- Expected vs actual behavior
- Edge cases or related features
- Any context not in the task description

Keep questions focused and batched (2-3 at a time max).

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
/task-2-execute

Investigation: .agents/.scratch/investigation-[filename].md
```
```

Do not proceed to code until the developer confirms.