---
name: pr-feedback
description: Fetch, prioritize, and address all review comments on a GitHub PR. Produces a step-by-step report for each comment addressed.
user_invocable: true
---

# Process GitHub PR Review Feedback

Accept a GitHub PR URL as the argument. If no URL is provided, ask the user for one.

## Step 1 — Fetch all review comments

Use `gh` CLI (must be authenticated) to fetch:

```
gh api repos/{owner}/{repo}/pulls/{number}/comments \
  --jq '.[] | "\(.id) | \(.user.login) | \(.path):\(.original_line // .line) | \(.html_url)\n\(.body)\n---"'
```

Also fetch top-level issue comments:

```
gh api repos/{owner}/{repo}/issues/{number}/comments \
  --jq '.[] | "\(.id) | \(.user.login) | \(.html_url)\n\(.body)\n---"'
```

And PR metadata:

```
gh pr view {number} --repo {owner}/{repo} --json title,state,body,reviewDecision
```

Parse the owner, repo, and PR number from the URL argument. The URL format is `https://github.com/{owner}/{repo}/pull/{number}`.

## Step 2 — Prioritize and present

Categorize each comment by importance:

1. **Critical** — Algorithm/kernel/correctness changes, performance issues, data-integrity bugs
2. **Important** — Benchmark configuration, test coverage, memory layout, API design
3. **Minor** — Naming, style, documentation, formatting

Present a numbered summary to the user showing for each comment:
- Priority tier
- Reviewer name
- File:line (if inline comment)
- The exact comment text
- The GitHub comment URL

Then ask the user to confirm proceeding, or let them reorder/skip comments.

## Step 3 — Process each comment

Work through comments in priority order. For each comment:

1. **Read the relevant file(s)** to understand the current code
2. **Make the code changes** to address the feedback
3. **Compile and verify**:
   - If the project has a build system, compile the changed code
   - If tests exist, run them
   - If this is an NPU kernel project (has bisheng compilation, torch-npu tests), follow the npu_kernel_general skill's mandatory requirements: compile with bisheng, execute on NPU, verify numerical correctness
4. **Write a step report** as `Step{N}_{short_title}.md` in a `claude_tasks_reports/` subdirectory relative to the changed files. The report must contain:

```markdown
# Step {N}: {Title}

## PR Comment Addressed

**Reviewer:** {reviewer}
**File:** `{file}:{line}` (if inline)
**Comment URL:** {github_comment_url}

> {exact comment text, blockquoted}

## Problem

{Brief description of what the reviewer identified}

## Changes

{What was changed and why}

## Files Changed

- `{file1}`: {description}
- `{file2}`: {description}

## Verification

- {Compilation result}
- {Test results}
- {Any numerical checks}
```

5. **Mark the task complete** before moving to the next comment

## Step 4 — Final verification

After all comments are addressed:

1. Run the full test suite to check for regressions
2. Summarize all changes made across all steps
3. List the report files created

## Important notes

- Always read files before modifying them
- Do not batch multiple comments into a single step — one step per comment
- If a comment requires investigation (e.g., "can this be done in FP16?"), investigate first, then implement if feasible
- If two comments conflict, flag this to the user before proceeding
- Use task tracking (TaskCreate/TaskUpdate) to show progress
