
# Please review this file for bugs, security issues, and code quality problems.
Detect the project type, tech stack, and purpose from the code itself.
Also cross-check it against the project's locked rules/ADRs if provided.
Report issues grouped by severity (Critical / High / Medium / Low), 
with exact line numbers and a short explanation for each.
Do not fix anything yet — just report.
Please answer in Persian (Farsi).



## حالت

**Review/Planning Mode (بازبینی، بدون اعمال تغییر)**

> **محدودیت شماره‌خط:** فایل پیوست‌شده به‌صورت متن Markdown تجمیع‌شده و بدون شماره‌خط پایدار ارائه شده است. بنابراین نمی‌توانم شماره‌خط مطلق و قابل‌اتکای فایل اصلی را ادعا کنم. برای جلوگیری از شماره‌گذاری ساختگی، هر مورد را با **مسیر فایل، بخش، و عبارت/دستور دقیق** مشخص کرده‌ام. اگر نسخهٔ فایل با شماره‌خط یا مخزن Git ارائه شود، می‌توان گزارش را به شماره‌خط مطلق تبدیل کرد.

---

## تشخیص پروژه از محتوای موجود

| مورد | تشخیص |
|---|---|
| نوع پروژه | وب‌اپلیکیشن داخلی برای مدیریت، جست‌وجو و اشتراک‌گذاری dieline / الگوهای برش جعبه |
| نام پروژه | **Box Dieline Manager (Die Cut Library)** |
| هدف | ثبت قالب‌ها و فایل‌های وابسته، جست‌وجوی ابعادی و نزدیک، دانلود فایل‌ها و ساخت لینک اشتراک عمومی ۷روزه |
| فریم‌ورک | Next.js 14.2.x با App Router |
| زبان | TypeScript 5.x با strict mode |
| Backend / Database | Supabase PostgreSQL |
| احراز هویت | Supabase Auth با ایمیل/رمز عبور، یک admin در MVP |
| Storage | Supabase Storage با bucket خصوصی و Signed URL |
| UI | Tailwind CSS + shadcn/ui |
| Deploy | Vercel |
| وضعیت فعلی | هنوز کد اجرایی وجود ندارد؛ محتوا شامل specification، ADR و نمونه‌های SQL/TypeScript است |

---

# یافته‌های بحرانی — Critical

## 1. RLS عمومی همهٔ قالب‌های دارای لینک فعال را قابل خواندن می‌کند، نه فقط قالب توکنِ درخواست‌شده

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → بخش `templates` → RLS Policy با نام:

```sql
CREATE POLICY "Public can view via share token"
ON templates FOR SELECT
USING (
  public_share_token IS NOT NULL
  AND share_expires_at > NOW()
  AND deleted_at IS NULL
);
```

**مشکل:**  
این policy فقط بررسی می‌کند که قالب دارای یک توکن اشتراک فعال باشد؛ اما بررسی نمی‌کند که کاربر ناشناس، **همان توکن مربوط به ردیف** را در URL یا درخواست ارائه کرده باشد.

در نتیجه، یک درخواست ناشناس به جدول `templates` می‌تواند تمام templateهای دارای `public_share_token` فعال را مشاهده کند.

**اثر امنیتی:**

- افشای همهٔ قالب‌های shareشده و فعال.
- نقض ADR-010 که لینک عمومی را «token-based» و محدود به اشتراک مشخص توصیف می‌کند.
- نقض Rule 8 در `01_RULES.md` دربارهٔ محدودسازی دسترسی عمومی.
- نقض هدف محصول: مشتری باید فقط قالبی را ببیند که لینک آن را دریافت کرده است.

---

## 2. خود `public_share_token` احتمالاً از طریق policy عمومی افشا می‌شود

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → بخش `templates` → تعریف ستون:

```sql
public_share_token TEXT UNIQUE
```

و policy عمومی `Public can view via share token`.

**مشکل:**  
RLS در سطح row عمل می‌کند، نه column. اگر query عمومی مجاز باشد `SELECT *` انجام دهد، ستون `public_share_token` نیز در پاسخ وجود خواهد داشت؛ مگر اینکه API، View یا RPC به‌طور صریح آن را حذف کند. چنین مکانیزمی در اسناد تعریف نشده است.

**اثر امنیتی:**

- مهاجم ممکن است tokenهای معتبر همهٔ templateهای shareشده را دریافت کند.
- توکن‌ها که باید مانند credential رفتار کنند، در دادهٔ عمومی قابل افشا می‌شوند.
- با توجه به اینکه توکن برای ۷ روز معتبر است، افشای آن می‌تواند دسترسی غیرمجاز موقت اما مهم ایجاد کند.

---

## 3. RLS عمومی جدول `files` تمام فایل‌های متعلق به templateهای shareشده را قابل مشاهده می‌کند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → بخش `files` → RLS Policy با نام:

```sql
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

**مشکل:**  
همان ایراد policy مربوط به `templates` در اینجا نیز وجود دارد: هیچ مقایسه‌ای بین توکن ارائه‌شده در URL و `templates.public_share_token` انجام نمی‌شود.

**اثر امنیتی:**

- کاربر ناشناس می‌تواند metadata همهٔ فایل‌های templateهای دارای لینک فعال را ببیند.
- نام فایل، مسیر storage، MIME type، حجم، شناسهٔ template و احتمالاً `storage_url` افشا می‌شود.
- اگر `storage_url` حاوی Signed URL معتبر باشد، دسترسی مستقیم به فایل نیز ممکن است افشا شود.

---

## 4. سازوکار امن تولید Signed URL برای کاربران ناشناس تعریف نشده است

**محل دقیق:**

- `docs/01_RULES.md` → Rule 8 و Rule 14
- `docs/09_DECISIONS.md` → ADR-016
- `docs/06_DATA_SCHEMA.md` → جدول `files`

**عبارات مرتبط:**

```text
All Storage buckets must be Private.
File access is granted only via time-limited Signed URLs.
```

و:

```text
Signed URLs generated at request time
```

**مشکل:**  
اسناد الزام می‌کنند Storage خصوصی باشد و URL فقط به‌صورت signed صادر شود؛ اما مشخص نمی‌کنند که کاربر ناشناسِ دارای لینک share چگونه پس از اعتبارسنجی token، Signed URL دریافت می‌کند.

صدور Signed URL معمولاً نیازمند سطح دسترسی حساس‌تر در backend است. اگر endpoint با Service Role اجرا شود، باید به‌صورت بسیار دقیق token را اعتبارسنجی کند و فقط فایل‌های متعلق به همان template را sign کند. این جریان در اسناد تعریف نشده است.

**اثر امنیتی / اجرایی:**

- قابلیت دانلود فایل از لینک عمومی به‌صورت امن قابل پیاده‌سازی قطعی نیست.
- پیاده‌سازی عجولانه ممکن است Service Role key را در client افشا کند یا endpointی ایجاد کند که هر فایل را sign کند.
- با ADR-016 سازگار بودن سیستم قابل تضمین نیست.

---

# یافته‌های شدید — High

## 5. سیاست‌های RLS با مدل «فقط یک admin» هم‌راستا نیستند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → policyهای `templates` و `files`.

نمونه‌ها:

```sql
USING (auth.uid() IS NOT NULL);
```

```sql
WITH CHECK (auth.uid() IS NOT NULL);
```

**مشکل:**  
مدل محصول می‌گوید MVP فقط یک admin دارد، اما policyها هر کاربر احراز هویت‌شده را مجاز می‌دانند. اگر حساب Auth دیگری به هر دلیل ایجاد شود، آن حساب نیز می‌تواند داده‌ها را ببیند، template ثبت کند، templateها را تغییر دهد یا فایل‌ها را حذف کند.

**تعارض با اسناد قفل‌شده:**

- `docs/02_PROJECT_GOAL.md`: «Admin login (single user)»
- `docs/05_TECH_SPEC.md`: «Single admin user for MVP»
- `docs/09_DECISIONS.md` → ADR-005 و ADR-011.

**نکته:**  
یادداشت schema می‌گوید public sign-up غیرفعال خواهد شد، اما هیچ تنظیم، migration، policy یا روند bootstrap برای تضمین آن تعریف نشده است.

---

## 6. ایجاد profile برای کاربر admin تعریف نشده و احتمالاً ورود/ثبت داده را مسدود می‌کند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → بخش `profiles` → policyها:

```sql
CREATE POLICY "Users can view their own profile"
ON profiles FOR SELECT
USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile"
ON profiles FOR UPDATE
USING (auth.uid() = id);
```

**مشکل:**  
برای `profiles` هیچ policy مربوط به `INSERT` و هیچ trigger مربوط به ایجاد خودکار profile بعد از ایجاد کاربر در `auth.users` تعریف نشده است.

در حالی که `templates.created_by` و `files.uploaded_by` به `profiles(id)` اشاره می‌کنند:

```sql
created_by UUID REFERENCES profiles(id)
```

```sql
uploaded_by UUID REFERENCES profiles(id)
```

**اثر:**

- مشخص نیست رکورد profile اولین admin چگونه ساخته می‌شود.
- ثبت template یا file ممکن است با foreign-key failure مواجه شود.
- جریان ورود و bootstrap کاربر admin از نظر عملیاتی ناقص است.

---

## 7. `profiles.email` قابل تغییر است و می‌تواند با `auth.users.email` ناسازگار شود

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → تعریف جدول `profiles`:

```sql
email TEXT UNIQUE NOT NULL
```

و policy:

```sql
CREATE POLICY "Users can update their own profile"
ON profiles FOR UPDATE
USING (auth.uid() = id);
```

**مشکل:**  
policy فعلی به کاربر اجازه می‌دهد تمام ستون‌های profile خود، از جمله `email` را update کند. اما email احراز هویت در `auth.users` مدیریت می‌شود. در نتیجه ممکن است:

- `profiles.email` با email واقعی login متفاوت شود؛
- دادهٔ پروفایل از Supabase Auth خارج از همگام‌سازی بماند؛
- unique constraint در `profiles` با هویت Auth ارتباط قابل‌اعتماد نداشته باشد.

**کیفیت طراحی:**  
باید روشن شود email در profile فقط cache است، فقط خواندنی است، یا با trigger همگام‌سازی می‌شود.

---

## 8. ارجاع به Rules 6.4 و 6.5 وجود دارد، اما این قوانین تعریف نشده‌اند

**محل دقیق:**  
`docs/07_master_audit_prompt.md` → چندین بخش، از جمله:

```text
Do NOT modify CURRENT_TASK.md unless Rules 6.4 and 6.5 are fully satisfied.
```

و:

```text
Do NOT modify CHANGELOG.md unless Rule 6.4 verification exists.
```

**مشکل:**  
در `docs/01_RULES.md`، Rule 6 فقط شامل موارد زیر است:

- 6.1 Clarification Mode
- 6.2 Review/Planning Mode
- 6.3 Implementation Mode

Rule 6.4 و 6.5 وجود ندارند.

**اثر:**

- اجرای فرآیند audit/repair طبق دستورالعمل خودش غیرممکن یا مبهم می‌شود.
- عامل کدنویسی نمی‌تواند تشخیص دهد چه زمانی مجاز به update کردن `CURRENT_TASK.md` یا `CHANGELOG.md` است.
- این مورد با Rule 12 نیز درگیر می‌شود، چون ambiguity باید باعث توقف و Clarification Mode شود.

---

## 9. Master Audit به فایل‌ها و Taskهایی وابسته است که در ساختار ارائه‌شده وجود ندارند

**محل دقیق:**

- `docs/07_master_audit_prompt.md`
- `docs/08_PROJECT_PHASES_AND_TASKS.md`

**فایل‌های موردنیاز اما ارائه‌نشده:**

```text
CURRENT_TASK.md
PROJECT_STATUS.md
AI_DOCS/PARTS/
```

همچنین ۲۷ Task نام‌برده در `08_PROJECT_PHASES_AND_TASKS.md` در فایل پیوست‌شده وجود ندارند.

**مشکل:**  
Master Audit الزام می‌کند که همهٔ این فایل‌ها پیش از audit خوانده شوند، اما فعلاً وجودشان تأیید نشده است.

**اثر:**

- Rule 3 از `01_RULES.md`، یعنی «One Task Per Request»، قابل اعمال نیست.
- هیچ `Allowed Files`ای برای شروع implementation وجود ندارد.
- شروع کار مطابق قواعد خود پروژه متوقف می‌شود.

---

## 10. عبارت «Template CRUD» با Taskهای فاز ۴ سازگار نیست

**محل دقیق:**

- `docs/09_DECISIONS.md` → ADR-011:
  ```text
  Template CRUD
  ```
- `docs/08_PROJECT_PHASES_AND_TASKS.md` → Phase 4.

**مشکل:**  
Taskهای Phase 4 شامل فرم، upload، list و detail هستند، اما Taskهای صریح برای موارد زیر وجود ندارند:

- ویرایش template؛
- soft delete template؛
- بازگردانی template؛
- حذف یا جایگزینی file؛
- update status یا tags.

در عین حال schema دارای این ستون است:

```sql
deleted_at TIMESTAMPTZ
```

که نشان می‌دهد soft delete مدنظر بوده است.

**اثر:**  
عامل پیاده‌سازی نمی‌داند MVP واقعاً CRUD کامل است یا فقط Create/Read. این ابهام می‌تواند باعث scope creep یا implementation ناقص شود.

---

# یافته‌های متوسط — Medium

## 11. `updated_at` تعریف شده اما خودکار به‌روزرسانی نمی‌شود

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → جداول `profiles` و `templates`.

نمونه:

```sql
updated_at TIMESTAMPTZ DEFAULT NOW()
```

**مشکل:**  
`DEFAULT NOW()` فقط هنگام insert اجرا می‌شود. هیچ trigger مانند `BEFORE UPDATE` برای تغییر خودکار `updated_at` تعریف نشده است.

**اثر:**

- مقدار `updated_at` پس از ویرایش template یا profile ممکن است قدیمی باقی بماند.
- مرتب‌سازی، audit، cache invalidation یا نمایش «آخرین ویرایش» نادرست می‌شود.
- با Rule 13 در `01_RULES.md` که وجود `updated_at` را الزام می‌کند، از نظر معنایی ناقص است.

---

## 12. ستون `storage_url` با ADR-016 ناسازگار یا دست‌کم مبهم است

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → جدول `files`:

```sql
storage_url TEXT
```

توضیح همراه:

```text
cached Signed URL (nullable)
```

و ADR-016 در `docs/09_DECISIONS.md`:

```text
Signed URLs generated at request time
```

**مشکل:**  
Signed URL زمان‌دار است. ذخیره‌کردن آن در database بدون فیلد زمان انقضا، سیاست refresh و سیاست پاک‌سازی، باعث ذخیرهٔ URL منقضی یا بالقوه حساس می‌شود.

همچنین این کار با عبارت ADR-016 مبنی بر اینکه `storage_path` مسیر authoritative است، هم‌راستایی کامل ندارد.

**اثر:**

- احتمال نمایش لینک دانلود منقضی.
- احتمال افشای URL زمان‌دار از طریق queryهای database.
- پیچیدگی غیرضروری در cache و invalidation.

---

## 13. محدودیت 50MB و تولید thumbnail با معماری Vercel/Supabase عملیاتی نشده است

**محل دقیق:**

- `docs/05_TECH_SPEC.md` → File Upload Rules:
  ```text
  Maximum file size: 50MB per file
  ```
- `docs/01_RULES.md` → Rule 14:
  ```text
  Thumbnails generated automatically on upload
  ```
- `docs/09_DECISIONS.md` → ADR-009.

**مشکل:**  
مشخص نشده است:

- فایل مستقیماً از browser به Supabase Storage upload می‌شود یا از Next.js API Route عبور می‌کند؛
- تولید thumbnail در browser، Vercel Function، Supabase Edge Function یا سرویس دیگر انجام می‌شود؛
- برای کدام فرمت‌ها thumbnail ساخته می‌شود؛
- اگر thumbnail generation شکست بخورد، چه رخ می‌دهد؛
- آیا uploadهای 50MB با محدودیت‌های request body و serverless runtime سازگار خواهند بود.

**اثر:**

- پیاده‌سازی ممکن است با محدودیت حجم درخواست یا timeout روبه‌رو شود.
- نیاز به dependency یا سرویس پردازش تصویر ممکن است با Rule 1 دربارهٔ «عدم افزودن dependency عمده بدون ADR» تضاد پیدا کند.

---

## 14. واحد ابعاد و قاعدهٔ ترتیب Length/Width/Height تعریف نشده است

**محل دقیق:**

- `docs/02_PROJECT_GOAL.md`:
  ```text
  18×12×5
  ```
- `docs/06_DATA_SCHEMA.md` → ستون‌های:
  ```sql
  length DECIMAL(10,2)
  width DECIMAL(10,2)
  height DECIMAL(10,2)
  ```

**مشکل:**  
هیچ واحدی تعیین نشده است: میلی‌متر، سانتی‌متر یا اینچ. همچنین معلوم نیست آیا `18×12×5` و `12×18×5` باید دو قالب متفاوت تلقی شوند یا یکسان.

**اثر:**

- tolerance پیش‌فرض ±2 معنای عملی مشخصی ندارد.
- نزدیک‌بودن ابعاد در واحدهای مختلف نتایج بسیار متفاوت تولید می‌کند.
- parse کردن ورودی کاربر مبهم است.

---

## 15. Validation schema محدودیت‌های فنی فایل را enforce نمی‌کند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → `fileUploadSchema`:

```ts
export const fileUploadSchema = z.object({
  file_type: z.enum(['image', 'thumbnail', 'pdf', 'ai', 'cdr']),
  file_name: z.string().min(1),
  file_size: z.number().positive(),
  mime_type: z.string().min(1),
});
```

**مشکل:**  
این schema موارد زیر را بررسی نمی‌کند:

- حداکثر حجم 50MB؛
- MIME type مجاز برای هر `file_type`؛
- پسوندهای مجاز؛
- جلوگیری از تناقض میان `file_type` و `mime_type`؛
- اعتبارسنجی سمت سرور پس از دریافت واقعی فایل.

**نمونه:**  
در schema فعلی، file با `file_type: 'pdf'` و `mime_type: 'image/png'` معتبر تلقی می‌شود، چون فقط non-empty بودن MIME بررسی می‌شود.

---

## 16. `template_id` در جدول `files` باید احتمالاً `NOT NULL` باشد

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → جدول `files`:

```sql
template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
```

**مشکل:**  
طبق Rule 14 در `01_RULES.md`:

```text
Templates reference files via template_id foreign key.
```

و طبق هدف سیستم، فایل‌ها متعلق به template هستند. با این حال، `template_id` nullable تعریف شده است.

**اثر:**

- می‌توان رکورد file بدون template ساخت.
- فایل orphan در database و storage ممکن است ایجاد شود.
- سیاست عمومی `files` نیز برای رکوردهای بدون template رفتار نامشخص خواهد داشت.

---

## 17. مسیر، نام bucket و policyهای Supabase Storage تعریف نشده‌اند

**محل دقیق:**

- `docs/01_RULES.md` → Rule 8 و Rule 14
- `docs/05_TECH_SPEC.md` → File Upload Rules
- `docs/06_DATA_SCHEMA.md` → `storage_path`
- `docs/09_DECISIONS.md` → ADR-016

**مشکل:**  
تنها `storage_path` ذخیره می‌شود، اما این موارد مشخص نشده‌اند:

- نام bucket؛
- ساختار path، مانند `templates/{templateId}/{fileId}/{filename}`؛
- جلوگیری از overwrite؛
- storage.objects RLS policyها؛
- نحوهٔ پاک‌سازی Storage در soft delete؛
- rollback هنگام upload موفق ولی insert metadata ناموفق؛
- پاک‌سازی metadata هنگام حذف فیزیکی فایل.

**اثر:**  
آپلود امن و قابل بازیابی قابل طراحی دقیق نیست و خطر orphan file یا حذف‌نشدن فایل‌های قدیمی وجود دارد.

---

## 18. نمونهٔ API response دارای `any` است و با Rule 9 تناقض دارد

**محل دقیق:**  
`docs/05_TECH_SPEC.md` → API Design:

```ts
{ success: boolean, data?: any, error?: string }
```

**تعارض با:**  
`docs/01_RULES.md` → Rule 9:

```text
No any type unless absolutely necessary with justification.
```

**مشکل:**  
نمونهٔ API رسمی، `any` را پیشنهاد می‌کند، در حالی که قوانین کیفیت استفاده از آن را منع می‌کنند.

**اثر:**  
عامل کدنویسی نمی‌داند باید قرارداد API را عیناً دنبال کند یا از generic type استفاده کند.

---

## 19. جست‌وجوی full-text از نظر ورودی و دامنهٔ فیلدها ناقص است

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → function:

```sql
templates_search_vector_update()
```

و بخش `Search Implementation` در `docs/05_TECH_SPEC.md`.

**مشکل:**  
`search_vector` فقط این ستون‌ها را شامل می‌شود:

```sql
code
name
box_type
material
```

اما مشخص نیست `description` و `tags` باید قابل جست‌وجو باشند یا خیر. همچنین مشخص نیست query کاربر با کدام تابع تبدیل می‌شود:

- `plainto_tsquery`
- `websearch_to_tsquery`
- `phraseto_tsquery`
- روش دیگر

**اثر:**

- نتایج full-text search بین پیاده‌سازی‌های مختلف متفاوت خواهد بود.
- رفتار برای فارسی، انگلیسی، ترکیب فارسی/لاتین و کدهای محصول مشخص نیست.
- رتبه‌بندی relevance تعریف نشده است.

---

# یافته‌های کم‌اهمیت‌تر — Low

## 20. `EXCEPTION WHEN OTHERS` در trigger جست‌وجو همهٔ خطاها را پنهان می‌کند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → function `templates_search_vector_update()`:

```sql
EXCEPTION WHEN OTHERS THEN
```

**مشکل:**  
هدف این بخش fallback از config `persian` به `simple` است، اما `WHEN OTHERS` هر خطای دیگری را نیز می‌بلعد؛ از جمله خطاهای غیرمرتبط در داده یا تابع.

**اثر:**

- خطاهای واقعی ممکن است پنهان شوند.
- debugging مشکل‌تر می‌شود.
- ممکن است بدون اطلاع، کیفیت search کاهش پیدا کند.

**کیفیت بهتر موردنیاز در آینده:**  
فقط خطای مشخص مربوط به نبود text-search configuration باید مدیریت شود، نه تمام exceptionها.

---

## 21. نوع TypeScript با نام `File` احتمال ابهام با DOM `File` ایجاد می‌کند

**محل دقیق:**  
`docs/06_DATA_SCHEMA.md` → TypeScript Types:

```ts
export interface File {
```

**مشکل:**  
در محیط browser و TypeScript، یک نوع سراسری `File` برای فایل‌های انتخاب‌شده در input وجود دارد. هرچند export شدن interface در module از نظر فنی الزاماً خطا نیست، این نام‌گذاری در importها و فرم‌های upload می‌تواند خوانایی را کاهش دهد و احتمال اشتباه بین Database File و browser `File` را افزایش دهد.

**اثر:**  
مشکل امنیتی مستقیم ندارد، اما code quality و maintainability را پایین می‌آورد.

---

## 22. Dashboard در هدف MVP آمده، اما در فهرست قوانین و Taskهای Phaseها تعریف نشده است

**محل دقیق:**

- `docs/02_PROJECT_GOAL.md`:
  ```text
  Dashboard: total templates count, recent additions
  ```
- `docs/01_RULES.md` → MVP includes، بدون اشاره به Dashboard.
- `docs/08_PROJECT_PHASES_AND_TASKS.md` → بدون Task مشخص برای Dashboard.

**مشکل:**  
محدودهٔ Dashboard متناقض است. Rule 1 و Rule 12 می‌گویند در صورت تعارض باید Clarification انجام شود.

**اثر:**  
عامل کدنویسی ممکن است Dashboard را پیاده‌سازی نکند، یا بدون Task و خارج از scope آن را اضافه کند.

---

## 23. Pagination الزامی است اما قرارداد آن مشخص نیست

**محل دقیق:**  
`docs/05_TECH_SPEC.md` → API Design:

```text
Pagination: cursor-based for large datasets
```

**مشکل:**  
تعریف نشده است:

- cursor بر چه مبنایی تولید می‌شود؛
- sort پیش‌فرض چیست؛
- ساختار پاسخ pagination چیست؛
- limit پیش‌فرض و حداکثر چیست؛
- تعامل pagination با dimension search و full-text search چگونه است.

**اثر:**  
APIهای لیست و search ممکن است ناسازگار طراحی شوند.

---

## 24. Rate limiting برای endpoint عمومی الزام شده، اما راهکار آن تعریف نشده است

**محل دقیق:**  
`docs/05_TECH_SPEC.md` → API Design:

```text
Rate limiting on all public endpoints
```

**مشکل:**  
هیچ‌کدام از موارد زیر تعیین نشده‌اند:

- ابزار یا سرویس rate limiting؛
- وابستگی مجاز؛
- کلید rate limit، مانند IP یا token؛
- محدودیت تعداد درخواست؛
- پاسخ HTTP هنگام عبور از حد؛
- رفتار در Vercel serverless یا Edge Runtime.

**اثر:**  
قانون امنیتی وجود دارد اما قابلیت پیاده‌سازی مشخص نیست؛ همچنین افزودن سرویس جدید ممکن است با Rule 1 نیازمند ADR باشد.

---

## جمع‌بندی

مهم‌ترین ریسک پروژه در وضعیت فعلی، **طراحی ناامن RLS برای لینک‌های اشتراک عمومی** است. policyهای ارائه‌شده به‌جای محدودکردن کاربر ناشناس به یک توکن مشخص، تمام templateها و فایل‌های دارای لینک فعال را قابل مشاهده می‌کنند. این مورد باید پیش از هر پیاده‌سازی دیتابیس یا endpoint عمومی اصلاح و طراحی شود.

همچنین، پیش از شروع توسعه باید جریان‌های عملیاتی زیر به‌صورت دقیق مشخص شوند:

1. ایجاد و مدیریت تنها admin و profile او؛
2. اعتبارسنجی امن share token؛
3. صدور Signed URL برای کاربران عمومی؛
4. معماری upload و thumbnail برای فایل‌های بزرگ؛
5. مسیرها، Taskها، `CURRENT_TASK.md` و `Allowed Files`؛
6. محدودهٔ واقعی CRUD و Dashboard.





