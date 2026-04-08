---
name: unit-tester
description: Run unit tests for changed files and write missing tests before proceeding to regression testing
---

Run unit tests for all files changed in this branch.

Load testing reference:
@.claude-guidelines/standards/testing-standards.md

## Steps

### 1. Identify changed files
```bash
git diff origin/master4...HEAD --name-only
```

### 2. Run targeted RSpec for changed files
```bash
bundle exec rspec spec/path/to/changed_spec.rb
```

Run each relevant spec file. For command changes, run the matching `spec/commands/` file.

### 3. Check for missing tests
- Every new method or behaviour change must have a corresponding spec
- New tests go in `spec/` (RSpec) — never `test/` (Minitest)
- Use shared contexts from `spec/support/shared_context/` where applicable
- Use `build_dummy_command_klass` for isolated command step testing

### 4. Write missing tests if needed
Write the minimum tests that validate the specific change made. Do not write broad test suites — focus on the changed behaviour.

## Output

If all tests pass:
> ✅ Unit tests passed. Proceed to /regression-tester.

If tests fail:
> ❌ Unit test failures:
> - Test: `<test description>`
> - Failure: `<error message>`
> - Fix: `<what to change in code or test>`

Fix failures, re-run `/unit-tester`. Do not proceed to `/regression-tester` until clean.
