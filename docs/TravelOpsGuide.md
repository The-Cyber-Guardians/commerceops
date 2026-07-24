# TravelOps — نسخه ساده‌شده و قابل‌اجرا

> برای یک تیم ۳ نفره مبتدی. هدف: ساخت یک پروژه Backend واقعی و قابل ارائه.

---

## ۰. تصمیم‌های قفل‌شده (Locked Decisions)

این تصمیم‌ها با تیم نهایی شده‌اند و پایه‌ی بقیه‌ی این سند هستند:

* **D1 — ترتیب کار:** فعلاً فقط **بک‌اند + Swagger/OpenAPI** ساخته می‌شود. فرانت‌اند بعداً اضافه می‌شود.
* **D2 — User و Customer:** مشتری (Customer) یک نوع **User است و باید لاگین کند** (نه صرفاً یک رکورد اطلاعاتی).
  کاربر **موقع ثبت‌نام** انتخاب می‌کند که به‌عنوان «آژانس مسافرتی» ثبت‌نام کند یا به‌عنوان «مشتری» که دنبال تور می‌گردد.
* **D3 — زمان کم شدن ظرفیت:** ظرفیت **موقع پرداخت** کم می‌شود، نه موقع ساختِ رزرو. هر کس زودتر پرداخت کند، تور مالِ اوست.
* **D4 — وضعیت تور:** هر `Tour` فیلد `status` دارد (`DRAFT → PUBLISHED → CLOSED`). تا Publish نشود، مشتری تور را نمی‌بیند.
* **D5 — دامنه‌ی پروژه:** پروژه فقط دمو/تمرینی است. تنها بخشی که باید جدی و درست پیاده شود **JWT + Permissions** است.

جزئیات هر کدام در بخش‌های مربوطه‌ی همین سند آمده است.

---

## ۱. چه چیزی می‌سازیم؟

یک API بک‌اند برای مدیریت آژانس‌های مسافرتی و فرایند کامل رزرو تور:

1. ثبت‌نام و ورود با JWT (با انتخاب نقش «آژانس» یا «مشتری» در ثبت‌نام — نگاه کن D2)
2. مدیریت آژانس‌ها و کاربران
3. مدیریت مشتریان
4. ایجاد و مدیریت تورهای داخلی و خارجی
5. مدیریت مقصد و برنامه سفر
6. مدیریت هتل، حمل‌ونقل و راهنمای تور
7. ثبت و مدیریت رزرو
8. مدیریت ظرفیت تور
9. پرداخت Mock
10. مشاهده تاریخچه سفر و گزارش‌های ساده

> **توجه (D1):** در این مرحله فقط بک‌اند به‌همراه Swagger/OpenAPI ساخته می‌شود. فرانت‌اند بعداً اضافه می‌شود؛
> بخشِ «انتخاب نقش در ثبت‌نام» هم در همان مرحله‌ی فرانت‌اند کامل می‌شود.

---

## ۲. تکنولوژی‌ها

* Python
* Django
* Django REST Framework
* PostgreSQL
* JWT
* Docker / Docker Compose
* Git / GitHub
* GitHub Actions
* Pytest
* Ruff
* Swagger / OpenAPI

---

## ۳. ماژول‌های اصلی

```text
travelops/
├── accounts/       # User، JWT، Organization، Permission
├── customers/      # Customer
├── tours/          # Tour، Destination، Itinerary، Hotel، Transportation، TourGuide
├── reservations/   # Reservation، Payment، Workflow، Capacity
├── reports/        # Dashboard و گزارش‌های ساده
├── common/         # ابزارهای مشترک
└── tests/
```

---

## ۴. مدل داده

```text
User
Organization
Membership
Customer
Destination
Tour
ItineraryItem
Hotel
Transportation
TourGuide
Reservation
Payment
ReservationStatusHistory
```

روابط اصلی:

```text
Organization
├── Users / Memberships
├── Customers
├── Tours
├── Destinations
├── Hotels
├── Transportation
└── TourGuides

Customer ──< Reservation >── Tour
Reservation ──< Payment
Reservation ──< StatusHistory
Tour ──< ItineraryItem
```

### Customer به‌عنوان یک User (D2)

مشتری یک **کاربرِ لاگین‌کننده** است، نه فقط یک رکورد داده. رفتار مورد انتظار:

* موقع ثبت‌نام، کاربر انتخاب می‌کند «آژانس مسافرتی» است یا «مشتری». این انتخاب نقشِ او را تعیین می‌کند.
* مشتری با همان حساب لاگین می‌کند، تورهای منتشرشده را می‌بیند و برای خودش رزرو می‌سازد.
* از نظر مفهومی، `Customer` به یک `User` وصل است (پیاده‌سازی دقیقِ این اتصال — مثل `OneToOne` یا فیلدِ نقش —
  تصمیمِ مرحله‌ی کدنویسی است و در این سندِ مفهومی قطعی نمی‌شود).

### وضعیت تور: فیلد status (D4)

`Tour` یک فیلد `status` دارد:

```text
DRAFT → PUBLISHED → CLOSED   (یا ARCHIVED)
```

* کارمند آژانس تور را در حالت `DRAFT` می‌سازد و کامل می‌کند (هتل، راهنما، برنامه‌ی سفر و...).
* تا وقتی تور **Publish** نشده، مشتری/کاربر عادی آن را **اصلاً نمی‌بیند** («انگار وجود ندارد») و نمی‌تواند رزروش کند.
* تورهای `DRAFT` فقط برای کارمندِ **همان آژانس** قابل‌مشاهده‌اند. (تشبیه: مثل پستِ بلاگ که Draft و Published دارد.)

---

## ۵. Workflow رزرو

```text
PENDING
→ UNDER_REVIEW
→ APPROVED
→ WAITING_FOR_PAYMENT
→ PAID
→ CONFIRMED
→ READY_FOR_TRAVEL
→ IN_PROGRESS
→ COMPLETED
```

مسیرهای جایگزین:

```text
PENDING → REJECTED
PENDING → CANCELLED
WAITING_FOR_PAYMENT → PAYMENT_EXPIRED
CONFIRMED → CANCELLED
```

هر تغییر وضعیت باید از طریق Transition معتبر انجام شود.

> **نکته‌ی هماهنگی با ظرفیت (D3):** کم شدن ظرفیتِ تور دقیقاً روی گذارِ **به‌سمت `PAID`** (لحظه‌ی پرداخت موفق) اتفاق می‌افتد،
> نه هنگام ساختِ رزرو در حالت `PENDING`. جزئیات در بخش ۶.

---

## ۶. مدیریت ظرفیت (D3)

هر Tour ظرفیت مشخص دارد.

```text
Capacity = 20
Reserved = 8   (تعداد کسانی که پرداخت کرده‌اند)
Remaining = 12
```

قانون اصلی: **ظرفیت موقعِ پرداخت کم می‌شود، نه موقع ساختِ رزرو.**

* تا وقتی مشتری پرداخت نکرده، تور برایش رزرو **نمی‌شود**؛ فقط می‌بیند که آن تور را در حالت `PENDING` دارد.
* چند مشتری می‌توانند هم‌زمان همان تور را در حالت `PENDING` داشته باشند. **هر کس زودتر پرداخت کند، تور مالِ اوست.**
* با پرداخت موفق، ظرفیت باقی‌مانده یک واحد کم می‌شود؛ با لغوِ یک رزروِ پرداخت‌شده، ظرفیت آزاد می‌گردد.

**پیامد معماری:** نقطه‌ی جلوگیری از Overbooking و قفلِ همزمانی (`Transaction` / `select_for_update`) باید روی
**مرحله‌ی پرداخت** بنشیند، نه روی ساختِ رزرو. یعنی هنگام پرداخت، ظرفیت با قفل بررسی و کم می‌شود تا دو پرداختِ
هم‌زمان نتوانند از ظرفیت عبور کنند.

---

## ۷. Multi-Tenancy

هر Organization اطلاعات مستقل خود را دارد.

* کارمندان یک آژانس فقط داده‌های Organization خودشان را می‌بینند.
* دسترسی Cross-tenant برای داده‌های مدیریتیِ آژانس (مثل لیست مشتریان، تورهای `DRAFT`، گزارش‌ها) ممنوع است:
  کارمند آژانس A هیچ‌وقت داده‌های مدیریتیِ آژانس B را نمی‌بیند.
* منابعی مانند Tour، Hotel و TourGuide متعلق به همان Organization هستند.

### جایگاه مشتری در مدل چند-مستأجری (marketplace)

مشتری یک‌بار در سطح **پلتفرم** ثبت‌نام می‌کند (نه زیرمجموعه‌ی یک آژانس خاص):

* مشتری می‌تواند تورهای **Published** همه‌ی آژانس‌ها را مرور و ببیند (مانند یک مارکت‌پلیس — با «مشتری دنبال تور می‌گردد» جور است).
* اما هر `Reservation` به آژانسِ **صاحبِ همان تور** تعلق دارد؛ پس ایزوله‌سازی سمتِ آژانس حفظ می‌شود.
* مشتری فقط رزروها و اطلاعات خودش را می‌بیند.

Tenant Isolation باید با تست‌های API بررسی شود (به‌ویژه اینکه کارمند یک آژانس نتواند به داده‌های مدیریتیِ آژانس دیگر برسد).

---

## ۸. تقسیم کار

### عضو A

* Accounts
* Organization
* JWT
* Permission
* Docker / CI

### عضو B

* Tour
* Destination
* Itinerary
* Hotel
* Transportation
* TourGuide

### عضو C

* Customer
* Reservation
* Payment
* Capacity
* Reports

مالکیت دائمی نیست؛ هر Feature یک Owner و Reviewer دارد.

> **توجه:** تقسیم کارِ دقیق (به‌خصوص مسئولِ رزرو/ظرفیت) هنوز نهایی نشده و بین این سند و `Delivery-Playbook.md`
> ناهماهنگی جزئی وجود دارد. چون فعلاً روی مرحله‌ی Delivery تمرکز نمی‌کنیم (D1)، این بعد از افزودن فرانت‌اند نهایی می‌شود.

---

## ۹. ترتیب یادگیری و توسعه

1. Django و Models
2. DRF و REST API
3. JWT و Permission
4. PostgreSQL و Migration
5. Docker
6. ساخت ماژول‌ها
7. تست و CI
8. OpenAPI و Deployment

همزمان با توسعه، تست نوشته شود و تغییرات مرتب در GitHub ثبت شوند.

> **یادآوری دامنه (D5):** این پروژه دمو/تمرینی است؛ نگرانی‌های production واقعی (مقیاس‌پذیری بالا، ثبت‌نام آژانسِ واقعی و...)
> موضوعیت ندارند. تنها بخشی که باید جدی و درست پیاده شود **JWT + Permissions** است (به‌خاطر انتخاب نقش در ثبت‌نام — نگاه کن D2).

---

## ۱۰. تست‌های مهم

* Authentication
* Permission
* Tenant Isolation
* ایجاد و مدیریت Tour (شامل گذارِ `DRAFT → PUBLISHED` و پنهان‌بودن تورهای `DRAFT` از مشتری)
* Workflow رزرو
* Payment Flow
* کم شدن ظرفیت **در لحظه‌ی پرداخت** و سناریوی «هر کس زودتر پرداخت کند» (D3)
* جلوگیری از Overbooking
* Transaction و Rollback
* Transitionهای نامعتبر
* Status History
* Travel History

---

## ۱۱. چیزهایی که فعلاً نمی‌سازیم

* درگاه پرداخت واقعی
* SMS و Email واقعی
* اپلیکیشن موبایل
* رزرو مستقیم پرواز و هتل
* GDS
* CRM و حسابداری کامل
* Microservices
* Kubernetes
* AI و Dynamic Pricing
* Celery و سیستم‌های Async پیچیده

---

## ۱۲. بعداً در صورت داشتن زمان

* فرانت‌اند (طبق D1 مرحله‌ی بعدی است)
* Redis و Cache
* Celery
* نقش‌های بیشتر
* Audit Log
* Idempotency
* Concurrency Locking پیشرفته
* mypy

---

## ۱۳. معیار موفقیت پروژه

پروژه باید بتواند این مسیر را به‌صورت کامل اجرا کند:

```text
Create Organization
→ Create Customer (User با نقش مشتری)
→ Create Tour (DRAFT)
→ Add Itinerary / Hotel / Transportation
→ Assign Tour Guide
→ Publish Tour (DRAFT → PUBLISHED)
→ Create Reservation (PENDING)
→ Approve Reservation
→ Payment (اینجا ظرفیت کم می‌شود)
→ Confirm Reservation
→ Start Travel
→ Complete Travel
→ Travel History
```

همچنین باید **Multi-Tenancy، Permission، مدیریت ظرفیت و جلوگیری از Overbooking** به‌صورت واقعی تست شده باشند.

هدف پروژه، ساخت یک سیستم کوچک اما واقعی است؛ نه یک CRUD ساده و نه یک سیستم Enterprise پیچیده.
