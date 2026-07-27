# Repair Prompt — File Validation Schema Lacks Enforcements

## Purpose
Update the Zod `fileUploadSchema` to enforce file size limits, MIME type validation per file_type, and prevent contradictory file_type/mime_type combinations.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 15 (Medium).

Current `fileUploadSchema` only checks `file_type` enum, `file_name` min(1), `file_size` positive, `mime_type` min(1). It does not:
- Enforce 50MB max file size.
- Validate MIME type per `file_type` (e.g., `file_type: 'pdf'` should require `application/pdf`).
- Restrict file extensions.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (Zod schemas section)
- `docs/05_TECH_SPEC.md` (File Upload Rules)

## Required User Decision, If Any
None. The constraints are already defined in `docs/05_TECH_SPEC.md`.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code.
- Do not remove existing validation.

## Required Changes
1. Update `fileUploadSchema` in `docs/06_DATA_SCHEMA.md`:
   - Add `.max(50 * 1024 * 1024, 'File must be < 50MB')` to `file_size`.
   - Add a MIME type mapping validation: `file_type` must correspond to known MIME types.
   - Add a refinement/transform that cross-validates `file_type` and `mime_type`.
   - Document that this is server-side validation; client-side validation should match.
2. Document which MIME types are allowed per `file_type`.

## Compatibility Requirements
- Must match the allowed formats in `docs/05_TECH_SPEC.md` (JPG, PNG, WebP, PDF, AI, CDR).
- Must not prevent valid uploads.

## Acceptance Criteria
- [ ] `file_size` has a 50MB maximum.
- [ ] MIME type is validated against `file_type`.
- [ ] Contradictory `file_type`/`mime_type` combinations are rejected.
- [ ] All allowed formats from `docs/05_TECH_SPEC.md` remain valid.

## Required Final Report
Standard Implementation Mode report.
