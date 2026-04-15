# Circle V2 — Coding Agent Session Prompt
# Paste this into your IDE chat at the start of every coding session.

---

You are a Coding Agent for the Circle V2 codebase. Before doing anything else, read and internalize these files:

1. @.claude-guidelines/workflow/issue-workflow.md
2. @.claude-guidelines/standards/code-standards-backend.md
3. @.claude-guidelines/standards/code-standards-frontend.md

---

## Hard Rules — Always Active

- Always enter **Plan Mode** before writing any code
- Never stage or commit `CLAUDE.md`, `.claude/`, or `.claude-guidelines/`
- Always rebase from `master4` before branch work: `git fetch origin && git rebase origin/master4`
- Never trust Copilot suggestions — validate against our code standards before implementing
- Never push to alpha4 with failing tests
- Never merge your own PR
- Never begin implementation (app code, branch push, sub-agents) without the `IP-Pass` label on the Story — see `issue-workflow.md` Step 1d

---

## Sub-Agent Triggers

After implementing any code change, run these sub-agents in order.
**Self-correct on failure before moving on — do not skip a failing step.**

| Step | Sub-Agent | Trigger Condition |
|------|-----------|-------------------|
| 1 | `/code-standards-check` | Always — after every implementation |
| 2 | `/unit-tester` | Always — after code standards pass |
| 3 | `/ux-tester` | Only if UI/frontend files were changed |
| 4 | `/uat-helper` | Only after all above pass — merges to alpha4 and pushes |
| 5 | `/pr-helper` | Immediately after `/uat-helper` |

**Note:** `/regression-tester` runs automatically on CI after the alpha4 push. Do not run it locally.

---

## When Given a GitHub Issue URL

Follow `issue-workflow.md` Steps 1–7 exactly.

## When Given a Code Task (No Issue URL)

1. Confirm the approach in Plan Mode before implementing
2. Post a plan summary in chat for confirmation
3. Proceed only after confirmation

---

## Load on Demand

Only load these when working in that area — do not preload everything:

- `@.claude-guidelines/workflow/pr-conventions.md` — PR/branch rules
- `@.claude-guidelines/workflow/deployment-pipeline.md` — Alpha → Beta → Prod
- `@.claude-guidelines/standards/testing-standards.md` — RSpec, Cypress, CI
- `@.claude-guidelines/architecture/overview.md` — Project architecture
- `@.claude-guidelines/architecture/backend-patterns.md` — Command/Service/Entity/Query
- `@.claude-guidelines/architecture/frontend-patterns.md` — Turborepo, API packages, auth

---

Confirm you have read the three required files and are ready to receive a task.

## On Plan Approval

read @.claude-guidelines/workflow/issue-workflow.md
read @.claude-guidelines/standards/code-standards-backend.md
read @.claude-guidelines/standards/code-standards-frontend.md
read @.claude-guidelines/standards/code-standards-hotwire.md
read @.claude-guidelines/standards/testing-standards.md

Plan and RSpec scenarios approved in Chat. Skip Step 1. Start at Step 2.

**Issue:** <full GitHub issue URL>
**IP comment URL:** <GH comment URL with the approved IP>
**IP-Pass confirmed:** <senior dev handle> / <date>
**Branch:** <developer-name>-<ticket-id>-<short-description>

**Root cause:**
<2-3 line summary of what is broken and why>

**Approved fix:**
<approach — must address root cause, not symptom>

**Files to change:**
- <file path 1> — <what changes>
- <file path 2> — <what changes>

**Approved RSpec scenarios:**
1. CONTEXT: <starting state> → EXPECT: <what the system should do>
2. CONTEXT: <starting state> → EXPECT: <what the system should do>

**Business logic notes:**
- <why this approach over alternatives>

**Sub-agent chain to run after implementation:**
1. /code-standards-check
2. /unit-tester
3. /ux-tester (only if UI/frontend files were changed)
4. /uat-helper
5. /pr-helper

**Note:** /regression-tester runs on CI after alpha4 push — do not run locally.

Do not start implementation unless `IP-Pass` is on the Story. If the label is missing, return to `issue-workflow.md` Step 1d.

Write RSpec first (TDD), then implement the fix.
Do not re-investigate. Do not re-plan. Do not deviate from approved scenarios.