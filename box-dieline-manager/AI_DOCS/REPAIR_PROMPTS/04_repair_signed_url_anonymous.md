# Repair Prompt — Anonymous Signed URL Flow Undefined

## Purpose
Document the locked architecture (per ADR-017) for secure file download by unauthenticated users who possess a valid share token — using a Next.js API Route with Supabase Service Role to validate the token, verify file ownership, and generate a short-lived Signed URL.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 4 (High), `docs/11_STRUCTURAL_AND_GOVERNANCE_AUDIT.md` — Item 4.

Architecture locked per ADR-017:
- All public access via Next.js API Route with Service Role.
- Token validation, file ownership check, and Signed URL generation happen server-side.
- Public RLS policies removed (Items 1, 3).
- Signed URL TTL: 5 minutes.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rules 8, 14)
- `docs/09_DECISIONS.md` (ADR-010, ADR-016, ADR-017)
- `docs/05_TECH_SPEC.md`

## Required User Decision, If Any
None. Architecture locked per ADR-017.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not add application code for this flow — only document the architecture.
- Do not expose the Service Role key in documentation.
- Do not modify `docs/01_RULES.md` or `docs/09_DECISIONS.md`.

## Required Changes
1. In `docs/06_DATA_SCHEMA.md`, document the locked public file download flow:
   - `GET /api/share/[token]/files/[fileId]/download` (Next.js API Route, Service Role).
   - Validates share token against `templates.public_share_token`, checks expiry and deleted_at.
   - Verifies `fileId` belongs to the template matching the token.
   - Generates 5-minute Signed URL via `supabase.storage.createSignedUrl()`.
   - Returns the Signed URL; client downloads directly from Storage.
   - Reference ADR-017 for the locked architecture.

2. In `docs/05_TECH_SPEC.md`, add a section describing:
   - `GET /api/share/[token]` — returns template metadata (no token, no sensitive fields).
   - `GET /api/share/[token]/files/[fileId]/download` — returns Signed URL.
   - Service Role usage (server-side only, never exposed to client).
   - Rate limiting per Item 24.

3. Document that the public RLS policies are removed (per Repair Prompts 01 and 03) — all public access is through these API Routes.

## Compatibility Requirements
- ADR-017 (locked): Authoritative architecture for public access.
- ADR-016: Signed URLs at request time, never permanent.
- ADR-010: Token-based, read-only, 7-day expiry.
- Must work after RLS policy removals (Items 1, 3).

## Verification Boundaries
- **Static verification**: Flow documentation can be reviewed for security correctness.
- **Requires runtime verification**: End-to-end test with real Supabase instance.

## Acceptance Criteria
- [ ] The locked secure flow (per ADR-017) is fully documented in both schema and tech spec.
- [ ] Token validation happens before any Signed URL generation.
- [ ] File ownership is verified (fileId belongs to token's template).
- [ ] No Service Role key is exposed to the client.
- [ ] Signed URL TTL is documented as 5 minutes.

## Required Final Report
Standard Implementation Mode report.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
