---

## 📄 فایل واحد: `BOX_DIELINE_MANAGER_COMPLETE_V2.md`

```markdown
# Box Dieline Manager — Complete Project Package (V2)
## All Documentation, Rules, Schema, and Prompts in One File

**Project:** Box Dieline Manager (Die Cut Library)  
**Purpose:** Web application for managing and searching box cutting templates  
**Target Users:** Box manufacturing company (internal use + customer sharing)  
**Tech Stack:** Next.js 14 + TypeScript + Supabase + Vercel  
**Status:** Planning phase, ready for implementation

---

# ═══════════════════════════════════════════════════════
# PART 1: PROJECT RULES (01_RULES.md)
# ═══════════════════════════════════════════════════════

# Project Rules — Non-Negotiable

This file has the highest priority. If any other document contradicts it,
this file wins, and the AI must point out the conflict instead of proceeding.

## 1. Tech Stack Lock
- Framework: Next.js 14 (App Router) — locked in 05_TECH_SPEC.md and ADR-002
- Language: TypeScript only (strict mode)
- Database: Supabase PostgreSQL — locked in ADR-003
- Storage: Supabase Storage — locked in ADR-004
- Authentication: Supabase Auth — locked in ADR-005
- Styling: Tailwind CSS + shadcn/ui
- Deployment: Vercel
- Do not introduce new major dependencies without a new ADR.

## 2. Scope Control (MVP — Minimal)
The MVP is intentionally small. Implement ONLY what CURRENT_TASK.md describes.

**MVP includes:**
- Admin login (single user, no roles yet)
- Template registration form with file uploads
- Search by dimensions (exact + near-match)
- Template view page with file downloads
- Public share link (no login required)

**MVP explicitly excludes (post-MVP only):**
- Multiple user roles (operator, guest)
- Customer management table
- Favorites / usage history
- OCR / AI dimension detection
- Visual search
- Analytics dashboard

New ideas are recorded as a proposal in Review/Planning Mode.

## 3. One Task Per Request
- Each AI response addresses exactly one Task file from AI_DOCS/PARTS/.
- Code belonging to other tasks must not be modified unless that file
  appears in the current task's "Allowed Files" list.

## 4. File and Path Conventions
- Repository root: `box-dieline-manager/`
- Next.js app directory: `app/`
- Components: `components/[feature]/[Component].tsx`
- API routes: `app/api/[route]/route.ts`
- Utils: `lib/[name].ts`
- Types: `types/[name].ts`
- Tests: `__tests__/[feature]/[name].test.ts`

## 5. Naming Conventions
- Files: `kebab-case` (e.g., `user-profile.tsx`)
- Components: `PascalCase` (e.g., `UserProfile`)
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Database tables: `snake_case`
- API routes: `kebab-case`

## 6. Response Modes
Every AI response must declare its mode:
- **6.1 Clarification Mode** — when task is ambiguous
- **6.2 Review/Planning Mode** — for critique, not implementation
- **6.3 Implementation Mode** — only when task is fully specified

## 7. Testing Discipline
- Every non-trivial feature must have tests in the same Task.
- No test may touch the real production database.
- Use test-specific database or mocks.
- Use Supabase local emulator for integration tests.

## 8. Security Rules
- Never hardcode secrets (API keys, DB passwords).
- Use environment variables via `.env.local`.
- All user input must be validated and sanitized.
- Authentication required for all write operations.
- RLS (Row Level Security) enabled on ALL tables.
- Public share links must be read-only and expire after 7 days.

## 9. Code Quality
- TypeScript strict mode enabled.
- ESLint + Prettier configured and enforced.
- No `any` type unless absolutely necessary with justification.
- Functions must have explicit return types.

## 10. Deployment
- Production deployment only via authorized task.
- Never deploy directly from local machine without CI/CD.

## 11. Git Discipline
- Every change must be committed with a clear message.
- No commit without testing.
- After every merge, run regression tests.
- Project is "Production Ready" only after all Phase 7 tests pass.

## 12. Ambiguity Policy
If a user request conflicts with 01_RULES.md, 02_PROJECT_GOAL.md, or
09_DECISIONS.md, the AI must use Clarification Mode. It must never
silently choose an interpretation and proceed.

## 13. Database Rules
- All tables must have `id` (UUID), `created_at`, `updated_at` columns.
- Use UUID for primary keys.
- Soft delete via `deleted_at` column where applicable.
- Foreign keys must have proper indexes.
- RLS enabled on ALL tables — no exceptions.
- File metadata stored ONLY in `files` table, never duplicated in
  other tables (no `image_url`, `pdf_url` in `templates`).
- User profile data stored in `profiles` table, NOT `users`
  (to avoid conflict with Supabase `auth.users`).

## 14. File Storage Rules
- Each template can have multiple files (image, PDF, AI, CDR).
- All files stored in Supabase Storage.
- File metadata (name, type, size, path) stored in `files` table.
- Templates reference files via `template_id` foreign key.
- Thumbnails generated automatically on upload (max 400x400).
- Original images preserved for download.

---

# ═══════════════════════════════════════════════════════
# PART 2: PROJECT GOAL (02_PROJECT_GOAL.md)
# ═══════════════════════════════════════════════════════

# Project Goal — Box Dieline Manager (MVP)

## Working Title
Box Dieline Manager (Die Cut Library)

## Purpose
Build a web application for managing and quickly searching box cutting
templates (dielines) for a box manufacturing company. The ultimate goal
is to become the central, searchable archive of all templates.

## Core User Story
When a customer says:
> "I need a box 18×12×5."

The system should display all templates close to this size in seconds,
not require manual browsing of thousands of photos.

## MVP Feature List (Minimal)
- [ ] Admin login (single user, email/password)
- [ ] Dashboard: total templates count, recent additions
- [ ] Template registration form with file uploads
  - Image (auto-generate thumbnail)
  - PDF
  - AI/CDR files
- [ ] Quick search by dimensions (18×12×5)
- [ ] Near-match search (±2 units tolerance)
- [ ] Template view page with all details
- [ ] Download files (PDF, AI, CDR)
- [ ] Public share link (no login required, 7-day expiration)
- [ ] Mobile responsive (for factory use)

## Explicitly Out of Scope for MVP
- Multiple user roles (operator, guest)
- Customer management table
- Favorites / bookmarks
- Usage history tracking
- OCR on template images
- AI-based dimension detection
- Visual search (upload image → find similar)
- Analytics dashboard
- Advanced filters (by customer, material, date)

## MVP Success Criterion
An admin can:
1. Log in with email/password
2. Register a new template with all files
3. Search for a template by dimensions
4. View the template with all details
5. Download the associated files
6. Generate a public share link
7. Share the link with a customer (no login required)

## Database Tables (MVP)
- `profiles` — admin user (references auth.users)
- `templates` — template metadata (NO file URLs)
- `files` — all file metadata (image, thumbnail, PDF, AI, CDR)

## Build Order Rationale
Database must exist before API can query it. API must exist before
frontend can display data. Authentication must exist before protected
routes. File upload must exist before search can show thumbnails.
Search must exist before public share links can be tested.

---

# ═══════════════════════════════════════════════════════
# PART 3: TECHNICAL SPECIFICATION (05_TECH_SPEC.md)
# ═══════════════════════════════════════════════════════

# Technical Specification

## Tech Stack (LOCKED — set by TASK-01-00)
- **Framework:** Next.js 14.2.x (App Router)
- **Language:** TypeScript 5.x (strict mode)
- **Styling:** Tailwind CSS 3.x + shadcn/ui
- **Database:** Supabase PostgreSQL
- **Storage:** Supabase Storage
- **Authentication:** Supabase Auth
- **Deployment:** Vercel
- **Node.js:** 20.x LTS

## Project Structure
```
box-dieline-manager/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Auth routes group
│   ├── (dashboard)/       # Dashboard routes group
│   ├── api/               # API routes
│   │   ├── templates/     # Template CRUD + search
│   │   └── upload/        # File upload endpoints
│   └── layout.tsx
├── components/            # Reusable components
│   ├── ui/               # Base UI components (shadcn)
│   ├── templates/        # Template-specific components
│   └── forms/            # Form components
├── lib/                  # Utilities and helpers
│   ├── supabase/        # Supabase client
│   ├── utils/           # General utilities
│   └── validations/     # Zod schemas
├── hooks/               # React hooks
├── types/               # TypeScript type definitions
├── public/              # Static assets
└── supabase/            # Supabase migrations
```

**Note:** The `api/customers/` route is NOT part of MVP. Customer management
is explicitly out of scope and will be added in post-MVP phases if needed.

## Database Schema Rules
- All tables must have `id` (UUID), `created_at`, `updated_at` columns.
- Use UUID for primary keys.
- Soft delete via `deleted_at` column where applicable.
- Foreign keys must have proper indexes.
- Row Level Security (RLS) enabled on all tables.

## API Design
- RESTful endpoints under `/api/`
- Response format: `{ success: boolean, data?: any, error?: string }`
- Pagination: cursor-based for large datasets
- Rate limiting on all public endpoints
- Authentication required for all write operations

## Performance Targets
- Lighthouse score: > 90 on all metrics
- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Bundle size: < 200KB (initial load)
- Image optimization: WebP format, lazy loading

## File Upload Rules
- Maximum file size: 50MB per file
- Allowed formats:
  - Images: JPG, PNG, WebP
  - Documents: PDF
  - Design files: AI, CDR
- Files stored in Supabase Storage with organized folder structure
- File metadata stored in `files` table

## Search Implementation
- Full-text search using PostgreSQL `tsvector`
- Dimension search: exact match + near-match algorithm
- Near-match threshold: configurable (default ±2 units)
- Search results ranked by relevance

## Authentication & Authorization
- Single admin user for MVP (no roles yet)
- Supabase Auth with email/password
- Session management via Supabase cookies
- RLS policies protect all data

---

# ═══════════════════════════════════════════════════════
# PART 4: DATA SCHEMA (06_DATA_SCHEMA.md)
# ═══════════════════════════════════════════════════════

# Data Schema — Database Structure (MVP)

## Core Tables

### 1. profiles (NOT users — avoids conflict with auth.users)
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS Policies
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

```

### 2. templates (NO file URLs — files are in separate table)
```sql
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  length DECIMAL(10,2) NOT NULL,
  width DECIMAL(10,2) NOT NULL,
  height DECIMAL(10,2) NOT NULL,
  box_type TEXT NOT NULL,
  material TEXT,
  description TEXT,
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'archived')),
  tags TEXT[] DEFAULT '{}',
  public_share_token TEXT UNIQUE,  -- for public links
  share_expires_at TIMESTAMPTZ,    -- link expiration
  created_by UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);

-- Indexes
CREATE INDEX idx_templates_code ON templates(code);
CREATE INDEX idx_templates_name ON templates(name);
CREATE INDEX idx_templates_dimensions ON templates(length, width, height);
CREATE INDEX idx_templates_status ON templates(status);
CREATE INDEX idx_templates_tags ON templates USING GIN(tags);
CREATE INDEX idx_templates_share_token ON templates(public_share_token);

-- Full-text search (with fallback for Persian)
ALTER TABLE templates ADD COLUMN search_vector tsvector;

CREATE OR REPLACE FUNCTION templates_search_vector_update() RETURNS TRIGGER AS $$
BEGIN
  -- Try Persian config, fallback to simple if not available
  BEGIN
    NEW.search_vector := 
      setweight(to_tsvector('persian', COALESCE(NEW.code, '')), 'A') ||
      setweight(to_tsvector('persian', COALESCE(NEW.name, '')), 'B') ||
      setweight(to_tsvector('persian', COALESCE(NEW.box_type, '')), 'C') ||
      setweight(to_tsvector('persian', COALESCE(NEW.material, '')), 'D');
  EXCEPTION WHEN OTHERS THEN
    -- Fallback to simple config
    NEW.search_vector := 
      setweight(to_tsvector('simple', COALESCE(NEW.code, '')), 'A') ||
      setweight(to_tsvector('simple', COALESCE(NEW.name, '')), 'B') ||
      setweight(to_tsvector('simple', COALESCE(NEW.box_type, '')), 'C') ||
      setweight(to_tsvector('simple', COALESCE(NEW.material, '')), 'D');
  END;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER templates_search_vector_trigger
  BEFORE INSERT OR UPDATE ON templates
  FOR EACH ROW EXECUTE FUNCTION templates_search_vector_update();

CREATE INDEX idx_templates_search ON templates USING GIN(search_vector);

-- RLS Policies
ALTER TABLE templates ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Authenticated users can view templates"
  ON templates FOR SELECT
  USING (auth.uid() IS NOT NULL AND deleted_at IS NULL);

CREATE POLICY "Authenticated users can insert templates"
  ON templates FOR INSERT
  WITH CHECK (auth.uid() IS NOT NULL);

CREATE POLICY "Authenticated users can update templates"
  ON templates FOR UPDATE
  USING (auth.uid() IS NOT NULL);

CREATE POLICY "Public can view via share token"
  ON templates FOR SELECT
  USING (
    public_share_token IS NOT NULL 
    AND share_expires_at > NOW()
    AND deleted_at IS NULL
  );
```

### 3. files (ALL file metadata here, NOT in templates)
```sql
CREATE TABLE files (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
  file_type TEXT NOT NULL CHECK (file_type IN ('image', 'thumbnail', 'pdf', 'ai', 'cdr')),
  file_name TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type TEXT NOT NULL,
  storage_path TEXT NOT NULL,  -- path in Supabase Storage
  storage_url TEXT NOT NULL,   -- public URL
  uploaded_by UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_files_template ON files(template_id);
CREATE INDEX idx_files_type ON files(file_type);

-- RLS Policies
ALTER TABLE files ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Authenticated users can view files"
  ON files FOR SELECT
  USING (auth.uid() IS NOT NULL);

CREATE POLICY "Authenticated users can insert files"
  ON files FOR INSERT
  WITH CHECK (auth.uid() IS NOT NULL);

CREATE POLICY "Authenticated users can delete files"
  ON files FOR DELETE
  USING (auth.uid() IS NOT NULL);

CREATE POLICY "Public can view files via template share token"
  ON files FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM templates
      WHERE templates.id = files.template_id
      AND templates.public_share_token IS NOT NULL
      AND templates.share_expires_at > NOW()
      AND templates.deleted_at IS NULL
    )
  );
```

## TypeScript Types

```typescript
// types/database.ts
export interface Profile {
  id: string;
  email: string;
  full_name: string;
  is_active: boolean;
  created_at: string;
  updated_at: string;
}

export interface Template {
  id: string;
  code: string;
  name: string;
  length: number;
  width: number;
  height: number;
  box_type: string;
  material?: string;
  description?: string;
  status: 'active' | 'inactive' | 'archived';
  tags: string[];
  public_share_token?: string;
  share_expires_at?: string;
  created_by?: string;
  created_at: string;
  updated_at: string;
  deleted_at?: string;
}

export interface File {
  id: string;
  template_id: string;
  file_type: 'image' | 'thumbnail' | 'pdf' | 'ai' | 'cdr';
  file_name: string;
  file_size: number;
  mime_type: string;
  storage_path: string;
  storage_url: string;
  uploaded_by?: string;
  created_at: string;
}

// Helper type for template with files
export interface TemplateWithFiles extends Template {
  files: File[];
  thumbnail_url?: string;
}
```

## Validation Schemas (Zod)

```typescript
// lib/validations/template.ts
import { z } from 'zod';

export const templateSchema = z.object({
  code: z.string().min(1, 'Code is required'),
  name: z.string().min(1, 'Name is required'),
  length: z.number().positive('Length must be positive'),
  width: z.number().positive('Width must be positive'),
  height: z.number().positive('Height must be positive'),
  box_type: z.string().min(1, 'Box type is required'),
  material: z.string().optional(),
  description: z.string().optional(),
  status: z.enum(['active', 'inactive', 'archived']),
  tags: z.array(z.string()),
});

export const fileUploadSchema = z.object({
  file_type: z.enum(['image', 'thumbnail', 'pdf', 'ai', 'cdr']),
  file_name: z.string().min(1),
  file_size: z.number().positive(),
  mime_type: z.string().min(1),
});

// Dimension search schema
export const dimensionSearchSchema = z.object({
  length: z.number().optional(),
  width: z.number().optional(),
  height: z.number().optional(),
  tolerance: z.number().min(0).max(10).default(2), // ±2 units for near-match
});
```

## Near-Match Search Algorithm

```typescript
// lib/utils/dimension-search.ts
export function findNearMatches(
  templates: Template[],
  searchLength: number,
  searchWidth: number,
  searchHeight: number,
  tolerance: number = 2
): Template[] {
  return templates
    .map(template => {
      const lengthDiff = Math.abs(template.length - searchLength);
      const widthDiff = Math.abs(template.width - searchWidth);
      const heightDiff = Math.abs(template.height - searchHeight);
      const totalDiff = lengthDiff + widthDiff + heightDiff;
      
      // Only include if within tolerance for each dimension
      if (lengthDiff <= tolerance && widthDiff <= tolerance && heightDiff <= tolerance) {
        return { template, totalDiff };
      }
      return null;
    })
    .filter((item): item is { template: Template; totalDiff: number } => item !== null)
    .sort((a, b) => a.totalDiff - b.totalDiff)
    .map(item => item.template)
    .slice(0, 10); // Return top 10 matches
}
```

---

# ═══════════════════════════════════════════════════════
# PART 5: ARCHITECTURE DECISIONS (09_DECISIONS.md)
# ═══════════════════════════════════════════════════════

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

# ═══════════════════════════════════════════════════════
# PART 6: PROJECT PHASES AND TASKS
# ═══════════════════════════════════════════════════════

# Project Phases (MVP — 7 Phases, 27 Tasks)

## Phase 1: Setup & Foundation (4 Tasks)
- task_01_00_lock_tech_stack.md
- task_01_01_project_initialization.md
- task_01_02_dev_environment.md
- task_01_03_git_setup.md

## Phase 2: Database & Schema (4 Tasks)
- task_02_01_supabase_setup.md
- task_02_02_profiles_table.md
- task_02_03_templates_table.md
- task_02_04_files_table.md

## Phase 3: Authentication (3 Tasks)
- task_03_01_supabase_auth_setup.md
- task_03_02_admin_login_page.md
- task_03_03_protected_routes.md

## Phase 4: Template CRUD & File Upload (6 Tasks)
- task_04_01_template_form.md
- task_04_02_image_upload_with_thumbnail.md
- task_04_03_pdf_upload.md
- task_04_04_ai_cdr_upload.md
- task_04_05_template_list_page.md
- task_04_06_template_detail_page.md

## Phase 5: Search & Filters (4 Tasks)
- task_05_01_dimension_search.md
- task_05_02_near_match_algorithm.md
- task_05_03_full_text_search.md
- task_05_04_search_results_page.md

## Phase 6: Public Share Links (3 Tasks)
- task_06_01_share_token_generation.md
- task_06_02_public_view_page.md
- task_06_03_link_expiration.md

## Phase 7: Testing & Deployment (3 Tasks)
- task_07_01_e2e_tests.md
- task_07_02_production_deploy.md
- task_07_03_documentation.md

**Total: 27 tasks across 7 phases (MVP).**

---

# ═══════════════════════════════════════════════════════
# PART 7: MASTER AUDIT PROMPT
# ═══════════════════════════════════════════════════════

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

# ═══════════════════════════════════════════════════════
# END OF COMPLETE PACKAGE V2
# ═══════════════════════════════════════════════════════
```

---

## ✅ تغییرات اعمال‌شده در V2

| بخش | تغییر |
|-----|-------|
| `05_TECH_SPEC.md` — ساختار پوشه‌ها | حذف `api/customers/` از ساختار |
| `05_TECH_SPEC.md` — Note جدید | اضافه شدن یادآوری صریح که `api/customers/` خارج از MVP است |
| `09_DECISIONS.md` | اضافه شدن **ADR-015** برای قفل‌کردن تصمیم "بدون جدول customers در MVP" |
| `Master Audit Prompt` — بخش D | اضافه شدن چک صریح برای عدم ارجاع به `api/customers/` یا `customers` table |
| `Master Audit Prompt` — بخش E | اضافه شدن چک برای عدم وجود `customer_id` foreign key |

---

## 🎯 نحوه استفاده

### مرحله ۱: ذخیره فایل
فایل بالا را به‌عنوان `BOX_DIELINE_MANAGER_COMPLETE_V2.md` ذخیره کنید.

### مرحله ۲: ارسال به AI Agent
به Agent بگویید:

```
این فایل کامل پروژه Box Dieline Manager V2 است. لطفاً ابتدا Phase 1 را اجرا کن:

EXECUTE: MASTER AUDIT — Generate all repair prompts

فقط بررسی کن و گزارش بده. هیچ فایلی را تغییر نده.
```

### مرحله ۳: اجرای پارت پارت
بعد از دریافت گزارش، یکی یکی دستور بدهید:

```text
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/01_phase_01_setup_repair.md
```

Agent فقط همین یکی را اجرا می‌کند و متوقف می‌شود.

### مرحله ۴: Final Verification
بعد از اجرای همه پرامپت‌ها:

```text
EXECUTE: FINAL VERIFICATION — Confirm all repairs applied
```

---

## 🎉 نتیجه

این نسخه V2:
- ✅ **ناسازگاری `api/customers/`** برطرف شد
- ✅ **ADR-015** برای قفل‌کردن تصمیم اضافه شد
- ✅ **چک‌های اضافی** در Master Audit Prompt اضافه شد
- ✅ تمام ۷ بخش اصلی حفظ شدند
- ✅ آماده برای اجرا توسط AI Agent

آماده‌اید شروع کنید؟ 🚀