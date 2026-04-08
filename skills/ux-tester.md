---
name: ux-tester
description: Triggered only when UI or frontend files were changed — validates UX, accessibility, and frontend standards
---

**Only run this if UI/frontend files were changed in this branch.**

If no frontend files were changed, skip this step entirely.

Load frontend standards:
@.claude-guidelines/standards/code-standards-frontend.md

## Trigger Condition

Run `/ux-tester` if any of the following were changed:
- `app/vue3/apps/backoffice/**`
- `app/vue3/apps/pos/**`
- `app/vue3/packages/components/**`
- `app/components/**` (ViewComponent)
- `app/javascript/controllers/**` (Stimulus)
- `app/views/**`

## Steps

### 1. TypeScript type-check
```bash
# Backoffice
cd app/vue3/apps/backoffice && npx vue-tsc --noEmit

# POS
cd app/vue3/apps/pos && npx vue-tsc --noEmit
```

### 2. ESLint
```bash
cd app/vue3 && npx eslint .
```

### 3. UX review checklist

For every changed UI component or view:
- [ ] No hardcoded English strings — uses `t('validation.key')`
- [ ] Error states are handled and displayed to the user
- [ ] Loading states are shown where async operations occur
- [ ] Form validation uses Zod schema — no ad hoc validation logic
- [ ] Responsive layout not broken
- [ ] No console errors introduced
- [ ] Hotwire: ViewComponent renders correctly in `backoffice_hotwire` layout
- [ ] New pages gated behind feature flag if not yet QA-approved

### 4. Cypress smoke check (if applicable)
If the changed UI has existing Cypress tests:
```bash
cd app/vue3 && npx cypress run --spec "apps/<app>/cypress/e2e/<relevant-spec>"
```

## Output

If all checks pass:
> ✅ UX checks passed. Proceed to /uat-helper.

If checks fail:
> ❌ UX issues detected:
> - File: `<filename>`
> - Issue: `<what failed>`
> - Fix: `<what to change>`

Fix issues, re-run `/ux-tester` before proceeding.
