# Repair Prompt — Dimension Units and Axis Order Undefined

## Purpose
Define the unit of measurement (mm, cm, or inch) and the semantic order of length/width/height for all dimension fields, searches, and displays.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 14 (Medium, Needs Decision).

`docs/06_DATA_SCHEMA.md` defines `length DECIMAL(10,2)`, `width DECIMAL(10,2)`, `height DECIMAL(10,2)` without units.
`docs/02_PROJECT_GOAL.md` shows example "18×12×5" without units.
The tolerance of ±2 in the near-match algorithm has no meaningful interpretation without units.
It is also unclear whether "18×12×5" and "12×18×5" are considered different templates or the same (whether order is semantically meaningful or must follow a convention like length ≥ width ≥ height).

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/02_PROJECT_GOAL.md`
- `docs/05_TECH_SPEC.md` (Search Implementation)

## Required User Decision, If Any
**Yes — Resolved.** The user decided:
1. **Unit is selectable**: Add a `unit` column (enum: 'cm', 'mm', 'inch') to the `templates` table, default `'cm'`. The user chooses the unit per template, not globally fixed.
2. **Axis order matters**: "18×12×5" and "12×18×5" are different templates (separate records).
3. **Near-match search is permutation-aware**: When searching by dimensions, the algorithm should consider all permutations of the three search values, not just strict column-to-column matching.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not implement application code.
- Do not change the column types without user approval.

## Required Changes
1. **Schema**: Add a `unit` column to the `templates` table definition in `docs/06_DATA_SCHEMA.md`:
   ```sql
   unit TEXT NOT NULL DEFAULT 'cm' CHECK (unit IN ('cm', 'mm', 'inch'))
   ```
2. **Schema comment**: Add a comment to `length`/`width`/`height` columns noting they are stored as-is (order is semantically meaningful), and the unit is per-record in the `unit` column.
3. **Near-match algorithm**: Update the `findNearMatches()` function in `docs/06_DATA_SCHEMA.md` to be permutation-aware. For a search query (L, W, H), compute the set of all 6 permutations of the search values and return any template whose dimensions fall within tolerance of any permutation. The tolerance should still be configurable (default ±2) but now meaningful within the stored unit.
4. **Tech spec**: Document the permutation-aware search behavior in `docs/05_TECH_SPEC.md` under Search Implementation.
5. **Note on cross-unit search**: Since searches are within the same unit system (no automatic conversion), the near-match algorithm operates on same-unit comparisons only.

## Compatibility Requirements
- Near-match algorithm in `docs/06_DATA_SCHEMA.md` must be consistent with the unit.
- Tolerance value must be meaningful in the chosen unit.

## Acceptance Criteria
- [ ] Unit is documented and unambiguous.
- [ ] Axis order convention is documented.
- [ ] Near-match tolerance is meaningful in context.

## Required Final Report
Standard Implementation Mode report.
