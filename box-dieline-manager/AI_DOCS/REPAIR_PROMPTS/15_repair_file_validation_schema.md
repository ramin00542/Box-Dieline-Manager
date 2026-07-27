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
2. Document the exact MIME type mapping per `file_type`:
   ```typescript
   const ALLOWED_MIME_TYPES: Record<string, string[]> = {
     'image': ['image/jpeg', 'image/png', 'image/webp'],
     'thumbnail': ['image/jpeg', 'image/png', 'image/webp'],
     'pdf': ['application/pdf'],
     'ai': ['application/postscript', 'application/illustrator'],
     'cdr': ['application/coreldraw', 'application/x-coreldraw', 'application/octet-stream'],
   };
   ```
3. Add the cross-validation refinement to `fileUploadSchema`:
   ```typescript
   export const fileUploadSchema = z.object({
     file_type: z.enum(['image', 'thumbnail', 'pdf', 'ai', 'cdr']),
     file_name: z.string().min(1),
     file_size: z.number().positive().max(50 * 1024 * 1024, 'File must be < 50MB'),
     mime_type: z.string().min(1),
   }).refine(
     (data) => {
       const allowed = ALLOWED_MIME_TYPES[data.file_type];
       return allowed ? allowed.includes(data.mime_type) : false;
     },
     {
       message: 'MIME type does not match the declared file type',
       path: ['mime_type'],
     }
   );
   ```
4. Document file extension check as an additional validation layer (applications should check both MIME type and extension).

## Compatibility Requirements
- Must match the allowed formats in `docs/05_TECH_SPEC.md` (JPG, PNG, WebP, PDF, AI, CDR).
- Must not prevent valid uploads — `application/octet-stream` for CDR files accounts for non-standard MIME registrations.
- ADR-016/017: Upload validation runs server-side via API Route; the `storage_url` column is NOT populated at upload time (see Repair Prompt 12).

## Acceptance Criteria
- [ ] `file_size` has a 50MB maximum.
- [ ] MIME type is validated against `file_type`.
- [ ] Contradictory `file_type`/`mime_type` combinations are rejected.
- [ ] All allowed formats from `docs/05_TECH_SPEC.md` remain valid.

## Required Final Report
Standard Implementation Mode report.
