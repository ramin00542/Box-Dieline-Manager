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
