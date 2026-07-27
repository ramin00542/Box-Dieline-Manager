# Repair Prompt — RLS Public Files Not Token-Scoped

## Purpose
Fix the RLS policy `"Public can view files via template share token"` on the `files` table so that a public request can only see files belonging to the **specific template** whose token they provide — not all files of any template that has an active share token.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 3 (Critical).

The current policy in `docs/06_DATA_SCHEMA.md`:
```sql
CREATE POLICY "Public can view files via template share token"
  ON files FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM templates
      WHERE templates.id = files.template_id
      AND templates.public_share_token IS NOT NULL
      AND templates.share_expires_at > NOW()
      AND templates.deleted_at IS NULL
    )
  );
```
This policy checks whether the **parent template** has any share token — it does not verify that the caller provided **that specific token**. An unauthenticated query can see all files of all shared templates.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (files table definition, RLS policies)
- `docs/09_DECISIONS.md` (ADR-010, ADR-014, ADR-016)

## Required User Decision, If Any
None. The fix follows the same pattern as Repair Prompt 01.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify `docs/01_RULES.md`, `docs/09_DECISIONS.md`, or `docs/05_TECH_SPEC.md`.
- Do not create or modify application code.
- Do not remove the public share feature.

## Required Changes
1. Replace the existing `"Public can view files via template share token"` policy with one that requires the caller to supply the specific template's share token.
2. The mechanism must match whatever is chosen for Repair Prompt 01 (RPC function, `current_setting`, or application-layer approach).
3. Ensure the policy still enforces `deleted_at IS NULL` on the parent template and `share_expires_at > NOW()`.

## Compatibility Requirements
- Must be consistent with the fix in Repair Prompt 01 (same token-passing mechanism).
- Must remain compatible with ADR-010 and ADR-016.
- Files are append-only — no update or delete policies needed for public access.

## Verification Boundaries
- **Static verification**: Policy SQL can be reviewed.
- **Requires runtime verification**: Test with unauthenticated client against a real Supabase instance.

## Acceptance Criteria
- [ ] An unauthenticated request without a token cannot see any files.
- [ ] An unauthenticated request with a valid token sees only files belonging to that specific template.
- [ ] `deleted_at` and `share_expires_at` constraints are still enforced via the parent template.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
