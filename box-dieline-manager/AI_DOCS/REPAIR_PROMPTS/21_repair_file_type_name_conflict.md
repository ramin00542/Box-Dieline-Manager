# Repair Prompt — TypeScript `File` Interface Name Conflicts with DOM `File`

## Purpose
Rename the TypeScript `File` interface to avoid shadowing the global DOM `File` type, improving code clarity and avoiding potential type conflicts in browser contexts.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 21 (Low).

`docs/06_DATA_SCHEMA.md` defines `export interface File { ... }` for the database file entity. In browser/TypeScript environments, `File` is a global type for uploaded files. While the module export prevents direct collision in strict TypeScript, the naming creates confusion and potential import shadowing issues.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (TypeScript Types section)

## Required User Decision, If Any
None — resolved. Rename to `TemplateFileRecord` for maximum clarity.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code or other docs.
- Do not change the database table name `files` — only the TypeScript interface.

## Required Changes
1. Rename `File` to `TemplateFileRecord` in the TypeScript types section of `docs/06_DATA_SCHEMA.md`:
   ```typescript
   export interface TemplateFileRecord {
     id: string;
     template_id: string;
     file_type: 'image' | 'thumbnail' | 'pdf' | 'ai' | 'cdr';
     file_name: string;
     file_size: number;
     mime_type: string;
     storage_path: string;
     // storage_url is NOT stored — see ADR-016/017; Signed URL generated at request time
     uploaded_by?: string;
     created_at: string;
   }
   ```
2. Update `TemplateWithFiles` to use `TemplateFileRecord`:
   ```typescript
   export interface TemplateWithFiles extends Template {
     files: TemplateFileRecord[];
     thumbnail_url?: string;
   }
   ```
3. Add a comment explaining the naming rationale:
   ```typescript
   // NOTE: Named TemplateFileRecord instead of File to avoid shadowing the
   // global DOM File type used in browser upload forms. The database table
   // remains `files`.
   ```

## Compatibility Requirements
- The database table remains `files`.
- The generated types must use `TemplateFileRecord` (or equivalent).
- All references within `docs/06_DATA_SCHEMA.md` must be updated.
- The `storage_url` field is removed per ADR-016/017 (see Repair Prompt 12).

## Acceptance Criteria
- [ ] The interface is renamed to avoid DOM `File` conflict.
- [ ] `storage_url` is removed from the interface (aligned with Repair Prompt 12).
- [ ] All references within `docs/06_DATA_SCHEMA.md` are updated.

## Required Final Report
Standard Implementation Mode report.
