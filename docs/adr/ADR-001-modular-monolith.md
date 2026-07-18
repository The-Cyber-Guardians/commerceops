# ADR-001 — انتخاب Modular Monolith

- Status: Accepted
- Date: 2026-07-16

## Context

دامنه شامل Identity، Catalog، Inventory، Order، Payment و Fulfillment است و مرزهای منطقی لازم دارد. در عین حال Release 1.0 باید در هشت هفته قابل تحویل، تست و اجرا باشد.

## Decision

یک Modular Monolith با یک Deployment و یک PostgreSQL ساخته می‌شود. مرز ماژول‌ها در Code و Interfaceها حفظ می‌شود.

## Alternatives

- Microservices: پیچیدگی Deployment، Distributed Transaction، Observability و Contract management زودهنگام.
- Monolith بدون مرز: توسعه سریع اولیه، اما Business Logic پراکنده و تست دشوار.

## Consequences

### Positive

- Transaction اتمیک برای Order/Inventory ساده‌تر است.
- Debug، Migration و Local setup قابل کنترل است.
- مرزهای Domain برای رشد بعدی حفظ می‌شوند.

### Negative

- استقلال Deployment ماژول‌ها وجود ندارد.
- Discipline معماری باید با Review حفظ شود.
