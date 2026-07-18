# 09 — برنامه تحویل هشت‌هفته‌ای CommerceOps

## 1. اصل برنامه

هر هفته باید Increment قابل اجرا و تست‌شده تولید شود. تست، مستندات و Migration بخشی از همان Feature هستند و به هفته آخر منتقل نمی‌شوند.

## هفته 1 — Foundation و Requirements Baseline

### خروجی

- Repository و CI skeleton
- Docker Compose برای API/PostgreSQL/Redis
- Django project و module boundaries
- Custom User
- Organization و Membership
- Requirement IDs و ADR-001

### Exit Criteria

- محیط از صفر بالا می‌آید.
- CI lint/test پایه سبز است.
- User/Organization migration اجرا می‌شود.
- Cross-tenant test اولیه وجود دارد.

## هفته 2 — Authentication و Authorization

### خروجی

- JWT login/refresh/logout
- Organization context
- Role/Permission seed
- scoped repositories/querysets
- error envelope و trace ID

### Exit Criteria

- Permission matrix پایه تست شده.
- Org A به Org B دسترسی ندارد.
- Auth throttle و token rotation کار می‌کند.

## هفته 3 — Catalog و Inventory Ledger

### خروجی

- Product/Variant/Warehouse/InventoryItem
- Receipt و Adjustment
- Stock Movement
- Database constraints/indexes

### Exit Criteria

- SKU uniqueness و Money validation تست شده.
- موجودی منفی توسط Service و DB رد می‌شود.
- Inventory ledger قابل Query است.

## هفته 4 — Order Creation و Reservation Concurrency

### خروجی

- Customer/Address
- Order/OrderItem snapshot
- Transactional reservation
- Idempotent create order
- Reservation expiry model
- ADR-002

### Exit Criteria

- Multi-item all-or-nothing test سبز.
- Concurrent overselling test سبز.
- Duplicate Idempotency Key رفتار مشخص دارد.

## هفته 5 — Payment Simulator و Async Expiry

### خروجی

- Payment initiation
- signed simulated webhook
- duplicate event protection
- success/failure/late success
- Celery expiry task
- notification after commit
- ADR-003

### Exit Criteria

- Webhook replay Side Effect تکراری ندارد.
- Expiry Task idempotent است.
- rollback باعث enqueue notification نمی‌شود.

## هفته 6 — Fulfillment و Shipment

### خروجی

- Fulfillment lifecycle
- Picking/Packing
- Shipment
- atomic inventory consume
- delivery action

### Exit Criteria

- End-to-End order to shipment سبز.
- Ship transaction در خطای میانی rollback می‌شود.
- Warehouse scope تست شده.

## هفته 7 — Hardening و Reporting

### خروجی

- Audit log کامل
- order summary/low-stock/stalled orders
- query optimization
- security negative tests
- dependency/secret scan
- OpenAPI validation

### Exit Criteria

- Coverage gate برقرار.
- Queryهای بحرانی N+1 ندارند.
- OWASP-focused test checklist تکمیل شده.

## هفته 8 — Release و Portfolio Packaging

### خروجی

- Demo seed data
- README اجرای پروژه
- Architecture/ERD diagrams
- API examples
- Test report
- backup/restore script and test
- release tag `v1.0.0`

### Exit Criteria

- Docker clean-start موفق.
- تمام Critical Acceptance Tests سبز.
- Migration از صفر موفق.
- Demo سناریوی کامل اجرا می‌شود.
- Known Issues مستند است.

## 2. Backlog پس از Release

اولویت پیشنهادی:

1. Return/Refund workflow کامل
2. Partial shipment
3. Promotion/discount engine
4. Real payment adapter sandbox
5. Shipping provider adapter
6. Outbox table کامل
7. PostgreSQL RLS hardening
8. UI مستقل

## 3. Scope Cut Order

اگر برنامه عقب افتاد، فقط با Change Request و افزایش نسخه اسناد می‌توان به این ترتیب موردی را Deferred کرد. تا پیش از تصویب تغییر، همه نیازهای Approved همچنان اجباری‌اند:

1. گزارش Stalled Orders
2. Customer portal endpointهای غیرضروری
3. UI اضافی خارج Django Admin
4. Metrics پیشرفته
5. Refund شبیه‌سازی‌شده PAID cancellation

موارد زیر قابل حذف نیستند:

- Tenant isolation
- Inventory transaction/concurrency
- Idempotency
- Payment webhook duplicate protection
- Workflow validity
- Tests و Docker/CI
