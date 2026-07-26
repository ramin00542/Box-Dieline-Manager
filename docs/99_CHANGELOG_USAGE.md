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
