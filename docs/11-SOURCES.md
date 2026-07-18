# 11 — منابع و استانداردهای مرجع

این بسته یک پیاده‌سازی رسمی یا گواهی‌شده از استانداردها نیست؛ ساختار Requirements آن از اصول Requirements Engineering و منابع رسمی زیر الهام گرفته است.

## Requirements and Documentation

1. ISO/IEC/IEEE 29148:2018 — Requirements Engineering  
   https://www.iso.org/standard/72089.html
2. NASA Software Requirements Specification guidance  
   https://swehb.nasa.gov/display/SWEHBVB/SRS+-+Software+Requirements+Specification
3. IIBA — Requirements documenting and BABOK references  
   https://www.iiba.org/business-analysis-blogs/requirements-documenting--the-foundation-of-a-projects-success/

## Django and DRF

4. Django 5.2 Database Transactions  
   https://docs.djangoproject.com/en/5.2/topics/db/transactions/
5. Django QuerySet `select_for_update()`  
   https://docs.djangoproject.com/en/5.2/ref/models/querysets/#select-for-update
6. Django REST Framework Authentication  
   https://www.django-rest-framework.org/api-guide/authentication/
7. Django REST Framework Permissions  
   https://www.django-rest-framework.org/api-guide/permissions/
8. Django REST Framework Exceptions  
   https://www.django-rest-framework.org/api-guide/exceptions/
9. Django REST Framework Testing  
   https://www.django-rest-framework.org/api-guide/testing/

## PostgreSQL and Concurrency

10. PostgreSQL Transaction Isolation  
    https://www.postgresql.org/docs/current/transaction-iso.html
11. PostgreSQL Explicit Locking  
    https://www.postgresql.org/docs/current/explicit-locking.html
12. PostgreSQL Unique Index Checks  
    https://www.postgresql.org/docs/current/index-unique-checks.html

## Async Processing

13. Celery Tasks and Retry  
    https://docs.celeryq.dev/en/stable/userguide/tasks.html
14. Celery Calling Tasks and Retry Policy  
    https://docs.celeryq.dev/en/stable/userguide/calling.html

## API and Security

15. OpenAPI Specification  
    https://spec.openapis.org/oas/latest.html
16. OWASP API Security Top 10 — 2023  
    https://owasp.org/API-Security/editions/2023/en/0x11-t10/
17. OWASP Broken Object Level Authorization  
    https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
18. OWASP Broken Authentication  
    https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/

## Development Workflow

19. GitHub Flow  
    https://docs.github.com/en/get-started/using-github/github-flow

## Version Baseline

- Django 5.2 is an LTS series with extended support through April 2028 at the time this document was prepared.
- PostgreSQL 17 is a supported series.
- Celery 5.6 is the stable documentation series used for this baseline.

Versions must still be locked and reviewed in the project dependency file.
