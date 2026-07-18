# 03 — SRD: سند نیازمندی‌ها و طراحی فنی CommerceOps

## 1. هدف

این سند معماری، فناوری، نیازمندی‌های غیرعملکردی، امنیت، استقرار و قواعد پیاده‌سازی Release 1.0 را تعیین می‌کند.

## 2. فناوری مبنا

| جزء | انتخاب |
|---|---|
| Runtime | Python 3.13 |
| Web Framework | Django 5.2 LTS |
| API | Django REST Framework |
| Database | PostgreSQL 17 |
| Cache/Broker | Redis 7.x |
| Async Jobs | Celery 5.6 |
| Test | pytest, pytest-django, factory_boy |
| API Schema | OpenAPI 3.x |
| Container | Docker Compose |
| CI | GitHub Actions |
| Lint/Format | Ruff |
| Type Check | mypy + django-stubs |

نسخه Patch در lock file تعیین می‌شود. ارتقای Major/Minor وابستگی اصلی نیازمند ADR و اجرای کامل تست‌ها است.

## 3. سبک معماری

### 3.1 Modular Monolith

سیستم یک Deployment واحد و یک Database دارد، اما ماژول‌ها مرز مشخص دارند:

```text
API / Presentation
        ↓
Application / Use Cases
        ↓
Domain / Policies and Rules
        ↓
Infrastructure / ORM, Providers, Tasks
```

### 3.2 ماژول‌ها

```text
src/
├── config/
├── common/
├── identity/
├── organizations/
├── customers/
├── catalog/
├── inventory/
├── orders/
├── payments/
├── fulfillment/
├── notifications/
├── audit/
└── reporting/
```

### 3.3 قانون وابستگی

- Domain نباید DRF، Celery یا HTTP را import کند.
- Application می‌تواند Domain و Interfaceهای Repository/Provider را استفاده کند.
- Infrastructure Interfaceها را پیاده‌سازی می‌کند.
- API فقط Validation سطح Request، Authentication، فراخوانی Use Case و Serialization پاسخ را انجام می‌دهد.
- Business Rule نباید در ViewSet یا Serializer پراکنده شود.

## 4. Multi-tenancy و Security Boundary

### 4.1 مدل Tenant

`Organization` مرز Tenant است. تمام موجودیت‌های Business دارای `organization_id` مستقیم هستند، حتی اگر از رابطه قابل استنتاج باشد.

### 4.2 Organization Context

- Header: `X-Organization-ID`
- Middleware فقط Format را بررسی می‌کند.
- Permission/Service فعال‌بودن Membership را تأیید می‌کند.
- QuerySetهای Business از Manager/Repository scoped استفاده می‌کنند.

### 4.3 دفاع چندلایه

1. API Permission
2. Scoped QuerySet/Repository
3. Domain Ownership Check
4. Foreign Key و Unique Constraint مرکب
5. تست Cross-tenant

PostgreSQL Row Level Security در Release 1.0 الزامی نیست؛ می‌تواند Hardening آینده باشد.

## 5. Authentication و Authorization

### 5.1 Authentication

- Access JWT: عمر پیشنهادی 15 دقیقه
- Refresh JWT: عمر پیشنهادی 7 روز
- Refresh Rotation و Blacklist فعال
- Password hashing با تنظیم امن پیش‌فرض Django
- Secretها فقط از Environment/Secret Store
- Token خام در Database یا Log ذخیره نشود؛ در صورت نیاز hash ذخیره شود

### 5.2 Authorization

ترکیب:

- RBAC: Permission بر اساس Role
- Resource-based: Organization، Customer ownership، Warehouse scope

Permissionها به‌صورت ثابت در Code تعریف و با Migration/Data Seed ایجاد می‌شوند. ساخت Permission دلخواه در UI خارج از MVP است.

## 6. Transaction و Concurrency

### NFR-DATA-001 — Atomic Business Operations

عملیات زیر باید داخل `transaction.atomic()` باشند:

- ایجاد Order و Reservationها
- Payment Success/Failure و تغییر Order
- لغو Order و Release/Refund
- Ship و Consume Inventory
- Inventory Adjustment و Stock Movement

### NFR-DATA-002 — Row Locking

برای تغییر `InventoryItem` باید رکوردها با `select_for_update()` و ترتیب ثابت بر اساس ID قفل شوند تا ریسک Deadlock کاهش یابد.

### NFR-DATA-003 — Short Transactions

Network Call، ارسال Email و Queue Publish طولانی داخل Transaction انجام نمی‌شود.

### NFR-DATA-004 — After Commit

Side Effect غیرتراکنشی با `transaction.on_commit()` یا الگوی Outbox سبک پس از Commit آغاز می‌شود.

### NFR-DATA-005 — Database Constraints

قواعدی که Database می‌تواند تضمین کند باید Constraint داشته باشند؛ از جمله Unique، Check و Foreign Key.

## 7. Idempotency

### 7.1 Client Idempotency

جدول `IdempotencyRecord` شامل Organization، Key، Endpoint، Request Hash، Response Status/Body و Expiry است.

الگوریتم:

1. Lock/Insert Key
2. اگر Key موجود و Hash برابر است، پاسخ ذخیره‌شده بازگردد.
3. اگر Hash متفاوت است، Conflict.
4. Business Operation اجرا شود.
5. پاسخ پس از موفقیت ذخیره شود.

### 7.2 Webhook Idempotency

`provider_event_id` Unique است. Webhook Body و Verification Result ثبت می‌شود، اما Secret و داده حساس Redact می‌شوند.

### 7.3 Task Idempotency

Task باید بر وضعیت فعلی Entity تصمیم بگیرد؛ مثال: Expire فقط Reservation با وضعیت `ACTIVE` و `expires_at <= now`.

## 8. Async Processing

- Broker: Redis
- Taskهای MVP: Expire Reservation، Send Notification، Stalled Order Scan
- Retry فقط برای خطای موقت و با Backoff
- خطای Validation/Domain Retry نمی‌شود
- Task Arguments فقط ID و داده کوچک‌اند، نه ORM Object
- Task باید Structured Log و Trace/Correlation ID داشته باشد
- Result Backend برای Business Truth استفاده نمی‌شود

## 9. API Design

- REST JSON
- Prefix `/api/v1`
- OpenAPI به‌صورت Generated و در CI Validation می‌شود
- Pagination پیش‌فرض Cursor یا Page-number پایدار؛ برای MVP Page-number قابل قبول است
- Filtering صریح و Allowlist شده
- PUT برای Replace کامل استفاده نمی‌شود مگر تعریف شده؛ PATCH برای تغییر جزئی
- Actionهای دامنه Endpoint صریح دارند؛ مانند `/orders/{id}/cancel`
- هیچ Clientی اجازه Set مستقیم Status حساس ندارد

## 10. Error Architecture

### 10.1 Domain Exceptions

نمونه‌ها:

- `InsufficientInventory`
- `InvalidOrderTransition`
- `ReservationExpired`
- `DuplicatePaymentEvent`
- `WarehouseAccessDenied`
- `IdempotencyKeyReused`

### 10.2 Mapping

| نوع | HTTP |
|---|---:|
| Validation | 400 |
| Unauthenticated | 401 |
| Permission | 403 |
| Resource خارج Tenant یا Not Found | 404 |
| State/Concurrency Conflict | 409 |
| Rate Limit | 429 |
| Unexpected | 500 |

## 11. Data and Money

- مبلغ: PostgreSQL `numeric(18,2)` و Python `Decimal`
- Float برای پول ممنوع
- Currency: ISO 4217 code سه‌حرفی
- Quantity: Positive Integer
- Timestamp: `timestamptz`
- UUID برای شناسه عمومی
- Sequence/BigInt داخلی فقط در صورت نیاز Performance
- Snapshot Order Item و Address تغییرناپذیر است

## 12. Logging، Audit و Observability

### NFR-OBS-001 — Structured Logging

Log باید JSON و شامل timestamp، level، service، environment، trace_id، user_id، organization_id و event باشد.

### NFR-OBS-002 — Trace ID

هر Request باید Trace ID داشته باشد؛ از Header معتبر دریافت یا تولید شود و در Response برگردد.

### NFR-OBS-003 — Metrics MVP

حداقل شمارنده‌ها:

- request count/latency/error
- order created
- reservation failed/expired
- payment succeeded/failed/duplicate
- task success/retry/failure

### NFR-OBS-004 — Audit

Audit با Application Log متفاوت است و در PostgreSQL نگهداری می‌شود.

## 13. Security Requirements

### NFR-SEC-001

تمام Endpointهای Business به Authentication نیاز دارند، مگر Login/Refresh و Webhook شبیه‌سازی‌شده.

### NFR-SEC-002

هر Endpoint با ID باید Object-level Authorization داشته باشد.

### NFR-SEC-003

Serializer باید فیلدهای Writable را Allowlist کند تا Mass Assignment رخ ندهد.

### NFR-SEC-004

Rate Limit برای Auth، Webhook و عملیات حساس اعمال شود.

### NFR-SEC-005

Webhook باید Signature، Timestamp و Replay Window را بررسی کند.

### NFR-SEC-006

CORS، Allowed Hosts، CSRF و Secure Headers بر اساس محیط تنظیم شوند.

### NFR-SEC-007

Dependency Scan و Secret Scan در CI اجرا شود.

### NFR-SEC-008

Logها نباید Password، Token، Authorization Header، Cookie یا Secret را ثبت کنند.

### NFR-SEC-009

خطای Cross-tenant باید 404 باشد.

### NFR-SEC-010

Backup و Restore حداقل در محیط Demo مستند و یک‌بار تمرین شود.

## 14. Performance و ظرفیت MVP

اعداد زیر هدف مهندسی Demo هستند، نه SLA قراردادی Production:

| ID | نیاز |
|---|---|
| NFR-PERF-001 | P95 Endpointهای Read معمولی زیر 500ms با Seed Data هدف باشد. |
| NFR-PERF-002 | P95 ایجاد Order بدون زمان Provider زیر 1000ms هدف باشد. |
| NFR-PERF-003 | List Endpointها N+1 Query نداشته باشند. |
| NFR-PERF-004 | Pagination اجباری است؛ page size پیش‌فرض 25 و حداکثر 100. |
| NFR-PERF-005 | Query Count Endpointهای بحرانی تست یا بررسی شود. |
| NFR-PERF-006 | Transactionهای موجودی کوتاه و بدون Network I/O باشند. |

## 15. Reliability

| ID | نیاز |
|---|---|
| NFR-REL-001 | عملیات Business در برابر Retry Client Idempotent باشد، اگر Endpoint آن را الزام کرده است. |
| NFR-REL-002 | Worker restart نباید Business State را خراب کند. |
| NFR-REL-003 | Task شکست‌خورده قابل شناسایی و Retry دستی باشد. |
| NFR-REL-004 | Migration باید Forward-only و قابل اجرای خودکار باشد. |
| NFR-REL-005 | Health endpoints برای App، DB، Redis و Worker تعریف شوند. |

## 16. Maintainability

| ID | نیاز |
|---|---|
| NFR-MAINT-001 | Cyclomatic complexity غیرعادی باید در Review شکسته شود. |
| NFR-MAINT-002 | Public Serviceها Type Hint و Docstring هدف‌محور داشته باشند. |
| NFR-MAINT-003 | Business Rule باید حداقل Unit Test داشته باشد. |
| NFR-MAINT-004 | Migration و Model Change در یک PR باشند. |
| NFR-MAINT-005 | ADR برای تصمیم معماری مهم اجباری است. |
| NFR-MAINT-006 | Deprecated API پیش از حذف حداقل یک Minor Version علامت‌گذاری شود. |

## 17. Repository Structure

```text
commerceops/
├── src/
│   ├── manage.py
│   ├── config/
│   ├── common/
│   └── modules...
├── tests/
│   ├── integration/
│   ├── api/
│   └── concurrency/
├── docs/
├── docker/
├── scripts/
├── pyproject.toml
├── compose.yaml
├── .env.example
└── README.md
```

داخل هر Module:

```text
orders/
├── api/
├── application/
├── domain/
├── infrastructure/
├── models.py
├── migrations/
└── tests/
```

## 18. Configuration

- تنظیمات `base`, `local`, `test`, `production`
- `.env.example` بدون Secret واقعی
- Fail-fast برای Environment Variableهای ضروری
- Debug در Production خاموش
- Timezone داخلی UTC

## 19. CI Pipeline

ترتیب حداقل:

1. Install locked dependencies
2. Ruff format check
3. Ruff lint
4. mypy
5. Django system check
6. Migration consistency check
7. Unit/Integration/API tests
8. Coverage gate
9. OpenAPI schema generation/validation
10. Dependency and secret scan
11. Docker build

## 20. Deployment MVP

Docker Compose شامل:

- `api`
- `postgres`
- `redis`
- `celery-worker`
- `celery-beat`

برای Demo می‌توان Reverse Proxy را افزود، اما الزام Release نیست.

## 21. Backup و Recovery

- Script برای `pg_dump`
- Script برای Restore روی Database خالی
- فایل‌های Upload در MVP حداقلی‌اند؛ اگر Attachment افزوده شد باید Storage جداگانه داشته باشد
- حداقل یک Restore Test پیش از Release

## 22. Architecture Decision Records

- ADR-001: Modular Monolith
- ADR-002: Transactional Inventory Reservation
- ADR-003: Side Effects after Commit
