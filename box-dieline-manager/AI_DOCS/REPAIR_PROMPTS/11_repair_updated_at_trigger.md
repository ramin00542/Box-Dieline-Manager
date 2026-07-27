# Repair Prompt — Updated_at Has No Auto-Update Trigger

## Purpose
Add a trigger to automatically update `updated_at` on row modification for `profiles` and `templates` tables.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 11 (Medium).

Current `profiles` and `templates` define `updated_at TIMESTAMPTZ DEFAULT NOW()`, but `DEFAULT NOW()` only fires on INSERT. No `BEFORE UPDATE` trigger updates the column on modification. Rule 13 requires `updated_at` on all tables (except append-only).

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rule 13)

## Required User Decision, If Any
None. Standard pattern.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code or other docs.
- Do not add the trigger to the `files` table (it is intentionally append-only per Rule 13 exception).

## Required Changes
1. Add a `BEFORE UPDATE` trigger function for both `profiles` and `templates`:
   ```sql
   CREATE OR REPLACE FUNCTION update_updated_at_column()
   RETURNS TRIGGER AS $$
   BEGIN
     NEW.updated_at = NOW();
     RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;

   CREATE TRIGGER set_profiles_updated_at
     BEFORE UPDATE ON profiles
     FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

   CREATE TRIGGER set_templates_updated_at
     BEFORE UPDATE ON templates
     FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
   ```

## Compatibility Requirements
- Rule 13: Must NOT add to `files` table (append-only exception).
- UUID primary keys, snake_case naming.

## Acceptance Criteria
- [ ] Trigger function `update_updated_at_column()` is defined.
- [ ] Trigger is applied to `profiles`.
- [ ] Trigger is applied to `templates`.
- [ ] `files` table is NOT modified.
- [ ] SQL syntax is valid PostgreSQL.

## Required Final Report
Standard Implementation Mode report.
