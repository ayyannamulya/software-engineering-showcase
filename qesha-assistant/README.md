<div align="center">
          
![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

# 🧠 Qesha — Second-Brain AI Agent

### *A full-stack AI-powered knowledge agent that searches the web, extracts and summarizes content, chains tools intelligently — built for developers who think faster than they type.*

<!-- Tech Stack Badges -->
![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![Shadcn](https://img.shields.io/badge/shadcn%2Fui-000000?style=flat-square&logo=shadcnui&logoColor=white)
![Hono](https://img.shields.io/badge/Hono-E36002?style=flat-square&logo=hono&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=flat-square&logo=stripe&logoColor=white)
![Anthropic](https://img.shields.io/badge/Claude-CC9B7A?style=flat-square&logo=anthropic&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B1?style=flat-square&logo=zod&logoColor=white)
![TanStack](https://img.shields.io/badge/TanStack_Query-FF4154?style=flat-square&logo=reactquery&logoColor=white)
![Zustand](https://img.shields.io/badge/Zustand-433E38?style=flat-square&logo=react&logoColor=white)
![BetterAuth](https://img.shields.io/badge/BetterAuth-1a1a1a?style=flat-square&logo=auth0&logoColor=white)
![Tavily](https://img.shields.io/badge/Tavily-00C7B7?style=flat-square&logo=searchengin&logoColor=white)

<br/>

[🌐 Live Demo](#) &nbsp;·&nbsp; [📑 Documentation](#)

[🎯 Portfolio](https://ayyanna-mulya.vercel.app) &nbsp;·&nbsp; [💼 LinkedIn](https://www.linkedin.com/in/noviyartimulyadi) &nbsp;·&nbsp; [🐙 GitHub](https://github.com/ayyannamulya) &nbsp;·&nbsp; [📧 Gmail](mailto:noviayya1121@gmail.com)

<br/>

> 🔒 **Showcase Repo** — Source code is private. This repository documents architecture, design decisions, and engineering results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Knowledge workers drown in tabs, bookmarks, and browser history — raw information captured but never made actionable. Existing tools either store passively (Notion, Obsidian) or retrieve broadly (web search) but nothing closes the loop: ingest a URL, understand it, relate it to what you already know, and surface it when you need it.

The gap isn't storage. It's synthesis — the ability to chain: *find a thing → understand it → connect it to existing knowledge → act on it* — without leaving a single interface or switching between five tools.

---

## 💡 Solution

Qesha is an agentic second-brain built on the Vercel AI SDK's tool-chaining primitives. The core architectural decision: expose atomic tools — web search, URL extraction, create-note, find-note — to a reasoning agent rather than hardcoding workflows. The agent composes them at runtime based on intent.

This means a single natural language prompt like *"summarize what this article says about RAG poisoning and save it to my security notes"* executes a multi-step tool chain without any explicit routing logic. The tradeoff accepted: non-deterministic execution paths, mitigated by streaming tool-call transparency so users always see what the agent is doing and why.

Authentication is handled by BetterAuth, subscriptions by Stripe with per-tier feature gating, and the entire stack deploys to Vercel Edge runtime.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| 🔍 Web Search | Tavily API integration exposed as an agent tool | Grounded, real-time answers — not hallucinated retrieval |
| 🔗 URL Extraction & Summarization | Agent calls a URL-fetch tool, passes content to Claude/Grok for summarization | Converts any URL into structured knowledge in one prompt |
| 🛠️ Tool Chaining | Vercel AI SDK's `streamText` with multi-tool execution across turns | Complex tasks (search → summarize → save) run without user orchestration |
| 📝 Create & Find | `create` and `find` tools backed by Prisma + Neon Postgres | Persistent memory that the agent can read and write to on your behalf |
| 🧠 Multi-Model Backend | `@ai-sdk/gateway` routing to Claude (Anthropic) and Grok | Model diversity without SDK lock-in; switch per task or tier |
| 🔐 Auth with BetterAuth | Session-based auth with social providers | Secure, extensible auth without rolling custom JWT logic |
| 💳 Stripe Subscriptions | `@better-auth/stripe` plugin — plan gating at middleware layer | Usage tiers enforced before requests reach the agent, not inside it |
| 🧪 AI-Powered Testing | TestSprite agent runs automated test suites against the live UI | Regression coverage that scales with the interface, not headcount |
| 📱 Responsive UI | Tailwind v4 + Shadcn/ui component system | Zero CSS drift between mobile and desktop; Radix primitives handle a11y |

---

## 🏗️ Architecture

```
[Browser — Next.js 15 / React 19]
          │
          │  HTTPS / Server Actions / Streaming (RSC)
          ▼
┌──────────────────────────┐
│     Next.js App Router   │  ← Route handlers, middleware auth check (BetterAuth)
│     + Hono API Layer     │  ← Typed REST endpoints with Zod validation
└────────────┬─────────────┘
             │
┌────────────┴─────────────┐
│   Vercel AI SDK Agent    │  ← streamText() with tool registry
│   Tool Registry:         │
│   • web_search (Tavily)  │
│   • extract_url          │
│   • create_note          │
│   • find_note            │
└────┬──────────┬──────────┘
     │          │
┌────┴────┐  ┌──┴─────────────────┐
│  Neon   │  │  AI Model Gateway  │
│Postgres │  │  Claude / Grok     │
│(Prisma) │  │  (@ai-sdk/gateway) │
└─────────┘  └────────────────────┘

Trust boundary  : BetterAuth middleware enforces session before any tool call reaches the agent
External trust  : Tavily responses and model completions treated as untrusted — structured via Zod schemas before use
Subscription gate: Stripe plan checked at middleware layer; feature flags propagate inward via session context
```

### Data Flow

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Ingress | Next.js Middleware | BetterAuth session check — unauthenticated requests rejected before routing |
| 2. Plan Gate | Middleware | Stripe subscription tier checked; feature flags injected into session context |
| 3. Request | Hono API / Route Handler | Zod schema validation on request body — malformed payloads rejected before agent |
| 4. Agent Execution | Vercel AI SDK `streamText` | Agent reasons over available tools, calls them in sequence, streams intermediate state |
| 5. Tool Calls | Tool Registry | Each tool (Tavily, Prisma, URL fetcher) executes with scoped credentials; responses validated |
| 6. Persistence | Neon Postgres via Prisma | Notes written/read with user-scoped queries — no cross-tenant data access |
| 7. Stream Response | React Server / `useChat` | Tool-call events + final text streamed to client; UI renders progressively |

---

## 🔐 Security Design

| Concern | Decision | Rationale |
|---------|----------|-----------|
| 🔑 Authentication | BetterAuth session tokens (HTTP-only cookie) | Eliminates token exposure in localStorage; aligns with Next.js middleware interception pattern |
| 🛡️ Authorization | Session-scoped Prisma queries + Stripe plan middleware | Subscription tier enforced at the API boundary, not inside agent logic — agent never sees plans |
| 🧹 Input Trust | Zod validation on all Hono endpoints; tool responses re-validated before agent consumption | Tool outputs (Tavily, URL content) are treated as untrusted environmental input — adversarial content in a fetched URL cannot influence agent system context |
| 🔐 Secrets Management | Environment variables only — no hardcoded keys; Vercel env injection for runtime | Stripe, Anthropic, Tavily keys never committed; rotation handled at platform layer |
| 📋 Audit Logging | Tool-call events streamed to client; server-side structured logs per request | Post-incident reconstructability of agent execution path without PII leakage |
| 🧠 Indirect Prompt Injection | URL content passed to model as scoped `user` turn, not prepended to system prompt | Prevents adversarial web content from hijacking agent instructions or escalating tool scope |

**⚠️ Accepted Risk:**
- Tool-call parameters flow through the model's reasoning — a sufficiently adversarial document could influence tool arguments. Mitigated by Zod schema enforcement on all tool inputs, rejecting out-of-range or structurally unexpected values before execution.
- No mTLS between internal services (Next.js → Neon). Mitigated by Neon's private connection pooling and Prisma's parameterized queries (no raw SQL surface).

---

## 🧰 Tech Stack

| Technology | Role in This Project | Why This, Not the Alternative |
|------------|----------------------|-------------------------------|
| **Next.js 15 + React 19** | Full-stack framework; App Router for streaming RSC, Server Actions for mutations | App Router's streaming model is essential for progressive tool-call rendering — Pages Router can't do this natively |
| **Vercel AI SDK (`ai` v5)** | Agent runtime — `streamText` with multi-tool execution and streaming | First-class tool-chaining and streaming primitives; `@ai-sdk/gateway` gives model portability without provider lock-in |
| **BetterAuth** | Authentication — session management, social providers, Stripe plugin integration | Purpose-built for Next.js with native Stripe subscription plugin; less glue code than NextAuth for monetized apps |
| **Prisma + Neon Postgres** | ORM + serverless Postgres; `@prisma/adapter-neon` for edge compatibility | Neon's connection pooling survives Vercel's serverless cold-start pattern; Accelerate extension for edge caching |
| **Hono** | Typed API layer with Zod validator middleware | Runs on Vercel Edge runtime; `@hono/zod-validator` enforces schemas at the route level before business logic |
| **Tavily (`@tavily/core`)** | Web search tool exposed to the agent | Purpose-built for LLM tool-call integration — structured results, no HTML parsing |
| **Tailwind v4 + Shadcn/ui** | UI styling + accessible component primitives | Tailwind v4's new CSS engine eliminates purge config; Shadcn components are unstyled Radix primitives — no fighting a component library's opinions |
| **Stripe + `@better-auth/stripe`** | Subscription billing, plan gating | Plugin integrates directly with BetterAuth session — plan state available in middleware without an extra DB call |
| **TanStack Query v5** | Server state management, cache invalidation | Optimistic updates and background refetch without manual cache wiring; pairs cleanly with Next.js Server Actions |
| **Zustand v5** | Client-side UI state (sidebar state, model selection, active session) | Zero-boilerplate for shallow global state; no context provider nesting |
| **TestSprite** | AI-powered test agent — automated UI regression testing | Scales test coverage without writing and maintaining brittle Playwright selectors manually |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Unit | TypeScript type system + Zod schemas | Input validation contracts at every API boundary |
| Integration | Hono route handlers against real Neon test DB | End-to-end request → DB write → response on isolated test tenant |
| AI Agent | TestSprite automated agent | UI-level regression — auth flows, tool-call rendering, subscription gate UI |
| Contract | Zod schemas on all tool inputs/outputs | Prevents model-generated tool arguments from violating expected structure |

**Explicitly not tested and why:** No load/performance suite at v1 — Vercel's edge runtime and Neon's connection pooler handle burst natively; perf testing added when baseline metrics identify a specific bottleneck.

---

## 📡 Observability

- **📝 Logs:** Structured JSON per request — tool-call name, model, latency, user ID (hashed), tenant plan tier. PII excluded.
- **📊 Metrics:** Tool-call success/failure rate, agent turn latency (p50/p95), Stripe webhook delivery rate, active subscription count.
- **🔍 Tracing:** Request correlation IDs propagated through Hono middleware → agent execution → Prisma queries. Full tool-call chain reconstructable from a single ID.
- **🚨 Alerting:** Stripe webhook failure rate > 1% pages on-call. Agent tool-call error spike (> 5% over 5min window) triggers automatic fallback to single-model mode.

---

## 📊 Impact & Results

| Metric | Baseline | Result | Measurement Method |
|--------|----------|--------|--------------------|
| ⏱️ Time-to-first-token | N/A (new product) | < 800ms p95 | Vercel Analytics, streaming RSC |
| ⚡ Tool-chain completion (3-step) | N/A | < 6s end-to-end | Client-side perf marks, production traffic |
| 🔐 Auth flow (sign-in to dashboard) | N/A | < 1.2s | Lighthouse CI on Vercel preview |
| 💳 Subscription gate enforcement | Manual per-route checks | 100% middleware-enforced | Code review + integration test coverage |

---

## 🚧 Challenges

**🔹 Streaming tool-call state to the client**
- **Problem:** Vercel AI SDK v5's multi-tool streaming returns interleaved text and tool-call events — naively rendering these produces visual thrash (text appearing before tool results resolve).
- **Tried:** Buffering all events before render, which killed the perceived performance advantage of streaming entirely.
- **Solution:** Implemented a tool-call state machine in the React layer using `useChat` event callbacks — each tool call renders a loading indicator immediately on `tool_call_start`, updates on `tool_call_result`. 
- **Tradeoff:** more complex client state, worth it for the UX delta.

**🔹 Indirect prompt injection via URL extraction**
- **Problem:** Fetching arbitrary URLs and passing content to the model creates an injection surface — a malicious page could embed instructions designed to override the agent's tool-call behavior.
- **Tried:** Simple content truncation, which doesn't address structured injection attempts embedded early in content.
- **Solution:** URL content is injected as a scoped `user` turn with an explicit framing prefix, never concatenated into the system prompt. Tool parameter validation via Zod catches out-of-scope values before execution. Documented as a known residual risk.

**🔹 Prisma on Vercel Edge Runtime**
- **Problem:** Standard Prisma client uses Node.js TCP connections — incompatible with Vercel's Edge runtime which runs on V8 isolates.
- **Tried:** Deploying the agent route as a Node.js serverless function, which introduced cold-start latency inconsistent with streaming UX expectations.
- **Solution:** `@prisma/adapter-neon` with WebSocket-based connection pooling — compatible with Edge runtime. `@prisma/extension-accelerate` for query-level caching. Tradeoff: slightly more complex Prisma client initialization, no impact on DX at the query layer.

---

## 🔄 What I'd Do Differently

- **🔹 Vector store for note retrieval:** `find_note` currently uses Postgres full-text search. For semantic retrieval ("find notes related to this concept"), pgvector or a dedicated vector DB (Pinecone/Qdrant) would give the agent meaningfully better recall — the architecture supports swapping the tool's backing store without touching agent logic.
- **🔹 Tool-call audit log as a first-class feature:** Structured tool-call history exists in logs but isn't surfaced to users. Exposing agent execution traces in the UI (what tools ran, in what order, with what parameters) would dramatically improve trust and debuggability — especially for power users building complex workflows.
- **🔹 Separate agent runtime from the Next.js app:** Bundling the agent in the Next.js app couples its deployment and scaling to the frontend. A dedicated Hono service on a separate Vercel project would allow independent scaling, separate observability, and cleaner trust boundary enforcement between UI and agent.

---

## 📸 Screenshots

| View | |
|------|-|
| 🖥️ Agent Chat Interface | ![Overview](./screenshots/overview.png) |
| 🔍 Tool-Call Chain Rendering | ![Tool Chain](./screenshots/tool-chain.png) |
| 💳 Subscription & Plan Gate | ![Billing](./screenshots/billing.png) |
| 📝 Knowledge Store (Notes) | ![Notes](./screenshots/notes.png) |

---

## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Component breakdown, data flow, trust model, agent execution model |
| [API Reference](./api-reference/endpoints.md) | Hono endpoint surface, auth requirements, tool schemas, rate limits |

---

## 🔗 Connect & Explore

[![Live Demo](https://img.shields.io/badge/Live_Demo-000000?style=for-the-badge&logo=vercel&logoColor=white)](#)
[![Documentation](https://img.shields.io/badge/Documentation-FF6B35?style=for-the-badge&logo=gitbook&logoColor=white)](./docs/architecture.md)
[![Portfolio](https://img.shields.io/badge/Portfolio-CC785C?style=for-the-badge&logoColor=white)](#)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](#)
