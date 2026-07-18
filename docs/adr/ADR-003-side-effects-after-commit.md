# ADR-003 — اجرای Side Effect پس از Commit

- Status: Accepted
- Date: 2026-07-16

## Context

ارسال Notification یا Queue task داخل Transaction ممکن است پیش از Commit انجام شود یا Transaction را طولانی کند. در Rollback نباید پیام موفقیت ارسال شود.

## Decision

Side Effectهای غیرتراکنشی با `transaction.on_commit()` آغاز می‌شوند. Taskها Idempotent هستند. اگر reliability بیشتر لازم شد، Outbox Table کامل در Release بعدی اضافه می‌شود.

## Alternatives

- Publish داخل Transaction: خطر پیام برای داده rollback‌شده و Transaction طولانی.
- Distributed transaction: خارج از نیاز و Scope.
- Full transactional outbox از ابتدا: قوی‌تر، ولی حجم پیاده‌سازی بیشتر؛ برای MVP Deferred.

## Consequences

- عدم ارسال پیام در rollback
- شکست Queue بعد از Commit ممکن است نیازمند recovery باشد
- Monitoring و Retry ضروری است
