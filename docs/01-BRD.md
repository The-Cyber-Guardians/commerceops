# 01 — BRD: سند نیازمندی‌های کسب‌وکار CommerceOps

## 1. مشخصات سند

| فیلد | مقدار |
|---|---|
| نام محصول | CommerceOps |
| عنوان | Multi-tenant Order, Inventory & Fulfillment Management Platform |
| نسخه | 1.0.0 |
| Release هدف | 1.0 MVP |
| افق تحویل | هشت هفته |
| وضعیت | Approved |

## 2. خلاصه اجرایی

CommerceOps یک سامانه Backend-first برای مدیریت عملیات پس از فروش است. سامانه سفارش را دریافت می‌کند، موجودی را به‌صورت اتمیک رزرو می‌کند، پرداخت شبیه‌سازی‌شده را ثبت می‌کند و مراحل آماده‌سازی، ارسال و تحویل را با کنترل دسترسی، تاریخچه و تست مدیریت می‌کند.

محصول **فروشگاه اینترنتی ویترینی نیست**؛ هسته آن عملیات سفارش و انبار است.

## 3. مسئله کسب‌وکار

کسب‌وکارهایی که سفارش را در فایل، پیام‌رسان یا چند ابزار جدا نگهداری می‌کنند با مشکلات زیر روبه‌رو می‌شوند:

- فروش بیشتر از موجودی واقعی
- نبود تاریخچه معتبر تغییر وضعیت
- خطای انسانی در انتقال سفارش به انبار
- دسترسی بیش‌ازحد کاربران به اطلاعات سایر واحدها
- تکرار ثبت پرداخت یا سفارش در Retry/Webhook
- ناتوانی در تشخیص سفارش‌های معطل
- نبود گزارش قابل اتکا از Stock Movement و عملیات

## 4. چشم‌انداز محصول

ایجاد یک مرجع واحد و قابل ممیزی برای چرخه:

`Order → Inventory Reservation → Payment → Fulfillment → Shipment → Delivery`

به‌گونه‌ای که هر تغییر مهم قابل ردیابی، مجاز، اتمیک و قابل تست باشد.

## 5. اهداف کسب‌وکار

| ID | هدف | معیار موفقیت MVP |
|---|---|---|
| BR-001 | جلوگیری از Overselling | هیچ سناریوی Concurrency نتواند `reserved > on_hand` ایجاد کند. |
| BR-002 | مدیریت یکپارچه چرخه سفارش | هر سفارش از طریق Transitionهای تعریف‌شده به وضعیت نهایی برسد. |
| BR-003 | تفکیک داده سازمان‌ها | هیچ کاربر نتواند داده Organization دیگر را مشاهده یا تغییر دهد. |
| BR-004 | جلوگیری از عملیات تکراری | درخواست دارای Idempotency Key یا Webhook تکراری فقط یک اثر تجاری داشته باشد. |
| BR-005 | قابلیت ممیزی | تغییر وضعیت، موجودی، پرداخت و Permission حساس Audit شود. |
| BR-006 | کاهش خطای عملیاتی | Validation و Constraint از وضعیت‌های نامعتبر جلوگیری کنند. |
| BR-007 | قابلیت ارائه و نگهداری | محیط با Docker اجرا، API مستند و CI سبز باشد. |
| BR-008 | اثبات کیفیت | نیازهای بحرانی دارای Unit، Integration و API Test باشند. |

## 6. ذی‌نفعان و Actors

| Actor | مسئولیت کسب‌وکاری |
|---|---|
| Organization Admin | مدیریت Organization، Membership و Role |
| Sales Operator | ثبت Customer و ایجاد سفارش |
| Warehouse Operator | Picking، Packing و ثبت Shipment |
| Warehouse Manager | دریافت/اصلاح موجودی و نظارت بر انبار |
| Finance Operator | ثبت و بررسی پرداخت شبیه‌سازی‌شده |
| Customer | مشاهده سفارش‌های خود در API محدود یا Demo |
| Platform Admin | مدیریت فنی Organizationها بدون دسترسی پیش‌فرض به عملیات روزمره |
| External Payment Simulator | ارسال نتیجه پرداخت و Webhook |
| Scheduler/Worker | انقضای Reservation و ارسال Notification |

## 7. قابلیت‌های کسب‌وکار

### BR-CAP-01 — Identity and Organization

- ایجاد Organization
- عضویت کاربر در Organization
- نقش و Permission
- محدودسازی Resource به Organization

### BR-CAP-02 — Catalog

- Product و Variant
- SKU یکتا در Organization
- فعال/غیرفعال کردن کالا بدون حذف تاریخچه

### BR-CAP-03 — Inventory

- موجودی هر Variant در هر Warehouse
- Stock Movement
- Reservation، Release و Consume
- جلوگیری از موجودی منفی

### BR-CAP-04 — Order Management

- Customer و Address
- ایجاد سفارش و Order Item
- Snapshot قیمت، نام کالا و آدرس
- محاسبه Subtotal و Total
- لغو در وضعیت مجاز

### BR-CAP-05 — Payment

- ساخت Payment Attempt شبیه‌سازی‌شده
- ثبت Webhook با Idempotency
- موفق/ناموفق شدن پرداخت

### BR-CAP-06 — Fulfillment and Shipment

- ایجاد Fulfillment برای سفارش پرداخت‌شده
- Picking و Packing
- Shipment و Tracking Code
- تحویل سفارش

### BR-CAP-07 — Audit and Notification

- ثبت Actor، Action، Entity، Before/After و Trace ID
- ارسال Notification پایه پس از Commit

## 8. محدوده Release 1.0

### 8.1 داخل محدوده

همه قابلیت‌های BR-CAP-01 تا BR-CAP-07 با جزئیات FRD.

### 8.2 خارج از محدوده

- UI کامل مشتری و فروشگاه
- درگاه بانکی واقعی
- Refund واقعی و Reconciliation مالی
- Tax Engine، Promotion Engine و Coupon
- Return Merchandise Authorization کامل
- Multi-currency conversion
- Shipment چندبخشی
- Purchase Order و Supplier Management
- Forecasting و BI پیشرفته
- Event-driven Microservices

## 9. فرضیات

| ID | فرض |
|---|---|
| BA-001 | هر Organization در MVP یک Base Currency دارد. |
| BA-002 | هر Order از یک Warehouse Fulfill می‌شود. |
| BA-003 | هر Order در MVP یک Shipment نهایی دارد. |
| BA-004 | Payment Provider شبیه‌سازی‌شده است. |
| BA-005 | Inventory با عدد صحیح و واحد شمارشی مدیریت می‌شود. |
| BA-006 | زمان انقضای Reservation پیش‌فرض ۱۵ دقیقه و قابل تنظیم است. |
| BA-007 | UI حداقلی یا Django Admin برای Demo کافی است. |

## 10. محدودیت‌ها

- داده‌های Business باید در PostgreSQL ذخیره شوند.
- Redis داده نهایی و منبع حقیقت نیست.
- عملیات موجودی نباید فقط در Cache انجام شود.
- Payment Webhook نباید به اعتماد ورودی بدون Verification شبیه‌سازی‌شده متکی باشد.
- اطلاعات Organization نباید از طریق شناسه URL قابل دورزدن باشد.
- Audit Log نباید از API عمومی قابل Update/Delete باشد.

## 11. KPIهای Demo و پذیرش

| KPI | هدف |
|---|---:|
| End-to-End Happy Path | 100% موفق |
| Overselling در تست هم‌زمان | 0 |
| نشت Cross-tenant در تست امنیتی | 0 |
| پردازش تکراری Webhook | 0 اثر تکراری |
| Test Coverage کل | حداقل 80% |
| Coverage سرویس‌های دامنه بحرانی | حداقل 90% |
| OpenAPI validation | بدون خطای بحرانی |
| CI روی Pull Request | اجباری و سبز |

## 12. ریسک‌های کسب‌وکار

| ID | ریسک | پاسخ |
|---|---|---|
| R-001 | بزرگ‌شدن Scope | قفل‌کردن Release 1.0 و انتقال بقیه به Backlog |
| R-002 | پیچیدگی Concurrency | اولویت‌دادن به Reservation و تست هم‌زمان از هفته سوم |
| R-003 | ابهام وضعیت‌ها | ماشین حالت واحد در Workflow Document |
| R-004 | تناقض API و Data Model | Traceability و Review هم‌زمان Migration/API |
| R-005 | وابستگی به سرویس خارجی | Providerهای Fake و Adapter Interface |
| R-006 | تأخیر تست | تست همراه Feature، نه در انتهای پروژه |

## 13. معیار پذیرش نهایی Release

Release 1.0 پذیرفته است اگر:

1. سناریوی کامل ایجاد سفارش تا تحویل اجرا شود.
2. سناریوی انقضای Reservation موجودی را آزاد کند.
3. دو درخواست هم‌زمان نتوانند یک واحد موجودی را دوبار رزرو کنند.
4. Webhook تکراری نتیجه تکراری ایجاد نکند.
5. مجوزهای Organization و Warehouse در API تست شوند.
6. Docker Compose سرویس‌ها را از محیط پاک اجرا کند.
7. Migrationها از صفر بدون خطا اجرا شوند.
8. CI شامل lint، type/static check، test و coverage موفق باشد.
9. OpenAPI و README با پیاده‌سازی تطبیق داشته باشند.
10. هیچ Requirement بحرانی بدون Test Case باقی نماند.

## 14. Glossary

| واژه | تعریف |
|---|---|
| Organization | مرز Tenant و مالک داده‌های کسب‌وکار |
| SKU | شناسه تجاری یکتای Product Variant در Organization |
| On Hand | موجودی فیزیکی ثبت‌شده |
| Reserved | بخشی از On Hand که برای سفارش فعال نگه داشته شده |
| Available | `on_hand - reserved` |
| Reservation | قفل تجاری موقت موجودی برای Order |
| Consume | کاهش هم‌زمان On Hand و Reserved هنگام نهایی‌شدن خروج کالا |
| Fulfillment | فرایند آماده‌سازی سفارش در انبار |
| Idempotency | اجرای چندباره درخواست با یک کلید و ایجاد تنها یک اثر تجاری |
| Snapshot | کپی تغییرناپذیر داده در زمان سفارش، مانند قیمت و آدرس |
