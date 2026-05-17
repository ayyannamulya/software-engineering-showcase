<div align="center">

![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

# 🏥 WelorraCare — Hospital Management System

### *A production-grade Hospital Management System engineered to reflect how clinical environments actually operate — patients self-register, build their medical profile, and book appointments with available doctors; doctors manage their incoming queue, approve or reject requests, conduct examinations, and administer post-consultation medications.*

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma_ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)
![shadcn/ui](https://img.shields.io/badge/shadcn%2Fui-000000?style=for-the-badge&logo=shadcnui&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)

<br/>

[🌐 Live Demo](#) &nbsp;·&nbsp; [📑 Documentation](#)

[🎯 Portfolio](https://ayyanna-mulya.vercel.app) &nbsp;·&nbsp; [💼 LinkedIn](https://www.linkedin.com/in/noviyartimulyadi) &nbsp;·&nbsp; [🐙 GitHub](https://github.com/ayyannamulya) &nbsp;·&nbsp; [📧 Gmail](mailto:noviayya1121@gmail.com)

<br/>

> 🔒 **Showcase Repo** — Source code is private. This repo documents architecture, design decisions, and results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Hospital workflows collapse at the coordination layer — appointments are booked without conflict awareness, doctors have no structured queue, and patient records are disconnected from the clinical events that generated them. In small-to-mid-size clinic settings, this is typically managed through a mix of spreadsheets, phone calls, and disconnected tools with no unified audit trail.

The result: double-booked slots, lost patient histories, and medication records created in isolation with no traceable link back to the appointment or examination that initiated them.

---

## 💡 Solution

WelorraCare models the actual hospital workflow as a state machine — patient registers → builds their medical profile → books an appointment with an available doctor → doctor reviews and approves or rejects → examination is conducted → doctor logs post-consultation medications → admin maintains full oversight across all of it.

The defining architectural decision: **role isolation is enforced at two layers** — Clerk middleware gates route access by role before any request reaches the server, and Prisma queries are scoped per-role so no cross-tenant data leaks through the API surface. A patient cannot query another patient's record. A doctor's appointment view is filtered to their assigned patients only. Admin holds full read-write across all entities.

The tradeoff accepted: admin-provisioned doctors (no self-registration) to prevent role escalation through the auth flow — operational friction exchanged for a simpler, more auditable trust boundary.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| Appointment lifecycle management | Patient books from doctor's available slots → doctor approves/rejects → status propagates across dashboards | Eliminates double-booking; gives both parties a single source of truth on appointment state |
| Three-role RBAC | Clerk roles wired into Next.js middleware; route groups enforce access before page or API handler executes | Role enforcement at the edge — no server-side logic required to guard route access |
| Admin-provisioned doctor accounts | Admin creates doctor records; Clerk handles credential issuance; no public doctor registration path | Closes the role escalation vector present in open-registration systems |
| Per-role data isolation | Prisma queries filtered by `userId` or `doctorId` at the ORM layer — not the UI layer | Data scoping is structural, not cosmetic; wrong-role access fails at query time |
| Post-consultation medication management | Doctor logs medications after examination; records are linked to the appointment ID | Medication is tied to a clinical event — preserves the audit trail and prevents orphaned records |
| Admin operations dashboard | Aggregated stats across patients, doctors, appointments, and consultations with visual breakdown | Single-pane visibility across all entities; supports operational decisions without manual aggregation |

---

## 🏗️ Architecture

```
[Browser — Patient / Doctor / Admin]
          │
          │  HTTPS
          ▼
┌─────────────────────────────────┐
│     Next.js 15 Middleware       │  ← Clerk session validation, role extraction,
│     (Edge Runtime)              │    route group enforcement (/patient, /doctor, /admin)
└────────────────┬────────────────┘
                 │
     ┌───────────┴────────────┐
     │                        │
┌────▼─────┐         ┌────────▼───────┐
│ RSC Page │         │  API Routes    │  ← Server Actions / Route Handlers
│  Layer   │         │  (/api/*)      │    Input validated before ORM call
└────┬─────┘         └────────┬───────┘
     │                        │
     └───────────┬────────────┘
                 │
     ┌───────────▼────────────┐
     │      Prisma ORM        │  ← Query-level role scoping
     │   (Type-safe client)   │    All queries carry identity context
     └───────────┬────────────┘
                 │
     ┌───────────▼────────────┐
     │  PostgreSQL (Neon)     │  ← 14-table schema; managed serverless Postgres
     └────────────────────────┘

External:
  Clerk Auth  →  Session tokens, role metadata, webhook sync to DB
  Vercel      →  Edge middleware hosting, serverless function execution

Trust boundary : Clerk middleware (edge) — role verified before any app logic runs
Internal trust : Prisma client initialized per-request with user identity — no shared mutable state
```

### Data Flow — Appointment Lifecycle

| Step | Actor | Component | What Happens |
|------|-------|-----------|--------------|
| 1. Registration | Patient | Registration Page → API Route | Patient fills profile; record created in `Patient` table linked to Clerk `userId` |
| 2. Booking | Patient | Appointment Booking UI → API Route | Patient selects doctor and available slot; `Appointment` record created with status `PENDING` |
| 3. Review | Doctor | Appointments Dashboard | Doctor sees filtered queue (own patients only); approves or rejects → status updated to `APPROVED` / `REJECTED` |
| 4. Examination | Doctor | Medical Records → Medications | Post-consultation: doctor writes medical record and logs medication — all linked to the appointment ID |
| 5. Oversight | Admin | Admin Dashboard | Aggregated stats update; admin manages all entities (users, doctors, staffs, appointments, records) |

> Full schema breakdown and component dependency graph → [Architecture Doc](./docs/architecture.md)

---

## 🔐 Security Design

| Concern | Decision | Rationale |
|---------|----------|-----------|
| Authentication | Clerk (OAuth + credential) | Delegates session management, MFA, and credential storage — avoids rolling auth for a multi-role system |
| Authorization | Role-based via Clerk `publicMetadata` + Next.js middleware route groups | Role is set at provisioning time by admin; middleware enforces it at the edge before any page or API handler executes |
| Doctor provisioning | Admin-only creation path; no public registration for doctor/admin roles | Prevents role escalation through the signup flow — the highest-impact trust boundary in a multi-role system |
| Data isolation | Prisma queries scoped by `userId` / `doctorId` on every protected query | Isolation is structural at the ORM layer, not dependent on UI-layer filtering |
| Input trust | Server Actions and API Route Handlers validate inputs server-side before ORM calls | Client-side validation is UX only; server enforces schema independently |
| Secrets | Environment variables via Vercel; no hardcoded credentials in source | Clerk keys, DB connection string injected at deploy time |
| Audit trail | Appointment status transitions and medical records are append-only linked to appointment ID | Post-incident reconstruction is possible from DB state alone |

**Accepted risk:**
- No row-level locking on appointment slot selection — concurrent bookings on the same slot are handled at the application layer; under high concurrency this is a known race window
- Clerk webhook sync latency — DB user record may lag Clerk user creation by milliseconds on first login; handled with upsert pattern
- No mTLS between Next.js serverless functions and Neon — mitigated by Neon's TLS-enforced connection strings and Vercel's private networking

---

## 🧰 Tech Stack

| Layer | Technology | Decision Rationale |
|-------|------------|--------------------|
| Framework | Next.js 15 (App Router) | RSC + Server Actions reduce client-side data exposure; middleware runs at edge for zero-latency auth enforcement |
| Language | TypeScript | End-to-end type safety across Prisma schema, API contracts, and UI components |
| Auth | Clerk | Multi-role auth with metadata-based RBAC without building session infrastructure; webhook integration syncs identity to application DB |
| ORM | Prisma | Type-safe query client generated from schema; migrations are version-controlled; eliminates raw SQL injection surface |
| Database | PostgreSQL via Neon | Serverless Postgres fits Vercel's execution model; no connection pool management overhead |
| UI Components | shadcn/ui + Tailwind CSS | Accessible, unstyled primitives composed with utility classes — consistent design system across all three role dashboards without runtime CSS-in-JS overhead |
| Deployment | Vercel | Native Next.js 15 support; edge middleware execution; managed environment injection |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Manual | Role-switching across all three auth paths | Full appointment lifecycle: patient → doctor → admin state transitions |
| Integration | API routes tested with scoped identity context | Verified that cross-role data queries return empty, not error — silent isolation confirmed |
| Auth boundary | Attempted direct URL access across role groups without valid session | Middleware redirect behavior validated for all protected route groups |

**Explicitly not tested and why:** No automated E2E suite — the appointment state machine is linear and short enough that manual role-switching covers the critical paths. Load testing was not in scope for v1; the known concurrency risk on slot selection is documented as accepted.

---

## 📡 Observability

- **Logs:** Vercel function logs capture server action errors and API route failures; Clerk dashboard provides auth event history
- **Metrics:** Admin dashboard surfaces key operational signals — total patients, doctors, appointments, consultations, and appointment-to-consultation conversion ratio
- **Tracing:** Clerk webhook events provide identity lifecycle traceability; appointment ID acts as correlation key across medical records and medication tables
- **Alerting:** Not instrumented in v1 — Vercel error notifications enabled for function failures

---

## 📊 Impact & Results

| Metric | Result | Notes |
|--------|--------|-------|
| DB schema coverage | 14 tables | Covers identity, appointments, medical records, medications, staffing |
| Role surfaces covered | 3 (Patient, Doctor, Admin) | Fully isolated route groups with independent dashboard views |
| Appointment states | 3 (Pending → Approved / Rejected) | Full lifecycle from patient booking to doctor disposition |
| Post-consultation records | 2 linked entities | Medical record + medication both tied to appointment ID |
| Auth boundary violations | 0 in testing | Cross-role direct URL access correctly redirected in all tested paths |

---

## 🚧 Challenges

**Clerk role metadata sync into application DB**
Problem: Clerk's user object and the application's `User` table are separate stores. On first login, the DB record may not exist yet, causing downstream queries to fail.
Tried: Creating user records only on explicit registration form submission — failed for doctor accounts provisioned by admin before first login.
Solution: Upsert pattern on every authenticated request's first DB touch — idempotent write ensures record exists regardless of which path triggered the session. Tradeoff: one extra write on cold sessions.

**Role-scoped data isolation without duplicating query logic**
Problem: Three roles need overlapping data (appointments, patients) but with different filter scopes. Naive approach duplicates Prisma queries per role.
Tried: Single generic query with a role switch in the service layer — readable but brittle; easy to miss a filter condition when adding new fields.
Solution: Explicit per-role query functions with identity context required as a typed parameter — filter scope is structural, not conditional. Slightly more code, significantly more auditable.

**Doctor provisioning without exposing the admin role creation path**
Problem: Clerk's default signup flow assigns no role — if a doctor hits the public signup page, they land in patient-territory with no way to escalate legitimately, but also no guard against attempting it.
Tried: Hiding the doctor signup link — security through obscurity, rejected.
Solution: Admin-only doctor creation via a server action that sets Clerk `publicMetadata.role = "doctor"` and creates the DB record in a single transaction. No public path to doctor or admin roles exists.

---

## 🔄 What I'd Do Differently

- **Appointment concurrency:** Implement `SELECT FOR UPDATE` within a Prisma transaction on slot selection to close the race window on concurrent bookings. The current application-layer approach is sufficient at low concurrency but would fail under real clinic load.
- **Webhook-first DB sync:** Replace the upsert-on-request pattern with a Clerk webhook handler that creates the DB user record immediately on `user.created` — removes the cold-session write and makes the sync explicit and auditable.
- **Audit log table:** Add an explicit `AuditLog` table capturing who changed what and when across appointments and medical records — currently reconstructable from DB state but not directly queryable.
- **Medication as a prescription model:** The current medication records are flat post-consultation entries. A proper prescription model with dosage, frequency, duration, and dispensing status would make it clinically useful rather than just a log.

---

## 📸 Screenshots

| View | |
|------|-|
| Admin Dashboard | ![Admin Dashboard](./screenshots/admin-dashboard.png) |
| Doctor Appointments | ![Doctor Appointments](./screenshots/doctor-appointments.png) |
| Patient Booking | ![Patient Booking](./screenshots/patient-booking.png) |
| Medical Records | ![Medical Records](./screenshots/medical-records.png) |

---

## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Full schema breakdown, component dependency graph, trust model, Clerk integration detail |
| [Role Guide](./docs/roles.md) | What each role can read and write, provisioning flow, and permission boundaries |

