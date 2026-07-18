# ADR-002 — رزرو موجودی با Transaction و Row Lock

- Status: Accepted
- Date: 2026-07-16

## Context

دو سفارش هم‌زمان ممکن است Available یکسان را بخوانند و Overselling ایجاد کنند. بررسی ساده `if available >= quantity` بدون Lock کافی نیست.

## Decision

- InventoryItemهای لازم داخل `transaction.atomic()` با `select_for_update()` قفل می‌شوند.
- Lockها با ترتیب ثابت ID گرفته می‌شوند.
- کفایت تمام Itemها پیش از هر Update بررسی می‌شود.
- رزرو همه Itemها All-or-nothing است.
- DB Check Constraint از `reserved > on_hand` جلوگیری می‌کند.

## Alternatives

- Optimistic locking با version: قابل استفاده، اما Retry logic پیچیده‌تر برای MVP.
- Redis distributed lock: Redis منبع حقیقت نیست و خطر ناسازگاری با DB دارد.
- Serializable isolation برای همه عملیات: هزینه و retry بیشتر از نیاز MVP.

## Consequences

- correctness قوی در PostgreSQL
- احتمال انتظار Lock؛ Transaction باید کوتاه باشد
- نیاز به concurrency test واقعی
