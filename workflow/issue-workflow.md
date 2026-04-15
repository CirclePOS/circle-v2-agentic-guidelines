# Issue Workflow — Steps 1–7

Full pipeline for every GitHub issue assigned to the Coding Agent.
**Follow this exactly — no shortcuts.**

---

## Step 1 — Read the issue, review business logic, and plan

```bash
gh issue view <url>
```

- **Always enter Plan Mode before writing any code** — no exceptions

### Step 1a — Research and root-cause analysis

- Research the codebase, identify the root cause
- **Fix the source, not the symptom.** If a value can be nil/blank but shouldn't be, the fix is to prevent the bad state — not to add nil-guards downstream. Ask: *"Why is this data bad?"* before asking *"How do I survive bad data?"*
- If the story includes a suggested fix (Copilot or human), critically evaluate it:
  - Does it treat the root cause or just mask the symptom?
  - Does it allow invalid data to persist in the database?
  - Would a user hit the same class of bug again through a different path?

### Step 1b — Plan RSpec scenarios (business logic review)

Before writing the implementation plan, draft the **test scenarios** that prove the fix is correct.
This happens in Chat / Plan Mode — no code yet.

For each scenario, define:
- **Context:** the starting state (e.g., "when customer has no country on their address")
- **Expected behavior:** what the system should do (e.g., "pre-fills country from the website's default country")
- **Why this scenario matters:** ties back to the root cause

Anti-pattern to watch for:
> Writing a spec that asserts nil/blank/invalid data is _acceptable_ when the correct fix is to prevent that data from ever existing. See `testing-standards.md` — "Fix the Source, Not the Symptom."

### Step 1c — Present plan and scenarios for PM business-logic review

Present in chat (or post to the GH issue) **both**:
1. The implementation plan (root cause, approach, files to change)
2. The list of RSpec scenarios with context / expected behavior

**Wait for the PM to sign off on the business logic and scenarios.** Once signed off, proceed to Step 1d to formalize the Implementation Plan and request Senior Dev approval.

---

## Step 1d — Submit the Implementation Plan (IP) for Senior Dev approval

Every Story requires a Senior Developer–approved Implementation Plan before any code (other than draft RSpec) is written. Approval is indicated by the `IP-Pass` label on the Story. Senior dev handles live in [`senior-devs.md`](senior-devs.md).

### Step 1d.1 — Write the IP into the Story

Post the Implementation Plan in **both** places:

1. The **`Implementation Plan`** section of the Story description (edit the Story body).
2. A **new comment** on the Story with the same IP content — the comment is what the senior dev will review and respond to.

The IP content must include:
- **Root cause** (2–3 lines)
- **Approach** (must address the root cause, not the symptom — see Step 1a)
- **Files to change** (path + what changes)
- **Approved RSpec scenarios** from Step 1b (Context → Expected behavior)
- **Business logic notes** (why this approach vs. alternatives)
- **Risk / rollback notes** (if any)

### Step 1d.2 — Tag the appropriate Senior Developer(s)

At the bottom of the IP comment, @-mention the senior devs whose domain matches the Story. Handles live in [`senior-devs.md`](senior-devs.md):

- **Frontend Story** → tag `@markhermano` and `@jpyepez` (pick one; the other is backup)
- **Backend Story** → tag all backend seniors: `@DiegoOliveiras @maurodias @maksym-bobmm @dyogoveiga`
- **Full-stack Story** → tag one backend senior and all frontend seniors

The Coding Agent can automate this by posting the comment via:

```bash
gh issue comment <url> --body "<IP content>

cc @handle1 @handle2"
```

### Step 1d.3 — Wait for `IP-Pass` (soft gate)

There are **two** possible labels the senior dev can apply:

- **`IP-Pass`** — plan is approved; proceed to Step 1d.4.
- **`IP-Fail`** — plan needs revision; senior dev will leave feedback in a comment.

While waiting for review:
- ✅ **Allowed:** drafting RSpec files locally (Red phase only — no implementation), reading related code, setting up the branch name
- ❌ **Blocked:** writing application code, pushing the branch, running the sub-agent chain, opening the PR

**If `IP-Fail` is applied (or feedback is posted without a label):**

1. Read the senior dev's feedback comment(s)
2. Revise the IP in the Story **description** (update the `Implementation Plan` section)
3. Post a **new comment** containing the revised IP
4. Re-tag the senior dev(s) at the bottom of the new comment (same handles as Step 1d.2)
5. **Remove the `IP-Fail` label** from the Story:

   ```bash
   gh issue edit <url> --remove-label "IP-Fail"
   ```
6. Wait for `IP-Pass` (or another `IP-Fail` cycle)

Repeat until `IP-Pass` is applied.

### Step 1d.4 — Proceed once `IP-Pass` is on the Story

Verify the label is present before moving on:

```bash
gh issue view <url> --json labels
```

Only then move to Step 2.

---

## Step 2 — Set up the branch

On `IP-Pass` label (verified in Step 1d.4):

```bash
git fetch origin && git rebase origin/master4
git checkout -b <developer-name>-<ticket-id>-<short-description>
```

Examples:
```
bryan-14001-add-ledger-export
bryan-14020-fix-payment-serializer
```

**Branch naming rule:** `<developer-name>-<ticket-id>-<short-description>`

---

## Step 3 — Evaluate any code suggestion in the story

Stories may include a code fix suggested by Copilot or by the story author.
**Do NOT blindly trust it.** Suggestions often treat the symptom, not the root cause, and may not follow our code standards.

Before writing any code, evaluate the suggestion:
- Does it fix the root cause, or just add a nil-guard / rescue around the symptom? (See Step 1a)
- Does it put logic in the right layer (Command/Service vs. model vs. view)?
- Does it follow "Tell don't ask" (no inline computation in views)?
- Does it use correct patterns (Command pipeline, serializers, etc.)?

**If the suggestion violates a code standard:**

1. Post a PR comment identifying the violation and corrected approach
2. Report in-chat with this format:

```
Code Standard Violation Detected
- Story suggestion: <what Copilot suggested>
- Violation: <which standard it breaks>
- Correct approach: <what we did instead and why>
```

Then implement the correct version — not the Copilot suggestion.

---

## Step 4 — Implement the fix

Then run sub-agents in sequence. **Self-correct on failure before moving on.**

### Sub-agent trigger conditions

| Sub-agent               | Trigger                                      | On Fail                  |
|-------------------------|----------------------------------------------|--------------------------|
| `/code-standards-check` | Always — after every implementation          | Fix code, re-run         |
| `/unit-tester`          | Always — after code standards pass           | Fix code, re-run         |
| `/ux-tester`            | **Only if UI/frontend was changed**          | Fix code, re-run         |

Run order:
1. `/code-standards-check` → if fail: fix, re-run
2. `/unit-tester` → if fail: fix, re-run
3. `/ux-tester` → **only if UI/frontend changed**; if fail: fix, re-run

**Note:** Do NOT run `/regression-tester` locally. CI will run the full RSpec suite automatically on alpha4 push (see Step 5). This keeps delivery velocity fast.

Do not proceed to Step 5 until all applicable sub-agents pass.

---

## Step 4b — Discuss with team before alpha4 push

Once all local sub-agents pass (code-standards-check, unit-tester, and ux-tester if applicable), **stop and discuss** before proceeding:

- Share the implementation summary and any non-obvious decisions made
- Confirm the approved RSpec scenarios from Step 1b are all green
- Flag any pre-existing failures encountered during unit testing
- Raise questions about approach, naming, or scope
- Get explicit go-ahead before pushing to alpha4

Do not proceed to Step 5 without confirmation.

---

## Step 5 — Push to alpha4 for QA

**Alpha4 safety gate — do NOT push if `/unit-tester` is failing.**

Pre-push checklist:
1. ✅ `/code-standards-check` passed
2. ✅ `/unit-tester` passed (tests on changed files)
3. ✅ `/ux-tester` passed **if UI/frontend changed**
4. Branch is rebased from master4

Then push to alpha4:
```bash
# Run the uat-helper slash command — it handles the merge, push, and QA notification
/uat-helper
```

Alpha deployment flow (what `/uat-helper` does):
```bash
git checkout alpha4
git merge <feature-branch>
git push
```

**Full regression suite runs automatically on CI** after push. Any failures appear in GitHub CI logs and PR status. QA uses CI results to classify issues.

---

## Step 6 — Create the PR immediately after alpha4 push

```bash
/pr-helper
```

The PR must exist while code is on alpha4 — QA links it to the story, and devs begin peer review in parallel with QA testing.

**PR rules:**
- Target branch: `master4` (never alpha4)
- Title: `#GITHUB_ISSUE_NUMBER - Short description`
- Description **Why** section must include `Closes #GITHUB_ISSUE_NUMBER` on its own line
- Every PR requires automated tests; new tests go in `spec/` (RSpec), not `test/` (Minitest)

---

## Step 7 — Wait for the merge gate

**Do NOT merge your own PR.**

Merge requires all three:
- ✅ CI green — 8 RSpec groups + 2 Cypress shards
- ✅ QA accepted on GitHub
- ✅ 2 peer code reviews approved
