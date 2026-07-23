# AfterCare — Delivery Playbook

> برنامه اجرا، همکاری، کیفیت و ارائه

## 1. فرض تیم

- سه عضو تازه‌کار
- کار تقریباً یک روز در میان
- هدف: پایان پروژه در تابستان
- اولویت: مسیر کامل و قابل Demo، نه تعداد Feature

## 2. شیوه برنامه‌ریزی

کارهای اجرایی در GitHub Issues و Project Board نگهداری می‌شوند، نه در اسناد اصلی.

Board پیشنهادی:

```text
Backlog → Ready → In Progress → In Review → Done
```

فیلدهای مفید:

- Priority
- Iteration
- Owner
- Reviewer
- Area: Backend / Frontend / DevOps / Docs
- Risk

## 3. Iterationها

### Iteration 0 — Foundation

- Repository
- Backend و Frontend skeleton
- Docker Compose
- PostgreSQL
- Custom User
- Ruff و pytest
- GitHub Actions

Exit:

- پروژه با راهنمای کوتاه اجرا شود.
- Migration و CI پایه موفق باشند.

### Iteration 1 — Auth and Organizations

- JWT
- Organization
- Membership
- Role
- Organization context
- Tenant-scoped access پایه

Exit:

- User عضو دو Organization بتواند Context را تغییر دهد.
- Cross-tenant test موفق باشد.

### Iteration 2 — Customer and Device

- Customer
- Registered Device
- Serial validation
- Search
- Device detail و history shell

Exit:

- Device از UI ثبت و به Customer صحیح متصل شود.

### Iteration 3 — Repair Intake

- Repair Ticket
- Tracking Number
- Attachment
- Technician assignment
- Status History
- Reception UI

Exit:

- Ticket از UI ایجاد و تخصیص داده شود.

### Iteration 4 — Diagnosis and Estimate

- Diagnosis
- Estimate
- Estimate Items
- Customer approval/rejection
- Customer timeline

Exit:

- Customer فقط Estimate خودش را پاسخ دهد.
- Transition نامعتبر رد شود.

### Iteration 5 — Inventory

- Part
- Inventory
- Receipt
- Adjustment
- Stock Movement
- Low Stock

Exit:

- موجودی منفی در Service و Database رد شود.

### Iteration 6 — Repair Execution

- Start Repair
- Part Usage
- Waiting for Part
- Transaction
- Concurrency Test

Exit:

- مصرف چند Part اتمیک باشد.
- آخرین Part دو بار مصرف نشود.

### Iteration 7 — Quality and Delivery

- Quality Check
- Rework
- Ready for Pickup
- Delivery
- Service History کامل

Exit:

- چرخه کامل از UI اجرا شود.
- E2E اصلی سبز باشد.

### Iteration 8 — Dashboard and Release

- Dashboard
- Query review
- Seed Data
- README
- OpenAPI
- Architecture diagrams
- Deployment
- Demo script

Exit:

- فرد جدید بتواند پروژه را اجرا کند.
- نسخه Deployشده و Demo آماده باشد.

## 4. Scope Cut

اگر عقب افتادید، به این ترتیب حذف کنید:

1. قابلیت‌های اختیاری
2. Export
3. Commentهای پیشرفته
4. Attachmentهای متعدد
5. گزارش‌های فرعی
6. Platform dashboard

حذف نشوند:

- Tenant isolation
- Repair workflow
- Customer approval
- Transactional inventory
- Status history
- E2E اصلی

## 5. تقسیم اولیه

### عضو A

- Accounts
- Organizations
- Permission
- Docker و CI

### عضو B

- Repair Workflow
- Diagnosis
- Estimate
- Quality Check

### عضو C

- Customer
- Device
- Inventory
- Dashboard

مالکیت دائمی نیست. هر Feature یک Owner و Reviewer از بخش دیگر دارد.

## 6. جریان توسعه

```text
Issue → Branch → Code + Test → Pull Request → Review → CI → Merge
```

Branch اصلی همیشه قابل اجرا بماند.

### Branch

```text
feature/repair-create-ticket
feature/inventory-consume-parts
fix/tenant-device-access
test/repair-concurrency
docs/update-workflow
```

### Commit

```text
feat(repairs): add technician assignment
fix(inventory): prevent negative stock
test(tenancy): cover cross-organization access
```

## 7. قالب Issue

```md
## Goal

## User or Business Value

## Acceptance Criteria

## API / Data Impact

## Permission and Tenant Risks

## Test Scenarios

## Out of Scope
```

## 8. Pull Request Checklist

- Scope کوچک و قابل Review
- لینک Issue
- توضیح تغییر
- Migration impact
- API impact
- Tenant و Security impact
- Tests
- Screenshot برای UI
- CI سبز

هیچ PR بدون Review Merge نشود.

## 9. Definition of Ready

Issue آماده توسعه است اگر:

- هدف روشن دارد.
- Acceptance Criteria قابل تست دارد.
- Scope محدود است.
- Dependency معلوم است.
- Permission و Data impact مشخص است.

## 10. Definition of Done

Feature Done است اگر:

- Acceptance Criteria پاس شده‌اند.
- Happy و Failure path تست شده‌اند.
- Permission و Tenant isolation بررسی شده‌اند.
- Migration و API schema به‌روزند.
- Review انجام شده است.
- CI سبز است.
- رفتار جدید در سند اصلی اصلاح شده است.

## 11. ریتم جلسه‌ها

هر جلسه:

1. Blockerها
2. هدف کوچک جلسه
3. Pairing در بخش سخت
4. Push و ثبت وضعیت
5. Demo کوتاه تغییر

هر سه یا چهار جلسه:

- Demo داخلی
- مرور Scope
- مرور تست‌ها
- حذف کارهای کم‌ارزش
- تصمیم Iteration بعدی

## 12. استفاده از AI

مجاز:

- توضیح مفهوم
- پیشنهاد Edge Case
- Review اولیه
- تولید Fixture و Demo Data
- کمک به Debug

شرط Merge:

- عضو مسئول کد را توضیح دهد.
- Failure mode را بداند.
- تست مناسب بنویسد.
- بتواند بدون بازتولید کامل کد آن را تغییر دهد.

هفته‌ای حداقل یک مسئله بدون گرفتن راه‌حل مستقیم از AI حل شود.

## 13. مستندات لازم برای ارائه

فقط این موارد نگهداری شوند:

- سه سند همین بسته
- OpenAPI تولیدشده
- System Context Diagram
- Container Diagram
- README نهایی
- GitHub Issues و Project Board

نمودار بیشتر فقط زمانی ساخته شود که واقعاً به فهم بخش دشواری کمک کند.

## 14. Demo

سناریوی پیشنهادی:

1. ورود Receptionist
2. ثبت Customer و Device
3. ساخت Ticket
4. تخصیص Technician
5. Diagnosis و Estimate
6. تأیید Customer
7. مصرف Part و کاهش موجودی
8. Quality Check
9. Delivery
10. Timeline و Dashboard
11. نمایش جلوگیری از Cross-tenant access

صفحات مهم برای Polish:

- Login / Organization selector
- Dashboard
- Repair list/detail
- Reception form
- Technician queue
- Customer portal
- Inventory
- Device service history

## 15. README نهایی

README در پایان پروژه نوشته و شامل این موارد شود:

- معرفی
- Problem
- Features
- Screenshots
- Tech stack
- Architecture
- Setup
- Demo accounts
- Swagger
- Tests و CI
- تصمیم‌های فنی
- Limitations
- Contributions

## 16. آمادگی مصاحبه

هر عضو باید بتواند درباره این چهار موضوع با مثال صحبت کند:

- Tenant isolation
- Workflow و جلوگیری از Transition نامعتبر
- Transaction و هم‌زمانی Inventory
- Test و Failure Path

از ادعاهای بدون مدرک مانند Production-ready، Microservices، Clean Architecture کامل یا مقیاس‌پذیری بالا پرهیز شود.
