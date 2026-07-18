# 00 — کنترل اسناد و منبع حقیقت

## 1. هدف

این سند تعیین می‌کند هنگام اختلاف یا تغییر، کدام فایل مرجع نهایی است و چگونه از تناقض جلوگیری شود.

## 2. سلسله‌مراتب اعتبار

| اولویت | سند | مسئول پاسخ به سؤال |
|---:|---|---|
| 1 | BRD | چرا محصول ساخته می‌شود، ارزش و محدوده چیست؟ |
| 2 | FRD | سیستم از دید کاربر و API چه رفتاری دارد؟ |
| 3 | Business Workflows | تغییر وضعیت‌ها و قواعد دامنه دقیقاً چگونه‌اند؟ |
| 4 | SRD | سیستم چگونه پیاده‌سازی و اجرا می‌شود؟ |
| 5 | Data Model | داده‌ها، روابط و قیود پایگاه داده چیست؟ |
| 6 | API Contract | ورودی، خروجی و خطاهای HTTP چیست؟ |
| 7 | Test Strategy | چگونه درستی نیازها اثبات می‌شود؟ |
| 8 | Delivery Plan | چه زمانی و با چه ترتیب تحویل می‌شود؟ |

**قاعده تعارض:** سند پایین‌تر اجازه ندارد رفتار یا محدوده سند بالاتر را تغییر دهد. در صورت تعارض، پیاده‌سازی متوقف و یک Change Request ثبت می‌شود.

## 3. شناسه نیازمندی‌ها

| پیشوند | نوع |
|---|---|
| `BR-` | Business Requirement |
| `FR-` | Functional Requirement |
| `NFR-` | Non-Functional Requirement |
| `BRULE-` | Business Rule |
| `API-` | API Contract Item |
| `TC-` | Test Case |
| `ADR-` | Architecture Decision Record |

شناسه حذف یا بازیافت نمی‌شود. نیاز حذف‌شده با وضعیت `Retired` باقی می‌ماند.

## 4. واژگان الزام

- **باید / SHALL:** الزام Release 1.0 و قابل تست
- **نباید / SHALL NOT:** رفتار ممنوع و قابل تست
- **بهتر است / SHOULD:** توصیه مهم، اما مانع Release نیست
- **می‌تواند / MAY:** اختیاری
- **خارج از محدوده:** نباید در Release 1.0 پیاده‌سازی شود مگر با Change Request

## 5. وضعیت نیازمندی

`Draft → Reviewed → Approved → Implemented → Verified`

حالت‌های جانبی: `Deferred`, `Changed`, `Retired`.

## 6. قواعد تغییر

تغییرات زیر نیازمند Change Request و ADR هستند:

- افزودن یا حذف موجودیت اصلی
- تغییر Workflow سفارش، پرداخت یا موجودی
- تغییر API ناسازگار با نسخه قبل
- تغییر معماری Modular Monolith
- تغییر روش Multi-tenancy
- تغییر روش رزرو و قفل موجودی
- ورود سرویس خارجی واقعی

Change Request باید شامل دلیل، اسناد متأثر، Migration، ریسک، تست و اثر بر برنامه تحویل باشد.

## 7. Versioning اسناد

Semantic Versioning برای کل بسته:

- Patch: اصلاح نگارشی بدون تغییر رفتار
- Minor: نیاز جدید Backward-compatible
- Major: تغییر Scope یا قرارداد ناسازگار

## 8. Definition of Ready برای نیازمندی

یک Requirement آماده توسعه است اگر:

- شناسه یکتا دارد.
- Actor و Trigger مشخص‌اند.
- Precondition و Outcome روشن‌اند.
- Acceptance Criteria قابل تست است.
- خطاها و Permissionها تعریف شده‌اند.
- وابستگی Data/API شناخته شده است.
- ابهام یا `TBD` حل‌نشده ندارد.

## 9. Definition of Done برای سند

- تمام لینک‌های داخلی معتبرند.
- شناسه‌های اشاره‌شده وجود دارند.
- واژه‌های Glossary یکسان استفاده شده‌اند.
- Scope آن با BRD تطبیق دارد.
- سناریوی Happy Path و Failure Path دارد.
- Traceability Matrix به‌روز شده است.

## 10. فهرست تصمیم‌های قفل‌شده

| تصمیم | وضعیت | مرجع |
|---|---|---|
| معماری Modular Monolith | قفل‌شده | ADR-001 |
| Multi-tenancy با `organization_id` | قفل‌شده | SRD-4.4 |
| رزرو موجودی با Transaction و Row Lock | قفل‌شده | ADR-002 |
| Side Effect پس از Commit | قفل‌شده | ADR-003 |
| درگاه پرداخت در MVP شبیه‌سازی‌شده | قفل‌شده | BRD Scope |
| REST API نسخه‌بندی‌شده | قفل‌شده | API Contract |
