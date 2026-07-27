# Repair Prompt — Phase 03: Authentication

## Purpose
Write the three Phase 3 task files (`task_03_01` through `task_03_03`) with complete, actionable content. Incorporate the single-admin model requirement and the RLS enforcement decisions from the security audit.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓. Single-admin model is locked in ADR-005 and ADR-011.

**B (Dependency order):** Phase 3 depends on Phase 2 (profiles table must exist). Correctly positioned after Phase 2 in ADR-012 ✓.

**C (Next.js/TS Correctness):** No code in Phase 3 tasks yet (placeholders). Must ensure correct Supabase Auth client usage patterns are documented.

**D (Database/API):** Key audit findings:
- **Item 5** (RLS single-admin): The RLS policies must enforce single-user access, not just `auth.uid() IS NOT NULL`.
- **Item 6** (Profile bootstrap): The auth signup/login flow must create the `profiles` record.

**E (Schema consistency):** Auth tasks reference `profiles` table created in Phase 2. Must ensure foreign key consistency.

**F (Tests):** Auth flow tests are critical. Phase 3 tasks should document test strategy (Supabase local emulator or mock).

**G (Cross-task):** Phase 3 is prerequisite for Phase 4 (protected routes, template CRUD).

**H (UI/UX):** Login page (task_03_02) must use shadcn/ui components per ADR-006.

**I (Security):** Authentication is the security foundation. Must document:
- Environment variables for Supabase Auth.
- Session management via cookies.
- Password policies (or defer to Supabase Auth defaults).
- No hardcoded credentials.

**J (Performance):** No performance concerns expected for auth ✓.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (profiles table, RLS policies)
- `docs/01_RULES.md` (Rules 5, 8)
- `docs/09_DECISIONS.md` (ADR-005, ADR-011, ADR-014)
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 5, 6)
- Related per-item repair prompts: `05_repair_rls_single_admin.md`, `06_repair_profile_bootstrap.md`

## Required User Decision, If Any
None. Single-admin model is locked.

## Allowed Files
- `AI_DOCS/PARTS/phase_03_auth/task_03_01_supabase_auth_setup.md`
- `AI_DOCS/PARTS/phase_03_auth/task_03_02_admin_login_page.md`
- `AI_DOCS/PARTS/phase_03_auth/task_03_03_protected_routes.md`

## Forbidden Actions
- Do NOT implement actual auth code.
- Do NOT modify Phase 2 or Phase 4 task files.
- Do NOT modify governance docs.

## Required Changes
1. Write `task_03_01_supabase_auth_setup.md`: Configure Supabase Auth in Next.js (create Supabase client for browser + server). Document environment variables (NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY). Create `lib/supabase/client.ts` and `lib/supabase/server.ts`. Document session management.
2. Write `task_03_02_admin_login_page.md`: Create login page with email/password form using shadcn/ui components. Document error handling (invalid credentials, network errors). No sign-up page (single admin only — admin is created manually in Supabase dashboard).
3. Write `task_03_03_protected_routes.md`: Implement middleware or layout-level auth check. Redirect unauthenticated users to login. Reference Rule 8 (auth required for write operations). Document that all `/api/*` routes (except public share endpoints) must verify auth.

## Compatibility Requirements
- ADR-005: Supabase Auth, email/password only.
- ADR-011: Single admin user.
- Must work with RLS policies from Phase 2.
- Login page must match `05_TECH_SPEC.md` project structure (`app/(auth)/`).

## Acceptance Criteria
- [ ] All 3 Phase 3 task files contain complete, actionable instructions.
- [ ] Auth setup references correct environment variables.
- [ ] Login page uses shadcn/ui components.
- [ ] Protected routes document middleware or layout pattern.
- [ ] Single-admin constraint is enforced at the auth level.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
