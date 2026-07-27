# Repair Prompt — API Response Uses `any` Type

## Purpose
Replace the `any` type in the API response format with a proper generic type, aligning with Rule 9 (no `any` without justification).

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 18 (Medium).

`docs/05_TECH_SPEC.md` defines: `{ success: boolean, data?: any, error?: string }`.
`docs/01_RULES.md` Rule 9: "No `any` type unless absolutely necessary with justification."

## Mandatory Reading
- `docs/05_TECH_SPEC.md`
- `docs/01_RULES.md` (Rule 9)

## Required User Decision, If Any
None. The fix is a standard TypeScript generic.

## Allowed Files
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not modify application code.
- Do not add new documentation sections unrelated to the API type.

## Required Changes
1. Replace `any` in the API response format with a generic type parameter:
   ```
   ApiResponse<T> = { success: boolean, data?: T, error?: string }
   ```
2. Document that all API routes should use `ApiResponse<T>` with a specific type for `T`.
3. The generic is self-justifying and no special `any` exception is needed.

## Compatibility Requirements
- Must maintain the same structure (success, data, error fields).
- Must work with both single objects and arrays.

## Acceptance Criteria
- [ ] `any` is removed from the API response format.
- [ ] Generic type parameter is used instead.
- [ ] The change is consistent with Rule 9.

## Required Final Report
Standard Implementation Mode report.
