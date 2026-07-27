# Repair Prompt — Phase 04: Template CRUD & File Upload

## Purpose
Write the Phase 4 task files (`task_04_01` through `task_04_06`, plus the two new tasks `task_04_07` and `task_04_08`) with complete, actionable content. Incorporate user decisions on CRUD scope (Full CRUD with soft-delete), upload architecture, file validation, and storage policies.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** User decided Full CRUD is in scope. Two new tasks needed: `task_04_07_template_edit.md` and `task_04_08_template_soft_delete.md`. Dashboard task also needs placement.

**B (Dependency order):** Phase 4 depends on Phase 2 (database) and Phase 3 (auth). Correctly positioned ✓.

**C (Next.js/TS Correctness):** No code yet. Must document correct Server/Client Component boundaries for forms. Must reference Rule 9 (no `any` type).

**D (Database/API):** 
- **Item 10** (CRUD scope): Full CRUD confirmed + soft-delete.
- **Item 13** (Upload architecture): Must specify direct-to-Supabase upload.
- **Item 15** (File validation): Must enforce 50MB limit and MIME validation.

**E (Schema consistency):** Templates and files tables from `06_DATA_SCHEMA.md` are the schema authority.

**F (Tests):** Forms and uploads require testing (both unit and integration). Must document test strategy.

**G (Cross-task):** Phase 4 is prerequisite for Phase 5 (search needs data) and Phase 6 (share links need existing templates).

**H (UI/UX):** All components should use shadcn/ui. Forms must have validation feedback. Template list must include loading states and pagination.

**I (Security):** Auth required for all write operations. Input validation (Zod on both client and server). File type restrictions. No direct file URL exposure.

**J (Performance):** Thumbnail generation (max 400x400 per ADR-009). Image optimization. Pagination for list views (cursor-based per Item 23).

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`
- `docs/01_RULES.md` (Rules 2, 9, 14)
- `docs/09_DECISIONS.md` (ADR-007, ADR-009, ADR-011, ADR-016)
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 10, 13, 15, 16, 17, 21)
- Related per-item repair prompts: `10`, `13`, `15`, `16`, `17`, `21`

## Required User Decision, If Any
None. All decisions resolved.

## Allowed Files
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_01_template_form.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_02_image_upload_with_thumbnail.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_03_pdf_upload.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_04_ai_cdr_upload.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_05_template_list_page.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_06_template_detail_page.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_07_template_edit.md` (new)
- `AI_DOCS/PARTS/phase_04_crud_upload/task_04_08_template_soft_delete.md` (new)

## Forbidden Actions
- Do NOT implement actual application code.
- Do NOT modify Phase 2, 3, or 5 task files.
- Do NOT modify governance docs.

## Required Changes
1. Write `task_04_01_template_form.md`: Create template registration form with shadcn/ui. Use Zod validation from `06_DATA_SCHEMA.md`. Include dimension fields with unit selector (`cm`/`mm`/`inch`). Reference Rule 8 (auth required for write).
2. Write `task_04_02_image_upload_with_thumbnail.md`: Image upload with client-side thumbnail generation (max 400x400 per ADR-009). Direct-to-Supabase upload flow. Enforce 50MB max. Store file metadata in `files` table.
3. Write `task_04_03_pdf_upload.md`: PDF upload, same pattern as image upload. Validate MIME type (`application/pdf`).
4. Write `task_04_04_ai_cdr_upload.md`: AI/CDR file upload. Note that these are design files with no preview — download only.
5. Write `task_04_05_template_list_page.md`: List page with pagination (cursor-based per Item 23). Show thumbnail, code, name, dimensions, status. Loading and empty states.
6. Write `task_04_06_template_detail_page.md`: Detail page showing all template info and downloadable file list. Reference Signed URL flow (Item 4).
7. Write `task_04_07_template_edit.md`: Edit form (pre-filled with existing data, reuses template form component). Updates `updated_at` via trigger.
8. Write `task_04_08_template_soft_delete.md`: Soft-delete (set `deleted_at` timestamp, toggle status). Show archived templates. Restore from archive. No hard delete in MVP.

## Compatibility Requirements
- ADR-007: Files in separate table, NOT in templates.
- ADR-009: Thumbnails max 400x400.
- ADR-016: Private buckets + Signed URLs.
- Rule 2: Scope control (no customer management, no analytics).
- File validation per Item 15.

## Acceptance Criteria
- [ ] All 8 Phase 4 task files contain complete, actionable instructions.
- [ ] CRUD scope reflects Full CRUD + soft-delete user decision.
- [ ] Upload architecture is documented (direct to Supabase).
- [ ] File validation (50MB, MIME) is documented.
- [ ] Pagination strategy is documented.
- [ ] Edit and soft-delete tasks are present.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
