# Architecture Decision Records (ADRs)

Decisions here are LOCKED and append-only. An ADR's Decision/Reason text
is never silently repurposed for an unrelated topic.

## ADR-001 — Frontend Framework
**Status:** Accepted
**Decision:** Next.js 14 with App Router
**Reason:** SSR/SSG flexibility, built-in routing, TypeScript support,
Vercel deployment simplicity.

## ADR-002 — TypeScript Strict Mode
**Status:** Accepted
**Decision:** TypeScript with strict mode enabled
**Reason:** Type safety, better IDE support, fewer runtime errors.

## ADR-003 — Database Choice
**Status:** Accepted
**Decision:** Supabase PostgreSQL
**Reason:** Managed service, built-in auth, real-time subscriptions,
TypeScript support, generous free tier.

## ADR-004 — Storage Solution
**Status:** Accepted
**Decision:** Supabase Storage
**Reason:** Integrated with database, built-in CDN, RLS policies.

## ADR-005 — Authentication
**Status:** Accepted
**Decision:** Supabase Auth (email/password only for MVP)
**Reason:** Integrated with database RLS, simple for single-admin MVP.
Post-MVP: Add OAuth providers if needed.

## ADR-006 — Styling Approach
**Status:** Accepted
**Decision:** Tailwind CSS + shadcn/ui
**Reason:** Utility-first, small bundle, accessible components.

## ADR-007 — File Storage Strategy
**Status:** Accepted
**Decision:** Separate `files` table, NOT file URLs in `templates`
**Reason:** Normalization, lifecycle management, metadata storage.
Templates reference files via `template_id` foreign key.

## ADR-008 — User Profile Table
**Status:** Accepted
**Decision:** Use `profiles` table, NOT `users`
**Reason:** Avoids conflict with Supabase `auth.users`. Standard pattern.

## ADR-009 — Thumbnail Strategy
**Status:** Accepted
**Decision:** Generate thumbnails on upload, separate from originals
**Reason:** Performance for list views with thousands of templates.
Thumbnails max 400x400, originals preserved.

## ADR-010 — Public Share Links
**Status:** Accepted
**Decision:** Token-based public links with 7-day expiration
**Reason:** Allows sharing templates with customers without login.
Read-only access, expires automatically.

## ADR-011 — MVP Scope (Minimal)
**Status:** Accepted
**Decision:** MVP includes ONLY:
- Single admin user (no roles)
- Template CRUD
- File uploads (image, PDF, AI, CDR)
- Dimension search (exact + near-match)
- Public share links

MVP excludes:
- Multiple roles (operator, guest)
- Customer management
- Favorites / usage history
- OCR / AI features
- Analytics

**Reason:** Get to usable version quickly. Add features post-MVP based
on actual usage patterns.

## ADR-012 — 7-Phase Structure (MVP)
**Status:** Accepted
**Decision:** MVP structured into 7 Phases:
1. Setup & Foundation
2. Database & Schema
3. Authentication (single admin)
4. Template CRUD & File Upload
5. Search & Filters
6. Public Share Links
7. Testing & Deployment

**Reason:** Minimal viable path to production. Post-MVP phases added
as needed.

## ADR-013 — Full-Text Search Fallback
**Status:** Accepted
**Decision:** Use `to_tsvector('persian')` with fallback to `'simple'`
**Reason:** Persian text search config may not be available in all
Supabase instances. Fallback ensures search always works.

## ADR-014 — RLS on All Tables
**Status:** Accepted
**Decision:** Row Level Security enabled on ALL tables
**Reason:** Security best practice. Prevents unauthorized access even
if API routes have bugs.

## ADR-015 — No Customer Table in MVP
**Status:** Accepted
**Decision:** Customer management is NOT part of MVP.
The `api/customers/` route and `customers` table are deferred to post-MVP.
**Reason:** MVP is intentionally minimal. Customer management adds complexity
without immediate value for a single-admin internal tool.

---
