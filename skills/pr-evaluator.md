---
name: pr-evaluator
description: Use when evaluating human reviewer comments on a GitHub PR to classify feedback against code standards and propose standards updates
---

# PR Comment Evaluator

Evaluate human reviewer comments from a GitHub PR, classify each one against the team's
code standards, and propose concise additions to `.claude-guidelines/standards/`.

Runs in **Plan Mode** — evaluate and propose, then wait for approval before writing to disk.

---

## Step 1 — Fetch PR Comments

```bash
gh pr view <PR_URL_OR_NUMBER> --json reviews,reviewThreads,comments
```

If only a PR number was given, resolve it:
```bash
gh pr view <number> --json url --jq '.url'
```

---

## Step 2 — Filter: Humans Only

Exclude any author login ending in `[bot]` or known automation accounts
(`dependabot`, `github-advanced-security`, etc.).

Keep: `OWNER`, `MEMBER`, `CONTRIBUTOR`, `COLLABORATOR` associations.

List the comments you'll evaluate (numbered) before proceeding.

---

## Step 3 — Read Relevant Standards

Standards are in `.claude-guidelines/standards/`:
- `code-standards-backend.md` — Rails/Ruby, API, DB, background jobs
- `code-standards-frontend.md` — Vue 3, TypeScript, composables
- `testing-standards.md` — RSpec, Vitest, coverage

Read only the files relevant to this PR's domain. Skim headings to know what's
already documented before classifying.

---

## Step 4 — Classify Each Comment

For each human comment, output:

```
### [N] @author — "<first ~10 words>"
Classification: Good Practice | Nitpick | Side Comment | Disagree
Rationale: one sentence
Already in standards: Yes (<file> › <section>) | Partially | No
Action: Add to <file> | Update <file> › <section> | No action | Flag for discussion
```

**Classification rules:**

- **Good Practice** — An explicit, repeatable recommendation where the reviewer
  states *what should be done*. A question without a recommendation is NOT this.
  > "Use keyword args here" → Good Practice
  > "Why do we use positional args?" → Nitpick (question, no recommendation)

- **Nitpick** — Minor style preference, or a question without a clear recommendation.

- **Side Comment** — Contextual to this PR: future work notes, "this is fine for now",
  general praise. Not a repeatable rule.

- **Disagree** — Contradicts an existing standard or architectural decision.
  Never auto-apply. Surface to the team.

---

## Step 5 — Propose Standards Updates

Only for **Good Practice** items where the standard is **Partially covered or missing**.

Keep proposed additions lean — one rule, one good/bad example pair:

```markdown
**Rule:** <one-sentence rule>
**Why:** <one-sentence rationale>
✅ `<good example>`
❌ `<bad example>`
```

State the target file and section. If a section doesn't exist, name a new one.

Do **not** write to any file yet.

---

## Step 6 — Summary Table

```
## PR #<N> Evaluation

Comments: <total> human, <N> bot excluded

| # | Author | Classification | Action |
|---|--------|---------------|--------|
| 1 | @alice | Good Practice | Add → code-standards-backend.md › Error Handling |
| 2 | @bob   | Nitpick       | No action |
| 3 | @alice | Disagree      | Flag for discussion |

Standards to update (pending approval):
- code-standards-backend.md — 1 new rule
- testing-standards.md — 1 new rule
```

Ask: "Apply all, apply specific items, or skip?"

---

## Step 7 — Apply Approved Updates

When approved:

1. Read the target standards file fully
2. Find the right section — integrate naturally, don't create a "PR #N additions" section
3. Write the rule in the lean format from Step 5, matching the file's existing style
4. Confirm what was added

---

## Key Rules

- Questions without explicit recommendations = Nitpick or Side Comment, not Good Practice
- Disagree items = never auto-apply, always surface to Bryan
- Proposed additions = one rule + one good/bad example pair, no essays
- Don't duplicate — if already in standards, note where and skip
- Don't remove or rewrite existing standards — only extend
