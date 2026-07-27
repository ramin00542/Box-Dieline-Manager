# Repair Prompt — Dashboard Scope Ambiguity

## Purpose
Resolve whether a Dashboard page (showing total template count, recent additions) is part of the MVP scope, and if so, add the corresponding task to the appropriate Phase.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 22 (Low, Needs Decision).

- `docs/02_PROJECT_GOAL.md` mentions "Dashboard: total templates count, recent additions."
- `docs/01_RULES.md` MVP includes list does NOT mention Dashboard.
- `docs/08_PROJECT_PHASES_AND_TASKS.md` has no Dashboard task.
- ADR-011 MVP scope does NOT list Dashboard.
- Conflicting scope creates ambiguity under Rule 12.

## Mandatory Reading
- `docs/02_PROJECT_GOAL.md`
- `docs/01_RULES.md` (Rule 2)
- `docs/09_DECISIONS.md` (ADR-011)
- `docs/08_PROJECT_PHASES_AND_TASKS.md`

## Required User Decision, If Any
**Yes — Resolved.** The user decided:
1. **Dashboard IS part of MVP.** It remains in the scope (not removed from `docs/02_PROJECT_GOAL.md`).
2. **A new task should be created** for Dashboard. The preferred placement is after Phase 4 (Template CRUD & File Upload) — either as the last task in Phase 4, or in a new small phase (Phase 4.5 / Phase 4b) after Phase 4 and before Phase 5.
3. ADR-011 must be updated to include Dashboard in the MVP scope list.

## Allowed Files
- `docs/09_DECISIONS.md` (ADR-011 clarification)
- `docs/02_PROJECT_GOAL.md` (if removing Dashboard)
- `docs/08_PROJECT_PHASES_AND_TASKS.md` (if adding Dashboard task)

## Forbidden Actions
- Do not implement code.
- Do not add the task without user confirmation.

## Required Changes
1. **Update ADR-011** in `docs/09_DECISIONS.md`: Add "Dashboard (template count, recent additions)" to the MVP includes list.
2. **Update `docs/02_PROJECT_GOAL.md`**: Ensure Dashboard is consistently described (it already mentions it, but verify alignment).
3. **Update `docs/01_RULES.md`** Rule 2 (MVP includes): Add Dashboard to the list.
4. **Add a Dashboard task** to the project structure:
   - Option A: Add it as `task_04_07_dashboard.md` in Phase 4 (last Upload/CRUD task).
   - Option B: Create a new phase folder `phase_04b_dashboard/` after Phase 4.
   - The task file placeholder should be created with the same format as other task files.
5. **Update `docs/08_PROJECT_PHASES_AND_TASKS.md`**: Add the Dashboard entry to the appropriate Phase.

## Compatibility Requirements
- Rule 12 (Ambiguity Policy) requires clarification before proceeding.

## Acceptance Criteria
- [ ] Dashboard scope is unambiguously documented.
- [ ] Either a task exists or the goal document is consistent.
- [ ] No ambiguity remains for implementation.

## Required Final Report
Standard Implementation Mode report.
