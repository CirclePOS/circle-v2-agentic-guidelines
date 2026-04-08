---
name: rspec-runner
description: Run RSpec test suite with intelligent filtering — specific file, tag, or full suite with pre/post-existing failure classification
user-invocable: true
---

# RSpec Runner

Smart test execution for Circle v2 backend. Classifies failures as "ours" vs "pre-existing" to avoid false negatives during branch work.

## Setup

Requires:
```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle install
```

## Usage

### 1. Run Specific Test File

```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec spec/commands/your_command_spec.rb
```

**Output:**
```
xx passed, yy failed (shows which are new failures)
```

### 2. Run by Tag (Focused Testing)

```bash
# Tag your test
# it "should do X", :focus do

BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --tag focus
```

### 3. Run Full Suite & Classify Failures

```bash
# Step 1: Save baseline (run on clean master4)
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --format json > /tmp/master4_tests.json

# Step 2: Run your branch
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --format json > /tmp/branch_tests.json

# Step 3: Compare and classify
# (Use regression-tester skill for automated classification)
```

## When to Use Each

| Scenario | Command |
|----------|---------|
| **Writing new tests** | `rspec spec/your_spec.rb` (watch mode) |
| **Debugging single failure** | `rspec spec/file_spec.rb:123 --format doc` |
| **Test one command** | `rspec spec/commands/command_name_spec.rb` |
| **Test one model** | `rspec spec/entities/entity_name_spec.rb` |
| **Before committing** | `rspec` (full suite) or `/regression-tester` (with classification) |
| **Pre-alpha4 push** | `/regression-tester` (identifies new vs pre-existing failures) |

## Output Interpretation

**All Green:**
```
XX examples, 0 failures
→ Safe to commit and push to alpha4
```

**Some Red (New Failures):**
```
XX examples, Y failures (Y new)
→ Fix these before pushing to alpha4
```

**Some Red (Pre-existing):**
```
XX examples, Y failures (0 new, Y pre-existing)
→ Safe to push (failures exist on master4 too)
```

## Integration with Workflow

1. **While developing**: Run focused tests frequently
   ```bash
   BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec spec/your_spec.rb
   ```

2. **Before commit**: Run full suite or use `/regression-tester`
   ```bash
   BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec
   # OR
   /regression-tester
   ```

3. **Before alpha4 push**: Confirm no new failures
   - Use `/regression-tester` for automated classification
   - Or manually compare baseline vs branch

## Debugging Tips

**Stop at first failure:**
```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --fail-fast
```

**Verbose output:**
```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --format doc --color
```

**Run with seed (reproducible randomization):**
```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --seed 1234
```

**Profile slowest tests:**
```bash
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rspec --profile
```

## See Also

- `/regression-tester` — Full suite + failure classification (use before alpha4 push)
- `/unit-tester` — Quick validation of changed files
- `/tdd-workflow` — Red-Green-Refactor workflow
