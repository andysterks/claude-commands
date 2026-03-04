# Create Agents Folder

You are setting up the `.agents/` documentation structure for just-in-time AI agent context.

## Phase 1: Detect Project Type

Scan the repository for indicators:

| Indicator | Project Type |
|-----------|--------------|
| `package.json` with React/Vue/Angular | frontend |
| `package.json` with Express/Fastify/Nest | backend/api |
| `.csproj` with ASP.NET | backend/api |
| `Dockerfile` + queue/worker keywords | jobs/workers |
| SQL migrations folder, `.sql` files | database |
| `schema.prisma`, `migrations/` | database |

If unclear, default to `backend/api` as the most common.

## Phase 2: Interview User

Use `AskUserQuestion` to confirm:

1. **Project type**: "Detected [type]. Is this correct?"
   - Options: frontend, backend/api, database, jobs/workers, Other

2. **Product name**: "What is the product name? (Used in product-overview.md)"
   - Free text response expected

3. **Project role**: "In one sentence, what does THIS repo do within the product?"
   - Free text response expected (this becomes project-overview.md)

## Phase 3: Create Structure

**Load the structure schema** from `~/.pdds-agents/templates/agents-lint/structure-schema.json` - this is the single source of truth for the `.agents/` folder structure.

The schema defines:
- `requiredFiles` - Files to create at `.agents/` root
- `requiredFolders` - Folders to create at `.agents/` root (each gets an `INDEX.md`)
- `optionalItems` - Items that may exist (like `.scratch/`)
- `nestedRequirements` - Required subfolders (e.g., `codebase/integrations/`)

**Create the structure based on the schema:**
1. Create `.agents/` directory
2. Create each file in `requiredFiles` using templates below
3. Create each folder in `requiredFolders` with an `INDEX.md` inside
4. Create nested folders from `nestedRequirements` with `INDEX.md` files
5. Create `.scratch/` folder (from `optionalItems`) - this is gitignored, used for temp workflow state
6. Copy lint scripts to `.agents/scripts/`

### File Contents

**AI-INSTRUCTIONS.md** (use template below, fill in [PRODUCT_NAME]):
```markdown
# Instructions for AI Agents

This documentation helps you understand [PRODUCT_NAME]. Read this file first.

## How to Find What You Need

1. **Start here**: Read `product-overview.md` for product context
2. **This repo**: Read `project-overview.md` for this repo's role
3. **Find operations**: Search `operations/INDEX.md` for domain operations
4. **Find terminology**: Search `reference/INDEX.md` for definitions
5. **Find patterns**: Search `codebase/INDEX.md` for code descriptions and standards
6. **Load on-demand**: Only load docs as needed for your task

## Documentation Structure

[GENERATE FROM SCHEMA: Build a tree visualization from `.agents/scripts/structure-schema.json` showing requiredFiles, requiredFolders with INDEX.md files, and nestedRequirements. Add brief comments explaining each item's purpose.]

## Token Limits

Run `node .agents/scripts/lint-docs.js` to check limits before committing.

| Doc Type | Limit | Purpose |
|----------|-------|---------|
| AI-INSTRUCTIONS.md | 1000 | Entry point |
| product-overview.md | 500 | Product context |
| project-overview.md | 100 | Repo role (brief!) |
| operations/*.md | 500 | Domain operations |
| reference/*.md | 500 | Terminology |
| codebase/*.md | 500 | Code descriptions & standards |
| **/INDEX.md | 200 | Keyword indexes |

## Templates

### Operations (operations/[domain].md)
Group related operations by domain. Example: users.md covers create, update, delete.

### Reference (reference/[concept].md)
Definitions and lookup tables. No how-to steps.

### Codebase (codebase/[topic].md)
Code descriptions (file roles, service layers, how components connect) and coding standards (error handling conventions, naming patterns, testing approaches). If you need to understand how the code works or what pattern to follow, look here.

### Integrations (codebase/integrations/[service].md)
External services this repo depends on: APIs, databases, queues.

## Writing Rules

1. **Operations = what you can do**: UI "flows", Domain actions, API endpoints, jobs
2. **Reference = definitions**: Reusable lookups, generic domain knowledge, no action steps
3. **Codebase = how the code works**: Code descriptions and coding standards
4. **Brief is better**: Stay under token limits, link for details
```

**product-overview.md**:
```markdown
# [PRODUCT_NAME] Product Overview

[Describe what the product is and who uses it]

## Who Uses It

- **[Role 1]** - [What they do]
- **[Role 2]** - [What they do]

## Core Capabilities

| Capability | Description |
|------------|-------------|
| [Capability 1] | [Description] |

## Key Terminology

| Term | Meaning |
|------|---------|
| [Term] | [Definition] |
```

**project-overview.md**:
```markdown
# Project Overview

[PROJECT_ROLE - one sentence describing this repo's role]

**Type**: [PROJECT_TYPE]
**Repo**: [Repo name or path]
```

**INDEX.md files** (all use same format):
```markdown
# [Folder Name] Index

Search this file for keywords. Add entries as documentation grows.

---

<!-- Example entry format:
**Keywords**: keyword1, keyword2, keyword3
- [Document Name](./document.md)
-->
```

### Lint Scripts

Copy lint scripts and schema from templates:
- `~/.pdds-agents/templates/agents-lint/lint-docs.js` → `.agents/scripts/lint-docs.js`
- `~/.pdds-agents/templates/agents-lint/lint-structure.js` → `.agents/scripts/lint-structure.js`
- `~/.pdds-agents/templates/agents-lint/structure-schema.json` → `.agents/scripts/structure-schema.json`

The `structure-schema.json` in `.agents/scripts/` becomes the local source of truth for this repo's structure validation.

## Phase 4: Update .gitignore

Append to `.gitignore` (create if doesn't exist):
```
# AI agent scratch space
.agents/.scratch/
```

## Phase 5: Add npm Scripts (if applicable)

If `package.json` exists, add these scripts:
```json
"lint:docs": "node .agents/scripts/lint-docs.js",
"lint:structure": "node .agents/scripts/lint-structure.js",
"lint:agents": "node .agents/scripts/lint-structure.js && node .agents/scripts/lint-docs.js"
```

## Phase 6: Install Git Hooks (Optional)

Use `AskUserQuestion` to ask the user:

**Question**: "Would you like to install a git pre-commit hook to enforce documentation linting?"
- **Options**:
  - "Yes (Recommended)" - Install pre-commit hook that runs both linters
  - "No" - Skip hook installation

### If user approves:

Create `.git/hooks/pre-commit` with this content:

```bash
#!/bin/sh

# Pre-commit hook for .agents/ documentation linting
# Runs both token limit and structure validation

echo "Running .agents/ documentation lints..."
echo ""

# Run token limit check
node .agents/scripts/lint-docs.js
DOCS_EXIT=$?

echo ""

# Run structure check
node .agents/scripts/lint-structure.js
STRUCTURE_EXIT=$?

echo ""

# Fail if either check failed
if [ $DOCS_EXIT -ne 0 ] || [ $STRUCTURE_EXIT -ne 0 ]; then
    echo "Pre-commit hook failed. Fix the issues above before committing."
    exit 1
fi

echo "All .agents/ checks passed."
exit 0
```

**Note**: The `.git/hooks/` directory is local and not version-controlled. If the team needs shared hooks, consider documenting how to set them up or using a tool like husky/lefthook.

### If user declines:

Skip hook installation. Inform the user they can manually run lints with:
- `node .agents/scripts/lint-docs.js`
- `node .agents/scripts/lint-structure.js`

## Phase 7: Report

When complete, output:

```
## Documentation Initialized

**Product**: [PRODUCT_NAME]
**Project Type**: [TYPE]
**Project Role**: [ONE_SENTENCE]

### Created Structure
- .agents/AI-INSTRUCTIONS.md
- .agents/product-overview.md
- .agents/project-overview.md
- .agents/operations/INDEX.md
- .agents/reference/INDEX.md
- .agents/codebase/INDEX.md
- .agents/codebase/integrations/INDEX.md
- .agents/scripts/lint-docs.js
- .agents/scripts/lint-structure.js
- .agents/.scratch/ (gitignored)
- Updated .gitignore
- Added npm scripts (if package.json exists)
- Installed pre-commit hook (if approved)

### Next Steps
1. Fill in product-overview.md with product details
2. Run `node .agents/scripts/lint-docs.js` to verify setup
3. Add operations/reference/codebase docs as needed
```

## Rules

- **DO NOT fill in operations/reference/codebase content** - only create empty INDEX.md stubs
- **Always create all folders** even if the project type doesn't obviously need them
- **Keep INDEX.md minimal** - just header and example comment
- **project-overview.md must be under 100 tokens** - enforce brevity

---

## Appendix A: Template Location

All templates are in `~/.pdds-agents/templates/agents-lint/`:
- `lint-docs.js` - Token limit validation
- `lint-structure.js` - Structure validation
- `structure-schema.json` - **Single source of truth** for folder structure

If templates don't exist, the user needs to install the PDDS.Agents shared commands first.
