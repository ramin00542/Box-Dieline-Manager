# Repair Prompt — 50MB Upload & Thumbnail Architecture Not Specified

## Purpose
Document the architecture for file uploads (including handling files up to 50MB) and thumbnail generation, specifying where each operation runs and what limits apply.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 13 (Medium).

`docs/05_TECH_SPEC.md` specifies: "Maximum file size: 50MB per file" and allowed formats (JPG, PNG, WebP, PDF, AI, CDR). Rule 14 and ADR-009 require thumbnail generation on upload. However, no document specifies:
- Whether upload is direct-to-Supabase (browser → Storage) or proxied through Next.js API route.
- Where thumbnails are generated (browser, Vercel Function, Supabase Edge Function, external service).
- How 50MB uploads interact with Vercel serverless limits (body size, duration, memory).
- What happens if thumbnail generation fails.

## Mandatory Reading
- `docs/05_TECH_SPEC.md`
- `docs/01_RULES.md` (Rule 14)
- `docs/09_DECISIONS.md` (ADR-009, ADR-016)
- `docs/06_DATA_SCHEMA.md`

## Required User Decision, If Any
None — resolved. Locked upload architecture per ADR-016/017 and `template_id NOT NULL` constraint.

## Allowed Files
- `docs/05_TECH_SPEC.md`
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not implement application code.
- Do not add dependencies without ADR.

## Required Changes
In `docs/05_TECH_SPEC.md`, add an **Upload & Cleanup Contract** section specifying the complete flow:

### Phase 1: Template Creation (pre-upload)
1. Admin creates the template record first (INSERT into `templates`) — this generates the `templateId`.
2. Response returns `templateId` to client. No files attached yet.

### Phase 2: File Upload
3. Client requests upload permission for each file by calling `POST /api/upload/init` with `{ templateId, file_type, file_name, file_size, mime_type }`.
4. The API Route:
   - Validates the request using the strengthened `fileUploadSchema` (Zod with 50MB cap + MIME cross-check).
   - Generates a Signed Upload URL (Supabase `createSignedUploadUrl()`) scoped to path:
     `templates/{templateId}/{fileId}/{original_filename}` where `fileId` is pre-generated as UUID.
   - Returns the Signed Upload URL and `fileId` to client.
5. Client uploads directly to Supabase Storage using the Signed Upload URL — **NOT proxied through Next.js** (Vercel serverless has 4.5MB body limit, not suitable for 50MB files).
6. On upload success, client calls `POST /api/upload/confirm` with `{ templateId, fileId }`.
7. The API Route:
   - Verifies the file exists in Storage at the expected path.
   - Inserts the file metadata row into the `files` table.
   - Initiates thumbnail generation (if file is an image).

### Phase 3: Thumbnail Generation
8. For images (JPG, PNG, WebP), generate thumbnail on the client-side before upload using canvas (max 400×400).
9. Upload thumbnail as a separate file with `file_type: 'thumbnail'` using the same Signed Upload URL flow.
10. If thumbnail generation fails, log the error and continue with the original image. Admin can regenerate thumbnails later via a dedicated API endpoint.

### Phase 4: Cleanup (failure recovery)
11. If the metadata insert fails (Phase 2, step 7), the uploaded file in Storage becomes an orphan. The cleanup strategy:
    - **Immediate**: The API Route must attempt to delete the orphan file from Storage using `supabase.storage.from('template-files').remove([storagePath])`.
    - **Background**: If the immediate delete fails, log the orphan path for manual cleanup. A periodic cleanup job (post-MVP) can scan for files without matching `files` table entries.
12. If the client disconnects mid-upload, the Signed Upload URL expires automatically (default 60-minute TTL). The file remains in Storage with no metadata — cleaned up by the background job.

### Compatibility Requirements
- ADR-009: Thumbnails on upload, max 400x400, originals preserved.
- ADR-016: Private buckets, Signed URLs only.
- ADR-017: All file operations for admin go through API Route with Service Role.
- `template_id NOT NULL` constraint: Template must exist before any file upload.
- Supabase Storage limits: Standard 5GB per file on paid plans; 50MB is well within limits.

### Acceptance Criteria
- [ ] Complete upload/cleanup contract is documented with all 4 phases.
- [ ] 50MB handling is addressed (direct-to-Storage, not proxied through Next.js).
- [ ] Cleanup strategy for orphan files is defined.
- [ ] Signed Upload URL flow is documented.
- [ ] The documentation is implementable without architectural surprises.

## Required Final Report
Standard Implementation Mode report.
