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
None — resolved to trigger-based approach.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not create application code.
- Do not modify other docs files.

## Required Changes
1. Add a trigger function `public.handle_new_user()` that runs `AFTER INSERT ON auth.users FOR EACH ROW`:
   ```sql
   CREATE OR REPLACE FUNCTION public.handle_new_user()
   RETURNS TRIGGER AS $$
   BEGIN
     INSERT INTO public.profiles (id, email, full_name, is_active)
     VALUES (
       NEW.id,
       NEW.email,
       COALESCE(NEW.raw_user_meta_data->>'full_name', 'Admin'),
       true
     );
     RETURN NEW;
   END;
   $$ LANGUAGE plpgsql SECURITY DEFINER;

   CREATE TRIGGER on_auth_user_created
     AFTER INSERT ON auth.users
     FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
   ```
2. This eliminates the need for a separate `INSERT` policy on `profiles` — the trigger runs as SECURITY DEFINER, bypassing RLS.
3. Document that the seed/migration script must include this trigger and function.
4. No `INSERT` policy for `profiles` is needed since the only insert is via the trigger; the `SELECT` and `UPDATE` policies suffice for the admin user.

## Compatibility Requirements
- ADR-008: Use `profiles` table, NOT `users`.
- The profile must have `id = auth.uid()` — the trigger ensures this via `NEW.id`.
- SECURITY DEFINER is required because `auth.users` is in the `auth` schema and the trigger runs as the table owner.

## Acceptance Criteria
- [ ] A method exists to create the admin's `profiles` row.
- [ ] The method is documented and reproducible.
- [ ] Template and file creation will not fail due to missing profile.

## Required Final Report
Standard Implementation Mode report.
