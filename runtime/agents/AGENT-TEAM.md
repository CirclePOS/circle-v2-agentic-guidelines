---
name: Agent Team — Circle V2
description: Specialized agents for backend, frontend, and testing. Each handles their domain with clear hand-off protocols.
---

# Circle V2 Agent Team

An orchestrated team of specialized agents, each responsible for a specific domain of Circle V2 development. Agents coordinate via clear hand-off protocols and shared context loading.

> ⚠️ **Use the agent team for complex work only.** Simple one-story or one-issue bug fixes should be handled directly by the relevant dev agent (`backend-dev-agent` or `frontend-dev-agent`) without PM or full-team orchestration. Invoking the full team for a single-issue fix adds unnecessary overhead.

## 🏗️ Team Structure

```
Agent Team: Circle V2 Development
│
├── 🗂️ backend-pm-agent         [COMPLEX WORK ONLY]
│   ├── Focus: Story decomposition, backend task delegation, coordination
│   ├── Delegates to: backend-dev-agent (one task at a time)
│   ├── Coordinates with: tester-agent, flaky-test-resolver-agent, github-agent
│   └── Does NOT implement code
│
├── 🗂️ frontend-pm-agent        [COMPLEX WORK ONLY]
│   ├── Focus: Story decomposition, frontend task delegation, UX alignment
│   ├── Delegates to: frontend-dev-agent (one task at a time)
│   ├── Coordinates with: tester-agent, flaky-test-resolver-agent, github-agent
│   └── Does NOT implement code
│
├── 🔧 backend-dev-agent
│   ├── Focus: Rails models, commands, services, API design
│   ├── Skills: code-standards-check, tdd-workflow, rspec-runner, pr-helper
│   ├── Constraints: No Vue files, escalate schema changes, scoped tasks only
│   └── Context: CLAUDE.md, code-standards-backend, testing-standards
│
├── 🎨 frontend-dev-agent
│   ├── Focus: Hotwire (new features), Vue 3 (bug fixes), E2E tests
│   ├── Skills: code-standards-check, tdd-workflow, ux-tester, test-writer
│   ├── Constraints: No Rails model files, scoped tasks only
│   └── Context: CLAUDE.md, code-standards-frontend, testing-standards
│
├── 🧪 tester-agent
│   ├── Focus: RSpec/Cypress strategy, failure triage, coverage gaps
│   ├── Skills: rspec-runner, regression-tester, unit-tester, smoke-tester
│   ├── Constraints: Analysis only, never implements code
│   └── Context: CLAUDE.md, testing-standards (both backend + frontend)
│
├── 🔬 flaky-test-resolver-agent
│   ├── Focus: Detect, reproduce, and permanently fix flaky tests (RSpec, Capybara, Cypress)
│   ├── Skills: rspec-runner, regression-tester
│   ├── Constraints: Never uses rerun-until-pass; stress-validates all fixes (20 passes); never modifies production code
│   └── Context: CLAUDE.md, testing-standards, code-standards-backend, code-standards-frontend
│
└── 📋 github-agent              [COMPLEX WORK ONLY]
    ├── Focus: Epic issue creation, sub-issue linking, progress tracking
    ├── Tools: gh CLI (issue create/edit/comment/close)
    ├── Constraints: No product code changes; existing labels/milestones only
    └── Context: CLAUDE.md, issue-workflow, pr-conventions
```

## 🎯 When to Use the Full Team

Use full team orchestration when work spans **multiple stories, multiple files across domains, or requires multi-phase validation**. Examples:
- New multi-step feature (backend + frontend + tests + GitHub tracking)
- Large refactor touching models, services, and views
- Cross-cutting change requiring coordinated backend and frontend work

**Do NOT invoke the full team for:**
- Single bug fix on one file
- One-line patch or label change
- Isolated test fix

For simple work: invoke `backend-dev-agent` or `frontend-dev-agent` directly.

## 🔁 PM → Dev Delegation Rule

PM agents (`backend-pm-agent`, `frontend-pm-agent`) assign **exactly one task at a time** to their dev agent. The next task is not assigned until the current one is complete, validated by `tester-agent`, and tracked via `github-agent`.

## 🖥️ tmux Orchestration

> ⚠️ **Complex work only.** Single-issue bug fixes need no tmux setup — invoke the dev agent directly.

For complex multi-story work, spawn agents into a tmux layout so you can watch progress live.

**Mental model:**
- **Session** = one project / major workstream
- **Window** = one PM/coordinator track (`main`, `be-pm`, `fe-pm`, `qa`)
- **Pane** = one active role inside that track (max 3 per window; open a new window if full)

**Agent → window mapping:**

| Agent | Window |
|-------|--------|
| main agent (you) | `main` — no pane splits |
| `backend-pm-agent` + `backend-dev-agent` | `be-pm` |
| `frontend-pm-agent` + `frontend-dev-agent` | `fe-pm` |
| `tester-agent` + `flaky-test-resolver-agent` | `qa` |
| `github-agent` | Temporary pane in the PM window that triggered it |

**Status report format (4 lines, no prose):**
```
Task:    <what was assigned>
Status:  done | in-progress | blocked | failed
Blocker: <reason if blocked>
Next:    <recommended action for main agent or PM>
```

**Key rules:**
- One scoped task per dev agent pane; rename the pane title when starting a new task
- Close panes after their task is validated; close windows when the track is complete
- PM agents decompose and delegate; dev agents execute only their assigned scoped task

→ **Full details:** `tmux-orchestration.md` in this directory

## 🤝 Hand-Off Protocols

### When Backend Agent Escalates

**→ To Frontend Agent:**
```
"This needs Vue components. Handing off to frontend-dev-agent."
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
"API contract needs review. Handing off to backend-dev-agent."
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

**→ To Flaky Test Resolver Agent:**
```
"Intermittent failure in [spec file]. Rerun-until-pass is not acceptable. Handing off for root cause analysis."
Trigger: Test fails intermittently; needs reproduce → classify → fix → stress-validate cycle
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
   ├── Tests against backend-dev-agent's API
   └── Hands off: "E2E tests passing, ready for QA"

3. Tester Agent
   ├── Confirms `/unit-tester` passed
   └── Reports: "All tests green, ready for QA"

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
   ├── Confirms `/unit-tester` clean
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
   ├── Confirms `/unit-tester` clean
   └── Reports green status (full regression runs on CI after alpha4 push)
```

## 📋 Shared Skills

All agents have access to these shared skills:

| Skill | Purpose | Invoked By |
|-------|---------|-----------|
| `/code-standards-check` | Validate code before PR | All agents (before review) |
| `/tdd-workflow` | Test-driven development | Backend & Frontend agents |
| `/rspec-runner` | Backend test execution | Backend & Tester agents |
| `/pr-helper` | Create GitHub PR | Backend or Frontend (after QA) |
| CI regression | Full suite on alpha4 push | Automatic (post-alpha4) |

## 🔗 How to Use Agent Team

### Invoke a Specific Agent

```bash
# Start backend agent for feature work
claude-code --agent backend-dev-agent -p "Implement checkout command with validation"

# Start frontend agent for UI changes
claude-code --agent frontend-dev-agent -p "Create payment form with validation"

# Start tester agent for test strategy
claude-code --agent tester-agent -p "Analyze test coverage gaps in checkout flow"
```

### Let Agent Decide Hand-Off

Agents are instructed to detect when work should transition to another agent:

```
User: "Add a new field to the user profile form"

Backend Agent: "This is primarily frontend work. Handing off to frontend-dev-agent."

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

| Agent | Status | Ready | Role |
|-------|--------|-------|------|
| backend-dev-agent | ✅ Created | ✅ Yes | Backend implementation (scoped tasks) |
| frontend-dev-agent | ✅ Created | ✅ Yes | Frontend implementation (scoped tasks) |
| backend-pm-agent | ✅ Created | ✅ Yes | Backend PM — complex work only |
| frontend-pm-agent | ✅ Created | ✅ Yes | Frontend PM — complex work only |
| github-agent | ✅ Created | ✅ Yes | Epic/issue tracking — complex work only |
| tester-agent | ✅ Created | ✅ Yes | RSpec/Cypress strategy and failure triage |
| flaky-test-resolver-agent | ✅ Created | ✅ Yes | Flaky test diagnosis and permanent fixes |
| db-engineer-agent | ⏳ Phase 2 | ⏹️ No | Migration runbooks, schema patterns |
| ux-tester-agent | ⏳ Phase 2 | ⏹️ No | WCAG 2.1 AA standards, accessibility patterns |

## 🎯 Next Steps

1. **Test the agents** — Invoke them individually with sample tasks
2. **Document hand-off URLs** — Add to team wiki
3. **Create agent orchestrator** (Phase 2) — Automated routing for complex features
4. **Build agent memory system** (Phase 2) — Persistent learning across sessions
5. **Add db-engineer-agent** (Phase 2) — After migration patterns documented

## 📚 Configuration Files

- `backend-dev-agent.md` — Backend developer agent (scoped implementation)
- `frontend-dev-agent.md` — Frontend developer agent (scoped implementation)
- `backend-pm-agent.md` — Backend PM agent (complex work only)
- `frontend-pm-agent.md` — Frontend PM agent (complex work only)
- `github-agent.md` — GitHub coordination agent (complex work only)
- `tester-agent.md` — Tester agent definition
- `flaky-test-resolver-agent.md` — Flaky test specialist agent definition
- `tmux-orchestration.md` — tmux layout conventions and status reporting for complex multi-agent work
- `AGENT-TEAM.md` — This file (team overview and protocols)

All agents are located in `.claude/agents/` (symlinks to `.claude-guidelines/runtime/agents/`) and load context from `.claude-guidelines/`.
