# Repair Prompt — Pagination Contract Undefined

## Purpose
Define the pagination contract: cursor mechanism, sort order, default/max limits, and how pagination interacts with dimension search and full-text search.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 23 (Low).

`docs/05_TECH_SPEC.md` states: "Pagination: cursor-based for large datasets."
No further details are specified: cursor basis, sort order, default/max limits, or pagination response format.

## Mandatory Reading
- `docs/05_TECH_SPEC.md`
- `docs/06_DATA_SCHEMA.md`

## Required User Decision, If Any
None. Standard cursor-based pagination pattern.

## Allowed Files
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not implement code.
- Do not change the cursor-based decision (already locked in ADR/tech spec).

## Required Changes
In `docs/05_TECH_SPEC.md`, define:
1. Cursor basis: Use `created_at` + `id` (for uniqueness).
2. Sort order: Default `created_at DESC` (newest first).
3. Default limit: 20 items per page.
4. Max limit: 100 items per page.
5. Response format: `{ data: T[], nextCursor?: string, hasMore: boolean }`.
6. Interaction with search: Pagination cursor works within search results (same cursor basis).
7. Empty/missing cursor: First page.

## Compatibility Requirements
- Must work with both list endpoints and search endpoints.
- Must be implementable with Supabase PostgreSQL queries.

## Acceptance Criteria
- [ ] Cursor basis is documented.
- [ ] Default and max limits are documented.
- [ ] Response format is specified.
- [ ] Search pagination interaction is documented.

## Required Final Report
Standard Implementation Mode report.
