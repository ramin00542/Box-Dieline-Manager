# Repair Prompt — EXCEPTION WHEN OTHERS Swallows Errors

## Purpose
Narrow the `EXCEPTION WHEN OTHERS` block in the `templates_search_vector_update()` function to catch only the specific exception related to missing text search configuration, rather than silently swallowing all errors.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 20 (Low).

Current trigger function:
```sql
EXCEPTION WHEN OTHERS THEN
  -- Fallback to simple config
```
This catches every possible error (data errors, type errors, permission errors, etc.) and silently falls back to `'simple'` config. The intent is only to handle the case where `'persian'` text search config is unavailable.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (search vector trigger function)

## Required User Decision, If Any
None. The fix is a SQL best practice.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code.
- Do not remove the fallback entirely.

## Required Changes
1. Replace `EXCEPTION WHEN OTHERS THEN` with a more specific exception handler. In PostgreSQL, the error for missing text search configuration is `SQLSTATE 42704` (undefined_object). Use:
   ```sql
   EXCEPTION WHEN SQLSTATE '42704' THEN
   ```
   Or use the condition name `UNDEFINED_OBJECT` if available.
2. Add a comment that any other exception should propagate so it can be detected during development/testing.
3. The existing comment about runtime verification should reference this narrower exception handling.

## Compatibility Requirements
- ADR-013: Fallback from Persian to simple config must still work.
- Must not break existing trigger behavior.

## Acceptance Criteria
- [ ] The exception handler catches only the expected error (missing text search config).
- [ ] Other errors propagate normally.
- [ ] The fallback to `'simple'` config still works when Persian is unavailable.

## Required Final Report
Standard Implementation Mode report.
