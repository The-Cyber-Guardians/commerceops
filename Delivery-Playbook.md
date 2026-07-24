# TravelOps — Delivery Playbook

> Execution, collaboration, quality, and project delivery plan

## 1. Team Assumptions

* Three junior developers
* Goal: Build a complete and presentable project
* Priority: Core business logic and the main user flow, not the number of features

## 2. Work Management

Manage tasks through GitHub Issues and Project Board.

```text
Backlog → Ready → In Progress → In Review → Done
```

Recommended fields:

* Priority
* Owner
* Reviewer
* Area
* Risk

## 3. Iterations

### Iteration 0 — Foundation

* Backend / Frontend
* Docker and PostgreSQL
* Custom User
* pytest
* CI

**Exit:** Project runs successfully, migrations work, and CI passes.

### Iteration 1 — Auth & Organizations

* JWT
* Organization
* Membership
* Roles and Permissions
* Tenant Isolation

**Exit:** Cross-tenant access tests pass.

### Iteration 2 — Customers & Destinations

* Customer
* Destination
* Search
* Customer History

**Exit:** Customers and Destinations can be managed through the UI.

### Iteration 3 — Tours

* Tour
* Capacity
* Itinerary
* Hotel
* Transportation
* Tour Guide

**Exit:** A complete Tour can be created and published through the UI.

### Iteration 4 — Reservation Workflow

* Reservation
* Status History
* Approval
* Mock Payment
* Customer Timeline

**Exit:** The complete reservation and payment workflow works.

### Iteration 5 — Capacity & Concurrency

* Capacity management
* Transactions
* Overbooking prevention
* Concurrency tests

**Exit:** Concurrent reservations cannot exceed the available capacity.

### Iteration 6 — Travel & Dashboard

* Travel preparation
* In Progress / Completed
* Travel History
* Dashboard

**Exit:** The complete Tour lifecycle from creation to trip completion works.

### Iteration 7 — Release

* Seed Data
* README
* OpenAPI
* Architecture Diagram
* Deployment
* Demo Script

**Exit:** The project is runnable, deployable, and ready for demonstration.

## 4. Scope Cut

If the team falls behind, remove these first:

1. Optional features
2. Export
3. Secondary reports
4. Advanced dashboards
5. Notifications

Do not remove:

* Tenant Isolation
* Reservation Workflow
* Payment Flow
* Capacity Transactions
* Status History
* Main E2E Flow

## 5. Initial Responsibilities

### Member A

* Accounts
* Organizations
* Permissions
* Docker / CI

### Member B

* Tours
* Reservation Workflow
* Payments

### Member C

* Customers
* Destinations
* Travel Resources
* Dashboard

Ownership is flexible. Every Feature must have an Owner and a Reviewer.

## 6. Development Flow

```text
Issue → Branch → Code + Test → PR → Review → CI → Merge
```

The main branch must always remain runnable.

Example branches:

```text
feature/tour-create
feature/reservation-workflow
fix/tenant-access
test/reservation-concurrency
```

## 7. Pull Request Checklist

* Scope is clear
* Issue is linked
* Tests are included
* Migration and API impact are reviewed
* Tenant Isolation and Permissions are checked
* CI passes
* Code review is completed

No PR should be merged without review.

## 8. Definition of Done

A Feature is Done when:

* Acceptance Criteria are met.
* Happy Path and Failure Path are tested.
* Permissions and Tenant Isolation are verified.
* Required migrations and APIs are updated.
* Review and CI are successful.

## 9. AI Usage

AI may be used for:

* Explaining concepts
* Suggesting edge cases
* Debugging
* Initial code review
* Generating tests and demo data

Every team member must understand, explain, test, and modify the code they merge.

## 10. Final Demo

Main scenario:

```text
Login
↓
Create Organization
↓
Create Customer
↓
Create Tour
↓
Add Itinerary / Hotel / Transportation
↓
Assign Tour Guide
↓
Publish Tour
↓
Customer Books Tour
↓
Approve Reservation
↓
Payment
↓
Confirm Reservation
↓
Start Travel
↓
Complete Travel
↓
Travel History
```

The demo must also show **Cross-Tenant Access Prevention** and **Overbooking Prevention**.

## 11. Final Documentation

* README
* OpenAPI / Swagger
* Architecture Diagram
* Setup Guide
* Demo Accounts
* Tests and CI
* Technical Decisions
* Limitations

## 12. Presentation Readiness

Every team member should be able to explain, with examples:

* Multi-Tenancy and Tenant Isolation
* Reservation Workflow
* Transactions and Overbooking Prevention
* Permissions and Security
* Testing and Failure Paths

Avoid unsupported claims such as "Production-ready" or "Highly scalable."