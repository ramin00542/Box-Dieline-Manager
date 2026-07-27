# Repair Prompt — Template_id Nullable in Files Table

## Purpose
Decide whether `template_id` in the `files` table should be `NOT NULL` and update the schema accordingly.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 16 (Medium, Needs Decision).

Current schema: `template_id UUID REFERENCES templates(id) ON DELETE CASCADE,` (nullable).
Rule 14: "Templates reference files via template_id foreign key."
System goal: All files belong to a template. Nullable `template_id` allows orphan files.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rule 14)

## Required User Decision, If Any
**Yes — Resolved.** The user decided:
1. `template_id` in the `files` table **must be `NOT NULL`**. Every file must belong to a template.
2. No orphan files are permitted — not even transient upload states. Any upload flow must ensure the template record is created first (or files are uploaded to a staging area with immediate cleanup on failure).

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code.
- Do not remove the foreign key constraint.

## Required Changes
1. Add `NOT NULL` to the `template_id` column in the `files` table definition in `docs/06_DATA_SCHEMA.md`:
   ```sql
   template_id UUID NOT NULL REFERENCES templates(id) ON DELETE CASCADE
   ```
2. Add a documentation note that the upload flow must create the template record (or a placeholder) before uploading files, or use a two-phase commit pattern (temp upload → on success, link to template → on failure, clean up).
3. Ensure the existing RLS policies on `files` are compatible with `NOT NULL` (they already filter by `template_id` existence, so no change needed there).

## Compatibility Requirements
- Must not break the `ON DELETE CASCADE` behavior.
- Must be consistent with Rule 13/14.

## Acceptance Criteria
- [ ] Decision is documented and schema reflects it.
- [ ] No orphan file edge case is unhandled.

## Required Final Report
Standard Implementation Mode report.
