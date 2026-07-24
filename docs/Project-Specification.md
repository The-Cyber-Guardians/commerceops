# TravelOps — Project Specification

## 1. Project Overview

TravelOps is a multi-tenant platform for managing travel agencies and the complete tour booking and travel lifecycle.

Each agency (`Organization`) has isolated data and can only access its own information.

### Core Features

* Agency and user management
* Customer management
* Domestic and international tour management
* Destination and itinerary management
* Tour capacity management
* Tour guide management
* Hotel and transportation management
* Reservation and payment management
* Customer travel history
* Reservation workflow from request to trip completion

---

## 2. Project Goal

Build a real-world, presentable backend project that demonstrates:

* Authentication and Authorization
* Role-Based Access Control
* REST API
* Multi-Tenancy and Tenant Isolation
* State Machine and Workflow
* Database Transactions
* Tour capacity management and overbooking prevention
* Payment management
* Testing and CI/CD
* Docker and Deployment

The project should go beyond simple CRUD and include real business logic.

---

## 3. Roles

### Platform Admin

Manages organizations across the platform.

### Organization Admin

Manages users, tours, customers, reservations, and agency resources.

### Travel Agent / Staff

Manages customers, tours, and reservations.

### Tour Guide

Views assigned tours, passengers, and itineraries.

### Customer

Views tours, creates reservations, makes payments, and views travel status and booking history.

---

## 4. Core Entities

```text
User
Organization
Membership
Customer
Destination
Tour
ItineraryItem
Reservation
Payment
TourGuide
Hotel
Transportation
ReservationStatusHistory
```

Main relationships:

```text
Organization
 ├── Users / Memberships
 ├── Customers
 ├── Tours
 ├── Destinations
 ├── Tour Guides
 ├── Hotels
 └── Transportation

Customer ──< Reservation >── Tour
Reservation ──< Payment
Reservation ──< StatusHistory
Tour ──< ItineraryItem
```

---

## 5. Reservation Workflow

```text
PENDING
→ UNDER_REVIEW
→ APPROVED
→ WAITING_FOR_PAYMENT
→ PAID
→ CONFIRMED
→ READY_FOR_TRAVEL
→ IN_PROGRESS
→ COMPLETED
```

Alternative paths:

```text
PENDING → REJECTED
PENDING → CANCELLED
WAITING_FOR_PAYMENT → PAYMENT_EXPIRED
CONFIRMED → CANCELLED
```

Every status change must go through a valid transition.

---

## 6. Tour Capacity Management

Each Tour has a defined capacity.

```text
Capacity = 20
Reserved = 8
Remaining = 12
```

Each new Reservation decreases the remaining capacity. A valid cancellation releases the reserved capacity.

Overbooking is not allowed, and capacity operations must be handled transactionally.

---

## 7. Multi-Tenancy

Each Organization has isolated data.

* Users can only access data belonging to their Organization.
* Customers can only access their own information.
* Cross-tenant access is forbidden.
* Resources such as Tours, Hotels, and Tour Guides must belong to the same Organization.

Tenant Isolation must be verified through API tests.

---

## 8. Team Responsibilities

### Member A

* Accounts
* Organization
* JWT
* Permissions
* Docker / CI

### Member B

* Tours
* Destinations
* Itineraries
* Hotels
* Transportation
* Tour Guides

### Member C

* Customers
* Reservations
* Payments
* Capacity
* Reports

Ownership is not permanent. Every Feature should have an Owner and a Reviewer.

---

## 9. Learning and Development Order

1. Django and Models
2. DRF and REST APIs
3. JWT and Permissions
4. PostgreSQL and Migrations
5. Docker
6. Build the modules
7. Testing and CI
8. OpenAPI and Deployment

Write tests alongside development and regularly push changes to GitHub.

---

## 10. Important Tests

* Authentication
* Permissions
* Tenant Isolation
* Tour creation and management
* Reservation workflow
* Payment flow
* Overbooking prevention
* Transactions and Rollbacks
* Invalid status transitions
* Status History
* Travel History

---

## 11. Out of Scope for Now

* Real payment gateways
* Real SMS and Email services
* Mobile application
* Direct flight and hotel booking
* GDS integration
* Full CRM and accounting
* Microservices
* Kubernetes
* AI and Dynamic Pricing
* Celery and complex async systems

---

## 12. Future Improvements

If time allows:

* Redis and Caching
* Celery
* Additional roles
* Audit Logs
* Idempotency
* Advanced concurrency locking
* mypy

---

## 13. Project Success Criteria

The project should support the following complete flow:

```text
Create Organization
→ Create Customer
→ Create Tour
→ Add Itinerary / Hotel / Transportation
→ Assign Tour Guide
→ Publish Tour
→ Create Reservation
→ Approve Reservation
→ Payment
→ Confirm Reservation
→ Start Travel
→ Complete Travel
→ Travel History
```

The project must also have real, tested implementations of **Multi-Tenancy, Permissions, Tour Capacity Management, and Overbooking Prevention**.

The goal is to build a small but realistic system — not a simple CRUD application and not an overly complex Enterprise system.