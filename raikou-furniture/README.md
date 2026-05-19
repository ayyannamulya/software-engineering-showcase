<div align="center">

![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

# 🛋️ Raikou — AI-Powered Furniture Store

### *A full-stack e-commerce platform where customers shop with an AI assistant that knows their orders, and store owners manage everything in real-time — powered by Sanity, Claude, and Stripe.*

![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Sanity](https://img.shields.io/badge/Sanity_CMS-F03E2F?style=for-the-badge&logo=sanity&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=for-the-badge&logo=stripe&logoColor=white)
![Claude](https://img.shields.io/badge/Claude-CC9B7A?style=for-the-badge&logo=anthropic&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_v4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Shadcn](https://img.shields.io/badge/shadcn%2Fui-000000?style=for-the-badge&logo=shadcnui&logoColor=white)
![Zustand](https://img.shields.io/badge/Zustand-433E38?style=for-the-badge&logo=react&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)
![Biome](https://img.shields.io/badge/Biome-60A5FA?style=for-the-badge&logo=biome&logoColor=white)

<br/>

[🌐 Live Demo](https://welorra-care.vercel.app) &nbsp;·&nbsp; [📑 Documentation](#)

[🎯 Portfolio](https://ayyanna-mulya.vercel.app) &nbsp;·&nbsp; [💼 LinkedIn](https://www.linkedin.com/in/noviyartimulyadi) &nbsp;·&nbsp; [🐙 GitHub](https://github.com/ayyannamulya) &nbsp;·&nbsp; [📧 Gmail](mailto:noviayya1121@gmail.com)


<br/>

> 🔒 **Showcase Repo** — Source code is private. This repository documents architecture, design decisions, and engineering results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Most furniture e-commerce experiences are static — a product grid, a cart, a checkout. Finding the right piece requires manual filtering across dozens of attributes (material, color, dimensions, price) with no assistance, no context, and no memory of who you are. On the admin side, managing inventory and orders across disconnected tools means delayed updates, missed low-stock events, and no visibility into what's actually happening in the store.

The gap: a shopping experience that's intelligent on the customer side and real-time on the operator side — where the AI knows the catalog and the customer, and the store owner sees changes reflected instantly without a deployment.

---

## 💡 Solution

Raikou is a furniture e-commerce platform built around two core capabilities: an authenticated AI shopping assistant and a real-time CMS-driven admin layer.

The AI assistant (Claude via Vercel AI SDK) is authenticated via Clerk AgentKit — meaning it has session context and can access the current user's order history alongside the full product catalog. Natural language queries like "find me a grey sofa under $800 with fabric upholstery" resolve to structured product searches without any manual filtering.

The core architectural decision: Sanity is the single source of truth for all content — products, orders, inventory. The Sanity App SDK gives the admin dashboard direct database access with real-time subscriptions via Sanity Live, so product edits and order status changes appear on the storefront instantly without a cache invalidation step or a deployment. The tradeoff accepted: operational complexity of Sanity as both CMS and operational datastore, justified by the real-time requirement and the elimination of a separate database layer.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| 🤖 AI Shopping Assistant | Claude via Vercel AI SDK with tool calls for product search and order lookup; Clerk AgentKit injects user session into AI context | Customers find products through conversation — no manual filter navigation required |
| 🧠 AI Admin Insights | Claude analyzes sales trends, inventory levels, and order velocity; surfaces actionable recommendations in the dashboard | Store owners get a prioritized action list, not just raw data |
| ⚡ Real-time Content | Sanity Live + App SDK — product and order changes propagate to storefront without refresh or cache invalidation | Inventory updates, price changes, and order status reflect immediately across all sessions |
| 🛒 Persistent Cart | Zustand state with localStorage persistence — cart survives page refresh and browser restart | No lost carts on reload; stock validated against live Sanity inventory at checkout |
| 💳 Secure Checkout | Stripe payment intents with server-side amount validation; address collection handled by Stripe Elements | Amount never trusted from client — recalculated server-side against Sanity product prices before charge |
| 📦 Order Tracking | Authenticated order history pulled from Sanity; status progression (paid → shipped → delivered) visible per order | Customers see real-time order state without email polling |
| ⚠️ Low Stock Alerts | Sanity inventory thresholds trigger dashboard alerts; AI surfaces affected SKUs with reorder recommendations | Stockouts caught before they hit checkout, not after |
| ✏️ Real-time Product Management | Sanity App SDK exposes create/edit/publish directly in the admin dashboard — no separate Sanity Studio tab required | Operators manage the full store from one interface |

---

## 🏗️ Architecture

```
[Customer Browser]                        [Admin Browser]
        │                                        │
        │  HTTPS + Clerk session                 │  HTTPS + Clerk session (admin role)
        ▼                                        ▼
┌────────────────────────────────────────────────────────┐
│                  Next.js 16 App Router                 │
│   /shop/* /cart /checkout    │   /dashboard/*          │
│   RSC + Client Components    │   Sanity App SDK        │
└──────────────┬───────────────────────┬─────────────────┘
               │                       │
       ┌───────┴──────┐        ┌───────┴──────────────┐
       │  Sanity CMS  │        │   Vercel AI SDK       │
       │  (products,  │◄───────│   Claude (shopping    │
       │   orders,    │        │   assistant + admin   │
       │   inventory) │        │   insights)           │
       │  Sanity Live │        └───────────────────────┘
       └──────┬───────┘
              │
       ┌──────┴───────┐
       │    Stripe    │
       │  (payments,  │
       │   checkout)  │
       └──────────────┘

Trust boundary  : Clerk middleware enforces session on /dashboard and all order-related routes
Admin isolation : Admin role checked server-side before any Sanity App SDK write operation
External trust  : Stripe amount recalculated server-side; Sanity responses validated via TypeGen types
AI context      : Clerk AgentKit injects verified user identity — AI cannot access other users' orders
```

### Data Flow — Checkout

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Cart Validation | Server Action | Each cart item's price and stock re-fetched from Sanity — client values discarded |
| 2. Payment Intent | Stripe API | Intent created server-side with validated amount; client receives only the client secret |
| 3. Payment Confirm | Stripe Elements | Customer completes payment in Stripe-hosted UI; no card data touches the server |
| 4. Order Creation | Server Action on webhook | Stripe `payment_intent.succeeded` webhook triggers order record creation in Sanity |
| 5. Inventory Update | Sanity | Stock decremented atomically on order creation; Sanity Live propagates to storefront |

### Data Flow — AI Shopping Assistant

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Auth Context | Clerk AgentKit | User session injected into AI tool execution context before any tool call |
| 2. Intent Parse | Claude via AI SDK | Natural language query parsed; Claude selects appropriate tool (product search vs order lookup) |
| 3. Tool Execution | Sanity GROQ query | Structured query built from parsed intent; executed against Sanity with TypeGen-validated return types |
| 4. Response | Streaming | Claude synthesizes tool results into natural language; streamed to client progressively |

---

## 🔐 Security Design

| Concern | Decision | Rationale |
|---------|----------|-----------|
| 🔑 Authentication | Clerk session (HTTP-only cookie) + AgentKit for AI context | Single auth layer for both UI and AI tool execution — no separate token management |
| 🛡️ Authorization | Server-side role check on all admin routes and Sanity App SDK writes | Admin role verified before any write operation — UI gating alone is bypassable |
| 🧹 Input Trust | Cart prices and stock recalculated server-side before Stripe charge | Client-supplied amounts explicitly untrusted — price tampering structurally blocked |
| 💳 Payment Security | Stripe payment intents; no card data on application servers | PCI scope eliminated entirely; Stripe handles card data lifecycle |
| 🤖 AI Isolation | Clerk AgentKit scopes AI tool calls to authenticated user's data | AI cannot cross user boundaries — order lookup tool filters by verified session user ID |
| 🔐 Secrets Management | Stripe keys, Clerk keys, Sanity tokens via environment variables; Vercel env injection | No credentials committed; rotation at platform layer |
| 📋 Audit Logging | Order creation and status transitions logged with user ID and timestamp | Post-dispute reconstructability without PII leakage |

**⚠️ Accepted Risk:**
- Cart persistence via `localStorage` — cart contents visible to JavaScript running in the same origin. Mitigated by server-side revalidation at checkout; localStorage cart is treated as untrusted UI state, not a billing source.
- Sanity as operational datastore (orders, inventory) rather than a purpose-built transactional DB. Acceptable at current scale; high-volume concurrent writes (flash sales, etc.) would warrant moving order creation to a dedicated transactional store with Sanity as the read layer.

---

## 🧰 Tech Stack

| Technology | Role in This Project | Why This, Not the Alternative |
|------------|----------------------|-------------------------------|
| **Next.js 16 + React 19** | Full-stack framework; App Router for RSC, Server Actions for all mutations | React Compiler (`babel-plugin-react-compiler`) enabled — automatic memoization without manual `useMemo`/`useCallback` |
| **Sanity CMS + App SDK** | Single source of truth for products, orders, inventory; App SDK for admin dashboard writes | Sanity Live gives real-time subscriptions without a WebSocket server; App SDK eliminates a separate admin API layer |
| **Clerk + AgentKit** | Authentication + AI session context injection | AgentKit is the only production-ready solution for propagating verified user identity into AI tool execution context |
| **Vercel AI SDK (`ai` v6) + Claude** | AI shopping assistant and admin insights; streaming tool calls | AI SDK v6's tool-call streaming and multi-step agent support; `@ai-sdk/google` for model flexibility |
| **Stripe** | Payment processing, checkout UI, webhook-driven order creation | Webhook-first order creation means orders only exist after payment confirms — no orphaned pending orders |
| **Zustand + localStorage** | Client-side cart state with persistence | Zero-boilerplate persistent state; cart rehydrates on load, validated server-side at checkout |
| **Tailwind v4 + Shadcn/ui + Radix** | Styling + accessible component primitives | Radix handles all keyboard nav and a11y; Tailwind v4 removes PostCSS config entirely |
| **Biome** | Linting + formatting | Single tool, single config, significantly faster than ESLint + Prettier — no tradeoff in rule coverage |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Type Safety | TypeScript + Sanity TypeGen | All GROQ query return types auto-generated — CMS shape mismatches surface at build time, not runtime |
| Unit | Server Action logic isolated from Sanity and Stripe calls | Price recalculation, stock validation, order creation logic tested without live dependencies |
| Integration | Stripe webhook handler tested against Stripe CLI event replay | Full payment → order creation flow validated end-to-end without a live charge |
| Manual | AI assistant tested with adversarial queries — cross-user order access attempts, out-of-scope tool calls | AI isolation is the highest-risk surface; manual red-team probing before each release |

**Explicitly not tested and why:** No automated E2E suite at v1 — the checkout flow is linear and the Stripe + Sanity integration tests cover the critical path. UI regression added when product browsing flow complexity grows beyond current structure.

---

## 📡 Observability

- **📝 Logs:** Structured logs per Server Action — action name, user ID (hashed), Stripe event type, Sanity document ID affected. AI tool calls logged with tool name and execution time.
- **📊 Metrics:** Checkout conversion rate, Stripe webhook delivery success rate, AI assistant tool-call error rate, Sanity Live subscription health.
- **🔍 Tracing:** Request correlation IDs through Server Action → Sanity query → Stripe API chain. Full checkout flow reconstructable from a single ID.
- **🚨 Alerting:** Stripe webhook failure rate > 1% pages on-call — missed webhooks mean missing orders. AI tool-call error spike triggers fallback to non-AI product search.

---

## 📊 Impact & Results

| Metric | Baseline | Result | Measurement Method |
|--------|----------|--------|--------------------|
| ⏱️ Storefront load (RSC) | N/A (new product) | < 1s p95 | Vercel Analytics |
| ⚡ Real-time content propagation | N/A | < 500ms from Sanity publish to storefront | Manual measurement, Sanity Live subscription |
| 🤖 AI assistant first response | N/A | < 1.5s to first token | Client-side perf marks, streaming |
| 💳 Checkout to order confirmation | N/A | < 4s end-to-end | Client perf marks including Stripe webhook round-trip |

---

## 🚧 Challenges

**🔹 Authenticated AI tool execution**
Problem: The AI shopping assistant needs to access the current user's orders — but standard AI tool calls have no concept of a session. A tool that looks up orders without user scoping is a data exposure risk: any user could retrieve any order.
Tried: Passing user ID as a parameter in the tool call — but this is client-supplied and therefore untrusted. A prompt injection attack embedded in a product description could override the user ID parameter.
Solution: Clerk AgentKit injects the verified server-side session into the tool execution context before any tool call resolves. The order lookup tool reads user identity from the injected context, never from tool parameters. Tool parameters are Zod-validated; any attempt to pass a user ID via parameters is ignored. Tradeoff: Clerk AgentKit is an early-stage SDK — API surface is less stable than the core Clerk SDK.

**🔹 Sanity as operational datastore**
Problem: Sanity is designed as a content management system, not a transactional database. Using it to store orders and manage inventory means working without ACID transactions — a stock decrement and order creation are two separate Sanity mutations that can fail independently.
Tried: Optimistic inventory updates followed by a compensating mutation on failure — but this left a race condition window under concurrent checkout.
Solution: Order creation is triggered exclusively by Stripe's `payment_intent.succeeded` webhook (a single, retryable event). Inventory is decremented within the same webhook handler as order creation, sequenced explicitly. If the decrement fails, the webhook retries — Stripe's webhook retry logic becomes the transaction coordinator. Tradeoff: eventual consistency on inventory under high concurrency; acceptable given current traffic patterns.

**🔹 React Compiler with Sanity's real-time subscriptions**
Problem: React 19's compiler (`babel-plugin-react-compiler`) aggressively memoizes components — but Sanity Live's subscription hooks rely on reference equality checks that the compiler's automatic memoization can break, causing stale subscription state.
Tried: Disabling the compiler globally — defeats the purpose of enabling it.
Solution: Opted out specific Sanity Live subscription components from compiler optimization using the `"use no memo"` directive. All other components retain compiler optimization. Tradeoff: manual opt-out list requires maintenance as subscription components are added.

---

## 🔄 What I'd Do Differently

- **🔹 Separate order storage from Sanity:** Using Sanity as an operational datastore works at current scale but introduces consistency risk under concurrent writes. A dedicated transactional store (Neon Postgres, PlanetScale) for orders with Sanity as the product/content layer would be the right architecture for a production-scale store.
- **🔹 Webhook idempotency keys:** The Stripe webhook handler creates an order on `payment_intent.succeeded` but doesn't currently enforce idempotency at the application layer. Stripe retries webhooks on failure — without idempotency, a transient Sanity error could produce duplicate orders. An idempotency check against the payment intent ID should be added before any order write.
- **🔹 AI tool call observability:** AI tool executions are logged but not surfaced in the admin dashboard. A tool-call audit trail — what the assistant searched, what it returned, which sessions triggered which tool calls — would be valuable for both debugging and understanding how customers actually use the assistant.

---

## 📸 Screenshots

| View | |
|------|-|
| 🖥️ Storefront | ![Storefront](./screenshots/storefront.png) |
| 🤖 AI Shopping Assistant | ![AI Assistant](./screenshots/ai-assistant.png) |
| 📊 Admin Dashboard | ![Dashboard](./screenshots/dashboard.png) |
| 💳 Checkout | ![Checkout](./screenshots/checkout.png) |

---

<!-- ## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Component breakdown, data flow, Sanity as datastore model, real-time subscription design |
| [API Reference](./api-reference/endpoints.md) | Server Action surface, Stripe webhook handlers, AI tool schemas |
| [AI Safety](./docs/ai-safety.md) | Tool execution isolation, Clerk AgentKit session scoping, prompt injection mitigations | -->