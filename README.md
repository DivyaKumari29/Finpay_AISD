# 💳 FinPay — Campus Wallet, Payments & Cashback

> A full-stack campus payment platform built with **FastAPI + SQLAlchemy + SQLite**, featuring a modern glassmorphism web UI, role-based dashboards for Students, Merchants, and Admins, and an atomic payment engine with automatic cashback.

---

## ✨ Features

| Area | Description |
|---|---|
| **Authentication** | JWT-based login/register with role-based access (student / merchant / admin) |
| **Student Wallet** | Auto-created wallet on registration — top up, check balance, view full ledger history |
| **Payments** | Student pays a merchant; balance validated, wallet debited, cashback credited atomically |
| **Cashback Engine** | 2% cashback on eligible payments, capped at ₹50 per transaction |
| **Merchant Profiles** | Merchants create a business profile; view all payments received |
| **Admin Reconciliation** | Monthly report: total sales, cashback, per-merchant settlement, wallet consistency check |
| **Modern Web UI** | Glassmorphism design with sidebar navigation, role dashboards, SVG icons, live clock |

---

## 🖥️ UI Preview

The web UI features a **glassmorphism** design inspired by modern dashboard interfaces:

- **Frosted-glass cards** with soft blue-grey gradient background
- **Left icon sidebar** with tooltip labels and active state indicators
- **Role-based dashboards**: Student / Merchant / Admin each get their own view
- **Inline SVG icons** (Feather-style) — no emoji, consistent across all platforms
- **Live clock** in the top bar, greeting, and user avatar pill
- Loading spinner on API calls, smooth page-entry animations

---

## 🏗️ Architecture — Layered

```
Client (Browser / Swagger UI)
        ↓
Router          → receives HTTP request, validates role
        ↓
Service         → all business rules (payment, cashback, reconciliation)
        ↓
Repository      → database queries only, no business logic
        ↓
Model           → SQLAlchemy table definitions
        ↓
Database        → SQLite (finpay.db)
```

| Layer | Folder | Responsibility |
|---|---|---|
| Router / API | `app/routers/` | Accept requests, check role, call service |
| Service | `app/services/` | Business rules — payment flow, cashback, reconciliation |
| Repository | `app/repositories/` | Database CRUD — no business rules |
| Model | `app/models/` | Table definitions |
| Database | `finpay.db` | SQLite (auto-created on first run) |

**Dependency rule:** Routers → Services → Repositories → Models. Never skip layers.

---

## 🗄️ Database Schema

| Table | Key Columns | Purpose |
|---|---|---|
| `users` | id, name, email (unique), password_hash, role | All accounts |
| `wallets` | id, user_id (unique), balance | One wallet per student |
| `wallet_ledger` | id, user_id, entry_type, amount, reference | Every balance change (TOPUP / PAYMENT / CASHBACK) |
| `merchants` | id, user_id (unique), business_name, category | Merchant profile |
| `transactions` | id, student_id, merchant_id, amount, cashback, status | Each payment (PENDING → SUCCESS) |
| `cashback_records` | id, student_id, transaction_id (unique), amount, rate | One cashback record per eligible transaction |

---

## 📐 Business Rules

**Wallet**
- Wallet is auto-created when a student registers.
- Top-up: amount > 0, maximum ₹50,000 per top-up.
- Payment rejected if balance < amount.

**Cashback**

| Rule | Value |
|---|---|
| Rate | **2%** of payment amount |
| Minimum payment to earn cashback | **₹100** |
| Maximum cashback per payment | **₹50** |

Examples: ₹500 → ₹10 cashback · ₹50 → ₹0 (below minimum) · ₹5,000 → ₹50 (capped)

---

## 🔄 Payment Flow (Atomic)

```
POST /payments  →  payment_router (validates student role)
                        ↓
                   payment_service (validates amount, creates PENDING txn)
                        ↓
                   wallet_service (checks balance, debits wallet, PAYMENT ledger entry)
                        ↓
                   cashback_service (calculates cashback, credits wallet, CASHBACK ledger entry)
                        ↓
                   repositories (persist all records)
                        ↓
                   payment_service (marks txn SUCCESS, single commit)
```

If any step fails → full rollback, nothing is persisted.

---

## 🧰 Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.10+ |
| Web Framework | FastAPI |
| ASGI Server | Uvicorn |
| ORM | SQLAlchemy 2.x |
| Database | SQLite |
| Auth | PyJWT (HS256) |
| Password Hashing | bcrypt (via passlib) |
| Testing | pytest + httpx (ASGI transport) |
| UI | Vanilla HTML/CSS/JS (glassmorphism) |

---

## 📁 Project Structure

```
finpay/
├── app/
│   ├── core/
│   │   ├── database.py        # SQLAlchemy engine & session
│   │   ├── errors.py          # AppError exception
│   │   ├── security.py        # JWT encode/decode, password hash
│   │   └── utils.py           # Shared utilities
│   ├── models/
│   │   ├── user.py
│   │   ├── wallet.py
│   │   ├── merchant.py
│   │   ├── transaction.py
│   │   └── cashback.py
│   ├── repositories/
│   │   ├── user_repository.py
│   │   ├── wallet_repository.py
│   │   ├── merchant_repository.py
│   │   ├── transaction_repository.py
│   │   └── cashback_repository.py
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── wallet_service.py
│   │   ├── merchant_service.py
│   │   ├── payment_service.py
│   │   ├── cashback_service.py
│   │   └── reconciliation_service.py
│   ├── routers/
│   │   ├── auth_router.py
│   │   ├── wallet_router.py
│   │   ├── merchant_router.py
│   │   ├── payment_router.py
│   │   ├── cashback_router.py
│   │   └── reconciliation_router.py
│   ├── static/
│   │   └── index.html         # Full single-page web UI
│   ├── schemas.py             # Pydantic request/response models
│   └── main.py                # FastAPI app factory
├── tests/
│   └── test_flow.py           # End-to-end integration tests
├── requirements.txt
└── README.md
```

---

## 🚀 Getting Started

### 1. Prerequisites
- Python 3.10 or newer
- Git

### 2. Clone & Install

```bash
git clone https://github.com/NikhilHatti/FinPay.git
cd FinPay
pip install -r requirements.txt
```

### 3. Run the Server

```bash
python -m uvicorn app.main:app --reload
```

> The SQLite database (`finpay.db`) is created automatically on first run.  
> To reset, delete `finpay.db` and restart.

### 4. Open in Browser

| URL | Description |
|---|---|
| http://127.0.0.1:8000/ | Web UI |
| http://127.0.0.1:8000/docs | Swagger / Interactive API Docs |
| http://127.0.0.1:8000/health | Health check |

---

## 🔌 API Reference

> Use Swagger at `/docs` → click **Authorize** → paste the JWT from `POST /auth/login`.

| Method | Endpoint | Role | Description |
|---|---|---|---|
| POST | `/auth/register` | public | Create account |
| POST | `/auth/login` | public | Returns JWT access token |
| GET | `/auth/me` | any | Current user info |
| GET | `/wallet/balance` | student | Wallet balance |
| POST | `/wallet/topup` | student | Add money |
| GET | `/wallet/history` | student | Ledger entries |
| POST | `/merchants` | merchant | Create merchant profile |
| GET | `/merchants` | any | List all merchants |
| GET | `/merchants/me` | merchant | Own profile |
| POST | `/payments` | student | Pay a merchant |
| GET | `/payments/history` | student | Own payment history |
| GET | `/payments/received` | merchant | Payments received |
| GET | `/cashback/rules` | any | Current cashback config |
| GET | `/cashback/history` | student | Cashback earned |
| GET | `/reconciliation/monthly?year=&month=` | admin | Monthly report |
| GET | `/health` | public | Health check |

### Sample Requests

**Register**
```json
POST /auth/register
{ "name": "Nikhil", "email": "student@kle.edu", "password": "secret123", "role": "student" }
```

**Login response**
```json
{ "access_token": "<jwt>", "token_type": "bearer", "role": "student" }
```

**Top up**
```json
POST /wallet/topup
{ "amount": 1000 }
```

**Pay a merchant**
```json
POST /payments
{ "merchant_id": 1, "amount": 500 }
```

**Response**
```json
{ "id": 1, "student_id": 1, "merchant_id": 1, "amount": 500.0, "cashback": 10.0, "status": "SUCCESS" }
```

---

## 🧪 Testing

```bash
python -m pytest -v
```

The test suite covers:
- Registration and login for all three roles
- Merchant profile creation
- Wallet top-up
- Payment with 2% cashback
- No cashback below ₹100 minimum
- Rejection of over-balance payment (balance unchanged)
- Role protection (student cannot access admin endpoints)
- Monthly reconciliation report with `BALANCED` status
- Web UI is served correctly

---

## 🎮 Demo Walkthrough

1. **Merchant** → Register (role: Merchant) → Create business profile (e.g., *KLE Canteen*, category: food)
2. **Student** → Register (role: Student) → Top up ₹1,000 → Pay the merchant ₹500 → see ₹10 cashback credited
3. **Try a failure** → Pay more than your balance → see "Insufficient wallet balance" (nothing saved)
4. **Admin** → Register (role: Admin) → Open monthly report → verify total sales ₹500, cashback ₹10, status **BALANCED**
5. **Merchant again** → Log in → see the ₹500 payment under "Payments received"

---

## ⚙️ Configuration

Edit constants in `app/services/cashback_service.py` to change cashback rules:

```python
CASHBACK_RATE = 0.02          # 2%
MIN_PAYMENT_FOR_CASHBACK = 100 # ₹100
MAX_CASHBACK_PER_PAYMENT = 50  # ₹50
```

JWT secret and algorithm are set in `app/core/security.py`.

---

## 🚧 Limitations & Future Work

- SQLite is used for simplicity; swap for PostgreSQL for production.
- No email verification or password reset flow.
- Cashback rules are hard-coded constants (no admin UI to change them).
- No pagination on ledger / transaction lists.
- Could add WebSocket notifications for real-time cashback alerts.

---

## 📄 License

MIT License — free to use and modify.

---

<div align="center">
  Built with FastAPI &nbsp;·&nbsp; SQLAlchemy &nbsp;·&nbsp; SQLite &nbsp;·&nbsp; Vanilla JS
</div>
