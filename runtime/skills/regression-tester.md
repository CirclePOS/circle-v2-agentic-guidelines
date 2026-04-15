---
name: regression-tester
description: (CI ONLY) Classify full RSpec suite failures after push to alpha4
---

**⚠️ DO NOT RUN LOCALLY.** This skill runs automatically on CI after code is pushed to alpha4.

The full RSpec suite takes significant time to run. To improve delivery velocity:
- **Local development**: Use `/unit-tester` to validate changed files only
- **CI pipeline**: GitHub Actions runs the full suite automatically on alpha4 push
- **QA testing**: Full test classification happens in the CI logs, not on developer machines

Load testing reference:
@.claude-guidelines/standards/testing-standards.md

## When This Runs

**On CI (automatically):**
- After code is merged into alpha4
- GitHub Actions runs the full RSpec suite
- Any failures are classified as ours vs pre-existing
- Results appear in CI logs and GitHub PR status

## If CI Detects New Failures

If CI shows **new regressions** (caused by this branch):

1. Rebase from master4
2. Reproduce locally with targeted tests:
   ```bash
   bundle exec rspec spec/path/to/failing_spec.rb
   ```
3. Fix the code
4. Re-run targeted tests locally to confirm fix
5. Push to alpha4 again — CI will re-run full suite

## Output

**CI logs show:**
- ✅ All tests passing — safe for QA
- ⚠️ Pre-existing failures only — known issues on master4, safe for QA
- ❌ New failures introduced — must fix before QA can accept
