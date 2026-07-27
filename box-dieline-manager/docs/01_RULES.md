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
- Dashboard (total template count, recent additions)
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
- Repository root: `box-dieline-manager/` (all file paths in docs/ and AI_DOCS/ are relative to this directory)
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
- **6.4 Verification Rule** — A task is verified only when all its acceptance
  criteria are met, all tests pass, and the changes have been reviewed. Only
  then may `CHANGELOG.md` be updated.
- **6.5 Task Transition Rule** — `CURRENT_TASK.md` may only be updated to
  transition to the next task after the current task's verification (Rule 6.4)
  is complete and the user has explicitly confirmed the transition.

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
- All Storage buckets must be Private. File access is granted only via
  time-limited Signed URLs, never permanent public Storage URLs.

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
- No commit without testing. (Exception: documentation-only changes to docs/ or AI_DOCS/ files do not require runtime tests before commit — static review is sufficient.)
- After every merge, run regression tests.
- Project is "Production Ready" only after all Phase 7 tests pass.

## 12. Ambiguity Policy
If a user request conflicts with 01_RULES.md, 02_PROJECT_GOAL.md, or
09_DECISIONS.md, the AI must use Clarification Mode. It must never
silently choose an interpretation and proceed.

## 13. Database Rules
- All tables must have `id` (UUID), `created_at`, `updated_at` columns,
  except append-only tables (e.g. `files`) where records are never edited
  after creation and only `created_at` is required.
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

## 15. Immediate Git Commit & Push
- Every approved file change (repair prompt execution, task implementation, or any user-authorized modification) must be immediately staged, committed, and pushed to the remote repository.
- Stage only the files changed in the current step — never use blind `git add .`.
- Use a clear, descriptive commit message that matches the specific change.
- If `git push` fails (conflict, authentication error, etc.), stop immediately, display the exact error, and wait for user instruction. Never force-push or take corrective action autonomously.
- This rule applies to changes explicitly requested or approved by the user — not to temporary or experimental files.
- For documentation-only changes (docs/ and AI_DOCS/ files), static review satisfies Rule 11's testing requirement — no separate runtime test is needed before committing.

## 16. Task Activation
- When no active task exists in CURRENT_TASK.md, the user command "START TASK: [path to task file]" causes the agent to:
  1. First write the complete content of that task file (with full implementation instructions, Allowed Files, and acceptance criteria).
  2. Then update CURRENT_TASK.md with the task name and status "In Progress".
  3. Then begin implementation.
- This rule bridges the gap between placeholder task files and Rule 3's requirement (One Task Per Request) that each task be fully specified before execution.

---
