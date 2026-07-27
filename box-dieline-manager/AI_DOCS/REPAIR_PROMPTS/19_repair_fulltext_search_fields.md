# Repair Prompt — Full-Text Search Fields and Query Function Undefined

## Purpose
Clarify which fields are included in the full-text search vector and specify which PostgreSQL search function (plainto_tsquery, websearch_to_tsquery, phraseto_tsquery) should be used for user queries.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 19 (Medium, Needs Decision).

Current `search_vector` includes: `code`, `name`, `box_type`, `material`.
Not included: `description`, `tags`.
No query function specified for converting user input to a search query.
Behavior for Persian, English, and mixed text is not addressed.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md` (search vector and trigger)
- `docs/05_TECH_SPEC.md` (Search Implementation)

## Required User Decision, If Any
**Yes — Resolved.** The user decided:
1. **Both `description` and `tags` MUST be added** to the `search_vector`, in addition to the existing fields (`code`, `name`, `box_type`, `material`).
2. The query function choice (`plainto_tsquery`, `websearch_to_tsquery`, or `phraseto_tsquery`) and case/accent sensitivity remain open for implementation choice — the repair prompt should recommend a default.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not implement application code.
- Do not remove existing search fields without approval.

## Required Changes
1. **Add fields to `search_vector`**: Update the `templates_search_vector_update()` function in `docs/06_DATA_SCHEMA.md` to include `description` and `tags`:
   ```sql
   NEW.search_vector := 
     setweight(to_tsvector('persian', COALESCE(NEW.code, '')), 'A') ||
     setweight(to_tsvector('persian', COALESCE(NEW.name, '')), 'B') ||
     setweight(to_tsvector('persian', COALESCE(NEW.box_type, '')), 'C') ||
     setweight(to_tsvector('persian', COALESCE(NEW.material, '')), 'D') ||
     setweight(to_tsvector('persian', COALESCE(NEW.description, '')), 'D') ||
     setweight(to_tsvector('persian', COALESCE(array_to_string(NEW.tags, ' '), '')), 'D');
   ```
   Same change in the `'simple'` fallback branch.
2. **Recommend query function**: Suggest `websearch_to_tsquery('persian', user_query)` for user-facing search — it supports quoted phrases, negation (`-`), and `OR` logic naturally.
3. **Document the field weights**: 'A' (code) > 'B' (name) > 'C' (box_type) > 'D' (material, description, tags) in `docs/05_TECH_SPEC.md`.
4. **Document mixed-language behavior**: The `'persian'` config handles Persian text; for English terms within Persian text, the parser should still tokenize them. Add a note that `websearch_to_tsquery` handles both naturally.
5. Clarify that `tags` is a `TEXT[]` array and must be converted via `array_to_string()` before being passed to `to_tsvector()`.

## Compatibility Requirements
- ADR-013: Persian fallback to `simple` config.
- The `EXCEPTION WHEN OTHERS` block (Item 20) should be narrowed alongside this change.

## Acceptance Criteria
- [ ] Search fields are explicitly documented.
- [ ] Query function is specified.
- [ ] Mixed-language search behavior is documented.

## Required Final Report
Standard Implementation Mode report.
