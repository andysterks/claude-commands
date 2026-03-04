# KB Build Scan - Inventory Documentable Items

You are scanning the codebase to discover all items that should be documented for the AI agent knowledge base.

## Workflow Position

This is **step 1 of 3** in the `kb-build` workflow. See `workflows.json` for details.

```
→ kb-build-1-scan (you are here)
  kb-build-2-research
  kb-build-3-write
```

**No workflow prerequisites** - this is the starting point. After completion, run `/kb-build-2-research [item-name]`.

## Prerequisites

Check if `.agents/AI-INSTRUCTIONS.md` exists. If not, inform user to run `/create-agents-folder` first and stop.

## Phase 1: Load Product/Project Context

Before scanning, read these files to understand what you're documenting:

1. **Read `.agents/product-overview.md`** - Understand the product, who uses it, core capabilities, key terminology
2. **Read `.agents/project-overview.md`** - Understand this repo's role (frontend, backend, jobs, etc.)

Use this context to:
- Recognize domain-specific terms vs generic code patterns
- Prioritize items that align with core capabilities
- Categorize items correctly (e.g., "Series" is a domain concept, not just a data structure)
- Understand user roles when documenting operations

## Phase 2: Scan Codebase

With product/project context in mind, spawn an **Explore subagent** with thoroughness "very thorough" to analyze the codebase:

```
Scan this codebase to identify ALL documentable items. Return structured JSON.

## Operations (UI Screens/Workflows/Endpoints)

Look for:
- Route definitions (React Router, Express routes, ASP.NET controllers)
- Page/screen components
- API endpoints
- Background jobs/workers
- CLI commands

For each, capture:
- name: Identifier (e.g., "patient-search", "image-viewer")
- type: "screen" | "endpoint" | "job" | "command"
- location: File path and line number
- route: URL pattern if applicable

## Reference (Domain Concepts)

Look for:
- Domain models/entities (classes, interfaces, types)
- Enums with business meaning
- Constants with domain significance
- Terms used frequently in comments/docs

For each, capture:
- name: Concept name (e.g., "DICOM", "AcquisitionDevice")
- type: "model" | "enum" | "constant" | "term"
- location: Where defined or most used
- frequency: Approximate usage count

## Codebase (Patterns/Integrations)

Look for:
- Authentication/authorization patterns
- Caching strategies
- Error handling patterns
- External service integrations (APIs, databases, queues)
- Shared utilities with significant logic

For each, capture:
- name: Pattern name (e.g., "JWT authentication", "Redis caching")
- type: "pattern" | "integration"
- location: Primary implementation file
- scope: "global" | "module-specific"

Return as JSON:
{
  "operations": [...],
  "reference": [...],
  "codebase": [...]
}
```

## Phase 3: Check Existing Documentation

**Load the structure schema** from `.agents/scripts/structure-schema.json` (or `~/.pdds-agents/templates/agents-lint/structure-schema.json` if not installed locally).

Read all files in each folder listed in `requiredFolders` (except `scripts`) to identify what's already documented.

Build a map of documented items by extracting:
- Filename (without .md)
- Keywords from the file content

## Phase 4: Generate Checklist

Create `.agents/CHECKLIST.md` with sections for each folder in `requiredFolders` from the schema (except `scripts`).

**Format:**

```markdown
# Knowledge Base Checklist

Generated: [ISO timestamp]
Last scan: [commit hash or "uncommitted"]

## How to Use

1. Pick an unchecked item below
2. Run `/kb-build-2-research [item-name]` to gather context
3. Run `/kb-build-3-write [item-name]` to interview and create the doc
4. Item will be marked complete automatically

---

[GENERATE SECTIONS FROM SCHEMA: Create a section for each folder in requiredFolders (except scripts). Group items by type within each section. Use this format for items:]

- [ ] `item-name` - [type/route info] - File: [path:line]
- [x] `documented-item` - Already documented: [folder]/[filename].md

---

## Summary

| Category | Total | Documented | Remaining |
|----------|-------|------------|-----------|
| [folder] | X | Y | Z |
| **Total** | **XX** | **YY** | **ZZ** |
```

## Phase 5: Report to User

Output a summary:

```markdown
## Knowledge Base Inventory Complete

Scanned codebase and found **[X] documentable items**.

| Category | Found | Already Documented | To Do |
|----------|-------|-------------------|-------|
| Operations | X | Y | Z |
| Reference | X | Y | Z |
| Codebase | X | Y | Z |

### Top Priority Items
1. [Most referenced undocumented model]
2. [Core screen with most routes]
3. [Key integration]

### Next Steps
- Review `.agents/CHECKLIST.md` for full list
- Run `/kb-build-2-research [name]` then `/kb-build-3-write [name]` to document items
- Re-run `/kb-build-1-scan` after major code changes to refresh
```

## Rules

- **DO NOT create any documentation** - only the checklist
- **DO NOT interview user** - this is automated scanning
- **Preserve existing checkmarks** - if CHECKLIST.md exists, merge new items and keep completion status
- **Sort by priority** - most-referenced/core items first within each section
- **Include file locations** - helps `/kb-build-2-research` find relevant code
- **Generic across repos** - no assumptions about specific frameworks
