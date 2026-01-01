# System Architecture

## Overview
The system follows a simple web architecture:

Frontend → Backend API → Database → File Storage

## Key Rules
- No hard delete
- All changes logged
- Role-based access

## Future Ready
Payment gateway can be added later
without breaking existing system.
🕌 Masjid Management App – Complete Architecture
<img width="600" height="385" alt="image" src="https://github.com/user-attachments/assets/6912d660-0a8f-4880-b8c5-0c780f5b8e61" />
<img width="990" height="979" alt="image" src="https://github.com/user-attachments/assets/9708921e-320c-4883-823a-db52d7629b72" />
<img width="963" height="372" alt="image" src="https://github.com/user-attachments/assets/1af90bf8-99f3-41bd-9bfe-fb30b8cfbb8d" />

1️⃣ High-Level Architecture (Logical View)
┌─────────────────────────────────────────────┐
│                USERS                        │
│ Admin | Verifier | Teacher | Auditor        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│           FRONTEND (Web / PWA)               │
│ Vue.js / React                               │
│ - RBAC UI                                   │
│ - Forms (Members, Donations, Makhtab)       │
│ - Dashboards & Reports                      │
│ - Offline Support (IndexedDB)               │
│ - Camera (Receipt Upload)                   │
└─────────────────────────────────────────────┘
                    │  HTTPS + JWT
                    ▼
┌─────────────────────────────────────────────┐
│           BACKEND API LAYER                  │
│ Node.js / ASP.NET Core                      │
│---------------------------------------------│
│ Auth & RBAC Middleware                      │
│ Business Logic (Stage-wise)                 │
│ Audit Logger (Append-only)                  │
│ Verification Workflow Engine                │
│ File Upload Handler                         │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌───────────────────┐   ┌────────────────────┐
│   DATABASE        │   │   FILE STORAGE      │
│ PostgreSQL        │   │ Receipts / Docs     │
│ (Stage-wise)      │   │ (Local / S3)        │
└───────────────────┘   └────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│        BACKUP & AUDIT ACCESS                 │
│ - Daily DB Backup                            │
│ - Read-only Auditor Access                  │
└─────────────────────────────────────────────┘

2️⃣ Frontend Architecture (Developer View)
frontend/
├─ src/
│  ├─ pages/
│  │  ├─ login/
│  │  ├─ dashboard/
│  │  ├─ members/
│  │  ├─ donations/
│  │  ├─ makhtab/
│  │  ├─ books/
│  │  ├─ assets/
│  │  └─ reports/
│  ├─ components/
│  │  ├─ forms/
│  │  ├─ tables/
│  │  └─ charts/
│  ├─ stores/ (Pinia / Redux)
│  ├─ services/ (API calls)
│  └─ utils/
└─ public/


Key Design Rules

Role-based menu rendering

No business logic in UI

Offline cache → sync on reconnect

3️⃣ Backend Architecture (Clean & Auditable)
backend/
├─ src/
│  ├─ controllers/     → HTTP layer
│  ├─ services/        → Business logic
│  ├─ repositories/   → DB access
│  ├─ middleware/
│  │   ├─ auth.guard
│  │   ├─ role.guard
│  │   └─ audit.logger
│  ├─ workflows/
│  │   └─ donationVerification.ts
│  ├─ modules/
│  │   ├─ members
│  │   ├─ donations
│  │   ├─ makhtab
│  │   ├─ books
│  │   ├─ assets
│  │   └─ governance
│  └─ utils/
└─ uploads/


Golden Rules

❌ No hard delete

✔ Every write → audit log

✔ Entry ≠ Verification

4️⃣ Database Architecture (Stage-Wise Growth)
🔹 Core Tables (Stage 1–2)
USERS ─┬─< DONATIONS >─┬─ RECEIPTS
       │               │
       └─ AUDIT_LOGS   └─ VERIFICATIONS

🔹 Makhtab (Stage 3)
TEACHERS ──< BATCHES >── STUDENTS
                       │
                       └── ATTENDANCE

🔹 Books & Assets (Stage 4)
BOOKS ──< BOOK_ISSUES >── MEMBERS
ASSETS ──< ASSET_USAGE >── EVENTS
          │
          └── DAMAGE_REPORTS

🔹 Governance (Stage 6)
COMMITTEE_MEMBERS
        │
        └── RESOLUTIONS ──< APPROVALS

5️⃣ Donation Verification Workflow (Critical)
Donation Entry
     │
     ▼
Status = PENDING
     │
     ▼
Verifier Review
     │
 ┌───┴─────────┐
 ▼             ▼
VERIFIED     REJECTED
 │
 ▼
Monthly Lock


✔ No auto-credit
✔ All actions logged
✔ Locked after month close

6️⃣ Security Architecture
Layer	Control
UI	Role-based access
API	JWT + RBAC guards
DB	Soft delete only
Files	Type & size validation
Audit	Append-only logs
Ops	Daily backups
7️⃣ Future-Ready Extension (Payment Gateway – Later)
Frontend
   │
   ▼
Payment Adapter API
   │
   ├─ Razorpay
   ├─ Cashfree
   └─ PayU
   │
   ▼
Webhook Handler
   │
   ▼
Verification Queue


Important

Gateway is plug-in, not core

Manual verification still mandatory

8️⃣ Why This Architecture Works
Aspect	Reason
Stage-wise	Easy approval & rollout
Modular	No rewrite later
Audit-first	Committee trust
Simple	Non-technical users
Scalable	500 → 5000 users
