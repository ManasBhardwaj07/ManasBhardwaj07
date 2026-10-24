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
  CLIENT TRAFFIC (X-API-Key)
              │
              ▼
  ┌──────────────────────────────── DATA PLANE (:4000) ────────────────────────────────┐
  │                                                                                     │
  │   [ 1. AUTH & TENANT ] ──► [ 2. ATOMIC POLICY ] ──► [ 3. SSRF & ROUTING ] ──► [ 4. PROXY ]
  │   • SHA-256 hash match     • Sliding-Window limit   • Longest-prefix match  • Inject X-Trace
  │   • Redis ➔ PG fallback    • Monthly quota check    • DNS rebinding shield  • Forward to    
  │   • Resolve Org & Plan     • Single Lua roundtrip   • Pin resolved IP         Upstream APIs 
  │                                                                                     │
  └───────────────────┬──────────────────────┬──────────────────────────────────────────┘
                      │                      │
          runtime cache & counters           │ proxied request
                      │                      ▼
                      ▼           ┌───────────────────────┐   ┌─────────────────────────┐
  ┌───────────────────────────────┤  UPSTREAM SERVICES    │   │  NEXT.JS DASHBOARD      │
  │  STATE & CACHE LAYER          │  • mock-orders (:5001)│   │  Port 3000 (via Caddy)  │
  │  • Redis 7: Rate limits,      │  • mock-payments(:5002│   │  • Live Telemetry & RPS │
  │    token cache & quota state  │  └────────────────────┘   │  • Decision Inspector   │
  │  • PostgreSQL 16: System      │                           │  • Traffic Playground   │
  │    source-of-truth (Prisma)   │◄──────────────────────────┴─────────────────────────┘
  └───────────────▲───────────────┘       admin management (internal only)
                  │
  ┌───────────────┴─────────────────────────────────────────────────────────────────────┐
  │  PRIVATE CONTROL PLANE (:4001) · Isolated Internal Docker Network                   │
  │  • Tenant & Plan Provisioning  • Dynamic Route Config  • Constant-Time Bearer Auth  │
  └─────────────────────────────────────────────────────────────────────────────────────┘
```

**Key decisions:**
- **Atomic rate limiting:** Sliding-window limits and monthly quotas are checked in a single Redis Lua script — one roundtrip, no distributed lock, no check-then-increment race under concurrent bursts.
- **SSRF defense in two phases:** Upstream targets are validated when routes are defined and re-validated at connection time with IP pinning, covering private subnets, link-local and cloud-metadata addresses, and DNS rebinding.
- **Control plane never public:** The admin API runs on `:4001` inside the internal Docker network and is never exposed through Caddy. API keys are stored only as SHA-256 hashes.

**108 Vitest tests across 11 suites** &nbsp;·&nbsp; [Live Demo](https://gateforgeapp.duckdns.org) &nbsp;·&nbsp; [Source Code](https://github.com/ManasBhardwaj07/GateForge)

---

### CodeForge — Sandboxed Code Execution Platform

> `Next.js · TypeScript · BullMQ · Redis · Docker · Prisma · Neon PostgreSQL · Azure · Caddy`
>
> Queue-backed code execution with Docker isolation, worker heartbeats, and deterministic verdict aggregation across 5 languages (JavaScript, Python, C, C++, Java).
>
> **Interface:** Next.js app with a code editor, submission history, and live status polling.

```text
  CLIENT / CODE EDITOR (JS · Python · C · C++ · Java)
                        │
                        ▼
  ┌──────────────────────────────── FRONTEND & INGRESS (Next.js) ───────────────────────┐
  │  • Monaco Editor + Status Polling UI                                                │
  │  • POST /api/run    ──► Ephemeral test run with custom stdin                        │
  │  • POST /api/submit ──► Graded submission evaluated against hidden test suites      │
  └───────────────────────────────────────────┬─────────────────────────────────────────┘
                                              │ enqueue execution job
                                              ▼
  ┌─────────────────────────────── ASYNC JOB QUEUE (Redis + BullMQ) ────────────────────┐
  │  • Job Lifecycle Tracking   • Worker TTL Heartbeats   • Exponential Backoff Retries │
  └───────────────────────────────────────────┬─────────────────────────────────────────┘
                                              │ worker pulls job
                                              ▼
  ┌────────────────────────────── WORKER EXECUTION ENGINE ──────────────────────────────┐
  │                                                                                     │
  │   ┌─ EPHEMERAL DOCKER SANDBOX (Spawns 1 fresh container per run) ───────────────┐   │
  │   │  • Isolation: Non-root user (uid 1000) & Read-only root filesystem          │   │
  │   │  • Guardrails: CPU/RAM cgroups · Hard timeout SIGKILL · stdout/err bounded  │   │
  │   │  • Compiles & runs code in isolated memory space                            │   │
  │   └───────────────────────────────────────┬─────────────────────────────────────┘   │
  │                                           │ captured exit code, streams & time      │
  │                                           ▼                                         │
  │   ┌─ DETERMINISTIC VERDICT ENGINE ──────────────────────────────────────────────┐   │
  │   │  Strict Hierarchy: TIMEOUT ──► COMPILE_ERR ──► RUNTIME_ERR ──► WA ──► ACCEPT │   │
  │   └─────────────────────────────────────────────────────────────────────────────┘   │
  │                                                                                     │
  └───────────────────────┬─────────────────────────────────────────────┬───────────────┘
                          │ success / graded verdicts                   │ poison / failed job
                          ▼                                             ▼
  ┌──────────────────────────────────────────────┐  ┌───────────────────────────────────┐
  │  STORAGE LAYER (Neon PostgreSQL + Prisma)    │  │  DEAD LETTER QUEUE (DLQ)          │
  │  • Problems catalog, test cases & submissions│  │  • Isolated failed & abusive jobs │
  │  • Connection pool tuned for scale-to-zero   │  │  • Endpoints: Inspect · Replay    │
  └──────────────────────────────────────────────┘  └───────────────────────────────────┘
```

**Key decisions:**
- **Untrusted code, contained:** Every execution runs in a newly spawned, ephemeral Docker container with a hard timeout, output truncation, a non-root user, and a read-only root filesystem.
- **Run vs Submit:** An ephemeral `Run` path with custom input vs. a DB-backed `Submit` path tested against hidden cases, with a deterministic verdict hierarchy (`TIMEOUT` strictly outranks `WRONG_ANSWER`).
- **Designed for failure:** Redis TTL heartbeats feed queue-health endpoints, failed jobs land in an inspectable Dead Letter Queue, and dedicated scripts test infinite loops, stdout flooding, orphan-container cleanup, and worker hard-kill recovery.

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
| **Languages** | ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) &nbsp; ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) &nbsp; ![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white) &nbsp; ![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white) |
| **Backend & Queues** | ![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=nodedotjs&logoColor=white) &nbsp; ![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square&logo=express&logoColor=white) &nbsp; ![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square) &nbsp; ![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white) |
| **Frontend** | ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) &nbsp; ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) &nbsp; ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white) &nbsp; ![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square&logo=framer&logoColor=white) |
| **Databases & ORM** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) &nbsp; ![Neon](https://img.shields.io/badge/Neon_Serverless-00E599?style=flat-square&logo=neon&logoColor=black) &nbsp; ![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white) &nbsp; ![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white) |
| **Cloud & Infra** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) &nbsp; ![AWS EC2](https://img.shields.io/badge/AWS_EC2-232F3E?style=flat-square) &nbsp; ![Azure VM](https://img.shields.io/badge/Azure_VM-0078D4?style=flat-square) &nbsp; ![Caddy](https://img.shields.io/badge/Caddy-1F88C0?style=flat-square&logo=caddy&logoColor=white) &nbsp; ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) |

---

<p align="center"><i>Systems that hold under burst traffic, contain hostile code, and recover from failure — every claim above links to code you can read.</i></p>
