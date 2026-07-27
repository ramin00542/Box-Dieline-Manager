# Repair Prompt — Admin Profile Bootstrap Undefined

## Purpose
Document how the admin user's `profiles` record is created during initial setup. Currently no `INSERT` policy or trigger exists for `profiles`, and foreign keys from `templates` and `files` reference `profiles(id)`, which means template/file creation will fail if no profile exists.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 6 (High).

Current `profiles` table has `SELECT` and `UPDATE` policies but no `INSERT` policy and no trigger on `auth.users` to auto-create profiles. The `created_by` and `uploaded_by` columns in `templates` and `files` reference `profiles(id)`.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (profiles table, RLS policies, foreign key constraints)
- `docs/01_RULES.md` (Rule 13: profile data in `profiles` table)
- `docs/09_DECISIONS.md` (ADR-008, ADR-011)

## Required User Decision, If Any
None. The approach (trigger-based or seed-script) can be determined during implementation.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not create application code.
- Do not modify other docs files.

## Required Changes
1. Add an `INSERT` policy for `profiles`:
   - Option A: A trigger `CREATE OR REPLACE FUNCTION public.handle_new_user()` that runs `ON AFTER INSERT ON auth.users` and creates the profile row automatically.
   - Option B: Document that the seed/migration script must insert the profile row explicitly.
2. Document the preferred approach with full SQL.

## Compatibility Requirements
- ADR-008: Use `profiles` table.
- The profile must have `id = auth.uid()` to match the foreign key relationship.

## Acceptance Criteria
- [ ] A method exists to create the admin's `profiles` row.
- [ ] The method is documented and reproducible.
- [ ] Template and file creation will not fail due to missing profile.

## Required Final Report
Standard Implementation Mode report.
