# Repair Prompt — Template CRUD Scope Ambiguity

## Purpose
Resolve the ambiguity between ADR-011's "Template CRUD" statement and Phase 4's actual task list, which lacks explicit update/delete tasks.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 10 (High).

ADR-011: "Template CRUD" is listed as an MVP feature.
Phase 4 tasks include: template form, image upload, pdf upload, AI/CDR upload, template list, template detail.
Missing tasks: explicit update/delete template, soft-delete restore, file replacement, status/tag editing.
Schema has `deleted_at` column, indicating soft-delete was intended.

## Mandatory Reading
- `docs/08_PROJECT_PHASES_AND_TASKS.md`
- `docs/09_DECISIONS.md` (ADR-011)
- `docs/06_DATA_SCHEMA.md`
- `docs/02_PROJECT_GOAL.md`

## Required User Decision, If Any
**Yes — Resolved.** The user decided:
1. **MVP includes full CRUD**: Create, Read, Update, and Delete (soft-delete only — no hard delete in MVP).
2. **Soft-delete IS part of MVP** — the `deleted_at` column is an active MVP feature, not a future placeholder.
3. **Two new tasks must be added** to Phase 4: one for template editing/update, one for template soft-delete/archive.
4. **No hard delete** from the database in MVP — all deletions are logical via `deleted_at`.

## Allowed Files
- `docs/09_DECISIONS.md` (ADR-011 clarification)
- `docs/08_PROJECT_PHASES_AND_TASKS.md` (if adding tasks)
- `docs/06_DATA_SCHEMA.md` (if removing `deleted_at`)

## Forbidden Actions
- Do not implement code.
- Do not modify task files in AI_DOCS/PARTS/ without explicit authorization.

## Required Changes
1. **Update ADR-011** in `docs/09_DECISIONS.md`: Clarify that MVP includes full CRUD (Create, Read, Update, Soft-Delete) — not just Create/Read. Replace ambiguous "Template CRUD" with explicit "Template Create, Read, Update, and Soft-Delete (via deleted_at)".
2. **Add three tasks to Phase 4** in `docs/08_PROJECT_PHASES_AND_TASKS.md`:
   - `task_04_07_dashboard.md` — Dashboard (total template count, recent additions).
   - `task_04_08_template_edit.md` — Template update/edit form (reuses the template form component with pre-filled data, updates `status`, `tags`, and all dimension fields).
   - `task_04_09_template_soft_delete.md` — Soft-delete & archive (set `deleted_at` timestamp, toggle `status` to 'archived'/'inactive', list archived templates, restore from archive).
3. **Create/update task placeholder files** in `AI_DOCS/PARTS/phase_04_crud_upload/`:
   - `task_04_07_dashboard.md` already exists (Dashboard).
   - `task_04_08_template_edit.md` — create with placeholder content.
   - `task_04_09_template_soft_delete.md` — create with placeholder content.
4. **Document `deleted_at` as active** in `docs/06_DATA_SCHEMA.md`: Add a comment in the schema noting that `deleted_at` is an active MVP feature for soft-delete, not a future placeholder. All query policies (`deleted_at IS NULL`) are already correct.
5. **Update `docs/01_RULES.md`** Rule 2 (MVP includes): Ensure the rule list aligns with full CRUD scope.

## Compatibility Requirements
- Must not contradict 01_RULES.md Rule 2 (Scope Control).
- Schema with `deleted_at` must be consistent with the decision.

## Acceptance Criteria
- [ ] The CRUD scope is clearly documented.
- [ ] Either tasks are added or the schema is adjusted.
- [ ] No ambiguity remains for the implementer.

## Required Final Report
Standard Implementation Mode report.
