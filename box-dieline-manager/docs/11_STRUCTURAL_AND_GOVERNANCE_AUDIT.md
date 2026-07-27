---

## تشخیص پروژه از اسناد

| مورد | تشخیص |
|---|---|
| نوع پروژه | وب‌اپ داخلی برای مدیریت آرشیو dieline / الگوهای برش جعبه |
| هدف | ثبت قالب‌های جعبه، آپلود فایل‌های طراحی، جست‌وجوی ابعادی و متنی، دانلود امن فایل‌ها و اشتراک‌گذاری عمومی زمان‌دار |
| فریم‌ورک | Next.js 14 با App Router |
| زبان | TypeScript با `strict` mode |
| Backend / DB | Supabase PostgreSQL |
| احراز هویت | Supabase Auth، ایمیل/رمز عبور، مدل MVP تک‌ادمین |
| Storage | Supabase Storage با bucket خصوصی و Signed URL |
| UI | Tailwind CSS + shadcn/ui |
| استقرار | Vercel |
| وضعیت | هنوز application code وجود ندارد؛ این فایل شامل اسناد، schema پیشنهادی، Task placeholderها و Repair Promptهاست |

---

# Critical

## 1. RLS عمومی، همهٔ templateهای دارای لینک فعال را افشا می‌کند

**محل:** `docs/06_DATA_SCHEMA.md` → جدول `templates` → policy با نام `Public can view via share token`

```sql
USING (
  public_share_token IS NOT NULL
  AND share_expires_at > NOW()
  AND deleted_at IS NULL
);
```

**مشکل:** این policy فقط وجود token فعال را بررسی می‌کند، نه اینکه درخواست‌کننده همان token مربوط به ردیف را ارائه کرده باشد.

**اثر:** کاربر ناشناس می‌تواند تمام templateهای دارای لینک اشتراک فعال را query کند؛ نه فقط template مربوط به URL اشتراک خود.

**تعارض:** با ADR-010 (لینک token-based)، ADR-014 (RLS محافظ داده) و Rule 8 (اشتراک عمومی محدود و امن) ناسازگار است.

---

## 2. `public_share_token` احتمالاً در پاسخ عمومی افشا می‌شود

**محل:** `docs/06_DATA_SCHEMA.md` → تعریف `templates.public_share_token` و همان policy عمومی

```sql
public_share_token TEXT UNIQUE
```

**مشکل:** RLS در سطح ردیف است، نه ستون. اگر client یا endpoint مجاز بتواند `SELECT *` اجرا کند، خود token نیز در خروجی قرار می‌گیرد، مگر اینکه View، RPC یا filtering صریح اعمال شود.

**اثر:** tokenهای معتبر ۷روزه ممکن است افشا شوند و افراد ناشناس بتوانند از آن‌ها برای دسترسی غیرمجاز استفاده کنند.

**ابهام مرتبط:** `02_repair_public_share_token_exposure.md` بین View، RPC و application-layer filtering انتخاب قطعی نکرده، ولی نادرست اعلام کرده که «تصمیم کاربر لازم نیست».

---

## 3. RLS عمومی جدول `files`، metadata همهٔ فایل‌های shareشده را قابل مشاهده می‌کند

**محل:** `docs/06_DATA_SCHEMA.md` → جدول `files` → policy با نام `Public can view files via template share token`

```sql
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

**مشکل:** همانند policy template، token ارائه‌شده توسط caller بررسی نمی‌شود.

**اثر:**

- افشای نام فایل، حجم، نوع MIME و `storage_path` همهٔ فایل‌های templateهای shareشده؛
- احتمال افشای `storage_url` در صورت ذخیره‌شدن Signed URL؛
- نقض اصل «فقط template لینک‌داده‌شده قابل مشاهده باشد».

---

## 4. معماری نهایی دسترسی عمومی و Signed URL تعیین نشده است

**محل:**

- `AI_DOCS/REPAIR_PROMPTS/02_repair_public_share_token_exposure.md`
- `AI_DOCS/REPAIR_PROMPTS/03_repair_rls_public_files.md`
- `AI_DOCS/REPAIR_PROMPTS/04_repair_signed_url_anonymous.md`

**مشکل:** اسناد چند راهکار متفاوت پیشنهاد می‌کنند:

- View
- RPC
- `current_setting`
- filtering در لایهٔ application
- Service Role در API Route

اما هیچ‌یک به‌عنوان معماری قفل‌شده انتخاب نشده‌اند.

**اثر:** این مورد صرفاً جزئیات implementation نیست؛ تصمیم مستقیم امنیتی است. انتخاب اشتباه، مثلاً استفادهٔ نادرست از Service Role، می‌تواند RLS را دور بزند یا امکان صدور Signed URL برای فایل‌های نامرتبط را ایجاد کند.

**نیاز به شفاف‌سازی:** باید یک الگوی واحد و رسمی تعیین شود:

1. اعتبارسنجی token در کدام لایه است؟
2. آیا public client مستقیماً Supabase query می‌زند یا فقط API Route؟
3. چه endpointی فایل را sign می‌کند؟
4. چه تضمینی وجود دارد که `fileId` متعلق به template همان token باشد؟
5. آیا token هرگز در response، log یا error message ظاهر نمی‌شود؟

---

# High

## 5. ساختار واقعی repository با path conventionهای پروژه ناسازگار یا دست‌کم مبهم است

**محل:** ساختار ابتدایی فایل پیوست‌شده و `docs/01_RULES.md` → Rule 4

**ساختار ارائه‌شده:**

```text
ramin00542-box-dieline-manager/
├── box-dieline-manager/
│   └── AI_DOCS/
└── docs/
```

**Rule 4:**

```text
Repository root: box-dieline-manager/
```

**مشکل:** طبق ساختار پیوست، `docs/` هم‌سطح `box-dieline-manager/` است، نه داخل آن. در حالی که تمام Ruleها و Repair Promptها مسیرهایی مانند `docs/01_RULES.md` را نسبت به repository root فرض می‌کنند.

**اثر:** coding agent ممکن است نتواند فایل‌های governance را در مسیرهای اعلام‌شده پیدا کند یا به اشتباه خارج از repository تغییر ایجاد کند.

**نیاز به شفاف‌سازی:** باید ساختار قطعی یکی از این دو باشد:

```text
box-dieline-manager/
├── AI_DOCS/
└── docs/
```

یا تمام مسیرهای Ruleها باید برای ساختار واقعی اصلاح شوند.

---

## 6. Taskهای اجرایی هنوز قابل اجرا نیستند

**محل:**

- `AI_DOCS/CURRENT_TASK.md`
- تمام فایل‌های `AI_DOCS/PARTS/**/task_*.md`
- `docs/01_RULES.md` → Rule 3 و Rule 6.5

**مشکل:** همهٔ Taskها placeholder هستند و فقط این متن را دارند:

```md
# TODO: Not yet written. See docs/08_PROJECT_PHASES_AND_TASKS.md for scope.
```

همچنین `CURRENT_TASK.md` هیچ Task فعالی ندارد.

**اثر:** مطابق Rule 3، عامل فقط باید یک Task مشخص را اجرا کند و فقط فایل‌های موجود در `Allowed Files` را تغییر دهد؛ اما هیچ Task فعلی دارای `Allowed Files`، acceptance criteria یا مراحل اجرا نیست.

**ابهام:** Rule 6.5 رفتن به Task بعدی را تعریف کرده، اما فرآیند فعال کردن **اولین Task** را تعریف نکرده است.

---

## 7. شماره‌گذاری و تعداد Taskهای Phase 4 متناقض است

**محل:**

- `docs/08_PROJECT_PHASES_AND_TASKS.md`
- `AI_DOCS/PARTS/phase_04_crud_upload/`
- `AI_DOCS/REPAIR_PROMPTS/04_phase_04_crud_upload_repair.md`
- `AI_DOCS/REPAIR_PROMPTS/10_repair_crud_scope_ambiguity.md`

**مشکل:**

| منبع | تعریف `task_04_07` |
|---|---|
| ساختار فعلی و `08_PROJECT_PHASES_AND_TASKS.md` | Dashboard |
| `04_phase_04_crud_upload_repair.md` | Template Edit |
| `10_repair_crud_scope_ambiguity.md` | Template Edit |
| `04_phase_04_crud_upload_repair.md` | `task_04_08` برای Soft Delete پیشنهاد می‌کند |

همچنین `docs/08_PROJECT_PHASES_AND_TASKS.md` در عنوان می‌گوید:

```text
7 Phases, 27 Tasks
```

اما در پایان می‌گوید:

```text
Total: 28 tasks across 7 phases
```

**اثر:** اجرای Repair Promptها می‌تواند فایل Dashboard را overwrite کند یا Taskهای edit/delete را با شمارهٔ اشتباه بسازد.

---

## 8. Full CRUD و Soft Delete هنوز در اسناد قفل‌شده به‌طور صریح تعریف نشده‌اند

**محل:**

- `docs/01_RULES.md` → Rule 2
- `docs/09_DECISIONS.md` → ADR-011
- `AI_DOCS/REPAIR_PROMPTS/10_repair_crud_scope_ambiguity.md`

**مشکل:** Repair Prompt می‌گوید کاربر تصمیم گرفته MVP شامل Full CRUD و Soft Delete باشد؛ اما ADR-011 فقط می‌گوید:

```text
Template CRUD
```

و Rule 2 فقط به registration form و view page اشاره دارد؛ edit، soft-delete، archive و restore در آن صریح نیستند.

**اثر:** طبق Rule 12، عامل نباید از روی Repair Prompt نتیجه بگیرد که Edit/Delete کاملاً مجاز است؛ چون Rule و ADR رسمی هنوز هم‌راستا نشده‌اند.

---

## 9. Rule 15 با Rule 11 و Master Audit دربارهٔ Git تضاد دارد

**محل:**

- `docs/01_RULES.md` → Rule 11
- `docs/01_RULES.md` → Rule 15
- `docs/07_master_audit_prompt.md` → Absolute Restrictions

**عبارات متعارض:**

```text
Every approved file change ... must be immediately staged, committed, and pushed.
```

در برابر:

```text
No commit without testing.
```

و:

```text
Do NOT create Git commits unless explicitly authorized.
```

**مشکل:** برای repairهای صرفاً مستنداتی روشن نیست:

- آیا «approved file change» خودبه‌خود authorization برای commit و push است؟
- آیا static review به‌جای test کافی است؟
- آیا push بدون دستور جداگانه مجاز است؟
- آیا عامل باید با نبود remote یا credential متوقف شود؟

**اثر:** عامل پیرو Ruleها ممکن است هم‌زمان ملزم و ممنوع از commit/push باشد.

---

## 10. مدل تک‌ادمین هنوز enforce نشده است

**محل:** `docs/06_DATA_SCHEMA.md` → RLS policyهای `templates` و `files`

نمونه:

```sql
USING (auth.uid() IS NOT NULL);
```

**مشکل:** این policyها به هر کاربر احراز هویت‌شده اجازه می‌دهند؛ نه فقط به admin واحد.

**تعارض:** با ADR-005، ADR-011 و Rule 2 که MVP را تک‌ادمین تعریف می‌کنند، ناسازگار است.

**مشکل ثانویه:** `05_repair_rls_single_admin.md` راهکارهای hardcode UUID یا `app_settings` را پیشنهاد می‌دهد، اما هیچ‌کدام قطعی نیستند و `app_settings` نیز جدول جدیدی است که در schema MVP وجود ندارد.

---

# Medium

## 11. `profiles` فاقد روند امن و قطعی برای bootstrap است

**محل:** `docs/06_DATA_SCHEMA.md` → جدول `profiles`

**مشکل:** فقط policyهای SELECT و UPDATE وجود دارد:

```sql
CREATE POLICY "Users can view their own profile"
...
CREATE POLICY "Users can update their own profile"
...
```

اما trigger یا policy قطعی برای ایجاد profile بعد از ایجاد کاربر در `auth.users` وجود ندارد.

**اثر:** `templates.created_by` و `files.uploaded_by` به `profiles(id)` وابسته‌اند؛ بنابراین ایجاد template/file ممکن است به دلیل نبود profile شکست بخورد.

**ابهام:** Repair Prompt شماره 06 بین trigger و seed script انتخاب نکرده، اما آن را «بدون نیاز به تصمیم کاربر» اعلام کرده است.

---

## 12. `profiles.email` می‌تواند از `auth.users.email` جدا شود

**محل:** `docs/06_DATA_SCHEMA.md` → جدول و policy `profiles`

**مشکل:** کاربر می‌تواند row پروفایل خود را update کند، از جمله:

```sql
email TEXT UNIQUE NOT NULL
```

اما email واقعی login در `auth.users` است. هیچ trigger یا مکانیزم sync تعریف نشده است.

**اثر:** دادهٔ پروفایل و هویت احراز هویت‌شده ممکن است ناسازگار شوند.

---

## 13. `updated_at` trigger تعریف نشده است

**محل:** `docs/06_DATA_SCHEMA.md` → جداول `profiles` و `templates`

**مشکل:**

```sql
updated_at TIMESTAMPTZ DEFAULT NOW()
```

فقط در INSERT مقداردهی می‌شود. هیچ `BEFORE UPDATE` trigger در schema فعلی نیست.

**اثر:** مقدار `updated_at` پس از edit، archive، restore یا تغییر profile درست نخواهد بود.

---

## 14. `EXCEPTION WHEN OTHERS` خطاهای واقعی full-text search را پنهان می‌کند

**محل:** `docs/06_DATA_SCHEMA.md` → تابع `templates_search_vector_update()`

```sql
EXCEPTION WHEN OTHERS THEN
```

**مشکل:** هدف fallback فقط نبودن Persian text search config است، اما این clause همهٔ exceptionها را می‌بلعد.

**اثر:** خطاهای واقعی schema، permission یا داده ممکن است پنهان شوند و سیستم بی‌صدا به `simple` fallback کند.

**نکته:** پیشنهاد Repair Prompt برای `SQLSTATE '42704'` منطقی است، ولی باید روی Supabase واقعی runtime-verify شود.

---

## 15. schema مربوط به واحد ابعاد با الگوریتم و index جست‌وجو همگام نیست

**محل:** `docs/06_DATA_SCHEMA.md`

**موارد ناسازگار:**

- ستون `unit` اضافه شده است:

```sql
unit TEXT NOT NULL DEFAULT 'cm' CHECK (unit IN ('cm', 'mm', 'inch'))
```

- اما index ابعاد هنوز unit را ندارد:

```sql
CREATE INDEX idx_templates_dimensions ON templates(length, width, height);
```

- الگوریتم `findNearMatches()` نه `unit` دریافت می‌کند، نه templateها را بر اساس unit فیلتر می‌کند.
- الگوریتم هنوز permutation-aware نیست، در حالی که Repair Promptها آن را الزام کرده‌اند.

**ابهام مفهومی:** تصمیم می‌گوید ترتیب محور معنادار است، یعنی `18×12×5` و `12×18×5` دو template متفاوت‌اند؛ ولی near-match باید permutation-aware باشد. باید روشن شود matching غیرمستقیم با permutation چگونه rank می‌گیرد و آیا exact match همچنان strict-order است یا خیر.

---

## 16. `fileUploadSchema` محدودیت‌های پروژه را enforce نمی‌کند

**محل:** `docs/06_DATA_SCHEMA.md` → `fileUploadSchema`

```ts
file_size: z.number().positive(),
mime_type: z.string().min(1),
```

**مشکل:** موارد زیر در schema فعلی نیست:

- سقف 50MB؛
- MIME type مجاز بر اساس `file_type`؛
- تطبیق `pdf` با `application/pdf`؛
- کنترل پسوند فایل؛
- جلوگیری از نوع متناقض، مانند `file_type: 'pdf'` همراه با `mime_type: 'image/png'`.

**تعارض:** با File Upload Rules و `15_repair_file_validation_schema.md`.

---

## 17. `storage_url` با ADR-016 مبهم و بالقوه ناسازگار است

**محل:** `docs/06_DATA_SCHEMA.md` → جدول `files`

```sql
storage_url TEXT
```

**مشکل:** ADR-016 می‌گوید Signed URL باید در زمان درخواست ایجاد شود، اما schema یک URL cacheشده نگه می‌دارد؛ بدون زمان انقضا، سیاست refresh یا محدودیت خروجی.

**اثر:** URL منقضی یا حساس می‌تواند در database باقی بماند و اشتباهاً به client بازگردانده شود.

---

## 18. جریان upload، staging، metadata و cleanup هنوز قرارداد قطعی ندارد

**محل:**

- `AI_DOCS/REPAIR_PROMPTS/13_repair_upload_thumbnail_architecture.md`
- `AI_DOCS/REPAIR_PROMPTS/17_repair_storage_bucket_policies.md`
- `AI_DOCS/REPAIR_PROMPTS/16_repair_template_id_not_null.md`

**مشکل:** اسناد هم‌زمان این الزامات را دارند:

- آپلود مستقیم browser به Supabase Storage؛
- `template_id` باید `NOT NULL` باشد؛
- مسیر شامل `templateId` و `fileId` باشد؛
- فایل orphan نباید وجود داشته باشد؛
- upload موقت و cleanup شکست باید وجود داشته باشد.

اما مشخص نیست:

1. template پیش از upload چگونه ساخته می‌شود؛
2. در شکست upload، template نیمه‌کاره چه می‌شود؛
3. `fileId` پیش از درج file metadata چگونه تولید می‌شود؛
4. چه endpointی Signed Upload URL می‌دهد؛
5. cleanup دقیقاً توسط browser، server یا background job انجام می‌شود.

---

# Low

## 19. Rule 1 به ADR اشتباه برای Next.js اشاره می‌کند

**محل:** `docs/01_RULES.md` → Rule 1

```text
Framework: Next.js 14 (App Router) — locked in 05_TECH_SPEC.md and ADR-002
```

**مشکل:** ADR-002 مربوط به TypeScript strict mode است. Next.js App Router در ADR-001 تعریف شده است.

---

## 20. نام TypeScript interface `File` با DOM `File` مبهم است

**محل:** `docs/06_DATA_SCHEMA.md` → TypeScript Types

```ts
export interface File {
```

**مشکل:** `File` در محیط browser یک type سراسری برای فایل upload شده است. این نام‌گذاری احتمال shadowing و سردرگمی در formهای upload ایجاد می‌کند.

**پیشنهاد طراحی برای آینده:** نامی مانند `FileRecord` یا `TemplateFile` شفاف‌تر است. این مورد فعلاً صرفاً کیفیت کد است.

---

## 21. Master Audit و Repair Prompt Index دربارهٔ اجرای prompt شماره 08 تناقض دارند

**محل:**

- `docs/07_master_audit_prompt.md` → Execution Commands
- `AI_DOCS/REPAIR_PROMPTS/00_REPAIR_PROMPTS_INDEX.md`

**تعارض:**

Master Audit دستور اجرای زیر را فهرست می‌کند:

```text
EXECUTE REPAIR PROMPT: AI_DOCS/REPAIR_PROMPTS/08_global_cross_cutting_repairs.md
```

اما Repair Prompt Index می‌گوید:

```text
Do NOT execute 08_global_cross_cutting_repairs.md directly.
```

**اثر:** agent نمی‌داند prompt 08 باید اجرا شود یا فقط نقش مرجع دارد.

---

## 22. چند Repair Prompt مقدار عددی جدید معرفی می‌کنند، برخلاف محدودیت Master Audit

**محل:**

- `04_repair_signed_url_anonymous.md` → TTL پیشنهادی 5 دقیقه
- `23_repair_pagination_contract.md` → limit پیش‌فرض 20 و حداکثر 100
- `24_repair_rate_limiting.md` → 10 و 100 درخواست در دقیقه
- `docs/07_master_audit_prompt.md` → Absolute Restrictions

**تعارض با:**

```text
Do NOT invent numeric values ... without explicit user approval.
```

**مشکل:** این مقادیر در promptها به Required Changes تبدیل شده‌اند، اما در سند ارائه‌شده تأیید صریح کاربر برای آن‌ها ثبت نشده است.

---

## 23. فایل‌های مهم برای audit کامل قابل بررسی نیستند

**محل:**

- `docs/05_TECH_SPEC.md`
- `docs/10_EXTERNAL_SECURITY_AUDIT.md`
- `AI_DOCS/REPAIR_PROMPTS/01_repair_rls_public_templates.md`

**مشکل:** در پیوست به‌صورت `[Binary file]` نمایش داده شده‌اند.

**اثر:** نمی‌توان هم‌خوانی کامل Repair Promptها با Tech Spec، گزارش خارجی یا راهکار RLS شماره 01 را تأیید کرد.

---

## جمع‌بندی

مهم‌ترین ریسک‌های فعلی عبارت‌اند از:

1. **RLS عمومی ناامن برای templateها و فایل‌ها؛**
2. **نبود معماری قطعی برای token validation و Signed URL عمومی؛**
3. **Taskهای placeholder و نبود Task فعال، که اجرای قانون One Task Per Request را ناممکن می‌کند؛**
4. **تناقض‌های Phase 4 در شماره‌گذاری Dashboard/Edit/Soft Delete؛**
5. **ابهام Git commit/push در برابر الزام testing و authorization؛**
6. **ناهمگامی schema فعلی با تصمیم‌های Repair Prompt دربارهٔ unit، validation، CRUD و storage.**







