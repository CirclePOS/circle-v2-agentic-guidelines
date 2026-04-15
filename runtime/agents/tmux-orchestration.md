# tmux Orchestration — Circle V2 Agent Team

> ⚠️ **Complex work only.** tmux orchestration is for multi-story, multi-role work. Single-issue bug fixes need no tmux setup — invoke `backend-dev-agent` or `frontend-dev-agent` directly.

## Mental Model

| tmux unit | Maps to |
|-----------|---------|
| **Session** | One project or major independent workstream |
| **Window** | One PM/coordinator stream or major task track |
| **Pane** | One active execution role inside that track |

---

## When to Create Each Unit

### Session — one per project / major workstream

Create a new tmux session when:
- Starting work on a distinct project or Epic unrelated to active sessions
- A workstream is large enough that mixing it with an existing session would cause confusion

```bash
tmux new-session -s circle-<project-slug>
# e.g. tmux new-session -s circle-checkout-v2
```

Reuse an existing session if the work is a follow-on story within the same project.

### Window — one per PM/coordinator track

Create a new window when:
- A PM agent is actively coordinating a major task track
- A workstream deserves its own visible track in the session

```bash
tmux new-window -t circle-<project-slug> -n "be-pm"
tmux new-window -t circle-<project-slug> -n "fe-pm"
```

**Naming convention:** use a short role slug — `main`, `be-pm`, `fe-pm`, `qa`, `github`.

### Pane — one per active execution role in a window

Split a window into panes when the window can absorb it readably (2–3 panes max).

```bash
tmux split-window -h   # side-by-side split
tmux split-window -v   # top-bottom split
```

**If the window is already at 3 panes, open a new window instead.** Unreadable pane density costs more than the extra tab switch.

---

## Recommended Layout for Complex Work

```
Session: circle-<project-slug>
│
├── Window 0: main          ← Human operator stays here. Top-level coordinator.
│   └── (no splits — main terminal stays clean)
│
├── Window 1: be-pm         ← backend-pm-agent when backend track is active
│   ├── Pane 0: backend-pm-agent
│   └── Pane 1: backend-dev-agent (current scoped task)
│
├── Window 2: fe-pm         ← frontend-pm-agent when frontend track is active
│   ├── Pane 0: frontend-pm-agent
│   └── Pane 1: frontend-dev-agent (current scoped task)
│
└── Window 3: qa            ← tester-agent / flaky-test-resolver-agent
    ├── Pane 0: tester-agent
    └── Pane 1: flaky-test-resolver-agent (when a flaky test is being investigated)
```

`github-agent` does not need a dedicated window — it runs briefly and reports back. Spawn it in a temporary pane in `main` or the PM window that triggered it, then close it after the report.

---

## Agent → Pane Mapping

| Agent | Preferred location |
|-------|--------------------|
| main agent (you) | Window `main` — no pane splits |
| `backend-pm-agent` | Window `be-pm`, Pane 0 |
| `backend-dev-agent` | Window `be-pm`, Pane 1 |
| `frontend-pm-agent` | Window `fe-pm`, Pane 0 |
| `frontend-dev-agent` | Window `fe-pm`, Pane 1 |
| `tester-agent` | Window `qa`, Pane 0 |
| `flaky-test-resolver-agent` | Window `qa`, Pane 1 (when active) |
| `github-agent` | Temporary pane in the PM window that triggered it; close after report |

---

## Pane Naming

Set pane titles for at-a-glance readability:

```bash
printf '\033]2;<role>: <story-slug>\033\\'
# e.g. printf '\033]2;be-dev: checkout-command\033\\'
```

Use the format: `<role>: <story-or-task-slug>`

Examples:
- `be-dev: checkout-command`
- `fe-dev: payment-form`
- `tester: checkout-rspec`
- `flaky: sell-gift-cards-spec`
- `github: epic-update`

---

## One Task Per Dev Agent

Each dev agent pane handles **one scoped task at a time**. The PM agent assigns the next task only after the current one is validated and closed.

- Do NOT queue multiple tasks in one pane
- Do NOT reuse a pane for a different story without updating the pane title
- When a task completes, the pane may be reused for the next task — rename it before starting

---

## Status Reporting Format

Agents should report back to the main agent using this 4-line structure:

```
Task:    <what was assigned>
Status:  done | in-progress | blocked | failed
Blocker: <if blocked — specific reason and what is needed>
Next:    <recommended next action for the main agent or PM>
```

Example — successful completion:
```
Task:    Implement CheckoutCommand with nil-country validation
Status:  done
Blocker: none
Next:    tester-agent to run unit-tester on spec/commands/checkout_command_spec.rb
```

Example — blocked:
```
Task:    Add gift card balance column to orders table
Status:  blocked
Blocker: Schema change required — needs db-engineer-agent review before migration
Next:    Escalate to db-engineer-agent (Phase 2 agent — not yet available)
```

Keep reports to 4 lines. No prose. Roll up to the main agent window.

---

## Lifecycle Rules

### When to reuse a pane
- The same agent is starting a new task in the same window
- Always update the pane title before starting

### When to close a pane
- The agent's work is fully validated and handed off
- A flaky-test-resolver investigation is complete and fixed
- A github-agent action is done and reported

### When to close a window
- All tasks in that track are complete and validated
- The PM agent has reported final status back to main

### When to close a session
- All epics/stories for the project are merged to alpha4
- QA has accepted on alpha4

---

## Avoiding Common Mistakes

| Mistake | Rule |
|---------|------|
| Too many panes → unreadable | Max 3 panes per window; open a new window instead |
| Reusing a pane silently for a different task | Always rename the pane title |
| Running full-team setup for a single bug fix | Only complex multi-story work uses this layout |
| PM agent assigning multiple tasks at once | One task at a time; wait for validation before assigning next |
| Leaving zombie panes open after task completion | Close panes after their task is validated |
| Verbose agent output cluttering panes | Agents use the 4-line status format; no prose summaries |

---

## Quick Setup Reference

```bash
# 1. Create session for a new project
tmux new-session -s circle-<slug>

# 2. Add windows for each active track
tmux new-window -t circle-<slug> -n "be-pm"
tmux new-window -t circle-<slug> -n "fe-pm"
tmux new-window -t circle-<slug> -n "qa"

# 3. Split a PM window for its dev agent
tmux select-window -t circle-<slug>:be-pm
tmux split-window -h

# 4. Name a pane
printf '\033]2;be-dev: checkout-command\033\\'

# 5. Navigate windows
Ctrl-b <number>      # jump to window by number
Ctrl-b n / Ctrl-b p  # next/prev window
Ctrl-b w             # interactive window list
```
