<h1 align="center">Manas Bhardwaj</h1>

<h3 align="center">Software Engineer &nbsp;·&nbsp; Backend Systems &nbsp;·&nbsp; Distributed Systems &nbsp;·&nbsp; Full-Stack</h3>

<p align="center">
  <a href="https://www.linkedin.com/in/manas-bhardwaj-/"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white" /></a>
  &nbsp;
  <a href="mailto:manasbhardwaj264@gmail.com"><img alt="Email" src="https://img.shields.io/badge/Email-D14836?style=flat-square&logo=gmail&logoColor=white" /></a>
  &nbsp;
  <a href="https://gateforgeapp.duckdns.org"><img alt="GateForge Live Demo" src="https://img.shields.io/badge/GateForge-Live-0ea5e9?style=flat-square" /></a>
  &nbsp;
  <a href="https://codeforgeapp.duckdns.org"><img alt="CodeForge Live Demo" src="https://img.shields.io/badge/CodeForge-Live-8b5cf6?style=flat-square" /></a>
  &nbsp;
  <a href="https://apex-drive-self.vercel.app"><img alt="ApexDrive Live Demo" src="https://img.shields.io/badge/ApexDrive-Live-f59e0b?style=flat-square" /></a>
</p>

---

I build backend and distributed systems designed for real-world reliability: a programmable API gateway, an isolated code-execution engine, and fault-tolerant job queues. While I build across the entire stack, I focus on the infrastructure layer — rate limiting, container isolation, retry strategies, and failure recovery.

Previously at **Emendo AI** &nbsp;·&nbsp; B.Tech Computer Science, Lovely Professional University.

---

## Featured Systems

### GateForge — Programmable API Gateway

> `TypeScript · Node.js · Express · Redis (Lua) · PostgreSQL · Prisma · Next.js · Docker · Azure · Caddy`
>
> A multi-tenant API gateway built from first principles. Authentication, rate limiting, quota enforcement, routing, and SSRF defense are hand-written rather than delegated to third-party abstractions.

```text
┌─ ADMIN SURFACE ──────────────────────────────────────────────┐
│ Control Plane :4001   internal Docker network only           │
│   plans · tenants · upstreams · routes · API keys            │
│   constant-time bearer auth · IP throttling · audit log      │
│ Dashboard :3000       Next.js, served via Caddy              │
│   metrics · traffic playground · Decision Inspector          │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ config + hashed keys
                                ▼
┌─ STATE ──────────────────────────────────────────────────────┐
│ PostgreSQL 16   source of truth (Prisma)                     │
│ Redis 7         key cache · rate-limit + quota counters      │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ read on every request
                                ▼
┌─ DATA PLANE :4000  ◄── client requests (X-API-Key) ──────────┐
│ 1  Auth     SHA-256 hash → Redis → Postgres fallback         │
│ 2  Tenant   resolve organization + plan                      │
│ 3  Policy   sliding-window limit + monthly quota,            │
│             one atomic Lua call (single roundtrip)           │
│ 4  Route    longest-prefix match                             │
│ 5  SSRF     block private / metadata / rebinding IPs,        │
│             pin the resolved IP                              │
│ 6  Proxy    forward + inject X-Request-Id                    │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ proxied request
                                ▼
┌─ UPSTREAM SERVICES ──────────────────────────────────────────┐
│ Your APIs (demo: mock-orders :5001 · mock-payments :5002)    │
└──────────────────────────────────────────────────────────────┘
```

**What makes this non-trivial:**
- **Atomic rate limiting:** Sliding-window limits and monthly quotas are checked in a single Redis Lua script — one roundtrip, no distributed lock, no check-then-increment race under concurrent bursts.
- **SSRF defense in two phases:** Upstream targets are validated when routes are defined and re-validated at connection time with IP pinning, protecting against private subnets, link-local and cloud-metadata addresses, and DNS rebinding.
- **Port-level plane isolation:** The admin control plane runs on `:4001` inside the internal Docker network and is never exposed through Caddy. Protected by constant-time bearer checks, IP throttling, and audit logging.
- **Key lifecycle:** API keys are stored solely as SHA-256 hashes. Lookups hit Redis first with a PostgreSQL fallback; revoked keys are denied at the gateway entry before any downstream logic runs.
- **108 Vitest tests across 11 suites** covering burst traffic, SSRF evasion vectors (decimal IP encoding, DNS rebinding), and upstream failure modes.
- [Live Demo](https://gateforgeapp.duckdns.org) &nbsp;·&nbsp; [Source Code](https://github.com/ManasBhardwaj07/GateForge)

---

### CodeForge — Sandboxed Code Execution Platform

> `Next.js · TypeScript · BullMQ · Redis · Docker · Prisma · Neon PostgreSQL · Azure · Caddy`
>
> An asynchronous, queue-backed platform running untrusted code across 5 languages (JavaScript, Python, C, C++, Java), engineered for deterministic execution and operational resilience under abusive workloads.

```text
┌─ NEXT.JS FRONTEND ───────────────────────────────────────────┐
│ Code editor · submission history · status polling            │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ POST /api/run  ·  /api/submit
                                ▼
┌─ NEXT.JS API ROUTES ─────────────────────────────────────────┐
│ Auth · run · submit · enqueue job                            │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ job written to Redis
                                ▼
┌─ REDIS · BULLMQ ─────────────────────────────────────────────┐
│ Job state · retry metadata · worker heartbeats (TTL keys)    │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ worker pulls job
                                ▼
┌─ NODE.JS WORKER ─────────────────────────────────────────────┐
│ Docker sandbox   (ephemeral container per execution)         │
│   Languages  JavaScript · Python · C · C++ · Java            │
│   Timeout    hard kill on breach                             │
│   Output     stdout/stderr truncated (no memory exhaustion)  │
│   Identity   non-root user · read-only root filesystem       │
│                                                              │
│ Verdict aggregation   (deterministic precedence)             │
│   Accepted · Compile Error · Timeout · Runtime Error         │
│   · Wrong Answer      TIMEOUT outranks WRONG_ANSWER          │
│                                                              │
│ Failure handling                                             │
│   Retry, then Dead Letter Queue                              │
│   DLQ endpoints: inspect · replay · delete                   │
└──────────────────────────────────────────────────────────────┘
                                │
                                │ results + verdicts
                                ▼
┌─ NEON POSTGRESQL (via Prisma) ───────────────────────────────┐
│ Problems · test cases · submissions · verdicts               │
│ Connection pooling tuned for scale-to-zero autosuspend       │
└──────────────────────────────────────────────────────────────┘
```

**What makes this non-trivial:**
- **Untrusted code isolation:** Each execution runs in a newly spawned, ephemeral Docker container with a hard timeout, output truncation, a non-root user, and a read-only root filesystem.
- **Dual execution paths:** An ephemeral `Run` pipeline with custom input vs. a DB-backed `Submit` pipeline tested against hidden cases, with a deterministic verdict hierarchy (`TIMEOUT` strictly outranks `WRONG_ANSWER`).
- **Worker observability and recovery:** Workers publish TTL heartbeat keys to Redis that feed queue-health endpoints; failed jobs move to a Dead Letter Queue with inspect, replay, and delete APIs.
- **Abuse and recovery testing:** Dedicated scripts cover infinite loops, stdout flooding, orphan-container sweeps (`ops:audit-cleanup`), and sudden worker termination (`ops:recovery-kill-worker`).
- **Serverless database integration:** Connection pooling tuned for Neon PostgreSQL's auto-suspend and resume behavior.
- [Live Demo](https://codeforgeapp.duckdns.org) &nbsp;·&nbsp; [Source Code](https://github.com/ManasBhardwaj07/CodeForge)

---

## Projects

| Project | What it solves | Stack | Key Outcomes |
|---|---|---|---|
| **[GateForge](https://github.com/ManasBhardwaj07/GateForge)** | Multi-tenant API gateway: auth, rate limiting, routing, and observability | TypeScript, Node.js, Express, Redis (Lua), PostgreSQL, Next.js, Docker, Azure | Atomic Lua rate limiting · two-phase SSRF defense · plane isolation · 108 tests |
| **[CodeForge](https://github.com/ManasBhardwaj07/CodeForge)** | Sandboxed multi-language code execution with deterministic verdict aggregation | Next.js, TypeScript, BullMQ, Redis, Docker, Prisma, Neon PostgreSQL, Azure | Container isolation · deterministic verdicts · DLQ and worker heartbeats · abuse-tested |
| **[QueueForge](https://github.com/ManasBhardwaj07/QueueForge)** | Asynchronous distributed job processing with worker isolation | Node.js, BullMQ, Redis, MongoDB, Docker Compose, AWS EC2 | Priority lanes · exponential backoff · Dead Letter Queue |
| **[ProjectHub](https://github.com/ManasBhardwaj07/ProjectHub)** | Real-time collaborative workspace with multi-user sync | React, Node.js, Express, Socket.IO, MongoDB Atlas, JWT | Live WebSocket sync · RBAC · tokenized password reset |
| **[ApexDrive](https://github.com/ManasBhardwaj07/ApexDrive)** | Subscription rewards platform with a monthly draw engine and charitable giving | React, Vite, Node.js, Supabase, PostgreSQL, Stripe | 4 Stripe webhook lifecycle events · Row Level Security · DB-trigger score window · draw-engine tests |

---

## Tech Stack

**Languages**<br>
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white)

**Backend & Queue Infrastructure**<br>
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=nodedotjs&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square&logo=express&logoColor=white)
![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square)
![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white)

**Frontend**<br>
![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white)
![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square&logo=framer&logoColor=white)

**Databases & ORM**<br>
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)
![Neon](https://img.shields.io/badge/Neon-00E599?style=flat-square&logo=neon&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=black)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white)

**DevOps, Cloud & Testing**<br>
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square)
![Caddy](https://img.shields.io/badge/Caddy-1F88C0?style=flat-square&logo=caddy&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)
![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white)

**Payments & Security**<br>
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=flat-square&logo=stripe&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)

---

<p align="center"><i>Open to software engineering roles &nbsp;·&nbsp; Based in India &nbsp;·&nbsp; Let's build something worth maintaining.</i></p>
