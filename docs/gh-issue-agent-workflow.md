# GitHub Issue Story Workflow

This document explains how the Circle V2 agent workflow should run when we pick up a GitHub Story or Issue.

## Simple/Human-friendly explanation

When we pick a story, the first job is not to code. The first job is to understand the problem properly.

The working flow is:

1. Read the GitHub issue carefully.
2. Research the existing code and find the real root cause.
3. Write down the plan before changing code.
4. Define the test scenarios that prove the fix or feature is correct.
5. Get business-logic signoff on the plan and scenarios.
6. Post the Implementation Plan into the story and wait for senior-dev approval (`IP-Pass`).
7. Only after approval, create the branch and start implementation.
8. Use the right specialist agent for the job.
9. Run the required checks before pushing.
10. Push to `alpha4`, open the PR, and wait for QA + CI + peer approvals.

In practical terms, the workflow is meant to stop us from rushing into a patch that only hides symptoms.

The idea is:

- Fix the source, not the symptom.
- Prove the behavior with tests.
- Get agreement on the business logic before coding.
- Use specialist agents only when they add value.

For a simple backend-only bug fix, using `backend-dev-agent` directly is usually enough.

For a simple frontend-only task, using `frontend-dev-agent` directly is usually enough.

For a testing or regression investigation, `tester-agent` is usually the right tool.

`Agent Team — Circle V2` is better for work that crosses domains, such as:

- backend + frontend
- implementation + test strategy
- unclear ownership
- multi-step handoff work

So no, the team orchestrator does not need to be used for every small issue. For small isolated fixes, using a single specialist agent is usually cheaper and more efficient.

## Technical explanation

The current repository workflow is primarily defined by:

- `.claude-guidelines/workflow/issue-workflow.md`
- `CLAUDE.md`
- the agent files under `.claude/agents/` and `.claude-guidelines/runtime/agents/`

### 1. Intake and planning

Before writing code, the agent should:

1. Read the GitHub issue.
2. Review relevant business logic and surrounding code.
3. Identify root cause.
4. Draft RSpec/Cypress scenarios before implementation.
5. Present both implementation plan and scenarios for business review.

This is an explicit requirement in the issue workflow. The process is intentionally front-loaded so the team can challenge the proposed fix before implementation starts.

### 2. Implementation Plan approval gate

Every Story requires an Implementation Plan in the story body and in a story comment.

The Implementation Plan must include:

- root cause
- approach
- files to change
- approved scenarios
- business-logic notes
- risk / rollback notes

The story then waits for senior developer review. Coding should not proceed past draft test work until the story receives the `IP-Pass` label.

### 3. Branching

After `IP-Pass`:

1. Rebase from `origin/master4`
2. Create the feature branch

Expected naming pattern:

`<developer-name>-<ticket-id>-<short-description>`

### 4. Agent selection

The agent model is domain-based, not mandatory team-orchestrator-first.

Recommended selection:

- `backend-dev-agent` for Rails models, commands, services, APIs, validations, jobs
- `frontend-dev-agent` for Hotwire or Vue work, forms, and UI behavior
- `tester-agent` for test strategy, failure triage, coverage, and regression analysis
- `Agent Team — Circle V2` for cross-domain stories or when hand-offs are expected

Important distinction:

`AGENT-TEAM.md` describes an orchestration model and hand-off expectations. It does not mean the full team must be invoked on every issue. It is best treated as an operating model, not as a mandatory per-story runtime requirement.

### 5. Implementation and validation sequence

Once implementation begins, the issue workflow says to run validation in this order:

1. `/code-standards-check`
2. `/unit-tester`
3. `/ux-tester` only if UI/frontend changed

If any of those fail, fix and rerun before moving on.

### 6. Team discussion gate

Before pushing to `alpha4`, the workflow requires a stop-and-discuss checkpoint:

- summarize implementation
- confirm approved scenarios are green
- flag pre-existing failures
- raise scope or naming questions
- get explicit go-ahead

### 7. Push and PR sequence

After local validation passes:

1. Push through `/uat-helper` to `alpha4`
2. Create PR through `/pr-helper`
3. Target `master4`, not `alpha4`

### 8. Merge gate

The PR cannot be merged until all three are satisfied:

1. CI is green
2. QA has accepted
3. Two peer reviews are approved

### 9. Practical usage guidance

Use the smallest agent setup that matches the work:

- Simple backend bug: `backend-dev-agent`
- Simple frontend bug: `frontend-dev-agent`
- Test investigation only: `tester-agent`
- Cross-functional story: `Agent Team — Circle V2`

This keeps token usage lower while still following the intended quality gates.