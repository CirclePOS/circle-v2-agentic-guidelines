---
name: code-standards-check
description: Validate all changed files against Circle V2 code standards before proceeding to testing
---

Review every file changed in this branch against Circle V2 code standards.

Load and apply:
@.claude-guidelines/standards/code-standards-backend.md
@.claude-guidelines/standards/code-standards-frontend.md

## Check each changed file for:

### Backend (Rails)
- [ ] Business logic is in Commands or Services — not in models or controllers
- [ ] Controllers only: authenticate → extract params → call command → render
- [ ] Models only: associations, scopes, validations, AASM, store accessors, ES config, display methods
- [ ] Serializers inherit from `Core::Serializers::ResourceSerializer` — no ActiveModel::Serializers or jbuilder
- [ ] Command domain layout is complete (create/update/find/search/parameters commands present)
- [ ] Custom steps use `attr_command_reader`, return `:success` or error symbol — never raise
- [ ] AASM side effects are in command steps, not callbacks
- [ ] "Tell don't ask" applied — no inline computation in views

### Frontend (Vue 3)
- [ ] `<script setup lang="ts">` on all components
- [ ] No `any`, no `@ts-ignore`
- [ ] Feature folder has `queries.ts` and `schema.ts`
- [ ] No separate interface written for form data — uses `z.infer<>`
- [ ] Pinia store in setup function style
- [ ] Form submit logic in composable, not inline in component
- [ ] API calls use `request` from `@repo/utils` — no direct axios imports
- [ ] No hardcoded English strings — uses `t('validation.key')`

### Hotwire
- [ ] New backoffice pages use Hotwire, not Vue
- [ ] ViewComponent: logic in `.rb`, display only in `.html.erb`
- [ ] New pages gated behind feature flag

## Output

If all checks pass:
> ✅ Code standards check passed. Proceed to /unit-tester.

If any check fails:
> ❌ Code standard violation detected:
> - File: `<filename>`
> - Violation: `<what rule was broken>`
> - Fix: `<what to change>`

Fix all violations, then re-run `/code-standards-check` before proceeding.
