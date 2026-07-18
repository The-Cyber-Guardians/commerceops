# 02 — FRD: سند نیازمندی‌های عملکردی CommerceOps

## 1. هدف

این سند رفتار قابل مشاهده Release 1.0 را تعریف می‌کند. هر الزام با `FR-` شناسه دارد و باید در API یا رفتار Worker قابل اثبات باشد.

## 2. قرارداد عمومی رفتار

- تمام Endpointهای Business زیر `/api/v1` هستند.
- زمان‌ها UTC و ISO 8601 هستند.
- همه لیست‌ها Pagination دارند.
- منابع تجاری فقط در محدوده Organization فعال کاربر قابل دسترسی‌اند.
- خطاها Envelope یکسان دارند.
- عملیات حساس دارای `trace_id` و Audit Log هستند.
- Requestهای Create Order و Payment Webhook باید Idempotent باشند.

## 3. Actors و دسترسی پایه

| Actor | سطح دسترسی پایه |
|---|---|
| Organization Admin | مدیریت Membership، Role، Catalog و تنظیمات Organization |
| Sales Operator | Customer، Address و Order |
| Warehouse Operator | Fulfillment و Shipment انبار مجاز |
| Warehouse Manager | Inventory، Stock Movement و Fulfillment |
| Finance Operator | Payment و گزارش وضعیت پرداخت |
| Customer | Read-only روی سفارش‌های متعلق به خود |
| Platform Admin | مدیریت وضعیت Organization؛ بدون عبور خودکار از Tenant Boundary |

## 4. Authentication و Session

### FR-AUTH-001 — Login

سیستم باید با ایمیل و رمز معتبر Access Token و Refresh Token صادر کند.

**Acceptance Criteria**

- کاربر غیرفعال Token دریافت نمی‌کند.
- خطای Login نباید مشخص کند ایمیل وجود دارد یا خیر.
- رخداد Login موفق و ناموفق با داده حساس‌زدایی‌شده Log می‌شود.

### FR-AUTH-002 — Refresh

سیستم باید Refresh Token معتبر را به Access Token جدید تبدیل کند.

### FR-AUTH-003 — Refresh Rotation

سیستم باید پس از Refresh، توکن قبلی را Blacklist و Refresh جدید صادر کند.

### FR-AUTH-004 — Logout

سیستم باید Refresh Token ارائه‌شده را باطل کند.

### FR-AUTH-005 — Current User

کاربر باید بتواند Profile، Membershipها و Permissionهای مؤثر خود را مشاهده کند.

### FR-AUTH-006 — Password Change

کاربر Authenticated باید با ارائه رمز فعلی بتواند رمز را تغییر دهد و نشست‌های قبلی را باطل کند.

### FR-AUTH-007 — Rate Limit

Login و Refresh باید Throttle مستقل داشته باشند.

## 5. Organization، Membership و Authorization

### FR-ORG-001 — Create Organization

Platform Admin باید بتواند Organization ایجاد کند. Organization دارای `name`, `slug`, `base_currency`, `timezone`, `status` است.

### FR-ORG-002 — Activate Organization Context

کاربر دارای چند Membership باید Organization فعال را از Header `X-Organization-ID` مشخص کند.

### FR-ORG-003 — Validate Membership

سیستم باید پیش از هر عملیات Business، Active بودن Organization و Membership را بررسی کند.

### FR-ORG-004 — Manage Membership

Organization Admin باید بتواند کاربر را دعوت، فعال، غیرفعال یا Role او را تغییر دهد.

### FR-ORG-005 — Last Admin Protection

سیستم نباید آخرین Organization Admin فعال را غیرفعال یا تنزل نقش دهد.

### FR-ORG-006 — Role and Permission

Permissionها باید بر Action و Resource تعریف شوند؛ مانند `order.create`, `inventory.adjust`, `shipment.update`.

### FR-ORG-007 — Resource Authorization

داشتن Permission عمومی به‌تنهایی کافی نیست؛ Resource باید متعلق به Organization فعال و در صورت نیاز Warehouse مجاز باشد.

### FR-ORG-008 — Cross-tenant Denial

برای Resource متعلق به Organization دیگر، سیستم باید پاسخ 404 بدهد تا وجود Resource افشا نشود.

## 6. Customer و Address

### FR-CUS-001 — Create Customer

Sales Operator باید بتواند Customer با نام، ایمیل و تلفن ایجاد کند.

### FR-CUS-002 — Customer Uniqueness

ایمیل Customer در هر Organization در صورت وجود باید یکتا باشد؛ یک ایمیل می‌تواند در Organization دیگر وجود داشته باشد.

### FR-CUS-003 — Manage Address

سیستم باید چند Address برای Customer نگهداری کند و یک Address پیش‌فرض تعیین شود.

### FR-CUS-004 — Address Snapshot

ویرایش Address اصلی نباید Shipping Address سفارش‌های قبلی را تغییر دهد.

### FR-CUS-005 — Customer Order Visibility

Customer فقط سفارش‌هایی را مشاهده می‌کند که `customer_id` آن‌ها متعلق به حساب اوست.

## 7. Catalog

### FR-CAT-001 — Create Product

کاربر مجاز باید بتواند Product با نام، توضیح و وضعیت فعال ایجاد کند.

### FR-CAT-002 — Create Variant

هر Product باید حداقل یک Variant داشته باشد. Variant شامل SKU، نام، قیمت و وضعیت فعال است.

### FR-CAT-003 — SKU Uniqueness

SKU باید در محدوده Organization یکتا باشد و به حروف بزرگ Normalized شود.

### FR-CAT-004 — Price Validation

قیمت نباید منفی باشد و Currency آن باید با Base Currency سازمان برابر باشد.

### FR-CAT-005 — Deactivate Variant

Variant استفاده‌شده در Order نباید حذف فیزیکی شود؛ فقط غیرفعال می‌شود.

### FR-CAT-006 — Catalog Search

لیست Variantها باید جستجو بر اساس SKU و نام و فیلتر وضعیت داشته باشد.

## 8. Warehouse و Inventory

### FR-INV-001 — Create Warehouse

Organization Admin باید بتواند Warehouse با Code یکتا ایجاد کند.

### FR-INV-002 — Initialize Inventory Item

برای هر Variant و Warehouse باید حداکثر یک Inventory Item وجود داشته باشد.

### FR-INV-003 — Receive Stock

Warehouse Manager باید بتواند موجودی را با Stock Movement نوع `RECEIPT` افزایش دهد.

### FR-INV-004 — Adjust Stock

Warehouse Manager باید Adjustment مثبت یا منفی با Reason ثبت کند.

### FR-INV-005 — Prevent Negative Stock

هیچ عملیات نباید `on_hand < 0`, `reserved < 0` یا `reserved > on_hand` ایجاد کند.

### FR-INV-006 — View Availability

سیستم باید `available = on_hand - reserved` را در پاسخ Inventory ارائه دهد.

### FR-INV-007 — Reserve Inventory

ایجاد Order باید برای هر Item موجودی را در Transaction اتمیک رزرو کند.

### FR-INV-008 — All-or-nothing Reservation

اگر یک Item موجودی کافی ندارد، هیچ‌یک از Itemهای Order نباید رزرو شوند.

### FR-INV-009 — Reservation Expiry

Reservation با وضعیت `ACTIVE` پس از `expires_at` باید توسط Worker منقضی و موجودی آن آزاد شود. Reservation پرداخت‌شده ابتدا `CONFIRMED` می‌شود و دیگر مشمول Expiry نیست.

### FR-INV-010 — Release Reservation

لغو یا شکست پرداخت باید Reservation در وضعیت مجاز (`ACTIVE` یا برای لغو سفارش پرداخت‌شده `CONFIRMED`) را دقیقاً یک‌بار آزاد کند.

### FR-INV-011 — Consume Reservation

هنگام ثبت Shipment، سیستم باید Reservation `CONFIRMED` را Consume کند، `on_hand` و `reserved` را به‌اندازه Quantity ارسال‌شده کاهش دهد و Reservation را `CONSUMED` کند.

### FR-INV-012 — Inventory Audit Trail

هر تغییر `on_hand` باید Stock Movement تغییرناپذیر ایجاد کند.

### FR-INV-013 — Concurrency Safety

دو Transaction هم‌زمان نباید بتوانند بیش از Available موجودی رزرو کنند.

## 9. Order Management

### FR-ORD-001 — Create Order

Sales Operator باید بتواند Order برای Customer، Warehouse و Itemهای معتبر ایجاد کند.

### FR-ORD-002 — Create Order Idempotency

`POST /orders` باید Header `Idempotency-Key` را اجباری کند. تکرار همان Key با Payload یکسان باید همان پاسخ اولیه را برگرداند.

### FR-ORD-003 — Idempotency Conflict

استفاده از Key قبلی با Payload متفاوت باید خطای `IDEMPOTENCY_KEY_REUSED` بدهد.

### FR-ORD-004 — Order Number

سیستم باید Order Number خوانا و یکتا در Organization تولید کند.

### FR-ORD-005 — Order Snapshot

Order Item باید SKU، نام، Unit Price و Currency را Snapshot کند.

### FR-ORD-006 — Address Snapshot

Order باید Shipping Address را به‌صورت Snapshot نگه دارد.

### FR-ORD-007 — Total Calculation

`subtotal = sum(quantity × unit_price)` و در MVP `total = subtotal` است.

### FR-ORD-008 — Initial Status

Order موفق باید در وضعیت `PENDING_PAYMENT` و دارای Reservation فعال باشد.

### FR-ORD-009 — List and Filter

لیست سفارش‌ها باید فیلتر بر اساس status، customer، warehouse، date range و order number داشته باشد.

### FR-ORD-010 — Order Detail

جزئیات باید Itemها، Paymentها، Reservation، Fulfillment، Shipment و Status History مجاز را ارائه دهد.

### FR-ORD-011 — Cancel Unpaid Order

Order در `PENDING_PAYMENT` باید قابل لغو باشد و Reservation آزاد شود.

### FR-ORD-012 — Cancel Paid Order

Order در `PAID` و پیش از شروع Picking فقط توسط Role مجاز قابل لغو است. Payment Simulator باید Refund شبیه‌سازی‌شده فوری ثبت کند؛ سپس Reservation آزاد و Order لغو شود.

### FR-ORD-013 — Reject Invalid Transition

هر Transition خارج از Workflow باید خطای Domain بدهد و هیچ Side Effect نداشته باشد.

### FR-ORD-014 — Status History

هر تغییر وضعیت باید From، To، Actor، Reason و Timestamp ثبت کند.

### FR-ORD-015 — Reservation Expiry

اگر Reservation یک Order در `PENDING_PAYMENT` منقضی شود، سیستم باید Reservation را `EXPIRED`، Order را `EXPIRED` و موجودی رزروشده را دقیقاً یک‌بار آزاد کند.

## 10. Payment

### FR-PAY-001 — Initiate Payment

برای Order در `PENDING_PAYMENT` باید Payment Attempt با Reference یکتا ایجاد شود.

### FR-PAY-002 — Payment Amount

مبلغ Payment باید دقیقاً با `order.total` و Currency سازمان برابر باشد.

### FR-PAY-003 — Webhook Verification

Webhook باید Signature شبیه‌سازی‌شده و Timestamp را بررسی کند.

### FR-PAY-004 — Webhook Idempotency

Event ID Provider باید یکتا باشد. دریافت دوباره همان Event نباید Payment یا Order را دوباره تغییر دهد.

### FR-PAY-005 — Payment Success

Webhook موفق باید در یک Transaction، Payment را `SUCCEEDED`، Reservation را `CONFIRMED` و Order را `PAID` کند؛ فقط اگر Reservation `ACTIVE` و منقضی‌نشده باشد.

### FR-PAY-006 — Late Payment

اگر Reservation منقضی شده باشد، Payment Success نباید Order را PAID کند و باید وضعیت `REVIEW_REQUIRED` برای Payment ثبت شود.

### FR-PAY-007 — Payment Failure

Webhook ناموفق باید Payment را `FAILED`، Order را `PAYMENT_FAILED` و Reservation را آزاد کند.

### FR-PAY-008 — Duplicate Success Protection

یک Order نباید بیش از یک Payment موفق داشته باشد.

### FR-PAY-009 — Simulated Refund

لغو Order پرداخت‌شده در وضعیت مجاز باید Refund Record موفق شبیه‌سازی‌شده ایجاد کند.

## 11. Fulfillment و Shipment

### FR-FUL-001 — Create Fulfillment

کاربر مجاز باید بتواند برای Order در `PAID` یک Fulfillment مربوط به Warehouse سفارش ایجاد کند. ایجاد تکراری باید همان Fulfillment را برگرداند یا Conflict کنترل‌شده بدهد.

### FR-FUL-002 — Start Picking

Warehouse Operator مجاز باید Fulfillment را از `PENDING` به `PICKING` ببرد.

### FR-FUL-003 — Confirm Picked Quantities

Quantity برداشته‌شده باید با Quantity سفارش برابر باشد؛ Partial Picking در MVP مجاز نیست.

### FR-FUL-004 — Pack

پس از تکمیل Picking، Fulfillment باید به `PACKED` منتقل شود.

### FR-FUL-005 — Create Shipment

برای Fulfillment بسته‌بندی‌شده، Shipment با Carrier Name و Tracking Code ایجاد می‌شود.

### FR-FUL-006 — Ship

ثبت Shipment باید Reservation را Consume، Fulfillment را `SHIPPED` و Order را `SHIPPED` کند.

### FR-FUL-007 — Delivery

Warehouse Manager یا Provider Simulator باید Shipment را `DELIVERED` و Order را `DELIVERED` کند.

### FR-FUL-008 — Tracking Uniqueness

Tracking Code در Organization باید یکتا باشد.

### FR-FUL-009 — Warehouse Boundary

Operator فقط Fulfillmentهای Warehouseهای مجاز خود را مشاهده و تغییر می‌دهد.

## 12. Notification و Background Jobs

### FR-ASYNC-001 — Reservation Expiry Task

Worker باید Reservationهای منقضی فعال را با Lock پردازش کند و اجرای تکراری آن بی‌اثر باشد.

### FR-ASYNC-002 — Notification After Commit

Notification فقط پس از Commit موفق تراکنش Business به Queue ارسال شود.

### FR-ASYNC-003 — Retry

خطاهای موقت Provider باید با Backoff و تعداد Retry محدود مجدداً اجرا شوند.

### FR-ASYNC-004 — Task Idempotency

Task تکراری نباید Notification یا تغییر موجودی تکراری ایجاد کند.

### FR-ASYNC-005 — Dead Task Visibility

Task شکست‌خورده نهایی باید Log ساخت‌یافته و قابل مشاهده داشته باشد.

## 13. Audit و Logging

### FR-AUD-001 — Audit Sensitive Actions

حداقل عملیات زیر Audit می‌شوند:

- تغییر Membership و Role
- Receipt/Adjustment موجودی
- Reservation/Release/Consume
- تغییر وضعیت Order
- Payment و Refund
- Fulfillment و Shipment

### FR-AUD-002 — Audit Immutability

Audit Log از API قابل ویرایش یا حذف نیست.

### FR-AUD-003 — Audit Fields

Audit باید شامل Organization، Actor، Action، Entity Type/ID، Before، After، IP، User Agent و Trace ID باشد.

### FR-AUD-004 — Sensitive Data Redaction

Password، Token، Signature کامل و Secret نباید در Log یا Audit ذخیره شوند.

## 14. Error Handling

### FR-ERR-001 — Standard Error Envelope

تمام خطاهای API باید ساختار زیر را داشته باشند:

```json
{
  "error": {
    "code": "INSUFFICIENT_INVENTORY",
    "message": "Requested quantity is not available.",
    "details": {"sku": "SKU-001", "requested": 2, "available": 1},
    "trace_id": "01J..."
  }
}
```

### FR-ERR-002 — Domain Error Mapping

Domain Exceptionها باید به HTTP Status پایدار نگاشت شوند.

### FR-ERR-003 — No Internal Leakage

Stack Trace، SQL، Secret و مسیر داخلی نباید به Client برگردد.

## 15. گزارش‌ها و Queryهای MVP

### FR-REP-001 — Order Summary

سیستم باید تعداد و مبلغ Orderها را بر اساس Status و بازه تاریخ ارائه کند.

### FR-REP-002 — Low Stock

سیستم باید Variantهایی با Available کمتر یا مساوی Threshold را فهرست کند.

### FR-REP-003 — Inventory Ledger

Stock Movementها باید با فیلتر Warehouse، Variant، Type و Date قابل مشاهده باشند.

### FR-REP-004 — Stalled Orders

Orderهای بیش از مدت تنظیم‌شده در وضعیت غیرنهایی باید قابل فهرست باشند.

## 16. نیازهای UI حداقلی Demo

### FR-UI-001

Django Admin یا UI حداقلی باید امکان مشاهده Organization، Product، Inventory، Order، Payment و Shipment را فراهم کند.

### FR-UI-002

عملیات حساس نباید صرفاً با ویرایش مستقیم Model در Admin انجام شوند؛ باید Action یا Service معتبر فراخوانی شود.

## 17. وضعیت‌های نهایی

- Order: `DELIVERED`, `CANCELLED`, `PAYMENT_FAILED`, `EXPIRED`
- Reservation: `CONSUMED`, `RELEASED`, `EXPIRED`؛ وضعیت میانی پس از پرداخت `CONFIRMED` است
- Payment: `SUCCEEDED`, `FAILED`, `REFUNDED`, `REVIEW_REQUIRED`
- Fulfillment: `SHIPPED`, `CANCELLED`
- Shipment: `DELIVERED`, `FAILED`
