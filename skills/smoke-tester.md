# Smoke Tester

Generate a targeted smoke test checklist for the changes in this branch. Guides manual and AI-assisted verification on the running app.

## Steps

1. **Identify what changed** — run `git diff --name-only origin/master4...HEAD` and summarize the user-facing impact:
   - Backend command changes → what API endpoints are affected?
   - Frontend changes → what screens/flows are affected?
   - Model/validation changes → what create/update flows could break?

2. **Generate a smoke test checklist** specific to the changes. Format each item as:
   ```
   [ ] <What to do> → <What to verify>
   ```
   Examples:
   - `[ ] Create a new [resource] via the UI → record appears in list, no 500 errors in Rails logs`
   - `[ ] Update an existing [resource] with invalid data → validation error shown, record not saved`
   - `[ ] Delete a [resource] → record removed, no orphaned associations`
   - `[ ] Navigate to [page] → page loads, no JS console errors`

3. **Add regression checks** — identify adjacent features that share models or services with the changed code and add a "make sure this still works" item for each.

4. **Add an API-level check** if the change is backend-only:
   ```bash
   # Example curl to verify the endpoint responds correctly
   curl -H "Authorization: Bearer <token>" -H "X-SITE-ID: <id>" \
     http://localhost:3008/api/v1/<endpoint>
   ```

5. **Check Rails logs** during testing for:
   - `5xx` errors
   - `DEPRECATION WARNING`
   - Unexpected SQL queries (N+1s)
   - Sidekiq job failures

6. **For frontend changes**, also verify:
   - No TypeScript errors in browser console
   - Network tab shows expected API calls with 2xx responses
   - Mobile (POS) behavior if the change touches POS code

7. **Report** the full checklist and ask the user to confirm each item is passing before proceeding to UAT.

## Notes

- This is a pre-UAT gate — the checklist should be completable in under 15 minutes
- If Cypress tests exist for the affected area, run them: `cd app/vue3 && npx cypress run --spec "cypress/e2e/<feature>*"`
