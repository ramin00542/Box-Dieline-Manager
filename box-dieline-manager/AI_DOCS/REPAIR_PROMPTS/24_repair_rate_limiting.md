# Repair Prompt — Rate Limiting Tool and Limits Not Specified

## Purpose
Define the rate limiting strategy: which tool/service to use, what limits apply to which endpoints, and what HTTP response is returned when limits are exceeded.

## Audit Evidence
Source: `docs/10_EXTERNAL_SECURITY_AUDIT.md` — Item 24 (Low).

`docs/05_TECH_SPEC.md` states: "Rate limiting on all public endpoints."
Not specified: tool/service, rate limit key (IP/token), request count, time window, HTTP response code, or Vercel compatibility.

## Mandatory Reading
- `docs/05_TECH_SPEC.md`
- `docs/01_RULES.md` (Rule 1: no major dependencies without ADR)

## Required User Decision, If Any
None. Recommend Vercel-native or lightweight approach compatible with serverless.

## Allowed Files
- `docs/05_TECH_SPEC.md`

## Forbidden Actions
- Do not add a new dependency without ADR.
- Do not implement code.

## Required Changes
In `docs/05_TECH_SPEC.md`, add Rate Limiting section:
1. **Approach**: Use Vercel's built-in rate limiting (via `@vercel/functions` or Edge Middleware) — no additional service needed.
2. **Scope**: Apply to public endpoints (`/api/share/*`) separately from authenticated endpoints. All authenticated endpoints (including search, templates CRUD, upload) share the same rate limit tier.
3. **Limit**: 10 requests per minute per IP for public endpoints; 100 requests per minute per user for all authenticated endpoints.
4. **Response**: HTTP 429 Too Many Requests with `Retry-After` header.
5. **Key**: IP address for unauthenticated; user ID for authenticated.
6. **Fallback**: If rate limiting service is unavailable, allow request but log warning (fail open for MVP).

## Compatibility Requirements
- Must work with Vercel serverless/edge runtime.
- Must not require a separate Redis or database service for MVP.
- Must not impact authenticated admin usability.

## Acceptance Criteria
- [ ] Rate limiting strategy is documented.
- [ ] Tool, limits, key, and response format are specified.
- [ ] Compatible with Vercel deployment.

## Required Final Report
Standard Implementation Mode report.
