# 10 — ماتریس ردیابی نیازمندی‌ها

## 1. هدف

این ماتریس نشان می‌دهد هر نیاز کسب‌وکار با کدام نیاز عملکردی، طراحی، API و تست اثبات می‌شود.

## 2. Business-to-Implementation Traceability

| Business Req | Functional Requirements | Design/Data | API | Tests |
|---|---|---|---|---|
| BR-001 جلوگیری از Overselling | FR-INV-005, 007, 008, 013 | SRD NFR-DATA-001/002/005; InventoryItem constraints; ADR-002 | API-ORD-002, API-INV-004 | TC-CON-001, TC-INV-001, TC-SHIP-001 |
| BR-002 چرخه یکپارچه سفارش | FR-ORD-001..015, FR-FUL-001..007 | Order/Fulfillment workflows | API-ORD-002, API-FUL-003..008 | TC-E2E-001, TC-CANCEL-001..003 |
| BR-003 تفکیک سازمان‌ها | FR-ORG-002,003,007,008 | SRD Multi-tenancy; organization_id | همه Endpointهای Business | TC-TENANT-001, TC-AUTHZ-001 |
| BR-004 جلوگیری از عملیات تکراری | FR-ORD-002,003; FR-PAY-004,008; FR-ASYNC-004 | IdempotencyRecord, PaymentEvent unique | API-ORD-002, API-PAY-003 | TC-IDEM-001, TC-IDEM-002, TC-PAY-001 |
| BR-005 ممیزی | FR-AUD-001..004; FR-ORD-014 | AuditLog, StatusHistory, StockMovement | API-AUD-001/002 | TC-E2E-001 plus module assertions |
| BR-006 کاهش خطای عملیاتی | FR-ERR; FR-INV; FR-ORD-013 | DB constraints + domain exceptions | error envelope | negative tests per module |
| BR-007 قابلیت ارائه/نگهداری | FR-UI-001/002 | CI, Docker, OpenAPI, modular architecture | API-SYS-001..004 | Docker smoke, schema validation |
| BR-008 اثبات کیفیت | همه FRهای بحرانی | Test Strategy | API contract | Critical Acceptance suite |

## 3. Workflow Requirement Traceability

| Workflow | Rules | Entities | API Actions | Test |
|---|---|---|---|---|
| Create Order | BRULE-INV-001..005; BRULE-ORD-001..007 | Order, OrderItem, Reservation, InventoryItem | POST `/orders` | TC-INV-001, TC-CON-001, TC-IDEM-001 |
| Payment Success | BRULE-PAY-001..005 | Payment, PaymentEvent, Order | POST payment webhook | TC-PAY-001, TC-PAY-002 |
| Reservation Expiry | BRULE-INV-006/007 | Reservation, InventoryItem | Worker | TC-INV-002 |
| Cancel Unpaid | BRULE-ORD-009; BRULE-INV-006 | Order, Reservation | POST order cancel | TC-CANCEL-001 |
| Cancel Paid | BRULE-PAY-006 | Order, Payment, Refund, Reservation | POST order cancel | TC-CANCEL-002/003 |
| Ship | BRULE-FUL-003..006; BRULE-INV-008/009 | Fulfillment, Shipment, Inventory, StockMovement | POST fulfillment ship | TC-SHIP-001, TC-E2E-001 |
| Delivery | BRULE-FUL-007 | Shipment, Order | POST mark-delivered | TC-E2E-001 |

## 4. Security Traceability

| Security Concern | Requirement | Control | Test |
|---|---|---|---|
| Broken object authorization | FR-ORG-007/008, NFR-SEC-002/009 | scoped queryset + ownership check | TC-TENANT-001 |
| Broken authentication | FR-AUTH-001..007, NFR-SEC-001 | JWT rotation, blacklist, throttle | auth test suite |
| Mass assignment | NFR-SEC-003 | serializer writable allowlist | security negative tests |
| Resource consumption | FR-AUTH-007, NFR-SEC-004 | throttle/pagination/max page | API limit tests |
| Unsafe external API consumption | FR-PAY-003/004, NFR-SEC-005 | signature, timestamp, replay ID | webhook security tests |
| Sensitive logging | FR-AUD-004, NFR-SEC-008 | redaction filter | log capture tests |

## 5. Release Gate Traceability

| Release Gate | Evidence |
|---|---|
| No overselling | TC-CON-001 report |
| No cross-tenant leak | TC-TENANT-001 and endpoint matrix |
| No duplicate webhook effect | TC-PAY-001 |
| End-to-end lifecycle | TC-E2E-001 |
| Transaction rollback | TC-SHIP-001 |
| Coverage | CI coverage artifact |
| API docs | generated OpenAPI artifact |
| Reproducible setup | Docker clean-start evidence |
| Data recovery | restore test log |
