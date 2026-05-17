<div align="center">

![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

# 📅 Manaka — Smart Scheduling SaaS

### *A full-stack scheduling platform that syncs with Google Calendar, auto-generates Google Meet links, and serves shareable public booking pages — with timezone intelligence and tiered billing baked in.*

![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_v4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Shadcn](https://img.shields.io/badge/shadcn%2Fui-000000?style=for-the-badge&logo=shadcnui&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)
![Sanity](https://img.shields.io/badge/Sanity_CMS-F03E2F?style=for-the-badge&logo=sanity&logoColor=white)
![Google Calendar](https://img.shields.io/badge/Google_Calendar-4285F4?style=for-the-badge&logo=googlecalendar&logoColor=white)
![Google Meet](https://img.shields.io/badge/Google_Meet-00897B?style=for-the-badge&logo=googlemeet&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)
![Biome](https://img.shields.io/badge/Biome-60A5FA?style=for-the-badge&logo=biome&logoColor=white)

<br/>

[🌐 Live Demo](#) &nbsp;·&nbsp; [📑 Documentation](#)

[🎯 Portfolio](https://ayyanna-mulya.vercel.app) &nbsp;·&nbsp; [💼 LinkedIn](https://www.linkedin.com/in/noviyartimulyadi) &nbsp;·&nbsp; [🐙 GitHub](https://github.com/ayyannamulya) &nbsp;·&nbsp; [📧 Gmail](mailto:noviayya1121@gmail.com)

> 🔒 **Showcase Repo** — Source code is private. This repository documents architecture, design decisions, and engineering results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Scheduling a meeting in 2024 still involves a back-and-forth thread of "does Tuesday work?", manual calendar checks, Zoom link generation, and timezone math — every time, for every meeting. Tools like Calendly solve this, but they're black boxes: no control over the data model, no CMS for content, no programmatic access to availability logic, and pricing that punishes growth.

The gap for developers and small SaaS operators: build your own scheduling layer that you actually own — one that syncs real calendar state, respects existing events, handles OAuth token lifecycle correctly, and ships with a tiered billing model baked in from day one.

---

## 💡 Solution

Manaka is a scheduling SaaS where availability is computed, not manually entered. Google Calendar is the source of truth — Manaka reads existing events across connected accounts to determine real free/busy state, so double-booking is structurally impossible, not just discouraged.

The core architectural decision: treat Google Calendar as the authoritative state store and use a lazy deletion pattern for cancellations — cancelled bookings are reflected in Google Calendar first, and Manaka's own records reconcile against that. This eliminates an entire class of sync bugs at the cost of a slightly more complex cancellation read path.

Clerk handles both authentication and subscription billing (Free / Starter / Pro), keeping plan enforcement co-located with identity — no separate billing service to keep in sync. Sanity CMS manages all content with an embedded Studio at `/studio`, giving non-technical operators real-time control without a deployment.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| 📅 Visual Availability Management | React Big Calendar with drag-and-drop time block creation | Availability feels intuitive — no form-filling, just block your free time directly on the calendar |
| 🔄 Multi-Calendar Sync | OAuth2-connected Google accounts queried in parallel; conflicts resolved before slot is offered | Guests only ever see times that are actually free — across all your calendars |
| 🎥 Automatic Google Meet | Google Calendar API creates Meet link at booking time, embedded in the calendar invite | Zero manual link generation; guest gets a working video call in the confirmation email |
| 🌍 Timezone Intelligence | Host timezone stored at setup; guest timezone auto-detected on booking page; all times converted server-side | Eliminates the most common scheduling failure mode for distributed teams |
| 🔗 Public Booking Pages | Clean routes at `/book/username/meeting-type`, statically renderable, no auth required for guests | Shareable by link — no account required to book, no friction for the guest |
| 🔐 Clerk Auth + Billing | Single SDK handles session auth and subscription plan enforcement; plan limits checked at Server Action layer | Feature gating (calendar slots, monthly booking count) enforced server-side, not just in UI |
| 📊 Admin Dashboard | Booking insights, attendee status tracking (accepted / declined / pending), user feedback management | Operators see the full picture without touching the database |
| ⏱️ Flexible Meeting Types | User-defined durations (15 / 30 / 45 / 60 / 90 min) with individual availability rules per type | One host, multiple meeting products — each with its own scheduling logic |

---

## 🏗️ Architecture

```
[Guest Browser]                    [Host Browser]
      │                                  │
      │  HTTPS (no auth required)        │  HTTPS + Clerk session
      ▼                                  ▼
┌─────────────────────────────────────────────────┐
│              Next.js 16 App Router              │
│  /book/[username]/[type]  │  /dashboard/*       │
│  Server Components (RSC)  │  Server Actions     │
│  No session required      │  Clerk-gated        │
└───────────────┬───────────────────┬─────────────┘
                │                   │
        ┌───────┴──────┐    ┌───────┴──────────┐
        │  Sanity CMS  │    │   Google Calendar │
        │  (content,   │    │   API (OAuth2)    │
        │   feedback)  │    │   Free/busy query │
        └──────────────┘    │   Event creation  │
                            │   Meet link gen   │
                            └──────────────────-┘

Trust boundary  : Clerk middleware enforces session on all /dashboard routes and Server Actions
Plan enforcement: Subscription tier checked in Server Action before any Google Calendar write
External trust  : Google Calendar API responses validated before slot availability is computed
OAuth lifecycle : Refresh tokens stored encrypted; automatic refresh on 401 from Google API
```

### Data Flow — Booking (Guest Side)

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Page Load | Next.js RSC `/book/[username]/[type]` | Host availability and meeting type fetched server-side; no auth cookie required |
| 2. Free/Busy Compute | Google Calendar API | All connected Google accounts queried; existing events merged; free slots calculated against host-defined availability blocks |
| 3. Timezone Conversion | Server-side | Guest timezone auto-detected from browser locale; all slots converted before rendering |
| 4. Slot Selection | Client Component | Guest selects slot, enters name + email; no account creation required |
| 5. Booking Creation | Server Action | Plan limit checked (monthly booking count); Google Calendar event created with both parties; Meet link auto-generated |
| 6. Confirmation | Google Calendar API | Invite sent to guest email with Meet link; event appears on host's calendar immediately |

### Data Flow — Cancellation (Lazy Deletion)

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Cancel Request | Server Action | Event deleted from Google Calendar (source of truth) |
| 2. Local Record | Sanity / DB | Booking marked cancelled; not hard-deleted — preserves audit trail |
| 3. Reconciliation | On next free/busy query | Cancelled slot no longer appears as busy; automatically opens for rebooking |

---

## 🔐 Security Design

| Concern | Decision | Rationale |
|---------|----------|-----------|
| 🔑 Authentication | Clerk session tokens (HTTP-only cookie) | Industry-standard session management; no custom JWT handling required |
| 🛡️ Authorization | Clerk middleware on all `/dashboard` routes + Server Action plan checks | Plan limits enforced server-side — UI gating alone is bypassable via direct API call |
| 🔑 OAuth2 Tokens | Refresh tokens stored encrypted; automatic refresh on Google API 401 | Token expiry is a common production failure mode; handled transparently without user re-auth |
| 🧹 Input Trust | Booking form inputs validated server-side before any Google API call | Guest-supplied name/email never interpolated into Calendar API payloads without sanitization |
| 🔐 Secrets Management | Google OAuth credentials, Clerk keys, Sanity tokens via environment variables only | No credentials committed; rotation handled at platform (Vercel) layer |
| 📋 Audit Logging | Lazy deletion preserves cancelled booking records; attendee status changes logged | Post-dispute reconstructability — who booked, when, what changed |

**⚠️ Accepted Risk:**
- Google Calendar API rate limits (per-user quota) — not explicitly throttled at application layer in v1. Mitigated by the booking flow making a bounded number of Calendar API calls per request (free/busy query + event creation). Will need explicit rate limiting if multi-tenant usage scales to high booking volume per user.
- Attendee status (accepted/declined/pending) is pulled from Google Calendar on demand, not webhooks. Status can be stale between polling intervals — acceptable for the current use case; webhook subscription is the v2 path.

---

## 🧰 Tech Stack

| Technology | Role in This Project | Why This, Not the Alternative |
|------------|----------------------|-------------------------------|
| **Next.js 16 + React 19** | Full-stack framework; App Router for RSC, Server Actions for all mutations | Server Actions eliminate a dedicated API layer for host-side operations; RSC enables the public booking page to be fully server-rendered without a separate backend |
| **Clerk** | Authentication + subscription billing in a single SDK | Clerk's billing primitives handle plan enforcement co-located with identity — no separate Stripe integration to maintain for feature gating |
| **Sanity CMS + TypeGen** | Content management with embedded Studio at `/studio`; TypeGen for type-safe GROQ queries | Non-technical operators get real-time content control without a deployment; TypeGen eliminates the `any` cast on CMS data |
| **Google Calendar API + OAuth2** | Free/busy queries, event creation, Meet link generation | Google Calendar is the only source of truth — reading existing events to compute availability is more reliable than maintaining a parallel availability state |
| **React Big Calendar** | Visual drag-and-drop availability block management | Drag-to-create time blocks are the most intuitive availability UI; no viable alternative in the React ecosystem at this interaction fidelity |
| **Tailwind v4 + Shadcn/ui** | Styling + accessible component primitives | Shadcn's Radix-based components handle keyboard nav and a11y without a library opinion on visual design; Tailwind v4 removes purge config entirely |
| **Biome** | Linting + formatting in a single fast tool | Replaces ESLint + Prettier with a single config, single tool, significantly faster CI — no tradeoff in rule coverage for this project's surface area |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Type Safety | TypeScript throughout + Sanity TypeGen | CMS query return types are generated, not handwritten — eliminates a class of runtime shape mismatches |
| Unit | Server Action logic isolated from Google API calls | Plan limit enforcement, timezone conversion, slot availability logic tested without live Google dependency |
| Integration | Google Calendar API calls tested against a dedicated test Google account | Full booking flow (create event, generate Meet link, send invite) validated end-to-end |
| Manual | Public booking page tested across timezones using browser locale overrides | Timezone conversion is the highest-risk user-facing logic; manual verification with known offset pairs |

**Explicitly not tested and why:** No automated E2E suite at v1 — the booking flow is linear and the Google Calendar integration test covers the critical path. Automated UI regression added when booking flow complexity grows beyond current linear structure.

---

## 📡 Observability

- **📝 Logs:** Structured logs per Server Action — action name, user ID (hashed), plan tier, Google API response code. OAuth refresh events logged explicitly with token age.
- **📊 Metrics:** Booking creation rate, Google API error rate (especially 401s indicating token refresh failures), plan limit hit rate per tier.
- **🔍 Tracing:** Request correlation ID propagated through Server Action → Google Calendar API call chain — full booking flow reconstructable from a single ID.
- **🚨 Alerting:** Google API 401 error rate spike (token refresh failure at scale) pages on-call. Booking creation error rate > 2% triggers investigation.

---

## 📊 Pricing Tiers

| Feature | Free | Starter | Pro |
|---------|------|---------|-----|
| Connected Calendars | 1 | 3 | Unlimited |
| Bookings per Month | 2 | 10 | Unlimited |
| Availability Management | ✅ | ✅ | ✅ |
| Google Calendar Sync | ✅ | ✅ | ✅ |
| Custom Booking Page | ✅ | ✅ | ✅ |
| Admin Dashboard | ❌ | ✅ | ✅ |
| Attendee Status Tracking | ❌ | ✅ | ✅ |

---

## 📊 Impact & Results

| Metric | Baseline | Result | Measurement Method |
|--------|----------|--------|--------------------|
| ⏱️ Booking page load (RSC) | N/A (new product) | < 900ms p95 | Vercel Analytics, server-rendered |
| ⚡ Time-to-booking-confirmation | N/A | < 3s end-to-end | Client perf marks, includes Google Calendar API round-trip |
| 🔄 OAuth token refresh success rate | N/A | > 99.5% | Server-side error logs, production traffic |
| 🌍 Timezone conversion accuracy | N/A | 100% on known offset pairs | Manual verification across 8 timezone pairs |

---

## 🚧 Challenges

**🔹 OAuth2 token lifecycle management**
Problem: Google OAuth access tokens expire after 1 hour. In a scheduling SaaS, a user who authenticates and returns the next day to check bookings hits an expired token — silently, with no UX indication, until a Calendar API call fails.
Tried: Catching 401s at the API call site and surfacing a re-auth prompt — this worked but created a jarring mid-flow interruption for the user.
Solution: Proactive refresh on every Server Action that touches the Google Calendar API — check token expiry before the call, refresh if within a 5-minute window, persist the new token, then proceed. Tradeoff: one additional DB read per Calendar API call, negligible at current scale, eliminates the error class entirely.

**🔹 Multi-calendar free/busy computation**
Problem: A host with 3 connected Google accounts has events spread across all three. Computing truly available slots means querying all accounts in parallel, merging their event sets, and subtracting from the host's defined availability blocks — without showing any individual account's events to the guest.
Tried: Sequential queries per account, which introduced visible latency on the booking page as account count grew.
Solution: Parallel `Promise.all` across all connected accounts, merge on the server, compute slots server-side before RSC render. Guest receives a clean list of available slots — no raw event data ever leaves the server. Tradeoff: increased Google API quota consumption per page load, acceptable given current user-level quota limits.

**🔹 Sanity TypeGen integration with dynamic queries**
Problem: Sanity's GROQ queries return `any` by default — every CMS data access was an untyped runtime risk. TypeGen generates types from queries, but only for queries it can statically analyze at build time.
Tried: Hand-writing Zod schemas for all CMS return shapes — accurate but brittle, requiring manual updates on every schema change.
Solution: Committed to Sanity TypeGen with strictly typed query functions — all GROQ queries live in a single `queries.ts` file, TypeGen runs as a pre-build step, generated types imported throughout. Schema changes automatically surface as TypeScript errors. Tradeoff: slightly more rigid query structure, eliminates an entire category of production shape mismatch bugs.

---

## 🔄 What I'd Do Differently

- **🔹 Google Calendar webhooks over polling:** Attendee status (accepted/declined/pending) is currently pulled on demand. Google Calendar push notifications (webhooks) would give real-time status updates without polling — the architecture supports this as a drop-in upgrade to the status-tracking module.
- **🔹 Separate availability engine from the booking flow:** Free/busy computation and slot generation are currently co-located with the booking Server Action. Extracting this into a dedicated availability service would allow independent caching, testing, and scaling — particularly important as multi-calendar query complexity grows with user account count.
- **🔹 Explicit rate limiting on Google API calls:** High-volume users (consultants with 50+ bookings/day) approach Google's per-user API quotas without any application-layer throttling. A token bucket per user-account pair should be added before scaling to that tier.

---

## 📸 Screenshots

| View | |
|------|-|
| 🖥️ Public Booking Page | ![Booking](./screenshots/booking-page.png) |
| 📅 Availability Management | ![Availability](./screenshots/availability.png) |
| 📊 Admin Dashboard | ![Dashboard](./screenshots/dashboard.png) |
| 💳 Pricing & Plan Gate | ![Pricing](./screenshots/pricing.png) |

---

## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Component breakdown, Google Calendar integration model, lazy deletion pattern |
| [API Reference](./api-reference/endpoints.md) | Server Action surface, auth requirements, plan limit enforcement |
| [OAuth Setup](./docs/oauth-setup.md) | Google Cloud Console config, credential scopes, token storage model |
