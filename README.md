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

<p align="center"><b>3</b> live deployments &nbsp;·&nbsp; <b>108</b> tests in GateForge &nbsp;·&nbsp; <b>5</b> projects</p>

---

I build full-stack software — React and Next.js interfaces on top of backend systems I design for failure: an API gateway, a sandboxed code-execution engine, and distributed job queues. I focus on what happens when things break — burst traffic, hostile code, failed jobs. Previously at **Emendo AI**.

---

## Featured Systems

### GateForge — Programmable API Gateway

> `TypeScript · Node.js · Express · Redis (Lua) · PostgreSQL · Prisma · Next.js · Docker · Azure · Caddy`
>
> A multi-tenant API gateway built from first principles. It implements its own request pipeline for authentication, rate limiting, quota enforcement, route matching, and SSRF protection.
>
> **Interface:** Next.js dashboard with live metrics, a traffic playground, and a per-request Decision Inspector.

```mermaid
flowchart TD
    accTitle: GateForge architecture
    accDescr: A client request enters the data plane, which runs auth, tenant, policy, route, SSRF and proxy steps before reaching upstream services. The private control plane writes configuration to PostgreSQL and Redis state, which the data plane reads. A Next.js dashboard provides observability.
    CP["<b>PRIVATE CONTROL PLANE :4001</b><br/>internal Docker network only<br/>plans · tenants · upstreams · routes · API keys<br/>constant-time bearer auth · IP throttling · audit log"]
    CL["<b>CLIENT</b><br/>X-API-Key"]
    ST[("<b>STATE</b><br/>PostgreSQL 16 — source of truth (Prisma)<br/>Redis 7 — key cache · rate-limit + quota counters")]
    subgraph DP["DATA PLANE :4000"]
        direction TB
        A1["<b>1 · Auth</b><br/>SHA-256 hash → Redis → Postgres fallback"] --> A2["<b>2 · Tenant</b><br/>resolve organization + plan"]
        A2 --> A3["<b>3 · Policy</b><br/>sliding-window limit + monthly quota<br/>atomic Redis Lua policy evaluation"]
        A3 --> A4["<b>4 · Route</b><br/>longest-prefix match"]
        A4 --> A5["<b>5 · SSRF</b><br/>block private / metadata / rebinding IPs<br/>pin the resolved IP"]
        A5 --> A6["<b>6 · Proxy</b><br/>forward + inject X-Request-Id"]
    end
    UP["<b>UPSTREAM SERVICES</b><br/>your APIs (demo: mock-orders :5001 · mock-payments :5002)"]
    DASH["<b>DASHBOARD :3000</b><br/>Next.js served via Caddy<br/>metrics · traffic playground · Decision Inspector"]
    CP -->|"config + hashed keys"| ST
    CL --> DP
    ST -->|"runtime lookups + policy counters"| DP
    DP -->|"proxied request"| UP
    UP ~~~ DASH
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
> An asynchronous, queue-backed platform running untrusted code across 5 languages (JavaScript, Python, C, C++, Java), engineered for deterministic execution and operational resilience under abusive workloads.
>
> **Interface:** Next.js app with a code editor, submission history, and live status polling.

```mermaid
flowchart TD
    accTitle: CodeForge architecture
    accDescr: A Next.js frontend submits code through API routes into a Redis and BullMQ queue. A Node.js worker runs each job in an ephemeral Docker sandbox, aggregates a deterministic verdict, and stores results in Neon PostgreSQL. Failed jobs retry and then move to a dead letter queue.
    UI["<b>NEXT.JS FRONTEND</b><br/>code editor · submission history · status polling"]
    API["<b>NEXT.JS API ROUTES</b><br/>auth · run · submit · enqueue job"]
    Q[("<b>REDIS · BULLMQ</b><br/>job state · retry metadata<br/>worker heartbeats (TTL keys)")]
    subgraph W["NODE.JS WORKER"]
        direction TB
        SB["<b>Docker sandbox</b> — ephemeral container per execution<br/>Languages: JavaScript · Python · C · C++ · Java<br/>Timeout: hard kill on breach<br/>Output: stdout/stderr truncated (size bounded)<br/>Identity: non-root user · read-only root filesystem"]
        SB --> VD["<b>Verdict aggregation</b> — deterministic precedence<br/>Accepted · Compile Error · Timeout<br/>Runtime Error · Wrong Answer<br/>TIMEOUT outranks WRONG_ANSWER"]
    end
    DB[("<b>NEON POSTGRESQL</b> (via Prisma)<br/>problems · test cases · submissions · verdicts<br/>connection pooling tuned for scale-to-zero autosuspend")]
    DLQ["<b>Failure handling</b><br/>retry, then Dead Letter Queue<br/>DLQ endpoints: inspect · replay · delete"]
    UI -->|"POST /api/run · /api/submit"| API
    API -->|"job written to Redis"| Q
    Q -->|"worker pulls job"| W
    W -->|"results + verdicts"| DB
    W -.->|"on failure"| DLQ
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
| **Languages** | ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white) ![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white) |
| **Backend & Queues** | ![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=nodedotjs&logoColor=white) ![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square&logo=express&logoColor=white) ![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square) ![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white) ![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white) |
| **Frontend** | ![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwindcss&logoColor=white) ![Framer Motion](https://img.shields.io/badge/Framer_Motion-0055FF?style=flat-square&logo=framer&logoColor=white) |
| **Databases & ORM** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![Neon](https://img.shields.io/badge/Neon_Serverless-00E599?style=flat-square&logo=neon&logoColor=black) ![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=black) ![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white) |
| **Cloud & DevOps** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white) ![Azure VM](https://img.shields.io/badge/Azure_VM-0078D4?style=flat-square) ![AWS EC2](https://img.shields.io/badge/AWS_EC2-232F3E?style=flat-square) ![Caddy](https://img.shields.io/badge/Caddy-1F88C0?style=flat-square&logo=caddy&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white) |
| **Security & Payments** | ![Stripe Webhooks](https://img.shields.io/badge/Stripe_Webhooks-635BFF?style=flat-square&logo=stripe&logoColor=white) ![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white) ![Row Level Security](https://img.shields.io/badge/Row_Level_Security-3FCF8E?style=flat-square&logo=supabase&logoColor=black) ![bcrypt](https://img.shields.io/badge/bcrypt-5A5A5A?style=flat-square) ![SHA-256](https://img.shields.io/badge/SHA--256-6E40C9?style=flat-square) |

---

<p align="center"><i>Systems that hold under burst traffic, contain hostile code, and recover from failure — every claim above links to code you can read.</i></p>
