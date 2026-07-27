# Repair Prompt — Phase 06: Public Share Links

## Purpose
Write the three Phase 6 task files (`task_06_01` through `task_06_03`) with complete, actionable content. Incorporate all Critical RLS fixes for public share links (Items 1–4 from the security audit) — token-scoped policies, token column protection, secure Signed URL flow for anonymous users.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓.

**B (Dependency order):** Phase 6 depends on Phase 4 (templates with data) and Phase 5 (search to find templates to share). Correctly positioned after Phase 5 in ADR-012 ✓.

**C (Next.js/TS Correctness):** Public view page must be a Server Component or Client Component with correct data fetching. Must not expose Supabase Service Role key to client.

**D (Database/API):** This phase is directly affected by Critical audit items:
- **Item 1** (RLS public templates): Token-scoped policy required.
- **Item 2** (Token exposure): `public_share_token` must not be exposed in API responses.
- **Item 3** (RLS public files): Token-scoped file policy required.
- **Item 4** (Signed URL for anonymous): Secure flow must be defined and implemented.

Additionally: Rate limiting on public endpoints (Item 24) and pagination for shared template data (Item 23).

**E (Schema consistency):** `public_share_token` and `share_expires_at` columns already in schema.

**F (Tests):** Public share link flow is critical and must be tested end-to-end (token generation → public access → file download → expiration).

**G (Cross-task):** Public file download uses Signed URL flow defined in Item 4. Token validation must use the RLS/API patterns from Items 1-3.

**H (UI/UX):** Public view page must be clean, read-only, with mobile responsiveness. No admin UI elements.

**I (Security):** This is the most security-sensitive phase. All Critical items (1-4) must be implemented correctly. Rate limiting (Item 24) must be applied.

**J (Performance):** Signed URL generation is fast. No caching concerns for 7-day links.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (RLS policies for public access)
- `docs/01_RULES.md` (Rules 8, 14)
- `docs/09_DECISIONS.md` (ADR-010, ADR-016)
- `docs/05_TECH_SPEC.md`
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 1, 2, 3, 4, 24)
- Related per-item repair prompts: `01`, `02`, `03`, `04`, `24`

## Required User Decision, If Any
None. All share-link decisions resolved in per-item prompts.

## Allowed Files
- `AI_DOCS/PARTS/phase_06_share_links/task_06_01_share_token_generation.md`
- `AI_DOCS/PARTS/phase_06_share_links/task_06_02_public_view_page.md`
- `AI_DOCS/PARTS/phase_06_share_links/task_06_03_link_expiration.md`

## Forbidden Actions
- Do NOT implement actual code.
- Do NOT modify task files from other phases.
- Do NOT modify governance docs.

## Required Changes
1. Write `task_06_01_share_token_generation.md`: Server-side API route (`POST /api/templates/[id]/share`) generates a unique token via `gen_random_uuid()`, sets `public_share_token` and `share_expires_at` (7 days from now). Auth required (only admin can generate links). Token must be a cryptographically random string. Store token validation logic for use in public endpoints.
2. Write `task_06_02_public_view_page.md`: Public view page (`/share/[token]`) that:
   - Uses Secure Signed URL flow (Item 4): server validates token, generates short-lived Signed URLs for files.
   - Displays template read-only (name, dimensions, box type, material).
   - Provides download buttons for each file (Signed URL).
   - RLS must prevent accessing other templates' data (Items 1, 3).
   - `public_share_token` must not appear in API responses (Item 2).
   - Rate limiting applied (Item 24).
3. Write `task_06_03_link_expiration.md`: Document and implement expiration logic:
   - `share_expires_at` checked in RLS policies (already done).
   - Expired links show "This link has expired" message.
   - Admin can revoke links early (set `share_expires_at = NOW()`).
   - Cleanup/renewal flow for expired links.

## Compatibility Requirements
- ADR-010: Token-based, 7-day expiration, read-only.
- ADR-016: Private buckets, Signed URLs only.
- Rule 8: Public share links read-only, expire after 7 days.
- All Critical RLS fixes (Items 1-4) must be reflected.

## Acceptance Criteria
- [ ] All 3 Phase 6 task files contain complete, actionable instructions.
- [ ] Token generation uses `gen_random_uuid()` or equivalent.
- [ ] Public view page validates token before showing data.
- [ ] Signed URL flow is documented and secure.
- [ ] Expiration logic is documented (both automatic and manual revoke).

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
