# Claude Guidelines — Team Setup

This folder contains all Coding Agent configuration for Circle V2.
It lives **outside** the repo so it never gets pushed in PRs, survives rebases from `master4`, and can be maintained independently per developer.

---

## Steps

### 1. Copy this folder to your local machine

Place it anywhere you prefer, e.g.:
```
/Users/<yourname>/Documents/LocalProjects/Claude Guidelines
```

### 2. Create a symlink in the Circle V2 repo root

```bash
cd /path/to/circle-v2-master4-deploy
ln -s "/Users/<yourname>/Documents/LocalProjects/Claude Guidelines" .claude-guidelines
```

> Replace the path with wherever you saved this folder.

### 3. Confirm `.claude-guidelines` is gitignored

Check `.gitignore` in the repo root — it should already contain:
```
.claude-guidelines
CLAUDE.md
.claude/
```

If not, add them.

### 4. Copy skills into `.claude/commands/`

```bash
cp -r .claude-guidelines/skills/. .claude/commands/
```

These are your slash commands (`/code-standards-check`, `/unit-tester`, etc.).
Re-run this whenever the skills folder is updated.

### 5. Add `CLAUDE.md` to the repo root (first time only)

The root `CLAUDE.md` (in this folder) should be copied to the repo root:
```bash
cp .claude-guidelines/CLAUDE.md /path/to/circle-v2-master4-deploy/CLAUDE.md
```

It's gitignored — it stays local to your machine only.

### 6. Add to VSCode Workspace

Open VSCode → **File → Add Folder to Workspace** → select your Claude Guidelines folder.

This keeps it visible and editable alongside the repo without being part of it.

---

## Folder Structure

```
Claude Guidelines/
├── CLAUDE.md                        ← Copy to repo root (gitignored)
├── SETUP.md                         ← This file
├── README.md                        ← Overview
│
├── workflow/
│   ├── issue-workflow.md            ← Full Steps 1–7 pipeline
│   ├── pr-conventions.md            ← Branch naming, PR rules, merge gate
│   └── deployment-pipeline.md      ← Alpha → Beta → Prod
│
├── standards/
│   ├── code-standards-backend.md   ← Rails: Commands, Services, Models, Serializers
│   ├── code-standards-frontend.md  ← Vue3, Hotwire, Stimulus
│   └── testing-standards.md        ← RSpec, VCR, Cypress, CI
│
├── architecture/
│   ├── overview.md                  ← Project stack summary
│   ├── backend-patterns.md         ← Command/Service/Entity/Query patterns
│   └── frontend-patterns.md        ← Turborepo layout, API packages, auth
│
└── skills/                          ← Slash commands → copy to .claude/commands/
    ├── code-standards-check.md
    ├── unit-tester.md
    ├── regression-tester.md
    ├── ux-tester.md
    ├── uat-helper.md
    └── pr-helper.md
```

---

## Keeping It Updated

- Pull updates from the shared source (Google Drive / Notion / shared repo — TBD)
- Re-run Step 4 after any skills updates
- Changes to `CLAUDE.md`, standards, or workflow files take effect immediately on the next Claude Code session
