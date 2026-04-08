# Claude Guidelines — Circle V2

Coding Agent configuration for the Circle V2 codebase.
Shared across the dev and support teams. Lives outside the repo.

## What's in here

| Folder / File      | Purpose                                                  |
|--------------------|----------------------------------------------------------|
| `CLAUDE.md`        | Root agent config — copy to repo root (gitignored)       |
| `SETUP.md`         | How to set up your local workspace                       |
| `workflow/`        | Issue pipeline, PR rules, deployment flow                |
| `standards/`       | Code standards: backend, frontend, testing               |
| `architecture/`    | Project architecture and patterns reference              |
| `skills/`          | Slash commands for Claude Code sub-agents                |

## Quick Start

See `SETUP.md` for full instructions.

## Philosophy

- `CLAUDE.md` in the repo root stays **lean** — it's just an index and hard rules
- All detail lives here, loaded on demand via `@.claude-guidelines/...` references
- Sub-agents are triggered by slash commands defined in `skills/`
- Never put linting rules or style guides in CLAUDE.md — use the linter
