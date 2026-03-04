# KB Repair Analyze - Documentation Structure Analysis

You are scanning the `.agents/` directory to identify non-standard structure and items needing repair.

## Workflow Position

This is **step 1 of 3** in the `kb-repair` workflow. See `workflows.json` for details.

```
→ kb-repair-1-analyze (you are here)
  kb-repair-2-fix
  kb-repair-3-validate
```

**No workflow prerequisites** - this is the starting point. After completion, run `/kb-repair-2-fix`.

## Prerequisites

Check if `.agents/` exists. If not, inform user to run `/create-agents-folder` first and stop.

## Execute Scan

Spawn an **Explore subagent** with this prompt:

```
Scan the .agents/ directory and report findings in JSON format.

FIRST: Load the structure schema to determine what's standard vs non-standard.
Look for the schema in this order:
1. .agents/scripts/structure-schema.json (if lint scripts installed)
2. ~/.pdds-agents/templates/agents-lint/structure-schema.json (template location)

The schema defines:
- requiredFiles: files that MUST exist at .agents/ root
- requiredFolders: folders that MUST exist at .agents/ root
- optionalItems: items that MAY exist (like .scratch/)
- nestedRequirements: required subfolders (e.g., codebase/integrations/)

Check for:

1. NON-STANDARD ROOT FILES - Files not in requiredFiles or optionalItems

2. NON-STANDARD ROOT FOLDERS - Folders not in requiredFolders or optionalItems

3. NESTED STRUCTURES - Look for:
   - docs/ subdirectory with nested folders
   - technical/, product/, infrastructure/, quickstart/ folders
   - Any multi-level folder hierarchies

4. MISSING STANDARD ITEMS - Check if requiredFiles and requiredFolders exist

5. MISSING LINT SCRIPTS - Check for:
   - .agents/scripts/lint-docs.js
   - .agents/scripts/lint-structure.js
   - .agents/scripts/structure-schema.json

6. GIT HOOKS STATUS - Check:
   - .git/hooks/pre-commit exists?
   - Does it reference lint-structure.js or lint-docs.js?
   - Or check .husky/pre-commit, package.json lint-staged config

Return findings as JSON:
{
  "nonStandardFiles": ["file1.md", "file2.md"],
  "nonStandardFolders": ["docs/", "old-docs/"],
  "nestedStructures": [
    {"path": "docs/infrastructure/", "files": ["NGINX.md", "PM2.md"]},
    {"path": "docs/quickstart/", "files": ["DEPLOYMENT.md"]}
  ],
  "missingStandard": {
    "files": ["product-overview.md"],
    "folders": ["reference/"]
  },
  "lintScripts": {
    "lintDocs": true|false,
    "lintStructure": true|false,
    "structureSchema": true|false
  },
  "gitHooks": {
    "preCommitExists": true|false,
    "runsLinting": true|false,
    "hookType": "native|husky|lint-staged|none"
  },
  "totalMdFiles": 15,
  "estimatedMigrations": 8
}
```

## Save State

Write the findings to `.agents/.scratch/audit-state.json`:

```json
{
  "phase": "scanned",
  "timestamp": "ISO timestamp",
  "findings": { /* subagent response */ }
}
```

Create `.agents/.scratch/` directory if it doesn't exist.

## Report to User

Present a human-readable summary:

```markdown
## Analysis Complete

### Current Structure
- Total .md files: X
- Standard structure: [Complete|Partial|Missing]

### Non-Standard Items Found
| Type | Items |
|------|-------|
| Root files | file1.md, file2.md |
| Root folders | docs/, old-docs/ |
| Nested structures | docs/infrastructure/ (3 files), docs/quickstart/ (2 files) |

### Missing Standard Items
- [ ] product-overview.md
- [ ] reference/ folder

### Tooling Status
- Lint scripts: [Installed|Missing]
- Git hooks: [Configured|Missing|Partial]

### Next Step
Run `/kb-repair-2-fix` to plan and execute repairs.
```

## Rules

- **DO NOT modify any files** - this is read-only scanning
- **DO NOT interview user** - just report findings
- **ALWAYS save state** - enables `/kb-repair-2-fix` to continue
