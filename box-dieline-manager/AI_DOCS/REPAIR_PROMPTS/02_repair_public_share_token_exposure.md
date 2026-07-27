# Repair Prompt — Public Share Token Exposure via SELECT *

## Purpose
Prevent the `public_share_token` column from being returned in public API responses. The token must act as a credential — it authenticates the public request — and should not be readable by callers who have not provided it.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 2 (Critical).

The current RLS policy allows `SELECT *` on `templates` rows that have a share token. The `public_share_token` column would be included in the response unless explicitly excluded by a View, an RPC function, or application-layer filtering. No such mechanism is documented.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (templates table definition, existing RLS policies)
- `docs/09_DECISIONS.md` (ADR-010, ADR-014, ADR-016)

## Required User Decision, If Any
None. The approach must be chosen (View vs RPC vs application filtering), which can be decided during implementation.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not remove the `public_share_token` column from the schema (it is required by ADR-010).
- Do not modify application code or other docs files.
- Do not modify `CURRENT_TASK.md` or task files.

## Required Changes
1. Document a mechanism that ensures `public_share_token` is never returned in API responses that serve public template data. Options include:
   - A Postgres View that excludes the token column
   - An RPC function that accepts the token as a parameter and returns only safe columns
   - Application-layer column filtering
2. If choosing the View/RPC approach, define the View or RPC signature in `docs/06_DATA_SCHEMA.md`.
3. Add a note that `public_share_token` must be treated as sensitive (similar to a password hash or API key).

## Compatibility Requirements
- ADR-010: Token-based links must still work — the token is an input, not output.
- The RLS policy from Repair Prompt 01 must be in place first.
- Index `idx_templates_share_token` must remain usable.

## Verification Boundaries
- **Static verification**: Review whether the chosen mechanism actually excludes the column.
- **Requires runtime verification**: Test an unauthenticated request and confirm the token column is absent from the response.

## Acceptance Criteria
- [ ] A public endpoint returns template data without the `public_share_token` column.
- [ ] The token remains usable as an input parameter for the share-link flow.
- [ ] No existing functionality (authenticated views, CRUD) is affected.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
