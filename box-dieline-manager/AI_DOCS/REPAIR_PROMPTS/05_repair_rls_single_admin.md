# Repair Prompt — RLS Policies Not Enforcing Single-Admin Model

## Purpose
Align RLS policies with the project's single-admin model. Current policies allow any authenticated Supabase user to read/write all templates and files, which contradicts the architectural decision that MVP has exactly one admin user.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 5 (High).

Current policies in `docs/06_DATA_SCHEMA.md` use `auth.uid() IS NOT NULL`, which grants access to **any** authenticated user. The documents state:
- `docs/02_PROJECT_GOAL.md`: "Admin login (single user)"
- `docs/05_TECH_SPEC.md`: "Single admin user for MVP"
- ADR-005, ADR-011: Single-admin model

A note in `06_DATA_SCHEMA.md` says "public sign-up disabled" but no policy, migration, or trigger enforces the single-admin constraint.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (all RLS policies for profiles, templates, files)
- `docs/01_RULES.md` (Rules 5, 8, 13)
- `docs/09_DECISIONS.md` (ADR-005, ADR-011, ADR-014)

## Required User Decision, If Any
None. The admin user ID can be stored as a configuration value and checked in RLS policies.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not add application code.
- Do not modify other docs files.
- Do not create an actual admin user — only document the mechanism.

## Required Changes
1. Document the mechanism for enforcing single-admin:
   - Option A: RLS policy that checks `auth.uid() = 'specific-admin-uuid'` (hardcoded, set during setup).
   - Option B: A `app_settings` table or `current_setting` parameter that stores the admin user ID.
2. Update all `templates` and `files` RLS policies to check for the specific admin user ID, not just any authenticated user.
3. Add a migration/seed step in the Phase 2 task documentation that sets the admin UUID.

## Compatibility Requirements
- ADR-005: Supabase Auth with email/password.
- ADR-011: Single-admin MVP.
- ADR-014: RLS on all tables.
- Must still allow public share link access (after Repair Prompts 01–03).

## Verification Boundaries
- **Static verification**: Policy SQL can be reviewed.
- **Requires runtime verification**: Create a second auth user and confirm they cannot access templates/files.

## Acceptance Criteria
- [ ] A second authenticated Supabase user cannot read or write any templates or files.
- [ ] The single admin user retains full access.
- [ ] Public share link access (with valid token) is unaffected.
- [ ] The mechanism is documented and reproducible during setup.

## Required Final Report
Standard Implementation Mode report.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
