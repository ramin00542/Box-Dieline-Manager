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
None. The approach (read-only email vs trigger-synced) should be documented.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code or other docs.

## Required Changes
1. Clarify in `docs/06_DATA_SCHEMA.md` whether `profiles.email` is:
   - A read-only copy (should not be updatable by the user), OR
   - Synced via trigger from `auth.users` on email change.
2. If read-only: Update the `UPDATE` policy to exclude the `email` column, or document that the application layer must not allow email edits.
3. If trigger-synced: Document the trigger.

## Compatibility Requirements
- ADR-008: Use `profiles` table.
- Must not break the foreign key relationship or unique constraint.

## Acceptance Criteria
- [ ] The role of `profiles.email` is clearly documented.
- [ ] The mechanism to keep it in sync (or lock it) is specified.
- [ ] Single-admin context means this is low-risk, but should be documented for future multi-user scenarios.

## Required Final Report
Standard Implementation Mode report.
