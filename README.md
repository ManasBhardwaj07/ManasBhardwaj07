<h1 align="center">Manas Bhardwaj</h1>

<h3 align="center">Software Engineer &nbsp;·&nbsp; Full-Stack & Backend Systems</h3>

<p align="center">
  <a href="https://www.linkedin.com/in/manas-bhardwaj-/"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white" /></a>
  &nbsp;
  <a href="mailto:manasbhardwaj264@gmail.com"><img alt="Email" src="https://img.shields.io/badge/Email-D14836?style=flat-square&logo=gmail&logoColor=white" /></a>
  &nbsp;
  <a href="https://gateforgeapp.duckdns.org"><img alt="GateForge Live Demo" src="https://img.shields.io/badge/GateForge-Live-0ea5e9?style=flat-square" /></a>
  &nbsp;
  <a href="https://codeforgeapp.duckdns.org"><img alt="CodeForge Live Demo" src="https://img.shields.io/badge/CodeForge-Live-8b5cf6?style=flat-square" /></a>
  &nbsp;
  <a href="https://apex-drive-frontend.vercel.app/"><img alt="ApexDrive Live Demo" src="https://img.shields.io/badge/ApexDrive-Live-f59e0b?style=flat-square" /></a>
</p>

<p align="center">
  <b>Atomic Rate Limiting</b> &nbsp;·&nbsp; <b>Sandboxed Code Execution</b> &nbsp;·&nbsp; <b>Failure-Aware Job Processing</b>
</p>

---

I build full-stack software — React and Next.js interfaces on top of backend systems I design for failure: an API gateway, a sandboxed code-execution engine, and distributed job queues. I focus on what happens when things break — burst traffic, hostile code, failed jobs. Previously at **Emendo AI**.

---

## Featured Systems

### GateForge — Programmable API Gateway

> `TypeScript · Node.js · Express · Redis (Lua) · PostgreSQL · Prisma · Next.js · Docker · Azure · Caddy`
>
> Multi-tenant API traffic control with atomic Redis rate limits, quota enforcement, longest-prefix routing, and SSRF-aware upstream validation.
>
> **Interface:** Next.js dashboard with live metrics, a traffic playground, and a per-request Decision Inspector.

```text
REQUEST PATH   client sends X-API-Key
Authenticate + tenant ──►  Enforce policy ─────────►  Route + SSRF guard ─────►  Proxy to upstream
SHA-256 hash lookup        Sliding-window limit +     Longest-prefix match       Forward + X-Request-Id
Redis → Postgres fallback  monthly quota, atomic      SSRF: block private /      Request telemetry
Resolve org + plan         Redis Lua script [A]       metadata / rebinding IPs
                                                      pin resolved IP [B]

SUPPORTING COMPONENTS
┌─ PRIVATE CONTROL PLANE :4001 ──┐  ┌─ STATE ────────────────────────┐  ┌─ DASHBOARD :3000 ──────────────┐
│ Docker-internal only [C]       │  │ PostgreSQL 16                  │  │ Next.js served via Caddy       │
│ plans · tenants · upstreams    │  │   source of truth (Prisma)     │  │ live metrics                   │
│ routes · API keys              │─►│ Redis 7                        │  │ traffic playground             │
│ constant-time bearer auth      │  │   key cache · rate-limit +     │  │ Decision Inspector             │
│ IP throttling · audit log      │  │   quota counters               │  │                                │
└────────────────────────────────┘  └────────────────────────────────┘  └────────────────────────────────┘

```

**Key decisions:**
- **[A] Atomic rate limiting:** Sliding-window limits and monthly quotas are checked in a single Redis Lua script — one roundtrip, no distributed lock, no check-then-increment race under concurrent bursts.
- **[B] SSRF defense in two phases:** Upstream targets are validated when routes are defined and re-validated at connection time with IP pinning, covering private subnets, link-local and cloud-metadata addresses, and DNS rebinding.
- **[C] Control plane never public:** The admin API runs on `:4001` inside the internal Docker network and is never exposed through Caddy. API keys are stored only as SHA-256 hashes.

**108 Vitest tests across 11 suites** &nbsp;·&nbsp; [Live Demo](https://gateforgeapp.duckdns.org) &nbsp;·&nbsp; [Source Code](https://github.com/ManasBhardwaj07/GateForge)

---

### CodeForge — Sandboxed Code Execution Platform

> `Next.js · TypeScript · BullMQ · Redis · Docker · Prisma · Neon PostgreSQL · Azure · Caddy`
>
> Queue-backed code execution with Docker isolation, worker heartbeats, and deterministic verdict aggregation across 5 languages (JavaScript, Python, C, C++, Java).
>
> **Interface:** Next.js app with a code editor, submission history, and live status polling.

```text
EXECUTION PIPELINE
Editor / API routes (Next.js)
  │ POST /api/run (custom input)  ·  /api/submit (hidden test cases)
  ▼
Redis + BullMQ queue
  │ job state · retry metadata · worker heartbeats (TTL keys)
  ▼
Worker + ephemeral Docker sandbox [A]
  │ non-root user · read-only root filesystem · hard timeout kill · output size bounded
  ▼
Verdict aggregation [B]
  │ Accepted · Compile Error · Timeout · Runtime Error · Wrong Answer  (TIMEOUT outranks WRONG_ANSWER)
  ▼
Persist + return results (Neon PostgreSQL via Prisma)
    problems · test cases · submissions · verdicts · pooling tuned for scale-to-zero autosuspend

FAILURE PATHS [C]
• Hard timeout kills the container; stdout/stderr are truncated
• Worker heartbeats (TTL keys) feed queue-health endpoints
• Failed jobs: retry, then Dead Letter Queue — inspect · replay · delete
```

**Key decisions:**
- **[A] Untrusted code, contained:** Every execution runs in a newly spawned, ephemeral Docker container with a hard timeout, output truncation, a non-root user, and a read-only root filesystem.
- **[B] Run vs Submit:** An ephemeral `Run` path with custom input vs. a DB-backed `Submit` path tested against hidden cases, with a deterministic verdict hierarchy (`TIMEOUT` strictly outranks `WRONG_ANSWER`).
- **[C] Designed for failure:** Redis TTL heartbeats feed queue-health endpoints, failed jobs land in an inspectable Dead Letter Queue, and dedicated scripts test infinite loops, stdout flooding, orphan-container cleanup, and worker hard-kill recovery.

[Live Demo](https://codeforgeapp.duckdns.org) &nbsp;·&nbsp; [Source Code](https://github.com/ManasBhardwaj07/CodeForge)

---

## Projects

| Project | What it solves | Stack | Key Outcomes |
|---|---|---|---|
| **[GateForge](https://github.com/ManasBhardwaj07/GateForge)** | Multi-tenant API gateway: auth, rate limiting, routing, and observability | TypeScript, Node.js, Express, Redis (Lua), PostgreSQL, Next.js, Docker, Azure | Atomic Lua rate limiting · two-phase SSRF defense · plane isolation · 108 tests |
| **[CodeForge](https://github.com/ManasBhardwaj07/CodeForge)** | Sandboxed multi-language code execution with deterministic verdict aggregation | Next.js, TypeScript, BullMQ, Redis, Docker, Prisma, Neon PostgreSQL, Azure | Container isolation · deterministic verdicts · DLQ and worker heartbeats · abuse-tested |
| **[ApexDrive](https://github.com/ManasBhardwaj07/ApexDrive)** | Subscription rewards platform with a monthly draw engine and charitable giving | React, Vite, Node.js, Supabase, PostgreSQL, Stripe | 4 Stripe webhook lifecycle events · Row Level Security · DB-trigger score window · admin analytics |
| **[ProjectHub](https://github.com/ManasBhardwaj07/ProjectHub)** | Real-time collaborative workspace with multi-user sync | React, Node.js, Express, Socket.IO, MongoDB Atlas, JWT | Drag-and-drop React UI · live WebSocket sync · RBAC · tokenized password reset |
| **[QueueForge](https://github.com/ManasBhardwaj07/QueueForge)** | Asynchronous distributed job processing with worker isolation | Node.js, BullMQ, Redis, MongoDB, Docker Compose, AWS EC2 | Exponential backoff · retryable/permanent failure classification · 10/10 jobs completed in documented burst test |

---

## Tech Stack

| Layer | Technologies |
|---|---|
| **Languages** | ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white) ![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white) |
| **Backend & Queues** | ![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=nodedotjs&logoColor=white) ![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square&logo=express&logoColor=white) ![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square) ![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white) ![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white) |
| **Frontend** | ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white) ![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square&logo=framer&logoColor=white) |
| **Databases & ORM** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Neon](https://img.shields.io/badge/Neon_Serverless-00E599?style=flat-square&logo=neon&logoColor=black) ![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=black) ![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white) |
| **Cloud & DevOps** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white) ![Azure VM](https://img.shields.io/badge/Azure_VM-0078D4?style=flat-square) ![AWS EC2](https://img.shields.io/badge/AWS_EC2-232F3E?style=flat-square) ![Caddy](https://img.shields.io/badge/Caddy-1F88C0?style=flat-square&logo=caddy&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white) |
| **Security & Payments** | ![Stripe Webhooks](https://img.shields.io/badge/Stripe_Webhooks-635BFF?style=flat-square&logo=stripe&logoColor=white) ![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white) ![Row Level Security](https://img.shields.io/badge/Row_Level_Security-3FCF8E?style=flat-square&logo=supabase&logoColor=black) ![bcrypt](https://img.shields.io/badge/bcrypt-5A5A5A?style=flat-square) ![SHA-256](https://img.shields.io/badge/SHA--256-6E40C9?style=flat-square) |

---

<p align="center"><i>Systems that hold under burst traffic, contain hostile code, and recover from failure — every claim above links to code you can read.</i></p>
