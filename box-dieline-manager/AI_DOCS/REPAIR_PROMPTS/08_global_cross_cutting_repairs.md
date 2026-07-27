# Repair Prompt — Cross-Cutting & Governance Issues

## Purpose
Address governance and cross-cutting issues that span multiple phases or affect the overall project structure. These are not specific to any single implementation phase.

## Audit Evidence
**Cross-Cutting Issues from `docs/10_EXTERNAL_SECURITY_AUDIT.md`:**

- **Item 8** (Rules 6.4/6.5): **BLOCKING**. Master Audit references Rules 6.4 and 6.5 for CURRENT_TASK and CHANGELOG governance, but these rules do not exist in `01_RULES.md`. Must be resolved before any Phase 2 execution.
- **Item 9** (AI_DOCS structure): **FIXED**. Directory structure and placeholder task files have been created.
- **Item 10** (CRUD scope): **RESOLVED**: Full CRUD + soft-delete.
- **Item 18** (API `any` type): Response format in `05_TECH_SPEC.md` uses `any` contradicting Rule 9.
- **Item 22** (Dashboard scope): **RESOLVED**: Dashboard stays in MVP.
- **Item 23** (Pagination contract): Cursor-based pagination basis, limits, and response format undefined.
- **Item 24** (Rate limiting): Tool, limits, and response format undefined.

## Mandatory Reading
- `docs/01_RULES.md` (Rules 6, 9)
- `docs/05_TECH_SPEC.md` (API Design)
- `docs/07_MASTER_AUDIT_PROMPT.md` (all references to Rules 6.4/6.5)
- `docs/09_DECISIONS.md` (ADR-011, ADR-012)
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 8, 18, 22, 23, 24)
- Related per-item repair prompts: `08`, `18`, `22`, `23`, `24`

## Required User Decision, If Any
**Yes — One remaining decision:** Item 10 (CRUD scope) — User must decide whether MVP CRUD is Full CRUD or Create/Read only. All other cross-cutting items have been resolved.

## Allowed Files
- `docs/01_RULES.md`
- `docs/05_TECH_SPEC.md`
- `docs/07_MASTER_AUDIT_PROMPT.md`
- `docs/08_PROJECT_PHASES_AND_TASKS.md`
- `docs/09_DECISIONS.md`
- `docs/02_PROJECT_GOAL.md`

## Forbidden Actions
- Do NOT implement application code.
- Do NOT modify task files in AI_DOCS/PARTS/.
- Do NOT modify CURRENT_TASK.md or CHANGELOG.md.

## Required Changes
**This prompt is a coordination/orchestration layer. All 5 cross-cutting items have dedicated per-item repair prompts with detailed Required Changes. Execute them instead of this prompt directly.**

1. **Fix Rules 6.4/6.5 in `docs/01_RULES.md`** — See detailed per-item prompt: `08_repair_rules_6_4_6_5.md` (BLOCKING — must be first).
2. **Replace `any` type in `docs/05_TECH_SPEC.md`** — See detailed per-item prompt: `18_repair_api_response_any_type.md`.
3. **Define pagination contract in `docs/05_TECH_SPEC.md`** — See detailed per-item prompt: `23_repair_pagination_contract.md`.
4. **Define rate limiting in `docs/05_TECH_SPEC.md`** — See detailed per-item prompt: `24_repair_rate_limiting.md`.
5. **Update Dashboard scope** — See detailed per-item prompt: `22_repair_dashboard_scope.md`.

These 5 per-item prompts are the authoritative, detailed instructions. This cross-cutting prompt exists only to group them and define their execution order.

## Compatibility Requirements
- Rules 6.4/6.5 must be consistent with existing Rules 6.1-6.3.
- API type fix must be consistent with Rule 9 (no `any`).
- Pagination must work with both list and search endpoints.
- Rate limiting must work with Vercel serverless/edge.
- All changes must be non-contradictory with locked ADRs.

## Verification Boundaries
- **Static verification**: All changes are documentation-level. Cross-reference consistency can be verified statically.
- **No runtime verification required**: These are governance and spec changes.

## Acceptance Criteria
- [ ] Rules 6.4 and 6.5 are added to `docs/01_RULES.md`.
- [ ] `any` type is removed from API response format.
- [ ] Pagination contract (cursor, limits, format) is documented.
- [ ] Rate limiting strategy is documented.
- [ ] Dashboard scope is consistently documented across all governance files.
- [ ] All changes are internally consistent with locked ADRs.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
