# Repair Prompt — Storage_url Cached Signed URL Policy

## Purpose
Define the lifecycle policy for the `storage_url` column in the `files` table: when it is set, when it is cleared, and what guarantees (if any) the cached URL provides.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 12 (Medium).

Current schema: `storage_url TEXT` with comment "cached Signed URL (nullable)". ADR-016 says "Signed URLs generated at request time." Storing a time-limited Signed URL without an expiry field or refresh policy creates risk of expired or stale URLs.

## Mandatory Reading
- `docs/06_DATA_SCHEMA.md`
- `docs/09_DECISIONS.md` (ADR-016)

## Required User Decision, If Any
None. The behavior should be documented to match ADR-016.

## Allowed Files
- `docs/06_DATA_SCHEMA.md`

## Forbidden Actions
- Do not remove `storage_url` from the schema.
- Do not modify application code.

## Required Changes
1. Clarify in the schema comment for `storage_url`:
   - That it is a cache layer only — the authoritative access is via `storage_path` + runtime Signed URL generation.
   - That it may be NULL or expired and callers must always fall back to generating a fresh Signed URL.
   - That it has no guaranteed TTL and should not be relied upon for access control.
   - That it should never be returned to anonymous/public users directly (must go through the Signed URL flow in Repair Prompt 04).

## Compatibility Requirements
- ADR-016: Signed URLs at request time.
- Must not be the sole access mechanism.

## Acceptance Criteria
- [ ] The documentation clearly states `storage_url` is a non-authoritative cache.
- [ ] No caller should treat `storage_url` as a reliable long-term access URL.
- [ ] The column is clearly distinguished from the primary `storage_path` approach.

## Required Final Report
Standard Implementation Mode report.
