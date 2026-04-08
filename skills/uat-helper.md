---
name: uat-helper
description: Merge feature branch into alpha4, push, and notify QA — only after all tests pass
---

Push the current feature branch to alpha4 for QA testing.

**Pre-condition:** All sub-agents must have passed before running this:
- `/code-standards-check` ✅
- `/unit-tester` ✅
- `/regression-tester` ✅
- `/ux-tester` ✅ (if UI changed)

If any are unresolved, stop and fix before proceeding.

## Steps

### 1. Final rebase
```bash
git fetch origin && git rebase origin/master4
```

### 2. Confirm current branch
```bash
git branch --show-current
```
Confirm you are on the feature branch (not alpha4, not master4).

### 3. Merge to alpha4
```bash
git checkout alpha4
git merge <feature-branch> --no-ff -m "Merge <feature-branch> into alpha4"
git push origin alpha4
```

### 4. Return to feature branch
```bash
git checkout <feature-branch>
```

### 5. Notify QA
Post a comment on the GitHub issue:
```
🚀 Pushed to alpha4 for QA.
Branch: `<feature-branch>`
Ready for testing on alpha.
PR: <link — create with /pr-helper if not yet open>
```

## Output

> ✅ Merged to alpha4 and pushed. QA notified.
> Next step: run /pr-helper if PR not yet created.
