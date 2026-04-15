# Handoff Template — Chat to Terminal

read @.claude-guidelines/workflow/issue-workflow.md
read @.claude-guidelines/standards/testing-standards.md

Plan and RSpec scenarios already approved in Chat. Skip Step 1.

- **Issue:** <URL>
- **IP comment URL:** <link to the GitHub comment containing the approved IP>
- **IP-Pass confirmed:** yes — label applied by <senior dev handle> on <date>
- **Branch:** <developer-name>-<ticket-id>-<description>
- **Root cause:** <2-3 lines — what is broken and why>
- **Fix:** <approach — must address root cause, not just the symptom>
- **Files to change:**
  - <file path 1> — <what changes>
  - <file path 2> — <what changes>

### Approved RSpec Scenarios

| # | Context | Expected behavior |
|---|---------|-------------------|
| 1 | <starting state> | <what the system should do> |
| 2 | <starting state> | <what the system should do> |

### Business Logic Notes

- <why this approach was chosen over alternatives>

---

Start at Step 2. Write RSpec first (TDD), then implement the fix.
Do not re-investigate or re-plan. Do not deviate from the approved scenarios.

If `IP-Pass` is **not** on the Story, stop — return to `issue-workflow.md` Step 1d. Do not begin implementation.
