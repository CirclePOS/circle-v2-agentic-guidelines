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

## Workflow

### 1. Write Specification Tests (Red Phase)

**For Backend (RSpec):**
```bash
# Create spec file in spec/{entity}/
touch spec/commands/your_command_spec.rb

# Write failing tests first:
# - Happy path assertions
# - Edge cases (nil, empty, invalid)
# - Error scenarios
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

### 2. Write Implementation (Green Phase)

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

### 3. Verify Against Full Suite (Refactor Phase)

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

### 4. Create PR & Request Review

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
- `/regression-tester` — Run full test suite classification
- `/unit-tester` — Quick unit test validation
- `/pr-helper` — Create PR after tests pass
