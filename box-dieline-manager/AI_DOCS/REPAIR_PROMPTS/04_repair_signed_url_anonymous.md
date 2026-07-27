# Repair Prompt — Anonymous Signed URL Flow Undefined

## Purpose
Define the secure flow by which an unauthenticated user (who possesses a valid share token) obtains a time-limited Signed URL to download a file — without exposing the Service Role key or bypassing token validation.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 4 (High).

Current docs require (Rule 8, ADR-016):
- All Storage buckets must be Private.
- File access via time-limited Signed URLs only.

But no document specifies how an anonymous user with a share token:
1. Proves they have a valid token (authentication step).
2. Obtains a Signed URL for a specific file (authorization step).
3. Is prevented from obtaining Signed URLs for files outside the shared template.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rules 8, 14)
- `docs/09_DECISIONS.md` (ADR-010, ADR-016)
- `docs/05_TECH_SPEC.md`
- After fix: `01_repair_rls_public_templates.md` and `03_repair_rls_public_files.md`

## Required User Decision, If Any
None. The approach (server-side API route that validates token then calls Supabase Admin client to sign URL) is a standard pattern.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not add application code for this flow — only document the architecture.
- Do not expose the Service Role key in documentation.
- Do not modify `docs/01_RULES.md` or `docs/09_DECISIONS.md`.

## Required Changes
1. In `docs/06_DATA_SCHEMA.md`, add a section describing the public file download flow:
   - Public user provides share token + file ID.
   - Server validates share token against `templates` table (using RLS or direct query with Service Role).
   - Server verifies the file belongs to that template.
   - Server generates a Signed URL (short TTL, e.g. 5 minutes) using Supabase Admin client.
   - Signed URL is returned; client downloads directly from Storage.
2. In `docs/05_TECH_SPEC.md`, add an API endpoint description for `GET /api/share/[token]/files/[fileId]/download`.
3. Document that the endpoint must use Supabase Service Role (not anon key) to generate Signed URLs, and must never expose the Service Role key to the client.

## Compatibility Requirements
- ADR-016: Signed URLs at request time.
- ADR-010: Public share links are read-only.
- Must work after the RLS fixes in Repair Prompts 01 and 03.

## Verification Boundaries
- **Static verification**: Flow description can be reviewed for security correctness.
- **Requires runtime verification**: End-to-end test with real Supabase instance.

## Acceptance Criteria
- [ ] The secure flow for anonymous file download is fully documented.
- [ ] The flow validates the share token before generating any Signed URL.
- [ ] The flow ensures the file belongs to the token's template.
- [ ] No Service Role key or permanent URL is exposed to the client.
- [ ] The Signed URL has a short, documented TTL.

## Required Final Report
Standard Implementation Mode report.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
