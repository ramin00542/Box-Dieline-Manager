# Repair Prompt — RLS Public Templates Not Token-Scoped

## Purpose
Fix the RLS policy `"Public can view via share token"` on the `templates` table so that a public (unauthenticated) request can only see the **single template** whose `public_share_token` matches the token the caller presents — not all templates that have any active share token.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 1 (Critical).

The current policy in `docs/06_DATA_SCHEMA.md`:
```sql
CREATE POLICY "Public can view via share token"
  ON templates FOR SELECT
  USING (
    public_share_token IS NOT NULL 
    AND share_expires_at > NOW()
    AND deleted_at IS NULL
  );
```
This policy checks only whether a row **has** a token — it does not verify that the caller provided **that specific token**. An unauthenticated query to the `templates` table can read every template that has an active share token.

Audit finding: *"این policy فقط بررسی می‌کند که قالب دارای یک توکن اشتراک فعال باشد؛ اما بررسی نمی‌کند که کاربر ناشناس، همان توکن مربوط به ردیف را در URL یا درخواست ارائه کرده باشد."*

## Mandatory Reading
Before modifying, read the current contents of:
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (especially Rules 8, 13)
- `docs/09_DECISIONS.md` (especially ADR-010, ADR-014)

## Required User Decision, If Any
None. The fix is a security requirement with a single correct approach.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify `docs/01_RULES.md`, `docs/09_DECISIONS.md`, or `docs/05_TECH_SPEC.md`.
- Do not create or modify any application code.
- Do not modify `CURRENT_TASK.md` or any task files.
- Do not remove the public share link feature — the goal is to secure it, not remove it.

## Required Changes
1. Replace the existing `"Public can view via share token"` policy with a version that requires the caller to supply the specific token. The mechanism (parameterised RPC function, `current_setting`, or application-level middleware) must be chosen and documented.

2. Add a note that this policy alone is insufficient — the application layer must also validate the token before generating Signed URLs for files.

## Compatibility Requirements
- Must remain compatible with ADR-010 (token-based public links with 7-day expiration).
- Must remain compatible with ADR-014 (RLS on all tables).
- The `public_share_token` column and `share_expires_at` column must remain in the schema.
- The policy must still enforce `deleted_at IS NULL` and `share_expires_at > NOW()`.

## Verification Boundaries
- **Static verification**: Policy SQL syntax, referenced columns, and logic can be reviewed statically.
- **Requires runtime verification**: The policy must be tested against a real Supabase instance with at least two templates (one with share token, one without) and an unauthenticated client.

## Acceptance Criteria
- [ ] The revised policy prevents an unauthenticated `SELECT` on `templates` from returning rows unless the caller supplies the matching token.
- [ ] An unauthenticated request with a valid token can still see the corresponding template.
- [ ] `deleted_at`, `share_expires_at`, and `public_share_token IS NOT NULL` are still enforced.
- [ ] ADR-010 is not violated.
- [ ] The `idx_templates_share_token` index can still be used for lookups.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
