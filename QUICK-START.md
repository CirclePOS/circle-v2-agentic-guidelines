# Quick Start — Circle V2 Agents

**TL;DR** — Invoke an agent based on your task. Agents handle everything: context loading, skill invocation, and hand-offs.

---

## 🎯 Choose Your Agent

| Task | Agent | Invocation |
|------|-------|-----------|
| **Backend feature** (Rails models, commands, APIs) | `backend-agent` | `--agent backend-agent` |
| **Frontend UI** (Vue components, forms, Cypress) | `frontend-agent` | `--agent frontend-agent` |
| **Test issues** (RSpec failures, coverage, strategy) | `tester-agent` | `--agent tester-agent` |

---

## ⚡ Invoke an Agent

### VS Code Extension
1. Open VS Code
2. Open Claude Code chat
3. Click agent dropdown → Select `backend-agent` / `frontend-agent` / `tester-agent`
4. Type your task in the prompt
5. Agent loads context + skills automatically

### CLI
```bash
# Backend feature
claude-code --agent backend-agent -p "Implement checkout command"

# Frontend UI
claude-code --agent frontend-agent -p "Create payment form with Vee-Validate"

# Test issues
claude-code --agent tester-agent -p "Why are checkout tests failing?"
```

### Web App (claude.ai/code)
1. Go to claude.ai/code
2. Click "Agents" menu
3. Select agent
4. Chat with agent

---

## 📋 What Each Agent Does

### 🔧 Backend Agent
**When to use**: Building Rails features, APIs, commands, services

```
Tasks:
✅ Write RSpec tests (TDD)
✅ Implement Rails models, commands, services
✅ Build REST API endpoints
✅ Handle database operations
✅ Validate code against standards
❌ Never touches Vue files (escalates to frontend-agent)
❌ Never touches Cypress tests (escalates to tester-agent)

Skills:
- /code-standards-check → Validate code before PR
- /tdd-workflow → Test-first development
- /rspec-runner → Run backend tests
- /regression-tester → Full suite + failure classification
- /pr-helper → Create PR after QA approves
```

### 🎨 Frontend Agent
**When to use**: Building Vue components, forms, handling UX

```
Tasks:
✅ Write Cypress E2E tests (TDD)
✅ Implement Vue 3 components
✅ Create forms with Vee-Validate
✅ Handle component state & logic
✅ Check accessibility
❌ Never modifies Rails code (escalates to backend-agent)
❌ Never fixes RSpec tests (escalates to tester-agent)

Skills:
- /code-standards-check → Validate Vue/TypeScript code
- /tdd-workflow → Test-first with Cypress
- /ux-tester → Accessibility compliance
- /test-writer → Generate test templates
- /smoke-tester → Quick E2E validation
- /pr-helper → Create PR after QA approves
```

### 🧪 Tester Agent
**When to use**: Test strategy, debugging failures, coverage analysis

```
Tasks:
✅ Run RSpec suite & analyze failures
✅ Run Cypress E2E tests
✅ Classify failures (new vs pre-existing)
✅ Identify coverage gaps
✅ Debug flaky tests
❌ Never implements features (analysis only)
❌ Never modifies production code

Skills:
- /rspec-runner → Backend test execution
- /regression-tester → Full suite classification
- /unit-tester → Quick validation
- /smoke-tester → Lightweight E2E checks
- /test-writer → Generate test templates
```

---

## 🔄 Hand-Offs (Automatic)

Agents detect when work should go to another agent:

**Backend → Frontend:**
```
Backend Agent: "This needs a Vue component. Handing off to frontend-agent."
```

**Frontend → Backend:**
```
Frontend Agent: "API contract needs review. Handing off to backend-agent."
```

**Either → Tester:**
```
Backend/Frontend Agent: "Need to analyze test coverage. Handing off to tester-agent."
```

**Tester → Backend/Frontend:**
```
Tester Agent: "New RSpec failure detected in checkout. Escalating to backend-agent."
```

---

## 📝 Example Workflows

### Workflow 1: Add Payment Method Selection

```
You: "Add payment method dropdown to checkout form"
     (Invoke: frontend-agent)

Frontend Agent:
├── Reads API spec from backend
├── Writes Cypress test (failing)
├── Implements Vue component with Vee-Validate
├── Runs /ux-tester to check accessibility
└── "API call working? Need backend to finalize endpoint."

You: "Confirm API endpoint exists"
     (Invoke: backend-agent)

Backend Agent:
├── Confirms endpoint exists
├── Runs /rspec-runner to verify backend tests pass
└── "Good to go. Back to you frontend-agent."

Frontend Agent:
├── Finalizes E2E test
└── "Ready for QA. Using /pr-helper to create PR."
```

### Workflow 2: Debug Test Failures

```
You: "Why are checkout tests failing?"
     (Invoke: tester-agent)

Tester Agent:
├── Runs /regression-tester
├── Classifies failures: "3 new RSpec failures in Cart#calculate_total"
├── Identifies root cause
└── "New failures introduced by recent code. Escalating to backend-agent."

You: "Fix the cart calculation"
     (Invoke: backend-agent)

Backend Agent:
├── Reviews failing RSpec test
├── Implements fix
├── Runs /rspec-runner
└── "Cart tests passing. Back to tester-agent for full suite validation."

Tester Agent:
├── Runs /regression-tester
└── "All green. No new failures. Ready for alpha4 push."
```

### Workflow 3: Complete Feature from Scratch

```
You: "Implement new refund workflow"
     (Invoke: backend-agent for API design)

Backend Agent:
├── Plan Mode: Design API endpoints
├── /tdd-workflow: Write RSpec tests
├── Implement: Commands, services, models
├── /code-standards-check: Validate code
└── "API ready. Handing to frontend-agent for UI."

You: (Switch to frontend-agent)

Frontend Agent:
├── Plan Mode: Design form/UI
├── /tdd-workflow: Write Cypress tests
├── Implement: Vue components, forms
├── /ux-tester: Accessibility check
└── "UI ready. Handing to tester-agent."

You: (Switch to tester-agent)

Tester Agent:
├── /regression-tester: Full suite
└── "All tests pass. Feature complete!"

You: (Switch back to backend or frontend)

Backend/Frontend Agent:
├── /pr-helper: Create PR
└── Ready for QA review + peer review
```

---

## 🎬 Common Commands Inside Agent

Once an agent is active, you can invoke skills directly:

```
/tdd-workflow      # Start test-first development
/rspec-runner      # Run specific RSpec test
/regression-tester # Full suite + failure classification
/code-standards-check  # Validate code before PR
/pr-helper         # Create GitHub PR
/ux-tester         # Check accessibility
```

You can also ask the agent questions:
```
"What tests should I write first?"
"Why is this test failing?"
"What's the API contract I need to implement?"
"Is my code following standards?"
```

---

## 🚨 Critical Rules (Always Enforced)

1. **Plan Mode before code** — All agents enforce this
2. **TDD first** — Write failing tests, then code
3. **No .env edits** — Hook blocks these automatically
4. **Never push failing tests** — Agents verify before alpha4 push
5. **Code standards check** — Required before PR
6. **No self-merge** — PR requires 2 peer reviews

---

## ❓ Troubleshooting

**"Agent doesn't load"**
- Verify context files exist: `ls -la .claude-guidelines/standards/`
- Check agent file syntax: Ensure YAML frontmatter is valid

**"Skill doesn't work"**
- Verify symlink exists: `ls -la .claude/commands/[skill-name].md`
- Try full skill name: `/tdd-workflow` instead of `/tdd`

**"Agent hands off incorrectly"**
- Manually invoke the right agent instead
- Example: `--agent backend-agent` to override auto-detection

**"Tests failing but I'm not sure why"**
- Invoke tester-agent: `"Why are these tests failing?"`
- Tester agent will run `/regression-tester` and explain

---

## 📚 Full Documentation

- **Setup Guide**: `.claude/SETUP-SUMMARY.md` — What was installed
- **Agent Team**: `.claude/agents/AGENT-TEAM.md` — Team structure & hand-offs
- **Backend Agent**: `.claude/agents/backend-agent.md` — Full specification
- **Frontend Agent**: `.claude/agents/frontend-agent.md` — Full specification
- **Tester Agent**: `.claude/agents/tester-agent.md` — Full specification
- **TDD Skill**: `.claude-guidelines/skills/tdd-workflow.md` — Workflow guide
- **Project Guide**: `CLAUDE.md` — Circle V2 project guidelines

---

## ✨ You're Ready!

Pick an agent and start building. Agents handle the rest.

```bash
# Example:
claude-code --agent backend-agent -p "Implement cart discount command with validation"
```

Good luck! 🚀
