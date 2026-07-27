# Repair Prompt — RLS Public Files Not Token-Scoped

## Purpose
Remove the insecure public RLS policy on `files` and document that ALL public file access goes through a Next.js API Route with Supabase Service Role (per ADR-017), which validates the token and verifies file-to-template ownership before generating a Signed URL.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 3 (Critical), `docs/11_STRUCTURAL_AND_GOVERNANCE_AUDIT.md` — Item 3.

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
This policy is fundamentally insecure — it allows any unauthenticated user to see ALL files belonging to any template with an active share token, not just files of the specific token they possess.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (files table definition, RLS policies)
- `docs/09_DECISIONS.md` (ADR-010, ADR-014, ADR-016, ADR-017)

## Required User Decision, If Any
None. Architecture locked per ADR-017.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify `docs/01_RULES.md`, `docs/09_DECISIONS.md`, or `docs/05_TECH_SPEC.md`.
- Do not create or modify application code.
- Do not remove the public share feature.

## Required Changes
1. **Remove** the `"Public can view files via template share token"` RLS policy from the `files` table entirely. Public file access no longer uses RLS — it uses the API Route with Service Role (ADR-017).

2. Add a comment in the schema that public file access is handled by `GET /api/share/[token]/files/[fileId]/download` which validates the token, verifies file-to-template ownership, and generates a 5-minute Signed URL.

3. Add a cross-reference to ADR-017.

## Compatibility Requirements
- ADR-017: All public access via API Route with Service Role.
- ADR-016: Signed URLs generated at request time, never permanent.
- ADR-010: Token-based links, read-only.
- Authenticated RLS policies remain unchanged.

## Verification Boundaries
- **Static verification**: Policy removal and comments can be reviewed.
- **Requires runtime verification**: End-to-end test with real Supabase.

## Acceptance Criteria
- [ ] The insecure public RLS policy on files is removed.
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
