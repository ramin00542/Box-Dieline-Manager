# Repair Prompt — Phase 02: Database & Schema

## Purpose
Write the four Phase 2 task files (`task_02_01` through `task_02_04`) with complete, actionable content. The task files are currently empty placeholders. Several schema-level issues identified by the security audit must be incorporated into the task instructions.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓. Schema from `06_DATA_SCHEMA.md` is authoritative.

**B (Dependency order):** Phase 2 depends on Phase 1 (project setup must create Supabase project). Correctly positioned after Phase 1 in ADR-012 ✓.

**C (Next.js/TS Correctness):** No code in Phase 2 tasks (schema-only) ✓.

**D (Database/API):** The following schema issues from `docs/10_EXTERNAL_SECURITY_AUDIT.md` affect Phase 2 tasks:
- **Item 5** (RLS single-admin): Database RLS policies use `auth.uid() IS NOT NULL` which allows any authenticated user. Phase 2 must address single-admin enforcement.
- **Item 6** (Profile bootstrap): No `INSERT` policy or trigger for `profiles` table. Phase 2 must define a mechanism.
- **Item 7** (Profile email sync): `profiles.email` is updatable without sync to `auth.users.email`. Must be addressed.
- **Item 11** (`updated_at` trigger): No auto-update trigger exists. Must be added.
- **Item 16** (`template_id` NOT NULL): User decided `template_id` must be `NOT NULL`. Must be added.
- **Item 20** (`EXCEPTION WHEN OTHERS`): Overly broad. Must be narrowed.

**E (Schema consistency):** `06_DATA_SCHEMA.md` is the schema authority. Phase 2 tasks must reference it faithfully.

**F (Tests):** No tests in Phase 2 tasks (schema-only). Rule 7 exemption implied for database migrations.

**G (Cross-task):** Phase 2 is prerequisite for Phases 3 (auth), 4 (CRUD), and all later phases.

**H (UI/UX):** No UI content ✓.

**I (Security):** RLS policies are the primary security mechanism. See Items 1–5 from 10_EXTERNAL_SECURITY_AUDIT.md.

**J (Performance):** Database indexes are already specified in `06_DATA_SCHEMA.md` ✓.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rules 8, 13, 14)
- `docs/09_DECISIONS.md` (ADR-003, ADR-007, ADR-008, ADR-013, ADR-014, ADR-016)
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 5, 6, 7, 11, 16, 20)
- Related per-item repair prompts: `05`, `06`, `07`, `11`, `16`, `20`

## Required User Decision, If Any
None. All schema-level decisions have been resolved in the per-item repair prompts.

## Allowed Files
- `AI_DOCS/PARTS/phase_02_database/task_02_01_supabase_setup.md`
- `AI_DOCS/PARTS/phase_02_database/task_02_02_profiles_table.md`
- `AI_DOCS/PARTS/phase_02_database/task_02_03_templates_table.md`
- `AI_DOCS/PARTS/phase_02_database/task_02_04_files_table.md`

## Forbidden Actions
- Do NOT create actual database resources (tables, triggers, policies).
- Do NOT modify `docs/06_DATA_SCHEMA.md` in this repair.
- Do NOT modify task files outside Phase 2.

## Required Changes
1. Write `task_02_01_supabase_setup.md`: Create Supabase project, record project URL and anon key, set up `supabase/config.toml`, install `@supabase/supabase-js`.
2. Write `task_02_02_profiles_table.md`: Include the `profiles` table DDL from `06_DATA_SCHEMA.md`. Add the auto-profile-creation trigger (see Item 6 from audit). Add INSERT policy. Document the email-sync decision (see Item 7). Add `update_updated_at_column()` trigger.
3. Write `task_02_03_templates_table.md`: Include the `templates` table DDL. Add `unit` column (user decision from Item 14). Add `update_updated_at_column()` trigger. Update the search vector to include `description` and `tags` (user decision from Item 19). Narrow `EXCEPTION WHEN OTHERS` to `SQLSTATE '42704'` (Item 20). Document the soft-delete RLS filter (`deleted_at IS NULL`).
4. Write `task_02_04_files_table.md`: Include the `files` table DDL. Add `NOT NULL` to `template_id` (Item 16). Add Storage bucket configuration (Item 17). Document that files table is append-only (no `updated_at` per Rule 13 exception). Clarify `storage_url` is cached, not authoritative (Item 12).

## Compatibility Requirements
- All ADRs (003, 007, 008, 013, 014, 016) must be referenced.
- Schema must exactly match `06_DATA_SCHEMA.md` (with the fixes applied).
- RLS enabled on all tables per ADR-014.

## Verification Boundaries
- **Static verification**: DDL syntax correctness, trigger logic, RLS policy logic.
- **Requires runtime verification**: RLS policy behavior must be tested against a real Supabase instance.

## Acceptance Criteria
- [ ] All 4 Phase 2 task files contain complete DDL and documentation.
- [ ] Profile table includes auto-creation mechanism.
- [ ] `updated_at` trigger is documented for both `profiles` and `templates`.
- [ ] `EXCEPTION WHEN OTHERS` is narrowed.
- [ ] `template_id` is `NOT NULL` in `files`.
- [ ] Search vector includes `description` and `tags`.
- [ ] `unit` column is added to `templates`.
- [ ] Storage configuration is documented.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
