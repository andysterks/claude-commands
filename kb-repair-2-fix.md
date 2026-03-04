# KB Repair Fix - Plan and Execute Documentation Repairs

You are repairing non-standard `.agents/` documentation to match the standard structure.

## Workflow Position

This is **step 2 of 3** in the `kb-repair` workflow. See `workflows.json` for details.

```
  kb-repair-1-analyze
→ kb-repair-2-fix (you are here)
  kb-repair-3-validate
```

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/.scratch/audit-state.json` with `phase="scanned"`

- **If found**: Continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 2 of the **kb-repair** workflow.
Step 1 (`/kb-repair-1-analyze`) must be completed first.

Expected artifact: `.agents/.scratch/audit-state.json`

Run `/kb-repair-1-analyze` to start the workflow.
```

## Phase 1: Load State

Load findings from `.agents/.scratch/audit-state.json` and continue.

## Phase 2: Interview User About Repairs

Spawn a **general-purpose subagent** to conduct interviews and build repair plan:

```
You have these scan findings:
{findings from state file}

Interview the user about each non-standard item using AskUserQuestion.

INTERVIEW RULES:
- Ask ONE question at a time
- Group related items (e.g., all infrastructure docs together)
- Provide sensible defaults as first option

QUESTION TEMPLATES:

For infrastructure/devops docs (NGINX.md, PM2.md, SSL.md, etc.):
"Found infrastructure docs: [list]. Where should these go?"
Options:
- codebase/devops/ (Recommended)
- codebase/integrations/
- Delete (redundant)

For quickstart/setup docs:
"Found setup docs: [list]. Where should these go?"
Options:
- codebase/ (as deployment.md, setup.md)
- reference/setup.md
- Delete (redundant with root README)

For product/domain docs:
"Found product docs: [list]. Where should these go?"
Options:
- Merge into product-overview.md (Recommended)
- reference/
- Delete

For technical/api/database docs:
"Found technical docs: [list]. Where should these go?"
Options:
- codebase/
- reference/
- operations/
- Delete

For AI-INSTRUCTIONS.md:
"Found existing AI-INSTRUCTIONS.md. What should we do?"
Options:
- Keep existing
- Enhance with lazy loading policy
- Replace with standard template

For missing lint scripts:
"Lint scripts are missing. Install them?"
Options:
- Yes, install both (Recommended)
- Skip

For missing git hooks:
"Git pre-commit hook is missing/incomplete. Install linting hook?"
Options:
- Yes, install hook (Recommended)
- Skip

BUILD REPAIR PLAN as JSON:
{
  "moves": [
    {"from": "docs/infrastructure/NGINX.md", "to": "codebase/devops/NGINX.md"}
  ],
  "deletes": [
    {"path": "docs/technical/DATABASE.md", "reason": "Duplicates root schema"}
  ],
  "creates": [
    {"path": "codebase/devops/INDEX.md", "template": "index"}
  ],
  "merges": [
    {"sources": ["docs/product/OVERVIEW.md"], "target": "product-overview.md"}
  ],
  "aiInstructions": "keep|enhance|replace",
  "installLintScripts": true|false,
  "installGitHooks": true|false,
  "foldersToRemove": ["docs/"]
}
```

## Phase 3: Present Repair Plan

Display the plan in a clear table format:

```markdown
## Proposed Repair Plan

### Files to Move
| From | To |
|------|-----|
| docs/infrastructure/NGINX.md | codebase/devops/NGINX.md |

### Files to Delete
| File | Reason |
|------|--------|
| docs/technical/DATABASE.md | Duplicates root schema |

### New INDEX.md Files
- codebase/devops/INDEX.md

### AI-INSTRUCTIONS.md
- Action: [Keep|Enhance with lazy loading|Replace]

### Tooling
- Install lint scripts: [Yes|No]
- Install git hooks: [Yes|No]

### Cleanup
- Remove folders: docs/
```

Then ask:
```
"Ready to execute this repair plan?"
Options:
- Proceed with all changes
- Review each change individually
- Cancel
```

## Phase 4: Execute Repairs

**Only after explicit user approval.**

Spawn a **Bash subagent** for file operations:

```
Execute these file operations for .agents/ repair.

OPERATIONS:
{migration plan JSON}

RULES:
- Use `git mv` for tracked files to preserve history
- Create directories before moving files into them
- Use `cp` then `rm` if git mv fails
- Report each operation result

SEQUENCE:
1. Create new directories (mkdir -p)
2. Move/copy files to new locations
3. Create new INDEX.md files with template content
4. Remove old directories (only after confirming moves succeeded)
5. Report summary of operations
```

## Phase 5: Update INDEX Files

For each folder that received new files, update its INDEX.md with keyword entries:

```markdown
**Keywords**: nginx, reverse proxy, ssl, configuration
- [NGINX Configuration](./NGINX.md)
```

## Phase 6: Install Tooling (If Approved)

If user approved lint scripts:
1. Create `.agents/scripts/` if missing
2. Read lint script templates from `~/.pdds-agents/templates/agents-lint/`
3. If templates don't exist, use embedded minimal versions
4. Write to `.agents/scripts/`:
   - `lint-docs.js`
   - `lint-structure.js`
   - `structure-schema.json` (single source of truth for allowed structure)

If user approved git hooks:
1. Check for existing `.git/hooks/pre-commit`
2. If exists, ask whether to append or replace
3. Install hook that runs both lint scripts

## Phase 7: Update State

Update `.agents/.scratch/audit-state.json`:

```json
{
  "phase": "repaired",
  "timestamp": "ISO timestamp",
  "findings": { /* original findings */ },
  "plan": { /* executed plan */ },
  "results": {
    "moved": 5,
    "deleted": 2,
    "created": 3,
    "errors": []
  }
}
```

## Phase 8: Report

```markdown
## Repair Complete

### Summary
- Moved: X files
- Deleted: X files
- Created: X INDEX.md files
- Lint scripts: [Installed|Skipped|Already present]
- Git hooks: [Installed|Skipped|Already present]

### Folders Removed
- docs/ (old nested structure)

### Next Step
Run `/kb-repair-3-validate` to verify the repairs.
```

## Rules

- **NEVER execute without user approval from Phase 3**
- **NEVER delete files without confirmation**
- **Preserve git history** with `git mv` when possible
- **Stop on errors** and report what succeeded/failed
