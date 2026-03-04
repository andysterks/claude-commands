# KB Interview - Research, Test, Interview, and Document a Feature

You are conducting a comprehensive documentation session for a feature/workflow in this repository. Your job is to research the codebase, smoke test the feature in a browser, interview the user (and optionally their product owner), and produce `.agents/` documentation.

## Arguments

`/kb-interview [feature-name]`

- If `feature-name` provided: Use it as the topic
- If omitted: Ask "What feature or workflow do you want to document?" using AskUserQuestion

## Phase 0: Setup

### Read .agents structure
1. Read `.agents/AI-INSTRUCTIONS.md` for templates and conventions

### Identify participants
Ask using AskUserQuestion:
- "Who is present for this session?"
  - Options: "PO + Developer (both)", "Just the PO", "Just a developer", "Other"
- Adjust your question style based on the answer:
  - **PO present**: Focus on business rules, user workflows, why things work the way they do, edge cases from a user perspective
  - **Developer present**: Can ask about technical implementation, code patterns, API behavior
  - **Both**: Ask both types but label which audience each question targets

### Create scratch workspace
Create `.agents/.scratch/kb-interview-session.md` with this initial structure:

```markdown
# KB Interview Session: [feature-name]
Started: [current date/time]
Participants: [who's present]
Topic: [feature-name]

## Research Findings
(Populated by Phase 1)

## Browser Observations
(Populated by Phase 2)

## Interview Notes

### Round 1: Business Rules
(Populated during Phase 3)

### Round 2: UI Workflow
(Populated during Phase 3)

### Round 3: Edge Cases & Gotchas
(Populated during Phase 3)

### Round 4: Permissions & Configuration
(Populated during Phase 3)

## Discrepancies Found
(Code vs. interview conflicts)

## Documentation Plan
(What docs to create, populated after interview)
```

## Phase 1 + 2: Parallel Research (Background Subagents)

Launch both subagents in parallel using the Task tool with `run_in_background: true`. Do NOT wait for them to finish before starting Phase 3 setup — but DO wait for their results before starting the interview rounds.

### Subagent 1: Code Research (Explore agent)

Launch a Task with `subagent_type: "Explore"`. Read `templates/subagents/kb-research.md` for the prompt template. Pass `feature_name: [feature-name]` as input.

### Subagent 2: Browser Smoke Test (general-purpose agent)

Launch a Task with `subagent_type: "general-purpose"`. Read `templates/subagents/kb-browser-test.md` for the prompt template. Pass `feature_name: [feature-name]` as input.

### After subagents complete

When both subagents return results:
1. Write their findings into the appropriate sections of `.agents/.scratch/kb-interview-session.md`
2. Do NOT present these findings to the user — save them for cross-referencing during the interview

## Phase 3: Interview (Main Agent, Phased Rounds)

Conduct the interview using AskUserQuestion. Ask 2-4 focused questions per round. After each round, append a structured summary of the answers to `.agents/.scratch/kb-interview-session.md` under the appropriate section.

**IMPORTANT**: Do NOT share your research findings upfront. Interview fresh. Only reference your findings when you detect a discrepancy between what the user says and what the code/browser showed.

### Round 1: Business Rules

Focus: What is this feature, why does it exist, who uses it?

Example questions (adapt based on feature):
- "In one sentence, what does [feature-name] do and why does it exist?"
- "What triggers this feature? Is it automatic, manual, or both?"
- "Who are the primary users of this feature? What roles?"
- "What are the core business rules? What MUST be true for this to work correctly?"

After answers: Append summary to scratch file under "Round 1: Business Rules".

### Round 2: UI Workflow

Focus: Step-by-step user journey through the feature.

Example questions:
- "Walk me through the happy path — what does the user see first, and what do they do?"
- "What choices does the user make along the way? Any multi-step flows?"
- "What happens when the action completes successfully? What does the user see?"
- "Can the user cancel or undo at any point?"

After answers: Append summary to scratch file under "Round 2: UI Workflow".

### Round 3: Edge Cases & Gotchas

Focus: What goes wrong, what's non-obvious, what's hacky?

Example questions:
- "What are the most common mistakes users make with this feature?"
- "Are there any known bugs, glitches, or workarounds?"
- "What happens with bad data, missing data, or unexpected input?"
- "Are there any undocumented behaviors or 'gotchas' that trip people up?"

After answers: Append summary to scratch file under "Round 3: Edge Cases & Gotchas".

### Round 4: Permissions & Configuration

Focus: Who can do this, what settings affect it, environment differences.

Example questions:
- "What permissions or roles are required to use this feature?"
- "Are there any site/location settings that affect how this works?"
- "Does this behave differently across environments (dev, staging, prod)?"
- "Are there any feature flags or configuration that enable/disable this?"

After answers: Append summary to scratch file under "Round 4: Permissions & Configuration".

### Cross-Reference Check

After all rounds, review your research findings against the interview answers:
- If you find discrepancies (e.g., PO says "only admins" but code checks a different permission), flag them:
  - Present the discrepancy clearly using AskUserQuestion
  - Ask: "The code shows [X] but you mentioned [Y]. Which is correct, or is there nuance I'm missing?"
- Append any discrepancy resolutions to the "Discrepancies Found" section of the scratch file

### Wrap-Up Check

Ask using AskUserQuestion:
- "Is there anything else about [feature-name] we haven't covered? Any tribal knowledge, historical context, or planned changes?"
- Options: "We covered everything", "There's more" (with Other for details)

If "There's more", continue with follow-up questions until the user is satisfied.

## Phase 4: Plan & Draft Documentation

### Read the scratch file
Re-read `.agents/.scratch/kb-interview-session.md` in full to have clean, consolidated context.

### Recommend doc types
Based on the feature's nature, recommend which `.agents/` documents to create. Present your recommendation using AskUserQuestion:

- "Based on our interview, I recommend creating these docs: [list]. Does this look right?"
- Options might include:
  - Operations doc (UI workflow / how to use it)
  - Reference doc (business rules, definitions, data model)
  - Codebase doc (technical patterns, integrations)
  - Updates to existing docs

### Draft each document

For each recommended doc, draft it following the templates in `.agents/AI-INSTRUCTIONS.md` (read in Phase 0).

Also draft updates to:
- The relevant `INDEX.md` file(s) — add keywords and links
- `NAVIGATION.md` — add the screen if it's a new operations doc

**Token limit reminder**: Each doc must stay under 500 tokens (200 for INDEX files). Use `AI-INSTRUCTIONS.md` limits as the guide.

### Present drafts for review

Present each draft with Approve/Edit/Skip options using AskUserQuestion. Do not include the full document content in chat. Iterate on edits until approved.

## Phase 5: Write & Finalize

### Write approved files
For each approved draft, write the file to the appropriate `.agents/` location.

### Update indexes
Write the approved INDEX.md and NAVIGATION.md updates.

### Validate
Run `yarn lint:docs` to check token limits. If any doc exceeds its limit, trim it and show the user the trimmed version.

### Clean up
Delete `.agents/.scratch/kb-interview-session.md`.

### Report

Summarize: files created, files updated, lint status. Offer to commit with message `docs(agents): add [feature-name] documentation`.

## Rules

- **Interview fresh** — Do NOT present code/browser findings to the user upfront. Use them only for cross-referencing.
- **Flag discrepancies** — If code behavior contradicts what the user says, surface it and ask for clarification.
- **User has final say** — Never write files without showing drafts first.
- **Respect token limits** — Operations/reference/codebase docs: 500 tokens. INDEX files: 200 tokens.
- **Adapt questions** — The example questions are starting points. Adapt based on the feature, the participants, and what you learn as you go.
- **Be thorough** — This is a knowledge capture session. It's better to ask one too many questions than to miss something important.
