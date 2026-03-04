# KB Build Write - Interview and Create Documentation

You are authoring documentation for ONE item by interviewing the user and creating the doc. This uses research from `/kb-build-2-research`.

## Arguments

`/kb-build-3-write [item-name]`

- If `item-name` provided: Author that specific item
- If omitted: Check for existing research files and offer options

## Workflow Position

This is **step 3 of 3** in the `kb-build` workflow. See `workflows.json` for details.

```
  kb-build-1-scan
  kb-build-2-research
→ kb-build-3-write (you are here)
```

## Prerequisite Check (MANDATORY)

**Before proceeding, verify the previous step was completed:**

Check for: `.agents/.scratch/[item-name]-research.json`

- **If found**: Continue to Phase 1
- **If NOT found**: Output this message and STOP:

```
⚠️ PREREQUISITE MISSING

This is step 3 of the **kb-build** workflow.
Step 2 (`/kb-build-2-research`) must be completed first for this item.

Expected artifact: `.agents/.scratch/[item-name]-research.json`

Workflow: /kb-build-1-scan → /kb-build-2-research [item] → /kb-build-3-write [item]
```

Also check `.agents/AI-INSTRUCTIONS.md` exists for templates.

## Phase 1: Load Context

### Load Product/Project Context
Read briefly to re-ground:
1. **Read `.agents/product-overview.md`** - Product context
2. **Read `.agents/project-overview.md`** - Repo role

### Load Research
Read `.agents/.scratch/[item-name]-research.json` to get:
- Item details (name, type, category, location)
- Code context (files, functions, patterns)
- RAG context (related docs, terminology, suggested links)
- Visual context (UI elements, labels, layout - if captured)
- Suggested questions

### If No Item Specified
List available research files in `.agents/.scratch/`:
```
Found research ready for authoring:

1. `patient-search` - researched 2 hours ago
2. `DICOM` - researched yesterday

Which would you like to document? Or run `/kb-build-2-research [name]` for a new item.
```

## Phase 2: Interview User

Based on category, ask targeted questions using `AskUserQuestion`. Use research to make questions specific and informed.

### For Operations (screens/endpoints/jobs)

**Question 1**: "What is the primary purpose of `[item-name]`?"
- Options: [Infer 3-4 likely purposes from research] + Other

**Question 2**: "Who typically uses this? (Select all that apply)"
- Options: [Pre-select from research's relevant_user_roles] + Other
- multiSelect: true

**Question 3**: "Are there any gotchas or non-obvious behaviors?"
- Options: [Infer from research's patterns_observed] + "None" + Other

**Question 4**: "What related features should be linked?"
- Options: [From research's suggested_links] + Other
- multiSelect: true

### For Reference (models/enums/terms)

**Question 1**: "In one sentence, what is `[item-name]`?"
- Options: [Infer definition from research] + Other

**Question 2**: "What's a common misconception or point of confusion?"
- Options: [Infer from code complexity] + "None" + Other

**Question 3**: "Are there related terms that should be defined together?"
- Options: [From research's terminology] + Other
- multiSelect: true

### For Codebase (patterns/integrations)

**Question 1**: "What problem does this pattern/integration solve?"
- Options: [Infer from research] + Other

**Question 2**: "What are the key configuration points?"
- Free text (show research's patterns_observed as hints)

**Question 3**: "What are common failure modes or debugging tips?"
- Free text

## Phase 3: Draft Document

Based on category, generate the appropriate document using templates from `.agents/AI-INSTRUCTIONS.md`.

### Operations Template
```markdown
# [Screen/Endpoint/Job Name]

**What**: [One sentence from interview]
**Where**: `[Route or trigger from research]`
**Parent**: [parent.md](./parent.md) (if applicable)

## Keywords
[item-name], [synonyms], [related terms from research]

## UI Elements
(Include if visual context was captured)
- [List key buttons, inputs, sections observed]
- [Use actual labels from visual_context.observed_labels]

## Steps
1. [Action-oriented steps inferred from research + interview + visual context]
2. [Use imperative voice]
3. [Reference actual UI labels when possible]

## Gotchas
- [From interview Q3]
- [Inferred from research patterns]

## See Also
- [Links from interview Q4]
- [Links from research]
```

### Reference Template
```markdown
# [Concept Name]

> **Source**: [Link if external standard, or "Internal"]

[One sentence definition from interview]

## Keywords
[item-name], [synonyms], [related terms]

| Term | Meaning |
|------|---------|
| [Sub-term 1] | [Definition] |
| [Sub-term 2] | [Definition] |

## Common Misconceptions
- [From interview Q2]

## See Also
- [Related terms from interview Q3]
- [Links from research]
```

### Codebase Template
```markdown
# [Pattern/Integration Name]

**Purpose**: [Problem it solves from interview]

## Keywords
[item-name], [related patterns], [technologies from research]

## How It Works
[Brief explanation from research code context]

## Configuration
[Key config points from interview]

## Debugging
- [Failure modes from interview]
- [Common issues from research patterns]

## See Also
- [Links from research]
```

## Phase 4: Review with User

Present the draft:
```markdown
## Draft: [filename].md

[Full document content]

---

Does this look correct?
- **Yes, save it** - Write file and mark checklist complete
- **Edit first** - Tell me what to change
- **Start over** - Re-interview with different focus
```

If user wants edits, make them and present again.

## Phase 5: Save and Update

### Write the Document
Save to the appropriate folder based on the item's category. The valid folders are defined in `structure-schema.json` under `requiredFolders` and `nestedRequirements`.

Path format: `.agents/[category]/[name].md`

For nested folders (e.g., integrations), use: `.agents/[parent]/[nested]/[name].md`

### Update INDEX.md
Add entry to the relevant INDEX.md:
```markdown
**Keywords**: [keywords from doc]
- [Document Name](./[name].md)
```

### Mark Checklist Complete
Update `.agents/CHECKLIST.md`:
- Change `- [ ] \`item-name\`` to `- [x] \`item-name\` - Documented: [path]`
- Update summary counts

### Clean Up Research
Delete `.agents/.scratch/[item-name]-research.json` (no longer needed)

### Commit (Optional)
Ask user: "Commit these changes?"
- If yes: `git add .agents/ && git commit -m "docs(kb): add [name] documentation"`

## Phase 6: Report

```markdown
## Documentation Complete

**Created**: `.agents/[category]/[name].md`
**Updated**: `.agents/[category]/INDEX.md`
**Checklist**: [X] items remaining

### Next Item
`[next-unchecked-item]` - [brief description]

Run `/kb-build-2-research [next-item]` then `/kb-build-3-write [next-item]` to continue.
```

## Rules

- **Require research first** - Don't proceed without research file
- **Fresh context** - Don't assume knowledge from previous runs
- **User has final say** - Always show draft before saving
- **Respect token limits** - Keep docs under 500 tokens
- **Clean up** - Delete research file after successful authoring
- **Generic** - Works for any repo with `.agents/` structure
