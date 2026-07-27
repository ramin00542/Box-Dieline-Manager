# Repair Prompt — Storage Bucket Name, Path Structure, and RLS Undefined

## Purpose
Define the Supabase Storage bucket configuration: bucket name, folder path structure, Storage RLS policies, cleanup procedures, and consistency guarantees.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 17 (Medium).

Current docs specify that Storage is private (Rule 8, ADR-016) and `storage_path` is stored in the `files` table, but do not define:
- Bucket name.
- Folder structure: `templates/{templateId}/{fileId}/{filename}` or similar.
- Storage RLS policies (`storage.objects` policies).
- Overwrite prevention.
- Cleanup on soft-delete or file deletion.
- Rollback procedure if upload succeeds but metadata insert fails.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/01_RULES.md` (Rules 8, 14)
- `docs/09_DECISIONS.md` (ADR-004, ADR-016)
- `docs/05_TECH_SPEC.md`

## Required User Decision, If Any
None. Standard Supabase Storage patterns.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not implement code.
- Do not create actual Storage resources.

## Required Changes
1. In `docs/05_TECH_SPEC.md`, add Storage Configuration section:
   - Bucket name: `template-files` or user-approved name.
   - Path structure: `templates/{templateId}/{fileId}/{original_filename}`.
   - Overwrite prevention: Use unique file IDs in path, never overwrite.
2. In `docs/06_DATA_SCHEMA.md`, add Storage RLS policy section:
   - Bucket-level: Private (no public access).
   - `storage.objects` policies: Authenticated admin can read/write; public users access only via Signed URLs.
   - Cleanup: Document that file deletion from Storage must happen explicitly (separate from DB delete).
   - Rollback: Document the two-phase upload (upload to temp path → on success, move to final path → on failure, clean up temp file).

## Compatibility Requirements
- ADR-004: Supabase Storage.
- ADR-016: Private buckets, Signed URLs only.
- Must not conflict with File Upload Rules in `docs/05_TECH_SPEC.md`.

## Acceptance Criteria
- [ ] Bucket name is documented.
- [ ] Path structure is documented.
- [ ] Storage RLS policies are defined.
- [ ] Cleanup and rollback procedures are documented.

## Required Final Report
Standard Implementation Mode report.
