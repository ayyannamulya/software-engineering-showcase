<div align="center">

# 🎙️ Hawken — AI Podcast Content Engine

### *Upload one audio file and get a complete, platform-ready content distribution package in under 90 seconds — powered by parallel AI processing.*

![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini_AI-4285F4?style=flat-square&logo=google&logoColor=white)
![AssemblyAI](https://img.shields.io/badge/AssemblyAI-7B2FBE?style=flat-square&logoColor=white)
![Convex](https://img.shields.io/badge/Convex-EE342F?style=flat-square&logoColor=white)
![Inngest](https://img.shields.io/badge/Inngest-4F46E5?style=flat-square&logoColor=white)
![Clerk](https://img.shields.io/badge/Clerk-6C47FF?style=flat-square&logo=clerk&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)


![Status](https://img.shields.io/badge/Status-Production-2ea44f?style=flat-square)
![Version](https://img.shields.io/badge/Version-0.1.0-555555?style=flat-square)
![License](https://img.shields.io/badge/License-Private-red?style=flat-square)

> 🔒 **Showcase Repo** — Source code is private. This repository documents architecture, design decisions, and engineering results. All examples use sanitized data.

</div>

---

## 🧩 Problem

Podcast creators invest hours in recording and production — then face an entirely separate multi-hour workload before a single episode can be distributed. Writing platform-specific social copy for six networks, generating YouTube timestamps, crafting SEO titles, identifying clip moments, and formatting transcripts are each their own manual task. Done sequentially, this post-production content work routinely takes 3–5 hours per episode.

The tooling landscape doesn't close this gap. Transcription tools give you a wall of text. AI writing tools require you to paste context, prompt carefully, and repeat per platform. Nothing treats the podcast episode as a first-class input and produces a complete, distribution-ready content package as output — until now.

---

## 💡 Solution

Hawken accepts a single audio upload and runs an end-to-end AI pipeline that produces every content asset a creator needs to distribute an episode across six platforms — transcript, social posts, titles, hashtags, timestamps, and clip candidates — delivered in under 90 seconds.

**The defining architectural decision is parallel AI job execution via Inngest.** Generating content for six platforms sequentially would take 5+ minutes and fail silently on any single step. Instead, Inngest fans out six independent AI generation jobs the moment transcription completes, running them concurrently with automatic retry on failure. Total wall-clock time drops to ~90 seconds regardless of how many platforms are in scope. The tradeoff is added orchestration complexity — each job must be idempotent and self-contained — but the durability guarantees mean no partial results ever reach the user without a recovery path.

Real-time progress is surfaced via Convex subscriptions, eliminating polling and giving creators live visibility into which jobs have completed without any client-side orchestration logic.

---

## ✨ Key Features

| Feature | How It Works | Why It Matters |
|---------|--------------|----------------|
| ⚡ Parallel AI Processing | Inngest fans out 6 AI generation jobs simultaneously the moment AssemblyAI transcription completes | Reduces total generation time from ~5 minutes sequential to ~90 seconds — the difference between a tool creators use and one they abandon |
| 📱 Platform-Optimized Social Copy | Gemini generates per-platform posts with distinct tone, format, and length constraints: 280-char Twitter, professional LinkedIn, hook-driven Instagram, casual TikTok, CTA-heavy YouTube, community-focused Facebook | Generic AI copy fails on every platform; platform-aware prompting produces posts that match each network's native content patterns |
| 👥 Speaker Diarization | AssemblyAI identifies and labels individual speakers throughout the transcript with confidence scores | Enables accurate quote attribution in social copy and key moments — critical for interview-format podcasts with multiple guests |
| 🔄 Real-Time Progress Updates | Convex reactive queries push job completion state to the client as each Inngest step finishes — no polling, no manual refresh | Creators see live progress per platform; the UI reflects partial completion rather than a blocking spinner for the full 90-second window |
| 🛡️ Durable Workflow Execution | Inngest automatically retries failed AI steps with exponential backoff; job state is persisted between attempts | No silent failures, no lost work mid-pipeline — a transient Gemini timeout doesn't force the creator to restart the entire job |
| 💳 Plan-Based Feature Gating | Feature access (platforms, transcript length, clip count) enforced server-side against Clerk subscription metadata | Monetization logic lives at the service layer, not in client-side conditionals — plan state is authoritative and tamper-resistant |

---

## 🏗️ Architecture

```
[Browser — Next.js 16 App Router / React 19]
        │
        │  HTTPS  (Server Actions / Route Handlers)
        ▼
┌──────────────────────────────────┐
│       Next.js API Surface        │  ← Clerk JWT validation, plan-tier enforcement,
│  (Route Handlers + Server        │    Zod input validation before any downstream call
│   Actions)                       │
└───────────────┬──────────────────┘
                │
   ┌────────────┴──────────────────┐
   │        Inngest Engine         │  ← Durable workflow orchestration
   │  (Durable Job Orchestrator)   │    Fan-out: 6 parallel AI generation steps
   │                               │    Retry logic, step persistence, failure isolation
   └──────┬──────────┬─────────────┘
          │          │
          ▼          ▼
┌──────────────┐  ┌──────────────────────────┐
│  AssemblyAI  │  │  Google Gemini API        │
│              │  │                           │
│  · Transcription│  · Social post generation  │
│  · Speaker   │  │  · Title suggestions      │
│    diarization│  │  · Hashtag generation    │
│  · Timestamps│  │  · Clip identification    │
└──────┬───────┘  │  · Episode summary        │
       │          └──────────────┬────────────┘
       │                         │
       └──────────┬──────────────┘
                  │
   ┌──────────────┴──────────────┐
   │          Convex             │  ← Reactive datastore
   │   (Realtime Database)       │    Job state, generated content,
   │                             │    user data — reactive queries
   └─────────────────────────────┘
          │
   ┌──────┴───────┐
   │  Vercel Blob │  ← Audio file storage
   │              │    Uploaded episodes staged here
   └──────────────┘

Trust boundary  : Clerk session JWT validated at every Route Handler entry point.
                  Plan tier resolved server-side from Clerk metadata — never from
                  client payload. Inngest webhook signatures verified on every event.
External trust  : AssemblyAI and Gemini responses are validated against Zod schemas
                  before storage. Audio files are size and MIME-type validated before
                  Vercel Blob upload.
```

### Data Flow

| Step | Component | What Happens |
|------|-----------|--------------|
| 1. Upload | Route Handler | Clerk session validated; plan tier checked for file size and duration limits; audio MIME type and size enforced; file staged to Vercel Blob |
| 2. Transcription | Inngest → AssemblyAI | Inngest job triggered on upload; AssemblyAI processes audio with speaker diarization enabled; transcript, speaker labels, and timestamps written to Convex |
| 3. Fan-Out | Inngest Orchestrator | On transcription completion, Inngest fans out 6 parallel generation steps — one per platform plus summary, titles, hashtags, and clip identification |
| 4. Generation | Gemini (×6 parallel) | Each step runs an independent, platform-specific Gemini prompt against the transcript; output validated against Zod schema; result written to Convex on success; step retried on failure |
| 5. Progress | Convex → Client | Convex reactive query pushes job state update to client on each step completion — no polling; UI renders partial results as they arrive |
| 6. Delivery | Client (React 19) | Completed content rendered per platform with copy controls and export options; plan-gated features visually distinguished |

> Full component breakdown, degraded-path behavior, and retry topology → [Architecture Doc](./docs/architecture.md)

---

## 🔐 Security Design

Security decisions are engineering decisions — documented here, not treated as an afterthought.

| Concern | Decision | Rationale |
|---------|----------|-----------|
| 🔑 Authentication | Clerk session JWTs | Offloads token lifecycle, rotation, and session invalidation. No custom auth surface to maintain |
| 🛡️ Authorization | Plan-tier feature gating via Clerk metadata, enforced server-side | Feature access checked at the service layer against `publicMetadata.plan` — client UI state is never the enforcement point |
| 🧹 Input Trust | File upload validated for MIME type, size, and duration before Vercel Blob write; all AI responses validated against Zod schemas before Convex write | Two untrusted surfaces: user-uploaded audio and third-party AI responses — both sanitized before they touch persistent state |
| 🔐 Secrets Management | Environment variable injection via Vercel — `GEMINI_API_KEY`, `ASSEMBLYAI_API_KEY`, Clerk keys, Inngest signing key injected at deploy time | Zero hardcoded credentials; rotation handled through Vercel environment management |
| 🔏 Webhook Integrity | Inngest webhook payloads verified against signing key on every inbound event | Prevents spoofed job completions from writing fabricated AI content to Convex |
| 📋 Audit Logging | Per-job logs: `userId`, `episodeId`, `planTier`, `stepName`, `durationMs`, `retryCount` — transcript and generated content excluded | Sufficient for abuse investigation and billing reconciliation without retaining user audio content server-side |

**⚠️ Accepted Risk:**
- AssemblyAI processes audio on their infrastructure — episode audio leaves the application boundary. Mitigated by Vercel Blob signed URLs (time-limited, scoped access) and AssemblyAI's data retention policy. Creators handling sensitive interview content should be made aware of this boundary.
- Inngest job fan-out has no global rate limit cap per user in v1 — a user with Pro access could trigger multiple concurrent large jobs simultaneously, multiplying Gemini API spend. A per-user concurrent job limit is the correct fix; deferred for v1 given low initial user volume.
- Convex serves as both the operational datastore and the real-time subscription layer — no separation between transactional and reactive read paths. Acceptable for current scale; a dedicated read replica or event stream would be the right split under sustained high load.

---

## 🧰 Tech Stack

> Only what's worth explaining — technologies that define the project's character, solve a non-obvious problem, or represent a deliberate tradeoff.

| Technology | Role in This Project | Why This, Not the Alternative |
|------------|----------------------|-------------------------------|
| Inngest | Durable workflow orchestrator for the AI fan-out pipeline | Gives step-level retry, persistence, and failure isolation without running a queue infrastructure. The alternative — a raw job queue (BullMQ, etc.) — requires managing workers, Redis, and retry logic manually. Inngest handles all of that as a managed service, and the step model maps directly to the fan-out pattern needed here |
| Convex | Reactive datastore powering real-time job progress updates | Convex's reactive query model means the client receives push updates on every job state change with zero polling code. Supabase Realtime or Firebase would work similarly, but Convex's TypeScript-native schema and server functions eliminate a full ORM layer and keep the data access pattern end-to-end type-safe |
| AssemblyAI | Audio transcription with speaker diarization | Best-in-class diarization accuracy for interview-format podcasts at production API scale. Whisper (self-hosted or via OpenAI) provides comparable transcription quality but no native diarization — building speaker separation on top would be a significant additional scope |
| Google Gemini | Content generation for all six platform outputs | Strong instruction-following for format-constrained generation (character limits, tone, platform conventions). Parameterized behind the service layer — swappable without interface changes |
| Vercel Blob | Audio file staging before AssemblyAI processing | Provides signed URL uploads directly from the browser, keeping large audio files off the Next.js server entirely. S3 is the obvious alternative but adds IAM configuration overhead; Blob integrates natively with the Vercel deployment with no additional infra |
| Clerk | Authentication and subscription plan metadata | Session management plus plan-tier entitlement in one integration — eliminates a custom billing enforcement layer. Webhook-driven plan sync keeps server-side metadata authoritative |

---

## 🧪 Testing Strategy

| Layer | Approach | Coverage Focus |
|-------|----------|----------------|
| Unit | Node.js built-in test runner (`node:test`) | Inngest step isolation logic (fan-out graph correctness), Zod schema validation against fixture AI responses, plan-tier gating enforcement, file validation (MIME type, size, duration limits) |
| Integration | Inngest dev server + Convex local backend | Full upload → transcription → fan-out → generation → Convex write pipeline end-to-end; retry behavior simulated by injecting transient failures at the step level |
| AI Output Compliance | Zod schema validation against captured Gemini fixture payloads | Validates that schema enforcement catches malformed or truncated Gemini responses across all six platform output shapes before they reach Convex |
| Load & Performance | k6 | Concurrent job submission under sustained load; Convex subscription throughput under multiple simultaneous active jobs; Inngest fan-out timing vs. sequential baseline |

**Explicitly not tested and why:** No E2E browser suite. The correctness-critical surface — pipeline orchestration, retry behavior, AI output validation, plan gating — is fully covered at the integration layer using the Inngest dev server and local Convex backend. The UI is a thin rendering layer over reactive Convex queries; Playwright overhead is not warranted for v1.

---

## 📡 Observability

- **📝 Logs:** Structured JSON per Inngest step: `userId`, `episodeId`, `stepName`, `planTier`, `durationMs`, `retryCount`, `status`. Transcript text and generated content explicitly excluded from all log payloads.
- **📊 Metrics:** End-to-end pipeline duration (upload → all steps complete) p50/p95/p99; per-step generation latency by platform; AssemblyAI transcription duration by audio length; Gemini error rate by step type; Inngest retry rate.
- **🔍 Tracing:** Inngest's built-in run dashboard provides step-level execution traces — duration, retry history, and payload per step — without additional instrumentation. Correlated to Convex writes via `episodeId` carried through all steps.
- **🚨 Alerting:** Inngest step failure rate > 10% over a 10-minute window (AI provider degradation); AssemblyAI transcription timeout rate > 5%; Convex write error rate spike; Vercel Blob upload failure rate increase.

---

## 📊 Impact & Results

| Metric | Before (Manual) | After (Hawken) | Measurement Method |
|--------|-----------------|----------------|--------------------|
| ⏱️ Time to produce full content package | 3–5 hours per episode | < 90 seconds | End-to-end timing from upload confirmation to all 6 platform outputs complete |
| ⚡ AI generation time — parallel vs. sequential | ~300s (sequential baseline) | ~60s (parallel) | Inngest step timing logs across 20 test runs; sequential baseline measured with steps run serially |
| 🔄 Pipeline reliability (step success rate) | N/A | 97.4% first-attempt success | Inngest run logs across 150 pipeline executions; failures recovered via automatic retry |
| 📝 Transcription accuracy (WER) | N/A | < 8% word error rate on clean audio | Spot-checked against manual transcript on 10 representative episodes |

---

## 🚧 Challenges

**⚡ Orchestrating Parallel AI Jobs Without Infrastructure Overhead**
- **Problem:** Six sequential AI generation calls would produce a 5-minute wait — long enough to break the user experience entirely. Naive parallelism via `Promise.all` in a serverless function would hit execution timeout limits on long episodes and provide no retry path on transient failures.
- **Tried:** Initial prototype used `Promise.all` inside a single Next.js API route. This worked on short episodes but timed out reliably on episodes over 30 minutes, and any single Gemini failure aborted the entire batch.
- **Solution:** Migrated generation to Inngest with each platform as an isolated step. Inngest handles fan-out, step-level persistence, and retry automatically — a failed LinkedIn generation step retries independently without re-running transcription or any other completed step. Accepted tradeoff: each step must be idempotent and self-contained, which required restructuring the prompt assembly to carry full context rather than relying on shared in-memory state.

**🔄 Real-Time Progress Without Polling**
- **Problem:** With six parallel jobs running for up to 90 seconds, the user needs live visibility into which steps have completed — a static loading spinner is not acceptable UX at this time scale. Implementing polling from the client adds latency, hammers the server, and still feels laggy compared to push updates.
- **Tried:** Initial implementation used a 2-second polling interval against a Next.js API route that queried Convex for job state. This worked but added visible lag between step completion and UI update, and generated unnecessary load.
- **Solution:** Replaced polling with Convex reactive queries. The client subscribes to a Convex query keyed by `episodeId`; Convex pushes a diff to the client the moment any Inngest step writes its result. Update latency dropped to under 200ms from step completion, with zero client-side orchestration code. The tradeoff is Convex as a required dependency — this pattern doesn't work with a traditional REST datastore.

**🎙️ Speaker Diarization Accuracy on Overlapping Audio**
- **Problem:** AssemblyAI's diarization model struggles with segments where two speakers overlap or interrupt each other — common in interview formats. Overlapping segments were either misattributed or collapsed into a single speaker, breaking quote attribution in downstream content generation.
- **Tried:** Increasing the `speakers_expected` parameter to give the model more signal. Had minimal effect on overlap segments specifically.
- **Solution:** Post-processed the diarization output to flag low-confidence segments (confidence score < 0.7) and exclude them from quote attribution in the Gemini prompts. High-confidence segments drive social copy and key moments; overlapping segments contribute to the summary context but are not attributed to a specific speaker. Accepted tradeoff: some genuine quotes are lost from social copy — preferable to misattribution, which would be visible and embarrassing in published content.

---

## 🔄 What I'd Do Differently

- **🚦 Per-User Concurrent Job Limit:** In v1, a Pro user can trigger multiple large pipeline runs simultaneously with no cap, multiplying Gemini and AssemblyAI spend proportionally. Inngest supports concurrency keys natively — a `userId`-scoped concurrency limit of 2 concurrent runs would bound the blast radius with a single configuration change. Should have been set from day one.
- **📦 Separate Transcript Storage from Generated Content:** Currently both the raw transcript and all generated platform content live in the same Convex document. As episode count grows, this produces large documents that are expensive to query when only the social copy is needed. Splitting into separate collections (`transcripts`, `generated_content`) with a foreign key on `episodeId` would make read patterns significantly cheaper and allow independent TTL policies.
- **🔁 Streaming Generation Output:** Gemini supports streaming responses, but the current implementation waits for the full generation before writing to Convex. Streaming partial output into Convex as generation progresses would reduce perceived latency further — users would see the Twitter post appearing word by word rather than waiting for the full batch. The Convex write pattern would need restructuring to support partial document updates per step, which added scope I deferred for v1.
- **🎯 Prompt Versioning:** Gemini prompts for each platform are hardcoded strings in the service layer. Any prompt change requires a code deploy, and there's no way to A/B test prompt variations or roll back a bad prompt without redeploying. A lightweight prompt registry — even a Convex document — would decouple prompt iteration from deployment cycles.

---

## 📸 Screenshots

| View | |
|------|-|
| 🖥️ Dashboard — Episode Upload | ![Dashboard](./screenshots/dashboard.png) |
| ⚡ Generation — Live Progress | ![Progress](./screenshots/progress.png) |
| 📱 Social Posts — Platform View | ![Social](./screenshots/social.png) |
| 🎤 Transcript — Speaker Diarization | ![Transcript](./screenshots/transcript.png) |

---

## 📄 Documentation

| Doc | Description |
|-----|-------------|
| [Architecture](./docs/architecture.md) | Component breakdown, Inngest fan-out topology, Convex schema, trust model, retry behavior |
| [API Reference](./api-reference/endpoints.md) | Route Handler surface, auth requirements, plan-tier enforcement, file upload constraints |

---

## 🔗 Connect & Explore

[![Live Demo](https://img.shields.io/badge/Live_Demo-000000?style=for-the-badge&logo=vercel&logoColor=white)](#)
[![Documentation](https://img.shields.io/badge/Documentation-FF6B35?style=for-the-badge&logo=gitbook&logoColor=white)](./docs/architecture.md)
[![Portfolio](https://img.shields.io/badge/Portfolio-CC785C?style=for-the-badge&logoColor=white)](#)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](#)