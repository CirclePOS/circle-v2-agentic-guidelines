---
name: tdd-workflow
description: Test-driven development workflow for Circle v2 — write RSpec/Cypress tests first, then implementation, then verification
user-invocable: true
---

# Test-Driven Development Workflow

TDD enforces your CLAUDE.md "Plan Mode before code" rule by making tests the specification. This workflow pairs with your PR review requirements.

## Prerequisites

- Branch created and checked out
- Plan posted as GitHub issue comment (per CLAUDE.md)
- `.claude-guidelines/standards/testing-standards.md` loaded for this context
- `IP-Pass` label applied to the Story (see `issue-workflow.md` Step 1d). RSpec *drafting* (Red phase) is permitted before `IP-Pass` to accelerate planning, but **no implementation code** until the label is applied.

## Workflow

### 1. Load Approved RSpec Scenarios

Before writing any test code, confirm the approved scenarios from the handoff template.
If no scenarios were approved (e.g., ad-hoc task without a handoff), plan them now following the business logic review in `issue-workflow.md` Step 1b.

**Pre-writing checklist:**
- [ ] Approved scenarios are listed in the handoff template or chat history
- [ ] Each scenario tests correct system behavior, not tolerance of bad data
- [ ] No scenario validates a nil/blank/invalid state that the system should prevent (see testing-standards.md: "Fix the Source, Not the Symptom")

### 2. Write Specification Tests (Red Phase)

**For Backend (RSpec):**
```bash
# Create spec file in spec/{entity}/
touch spec/commands/your_command_spec.rb

# Write failing tests first — one example per approved scenario:
# - Context matches the scenario's "Context"
# - Expectation matches the scenario's "Expected behavior"
```

**For Frontend (Cypress):**
```bash
# Create spec in spec-cypress/
touch spec-cypress/e2e/feature_name.cy.ts

# Write failing E2E tests:
# - User interactions
# - Form validation
# - State changes
```

**Checklist:**
- [ ] Tests fail (red bar visible)
- [ ] Tests specify actual behavior, not implementation
- [ ] Both happy path AND edge cases covered
- [ ] Every approved scenario has a corresponding test example
- [ ] No spec asserts that bad data is acceptable when the fix should prevent it
- [ ] If `IP-Pass` is not yet applied: stop at the Red phase. Do not proceed to Green.

### 3. Write Implementation (Green Phase)

Only after tests are written and failing:

**Backend:**
```bash
# Implement in app/commands/ or app/services/
# Run test suite frequently:
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec spec/your_spec.rb
```

**Frontend:**
```bash
cd app/vue3
npm run dev  # Watch mode
npm run test:unit  # If unit tests exist
```

**Checklist:**
- [ ] Tests pass (green bar)
- [ ] No debugging code left behind
- [ ] Follows code standards (validate via code-standards-check skill)

### 4. Verify Against Full Suite (Refactor Phase)

Run regression tests before committing:

```bash
# Backend: full suite
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec

# Frontend: Cypress E2E
cd app/vue3 && npm run cypress:run

# Code quality checks
npm run eslint
rubocop
```

**Checklist:**
- [ ] All tests pass (no pre-existing failures introduced)
- [ ] Rubocop/ESLint pass
- [ ] No Pronto warnings
- [ ] Code standards check passes (use code-standards-check skill)

### 5. Create PR & Request Review

Link your tests to the specification:

```markdown
## Tests
- RSpec: spec/commands/your_command_spec.rb (XX assertions)
- Cypress: spec-cypress/e2e/feature_name.cy.ts (XX scenarios)

## Test Coverage
- Happy path: ✓
- Edge cases (nil, empty): ✓
- Error handling: ✓
```

Then run: `/code-standards-check` before requesting peer review.

## When TDD Isn't Needed

Skip TDD for:
- Documentation-only changes
- Dependency updates with no code changes
- Config-only changes (no behavior change)
- Cherry-picks or reverts (test already exists)

**For everything else**: Red → Green → Refactor → Review

## Commands to Use

- `/code-standards-check` — Validate standards before review
- `/unit-tester` — Unit test validation on changed files
- `/ux-tester` — UX/accessibility validation (only if UI files changed)
- `/uat-helper` — Merge to alpha4 and push (only after all local tests pass)
- `/pr-helper` — Create PR immediately after alpha4 push

**Note:** `/regression-tester` runs on CI only — do not run it locally. See `issue-workflow.md` Step 5.
