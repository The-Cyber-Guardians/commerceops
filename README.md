# CommerceOps — بسته مستندات شروع پروژه

> **نسخه سند:** 1.0.0  
> **تاریخ مبنا:** 2026-07-16  
> **وضعیت:** Approved for MVP Implementation  
> **دامنه انتشار:** Release 1.0 (هشت‌هفته‌ای)

CommerceOps یک پلتفرم چندسازمانی برای مدیریت چرخه سفارش، رزرو موجودی، عملیات انبار، پرداخت شبیه‌سازی‌شده و ارسال است. این مخزن مستندات طوری طراحی شده است که توسعه بدون ابهام از روی آن آغاز شود.

## ترتیب مطالعه

1. [`docs/00-DOCUMENT-CONTROL.md`](docs/00-DOCUMENT-CONTROL.md) — قواعد کنترل و رفع تعارض اسناد
2. [`docs/01-BRD.md`](docs/01-BRD.md) — نیازمندی‌های کسب‌وکار و محدوده محصول
3. [`docs/02-FRD.md`](docs/02-FRD.md) — نیازمندی‌های عملکردی و رفتار قابل مشاهده
4. [`docs/03-SRD.md`](docs/03-SRD.md) — معماری، قیود فنی و نیازمندی‌های غیرعملکردی
5. [`docs/04-DOMAIN-DATA-MODEL.md`](docs/04-DOMAIN-DATA-MODEL.md) — مدل دامنه، ERD، قیود و ایندکس‌ها
6. [`docs/05-BUSINESS-WORKFLOWS.md`](docs/05-BUSINESS-WORKFLOWS.md) — ماشین حالت و قوانین کسب‌وکار
7. [`docs/06-API-CONTRACT.md`](docs/06-API-CONTRACT.md) — قرارداد API نسخه اول
8. [`docs/07-TEST-STRATEGY.md`](docs/07-TEST-STRATEGY.md) — راهبرد تست و سناریوهای پذیرش
9. [`docs/08-DEVELOPMENT-WORKFLOW.md`](docs/08-DEVELOPMENT-WORKFLOW.md) — Workflow توسعه، Git، PR و Definition of Done
10. [`docs/09-DELIVERY-PLAN.md`](docs/09-DELIVERY-PLAN.md) — برنامه تحویل هشت‌هفته‌ای
11. [`docs/10-TRACEABILITY-MATRIX.md`](docs/10-TRACEABILITY-MATRIX.md) — نگاشت نیازها به طراحی و تست
12. [`docs/11-SOURCES.md`](docs/11-SOURCES.md) — منابع و استانداردهای مرجع
13. [`docs/12-CHANGELOG.md`](docs/12-CHANGELOG.md) — تاریخچه تغییر اسناد
14. [`docs/13-CONSISTENCY-REPORT.md`](docs/13-CONSISTENCY-REPORT.md) — گزارش کنترل سازگاری

## مرز MVP

### داخل Release 1.0

- احراز هویت مبتنی بر JWT و مدیریت نشست Refresh Token
- Organization، Membership، Role و Permission
- Product، Product Variant، Warehouse و Inventory
- ثبت Customer و Address
- ایجاد سفارش با Snapshot قیمت و آدرس
- رزرو اتمیک موجودی و انقضای رزرو
- پرداخت شبیه‌سازی‌شده و Webhook تکرارپذیر امن
- Workflow سفارش از ثبت تا تحویل یا لغو
- عملیات ساده Picking، Packing و Shipment
- Audit Log، Structured Logging و Trace ID
- Notification غیرهمزمان پایه
- تست‌های Unit، Integration، API، Permission و Concurrency
- Docker Compose، CI و مستندات OpenAPI

### خارج از Release 1.0

- درگاه پرداخت واقعی و تسویه مالی واقعی
- Marketplace و اتصال فروشگاه‌های خارجی
- سیستم تخفیف، کوپن و مالیات پیشرفته
- مرجوعی و Refund چندمرحله‌ای کامل
- چندارزی هم‌زمان در یک Organization
- Split Shipment و Backorder پیچیده
- Forecasting، هوش مصنوعی و Recommendation
- Microservices، Kubernetes و Event Sourcing
- رابط کاربری Production-grade؛ Django Admin یا UI حداقلی برای Demo کافی است

## فناوری مبنا

- Python 3.13
- Django 5.2 LTS
- Django REST Framework
- PostgreSQL 17
- Redis 7.x
- Celery 5.6
- pytest + pytest-django
- Docker Compose
- GitHub Actions
- OpenAPI 3.x

نسخه‌های Patch باید در lock file ثبت شوند و بدون ADR یا Pull Request وابستگی اصلی تغییر نکند.

## اصل معماری

پروژه یک **Modular Monolith** است. API، Application، Domain و Infrastructure از هم جدا هستند، اما همه ماژول‌ها در یک Deployment و یک PostgreSQL اجرا می‌شوند.

## قراردادهای سراسری

- API Prefix: `/api/v1`
- شناسه عمومی منابع: UUID v4
- زمان‌ها: UTC و ISO 8601
- پول: `Decimal(18,2)` به‌همراه `currency` سه‌حرفی
- حذف رکوردهای عملیاتی: Soft Delete فقط در جاهایی که صراحتاً تعریف شده؛ تراکنش‌های مالی، سفارش و Audit حذف فیزیکی نمی‌شوند
- تمام Queryهای تجاری باید با `organization_id` محدود شوند
- عملیات حساس باید Audit Log داشته باشند
- عملیات چندمرحله‌ای تغییر وضعیت و موجودی باید Transactional باشند

## وضعیت اسناد

همه اسناد این بسته با نسخه `1.0.0` سازگارند. هر تغییری که محدوده، Workflow، Data Model یا API را عوض کند، باید هم‌زمان در Traceability Matrix و Changelog ثبت شود.
