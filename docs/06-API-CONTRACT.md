# 06 — قرارداد API نسخه 1 CommerceOps

## 1. اصول

- Base path: `/api/v1`
- Content-Type: `application/json`
- Authentication: `Authorization: Bearer <access-token>`
- Tenant context: `X-Organization-ID: <uuid>`
- Trace: Client MAY send `X-Trace-ID`; Server always returns it.
- Create Order requires `Idempotency-Key`.
- Timestamps are UTC ISO 8601.
- Public IDs are UUID.

OpenAPI generated از Code قرارداد ماشینی نهایی است؛ این فایل قرارداد انسانی و قواعد دامنه را ثبت می‌کند. هر اختلاف باید پیش از Merge رفع شود.

## 2. پاسخ موفق استاندارد

برای Resource منفرد:

```json
{
  "data": {
    "id": "6e28d71f-b78c-4bea-8a64-988f7a77c4ca",
    "type": "order",
    "attributes": {}
  },
  "meta": {"trace_id": "01J..."}
}
```

برای List:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 25,
    "total": 0,
    "pages": 0
  },
  "meta": {"trace_id": "01J..."}
}
```

## 3. پاسخ خطا

```json
{
  "error": {
    "code": "INVALID_ORDER_TRANSITION",
    "message": "Order cannot transition from PROCESSING to CANCELLED.",
    "details": {
      "current_status": "PROCESSING",
      "requested_action": "cancel"
    },
    "trace_id": "01J..."
  }
}
```

## 4. Status Code Policy

| Code | کاربرد |
|---:|---|
| 200 | Read/Action موفق یا replay موفق Idempotent |
| 201 | Resource ایجاد شد |
| 202 | Task غیرهمزمان پذیرفته شد |
| 204 | حذف/غیرفعال‌سازی بدون Body |
| 400 | Validation یا malformed request |
| 401 | Authentication نامعتبر/غایب |
| 403 | Authenticated ولی بدون Permission عمومی |
| 404 | Resource وجود ندارد یا خارج Tenant است |
| 409 | Conflict وضعیت، موجودی یا Idempotency |
| 422 | فقط در صورت تصمیم یکپارچه؛ MVP از 400 برای Validation استفاده می‌کند |
| 429 | Rate limited |
| 500 | خطای غیرمنتظره با Trace ID |

## 5. Authentication Endpoints

| ID | Method | Path | Auth | شرح |
|---|---|---|---|---|
| API-AUTH-001 | POST | `/auth/login` | No | دریافت Token |
| API-AUTH-002 | POST | `/auth/refresh` | Refresh | چرخش Refresh |
| API-AUTH-003 | POST | `/auth/logout` | Yes | Blacklist Refresh |
| API-AUTH-004 | GET | `/users/me` | Yes | Profile/Memberships |
| API-AUTH-005 | POST | `/users/me/change-password` | Yes | تغییر رمز |

### Login Request

```json
{"email": "user@example.com", "password": "secret"}
```

### Login Response

```json
{
  "data": {
    "access": "<jwt>",
    "refresh": "<jwt>",
    "expires_in": 900
  },
  "meta": {"trace_id": "01J..."}
}
```

## 6. Organization و Membership

| ID | Method | Path | Permission |
|---|---|---|---|
| API-ORG-001 | GET | `/organizations` | authenticated memberships |
| API-ORG-002 | POST | `/organizations` | platform admin |
| API-ORG-003 | GET | `/organizations/{id}` | membership |
| API-ORG-004 | PATCH | `/organizations/{id}` | org admin |
| API-ORG-005 | GET | `/memberships` | org admin |
| API-ORG-006 | POST | `/memberships` | org admin |
| API-ORG-007 | PATCH | `/memberships/{id}` | org admin |
| API-ORG-008 | DELETE | `/memberships/{id}` | org admin; deactivate |

## 7. Customer و Address

| ID | Method | Path | Permission |
|---|---|---|---|
| API-CUS-001 | GET | `/customers` | customer.read |
| API-CUS-002 | POST | `/customers` | customer.create |
| API-CUS-003 | GET | `/customers/{id}` | customer.read/resource |
| API-CUS-004 | PATCH | `/customers/{id}` | customer.update |
| API-CUS-005 | GET | `/customers/{id}/addresses` | customer.read |
| API-CUS-006 | POST | `/customers/{id}/addresses` | customer.update |
| API-CUS-007 | PATCH | `/addresses/{id}` | customer.update |
| API-CUS-008 | DELETE | `/addresses/{id}` | customer.update; soft deactivate |

## 8. Catalog

| ID | Method | Path | Permission |
|---|---|---|---|
| API-CAT-001 | GET | `/products` | catalog.read |
| API-CAT-002 | POST | `/products` | catalog.manage |
| API-CAT-003 | GET | `/products/{id}` | catalog.read |
| API-CAT-004 | PATCH | `/products/{id}` | catalog.manage |
| API-CAT-005 | POST | `/products/{id}/variants` | catalog.manage |
| API-CAT-006 | GET | `/variants` | catalog.read |
| API-CAT-007 | GET | `/variants/{id}` | catalog.read |
| API-CAT-008 | PATCH | `/variants/{id}` | catalog.manage |

### Create Variant

```json
{
  "sku": "TSHIRT-BLK-M",
  "name": "Black / Medium",
  "unit_price": "1200.00",
  "currency": "EUR",
  "is_active": true
}
```

## 9. Warehouse و Inventory

| ID | Method | Path | Permission |
|---|---|---|---|
| API-INV-001 | GET | `/warehouses` | warehouse.read |
| API-INV-002 | POST | `/warehouses` | warehouse.manage |
| API-INV-003 | PATCH | `/warehouses/{id}` | warehouse.manage |
| API-INV-004 | GET | `/inventory` | inventory.read + scope |
| API-INV-005 | GET | `/inventory/{id}` | inventory.read + scope |
| API-INV-006 | POST | `/inventory/receipts` | inventory.adjust + scope |
| API-INV-007 | POST | `/inventory/adjustments` | inventory.adjust + scope |
| API-INV-008 | GET | `/inventory/movements` | inventory.read + scope |
| API-INV-009 | GET | `/inventory/low-stock` | inventory.read + scope |

### Stock Receipt Request

```json
{
  "warehouse_id": "uuid",
  "variant_id": "uuid",
  "quantity": 50,
  "reference": "PO-DEMO-001",
  "reason": "Initial stock"
}
```

### Inventory Response

```json
{
  "data": {
    "id": "uuid",
    "warehouse_id": "uuid",
    "variant_id": "uuid",
    "on_hand": 50,
    "reserved": 3,
    "available": 47,
    "low_stock_threshold": 5
  },
  "meta": {"trace_id": "01J..."}
}
```

## 10. Orders

| ID | Method | Path | Permission |
|---|---|---|---|
| API-ORD-001 | GET | `/orders` | order.read/resource |
| API-ORD-002 | POST | `/orders` | order.create + Idempotency-Key |
| API-ORD-003 | GET | `/orders/{id}` | order.read/resource |
| API-ORD-004 | POST | `/orders/{id}/cancel` | order.cancel/resource |
| API-ORD-005 | GET | `/orders/{id}/history` | order.read/resource |
| API-ORD-006 | GET | `/orders/summary` | reporting.read |
| API-ORD-007 | GET | `/orders/stalled` | reporting.read |

### Create Order Request

Headers:

```text
Authorization: Bearer ...
X-Organization-ID: uuid
Idempotency-Key: 3a8d70b7-...
```

Body:

```json
{
  "customer_id": "uuid",
  "warehouse_id": "uuid",
  "shipping_address_id": "uuid",
  "items": [
    {"variant_id": "uuid", "quantity": 2},
    {"variant_id": "uuid", "quantity": 1}
  ]
}
```

### Create Order Response — 201

```json
{
  "data": {
    "id": "uuid",
    "order_number": "ORD-2026-000001",
    "status": "PENDING_PAYMENT",
    "currency": "EUR",
    "subtotal": "3600.00",
    "total": "3600.00",
    "reservation": {
      "status": "ACTIVE",
      "expires_at": "2026-07-16T13:15:00Z"
    },
    "items": [
      {
        "sku": "TSHIRT-BLK-M",
        "name": "Black / Medium",
        "unit_price": "1200.00",
        "quantity": 2,
        "line_total": "2400.00"
      }
    ]
  },
  "meta": {"trace_id": "01J..."}
}
```

### مهم‌ترین خطاها

| Error Code | HTTP | شرح |
|---|---:|---|
| `INSUFFICIENT_INVENTORY` | 409 | Available کافی نیست |
| `DUPLICATE_ORDER_ITEM` | 400 | Variant تکراری در Payload |
| `IDEMPOTENCY_KEY_REQUIRED` | 400 | Header غایب |
| `IDEMPOTENCY_KEY_REUSED` | 409 | Key با Payload متفاوت |
| `INVALID_ORDER_TRANSITION` | 409 | لغو یا Action نامجاز |
| `RESERVATION_EXPIRED` | 409 | رزرو منقضی |

## 11. Payment Simulator

| ID | Method | Path | Auth |
|---|---|---|---|
| API-PAY-001 | POST | `/orders/{id}/payments` | finance/order action |
| API-PAY-002 | GET | `/orders/{id}/payments` | payment.read |
| API-PAY-003 | POST | `/webhooks/payment-simulator` | Signature, no JWT |
| API-PAY-004 | GET | `/payments/{id}` | payment.read |

### Initiate Payment

```json
{"return_url": "https://example.test/payment-result"}
```

Response includes simulator reference and demo action URL/token.

### Webhook Headers

```text
X-Provider-Event-ID: evt_123
X-Provider-Timestamp: 1784210400
X-Provider-Signature: sha256=...
```

### Webhook Body

```json
{
  "payment_reference": "pay_123",
  "event_type": "payment.succeeded",
  "amount": "3600.00",
  "currency": "EUR"
}
```

Webhook replay با Event ID یکسان باید 200 و نتیجه قبلی برگرداند.

## 12. Fulfillment و Shipment

| ID | Method | Path | Permission |
|---|---|---|---|
| API-FUL-001 | GET | `/fulfillments` | fulfillment.read + scope |
| API-FUL-002 | GET | `/fulfillments/{id}` | fulfillment.read + scope |
| API-FUL-003 | POST | `/orders/{id}/fulfillment` | fulfillment.create |
| API-FUL-004 | POST | `/fulfillments/{id}/start-picking` | fulfillment.update + scope |
| API-FUL-005 | POST | `/fulfillments/{id}/confirm-picking` | fulfillment.update + scope |
| API-FUL-006 | POST | `/fulfillments/{id}/pack` | fulfillment.update + scope |
| API-FUL-007 | POST | `/fulfillments/{id}/ship` | shipment.create + scope |
| API-FUL-008 | POST | `/shipments/{id}/mark-delivered` | shipment.update + scope |
| API-FUL-009 | GET | `/shipments/{id}` | shipment.read/resource |

### Confirm Picking Request

```json
{
  "items": [
    {"order_item_id": "uuid", "quantity_picked": 2}
  ]
}
```

### Ship Request

```json
{
  "carrier_name": "Demo Carrier",
  "tracking_code": "TRK-000001"
}
```

Ship در یک Transaction:

- Fulfillment → SHIPPED
- Shipment → IN_TRANSIT
- Reservation `CONFIRMED` → `CONSUMED`
- Inventory on_hand/reserved کاهش
- Order → SHIPPED
- Stock Movement ثبت

## 13. Audit

| ID | Method | Path | Permission |
|---|---|---|---|
| API-AUD-001 | GET | `/audit-logs` | audit.read |
| API-AUD-002 | GET | `/audit-logs/{id}` | audit.read/resource |

Create/Update/Delete عمومی برای Audit وجود ندارد.

## 14. Health and Schema

| ID | Method | Path | شرح |
|---|---|---|---|
| API-SYS-001 | GET | `/health/live` | Process alive |
| API-SYS-002 | GET | `/health/ready` | DB/Redis dependencies ready |
| API-SYS-003 | GET | `/schema` | OpenAPI JSON/YAML |
| API-SYS-004 | GET | `/docs` | Swagger/ReDoc در محیط مجاز |

## 15. Filter و Pagination

- `page`, `page_size`
- حداکثر page_size = 100
- Sort فقط Allowlist
- Date filters با `created_from`, `created_to`
- Search نباید Raw SQL یا field arbitrary بپذیرد

## 16. Backward Compatibility

- فیلد جدید Optional در v1 مجاز است.
- حذف یا تغییر معنی فیلد در v1 ممنوع است.
- Enum جدید فقط با بررسی Client و Documentation اضافه شود.
- Endpoint ناسازگار نیازمند `/api/v2` یا دوره Deprecation است.
