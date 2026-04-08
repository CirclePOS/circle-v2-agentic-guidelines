# Circle V2 Agentic Guidelines

Shared coding agent configuration, standards, and workflows for Circle V2 development team.

**This is a separate repo** — guidelines don't bloat your main codebase. Your main Circle V2 repo symlinks to this.

---

## 🚀 For Team Members (New Setup)

**Start here**: Read [`TEAM-SETUP.md`](./TEAM-SETUP.md) — 5-minute setup guide

```bash
# Quick version:
git clone https://github.com/BryanCamua/circle-v2-agentic-guidelines.git ~/Documents/LocalProjects/circle-v2-agentic-guidelines
cd ~/Documents/GitHub/Circle-v2-master4-deploy
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines
```

✅ Done! You now have:
- 3 specialized agents (backend, frontend, tester)
- 13 reusable skills (testing, validation, deployment)
- Code standards (backend, frontend, Hotwire)
- Testing standards (RSpec, Cypress, system tests)
- Workflow documentation

---

## 📖 Documentation

| Document | Purpose |
|----------|---------|
| **[`TEAM-SETUP.md`](./TEAM-SETUP.md)** | 🎯 **Start here** — Clone & symlink instructions |
| **[`QUICK-START.md`](./QUICK-START.md)** | How to use agents and skills |
| **[`CLAUDE.md`](./CLAUDE.md)** | Root agent configuration (loaded by Claude Code) |
| **[`CODE-REVIEW-PLUGIN.md`](./CODE-REVIEW-PLUGIN.md)** | Supplementary code review integration |
| **[`SETUP.md`](./SETUP.md)** | Legacy setup info (see TEAM-SETUP.md instead) |

---

## 📁 Directory Structure

```
.
├── TEAM-SETUP.md                ← Team members start here
├── QUICK-START.md               ← Agent & skill usage guide
├── CLAUDE.md                    ← Project configuration
├── CODE-REVIEW-PLUGIN.md        ← Code review integration
│
├── agents/                      ← Specialized agents
│   ├── AGENT-TEAM.md            ← Team structure & hand-offs
│   ├── backend-agent.md         ← Rails development
│   ├── frontend-agent.md        ← Hotwire + Vue 3 (legacy)
│   └── tester-agent.md          ← Test strategy & QA
│
├── skills/                      ← 13 reusable skills
│   ├── tdd-workflow.md          ← Test-first development
│   ├── rspec-runner.md          ← Backend test execution
│   ├── code-standards-check.md  ← Code validation
│   ├── regression-tester.md     ← Full test suite
│   ├── pr-helper.md             ← Create GitHub PR
│   └── ... (7 more skills)
│
├── standards/                   ← Development standards
│   ├── code-standards-backend.md   ← Rails patterns
│   ├── code-standards-frontend.md  ← Vue 3 patterns
│   ├── code-standards-hotwire.md   ← Hotwire patterns (NEW)
│   └── testing-standards.md        ← RSpec, Cypress, system tests
│
├── workflow/                    ← Team workflows
│   ├── issue-workflow.md        ← Issue → PR → merge (Steps 1–7)
│   ├── pr-conventions.md        ← Branch naming & PR rules
│   ├── deployment-pipeline.md   ← Alpha → beta → prod
│   └── handoff-template.md      ← Agent hand-off template
│
└── architecture/                ← Architecture & patterns
    ├── overview.md              ← Project stack summary
    ├── backend-patterns.md      ← Command/Service/Entity patterns
    └── frontend-patterns.md     ← Turborepo, API, Hotwire patterns
```

---

## 🎯 What This Enables

### **Agents** (Invoke with `claude-code --agent <name>`)

- **`backend-agent`** — Rails models, commands, services, APIs
- **`frontend-agent`** — Hotwire (primary), Vue 3 (maintenance)
- **`tester-agent`** — RSpec/Cypress strategy, failure triage

Each agent auto-loads relevant standards and skills.

### **Skills** (Invoke with `/skill-name`)

Workflow automation at each stage:

```
/tdd-workflow          → Test-first development (Red-Green-Refactor)
/code-standards-check  → Validate code before PR
/rspec-runner          → Run backend tests
/regression-tester     → Full suite + failure classification
/unit-tester           → Quick validation
/ux-tester             → Accessibility checks
/pr-helper             → Create GitHub PR
/pr-evaluator          → Analyze reviewer feedback
```

### **Standards** (Load with `@.claude-guidelines/standards/...`)

- Code patterns by framework
- Testing approaches for each layer
- Architecture decisions documented

### **Hand-Offs** (Auto-detected by agents)

Agents recognize when work should transition:

```
backend-agent: "This needs a Vue component. Handing to frontend-agent."
frontend-agent: "API call failing. Handing to backend-agent."
tester-agent: "New test failure. Escalating to backend-agent."
```

---

## 🌟 Key Features

✅ **Hotwire-first** — New features use Hotwire. Vue 3 in maintenance mode.  
✅ **TDD enforced** — All agents use test-first workflow  
✅ **Code review supplement** — Optional code-review plugin for early feedback  
✅ **Standards-based** — Agents validate against your code standards  
✅ **Team collaboration** — Seniors can update standards independently  
✅ **Non-invasive** — Separate repo, symlinked only (doesn't bloat main repo)  

---

## 🤝 Contributing Guidelines

**For team leads & seniors**: Update standards via PR in this repo

```bash
git checkout -b improve/hotwire-testing-patterns
# Edit standards/code-standards-hotwire.md
git commit -m "Clarify Stimulus controller testing"
git push origin improve/hotwire-testing-patterns
# → Create PR, get reviewed, merge
```

**For all developers**: Pull latest standards

```bash
cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines
git pull origin main
# All your projects now see latest standards!
```

---

## 📊 Repository Status

| Item | Status |
|------|--------|
| Agents | ✅ 3 specialized agents (backend, frontend, tester) |
| Skills | ✅ 13 reusable workflow skills |
| Standards | ✅ Code, testing, Hotwire (all documented) |
| Testing | ✅ RSpec, Cypress, Hotwire system tests |
| Architecture | ✅ Patterns documented |
| Code-Review | ✅ Supplementary plugin integrated |

---

## 🚀 Quick Start

1. **New developer?** → Read [`TEAM-SETUP.md`](./TEAM-SETUP.md)
2. **Using agents/skills?** → Read [`QUICK-START.md`](./QUICK-START.md)
3. **Project guidelines?** → Read [`CLAUDE.md`](./CLAUDE.md)
4. **Code standards?** → Load `@.claude-guidelines/standards/code-standards-*.md`

---

## 📞 Questions?

- **Setup help**: See `TEAM-SETUP.md`
- **Usage help**: See `QUICK-START.md`
- **Suggestion for standards**: Create PR in this repo
- **Emergency**: Use local `.claude/settings.json` to override

---

**Last updated**: 2026-04-08  
**Repository**: https://github.com/BryanCamua/circle-v2-agentic-guidelines  
**Main branch**: `main`
