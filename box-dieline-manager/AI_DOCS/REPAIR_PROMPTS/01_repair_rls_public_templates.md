# Repair Prompt — RLS Public Templates Not Token-Scoped

# Repair Prompt — RLS Public Templates Not Token-Scoped

## Purpose
Remove the insecure public RLS policy on `templates` and document that ALL public access goes through a Next.js API Route with Supabase Service Role (per ADR-017).

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 1 (Critical), `docs/11_STRUCTURAL_AND_GOVERNANCE_AUDIT.md` — Item 1.

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
This policy is fundamentally insecure — it allows any unauthenticated user to see ALL templates with active share tokens, not just the one whose token they possess. RLS alone cannot securely validate a client-supplied token.

## Mandatory Reading
Before modifying, read the current contents of:
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (especially Rules 8, 13)
- `docs/09_DECISIONS.md` (especially ADR-010, ADR-014, ADR-017)

## Required User Decision, If Any
None. Architecture locked per ADR-017.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do NOT modify `docs/01_RULES.md`, `docs/09_DECISIONS.md`, or `docs/05_TECH_SPEC.md`.
- Do NOT create or modify any application code.
- Do NOT modify `CURRENT_TASK.md` or any task files.
- Do NOT remove the public share link feature — the goal is to secure it, not remove it.

## Required Changes
1. **Remove** the `"Public can view via share token"` RLS policy from the `templates` table entirely. Public access no longer uses RLS — it uses the API Route with Service Role (ADR-017).

2. Add a comment in the schema that public access is handled by `GET /api/share/[token]` (Next.js API Route with Service Role), not by RLS.

3. Add a cross-reference in the schema to ADR-017 for the locked architecture.

## Compatibility Requirements
- ADR-017 (locked): All public access via API Route with Service Role.
- ADR-010 (token-based links): Token validation happens in the API Route.
- The `public_share_token`, `share_expires_at` columns must remain in schema.
- Authenticated RLS policies remain unchanged.

## Verification Boundaries
- **Static verification**: Policy removal and comments can be reviewed.
- **Requires runtime verification**: The `/api/share/[token]` endpoint must be tested against real Supabase.

## Acceptance Criteria
- [ ] The insecure public RLS policy is removed.
- [ ] Schema documents the replacement: API Route with Service Role.
- [ ] ADR-017 is referenced.
- [ ] Authenticated RLS policies remain unchanged.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
