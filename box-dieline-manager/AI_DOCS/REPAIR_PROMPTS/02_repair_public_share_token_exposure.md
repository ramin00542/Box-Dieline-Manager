# Repair Prompt — Public Share Token Exposure via SELECT *

# Repair Prompt — Public Share Token Exposure via SELECT *

## Purpose
Ensure `public_share_token` is never returned in public API responses. Per ADR-017, the token is validated server-side by a Next.js API Route — the API Route selects only non-sensitive fields, so the token is automatically excluded from responses.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 2 (Critical), `docs/11_STRUCTURAL_AND_GOVERNANCE_AUDIT.md` — Item 2.

The current RLS policy allows `SELECT *` on `templates` rows that have a share token. Without the ADR-017 architecture, the token would be exposed. ADR-017 resolves this by routing all public access through an API Route that selects only safe columns.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (templates table definition, existing RLS policies)
- `docs/09_DECISIONS.md` (ADR-010, ADR-014, ADR-016, ADR-017)

## Required User Decision, If Any
None. Architecture locked per ADR-017: API Route selects only non-sensitive fields.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not remove the `public_share_token` column from the schema (it is required by ADR-010).
- Do not modify application code or other docs files.
- Do not modify `CURRENT_TASK.md` or task files.

## Required Changes
1. In `docs/06_DATA_SCHEMA.md`, add a note that `public_share_token` is NEVER returned in any API response. The API Route (`GET /api/share/[token]`) selects only non-sensitive fields per ADR-017.

2. Add a cross-reference to ADR-017 for the column-filtering mechanism.

3. Add a note that `public_share_token` must be treated as sensitive at the database level (similar to a password hash or API key) — never logged, never exposed in error messages.

## Compatibility Requirements
- ADR-017: API Route controls column selection.
- ADR-010: Token is input-only, never output.
- The public RLS policy on `templates` is removed (per Repair Prompt 01) — API Route replaces it.

## Verification Boundaries
- **Static verification**: Schema comments and ADR-017 reference can be reviewed.
- **Requires runtime verification**: Test public endpoint response to confirm token column is absent.

## Acceptance Criteria
- [ ] A public endpoint returns template data without the `public_share_token` column.
- [ ] The token remains usable as an input parameter for the share-link flow.
- [ ] ADR-017 is referenced as the locking mechanism.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
