# Data Schema — Database Structure (MVP)

## Core Tables

### 1. profiles (NOT users — avoids conflict with auth.users)
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  is_active BOOLEAN DEFAULT true,
  is_admin BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Seed: The first admin user must have is_admin set to true manually
-- via a one-time SQL operation (INSERT INTO profiles ... SET is_admin = true)
-- or via the Supabase Dashboard after the user signs up.
-- There is no self-service registration; public sign-up is disabled.
--
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
  unit TEXT NOT NULL DEFAULT 'cm' CHECK (unit IN ('cm', 'mm', 'inch')),
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
      setweight(to_tsvector('persian', COALESCE(NEW.material, '')), 'D') ||
      setweight(to_tsvector('persian', COALESCE(NEW.description, '')), 'D') ||
      setweight(to_tsvector('persian', COALESCE(array_to_string(NEW.tags, ' '), '')), 'D');
  EXCEPTION WHEN OTHERS THEN
    -- Fallback to simple config
    NEW.search_vector := 
      setweight(to_tsvector('simple', COALESCE(NEW.code, '')), 'A') ||
      setweight(to_tsvector('simple', COALESCE(NEW.name, '')), 'B') ||
      setweight(to_tsvector('simple', COALESCE(NEW.box_type, '')), 'C') ||
      setweight(to_tsvector('simple', COALESCE(NEW.material, '')), 'D') ||
      setweight(to_tsvector('simple', COALESCE(NEW.description, '')), 'D') ||
      setweight(to_tsvector('simple', COALESCE(array_to_string(NEW.tags, ' '), '')), 'D');
  END;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER templates_search_vector_trigger
  BEFORE INSERT OR UPDATE ON templates
  FOR EACH ROW EXECUTE FUNCTION templates_search_vector_update();

CREATE INDEX idx_templates_search ON templates USING GIN(search_vector);

-- ⚠️ Needs runtime verification: test whether the 'persian' text search
-- configuration is available in the target Supabase Postgres instance.
-- The EXCEPTION block provides a fallback to 'simple', but this code path
-- must be verified against a real Supabase database during Phase 2 (Database
-- & Schema). Static review alone cannot confirm this.

-- Only admin users (profiles.is_admin = true) can access templates.
-- The single-admin model is enforced by RLS, not just by disabling public sign-up.
-- See ADR-018 in docs/09_DECISIONS.md for the decision.
-- RLS Policies
ALTER TABLE templates ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Admin users can view templates"
  ON templates FOR SELECT
  USING (
    auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true)
    AND deleted_at IS NULL
  );

CREATE POLICY "Admin users can insert templates"
  ON templates FOR INSERT
  WITH CHECK (auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true));

CREATE POLICY "Admin users can update templates"
  ON templates FOR UPDATE
  USING (auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true));

-- Public access to templates is handled by GET /api/share/[token]
-- (Next.js API Route with Supabase Service Role), NOT by RLS.
-- See ADR-017 in docs/09_DECISIONS.md for the locked architecture.
```

### 3. files (ALL file metadata here, NOT in templates)
-- Note: `files` is an append-only table — records are never edited after
-- creation, so only `created_at` is present (no `updated_at`). This is an
-- intentional exception to the general rule (see Rule 13 in 01_RULES.md).
```sql
CREATE TABLE files (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  template_id UUID NOT NULL REFERENCES templates(id) ON DELETE CASCADE,
  file_type TEXT NOT NULL CHECK (file_type IN ('image', 'thumbnail', 'pdf', 'ai', 'cdr')),
  file_name TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type TEXT NOT NULL,
  storage_path TEXT NOT NULL,  -- path in Supabase Storage
  storage_url TEXT,            -- cached Signed URL (nullable); the authoritative access
                                  -- mechanism is a Signed URL generated from storage_path at
                                  -- request time, NOT this field — see ADR-016
  uploaded_by UUID REFERENCES profiles(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_files_template ON files(template_id);
CREATE INDEX idx_files_type ON files(file_type);

-- Only admin users (profiles.is_admin = true) can access file metadata.
-- Matches the same pattern used on templates.
-- RLS Policies
ALTER TABLE files ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Admin users can view files"
  ON files FOR SELECT
  USING (auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true));

CREATE POLICY "Admin users can insert files"
  ON files FOR INSERT
  WITH CHECK (auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true));

CREATE POLICY "Admin users can delete files"
  ON files FOR DELETE
  USING (auth.uid() IN (SELECT id FROM profiles WHERE is_admin = true));

-- Public file access is handled by GET /api/share/[token]/files/[fileId]/download
-- which validates the token, verifies file-to-template ownership, and generates
-- a 5-minute Signed URL. See ADR-017 in docs/09_DECISIONS.md.
```

## TypeScript Types

```typescript
// types/database.ts
export interface Profile {
  id: string;
  email: string;
  full_name: string;
  is_active: boolean;
  is_admin: boolean;
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
  unit: 'cm' | 'mm' | 'inch';
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
  storage_url?: string;  // nullable — see ADR-016
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
  unit: z.enum(['cm', 'mm', 'inch']).default('cm'),
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
