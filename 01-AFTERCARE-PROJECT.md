# AfterCare — Project Specification

> سند جاری محصول و تفاوت‌های آن با Baseline قبلی

## 1. معماری اسناد

این مجموعه از مدل **Base + Delta** استفاده می‌کند:

1. `SIMPLIFIED-GUIDE(2).md` همان فایل قبلی تیم است و بدون هیچ تغییری نگهداری می‌شود.
2. این سند، محصول AfterCare، Scope، Workflow و تفاوت‌های دامنه‌ای را تعریف می‌کند.
3. `02-AFTERCARE-IMPLEMENTATION-GUIDE.md` راهنمای فنی، تست، تقسیم کار و تحویل است.
4. جزئیات متغیر مانند Task، Bug و Acceptance Criteria هر Feature در GitHub Issues نگهداری می‌شوند.

### ترتیب اعتبار

هنگام تناقض:

1. قواعد محصول و Workflow این سند
2. تصمیم‌های فنی سند Implementation Guide
3. بخش‌های بدون تعارض فایل قبلی
4. GitHub Issue مربوط به Feature

فایل قبلی نباید ویرایش شود. هر تصمیم جدید فقط در دو سند AfterCare یا Issue ثبت می‌شود.

## 2. چه چیزی از پروژه قبلی حفظ می‌شود؟

کار انجام‌شده تیم دور ریخته نمی‌شود. این بخش‌ها همچنان مبنا هستند:

- Django و Django REST Framework
- PostgreSQL و Migration
- JWT login/refresh/logout
- Docker Compose
- pytest و CI
- Swagger/OpenAPI
- Git و Pull Request workflow
- Multi-tenancy پایه
- الگوی REST و Error Response
- استفاده از Decimal برای Money
- یادگیری مرحله‌ای و تقسیم کار سه‌نفره

فقط داستان کسب‌وکار و مدل‌های اصلی از CommerceOps به AfterCare تبدیل می‌شوند.

## 3. تبدیل مفاهیم قبلی به AfterCare

| CommerceOps قبلی | AfterCare جدید | وضعیت کار قبلی |
|---|---|---|
| Organization | Service Center / Organization | حفظ می‌شود |
| User + role | User + Membership + role | توسعه داده می‌شود |
| Customer | Customer | حفظ می‌شود |
| Product | Registered Device یا Part | مفهوم تغییر می‌کند |
| ProductVariant | Device Model یا Part SKU | قابل استفاده مجدد در طراحی |
| Stock quantity | Inventory Item | دقیق‌تر می‌شود |
| Order | Repair Ticket | Workflow جایگزین می‌شود |
| Order Item | Estimate Item و Part Usage | به دو مفهوم تقسیم می‌شود |
| Paid | Estimate Approved | جایگزین می‌شود |
| Shipped | Ready for Pickup | جایگزین می‌شود |
| Delivered | Delivered | حفظ معنای نهایی |
| Order report | Repair dashboard | تغییر Queryها |
| Low-stock report | Low-stock parts | تقریباً حفظ می‌شود |

## 4. تعریف محصول

AfterCare یک پلتفرم Full-stack چندسازمانی برای مدیریت خدمات پس از فروش و تعمیر است.

هر مرکز خدماتی یک `Organization` مستقل است و مشتریان، دستگاه‌ها، کارکنان، درخواست‌های تعمیر، برآوردها، قطعات و موجودی خودش را مدیریت می‌کند. مشتری نیز می‌تواند وضعیت تعمیر، برآورد هزینه و تاریخچه خدمات دستگاه خود را مشاهده کند.

نسخه نمایشی عمومی است و می‌تواند برای ساعت، موبایل، لپ‌تاپ یا تجهیزات الکترونیکی کوچک داده نمونه داشته باشد.

## 5. هدف پروژه

هدف، ساخت پروژه‌ای است که در تابستان توسط یک تیم سه‌نفره و با کار تقریباً یک روز در میان تمام شود، اما از CRUD ساده فراتر برود.

پروژه باید توانایی تیم را در این موارد نشان دهد:

- مدل‌سازی یک فرایند واقعی
- Multi-tenancy و جلوگیری از نشت داده
- Authentication و Authorization
- Workflow و Business Rule
- Transaction و مدیریت صحیح موجودی
- تست Failure Pathها
- Docker، CI و Deployment
- ساخت Demo قابل فهم برای مصاحبه

## 6. مسئله

مراکز خدمات کوچک معمولاً اطلاعات مشتری، دستگاه، عیب‌یابی، قطعات و وضعیت تعمیر را بین دفتر، فایل و پیام‌رسان پخش می‌کنند. نتیجه:

- مشتری وضعیت دقیق تعمیر را نمی‌داند.
- تاریخچه خدمات دستگاه گم می‌شود.
- برآورد و تأیید مشتری مستند نیست.
- تغییر وضعیت‌ها بدون قانون انجام می‌شود.
- مصرف قطعه با موجودی هماهنگ نیست.
- مدیر تعمیرهای معطل را به‌راحتی نمی‌بیند.
- داده چند مرکز ممکن است با هم مخلوط شود.

## 7. نقش‌ها

### Platform Admin

- ایجاد، فعال یا مسدودکردن Organization
- مدیریت فنی پلتفرم
- نداشتن دسترسی روزمره خودکار به عملیات مراکز

### Organization Admin

- مدیریت اعضا
- مشاهده همه تعمیرهای سازمان
- تخصیص Technician
- مدیریت موجودی و Dashboard

### Receptionist

- ثبت Customer و Device
- پذیرش دستگاه و ایجاد Ticket
- ثبت تصویر و وضعیت ظاهری
- تحویل نهایی دستگاه

### Technician

- مشاهده کارهای تخصیص‌یافته
- ثبت Diagnosis و Estimate
- شروع تعمیر و مصرف Part
- ارسال برای Quality Check

### Customer

- مشاهده Deviceهای خودش
- مشاهده Timeline تعمیر
- تأیید یا رد Estimate
- مشاهده Service History

## 8. سناریوی اصلی Demo

1. Platform Admin یک Organization ایجاد می‌کند.
2. Admin کارکنان مرکز را اضافه می‌کند.
3. Receptionist یک Customer و Device ثبت می‌کند.
4. Repair Ticket با توضیح خرابی و تصویر ایجاد می‌شود.
5. Technician تخصیص می‌یابد.
6. Technician عیب‌یابی و Estimate را ثبت می‌کند.
7. Customer Estimate را تأیید می‌کند.
8. Technician تعمیر را شروع و Part مصرف می‌کند.
9. Quality Check انجام می‌شود.
10. دستگاه آماده تحویل و سپس تحویل داده می‌شود.
11. Timeline و Service History نمایش داده می‌شوند.
12. Dashboard تغییرات را نشان می‌دهد.
13. کاربر Organization دیگر نمی‌تواند Ticket را مشاهده کند.

## 9. MVP

### حساب و سازمان

- JWT login، refresh و logout
- User profile
- Organization و Membership
- نقش‌های تعریف‌شده
- انتخاب Organization فعال
- Tenant isolation

### مشتری و دستگاه

- ایجاد و ویرایش Customer
- جستجو با نام، تلفن یا ایمیل
- ثبت Device با نوع، برند، مدل و Serial Number
- مشاهده Service History

### تعمیر

- ایجاد Repair Ticket
- Tracking Number خوانا
- شرح مشکل و وضعیت ظاهری
- Priority و Attachment
- تخصیص Technician
- Diagnosis
- Repair Estimate
- تأیید یا رد توسط Customer
- شروع و تکمیل Repair
- Waiting for Part
- Quality Check
- Ready for Pickup
- Delivery
- Status Timeline

### قطعه و موجودی

- تعریف Part و SKU
- موجودی Part
- Receipt و Adjustment
- مصرف Part در Repair
- جلوگیری از موجودی منفی
- Stock Movement history

### گزارش و ارائه

- تعمیرهای باز
- منتظر تأیید Customer
- منتظر Part
- آماده تحویل
- Low-stock parts
- میانگین تقریبی زمان تعمیر
- OpenAPI، Docker، CI، Seed Data و Deployment

## 10. Workflow تعمیر

```text
RECEIVED
   ↓
DIAGNOSING
   ↓
WAITING_FOR_CUSTOMER_APPROVAL
   ↓
APPROVED
   ↓
REPAIRING
   ↓
QUALITY_CHECK
   ↓
READY_FOR_PICKUP
   ↓
DELIVERED
```

مسیرهای فرعی:

```text
DIAGNOSING → UNREPAIRABLE

WAITING_FOR_CUSTOMER_APPROVAL
    → APPROVED
    → REJECTED_BY_CUSTOMER

REPAIRING
    → WAITING_FOR_PART
    → QUALITY_CHECK

WAITING_FOR_PART → REPAIRING

QUALITY_CHECK
    → READY_FOR_PICKUP
    → REPAIRING
```

وضعیت‌های نهایی:

- `DELIVERED`
- `UNREPAIRABLE`
- `REJECTED_BY_CUSTOMER`
- `CANCELLED`

## 11. قواعد کسب‌وکار

1. همه داده‌های تجاری متعلق به یک Organization هستند.
2. کاربر یک Organization نباید داده Organization دیگر را ببیند یا تغییر دهد.
3. Status از PATCH عمومی تغییر نمی‌کند.
4. هر Transition فقط از وضعیت مجاز و توسط نقش مجاز انجام می‌شود.
5. هر Transition یک Status History تغییرناپذیر ایجاد می‌کند.
6. Technician باید عضو فعال همان Organization باشد.
7. بدون Diagnosis، Estimate ارسال نمی‌شود.
8. تعمیر غیرگارانتی بدون تأیید Estimate شروع نمی‌شود.
9. Estimate تأییدشده Snapshot تاریخی است.
10. تغییر قیمت Part روی Estimate یا Repair قدیمی اثر ندارد.
11. موجودی Part منفی نمی‌شود.
12. مصرف چند Part باید All-or-nothing باشد.
13. Quality Check ناموفق Repair را به مرحله تعمیر برمی‌گرداند.
14. فقط Ticket آماده تحویل قابل Delivery است.
15. Ticket نهایی دوباره وارد Workflow فعال نمی‌شود.
16. Customer فقط Device و Repair خودش را می‌بیند.
17. Serial Number در هر Organization یکتا است.
18. حذف فیزیکی Status History و Part Usage مجاز نیست.
19. نوع و اندازه Attachment محدود است.
20. اجرای دوباره عملیات حساس نباید اثر تکراری مخرب ایجاد کند.

## 12. مدل دامنه سطح بالا

```text
User ──< Membership >── Organization

Organization ──< Customer
Customer ──< RegisteredDevice
RegisteredDevice ──< RepairTicket

RepairTicket ──< RepairStatusHistory
RepairTicket ──0..1 Diagnosis
RepairTicket ──0..* RepairEstimate
RepairTicket ──0..* RepairAttachment
RepairTicket ──0..* QualityCheck
RepairTicket ──0..* PartUsage

Part ──1 InventoryItem
InventoryItem ──< StockMovement
PartUsage >── InventoryItem
```

## 13. خارج از MVP

- پرداخت واقعی
- SMS Provider واقعی
- چت Real-time
- اپ موبایل
- فروشگاه اینترنتی
- رزرو وقت
- حسابداری
- خرید از Supplier
- چند انبار پیچیده
- Workflow Builder
- گارانتی قراردادی پیچیده
- Microservices
- Kubernetes
- Elasticsearch
- AI تشخیص خرابی
- چندزبانه‌سازی

## 14. قابلیت‌های اختیاری

بعد از تکمیل MVP حداکثر دو مورد:

- QR Tracking
- Notification غیرهمزمان
- SLA برای Repairهای معطل
- Audit Log محدود
- Cache یک گزارش
- Export CSV

## 15. تست‌های پذیرش حیاتی

- چرخه کامل پذیرش تا تحویل از API و UI اجرا شود.
- User سازمان A با UUID مستقیم نیز نتواند داده سازمان B را ببیند.
- Transition نامعتبر رد شود.
- کمبود یک Part باعث Rollback کل مصرف شود.
- دو درخواست هم‌زمان نتوانند آخرین Part را دو بار مصرف کنند.
- تغییر قیمت Part مبلغ Estimate تأییدشده را تغییر ندهد.
- Customer نتواند Ticket مشتری دیگر را پاسخ دهد.
- Quality Check ناموفق Ticket را با History به `REPAIRING` برگرداند.
- Ticket تحویل‌شده قابل تحویل یا تعمیر مجدد نباشد.

## 16. تعریف پایان پروژه

پروژه تمام است اگر:

- مسیر اصلی Repair از UI اجرا شود.
- Tenant isolation و نقش‌ها تست شده باشند.
- مصرف Part تراکنشی باشد.
- Timeline و Service History نمایش داده شوند.
- Customer بتواند Estimate را پاسخ دهد.
- Swagger و README قابل استفاده باشند.
- پروژه با Docker اجرا شود.
- CI سبز باشد.
- Seed Data و Demo Script آماده باشند.
- نسخه Deployشده وجود داشته باشد.
