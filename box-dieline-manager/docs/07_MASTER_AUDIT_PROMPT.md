# Box Dieline Manager — Master Audit & Repair Prompt
## Complete Documentation Review and Controlled Execution

---

## Purpose
This is the single authoritative master prompt for the Box Dieline Manager
project. It operates in THREE phases:

- **Phase 1 — Full Audit:** Read-only inspection of all 7 Phases, all Tasks,
  and all governing documentation. Generate one repair prompt per Phase/topic.
- **Phase 2 — Controlled Execution:** Execute exactly ONE repair prompt per
  user command. Never auto-advance.
- **Phase 3 — Final Verification:** After all repairs, confirm consistency.

---

## Absolute Restrictions (All Phases)

- Do NOT create, modify, move, or delete any file unless explicitly authorized.
- Do NOT create application code, components, or pages.
- Do NOT modify `CURRENT_TASK.md` unless Rules 6.4 and 6.5 are fully satisfied.
- Do NOT modify `CHANGELOG.md` unless Rule 6.4 verification exists.
- Do NOT modify `01_RULES.md`, `05_TECH_SPEC.md`, or `09_DECISIONS.md` unless
  a repair prompt explicitly lists them.
- Do NOT run tests or claim runtime verification unless actually performed.
- Do NOT create Git commits unless explicitly authorized.
- Do NOT silently choose an architecture or tech stack.
- Do NOT invent numeric values (tolerance, file size limits) without explicit
  user approval.

---

## PHASE 1 — Full Audit and Repair Prompt Generation

### Trigger
User says:
```text
EXECUTE: MASTER AUDIT — Generate all repair prompts
```

### Mandatory Reading (Phase 1)
Read the CURRENT version of every file below before writing anything:

**Governance:**
1. `01_RULES.md`
2. `02_PROJECT_GOAL.md`
3. `05_TECH_SPEC.md`
4. `06_DATA_SCHEMA.md`
5. `09_DECISIONS.md`
6. `CURRENT_TASK.md`
7. `PROJECT_STATUS.md`

**All planned implementation tasks:**
Read every Markdown file under:
```text
AI_DOCS/PARTS/
```
including Phases 01 through 07.

**Relevant root/project files, read-only:**
- `.gitignore`
- `package.json`
- `tsconfig.json`
- `next.config.js`
- `tailwind.config.ts`
- `supabase/config.toml`

Do not modify any of them.

### Audit Checklist (A-J)

For every Phase from Phase 01 through Phase 07, inspect every Task file.

Check ALL of the following categories:

#### A. Authority and task-scope consistency
- [ ] Does the Task match the locked tech stack and ADRs?
- [ ] Does the Task introduce files not listed in Allowed Files?
- [ ] Does the Task require modifying a file that is not allowed?
- [ ] Does the Task require a CURRENT_TASK or CHANGELOG modification without
      satisfying Rules 6.4 and 6.5?

#### B. Dependency and execution-order consistency
- [ ] Are prerequisites correct?
- [ ] Does a task depend on a file/component/resource that does not exist yet?
- [ ] Does a task use a component before the component is introduced?
- [ ] Does it assume a database table before its creation task?
- [ ] Does a task conflict with ADR-012's locked 7-Phase order?
- [ ] Does a task incorrectly defer a required test without an explicit
      Rule 7 exemption?

#### C. Next.js and TypeScript Correctness
Check code examples for likely Next.js 14/TypeScript problems, including:
- [ ] Invalid syntax.
- [ ] Invalid type annotations.
- [ ] Invalid Server/Client Component boundaries.
- [ ] Invalid API route handlers.
- [ ] Invalid data fetching patterns.
- [ ] Incorrect use of `use client` directive.
- [ ] Incorrect use of `use server` actions.
- [ ] Unawaited/incorrect `async/await` usage.
- [ ] Invalid Supabase client usage.
- [ ] Incorrect form handling.
- [ ] Null-safety issues.
- [ ] Incorrect error handling.

Do not claim a runtime error unless it is provably invalid by static review.
For uncertain behavior, mark it as "Needs runtime verification."

#### D. Database and API Correctness
Check every occurrence of:
- [ ] SQL queries match schema
- [ ] RLS policies are correct
- [ ] API endpoints match RESTful conventions
- [ ] Request/response formats are consistent
- [ ] Validation schemas match database schema
- [ ] Error handling is consistent
- [ ] NO reference to `api/customers/` or `customers` table (out of MVP scope)

#### E. Data/resource schema consistency
Check against `06_DATA_SCHEMA.md`:
- [ ] Profile fields correct
- [ ] Template fields correct
- [ ] File fields correct
- [ ] TypeScript types match database schema
- [ ] Validation schemas match database schema
- [ ] File storage strategy consistent (no duplication)
- [ ] NO reference to `customers` table or `customer_id` foreign key

#### F. Tests and verification quality
For every non-trivial task:
- [ ] Does it have tests in the same Task?
- [ ] If not, does it include a valid explicit Rule 7 exemption?
- [ ] Does the test actually validate the claimed behavior?
- [ ] Does it depend on timing, random chance, or external services in a
      flaky way?
- [ ] Does it clean up test data?
- [ ] Does it avoid modifying real production data?
- [ ] Does it distinguish static implementation from actual runtime
      verification?

#### G. Cross-task composition/regression risks
Check especially:
- [ ] Cumulative database migrations
- [ ] Cumulative API route additions
- [ ] Cumulative component additions
- [ ] Whether a later task accidentally replaces logic added by an earlier task
- [ ] Whether database tables used by later tasks have been introduced first
- [ ] Whether test data remains compatible after later schema changes

#### H. UI/UX and Accessibility
Check against design requirements:
- [ ] Components use shadcn/ui correctly
- [ ] Responsive design is implemented
- [ ] Accessibility attributes are correct
- [ ] Form validation is user-friendly
- [ ] Error messages are clear
- [ ] Loading states are handled
- [ ] Mobile responsiveness is tested

#### I. Security & Privacy
- [ ] Environment variables are required (no hardcoded secrets)
- [ ] RLS policies protect data
- [ ] Authentication is required for write operations
- [ ] Public share links are read-only and expire
- [ ] Input validation is documented

#### J. Performance & Scalability
- [ ] Database indexes are specified
- [ ] Thumbnail generation is documented
- [ ] Pagination strategy is defined
- [ ] Caching strategy is mentioned
- [ ] File size limits are specified

### Phase 1 Output

Create directory: `AI_DOCS/REPAIR_PROMPTS/`

Generate these files:
```
AI_DOCS/REPAIR_PROMPTS/
├── 00_REPAIR_PROMPTS_INDEX.md
├── 01_phase_01_setup_repair.md
├── 02_phase_02_database_repair.md
├── 03_phase_03_auth_repair.md
├── 04_phase_04_crud_upload_repair.md
├── 05_phase_05_search_repair.md
├── 06_phase_06_share_links_repair.md
├── 07_phase_07_testing_deploy_repair.md
└── 08_global_cross_cutting_repairs.md
```

If a Phase has no confirmed issue, still create its file but state:
```text
No confirmed repair required from static audit.
This prompt is audit-only and must not modify any file.
```

### Format Required for Every Generated Repair Prompt

Every generated Markdown repair prompt must use this exact structure:

```markdown
# Repair Prompt — [Topic or Phase]

## Purpose
[Explain exactly what is being repaired.]

## Audit Evidence
[List exact source files and exact contradictions or risks found.
Do not invent issues. Use quotes or concise references.]

## Mandatory Reading
[List every file that must be read before editing.]

## Required User Decision, If Any
[If no user decision is needed, write: None.]

## Allowed Files
[List every file allowed to be modified by this repair prompt.]

## Forbidden Actions
[List what must not be changed.]

## Required Changes
[Numbered, precise changes.]

## Compatibility Requirements
[ADRs, Rules, Data Schema, test requirements, path conventions.]

## Verification Boundaries
[State what can be statically checked and what must later be tested.]

## Acceptance Criteria
[Checklist.]

## Required Final Report
[Implementation Mode report requirements.]
```

Every generated repair prompt must additionally contain:
- A statement that existing files must be read before modification.
- A statement that no runtime verification may be claimed without actual
  execution.
- A statement that CURRENT_TASK and CHANGELOG may only be changed according
  to Rules 6.4 and 6.5.
- A restriction preventing unrelated application implementation.

### Phase 1 Final Report

After creating all repair prompts, provide a final report in Review/Planning
Mode containing:
1. Total number of files inspected.
2. Total number of repair prompts generated.
3. A Phase-by-Phase issue summary table.
4. A severity table:
   - Critical;
   - High;
   - Medium;
   - Low;
   - Needs runtime verification.
5. A list of all user decisions required before repair execution.
6. The recommended execution order of generated prompts.
7. Explicit confirmation that:
   - no existing documentation was modified;
   - no application file was modified;
   - no CURRENT_TASK file was modified;
   - no CHANGELOG file was modified;
   - no runtime verification was claimed.

Do not apply any generated repair prompt in this audit task.

---

## PHASE 2 — Controlled Execution

### Trigger
User says:
```text
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/[filename].md
```

### Rules
- Execute EXACTLY ONE named repair prompt per user command.
- NEVER auto-advance to the next prompt.
- NEVER execute multiple prompts in one response.
- NEVER modify files outside the named prompt's Allowed Files.
- NEVER perform "obvious extra fixes" beyond the named prompt's scope.
- If the prompt requires an unresolved user decision → Clarification Mode.
- Read all Mandatory Reading files BEFORE any modification.
- After completion, STOP and wait for next user command.

### Pre-Execution Validation
Before modifying anything, verify:
- [ ] Named prompt file exists.
- [ ] Named prompt is listed in `00_REPAIR_PROMPTS_INDEX.md`.
- [ ] No unresolved Required User Decision.
- [ ] All target files are in Allowed Files.
- [ ] No target file is forbidden by `01_RULES.md`.
- [ ] All Mandatory Reading files have been read.
- [ ] Prompt does not require CURRENT_TASK/CHANGELOG changes without
      Rules 6.4/6.5.

If validation fails → Clarification Mode. State the exact blocker.

### Mandatory Clarification Conditions
STOP and ask the user if:
- A tech stack change not explicitly confirmed by the user.
- A database schema change not explicitly approved by the user.
- A new ADR.
- A change to a locked ADR.
- A change to `CURRENT_TASK.md` before Rule 6.5 requirements are met.
- A CHANGELOG entry before Rule 6.4 verification exists.
- Moving/deleting files not explicitly authorized.
- Editing a file not listed in Allowed Files.
- A runtime test that you cannot execute and the user has not executed.

Never guess.

### Implementation Rules
When a repair prompt is valid and fully authorized:
- Use Implementation Mode.
- Read the current contents of all existing target files first.
- Apply the minimal targeted changes required by the selected repair prompt.
- Preserve unrelated text and prior decisions.
- Do not rewrite a full existing file unless the selected repair prompt
  explicitly requires a full rewrite.
- Use diffs for modified existing files.
- Use full content only for newly created files.
- Do not claim tests passed unless they were actually run.
- Do not append to CHANGELOG unless all Rule 6.4 conditions are satisfied.
- Do not transition CURRENT_TASK unless all Rule 6.5 conditions are satisfied.

### Post-Execution Report Format
For every executed repair prompt, report in this exact order:

```markdown
## Mode
Implementation Mode / Clarification Mode

## Selected Repair Prompt
[Exact path]

## Summary
[One paragraph]

## Files Inspected
[List]

## Files Created
[Path + full content]

## Files Modified
[Path + diff summary]

## Files NOT Modified (inspected but out of scope)
[List + reason]

## Verification
- Static documentation verification: [result]
- Actual runtime verification: [performed / not performed / user required]

## Acceptance Criteria
[Each: Met / Implemented-not-verified / Blocked]

## Known Limitations / Next User Action
[What remains]
```

End every execution with:
```text
Repair prompt complete. No further prompt has been executed.
Waiting for next explicit user command.
```

---

## PHASE 3 — Final Verification

### Trigger
User says:
```text
EXECUTE: FINAL VERIFICATION — Confirm all repairs applied
```

### Actions
- Re-read every file modified by any executed repair prompt.
- Re-run the full audit checklist (A through J) for every Phase.
- Confirm no NEW contradictions were introduced by repairs.
- Confirm all ADRs are respected.
- Produce a Final Compliance Matrix.
- Confirm if the project is ready for Phase 1 implementation.

### Final Report
```markdown
## Final Compliance Matrix
| Phase | Tasks | ADR Compliant | Tests Valid | Issues Remaining |
|-------|-------|---------------|-------------|------------------|
| 01    |       |               |             |                  |
| 02    |       |               |             |                  |
| 03    |       |               |             |                  |
| 04    |       |               |             |                  |
| 05    |       |               |             |                  |
| 06    |       |               |             |                  |
| 07    |       |               |             |                  |

## Remaining User Decisions
[List any still-pending clarifications]

## Ready for Phase 1 Implementation?
[Yes / No — with exact blockers if No]
```

End with:
```text
Final verification complete. All documentation is consistent.
Project is ready for Phase 1 implementation.
```

---

## Execution Commands

### Phase 1
```text
EXECUTE: MASTER AUDIT — Generate all repair prompts
```

### Phase 2 (Execute ONE by ONE)
```text
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/01_phase_01_setup_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/02_phase_02_database_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/03_phase_03_auth_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/04_phase_04_crud_upload_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/05_phase_05_search_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/06_phase_06_share_links_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/07_phase_07_testing_deploy_repair.md
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/08_global_cross_cutting_repairs.md
```

### Phase 3
```text
EXECUTE: FINAL VERIFICATION — Confirm all repairs applied
```

---

## Completion Statements

After Phase 1:
```text
Audit complete. [N] repair prompts generated in AI_DOCS/REPAIR_PROMPTS/.
No existing file was modified. Awaiting user execution commands.
```

After each Phase 2 execution:
```text
Repair prompt complete. No further prompt has been executed.
Waiting for next explicit user command.
```

After Phase 3:
```text
Final verification complete. All documentation is consistent.
Project is ready for Phase 1 implementation.
```

---
