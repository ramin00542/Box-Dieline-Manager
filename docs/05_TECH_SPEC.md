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
