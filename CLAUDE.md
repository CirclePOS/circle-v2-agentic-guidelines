# CLAUDE.md — Circle V2 Coding Agent
# ⚠️ This file is gitignored. Do NOT stage, commit, or include in any PR.

## Project

Circle V2 is a Rails 8 full-stack POS platform with **Hotwire** frontend (primary) and Vue 3 legacy support.
- Backend: Rails 8 app + REST API (`/api/v1/`)
- Frontend: **Hotwire (Rails views + Turbo/Stimulus)** — primary for new features
- Legacy Frontend: Vue 3 Turborepo monorepo (`app/vue3/`) — maintenance & bug fixes only
- Mobile: POS via Capacitor (iOS/Android)
- Key infra: MySQL 8.4, Elasticsearch 8.15, Redis 6.2, Sidekiq, Clockwork, Doorkeeper OAuth2
- **Strategy**: New features → Hotwire. Vue 3 → Phase out gradually. Bug fixes welcome in both.

## Agent Guidelines Location

All workflow, standards, architecture docs, and agent definitions live in:
`.claude-guidelines/` (symlinked to your local Claude Guidelines folder)

## Using Specialized Agents

Invoke agents for domain-specific work. Each agent loads its own context automatically.

| Agent | When to Use | Invocation |
|-------|-----------|-----------|
| `backend-agent` | Rails APIs, commands, services, models | `claude-code --agent backend-agent` |
| `frontend-agent` | **Hotwire** (primary) or Vue 3 bug fixes | `claude-code --agent frontend-agent` |
| `tester-agent` | RSpec/Cypress strategy, failure triage, coverage | `claude-code --agent tester-agent` |

**Default behavior**: Agents detect when work should hand off to another agent. Load context files automatically. Use shared skill library.

**More**: See `.claude-guidelines/agents/AGENT-TEAM.md` for team structure, hand-off protocols, and workflows.

## Code-Review Plugin (Supplementary)

Automatic code review analysis (runs alongside skills, never blocks workflow).

| Trigger | Action |
|---------|--------|
| **Manual** | `/code-review` — Optional pre-PR analysis |
| **Auto on PR comments** | Analyzes reviewer feedback (informational) |
| **Configuration** | `.claude/settings.json` (enabled by default) |

**Scope**: Validates against `code-standards-*.md` + `testing-standards.md`  
**Strategy**: Supplementary (early signals, not enforcement)  
**Replaces**: None (complements `/code-standards-check`, `/pr-evaluator`)

**More**: See `CODE-REVIEW-PLUGIN.md` for integration details, best practices, and FAQ.

## On Session Start — Always Load

```
@.claude-guidelines/workflow/issue-workflow.md
@.claude-guidelines/standards/code-standards-backend.md
@.claude-guidelines/standards/code-standards-frontend.md
```

## Load on Demand

| When working on...          | Load this                                                              |
|-----------------------------|------------------------------------------------------------------------|
| PR / branch rules           | `@.claude-guidelines/workflow/pr-conventions.md`                      |
| Deployment pipeline         | `@.claude-guidelines/workflow/deployment-pipeline.md`                 |
| Testing setup               | `@.claude-guidelines/standards/testing-standards.md`                  |
| Architecture / patterns     | `@.claude-guidelines/architecture/overview.md`                        |
| Backend patterns deep-dive  | `@.claude-guidelines/architecture/backend-patterns.md`                |
| Frontend patterns deep-dive | `@.claude-guidelines/architecture/frontend-patterns.md`               |

## Skill Workflow — Always Use These Skills

> **This file is the single CLAUDE.md for this project.** The root `CLAUDE.md` is a stub that points here. Do not create or maintain a second CLAUDE.md.

Use the skills in `.claude-guidelines/skills/` at each stage. These override Superpowers equivalents (e.g. do not substitute `superpowers:test-driven-development` for `tdd-workflow`).

| Stage | Skill | Invocation |
|---|---|---|
| **Test-first development** | `tdd-workflow` | `/tdd-workflow` |
| Run backend tests (RSpec) | `rspec-runner` | `/rspec-runner` |
| Write or update RSpec tests | `test-writer` | `/test-writer` |
| Run unit tests for changed files | `unit-tester` | `/unit-tester` |
| Full regression before alpha4 push | `regression-tester` | `/regression-tester` |
| Smoke test after deploy | `smoke-tester` | `/smoke-tester` |
| UI / frontend changes | `ux-tester` | `/ux-tester` |
| Validate code against standards | `code-standards-check` | `/code-standards-check` |
| Merge feature branch into alpha4 | `uat-helper` | `/uat-helper` |
| Create a GitHub PR | `pr-helper` | `/pr-helper` |
| Review PR feedback from humans | `pr-evaluator` | `/pr-evaluator` |

**Superpowers**: Still use `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:systematic-debugging`, etc. for planning and debugging — they do not conflict with the skills above.

## Hard Rules — Always Active

1. **Never stage or commit:** `CLAUDE.md`, `.claude/`, `.claude-guidelines/`
2. **Always rebase from master4** before branch work: `git fetch origin && git rebase origin/master4`
3. **Always enter Plan Mode** before writing code — post plan as a GH issue comment first and in-chat for feedback before implementation
4. **Never trust Copilot suggestions blindly** — validate against code standards before implementing
5. **Never push to alpha4 with failing tests**
6. **Never merge your own PR** — requires CI green + QA accepted + 2 peer reviews

## Default Task Behaviour

When given a GitHub issue URL or PR URL with no other instruction:
1. Read @.claude-guidelines/workflow/issue-workflow.md
2. Read @.claude-guidelines/standards/code-standards-backend.md
3. Read @.claude-guidelines/standards/code-standards-frontend.md
4. Enter Plan Mode — post the implementation plan as a GH issue comment
5. Wait for approval before writing any code

This applies in Terminal automatically. In Chat, load AI-SESSION-PROMPT.md first:
@.claude-guidelines/AI-SESSION-PROMPT.md

## Dev Environment Quick Reference

**Hotwire Development (Primary):**
```bash
docker compose up app8                        # Rails on :3008 (includes Hotwire views)
docker compose --profile sidekiq up job8      # Sidekiq (for background jobs)
```

**Vue 3 Development (Legacy - Maintenance Only):**
```bash
cd app/vue3 && npm run dev                    # Vue3 devservers (:1233 backoffice, :5173 pos)
```

**Rails Commands:**
```bash
docker compose run --rm app8 bundle exec rails <command>
# or locally:
BUNDLE_GEMFILE=gemfiles/rails8_0/Gemfile bundle exec rails <command>
```
