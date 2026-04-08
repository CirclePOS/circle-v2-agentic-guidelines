---
name: Agent Team — Circle V2
description: Specialized agents for backend, frontend, and testing. Each handles their domain with clear hand-off protocols.
---

# Circle V2 Agent Team

An orchestrated team of specialized agents, each responsible for a specific domain of Circle V2 development. Agents coordinate via clear hand-off protocols and shared context loading.

## 🏗️ Team Structure

```
Agent Team: Circle V2 Development
│
├── 🔧 backend-agent
│   ├── Focus: Rails models, commands, services, API design
│   ├── Skills: code-standards-check, tdd-workflow, rspec-runner, pr-helper
│   ├── Constraints: No Vue files, escalate schema changes
│   └── Context: CLAUDE.md, code-standards-backend, testing-standards
│
├── 🎨 frontend-agent
│   ├── Focus: Vue 3 components, forms, state, E2E tests
│   ├── Skills: code-standards-check, tdd-workflow, ux-tester, test-writer
│   ├── Constraints: No Rails files, always use Vee-Validate
│   └── Context: CLAUDE.md, code-standards-frontend, testing-standards
│
└── 🧪 tester-agent
    ├── Focus: RSpec/Cypress strategy, failure triage, coverage gaps
    ├── Skills: rspec-runner, regression-tester, unit-tester, smoke-tester
    ├── Constraints: Analysis only, never implements code
    └── Context: CLAUDE.md, testing-standards (both backend + frontend)
```

## 🤝 Hand-Off Protocols

### When Backend Agent Escalates

**→ To Frontend Agent:**
```
"This needs Vue components. Handing off to frontend-agent."
Trigger: User requests form UI, component implementation, or E2E test changes
```

**→ To Tester Agent:**
```
"Test strategy needed for this complex scenario. Handing off to tester-agent."
Trigger: Need to create failing test first, or test coverage analysis
```

### When Frontend Agent Escalates

**→ To Backend Agent:**
```
"API contract needs review. Handing off to backend-agent."
Trigger: REST endpoint design, data serialization, auth logic
```

**→ To Tester Agent:**
```
"E2E test infrastructure needed. Handing off to tester-agent."
Trigger: Create test suite structure, fix flaky tests, analyze coverage
```

### When Tester Agent Escalates

**→ To Backend/Frontend Agent:**
```
"New test failure detected in [component/command]. Escalating for fix."
Trigger: Test-detected regression, blocking alpha4 push
```

**→ To Backend Agent (Urgent):**
```
"New RSpec failure in [file] is blocking alpha4 push. Fix required."
Trigger: Pre-alpha4 regression, must be fixed before QA deployment
```

## 🎯 Typical Workflows

### Workflow 1: Backend Feature with E2E Validation

```
1. Backend Agent
   ├── `/tdd-workflow` → Write RSpec tests first
   ├── Implement Rails command/service
   ├── `/rspec-runner` → Run backend tests
   └── Post: "API endpoint /api/v1/checkout ready for UI integration"

2. Frontend Agent
   ├── Creates Cypress E2E spec (failing)
   ├── Implements Vue component
   ├── Tests against backend-agent's API
   └── Hands off: "E2E tests passing, ready for QA"

3. Tester Agent
   ├── `/regression-tester` → Full suite classification
   └── Reports: "All tests green, no pre-existing failures introduced"

4. Result: PR merged to alpha4 after QA acceptance
```

### Workflow 2: Bug Fix Detected by E2E

```
1. Tester Agent
   ├── Cypress E2E failure discovered
   ├── Reproduces locally with /smoke-tester
   └── Creates RSpec test that fails (root cause)
   └── Escalates: "Checkout crash when country is nil. Created failing spec in spec/commands/checkout_spec.rb"

2. Backend Agent
   ├── Receives failing test from tester-agent
   ├── Implements fix to pass test
   ├── `/rspec-runner` → Verifies fix
   └── Hands off: "nil-country bug fixed in checkout command"

3. Frontend Agent
   ├── Verifies E2E test passes
   └── Confirms UI behavior fixed

4. Tester Agent
   ├── `/regression-tester` → Confirms no new failures
   └── Closes issue: "Bug fixed, no regressions introduced"
```

### Workflow 3: Form Validation Changes

```
1. Frontend Agent
   ├── Implements Vue form with Vee-Validate
   ├── Writes Cypress E2E for happy path + errors
   └── Escalates: "Form validation rules need backend enforcement"

2. Backend Agent
   ├── Reviews frontend form validation schema
   ├── Implements command with matching validation
   ├── `/rspec-runner` → Tests validation edge cases
   └── Hands off: "API validation matches frontend rules"

3. Frontend Agent
   ├── Cross-verifies E2E tests still pass
   └── Ready for QA

4. Tester Agent
   ├── `/regression-tester` → Full suite check
   └── Reports green status
```

## 📋 Shared Skills

All agents have access to these shared skills:

| Skill | Purpose | Invoked By |
|-------|---------|-----------|
| `/code-standards-check` | Validate code before PR | All agents (before review) |
| `/tdd-workflow` | Test-driven development | Backend & Frontend agents |
| `/rspec-runner` | Backend test execution | Backend & Tester agents |
| `/pr-helper` | Create GitHub PR | Backend or Frontend (after QA) |
| `/regression-tester` | Full suite + classification | Tester agent (pre-alpha4) |

## 🔗 How to Use Agent Team

### Invoke a Specific Agent

```bash
# Start backend agent for feature work
claude-code --agent backend-agent -p "Implement checkout command with validation"

# Start frontend agent for UI changes
claude-code --agent frontend-agent -p "Create payment form with validation"

# Start tester agent for test strategy
claude-code --agent tester-agent -p "Analyze test coverage gaps in checkout flow"
```

### Let Agent Decide Hand-Off

Agents are instructed to detect when work should transition to another agent:

```
User: "Add a new field to the user profile form"

Backend Agent: "This is primarily frontend work. Handing off to frontend-agent."

Frontend Agent: "Creating form with field, and Cypress test. Need backend validation?"

Backend Agent: (Auto-invoked) "Yes, adding command-level validation. Back to frontend for integration."

Tester Agent: (Auto-invoked for review) "All tests green, no coverage gaps."
```

## 🚨 Critical Hand-Offs

**Never ignore these escalations:**

1. **Test Failure Blocking Alpha4**
   ```
   Tester Agent → Backend/Frontend Agent
   "New RSpec failure in [X]. Fix required before QA push."
   → Immediate escalation, blocks merge
   ```

2. **Schema Change Required**
   ```
   Backend Agent → DB Engineer Agent (Phase 2)
   "Migration needed for new orders table. Escalating for safety review."
   → Must review migration strategy before implementation
   ```

3. **Accessibility Issue Found**
   ```
   Tester Agent → UX Tester Agent (Phase 2)
   "WCAG 2.1 AA violation in checkout form. Escalating for fix."
   → Must fix before QA acceptance
   ```

## 🧠 Agent Memory

Each agent maintains persistent memory:

- **Backend Agent**: Architecture decisions, API contracts, performance bottlenecks
- **Frontend Agent**: Component patterns, form validation rules, UX flows
- **Tester Agent**: Flaky test patterns, coverage gaps, pre-existing failures

Memory is loaded at session start and updated after each session. Agents reference memory when making architectural or strategic decisions.

## 📊 Agent Team Readiness

| Agent | Status | Ready | Dependencies |
|-------|--------|-------|--------------|
| backend-agent | ✅ Created | ✅ Yes | CLAUDE.md, testing-standards |
| frontend-agent | ✅ Created | ✅ Yes | CLAUDE.md, testing-standards |
| tester-agent | ✅ Created | ✅ Yes | CLAUDE.md, testing-standards |
| db-engineer-agent | ⏳ Phase 2 | ⏹️ No | Migration runbooks, schema patterns |
| ux-tester-agent | ⏳ Phase 2 | ⏹️ No | WCAG 2.1 AA standards, accessibility patterns |

## 🎯 Next Steps

1. **Test the agents** — Invoke them individually with sample tasks
2. **Document hand-off URLs** — Add to team wiki
3. **Create agent orchestrator** (Phase 2) — Automated routing for complex features
4. **Build agent memory system** (Phase 2) — Persistent learning across sessions
5. **Add db-engineer-agent** (Phase 2) — After migration patterns documented

## 📚 Configuration Files

- `backend-agent.md` — Backend agent definition
- `frontend-agent.md` — Frontend agent definition
- `tester-agent.md` — Tester agent definition
- `AGENT-TEAM.md` — This file (team overview and protocols)

All agents are located in `.claude/agents/` and load context from `.claude-guidelines/`.
