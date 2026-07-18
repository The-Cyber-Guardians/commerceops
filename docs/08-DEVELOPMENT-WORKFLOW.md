# 08 — Workflow توسعه CommerceOps

## 1. مدل کار

از GitHub Flow استفاده می‌شود:

`Issue → Branch → Implementation + Tests → Pull Request → Review → CI → Merge`

Branch اصلی همیشه باید قابل اجرا باشد.

## 2. Issue Types

- `epic`
- `feature`
- `bug`
- `tech-debt`
- `documentation`
- `security`
- `spike`

## 3. Issue Template

```md
## Goal

## Requirement IDs
FR-...

## Business Context

## Acceptance Criteria
- [ ] ...

## API/Data Changes

## Permission/Security Considerations

## Test Scenarios

## Out of Scope
```

هیچ Feature بدون Requirement ID یا Acceptance Criteria وارد توسعه نمی‌شود.

## 4. Branch Naming

```text
feature/ORD-12-create-order
bugfix/INV-31-reservation-race
chore/CI-04-coverage-gate
docs/BRD-02-scope-update
```

## 5. Commit Convention

Conventional Commits پیشنهادی:

```text
feat(orders): add idempotent order creation
fix(inventory): lock rows in stable order
test(payments): cover duplicate webhook
docs(srd): document on-commit side effects
```

Commit باید کوچک، معنی‌دار و قابل Review باشد.

## 6. Pull Request Template

```md
## Summary

## Requirement IDs

## Changes

## Business Rules Affected

## Database/Migration
- [ ] No migration
- [ ] Schema migration
- [ ] Data migration

## API Compatibility

## Security and Tenant Isolation

## Tests

## Evidence

## Checklist
- [ ] Acceptance criteria met
- [ ] Tests added/updated
- [ ] Docs updated
- [ ] No secrets/log leakage
- [ ] Migration checked
- [ ] OpenAPI updated
```

## 7. Definition of Ready

- Issue هدف مشخص دارد.
- Requirement ID معتبر دارد.
- Acceptance Criteria قابل تست است.
- Scope و Out of Scope مشخص است.
- Permission، Error و Data impact روشن است.
- وابستگی Blocker ندارد.

## 8. Definition of Done

- Code مطابق معماری است.
- Unit/Integration/API tests متناسب وجود دارند.
- Tenant isolation و Permission تست شده‌اند.
- Migration قابل اجرا است.
- API Schema و docs هماهنگ‌اند.
- Log داده حساس ندارد.
- CI سبز است.
- Review comments حل شده‌اند.
- Traceability Matrix در تغییر Requirement به‌روز شده است.

## 9. Code Review Checklist

### Correctness

- Business Rule در Service/Domain است؟
- Failure path و rollback بررسی شده؟
- Race condition محتمل است؟
- Idempotency لازم است؟

### Data

- Query tenant-scoped است؟
- Constraint/Index لازم اضافه شده؟
- N+1 ایجاد نشده؟
- Money از Decimal استفاده می‌کند؟

### Security

- AuthN و AuthZ هر دو بررسی شده؟
- Object-level permission وجود دارد؟
- Writable fields محدودند؟
- Secret/PII در log نیست؟

### Test

- Acceptance criteria پوشش داده شده؟
- تست منفی وجود دارد؟
- تست transaction/concurrency لازم است؟

## 10. Migration Workflow

1. Model change
2. Generate migration
3. Review SQL با `sqlmigrate`
4. Run from empty DB
5. Run over seeded previous schema
6. Add data migration if needed
7. Update Data Model doc

Migrationهای مستقل هم‌نام باید قبل از Merge رفع شوند.

## 11. API Change Workflow

- Contract ابتدا در FRD/API doc تغییر می‌کند.
- Schema/Serializer/View/Service پیاده‌سازی می‌شود.
- OpenAPI snapshot یا validation به‌روز می‌شود.
- Backward compatibility بررسی می‌شود.

## 12. Local Development

دستورهای هدف:

```bash
cp .env.example .env
docker compose up -d postgres redis
docker compose run --rm api python manage.py migrate
docker compose run --rm api python manage.py seed_demo
docker compose up api celery-worker celery-beat
```

## 13. Quality Commands

```bash
ruff format --check .
ruff check .
mypy src
pytest
python src/manage.py makemigrations --check --dry-run
python src/manage.py check --deploy
```

## 14. Environment Strategy

- local
- test
- staging/demo
- production-like configuration template

Test به سرویس خارجی واقعی متصل نمی‌شود.

## 15. ADR Workflow

ADR جدید زمانی لازم است که تصمیم:

- روی چند ماژول اثر دارد؛
- برگشت آن پرهزینه است؛
- Trade-off مهم دارد؛
- معماری، Data Integrity یا Security را تغییر می‌دهد.

ساختار ADR:

`Context → Decision → Alternatives → Consequences → Status`.

## 16. Bug Workflow

Bug report باید Expected، Actual، Steps، Environment، Trace ID و Severity داشته باشد.

Severity:

- Critical: data corruption/security/overselling
- High: workflow اصلی متوقف
- Medium: feature با workaround
- Low: cosmetic/docs

Critical/High قبل از Feature جدید Release حل می‌شوند.
