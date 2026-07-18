# 07 — راهبرد تست CommerceOps

## 1. هدف

اثبات اینکه نیازهای کسب‌وکار، Workflow، امنیت Tenant، Concurrency و API مطابق اسناد اجرا شده‌اند.

## 2. سطوح تست

| سطح | تمرکز | ابزار/روش |
|---|---|---|
| Unit | Domain Policy، محاسبات و Transition | pytest بدون DB در صورت امکان |
| Service/Application | Use Case با Fake Repository/Provider | pytest |
| Repository Integration | ORM، Constraint، Query و Transaction | pytest-django + PostgreSQL |
| API | Auth، Validation، Permission، Status Code | DRF APIClient |
| Concurrency | Race Condition و Row Lock | TransactionTestCase/threads/processes |
| Async | Celery task idempotency/retry | eager + integration worker where needed |
| End-to-End | سناریوی کامل Business | API + DB + Worker |
| Security | Cross-tenant، BOLA، mass assignment | API negative tests |

## 3. Coverage Gate

- کل پروژه: حداقل 80%
- Domain/Application ماژول‌های Inventory، Order، Payment: حداقل 90%
- Branch coverage برای Transitionها الزامی
- Migration و generated code از Gate مناسب مستثنی می‌شوند

Coverage عدد نهایی کیفیت نیست؛ Requirement بحرانی بدون Test حتی با Coverage بالا ناقص است.

## 4. Test Data Rules

- Factoryها Tenant-aware هستند.
- داده Organization A و B در Security tests هم‌زمان ساخته می‌شود.
- زمان با Clock/Freezer کنترل می‌شود.
- UUID و Money deterministic در Assertionها.
- تست‌ها نباید به ترتیب اجرا وابسته باشند.
- تست‌ها نباید اینترنت واقعی را فراخوانی کنند.

## 5. Critical Acceptance Tests

### TC-E2E-001 — Happy Path Order to Delivery

**Maps:** BR-002, FR-ORD-001, FR-PAY-005, FR-FUL-006, FR-FUL-007

1. Create Organization, Warehouse, Customer, Variant and Stock.
2. Create Order with Idempotency Key.
3. Assert reservation ACTIVE and inventory reserved.
4. Initiate Payment.
5. Send valid success Webhook.
6. Assert Order PAID and Reservation CONFIRMED.
7. Create Fulfillment, start and confirm Picking, pack.
8. Ship.
9. Assert stock consumed and Order SHIPPED.
10. Mark delivered.
11. Assert Order DELIVERED and histories/audits exist.

### TC-CON-001 — Concurrent Reservation Prevents Overselling

**Maps:** BR-001, FR-INV-013, BRULE-INV-004

- Available stock = 1.
- دو Transaction هم‌زمان هرکدام quantity=1 درخواست می‌کنند.
- دقیقاً یکی موفق و دیگری 409 می‌شود.
- `on_hand=1`, `reserved=1` و یک Reservation ACTIVE وجود دارد.

### TC-IDEM-001 — Duplicate Create Order

**Maps:** BR-004, FR-ORD-002

- Request یکسان با Key یکسان دو بار ارسال شود.
- Response و Order ID یکسان باشد.
- تنها یک Order و یک Reservation ایجاد شود.

### TC-IDEM-002 — Key Reuse with Different Payload

- Key یکسان با Quantity متفاوت.
- 409 `IDEMPOTENCY_KEY_REUSED`.
- Order جدید ایجاد نشود.

### TC-PAY-001 — Duplicate Payment Webhook

- Event ID یکسان دوبار ارسال شود.
- تنها یک Payment transition و یک Order status history ثبت شود.

### TC-PAY-002 — Late Success after Expiry

- Reservation ACTIVE منقضی و آزاد شود.
- سپس Success Webhook برسد.
- Payment → REVIEW_REQUIRED؛ Order → EXPIRED باقی بماند؛ Stock رزرو نشود.

### TC-TENANT-001 — Cross-tenant Order Access

- User در Org A، Order Org B را با UUID مستقیم درخواست کند.
- Response 404.
- Audit/Log داده حساس Org B را نمایش ندهد.

### TC-AUTHZ-001 — Warehouse Scope

- Operator فقط Warehouse A مجاز باشد.
- Fulfillment Warehouse B در List دیده نشود و Detail آن 404 باشد.

### TC-INV-001 — All-or-nothing Multi-item Reservation

- Item A کافی و Item B ناکافی.
- Order ایجاد نشود.
- Reserved هر دو Item بدون تغییر بماند.

### TC-INV-002 — Expiry Idempotency

- Task Expire دو بار اجرا شود.
- Reserved فقط یک بار کاهش یابد.
- تنها یک Transition نهایی ثبت شود.

### TC-CANCEL-001 — Cancel Unpaid

- Order PENDING_PAYMENT لغو شود.
- Reservation RELEASED و reserved کاهش یابد.

### TC-CANCEL-002 — Cancel Paid before Picking

- Payment موفق، Order PAID.
- Role مجاز Cancel کند.
- Refund SUCCEEDED، Reservation RELEASED، Order CANCELLED.

### TC-CANCEL-003 — Reject Cancel after Picking

- Order PROCESSING.
- Cancel → 409؛ هیچ Side Effect.

### TC-SHIP-001 — Ship Is Atomic

خطا در ایجاد Stock Movement شبیه‌سازی شود:

- Order/Fulfillment/Reservation/Inventory هیچ‌کدام نیمه‌تغییرکرده نباشند.

## 6. Module Test Checklist

### Authentication

- login success/failure
- inactive user
- refresh rotation
- revoked refresh
- password change invalidates sessions
- throttle

### Organization/Authorization

- membership status
- last admin protection
- permission matrix
- cross-tenant list/detail/action
- warehouse scope

### Catalog

- SKU normalization/uniqueness
- negative price
- inactive variant order rejection
- snapshot unaffected by price changes

### Inventory

- receipt/adjustment
- check constraints
- reservation/release/consume
- expiration
- concurrency/deadlock order
- ledger accuracy

### Order

- totals
- address snapshot
- duplicate item rejection
- valid/invalid transitions
- history and audit
- filters/pagination

### Payment

- amount/currency mismatch
- signature/timestamp invalid
- duplicate event
- late success
- one successful payment constraint
- refund path

### Fulfillment

- paid-only creation
- partial picking rejected
- pack precondition
- tracking uniqueness
- ship transaction
- delivery idempotency

### Async

- retry temporary only
- no retry domain error
- notification after commit
- rollback means no notification enqueue

## 7. Security Test Checklist

- Object-level Authorization for every `{id}` endpoint
- Function-level Authorization for admin actions
- Writable field allowlist
- User cannot set organization_id, status, price or totals
- JWT expiry/revocation
- Webhook replay and bad signature
- Rate limits
- Error does not leak stack/SQL/secrets
- Log redaction
- page size and expensive query limits

## 8. Performance Checks

- query count for order detail/list
- select_related/prefetch_related verification
- 1000 seeded orders list response
- concurrent order creation smoke test
- expired reservation batch uses indexed query and bounded batch size

## 9. CI Test Stages

- Fast unit tests on every push
- Full PostgreSQL integration tests on PR
- Concurrency tests on PR/main
- Docker smoke test before merge
- Optional scheduled security/dependency scan

## 10. Release Test Report

Release report باید شامل باشد:

- commit SHA
- environment versions
- passed/failed/skipped counts
- coverage summary
- critical acceptance result
- known issues
- migration smoke result
- restore test result
