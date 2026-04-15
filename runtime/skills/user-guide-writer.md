# User Guide Writer

Generate or update user-facing documentation for the feature changed in this branch.

## Steps

1. **Understand the change** — read `git diff origin/master4...HEAD` and the changed files to understand:
   - What new functionality was added?
   - What existing behavior changed?
   - Who is the audience? (backoffice staff, POS operator, admin, end customer)

2. **Ask the user** if not clear:
   - "Is this for internal staff (backoffice) or end customers (POS/web)?"
   - "Is there an existing doc to update, or is this new?"

3. **Draft the documentation** in plain, non-technical language:

   **For a new feature:**
   ```markdown
   ## [Feature Name]

   ### What it does
   [1-2 sentences describing the feature from the user's perspective]

   ### How to use it
   1. Navigate to [section]
   2. Click [button/link]
   3. Fill in [fields] — [what each field means]
   4. Click [Save/Submit]

   ### Notes
   - [Any constraints, e.g. "only available to Admin users"]
   - [Any gotchas, e.g. "changes take effect after the next sync"]
   ```

   **For a changed feature:**
   ```markdown
   ## [Feature Name] — Updated

   ### What changed
   [Before]: [old behavior]
   [After]: [new behavior]

   ### Why it changed
   [Brief reason if applicable]
   ```

4. **For API changes** (if relevant for integration partners), also draft:
   - Endpoint, method, required headers
   - Request body shape (with example)
   - Response shape (with example)
   - New/changed error codes

5. **Output location** — ask the user where to save it:
   - Internal only: suggest `docs/` folder in the repo
   - External: flag that it needs to go through the documentation process

6. **Review the draft** with the user before saving — ask "Does this accurately describe what a user will experience?"

## Notes

- Write for the audience, not the implementation — no Ruby class names, no database terms
- Use active voice: "Click Save" not "The Save button should be clicked"
- Screenshots/GIFs are out of scope — note where they should be added with `[SCREENSHOT: ...]`
