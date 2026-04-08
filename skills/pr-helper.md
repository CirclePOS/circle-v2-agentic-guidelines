---
name: pr-helper
description: Create the GitHub PR targeting master4 immediately after alpha4 push
---

Create the GitHub PR for the current feature branch.

Load PR conventions:
@.claude-guidelines/workflow/pr-conventions.md

## Pre-condition

Must be run immediately after `/uat-helper`. The PR must exist while code is on alpha4 so QA can link it to the story and peer review can begin in parallel.

## Steps

### 1. Get branch and ticket info
```bash
git branch --show-current
# Format: <developer-name>-<ticket-id>-<short-description>
```

Extract:
- `ticket-id` from branch name (this is the GitHub issue number)
- GitHub issue number (from the issue you fetched in Step 1)

### 2. Create PR
```bash
gh pr create \
  --base master4 \
  --title "#<GITHUB_ISSUE_NUMBER> - <Short description>" \
  --body "$(cat <<'EOF'
## Why
<!-- Describe the problem this solves -->

Closes #<GITHUB_ISSUE_NUMBER>

## What Changed
<!-- Summary of changes -->

## Testing
- [ ] `/code-standards-check` passed
- [ ] `/unit-tester` passed
- [ ] `/regression-tester` passed
- [ ] `/ux-tester` passed (if UI changed)
- [ ] Pushed to alpha4 for QA
EOF
)"
```

### 3. Post PR link on GitHub issue
```bash
gh issue comment <issue-number> --body "PR created: <pr-url>"
```

## Output

> ✅ PR created: <url>
> Target: master4
> QA and peer review can now begin in parallel.
>
> ⏳ Waiting for merge gate:
> - CI green (8 RSpec groups + 2 Cypress shards)
> - QA accepted on GitHub
> - 2 peer code reviews approved
>
> Do NOT merge your own PR.
