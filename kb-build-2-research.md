# KB Build Research - Gather Context for One Item

You are researching ONE item from the checklist by gathering code context and related documentation. This prepares for `/kb-build-3-write` which will interview the user and create the doc.

## Arguments

`/kb-build-2-research [item-name]`

- If `item-name` provided: Research that specific item
- If omitted: Show next unchecked item and ask user to confirm

## Workflow Position

This is **step 2 of 3** in the `kb-build` workflow. See `workflows.json` for details.

```
  kb-build-1-scan
→ kb-build-2-research (you are here)
  kb-build-3-write
```

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/CHECKLIST.md`

- **If found**: Continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 2 of the **kb-build** workflow.
Step 1 (`/kb-build-1-scan`) must be completed first.

Expected artifact: `.agents/CHECKLIST.md`

Run `/kb-build-1-scan` to start the workflow.
```

Also check `.agents/AI-INSTRUCTIONS.md` exists. If not, run `/create-agents-folder` first.

## Phase 1: Load Product/Project Context

Read these files to understand what you're researching:

1. **Read `.agents/product-overview.md`** - Understand the product, who uses it, core capabilities, key terminology
2. **Read `.agents/project-overview.md`** - Understand this repo's role

Use this context to:
- Recognize domain-specific patterns in the code
- Understand how this item fits the product
- Identify relevant user roles

## Phase 2: Load Item Context

### Find the Item
Parse `.agents/CHECKLIST.md` to find the item:
- Extract: name, type, category (operations/reference/codebase), file location
- If item not found, show available items and ask user to pick one

### If No Item Specified
Show the first 5 unchecked items and ask:
```
Which item would you like to research?

1. `patient-search` (operations/screen)
2. `DICOM` (reference/term)
3. `authentication-flow` (codebase/pattern)
4. `Patient` (reference/model)
5. `image-viewer` (operations/screen)

Or specify a different item name.
```

## Phase 3: Gather Code Context

Read the source file(s) associated with this item:
- Primary file from checklist location
- Related files (imports, tests, usages)
- Look for comments, JSDoc, or inline documentation

Capture:
- **Code snippets**: Key functions, interfaces, types (limit ~2000 lines total)
- **Dependencies**: What this item imports/uses
- **Dependents**: What uses this item
- **Patterns**: Observable coding patterns, error handling, etc.

## Phase 4: Query RAG for Related Docs

Query the RAG API to find related existing documentation:

```
GET /query?q=[item-name] [item-type] [key-terms]&top_n=5
```

Capture:
- **Related docs**: Titles and paths of relevant existing docs
- **Terminology**: How similar concepts are described
- **Links**: Docs that should appear in "See Also"
- **Gaps**: What's NOT documented that this item depends on

## Phase 5: Visual Exploration (Screens Only)

**Skip this phase if:**
- Item is not a screen (endpoints, models, patterns, etc.)
- No `.agents/app-config.json` exists
- Browser MCP tools are not available (no `mcp__claude-in-chrome__*` tools)

### 5a: Check App Config

Read `.agents/app-config.json` for startup instructions:

```json
{
  "frontend": {
    "node_version": "16.16",
    "setup": ["yarn run vendors"],
    "start": [
      { "command": "yarn run server", "background": true },
      { "command": "yarn run watchDefault", "background": true }
    ],
    "ready_url": "http://localhost:8080"
  },
  "backend": {
    "repo_marker": "PDDS.csproj",
    "search_paths": ["~/source/repos", "C:\\Users\\*\\source\\repos"],
    "branch": "master",
    "start": "dotnet run",
    "cwd_relative_to_marker": ".",
    "ready_url": "http://localhost:8080/api/health"
  }
}
```

### 5b: Check if App is Running

Hit `frontend.ready_url` (default `http://localhost:8080`).
- If responding: Skip to step 5e (visual exploration)
- If not responding: Proceed to start the app

### 5c: Start Backend (if configured)

1. **Find backend repo**: Search `backend.search_paths` for `backend.repo_marker` file
2. **Create worktree**: In the backend repo directory:
   ```bash
   git worktree add .worktrees/kb-build-2-research-master <backend.branch> 2>/dev/null || true
   ```
3. **Start backend**:
   ```bash
   cd <worktree>/<cwd_relative_to_marker>
   <backend.start>  # Run in background
   ```
4. **Wait for ready**: Poll `backend.ready_url` until responding (timeout 60s)

### 5d: Start Frontend

1. **Check node version**:
   ```bash
   node -v  # If not <node_version>, run: nvm use <node_version>
   ```
2. **Run setup commands**: Execute each command in `frontend.setup` sequentially
3. **Start app**: Execute each command in `frontend.start` (background commands in separate processes)
4. **Wait for ready**: Poll `frontend.ready_url` until responding (timeout 60s)

### 5e: Visual Exploration

1. **Get browser context**: Call `mcp__claude-in-chrome__tabs_context_mcp`
2. **Create new tab**: Call `mcp__claude-in-chrome__tabs_create_mcp`
3. **Navigate to screen**:
   - Construct URL from item's route (e.g., `http://localhost:8080/patients/search`)
   - Call `mcp__claude-in-chrome__navigate`
4. **Wait for load**: Call `mcp__claude-in-chrome__browser_wait_for` (2-3 seconds)
5. **Capture snapshot**: Call `mcp__claude-in-chrome__browser_snapshot` to get accessibility tree
6. **Take screenshot**: Call `mcp__claude-in-chrome__computer` with action "screenshot"

Capture from visual exploration:
- **UI elements**: Buttons, forms, inputs, navigation observed
- **Layout**: How the screen is organized
- **States**: Empty states, loading states if visible
- **Labels**: Actual text shown to users

### 5f: Cleanup (Optional)

Leave the app running for subsequent research. The worktree persists for future runs.

To manually clean up later:
```bash
git worktree remove .worktrees/kb-build-2-research-master
```

## Phase 6: Save Research

Create `.agents/.scratch/[item-name]-research.json` (includes visual context if captured):

```json
{
  "item": {
    "name": "patient-search",
    "type": "screen",
    "category": "operations",
    "location": "src/pages/PatientSearch.tsx:15",
    "route": "/patients/search"
  },
  "product_context": {
    "product_name": "PDDS",
    "project_type": "frontend",
    "relevant_capabilities": ["Search", "Patient management"],
    "relevant_user_roles": ["Front desk staff", "Dentists"]
  },
  "code_context": {
    "primary_file": "src/pages/PatientSearch.tsx",
    "related_files": ["src/api/patients.ts", "src/components/PatientList.tsx"],
    "key_functions": ["searchPatients", "handleSearch", "formatResults"],
    "dependencies": ["react-query", "PatientAPI"],
    "patterns_observed": ["Debounced search", "Pagination", "Error boundary"]
  },
  "rag_context": {
    "related_docs": [
      {"title": "Patient Model", "path": "reference/patient.md"},
      {"title": "API Authentication", "path": "codebase/auth.md"}
    ],
    "terminology": {
      "Patient": "Individual receiving dental care",
      "Series": "Collection of related images"
    },
    "suggested_links": ["reference/patient.md", "operations/image-viewer.md"]
  },
  "visual_context": {
    "captured": true,
    "screenshot_path": ".agents/.scratch/patient-search-screenshot.png",
    "ui_elements": ["Search input", "Filter dropdown", "Results table", "Pagination"],
    "observed_labels": ["Search patients", "Filter by date", "Name", "DOB", "Last visit"],
    "layout_notes": "Search bar at top, filters on left sidebar, results table in main area"
  },
  "suggested_questions": [
    "What filters are most commonly used?",
    "Are there any search behaviors that surprise users?",
    "What happens when no results are found?"
  ],
  "researched_at": "2024-01-15T10:30:00Z"
}
```

Create `.agents/.scratch/` directory if it doesn't exist.

## Phase 7: Report to User

Output a summary:

```markdown
## Research Complete: `[item-name]`

### Item Details
- **Type**: [screen/endpoint/model/etc.]
- **Category**: [operations/reference/codebase]
- **Location**: [file:line]

### Code Context Gathered
- Primary file: [filename]
- Related files: [count] files
- Key functions: [list]

### Visual Exploration
- **Status**: [Captured / Skipped (reason)]
- **UI Elements**: [list if captured]
- **Screenshot**: [path if captured]

### Related Documentation Found
- [X] related docs in knowledge base
- Suggested links: [list]

### Ready for Authoring
Research saved to `.agents/.scratch/[item-name]-research.json`

**Next step**: Run `/kb-build-3-write [item-name]` to interview and create documentation.
```

## Rules

- **DO NOT create documentation** - only gather research
- **DO NOT interview user** - save questions for `/kb-build-3-write`
- **Always save research** - enables `/kb-build-3-write` to work with fresh context
- **Overwrite existing research** - if re-running, replace old research file
- **Stay focused** - research ONE item, don't explore tangents
- **Generic** - works for any repo with `.agents/` structure
