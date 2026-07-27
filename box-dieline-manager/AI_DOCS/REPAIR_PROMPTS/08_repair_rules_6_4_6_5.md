# Repair Prompt — Rules 6.4 and 6.5 Referenced but Undefined

## Purpose
Add missing Rules 6.4 and 6.5 to `docs/01_RULES.md`. The Master Audit Prompt (`docs/07_MASTER_AUDIT_PROMPT.md`) depends on these rules for governance of `CURRENT_TASK.md` and `CHANGELOG.md` modifications. Without them, the audit/repair workflow defined in 07 cannot execute correctly.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 8 (High, Execution-Blocking).

`docs/07_MASTER_AUDIT_PROMPT.md` references Rules 6.4 and 6.5 in multiple places:
- "Do NOT modify CURRENT_TASK.md unless Rules 6.4 and 6.5 are fully satisfied."
- "Do NOT modify CHANGELOG.md unless Rule 6.4 verification exists."

`docs/01_RULES.md` Rule 6 only defines:
- 6.1 Clarification Mode
- 6.2 Review/Planning Mode
- 6.3 Implementation Mode

Rules 6.4 and 6.5 do not exist.

## Mandatory Reading
- `docs/01_RULES.md` (entire file, especially Rule 6)
- `docs/07_MASTER_AUDIT_PROMPT.md` (all references to 6.4 and 6.5)

## Required User Decision, If Any
**Yes — User must decide the content of Rules 6.4 and 6.5.** The rules should define when CURRENT_TASK.md and CHANGELOG.md may be modified. Suggested definitions:
- **6.4 (Verification Rule)**: A task is verified only when all its acceptance criteria are met, all tests pass, and the changes have been reviewed. Only then may CHANGELOG.md be updated.
- **6.5 (Task Transition Rule)**: CURRENT_TASK.md may only be updated to transition to the next task after the current task's verification (Rule 6.4) is complete and the user has explicitly confirmed the transition.

## Allowed Files
- `docs/01_RULES.md`

## Forbidden Actions
- Do not modify `docs/07_MASTER_AUDIT_PROMPT.md` or other governance docs.
- Do not modify application code or task files.
- Do not modify `CURRENT_TASK.md` or `CHANGELOG.md` as part of this repair.

## Required Changes
1. Add **Rule 6.4 (Verification Rule)** to `docs/01_RULES.md`.
2. Add **Rule 6.5 (Task Transition Rule)** to `docs/01_RULES.md`.
3. Ensure the new rules cover:
   - When CHANGELOG.md may be modified (6.4).
   - When CURRENT_TASK.md may be updated to transition tasks (6.5).
   - What constitutes "verification" (tests passing, acceptance criteria met, review done).
   - Requirement for explicit user confirmation before transitions.

## Compatibility Requirements
- Must be consistent with existing Rules 6.1–6.3 (Response Modes).
- Must be referenced correctly by `docs/07_MASTER_AUDIT_PROMPT.md`.
- Must align with Rules 11 (Git Discipline) and 7 (Testing Discipline).

## Verification Boundaries
- **Static verification**: Confirm the new rules are syntactically consistent with existing rules and that references in 07_MASTER_AUDIT_PROMPT.md now point to existing rules.
- No runtime verification required.

## Acceptance Criteria
- [ ] Rule 6.4 is added to `docs/01_RULES.md`.
- [ ] Rule 6.5 is added to `docs/01_RULES.md`.
- [ ] Both rules are numbered consistently with the existing 6.x scheme.
- [ ] References in `docs/07_MASTER_AUDIT_PROMPT.md` now point to real rules.
- [ ] The user has approved the content of both rules.

## Required Final Report
Standard Implementation Mode report.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
