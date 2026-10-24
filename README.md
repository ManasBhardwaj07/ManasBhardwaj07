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

I build full-stack software — React and Next.js interfaces on top of backend systems I design for failure: an API gateway, a sandboxed code-execution engine, and distributed job queues. I focus on what happens when things break — burst traffic, hostile code, failed jobs. Previously at Emendo AI.
Featured Systems
GateForge — Programmable API Gateway
TypeScript · Node.js · Express · Redis (Lua) · PostgreSQL · Prisma · Next.js · Docker · Azure · Caddy

Multi-tenant API traffic control with atomic Redis rate limits, quota enforcement, longest-prefix routing, and SSRF-aware upstream validation.

Interface: Next.js dashboard with live metrics, a traffic playground, and a per-request Decision Inspector.

  CLIENT TRAFFIC (X-API-Key)
              │
              ▼
  ┌─ DATA PLANE :4000 ─────────────────────────────────────────────────────────────────────────────────┐
  │ [1] AUTH + TENANT     ─► [2] POLICY [A]        ─► [3] ROUTE + SSRF [B]  ─► [4] PROXY               │
  │                                                                                                    │
  │ • SHA-256 hash match     • Sliding-window limit   • Longest-prefix match   • Forward upstream      │
  │ • Redis → PG fallback    • Monthly quota check    • Block private IPs,     • Inject X-Request-Id   │
  │ • Resolve org + plan     • One atomic Lua call      metadata, rebinding    • Request telemetry     │
  │                                                   • Pin resolved IP                                │
  └────────────────────────────────────────────────────────────────────────────────────────────────────┘
              ▲                                               │ proxied request
              │ read on every request                         ▼
  ┌─ STATE ────────────────────────────────────┐    ┌─ UPSTREAM SERVICES ────┐    ┌─ DASHBOARD :3000 ────┐
  │ PostgreSQL 16  source of truth             │    │ Your APIs (demo):      │    │ Next.js via Caddy    │
  │   (Prisma)                                 │    │ mock-orders :5001      │    │ live metrics         │
  │ Redis 7  key cache · rate-limit            │    │ mock-payments :5002    │    │ traffic playground   │
  │   + quota counters                         │    │                        │    │ Decision Inspector   │
  └────────────────────────────────────────────┘    └────────────────────────┘    └──────────────────────┘
              ▲
              │ config + hashed keys
  ┌─ PRIVATE CONTROL PLANE :4001 [C] ────────────────────────────────────────────────────────────────────┐
  │ Docker-internal only, never exposed through Caddy                                                    │
  │ plans · tenants · upstreams · routes · API keys                                                      │
  │ constant-time bearer auth · IP throttling · audit log                                                │
  └──────────────────────────────────────────────────────────────────────────────────────────────────────┘
Key decisions:
- [A] Atomic rate limiting: Sliding-window limits and monthly quotas are checked in a single Redis Lua script — one roundtrip, no distributed lock, no check-then-increment race under concurrent bursts.
- [B] SSRF defense in two phases: Upstream targets are validated when routes are defined and re-validated at connection time with IP pinning, covering private subnets, link-local and cloud-metadata addresses, and DNS rebinding.
- [C] Control plane never public: The admin API runs on :4001 inside the internal Docker network and is never exposed through Caddy. API keys are stored only as SHA-256 hashes.
108 Vitest tests across 11 suites  ·  Live Demo  ·  Source Code
CodeForge — Sandboxed Code Execution Platform
Next.js · TypeScript · BullMQ · Redis · Docker · Prisma · Neon PostgreSQL · Azure · Caddy

Queue-backed code execution with Docker isolation, worker heartbeats, and deterministic verdict aggregation across 5 languages (JavaScript, Python, C, C++, Java).

Interface: Next.js app with a code editor, submission history, and live status polling.

  CLIENT / CODE EDITOR  (JavaScript · Python · C · C++ · Java)
              │
              ▼
  ┌─ FRONTEND + API ROUTES (Next.js) ────────────────────────────────────────────────────────────┐
  │ Code editor · submission history · status polling                                            │
  │ POST /api/run     ──►  ephemeral run with custom input                                       │
  │ POST /api/submit  ──►  graded run against hidden test cases                                  │
  └──────────────────────────────────────────────────────────────────────────────────────────────┘
              │ enqueue job
              ▼
  ┌─ REDIS + BULLMQ QUEUE ───────────────────────────────────────────────────────────────────────┐
  │ job state · retry metadata                                                                   │
  │ worker heartbeats (TTL keys) → queue-health endpoints                                        │
  └──────────────────────────────────────────────────────────────────────────────────────────────┘
              │ worker pulls job
              ▼
  ┌─ NODE.JS WORKER ─────────────────────────────────────────────────────────────────────────────┐
  │ Docker sandbox [A]   one ephemeral container per execution                                   │
  │   non-root user · read-only root filesystem · hard timeout kill · output size bounded        │
  │                                                                                              │
  │ Verdict aggregation [B]   deterministic precedence                                           │
  │   Accepted · Compile Error · Timeout · Runtime Error · Wrong Answer                          │
  │   TIMEOUT outranks WRONG_ANSWER                                                              │
  └──────────────────────────────────────────────────────────────────────────────────────────────┘
              │ results + verdicts                          │ on failure [C]
              ▼                                             ▼
  ┌─ NEON POSTGRESQL (via Prisma) ─────────────┐    ┌─ RETRY → DEAD LETTER QUEUE ────────────────┐
  │ problems · test cases · submissions        │    │ failed jobs are retried, then parked       │
  │ verdicts · pooling tuned for               │    │ endpoints: inspect · replay · delete       │
  │ scale-to-zero autosuspend                  │    │                                            │
  └────────────────────────────────────────────┘    └────────────────────────────────────────────┘
Key decisions:
- [A] Untrusted code, contained: Every execution runs in a newly spawned, ephemeral Docker container with a hard timeout, output truncation, a non-root user, and a read-only root filesystem.
- [B] Run vs Submit: An ephemeral Run path with custom input vs. a DB-backed Submit path tested against hidden cases, with a deterministic verdict hierarchy (TIMEOUT strictly outranks WRONG_ANSWER).
- [C] Designed for failure: Redis TTL heartbeats feed queue-health endpoints, failed jobs land in an inspectable Dead Letter Queue, and dedicated scripts test infinite loops, stdout flooding, orphan-container cleanup, and worker hard-kill recovery.
Live Demo  ·  Source Code
Projects
Project	What it solves	Stack	Key Outcomes
GateForge	Multi-tenant API gateway: auth, rate limiting, routing, and observability	TypeScript, Node.js, Express, Redis (Lua), PostgreSQL, Next.js, Docker, Azure	Atomic Lua rate limiting · two-phase SSRF defense · plane isolation · 108 tests
CodeForge	Sandboxed multi-language code execution with deterministic verdict aggregation	Next.js, TypeScript, BullMQ, Redis, Docker, Prisma, Neon PostgreSQL, Azure	Container isolation · deterministic verdicts · DLQ and worker heartbeats · abuse-tested
ApexDrive	Subscription rewards platform with a monthly draw engine and charitable giving	React, Vite, Node.js, Supabase, PostgreSQL, Stripe	4 Stripe webhook lifecycle events · Row Level Security · DB-trigger score window · admin analytics
ProjectHub	Real-time collaborative workspace with multi-user sync	React, Node.js, Express, Socket.IO, MongoDB Atlas, JWT	Drag-and-drop React UI · live WebSocket sync · RBAC · tokenized password reset
QueueForge	Asynchronous distributed job processing with worker isolation	Node.js, BullMQ, Redis, MongoDB, Docker Compose, AWS EC2	Exponential backoff · retryable/permanent failure classification · 10/10 jobs completed in documented burst test


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
