# KB Update

Spawn a **general-purpose subagent** to analyze `.agents/` documentation encountered during this conversation.

## Subagent Instructions

Use the Task tool with `subagent_type: "general-purpose"` and this prompt:

---

You are reviewing `.agents/` documentation based on what was learned in this conversation.

**Purpose**: The `.agents/` folder provides **just-in-time context for AI agents**. The goals are:
1. **Minimize tokens loaded** - Agents should load only what they need, when they need it
2. **Minimize exploration** - Docs should answer questions directly so agents don't need to search the codebase
3. **Keyword-searchable** - INDEX.md files let agents find the right doc by searching for terms

**What belongs in each section:**
- `operations/` = **What you can do** - Domain actions, API endpoints, jobs (how to perform tasks)
- `reference/` = **Definitions** - Reusable lookups, terminology, no step-by-step instructions
- `codebase/` = **How the code works** - Code descriptions (file roles, layering) and coding standards (error handling, naming, testing patterns)
- `integrations/` = **External dependencies** - APIs, databases, services this repo connects to

**What makes good docs in this system:**
- Brief is better - stay under token limits, link for details
- Searchable - use keywords agents would search for
- Actionable - agents should know what to do after reading
- No redundancy - don't repeat what's in code comments or obvious from structure

**Structure**: The `.agents/` folder contains:
- `AI-INSTRUCTIONS.md` - Entry point (1000 token limit)
- `product-overview.md` - Product context (500 token limit)
- `project-overview.md` - Repo role (100 token limit)
- `operations/` - Domain operations with INDEX.md (500 token limit each)
- `reference/` - Terminology with INDEX.md (500 token limit each)
- `codebase/` - Code descriptions & standards with INDEX.md, includes `integrations/` (500 token limit each)
- All INDEX.md files have 200 token limit

**Analyze based on your conversation experience:**

1. **Missing information** - What would have saved time if documented?
2. **Misleading content** - What led you in the wrong direction?
3. **Navigation issues** - Did INDEX.md files point to the right docs?
4. **Structure problems** - Should docs be split, merged, or reorganized?
5. **Token efficiency** - Is any doc bloated or missing key details?

**Output format:**

```
## Documentation Review

### Summary
[One sentence: helpful/needs work/significant gaps]

### Suggestions (max 5, ranked by impact)

#### 1. [Title]
- **Type**: [missing | misleading | navigation | structure | efficiency]
- **File**: [path or "new file needed"]
- **Problem**: [What went wrong]
- **Proposed fix**: [Specific change]

[Repeat for each suggestion]

### No Changes Needed
[If docs were helpful, say so. Don't invent problems.]
```

**Rules:**
- Only suggest changes with clear justification from the conversation
- Be specific about file paths and content changes
- Prioritize high-impact, low-effort fixes
- If docs were adequate, report that honestly

---

After the subagent returns, display the report and ask which suggestions to implement.
