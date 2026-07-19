# CommerceOps — Simplified & Achievable Version

> For a 3-person beginner team. No deadline. Goal: internship.

---

## 1. What We're Keeping vs Cutting

### KEEP (Impressive + Learnable)

| Feature | Why Keep |
|---|---|
| JWT Auth (login/refresh/logout) | Every company uses this. Interview gold. |
| PostgreSQL + Migrations | Real database skills, not SQLite toy projects |
| Docker Compose | Every job listing asks for Docker |
| REST API with DRF | Industry standard |
| Git + GitHub Flow | Professional workflow |
| Basic Multi-tenancy | Shows you understand data isolation |
| Simple Order Model | Core business logic |
| Pytest + Basic CI | Testing is a must-have on resumes |
| Swagger/OpenAPI docs | Makes your API look professional |

### CUT (Too Complex for Beginners)

| Feature | Why Cut | What to Do Instead |
|---|---|---|
| Concurrent reservation locking | `select_for_update` + deadlock prevention is hard | Simple check: if available >= quantity, reserve. Accept the race condition exists. |
| Idempotency (IdempotencyRecord table) | Complex state machine for keys | Just document it. Skip implementation. |
| Payment webhook + simulator | External integration complexity | Simple boolean toggle: mark order as paid via a direct API call |
| Celery async tasks | Broker setup, retries, task state management | Do everything synchronously. No background jobs. |
| Multi-role RBAC (6 roles) | Permission matrix is huge | 2 roles only: Admin and Member |
| Audit Log (append-only) | Extra table, extra complexity | Skip entirely |
| 12 modules | Too many to learn and build | Merge into 5 simpler modules |
| Row-level security | Advanced PostgreSQL | Just filter by org_id in queries |
| Full Fulfillment lifecycle | Picking → Packing → Shipment is a lot | Simple: mark order as "shipped" with a tracking number |
| Structured JSON logging | Nice-to-have, not essential | Standard Django logging |
| mypy + type checking | Steep learning curve for beginners | Skip for now, add later if time allows |
| ADR documents | Overkill for an internship project | Just a README explaining choices |

---

## 2. What We're Actually Building (Simplified)

A backend API where a small business can:
1. **Sign up / Log in** with JWT
2. **Create products** with variants and prices
3. **Manage stock** (add/remove inventory)
4. **Create orders** (pick customer, products, quantities)
5. **Mark orders as paid / shipped / delivered**
6. **View reports** (order counts, low stock alerts)

**That's it.** Clean, simple, demonstrable.

---

## 3. Simplified Module Structure

```
commerceops/
├── src/
│   ├── manage.py
│   ├── config/          # Django settings, urls
│   ├── accounts/        # User, JWT auth, simple roles
│   ├── products/        # Product, Variant, Warehouse, Stock
│   ├── orders/          # Customer, Order, OrderItem, Payment status
│   ├── reports/         # Simple query endpoints
│   └── common/          # Shared utilities, base models
├── tests/
├── docker/
├── compose.yaml
├── pyproject.toml
└── README.md
```

**5 modules instead of 12.** Each person owns ~2 modules.

---

## 4. Simplified Data Model

```
User (id, email, password, role, org_id)
    │
Organization (id, name, slug)
    │
Product (id, org_id, name, description, is_active)
    │
ProductVariant (id, product_id, org_id, sku, name, price, stock_quantity)
    │
Customer (id, org_id, name, email, phone)
    │
Order (id, org_id, customer_id, status, total, created_at)
    │
OrderItem (id, order_id, variant_id, quantity, unit_price)
```

**Key simplifications:**
- Stock is just an integer on `ProductVariant` (no separate InventoryItem table)
- No separate Reservation model (stock deducted when order is created)
- No Fulfillment/Shipment models (just order status field)
- No Address model (skip for now)
- No StockMovement audit trail

**Order statuses:** `PENDING` → `PAID` → `SHIPPED` → `DELIVERED` (or `CANCELLED`)

---

## 5. JWT — What You Need to Know

### Install
```bash
pip install djangorestframework-simplejwt
```

### Config (settings.py)
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}
```

### Endpoints (auto-provided)
```
POST /api/token/          → get access + refresh tokens
POST /api/token/refresh/  → get new access token
POST /api/token/blacklist/ → logout (blacklist refresh)
```

### How to use in requests
```
Authorization: Bearer <access_token>
```

### What to implement yourself
- Custom User model with email login (not username)
- Role field (admin/member)
- `POST /auth/register/` endpoint
- `GET /auth/me/` endpoint (current user profile)
- Permission classes that check role

### What to explain in interview
- Access token is **short-lived** (15 min) — limits damage if stolen
- Refresh token is **long-lived** (7 days) — used to get new access tokens
- **Rotation**: each refresh gives a NEW refresh token, old one is blacklisted
- This prevents **replay attacks** with stolen refresh tokens
- Never store tokens in localStorage in production (use httpOnly cookies)

---

## 6. Key Concepts That Look Great in Interviews

### REST API Design
```
GET    /api/v1/products/          → list products
POST   /api/v1/products/          → create product
GET    /api/v1/products/{id}/     → get one product
PATCH  /api/v1/products/{id}/     → update product

GET    /api/v1/orders/            → list orders
POST   /api/v1/orders/            → create order
POST   /api/v1/orders/{id}/pay/   → mark as paid
POST   /api/v1/orders/{id}/ship/  → mark as shipped
```

### HTTP Status Codes
| Code | When |
|---|---|
| 200 | Success (GET, PATCH) |
| 201 | Created (POST) |
| 400 | Bad request / validation error |
| 401 | Not logged in |
| 402 | Logged in but no permission |
| 404 | Not found |
| 409 | Conflict (e.g., insufficient stock) |

### Error Response Pattern
```json
{
    "error": {
        "code": "INSUFFICIENT_STOCK",
        "message": "Only 3 items available.",
        "sku": "TSHIRT-M-BLK"
    }
}
```

### Money Handling
- Always use `Decimal` in Python, `numeric(10,2)` in PostgreSQL
- **Never use float** for money (floating point errors)

---

## 7. Docker Compose (Keep It Simple)

```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - postgres
    environment:
      - DATABASE_URL=postgres://user:pass@postgres:5432/commerceops
      - REDIS_URL=redis://redis:6379/0
      - SECRET_KEY=your-secret-key

  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: commerceops
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  pgdata:
```

### Dockerfile
```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src/ .
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

**No Celery, no worker, no beat.** Just API + Postgres + Redis.

---

## 8. Simple CI with GitHub Actions

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_DB: test_commerceops
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.13"
      - run: pip install -r requirements.txt
      - run: ruff check src/
      - run: pytest --tb=short
    env:
      DATABASE_URL: postgres://test:test@localhost:5432/test_commerceops
      SECRET_KEY: test-secret
```

This shows: linting, testing, database in CI — exactly what companies want.

---

## 9. How to Divide Work (3 People)

### Member A — Auth & Foundation (Week 1-2 focus)
```
accounts/ module:
- Custom User model (email-based login)
- JWT setup (register, login, refresh, logout)
- Simple role permission (admin vs member)
- Organization model (name, slug)
- Docker Compose setup
- CI pipeline
```

### Member B — Products & Stock (Week 2-3 focus)
```
products/ module:
- Product CRUD
- ProductVariant CRUD
- Warehouse (simple model: name, location)
- Stock management (receive stock, check availability)
- Low stock alert endpoint
```

### Member C — Orders & Reports (Week 3-4 focus)
```
orders/ module:
- Customer model
- Order creation (with stock validation)
- Order status transitions (pay, ship, deliver, cancel)
- Order list with filters
- Report endpoints (order summary, low stock list)
```

### After Week 4 — Together
- Integration testing (full flow: create user → add products → create order → pay → ship)
- README with setup instructions, API docs, screenshots
- Swagger/OpenAPI documentation
- Final polish and demo data

---

## 10. Learning Plan (Step by Step)

Since you're beginners, here's what to learn **in order**:

### Step 1: Django Basics (few days)
- Models, migrations, admin
- `python manage.py` commands
- Django shell

### Step 2: DRF Basics (few days)
- Serializers
- ViewSets + Routers
- `GET`, `POST`, `PATCH`, `DELETE`

### Step 3: JWT Auth (1-2 days)
- Install simplejwt
- Configure settings
- Login/logout/refresh
- `Permission` classes

### Step 4: PostgreSQL (1 day)
- Install and connect
- Understand migrations
- Basic queries

### Step 5: Docker (1-2 days)
- `Dockerfile`
- `docker compose up`
- Connect services

### Step 6: Build the Project
- Follow the module order above
- Write tests as you go
- Push to GitHub regularly

---

## 11. What Makes This Project Stand Out for Internships

Even simplified, this project demonstrates:

| Skill | How |
|---|---|
| **JWT Authentication** | Full login/refresh/logout flow |
| **REST API Design** | Consistent URL patterns, proper status codes, error handling |
| **PostgreSQL** | Real database (not SQLite), migrations, constraints |
| **Docker** | Containerized multi-service app |
| **Git Workflow** | Branches, PRs, meaningful commits |
| **Testing** | Pytest suite with API tests |
| **CI/CD** | GitHub Actions pipeline |
| **Clean Architecture** | Separated modules with clear responsibilities |
| **Business Logic** | Order flow, stock validation, status transitions |
| **Error Handling** | Structured error responses, not crashing |

### README That Impresses
Include:
1. What it does (1 paragraph)
2. Tech stack with badges
3. How to run it (copy-paste commands)
4. API documentation (endpoint table)
5. Architecture diagram (even simple)
6. What you learned

---

## 12. Things to Add Later If You Have Time

Only after the core is working:

1. **Redis caching** — cache product list
2. **Celery** — async email notifications
3. **More roles** — warehouse operator, finance
4. **Audit log** — track who did what
5. **Concurrency protection** — `select_for_update` on stock
6. **Idempotency** — prevent duplicate orders
7. **mypy** — static type checking

**Do not add these first.** Build the simple thing, make it work, make it clean, then add complexity.
