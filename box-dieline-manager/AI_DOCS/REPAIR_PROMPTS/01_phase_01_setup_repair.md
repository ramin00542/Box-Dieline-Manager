# Repair Prompt — Phase 01: Setup & Foundation

## Purpose
Standardize Phase 1 task files (task_01_00 through task_01_03) to include accurate project config requirements, ADR compliance, and complete setup instructions. Currently all four task files are empty placeholders. Additionally, document the missing project config files (`package.json`, `tsconfig.json`, `next.config.js`, `tailwind.config.ts`, `supabase/config.toml`) as deliverables of this phase.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & task-scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓. However, `task_01_00_lock_tech_stack.md` references locking the tech stack but the stack is already locked in ADR-001 through ADR-006 and `05_TECH_SPEC.md`. The task should be clarified as "confirm and document" rather than "decide".

**B (Dependency order):** Phase 1 is correctly positioned first in ADR-012's 7-phase order ✓. No upstream dependencies ✓.

**C (Next.js/TS Correctness):** Cannot assess — task files are empty placeholders with no code.

**D (Database/API):** No SQL or API content in Phase 1 tasks ✓ (Phase 1 is setup).

**E (Schema consistency):** No schema content in Phase 1 ✓.

**F (Tests):** No test content in Phase 1 tasks ✓ (setup phase typically exempt from testing requirements per Rule 7).

**G (Cross-task):** Phase 1 must create `.env.local` template, `package.json`, `tsconfig.json`, `next.config.js`, `tailwind.config.ts` — these are prerequisites for all later phases. No task file currently specifies these outputs.

**H (UI/UX):** No UI content in Phase 1 ✓.

**I (Security):** No security-specific content yet, but `.env.local` template should list required vars (NEXT_PUBLIC_SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, etc.).

**J (Performance):** No performance content expected in Phase 1 ✓.

## Mandatory Reading
- `docs/01_RULES.md`
- `docs/02_PROJECT_GOAL.md`
- `docs/05_TECH_SPEC.md`
- `docs/09_DECISIONS.md` (ADR-001 through ADR-006, ADR-012)

## Required User Decision, If Any
None. Phase 1 tasks are standard project initialization with no architectural decisions pending.

## Allowed Files
- `AI_DOCS/PARTS/phase_01_setup/task_01_00_lock_tech_stack.md`
- `AI_DOCS/PARTS/phase_01_setup/task_01_01_project_initialization.md`
- `AI_DOCS/PARTS/phase_01_setup/task_01_02_dev_environment.md`
- `AI_DOCS/PARTS/phase_01_setup/task_01_03_git_setup.md`

## Forbidden Actions
- Do NOT create actual project config files (`package.json`, etc.) — only document what they should contain.
- Do NOT modify other Phases' task files.
- Do NOT modify governance docs (`01_RULES.md`, `05_TECH_SPEC.md`, `09_DECISIONS.md`).

## Required Changes
1. Write `task_01_00_lock_tech_stack.md`: Confirm the locked stack (Next.js 14, TypeScript strict, Supabase, Tailwind + shadcn/ui, Vercel). Document each ADR reference (ADR-001 through ADR-006). State that no further tech-stack ADRs are needed for MVP.
2. Write `task_01_01_project_initialization.md`: Specify creation of `package.json`, `tsconfig.json`, `next.config.js`, `tailwind.config.ts`, `.env.local` template, and project root directory structure per `05_TECH_SPEC.md`. List all npm dependencies with versions.
3. Write `task_01_02_dev_environment.md`: Document Node.js 20 LTS requirement, VS Code extensions, `.gitignore` (already exists at root — confirm), and `supabase/config.toml` setup.
4. Write `task_01_03_git_setup.md`: Document git flow, branch naming, commit conventions. Reference Rule 11 (Git Discipline).

## Compatibility Requirements
- ADR-001 through ADR-006: Must reference all locked decisions.
- ADR-012: Phase 1 is the first phase.
- `01_RULES.md` Rule 3: One task per request.
- `05_TECH_SPEC.md` Project Structure: Must match.

## Verification Boundaries
- **Static verification**: Task files can be reviewed for completeness and ADR alignment.
- **Requires runtime verification**: Only after actual `npm create next-app` and `npm install`.

## Acceptance Criteria
- [ ] All 4 Phase 1 task files contain complete, actionable instructions.
- [ ] Tech stack references match locked ADRs.
- [ ] Dependencies listed are sufficient for all later phases.
- [ ] `.env.local` template includes all required environment variables.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
