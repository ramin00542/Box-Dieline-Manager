# Repair Prompt — RLS Policies Not Enforcing Single-Admin Model

## Purpose
~~Align RLS policies with the project's single-admin model.~~ ✅ **RESOLVED**

## Resolution Summary
**Decision:** Added `is_admin BOOLEAN DEFAULT false` column to `profiles` table.
All RLS policies on `templates` and `files` now check:
```sql
auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true)
```
instead of the insecure `auth.uid() IS NOT NULL`.

**Applied in:** `docs/06_DATA_SCHEMA.md` — commit `61a9faf`

**Decision Reference:** ADR-018 (see `docs/09_DECISIONS.md`)

**What was done:**
1. `is_admin BOOLEAN DEFAULT false` added to `profiles` table (after `is_active`)
2. A seed comment documents that the first admin's `is_admin` must be set manually
3. All 3 templates RLS policies renamed and updated to check `is_admin`
4. All 3 files RLS policies renamed and updated to check `is_admin`
5. TypeScript `Profile` interface includes `is_admin: boolean`
6. Old MVP note replaced with accurate enforcement description + ADR-018 reference

**This repair prompt is now RESOLVED and requires no further action.**
