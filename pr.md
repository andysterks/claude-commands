# PR - Commit, Push, and Create Pull Request

Create a versioned commit, push to remote, and open a pull request for the current feature branch.

## Arguments

Optional: `$ARGUMENTS` can contain a version bump type (`patch`, `minor`, `major`, `skip`) and/or a PR description override.

---

## Phase 1: Pre-flight Check

1. Run `git status` — confirm there are changes to commit
2. Run `git branch --show-current` — confirm we are NOT on `main` or `master`
   - If on main/master: **STOP** and tell the user to switch to a feature branch first
3. Run `git log --oneline -5` to understand recent commit style

## Phase 2: Determine Version Bump Type

Check if `$ARGUMENTS` contains `patch`, `minor`, `major`, or `skip`.

- **If found in arguments**: use it directly, skip to Phase 3
- **If NOT found**: use AskUserQuestion:

```
Question: What type of change is this?
Options:
  - patch — bug fix, small tweak, no new functionality
  - minor — new feature, backward compatible
  - major — breaking change
  - skip — no version bump (docs, chores, infra only)
```

## Phase 3: Stage and Commit

1. Show the user a summary of what will be staged (`git diff --stat`)
2. Stage relevant files — prefer specific file names over `git add .` unless scope is clearly everything
3. Write a commit message that:
   - Summarizes the **why**, not just the what
   - Starts with the bump type in brackets: `[patch]`, `[minor]`, `[major]`, or `[skip]`
   - Example: `[minor] Add multi-meal support per mealtime`
4. Commit using a HEREDOC:
   ```bash
   git commit -m "$(cat <<'EOF'
   [bump-type] Your commit message here
   EOF
   )"
   ```
5. If a pre-commit hook fails: fix the underlying issue, re-stage, and create a **new** commit — never use `--no-verify`

## Phase 4: Push

1. Check if remote tracking branch exists: `git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null`
   - If yes: `git push`
   - If no: `git push -u origin HEAD`

## Phase 5: Create Pull Request

1. Determine the target base branch (default: `main`; check if `develop` exists)
2. Get a summary of all commits since branching: `git log main..HEAD --oneline`
3. Get the latest commit message to use as the PR title:
   ```bash
   git log -1 --format="%s"
   ```
   Use this **exactly** as the PR title (including the `[patch]`/`[minor]`/etc. prefix).
4. Create the PR:

```bash
gh pr create --title "$(git log -1 --format='%s')" --body "$(cat <<'EOF'
## Summary
- [bullet points from commits]

## Test plan
- [ ] [manual or automated steps]

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

5. Output the PR URL to the user

## Phase 6: Optional Jira Link

If a Jira ticket ID is detectable (from branch name like `feature/NZRO-13` or from `$ARGUMENTS`):
- Ask: "Should I add the PR link to the Jira card?"
- If yes: use the Jira REST API with credentials from `.env` to add a comment with the PR URL

---

## Output

```
## PR Created ✅

**Branch**: feature/your-branch → main
**Commit**: [bump-type] Your commit message
**PR**: https://github.com/org/repo/pull/123

Next: When the PR is merged, run /close-ticket [TICKET-ID]
```
