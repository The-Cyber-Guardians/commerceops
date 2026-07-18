# 13 — گزارش کنترل سازگاری اسناد

> نتیجه: **PASS**  
> تاریخ: 2026-07-16

## کنترل‌های خودکار

- [x] شناسه تعریف‌شده تکراری وجود ندارد.
- [x] همه ارجاع‌های Requirement دارای تعریف هستند.
- [x] همه لینک‌های داخلی معتبرند.
- [x] همه Code/Mermaid fenceها بسته هستند.
- [x] TODO/FIXME حل‌نشده وجود ندارد.
- [x] وضعیت‌های Order/Reservation/Payment بین Data Model و Workflow همسان‌اند.

## شمارش نیازمندی‌ها

| نوع | تعداد |
|---|---:|
| BR | 8 |
| FR | 90 |
| NFR | 36 |
| BRULE | 34 |
| API | 64 |
| TC | 13 |

## کنترل‌های دستی انجام‌شده

- Scope داخل/خارج MVP بین README و BRD همسان است.
- رزرو پس از پرداخت از `ACTIVE` به `CONFIRMED` می‌رود و دیگر Expire نمی‌شود.
- Cancel پرداخت‌شده فقط پیش از Picking و با Refund شبیه‌سازی‌شده مجاز است.
- Order status مستقیماً از PATCH عمومی قابل تغییر نیست.
- تمام داده‌های Business با Organization محدود می‌شوند.
- Side effectهای غیرتراکنشی پس از Commit آغاز می‌شوند.

## موارد Deferred صریح

- درگاه واقعی، Promotion/Tax Engine، Return کامل، Partial Shipment و Microservices در Release 1.0 نیستند.
- هر Scope cut جدید نیازمند Change Request و افزایش نسخه اسناد است.
