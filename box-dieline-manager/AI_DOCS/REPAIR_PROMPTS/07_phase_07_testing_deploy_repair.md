# Repair Prompt — Phase 07: Testing & Deployment

## Purpose
Write the three Phase 7 task files (`task_07_01` through `task_07_03`) with complete, actionable content. Ensure test strategy covers all phases, deployment pipeline is documented, and project documentation is completed.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓. Testing (Rule 7) and Deployment (Rule 10) are locked.

**B (Dependency order):** Phase 7 is last — depends on all previous phases. Correctly positioned in ADR-012 ✓.

**C (Next.js/TS Correctness):** No new code expected in testing/deployment phase. Focus is on verification.

**D (Database/API):** No new schema or API changes in Phase 7.

**E (Schema consistency):** Not applicable (no new schema).

**F (Tests):** This phase IS the testing phase. Must define:
- Unit tests per feature.
- Integration tests with Supabase local emulator.
- E2E tests for critical flows (login, CRUD, search, share link).
- Test data fixtures and cleanup.
- Mock vs real service boundaries.

**G (Cross-task):** Tests must be designed to not interfere with each other. Test isolation is critical.

**H (UI/UX):** Not applicable (testing phase).

**I (Security):** Deployment must enforce environment variable configuration, no hardcoded secrets, and proper Supabase production settings.

**J (Performance):** Lighthouse targets defined in `05_TECH_SPEC.md` must be verified.

## Mandatory Reading
- `docs/01_RULES.md` (Rules 7, 10, 11)
- `docs/05_TECH_SPEC.md` (Performance Targets)
- `docs/09_DECISIONS.md` (ADR-012)
- Related per-item repair prompts: None directly

## Required User Decision, If Any
None. Testing and deployment patterns are standard.

## Allowed Files
- `AI_DOCS/PARTS/phase_07_testing_deploy/task_07_01_e2e_tests.md`
- `AI_DOCS/PARTS/phase_07_testing_deploy/task_07_02_production_deploy.md`
- `AI_DOCS/PARTS/phase_07_testing_deploy/task_07_03_documentation.md`

## Forbidden Actions
- Do NOT implement actual test code or deployment scripts.
- Do NOT modify task files from other phases.
- Do NOT modify governance docs.

## Required Changes
1. Write `task_07_01_e2e_tests.md`: Define E2E test strategy:
   - Use Supabase local emulator for integration tests (per Rule 7).
   - Test flows: login → create template → upload files → search → generate share link → access as anonymous.
   - No test touches real production database.
   - Coverage requirements: all CRUD operations, search variants, share link flows, auth edge cases.
   - Test data cleanup.
2. Write `task_07_02_production_deploy.md`: Deployment checklist:
   - Environment variables for production (Supabase production project, not local).
   - Vercel project setup (per ADR-001).
   - Supabase RLS policy verification.
   - Storage bucket configuration (private, per ADR-016).
   - No deployment from local machine without CI/CD (Rule 10).
   - Post-deployment smoke tests.
3. Write `task_07_03_documentation.md`: Documentation requirements:
   - Project README with setup instructions.
   - Admin user creation guide.
   - API documentation (auto-generated or manual).
   - Maintenance guide (backup, monitoring).

## Compatibility Requirements
- Rule 7: Tests must not touch real production database.
- Rule 10: Production deployment only via authorized task.
- Rule 11: No commit without testing.
- ADR-012: Phase 7 is the final phase.

## Acceptance Criteria
- [ ] All 3 Phase 7 task files contain complete, actionable instructions.
- [ ] Test strategy covers all previous phases.
- [ ] Deployment checklist is complete.
- [ ] Documentation requirements are specified.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
