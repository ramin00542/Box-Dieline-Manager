# Repair Prompts Index

Generated from full audit of 24 items in `docs/10_EXTERNAL_SECURITY_AUDIT.md` against current docs.

## Deduplication Notice

5 cross-cutting items from the security audit (Items 8, 18, 22, 23, 24) were covered in **both** the per-item repair prompts **and** the phase-level cross-cutting prompt (`08_global_cross_cutting_repairs.md`). Resolution:

- The **per-item prompts** (`08_repair_rules_6_4_6_5.md`, `18_repair_api_response_any_type.md`, `22_repair_dashboard_scope.md`, `23_repair_pagination_contract.md`, `24_repair_rate_limiting.md`) are the **authoritative detailed versions** — they contain complete Required Changes with full context.
- `08_global_cross_cutting_repairs.md` has been **simplified to a coordination prompt** that lists these 5 items and references their per-item prompts. Do NOT execute `08_global_cross_cutting_repairs.md` directly — use the individual per-item prompts instead.

## Phase-Level Repair Prompts (per `docs/07_MASTER_AUDIT_PROMPT.md`)

| # | Phase | Status | File | Purpose |
|---|-------|--------|------|---------|
| 01 | Phase 01: Setup & Foundation | Ready | `01_phase_01_setup_repair.md` | Write task files for project initialization |
| 02 | Phase 02: Database & Schema | Ready | `02_phase_02_database_repair.md` | Write task files for DB tables, RLS, triggers |
| 03 | Phase 03: Authentication | Ready | `03_phase_03_auth_repair.md` | Write task files for Supabase Auth |
| 04 | Phase 04: Template CRUD & File Upload | Ready | `04_phase_04_crud_upload_repair.md` | Write task files for CRUD, upload, edit, soft-delete |
| 05 | Phase 05: Search & Filters | Ready | `05_phase_05_search_repair.md` | Write task files for dimension + full-text search |
| 06 | Phase 06: Public Share Links | Ready | `06_phase_06_share_links_repair.md` | Write task files for share link flow |
| 07 | Phase 07: Testing & Deployment | Ready | `07_phase_07_testing_deploy_repair.md` | Write task files for tests and deploy |
| 08 | Cross-Cutting & Governance | Coordination only | `08_global_cross_cutting_repairs.md` | **Orchestration only** — references per-item prompts |

## Per-Item Repair Prompts (from 24-item Security Audit)

| # | Severity | Short Description | File | Maps To |
|---|----------|------------------|------|---------|
| 1 | 🔴 **Critical** | RLS doesn't restrict to specific share token (templates) | `01_repair_rls_public_templates.md` | Phase 06 |
| 2 | 🔴 **Critical** | `public_share_token` exposed via SELECT * | `02_repair_public_share_token_exposure.md` | Phase 06 |
| 3 | 🔴 **Critical** | RLS doesn't restrict files by specific share token | `03_repair_rls_public_files.md` | Phase 06 |
| 4 | 🟠 High | Anonymous Signed URL flow undefined | `04_repair_signed_url_anonymous.md` | Phase 06 |
| 5 | 🟠 High | RLS allows any authenticated user, not just single admin | `05_repair_rls_single_admin.md` | Phase 02/03 |
| 6 | 🟠 High | Profile auto-creation/bootstrap undefined | `06_repair_profile_bootstrap.md` | Phase 02 |
| 7 | 🟠 High | `profiles.email` can drift from `auth.users.email` | `07_repair_profile_email_sync.md` | Phase 02 |
| 8 | ⛔ **Blocking** | Rules 6.4 and 6.5 referenced but undefined | `08_repair_rules_6_4_6_5.md` | Cross-Cutting |
| 9 | 🟠 High | AI_DOCS/ structure and task files missing | **Fixed** | *(setup complete)* |
| 10 | 🟠 High | "Template CRUD" scope vs Phase 4 tasks mismatch | `10_repair_crud_scope_ambiguity.md` ✅ | Phase 04 |
| 11 | 🟡 Medium | `updated_at` has no auto-update trigger | `11_repair_updated_at_trigger.md` | Phase 02 |
| 12 | 🟡 Medium | `storage_url` cached Signed URL without expiry policy | `12_repair_storage_url_clarify.md` | Phase 02 |
| 13 | 🟡 Medium | 50MB upload + thumbnail architecture not specified | `13_repair_upload_thumbnail_architecture.md` | Phase 04 |
| 14 | 🟡 Medium | Dimension units and axis order undefined | `14_repair_dimension_units_order.md` ✅ | Phase 05 |
| 15 | 🟡 Medium | Zod validation doesn't enforce file size / MIME rules | `15_repair_file_validation_schema.md` | Phase 04 |
| 16 | 🟡 Medium | `template_id` in `files` table nullable | `16_repair_template_id_not_null.md` ✅ | Phase 02/04 |
| 17 | 🟡 Medium | Storage bucket name, path structure, RLS not defined | `17_repair_storage_bucket_policies.md` | Phase 02/04 |
| 18 | 🟡 Medium | API response uses `any` type contradicting Rule 9 | `18_repair_api_response_any_type.md` | Cross-Cutting |
| 19 | 🟡 Medium | Full-text search fields and query function undefined | `19_repair_fulltext_search_fields.md` ✅ | Phase 05 |
| 20 | 🟢 Low | `EXCEPTION WHEN OTHERS` swallows all errors | `20_repair_exception_when_others.md` | Phase 02/05 |
| 21 | 🟢 Low | TypeScript `File` interface conflicts with DOM `File` | `21_repair_file_type_name_conflict.md` | Phase 04 |
| 22 | 🟢 Low | Dashboard in goal but no task or rules definition | `22_repair_dashboard_scope.md` ✅ | Cross-Cutting |
| 23 | 🟢 Low | Pagination contract (cursor, sort, limits) undefined | `23_repair_pagination_contract.md` | Cross-Cutting |
| 24 | 🟢 Low | Rate limiting tool/service and limits not specified | `24_repair_rate_limiting.md` | Cross-Cutting |

✅ = User decision received and incorporated

## Severity Summary

| Severity | Count | Phase 1 | Phase 2 | Phase 3 | Phase 4 | Phase 5 | Phase 6 | Phase 7 | Cross |
|----------|-------|---------|---------|---------|---------|---------|---------|---------|-------|
| **Critical** | 3 | — | — | — | — | — | 3 | — | — |
| **High (incl. Blocking)** | 6 | — | 2 | — | 2 | — | — | — | 2 |
| **Medium** | 9 | — | 3 | — | 3 | 2 | — | — | 1 |
| **Low** | 5 | — | — | — | 1 | — | — | — | 4 |
| **Fixed** | 1 | — | — | — | — | — | — | — | — |
| **Total** | 24 | 0 | 5 | 0 | 6 | 2 | 3 | 0 | 7 |

## Recommended Execution Order

### Phase 0 — Cross-Cutting Governance Fixes (execute these first)
| Order | Repair Prompt | Reason |
|-------|---------------|--------|
| **1** | `08_repair_rules_6_4_6_5.md` | ⛔ BLOCKING: Rules 6.4/6.5 must exist first |
| **2** | `22_repair_dashboard_scope.md` | Dashboard scope needs docs update before Phase 1 tasks |
| **3** | `18_repair_api_response_any_type.md` | API contract needed before Phase 4 code |
| **4** | `23_repair_pagination_contract.md` | Pagination needed before Phase 5 search |
| **5** | `24_repair_rate_limiting.md` | Rate limiting needed before Phase 6 public links |

### Phase 1–7 — Write Phase Task Files
| Order | Repair Prompt | Reason |
|-------|---------------|--------|
| **6** | `01_phase_01_setup_repair.md` | Setup tasks are prerequisites for everything |
| **7** | `02_phase_02_database_repair.md` | DB tasks next (schema + RLS fixes) |
| **8** | `03_phase_03_auth_repair.md` | Auth tasks |
| **9** | `04_phase_04_crud_upload_repair.md` | CRUD + upload (biggest phase) |
| **10** | `05_phase_05_search_repair.md` | Search tasks |
| **11** | `06_phase_06_share_links_repair.md` | Share links (Critical RLS fixes) |
| **12** | `07_phase_07_testing_deploy_repair.md` | Testing + deploy |

**Note**: `08_global_cross_cutting_repairs.md` is NOT executed directly — it is the orchestration layer for the 5 cross-cutting per-item prompts above (Order 1–5).

## Final File Count After Deduplication

| Category | Count |
|----------|-------|
| Phase-level repair prompts (unique) | 7 (01–07) |
| Cross-cutting coordination prompt | 1 (08 — no duplicate content) |
| Per-item repair prompts (unique) | 23 (24 items − 1 fixed) |
| **Total unique executable prompts** | **30** |
