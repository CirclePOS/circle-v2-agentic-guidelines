# Testing Standards

## RSpec (Primary Test Framework)

### Fix the Source, Not the Symptom

Every spec must validate **correct system behavior**, not tolerance of bad data.

**The rule:** If a value should never be nil/blank/invalid in production, the test must prove the system **prevents** that state — not that the system **survives** it.

| Symptom-masking spec (WRONG) | Root-cause spec (RIGHT) |
|------------------------------|------------------------|
| `context "when country is nil"` / `it { is_expected.to be_nil }` | `context "when country is blank"` / `it "pre-fills from the website default"` |
| `expect { subject }.not_to raise_error` (with invalid data) | `expect(record.country).to eq(website.country)` (data was corrected) |

**When to add a nil-guard spec:** Only when nil is a *legitimate* business state (e.g., optional fields, soft-deleted records). If the field is required for the feature to function, the spec must prove the system fills or rejects the value — never that it silently accepts nil.

**RSpec Scenario Planning:** Test scenarios are planned and approved in Step 1b of `issue-workflow.md` *before* any code is written. The handoff template includes the approved scenario list. Do not invent new scenarios during implementation without discussing first.

```bash
bundle exec rspec                                           # full suite
bundle exec rspec spec/models/site_feature_spec.rb         # single file
bundle exec rspec spec/models/site_feature_spec.rb:10:15   # specific lines
bundle exec rspec --only-failures                           # re-run last failures only
```

### Setup Notes
- Uses `DatabaseCleaner` with `:deletion` strategy (not transactions)
- `sales_reports` and `dropped_sales_reports` tables excluded from deletion
- FactoryBot has a custom `:cache` strategy for de-duplicating associated records in tests
- Elasticsearch must be running for tests that touch ES-indexed models
- `.rspec` enables `--color`, `documentation` format, and `filter_run_when_matching :focus`
- Tag an example with `:focus` to run it in isolation

### Helpers
- `spec/support/auth_helper.rb` — `login_as`, `sign_as_admin`, `sign_in_as_customer`, `post_as_admin` for controller/request specs
- `spec/support/commands_steps_helper.rb` — `build_dummy_command_klass` for testing individual command steps in isolation

### Shared Contexts (`spec/support/shared_context/`)
- `core_commands` — command setup
- `freezing_time`
- `change_queue_adapter` — background jobs

### Type Metadata
RSpec type metadata (`:command_step`, `:request`, `:controller`) auto-includes appropriate support modules.

### Model Spec Structure

Unit tests for model methods follow a consistent structure:

- `subject` calls the method under test (not just instantiates the object)
- `subject` is the first line inside the `describe` block
- `context` describes the scenario in plain English ("when X is Y")
- `let` variable names match exactly what is named in the `context` clause

```ruby
describe "#country_representation" do
  subject { address.country_representation }

  context "when country is a valid ISO 3166 alpha2 code" do
    let(:address) { build(:address, country: "NZ") }
    it { is_expected.to eq "New Zealand" }
  end

  context "when country is blank" do
    let(:address) { build(:address, country: nil) }
    it "falls back to the website default country" do
      is_expected.to eq address.site.country
    end
  end

  context "when country is a legacy free-text value" do
    let(:address) { build(:address, country: "New Zealand") }
    it { is_expected.to eq "New Zealand" }
  end
end
```

### Controller / Request Spec Expectations

A bare HTTP status check is a survival check, not a test. Every controller or request spec example must include at least one expectation about rendered content or observable behavior in addition to the HTTP status:

```ruby
# Not enough on its own:
expect(response).to have_http_status(:ok)

# Required — assert something meaningful about what was rendered:
expect(response.body).to include("Country:")
```

This makes the intent of the test clear and prevents regressions in behavior even when the response code stays green.

### VCR
- `spec/cassettes/` records and replays external HTTP calls
- Use VCR cassettes in specs that hit external APIs

---

## Minitest (Deprecated)

Fix broken Minitest tests when encountered; write all new tests in RSpec.

```bash
bundle exec rails test --trace
bundle exec rails test test/unit/site_domain_test.rb
bundle exec rails test/unit/agency_test.rb:10
```

---

## Cypress E2E (Vue 3 Legacy)

- Tests in `app/vue3/apps/*/cypress/`
- CI runs two shards: `cypress1`, `cypress2`
- Both shards must pass before QA
- **Strategy**: For new features, prefer RSpec system tests (see below)

---

## RSpec System Tests (Hotwire)

System tests for Hotwire views, Turbo interactions, and Stimulus controllers.

```bash
bundle exec rspec spec/system/                            # all system tests
bundle exec rspec spec/system/hotwire/checkout_spec.rb   # single system test
bundle exec rspec spec/system/hotwire/checkout_spec.rb:5:20  # specific lines
```

### Hotwire System Test Structure

```ruby
# spec/system/hotwire/external_services_spec.rb
describe "External Services Settings" do
  before { login_as(admin) }

  # Simple page load (no JavaScript needed)
  it "displays payment integrations" do
    visit settings_path(anchor: "external_services")
    expect(page).to have_text("Payment Integrations")
  end

  # Turbo Frame interaction (update without full page reload)
  it "updates integrations list via Turbo Frame" do
    visit settings_path
    
    # Trigger Turbo Frame update (pagination link click, etc.)
    click_link "Page 2", inside: turbo_frame("integrations")
    
    expect(page).to have_current_path(settings_path)  # URL unchanged
    expect(page).to have_selector("turbo-frame#integrations")
  end

  # Stimulus controller interaction (requires JavaScript)
  it "toggles payment method via Stimulus", :js do
    visit settings_path
    
    # Stimulus action trigger
    find('[data-action="click->toggle#switch"]').click
    
    # Verify behavior (DOM changes, class additions, etc.)
    expect(page).to have_selector(".active")
  end

  # Turbo Stream response (real-time updates)
  it "creates integration and broadcasts update", :js do
    visit settings_path
    
    fill_in "service_name", with: "Stripe"
    click_button "Add"
    
    expect(page).to have_text("Stripe")
  end
end
```

### Key Differences from Cypress

| Aspect | RSpec System | Cypress E2E |
|--------|------------|----------|
| **Framework** | RSpec (Rails native) | Cypress (Node.js) |
| **Process** | Runs in same process as Rails app | Launches browser independently |
| **Speed** | Faster (no browser startup) | Slower (browser overhead) |
| **JavaScript** | Requires `js: true` metadata | Always has JS enabled |
| **Database** | Shares app database | Separate browser session |
| **Best for** | Hotwire, server-driven UI | Vue 3, complex browser interactions |

### Hotwire System Test Patterns

**Test Turbo Frames:**
```ruby
# Use turbo_frame() matcher
within(turbo_frame("results")) do
  expect(page).to have_text("Page 2")
end
```

**Test Stimulus Controllers:**
```ruby
# Use data attributes to target elements
find('[data-backoffice--toast-target="container"]')

# Use :js metadata for JavaScript execution
it "shows toast", :js do
  execute_script("window.dispatchEvent(new CustomEvent('toast:notification', {...}))")
  expect(page).to have_text("Notification")
end
```

**Test Forms in Turbo Frames:**
```ruby
# Form in frame, Turbo Stream response on update
it "validates and updates form" do
  within(turbo_frame("edit_service")) do
    fill_in "service_name", with: ""
    click_button "Save"
  end
  
  # Frame re-renders with errors (Turbo Stream response)
  expect(page).to have_text("can't be blank")
end
```

### Hotwire System Test Metadata

```ruby
# Standard system test
describe "Turbo pagination", type: :system do
end

# System test requiring JavaScript (Stimulus, etc.)
describe "Stimulus interactions", type: :system, js: true do
end

# Rack test (faster, no JavaScript)
describe "Rack-based interactions", type: :system do
end
```

### File Placement for Hotwire System Tests

```
spec/
├── system/
│   ├── hotwire/
│   │   ├── external_services_spec.rb
│   │   ├── checkout_spec.rb
│   │   └── [feature]_spec.rb
│   └── vue3/                  # Legacy Vue 3 system tests (if any)
```

---

## CI Requirements

All of the following must pass before a PR can merge:
- 8 parallel RSpec groups (all green)
- 2 Cypress shards (both green)

---

## Test File Placement

| Test type | Location | Framework |
|-----------|----------|-----------|
| Unit / model | `spec/models/` | RSpec |
| Request / controller | `spec/requests/` or `spec/controllers/` | RSpec |
| Command specs | `spec/commands/` mirroring `app/commands/` | RSpec |
| **Hotwire system tests** (primary) | `spec/system/hotwire/` | RSpec + Capybara |
| Vue 3 E2E (legacy) | `app/vue3/apps/*/cypress/` | Cypress |
| Legacy tests (fix only) | `test/` | Minitest |

**Guidance:**
- ✅ **New features → Hotwire system tests** (`spec/system/hotwire/`)
- ✅ **Bug fixes in Vue 3 → Cypress** (existing tests)
- ❌ **Never write new tests in `test/`** — always use `spec/`
- ⚠️ **Vue 3 E2E:** Phase out gradually as features move to Hotwire

---

## Test Priority (by Feature Type)

| Feature Type | Primary Test | Secondary | Notes |
|--------------|-------------|-----------|-------|
| Hotwire view | System test (RSpec) | — | Use `/rspec-runner` |
| Stimulus controller | System test with `:js` | Unit test | Test behavior, not implementation |
| Turbo Frame update | System test | — | Verify frame updates without full reload |
| Vue 3 component | Cypress E2E | Unit test | Legacy — no new features |
| API endpoint | Request spec | — | Both Hotwire + Vue 3 consume these |
| Rails model | Model spec | Integration via system test | Test logic, verify in system tests |

---

## Example Test Pyramid for Hotwire Feature

```
           📊 System Tests (10 tests)
              ↑ User workflows
           🧪 Request/Controller Tests (5 tests)
              ↑ API contracts
           ✅ Unit Tests (20 tests)
              ↑ Business logic, models, commands
```

**Hotwire tests favor the pyramid top**: System tests (user behavior) are primary, Request specs validate APIs, Unit tests validate logic.
