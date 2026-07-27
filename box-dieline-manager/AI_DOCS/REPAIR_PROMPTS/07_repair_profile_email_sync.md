# Repair Prompt — Profile Email Can Drift from Auth Email

## Purpose
Document that `profiles.email` is a cache/sync of `auth.users.email` and should not be user-modifiable, or provide a mechanism to keep them in sync.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 7 (High).

Current `profiles` has `email TEXT UNIQUE NOT NULL` and an `UPDATE` policy that allows updating all columns. No trigger or constraint prevents `profiles.email` from diverging from `auth.users.email`.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (profiles table, RLS policies)
- `docs/01_RULES.md` (Rule 13)
- `docs/09_DECISIONS.md` (ADR-008)

## Required User Decision, If Any
None — resolved to trigger-synced approach.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code or other docs.

## Required Changes
1. `profiles.email` is a sync of `auth.users.email` — it is NOT user-modifiable.
2. Extend the `handle_new_user()` trigger (from Repair Prompt 06) to sync email at creation time. This already covers the initial sync via `NEW.email`.
3. Add a **second trigger** to sync email on change in `auth.users`:
   ```sql
   CREATE OR REPLACE FUNCTION public.sync_user_email()
   RETURNS TRIGGER AS $$
   BEGIN
     UPDATE public.profiles
     SET email = NEW.email
     WHERE id = NEW.id;
     RETURN NEW;
   END;
   $$ LANGUAGE plpgsql SECURITY DEFINER;

   CREATE TRIGGER on_auth_user_email_change
     AFTER UPDATE OF email ON auth.users
     FOR EACH ROW EXECUTE FUNCTION public.sync_user_email();
   ```
4. Update the `profiles` RLS `UPDATE` policy to explicitly **exclude** the `email` column from user updates, OR restrict the policy to only allow updating `full_name` and `is_active` columns.
   Note: PostgreSQL RLS `WITH CHECK` cannot restrict columns — column-level security requires application-layer enforcement or split policies. Document that the application layer must NOT provide an email edit field.

## Compatibility Requirements
- ADR-008: Use `profiles` table.
- Must not break the foreign key relationship or unique constraint.

## Acceptance Criteria
- [ ] The role of `profiles.email` is clearly documented.
- [ ] The mechanism to keep it in sync (or lock it) is specified.
- [ ] Single-admin context means this is low-risk, but should be documented for future multi-user scenarios.

## Required Final Report
Standard Implementation Mode report.
