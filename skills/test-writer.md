# Test Writer

Write or update RSpec tests for changed code, following Circle's test conventions.

## Steps

1. **Identify files needing tests** — run `git diff --name-only origin/master4...HEAD`. Focus on:
   - New files with no corresponding spec
   - Changed files where the spec needs updating

2. **Read the implementation** before writing any test. Understand what the code actually does.

3. **Find an existing spec to mirror** — always follow an existing pattern:
   - New command step → read `spec/commands/api/<any_domain>/steps/` for a reference
   - New command → read `spec/commands/api/ledger/` as the reference pattern
   - New service → read nearest existing service spec
   - New model validation → read nearest model spec

4. **Write tests following these conventions**:
   - Use RSpec + FactoryBot — no fixtures
   - Factories live in `spec/factories/` — use existing factories, create new ones only if needed
   - Use shared contexts where applicable: `include_context 'core_commands'`, `include_context 'freezing_time'`
   - Use `build_dummy_command_klass` from `spec/support/commands_steps_helper.rb` for testing individual steps
   - Auth helpers: `login_as`, `sign_as_admin`, `post_as_admin` from `spec/support/auth_helper.rb`
   - Tag type metadata: `type: :command_step`, `type: :request`, `type: :controller`
   - Use `:focus` tag to run a single example in isolation during development (remove before committing)
   - VCR cassettes for specs that call external HTTP — never make real external calls in tests

5. **Structure per spec type**:
   - Command specs: test context setup, step execution, success/failure outcomes
   - Request specs: test HTTP status, response shape, auth requirements
   - Model specs: validations, scopes, associations
   - Service specs: input/output behavior, edge cases

6. **Run the new specs** to confirm they pass:
   ```bash
   bundle exec rspec <new_spec_file>
   ```

7. **Report** which spec files were created/updated and the pass count.

## Notes

- Write minimum tests that give confidence — don't write exhaustive permutation tests
- Test behavior, not implementation — if the internals change, tests should still pass
- DatabaseCleaner uses `:deletion` strategy (not transactions) — tests are slower but accurate
