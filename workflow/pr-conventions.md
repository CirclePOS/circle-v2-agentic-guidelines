# PR & Branch Conventions

## Branch Naming

```
<developer-name>-<ticket-id>-<short-description>
```

Examples:
```
bryan-14001-add-ledger-export
bryan-14020-fix-payment-serializer
```

---

## PR Rules

- **Target branch:** `master4` — never alpha4
- **Title:** `#GITHUB_ISSUE_NUMBER - Short description`
- **Description Why section** must include `Closes #GITHUB_ISSUE_NUMBER` on its own line (auto-closes issue on merge)
- Every PR requires automated tests — new tests go in `spec/` (RSpec), not `test/` (Minitest)
- PR must be created immediately after alpha4 push — QA links it to the story

---

## Files That Must Never Be in a PR

These are local agent config — they live on disk but must never be staged, committed, or included in any PR targeting master4:

- `CLAUDE.md`
- `.claude/` (all contents)
- `.claude-guidelines` (symlink)

All are in `.gitignore`. If you ever see them in `git status` as modified or untracked, **do NOT `git add` them**.

---

## Merge Gate

**Do NOT merge your own PR.** All three must be satisfied:

| Requirement              | Detail                                          |
|--------------------------|-------------------------------------------------|
| CI green                 | 8 RSpec groups + 2 Cypress shards must pass     |
| QA accepted              | GitHub Issue QA sign-off                        |
| 2 peer code reviews      | Both approved on the GitHub PR                  |
