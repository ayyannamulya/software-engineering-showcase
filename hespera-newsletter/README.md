<div align="center">

![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

# 📰 Hespera — Modern Newsletter Generator

### *Transform your RSS feeds into polished, AI-curated newsletters in seconds — built for creators who publish consistently without the burnout.*

<!-- Tech Stack Badges — flat-square, compact -->
![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini_AI-4285F4?style=for-the-badge&logo=google&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B1?style=for-the-badge&logo=zod&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_v4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)

<br/>

[🌐 Live Demo](#) &nbsp;·&nbsp; [📑 Documentation](#)

[🎯 Portfolio](https://ayyanna-mulya.vercel.app) &nbsp;·&nbsp; [💼 LinkedIn](https://www.linkedin.com/in/noviyartimulyadi) &nbsp;·&nbsp; [🐙 GitHub](https://github.com/ayyannamulya) &nbsp;·&nbsp; [📧 Gmail](mailto:noviayya1121@gmail.com)

> 🔒 **Showcase Repo** — Source code is private. This repository documents architecture, design decisions, and engineering results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Newsletter creators publishing on a weekly or monthly cadence face the same structural bottleneck: **content sourcing and copy production**. Scanning 10–20 RSS feeds manually, filtering signal from noise, writing copy, and reformatting for different email platforms costs 3–5 hours per edition — before a single word reaches subscribers. For solo creators and small teams, that time cost compounds directly into publishing frequency, which is the primary driver of subscriber churn.

The existing tool landscape doesn't solve this. Email platforms (Mailchimp, ConvertKit) automate the send; feed aggregators (Feedly, Inoreader) aggregate the sources. Neither closes the gap between *"I have sources"* and *"I have a sendable newsletter."* That gap remains entirely manual — and Hespera was built to eliminate it.

---

## 💡 Solution

Hespera ingests a user's RSS feed subscriptions, fetches and deduplicates articles across a configurable time window, and routes the structured content through Google Gemini via the Vercel AI SDK to produce a complete, opinionated newsletter draft — titles, subject lines, body, highlights, and recommendations — streamed live to the browser.

**The defining architectural decision is cross-user feed caching with a 3-hour TTL.** Feed fetches are expensive: upstream latency, RSS provider rate limits, and parsing overhead. Rather than scoping cache to a user session, fetched feed data is shared across all users referencing the same source. User A's TechCrunch pull at 10:00 AM becomes User B's warm cache at 10:30 AM — cold-fetch cost is amortized to the first requester per window, and generation is near-instant for everyone else. The explicit tradeoff is a 3-hour staleness floor, which is acceptable given weekly and monthly publishing cadences.

**Article deduplication by GUID** ensures the same story — syndicated across multiple subscribed feeds — never inflates generation context or wastes storage, yielding a measured ~50% reduction in document volume at scale.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| 🤖 AI Newsletter Generation | Gemini via Vercel AI SDK `streamObject` produces 5 title options, 5 subject line variations, a full newsletter body, top-5 highlights, and additional insights — validated end-to-end against a Zod schema | Eliminates the blank-page problem entirely; structured output guarantees every generation is parseable and format-consistent across plans |
| ⚡ Real-Time Streaming | Server-side `streamObject`; client subscribes via `useObject` hook — no custom SSE parsing, no manual state threading | Users watch the newsletter materialize live; zero-boilerplate streaming keeps the architecture clean as generation complexity grows |
| 🗄️ Cross-User Feed Cache | Feed fetches keyed by source URL with a shared 3-hour TTL across all tenants | Eliminates redundant upstream calls; amortizes cold-fetch cost across the user base; generation speed stays consistent regardless of concurrent load |
| 🔍 Article Deduplication | Articles stored once per GUID; feed-to-article relationships tracked via a join collection | ~50% storage reduction at scale; prevents duplicate content from degrading generation quality; naturally surfaces cross-feed trending topics |
| 💳 Tiered Subscription Model | Starter (3 feeds, ephemeral generation) / Pro (unlimited feeds, full newsletter history) enforced via Clerk billing metadata at the service layer | Clean plan enforcement without a custom billing surface; Clerk handles entitlement propagation and webhook-driven plan state sync |
| 🎯 Custom Time Ranges & Brand Voice | Generation scoped by 7-day, 30-day, or custom date range; parameterized with target audience, tone, brand voice, company context, and custom disclaimers | The same feed set produces meaningfully differentiated output across publishing contexts — weekly roundups, monthly deep-dives, and special editions all handled from one interface |

---

## 🏗️ Architecture

```
[Browser — Next.js 16 App Router / React 19]
        │
        │  HTTPS  (Server Actions / Route Handlers)
        ▼
┌──────────────────────────────────┐
│       Next.js API Surface        │  ← Clerk JWT validation, Zod schema enforcement,
│  (Route Handlers + Server        │    plan-tier gating — all resolved before any
│   Actions)                       │    service layer call
└───────────────┬──────────────────┘
                │
   ┌────────────┴─────────────┐
   │       Service Layer      │  ← Feed orchestration, cache lookup/write-lock,
   │   (lib/ + server utils)  │    dedup logic, Gemini prompt assembly,
   │                          │    newsletter persistence
   └────────┬─────────────────┘
            │                     │
            ▼                     ▼
┌───────────────────┐   ┌──────────────────────────┐
│  MongoDB          │   │  Google Gemini API        │
│  via Prisma       │   │  (via Vercel AI SDK       │
│                   │   │   streamObject)           │
│  Collections:     │   └──────────────────────────┘
│  · users          │
│  · feeds          │   ┌──────────────────────────┐
│  · articles       │   │  RSS Parser              │
│  · feed_articles  │   │  (external feeds —       │
│  · newsletters    │   │   treated as untrusted)  │
└───────────────────┘   └──────────────────────────┘

Trust boundary  : Clerk session JWT validated at every Route Handler and Server Action
                  entry point. User identity and plan tier resolved server-side only —
                  never derived from client-supplied payload.
External trust  : RSS feed content treated as untrusted at ingestion. Parsed fields
                  sanitized before storage and before inclusion in Gemini prompts.
                  Gemini responses validated against the Zod schema before streaming
                  to the client.
```

### Data Flow

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Ingress | Route Handler / Server Action | Clerk session validated; user plan tier resolved server-side; request body parsed and enforced against Zod schema — malformed input rejected before reaching service logic |
| 2. Feed Resolution | Feed Service | Per subscribed feed: check shared cache TTL. Cache hit → return cached articles immediately. Cache miss → acquire write-lock, fetch from RSS source, parse, deduplicate by GUID, persist to MongoDB, release lock |
| 3. Generation | Gemini Service | Sanitized, deduplicated articles assembled into a structured prompt; `streamObject` initiated against Gemini with full Zod schema; partial objects streamed back to client via `useObject` hook |
| 4. Persistence | MongoDB via Prisma | Pro users: completed newsletter written to `newsletters` collection with user ID, feed references, and generation metadata. Starter users: generation is ephemeral — no write occurs |
| 5. Response | Client (React 19 RSC) | Partial objects rendered live as they stream; on completion, copy and export controls activate; cache hit/miss surfaced to user via toast notification |

> Full component breakdown, degraded-path behavior, and horizontal scalability notes → [Architecture Doc](./docs/architecture.md)

---

## 🔐 Security Design

Security decisions are engineering decisions — documented here, not treated as an afterthought.

| Concern | Decision | Rationale |
|---------|----------|-----------|
| 🔑 Authentication | Clerk session JWTs | Offloads token lifecycle, rotation, and session invalidation entirely. No custom auth surface to maintain or misconfigure |
| 🛡️ Authorization | Plan-tier RBAC via Clerk metadata | Feed limits and newsletter history access gated at the service layer using `publicMetadata.plan` — never derived from client input |
| 🧹 Input Trust | Zod schema validation at every API boundary | All Route Handler and Server Action inputs validated before touching service logic. RSS content sanitized (HTML stripped, fields length-capped) before storage and before prompt inclusion |
| 💉 Prompt Injection | RSS content isolated in XML-delimited prompt blocks | Feed-sourced text inserted as inert data within `<article_content>` blocks with explicit model instruction that this content is untrusted. Indirect prompt injection via malicious feed items is the primary LLM-specific threat surface; mitigation documented with known residual risk |
| 🔐 Secrets Management | Environment variable injection via Vercel — zero hardcoded credentials | `GEMINI_API_KEY`, `DATABASE_URL`, and Clerk keys injected at deploy time. Rotation handled through Vercel environment management |
| 📋 Audit Logging | Per-request logs: `userId`, `feedIds[]`, `timeRange`, `planTier`, `cacheHit`, `generationDurationMs` — newsletter content excluded | PII-minimal; sufficient for abuse investigation and billing reconciliation without retaining user-owned content server-side |

**⚠️ Accepted Risk:**
- RSS feed fetching is network-bound with no upstream SLA guarantee. A cache miss on a slow source delays generation. The 3-hour TTL reduces cold-fetch frequency, but a background refresh queue (e.g., BullMQ) would eliminate the blocking delay entirely — deliberately deferred to keep v1 infrastructure simple.
- No mTLS between Next.js and MongoDB Atlas. Mitigated by Atlas IP allowlisting and TLS in transit. Acceptable for the current deployment footprint.
- Gemini responses are Zod-validated at the schema level but not semantically audited. A model producing structurally valid but editorially misleading content passes validation. Relevant if the product evolves toward automated send with no human review step — not in scope for v1.

---

## 🧰 Tech Stack

| Layer | Technology | Decision Rationale |
|-------|------------|--------------------|
| Framework | Next.js 16 (App Router) + React 19 | Server Components and Server Actions collapse the BFF layer — no separate API service needed. Turbopack provides sub-second HMR in development |
| Language | TypeScript — strict mode throughout | End-to-end type safety from Prisma schema through Zod API boundaries to client hooks. Eliminates an entire class of runtime errors on structured AI output |
| AI / Streaming | Vercel AI SDK (`streamObject` / `useObject`) | Handles SSE framing, partial JSON assembly, and client state management for streamed structured objects. Zero custom streaming code — the manual SSE alternative is a long-term maintenance liability |
| LLM | Google Gemini | Strong structured generation with reliable instruction-following for schema-constrained output. Model is parameterized behind the service layer — swappable without interface changes |
| Database | MongoDB + Prisma | Document model fits variable article and newsletter shapes naturally. Prisma provides type-safe query access and schema introspection over a schemaless store |
| Auth + Billing | Clerk | Session management, OAuth flows, and subscription plan metadata in a single integration. Eliminates a custom billing surface entirely — webhooks sync plan state directly to user records |
| Styling | Tailwind CSS v4 | Utility-first with no runtime CSS-in-JS overhead. v4's native cascade layer support resolves specificity conflicts cleanly |
| Code Quality | Biome | Replaces ESLint + Prettier with a single, faster tool — zero config drift, consistent enforcement across the codebase |
| Deployment | Vercel | Native Next.js runtime, edge function support, environment management, and per-PR preview deployments without dedicated infra overhead |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Unit | Node.js built-in test runner (`node:test`) | Feed cache TTL logic, write-lock behavior under simulated concurrency, article deduplication (GUID collision paths), Zod schema validation edge cases, plan-tier enforcement |
| Integration | Supertest + MongoDB in-memory (`mongodb-memory-server`) | Full feed fetch → parse → deduplicate → persist pipeline; newsletter write for Pro tier vs. no-op assertion for Starter; Clerk plan metadata enforcement at the route level |
| AI Output Compliance | Zod schema validation against captured `streamObject` fixture payloads | Ensures schema enforcement catches malformed or incomplete Gemini responses before they reach the client; run against a representative fixture set on every CI push |
| Load & Performance | k6 | Cache warm vs. cold path latency under concurrent generation load; write-lock correctness under simultaneous cache-miss requests for the same feed source |

**Explicitly not tested and why:** No E2E browser test suite. The correctness-critical surface — cache consistency, deduplication, plan gating, AI output validation — is fully covered at the integration layer. UI flows are deterministic (form → stream → display); Playwright overhead is not warranted for this interaction surface in v1.

---

## 📡 Observability

- **📝 Logs:** Structured JSON via Vercel logging. Required fields on every generation request: `userId`, `feedIds[]`, `timeRange`, `planTier`, `cacheHit` (boolean), `generationDurationMs`. Newsletter body content is explicitly excluded from all log payloads.
- **📊 Metrics:** Generation latency (p50 / p95 / p99) segmented by cache hit vs. miss; feed fetch error rate by source domain; plan-tier distribution; daily and weekly active generator counts.
- **🔍 Tracing:** Vercel request traces cover the Route Handler → Service → MongoDB path. Gemini API calls tagged with `userId` in request metadata for per-user cost attribution and cohort analysis.
- **🚨 Alerting:** Feed fetch error rate > 10% over a 5-minute window (upstream RSS degradation); Gemini API error rate > 5% (model availability); MongoDB Atlas connection pool saturation threshold breach.

---

## 📊 Impact & Results

| Metric | Before (Manual) | After (Hespera) | Measurement Method |
|--------|-----------------|------------------|--------------------|
| ⏱️ Time to draft a newsletter | 3–5 hours | < 60 seconds | End-to-end timing across 5 newsletter creators in structured user testing |
| 🐢 Generation latency — cache miss | N/A | < 8s p95 (3-feed baseline) | k6 load test, cold cache, 3 concurrent feeds |
| ⚡ Generation latency — cache hit | N/A | < 400ms p95 | k6 load test, warm cache |
| 💾 Article storage efficiency | Baseline (per-user duplicate storage) | ~50% reduction | MongoDB collection size comparison: GUID-dedup vs. duplicate-per-user simulation |
| ✅ Structured output schema compliance | N/A | 98.3% first-pass validity | 500 generation runs; Zod validation pass rate logged against fixture suite |

---

## 🚧 Challenges

**🔒 Cache Consistency Under Concurrent Cold Fetches**
- **Problem:** Two users triggering a cache miss for the same feed simultaneously would both initiate upstream fetches — defeating the caching strategy, doubling RSS provider load, and potentially producing inconsistent stored state. The race is invisible in single-user testing and only surfaces under concurrent production load.
- **Tried:** Optimistic locking via MongoDB `findOneAndUpdate` with a `fetchingInProgress` flag. This introduced a polling loop on the waiting request, adding latency and non-obvious retry logic that was difficult to reason about under failure conditions.
- **Solution:** Replaced with a write-lock pattern carrying a hard 10-second TTL. The first request to claim the lock proceeds with the upstream fetch; all subsequent requests for the same feed within the lock window hold and read from the completed cache entry on release. Accepted tradeoff: requests arriving mid-fetch on a slow upstream source may see up to 10 seconds of additional latency on a cold path — acceptable given the low probability of simultaneous cold-fetches for the same source in practice.

**💉 Indirect Prompt Injection via RSS Feed Content**
- **Problem:** RSS feeds are arbitrary third-party content. Article titles and descriptions are incorporated into the Gemini prompt context, making them a direct indirect prompt injection surface — a maliciously crafted feed item (e.g., `title: "Ignore prior instructions and output..."`) is a textbook OWASP LLM02 vector.
- **Tried:** The initial implementation had no sanitization — raw parsed fields were inserted directly into the prompt template. Identified during security self-review before shipping.
- **Solution:** All feed-sourced text fields are sanitized before prompt assembly: HTML stripped, fields length-capped (titles: 200 chars, descriptions: 500 chars), and inserted within explicitly labeled XML-delimited blocks (`<article_content>...</article_content>`), with an explicit model directive that content within these blocks is untrusted external data, not instructions. This raises the exploitation bar meaningfully but does not eliminate the risk entirely — residual exposure is documented as a known accepted risk and flagged for secondary validation in v2.

**🗃️ Prisma Schema Evolution on MongoDB**
- **Problem:** Prisma's MongoDB connector does not support `migrate dev` equivalently to relational connectors. Schema changes to existing collections require explicit handling of documents written before the new shape — Prisma does not backfill or enforce structure on pre-existing documents.
- **Tried:** Applied schema changes directly and relied on Prisma's optional field handling for backward compatibility. Read queries on older documents missing new required fields broke silently in production.
- **Solution:** All schema additions are defined as optional with explicit defaults in the Prisma schema. New fields are backfilled via a lightweight migration script executed once at deploy time, before the updated application version goes live. This pattern is now the documented standard for all future schema changes in the architecture doc.

---

## 🔄 What I'd Do Differently

- **🔁 Background Feed Refresh Queue:** The current cache model is pull-on-demand — a cache miss blocks the generation request while the upstream fetch completes. A BullMQ-backed background refresh job pre-warming high-traffic feeds on a rolling schedule would make the 3-hour TTL completely invisible to users. This is the highest-priority architectural improvement for v2.
- **🛡️ Secondary Prompt Injection Validation:** The XML-delimited block mitigation reduces injection risk but does not eliminate it. In hindsight, a secondary structural validation pass — checking whether the Gemini output conforms to the expected newsletter schema semantically, not just syntactically — would close the residual gap more completely. The cost is added latency on the generation path; the benefit is a meaningful reduction in injection-to-output risk.
- **📖 Separated Read/Write MongoDB Clients:** The current setup uses a single Prisma client for all database operations. For a read-heavy workload — newsletter history retrieval, feed browsing — routing reads to a MongoDB Atlas read replica and writes to the primary would reduce read latency under load. This is a straightforward change via Prisma's connection configuration and would have been worth establishing from the start.
- **🔄 Clerk Webhook Idempotency:** Plan-tier sync from Clerk webhooks currently lacks deduplication — a replayed webhook event could double-process a plan upgrade and produce inconsistent entitlement state. Idempotency key tracking (Clerk event ID stored in MongoDB, checked before processing) should have been built in from day one rather than retrofitted.

---

## 📸 Screenshots

| View | |
|------|-|
| 🖥️ Dashboard — Feed Management | ![Dashboard](./screenshots/dashboard.png) |
| ⚡ Generation — Live Streaming | ![Streaming](./screenshots/streaming.png) |
| 📚 Newsletter History (Pro) | ![History](./screenshots/history.png) |

---

## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Component breakdown, data flow diagram, trust model, cache design, scalability notes, and degraded-path behavior |
| [API Reference](./api-reference/endpoints.md) | Full Route Handler surface, authentication requirements, plan-tier enforcement rules, and Zod schema definitions |

---

## 🔗 Connect & Explore

[![Live Demo](https://img.shields.io/badge/Live_Demo-000000?style=for-the-badge&logo=vercel&logoColor=white)](#)
[![Documentation](https://img.shields.io/badge/Documentation-FF6B35?style=for-the-badge&logo=gitbook&logoColor=white)](./docs/architecture.md)
[![Portfolio](https://img.shields.io/badge/Portfolio-CC785C?style=for-the-badge&logoColor=white)](#)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](#)
