# TravelOps — نسخه ساده‌شده و قابل‌اجرا

> برای یک تیم ۳ نفره مبتدی. هدف: ساخت یک پروژه Backend واقعی و قابل ارائه.

---

## ۱. چه چیزی می‌سازیم؟

یک API بک‌اند برای مدیریت آژانس‌های مسافرتی و فرایند کامل رزرو تور:

1. ثبت‌نام و ورود با JWT
2. مدیریت آژانس‌ها و کاربران
3. مدیریت مشتریان
4. ایجاد و مدیریت تورهای داخلی و خارجی
5. مدیریت مقصد و برنامه سفر
6. مدیریت هتل، حمل‌ونقل و راهنمای تور
7. ثبت و مدیریت رزرو
8. مدیریت ظرفیت تور
9. پرداخت Mock
10. مشاهده تاریخچه سفر و گزارش‌های ساده

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

---

## ۶. مدیریت ظرفیت

هر Tour ظرفیت مشخص دارد.

```text
Capacity = 20
Reserved = 8
Remaining = 12
```

با هر Reservation ظرفیت باقی‌مانده کاهش می‌یابد و با لغو Reservation معتبر، ظرفیت آزاد می‌شود.

رزرو بیش از ظرفیت مجاز نیست و عملیات ظرفیت باید با Transaction انجام شود.

---

## ۷. Multi-Tenancy

هر Organization اطلاعات مستقل خود را دارد.

* User فقط داده‌های Organization خودش را می‌بیند.
* Customer فقط اطلاعات خودش را می‌بیند.
* دسترسی Cross-tenant ممنوع است.
* منابعی مانند Tour، Hotel و TourGuide باید متعلق به همان Organization باشند.

Tenant Isolation باید با تست‌های API بررسی شود.

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

---

## ۱۰. تست‌های مهم

* Authentication
* Permission
* Tenant Isolation
* ایجاد و مدیریت Tour
* Workflow رزرو
* Payment Flow
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
→ Create Customer
→ Create Tour
→ Add Itinerary / Hotel / Transportation
→ Assign Tour Guide
→ Publish Tour
→ Create Reservation
→ Approve Reservation
→ Payment
→ Confirm Reservation
→ Start Travel
→ Complete Travel
→ Travel History
```

همچنین باید **Multi-Tenancy، Permission، مدیریت ظرفیت و جلوگیری از Overbooking** به‌صورت واقعی تست شده باشند.

هدف پروژه، ساخت یک سیستم کوچک اما واقعی است؛ نه یک CRUD ساده و نه یک سیستم Enterprise پیچیده.