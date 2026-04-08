---
name: regression-tester
description: Run full RSpec suite and classify failures as ours vs pre-existing before alpha4 push
---

Run the full RSpec suite and classify any failures before allowing an alpha4 push.

Load testing reference:
@.claude-guidelines/standards/testing-standards.md

## Steps

### 1. Rebase check
Ensure branch is current before running:
```bash
git fetch origin && git rebase origin/master4
```

### 2. Run full RSpec suite
```bash
bundle exec rspec
```

### 3. Classify failures

For each failure, determine:

**"Pre-existing" (broken on master4 before our branch):**
```bash
git stash
bundle exec rspec <failing_spec>
git stash pop
```
If it fails on master4 too → pre-existing.

**"Ours" (introduced by this branch):**
If it passes on master4 but fails on our branch → we broke it.

### 4. Decision matrix

| Failure type | Action |
|---|---|
| No failures | ✅ Proceed to Step 5 (alpha4 push) |
| Pre-existing only | Confirm they are already fixed on master4, then proceed |
| Pre-existing + not fixed on master4 | Block push — flag to team, do not push |
| Ours (new regression) | Fix code, re-run `/unit-tester` then `/regression-tester` |

## Output

If clean:
> ✅ Regression tests passed. Safe to proceed to /uat-helper.

If pre-existing failures only:
> ⚠️ Pre-existing failures detected (broken on master4, not caused by this branch):
> - `<list of specs>`
> These are pre-existing. Confirm they are fixed on master4 before proceeding.

If new regressions:
> ❌ New regressions detected (caused by this branch):
> - Test: `<test description>`
> - Failure: `<error>`
> Fix these before proceeding. Do NOT push to alpha4 with new regressions.
