# Deployment Pipeline

## Flow

```
Feature branch
    ↓ rebase from master4
    ↓ git merge <branch> into alpha4 (direct merge — no PR against alpha4)
ALPHA (alpha4 + alpha6) — CI deploys automatically
    ↓ Open GitHub PR targeting master4 immediately
      (QA links it; devs begin peer review in parallel with QA testing)
QA manual testing + GitHub Issue QA sign-off — runs in parallel with peer review
    ↓ CI green (8 RSpec groups + 2 Cypress shards) + QA accepted + 2 peer reviews
Merge PR → auto-deploys to BETA (beta4)
    ↓ Beta QA sign-off
PROD — tagged deploy (git tag deploy-v<YY>.<mm>.<dd>), weekly ~Tuesday
```

---

## Environment Reference

| Environment | Branch    | Database              | Notes                        |
|-------------|-----------|----------------------|------------------------------|
| Alpha       | `alpha4`  | Copy of production   | Separate ES/Redis            |
| Beta        | `master4` | Same as production   | Manual testing before prod   |
| Production  | —         | Production           | Tagged deploy, weekly ~Tue   |

---

## Key Rules

- **Always rebase from `master4`** before merging to alpha4:
  ```bash
  git fetch origin && git rebase origin/master4
  ```
- **Alpha deployment** = direct merge, no PR:
  ```bash
  git checkout alpha4 && git merge <feature-branch> && git push
  ```
  (handled by `/uat-helper`)
- **PRs target `master4`** — not alpha4
- **Never push to alpha4 with failing tests**
- **Never deploy to prod manually** — use the tagged deploy process
