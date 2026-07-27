# Repair Prompt — Phase 05: Search & Filters

## Purpose
Write the four Phase 5 task files (`task_05_01` through `task_05_04`) with complete, actionable content. Incorporate user decisions on dimension units (selectable, permutation-aware near-match), search fields (description + tags added), and the narrowed exception handler for the search trigger.

## Audit Evidence
**A-J Audit Findings:**

**A (Authority & scope):** Task names match `docs/08_PROJECT_PHASES_AND_TASKS.md` ✓.

**B (Dependency order):** Phase 5 depends on Phase 2 (database with search vector) and Phase 4 (templates with data). Correctly positioned after Phase 4 in ADR-012 ✓.

**C (Next.js/TS Correctness):** Near-match algorithm in `06_DATA_SCHEMA.md` needs permutation-awareness update (user decision from Item 14).

**D (Database/API):**
- **Item 14** (Dimension units): Selectable unit column (`cm`/`mm`/`inch`). Permutation-aware near-match.
- **Item 19** (Search fields): `description` and `tags` now in search vector. `websearch_to_tsquery` recommended.

**E (Schema consistency):** Search vector definition in `06_DATA_SCHEMA.md` must be consistent with the field additions.

**F (Tests):** Search tasks require test data setup and search result assertions. Must document test strategy.

**G (Cross-task):** Phase 5 search results are used by Phase 6 (share links share search results). No negative cross-task impact.

**H (UI/UX):** Search input should support both dimension format ("18×12×5") and keywords. Results should show relevance ranking. Loading and empty states.

**I (Security):** Search on authenticated templates requires auth. Public search (if any) must be read-only.

**J (Performance):** GIN index on `search_vector` already specified. Dimension index on `(length, width, height)` already specified. Pagination must apply to search results (Item 23).

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (search vector, near-match algorithm, indexes)
- `docs/05_TECH_SPEC.md` (Search Implementation)
- `docs/09_DECISIONS.md` (ADR-013)
- `docs/10_EXTERNAL_SECURITY_AUDIT.md` (Items 14, 19, 20, 23)
- Related per-item repair prompts: `14`, `19`, `20`, `23`

## Required User Decision, If Any
None. All search-related decisions resolved.

## Allowed Files
- `AI_DOCS/PARTS/phase_05_search/task_05_01_dimension_search.md`
- `AI_DOCS/PARTS/phase_05_search/task_05_02_near_match_algorithm.md`
- `AI_DOCS/PARTS/phase_05_search/task_05_03_full_text_search.md`
- `AI_DOCS/PARTS/phase_05_search/task_05_04_search_results_page.md`

## Forbidden Actions
- Do NOT implement actual search code.
- Do NOT modify task files from other phases.
- Do NOT modify `docs/06_DATA_SCHEMA.md` in this repair.

## Required Changes
1. Write `task_05_01_dimension_search.md`: Server-side dimension search endpoint (`/api/templates/search?length=18&width=12&height=5`). Document exact match query. Include unit-awareness (compare templates with the same unit only). Pagination with cursor.
2. Write `task_05_02_near_match_algorithm.md`: Permutation-aware near-match. For query (L, W, H), compute all 6 permutations and return templates within tolerance (default ±2). Tolerance is within the stored unit. Document algorithm. Test with sample data.
3. Write `task_05_03_full_text_search.md`: Full-text search using `websearch_to_tsquery('persian', query)`. Fallback to `'simple'` config (ADR-013). Search all 6 fields: code, name, box_type, material, description, tags. Return ranked results.
4. Write `task_05_04_search_results_page.md`: Combined search UI (dimension keywords + text search). Display results with thumbnail, code, name, dimensions. Paginated results with cursor. Loading, empty, and error states. Mobile-responsive per project goal.

## Compatibility Requirements
- ADR-013: Persian full-text search with fallback.
- ADR-012: Phase 5 is after Phase 4.
- Near-match algorithm must be permutation-aware (user decision).
- Search vector includes all 6 fields (user decision).
- Unit is selectable, search compares same-unit only.

## Acceptance Criteria
- [ ] All 4 Phase 5 task files contain complete, actionable instructions.
- [ ] Dimension search endpoint is documented.
- [ ] Near-match algorithm is permutation-aware.
- [ ] Full-text search uses `websearch_to_tsquery`.
- [ ] Search results page is documented with all states.

## Required Final Report
Standard Implementation Mode report as defined in `docs/07_MASTER_AUDIT_PROMPT.md`.

---
*Existing files must be read before modification.*
*No runtime verification may be claimed without actual execution.*
*CURRENT_TASK and CHANGELOG may only be changed according to Rules 6.4 and 6.5.*
*No unrelated application implementation is permitted.*
