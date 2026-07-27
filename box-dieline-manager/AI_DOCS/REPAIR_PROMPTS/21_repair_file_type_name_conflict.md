# Repair Prompt — TypeScript `File` Interface Name Conflicts with DOM `File`

## Purpose
Rename the TypeScript `File` interface to avoid shadowing the global DOM `File` type, improving code clarity and avoiding potential type conflicts in browser contexts.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 21 (Low).

`docs/06_DATA_SCHEMA.md` defines `export interface File { ... }` for the database file entity. In browser/TypeScript environments, `File` is a global type for uploaded files. While the module export prevents direct collision in strict TypeScript, the naming creates confusion and potential import shadowing issues.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (TypeScript Types section)

## Required User Decision, If Any
None. Renaming is a standard best practice.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not modify application code or other docs.
- Do not change the database table name `files` — only the TypeScript interface.

## Required Changes
1. Rename `File` to `FileRecord` (or similar descriptive name) in the TypeScript types section.
2. Update the `TemplateWithFiles` interface to reference the renamed type.
3. Add a comment explaining the naming rationale.

## Compatibility Requirements
- The database table remains `files`.
- The generated Prisma/types should use the new name.

## Acceptance Criteria
- [ ] The interface is renamed to avoid DOM `File` conflict.
- [ ] All references within `docs/06_DATA_SCHEMA.md` are updated.

## Required Final Report
Standard Implementation Mode report.
