<h1 align="center">Manas Bhardwaj</h1>

<h3 align="center">
  Full-Stack Engineer &nbsp;·&nbsp; Distributed Systems &nbsp;·&nbsp; Real-Time Infrastructure &nbsp;·&nbsp; MERN Stack
</h3>

<p align="center">
  <a href="https://www.linkedin.com/in/manas-bhardwaj-/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  <a href="mailto:manasbhardwaj264@gmail.com">
    <img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" />
  </a>
  <a href="https://queueforge.duckdns.org">
    <img src="https://img.shields.io/badge/QueueForge-Live-22c55e?style=for-the-badge" />
  </a>
  <a href="https://codeforgeapp.duckdns.org">
    <img src="https://img.shields.io/badge/CodeForge-Live-3b82f6?style=for-the-badge" />
  </a>
</p>

---

I build backend systems that hold up under real load — distributed job queues, real-time WebSocket platforms, and Docker-sandboxed execution pipelines. My stack is the MERN ecosystem, BullMQ, Redis, Docker, and AWS. I care about the gap between a working demo and infrastructure that ships.

Finishing B.Tech CSE at Lovely Professional University (May 2026).  
Open to **Backend** and **Full-Stack** roles at ambitious product teams.

---

## Featured Architecture — QueueForge

> **Distributed Asynchronous Job Processing System**  
> `Node.js · BullMQ · Redis · MongoDB · Docker Compose · AWS EC2`

**The engineering problem:** Build a job engine that handles massive, unpredictable task volume — reliably — even during server overload, Redis downtime, or worker crashes. No job loss. Fast recovery. Full observability.

```
┌─────────────────────────────────────────────────────┐
│                   API Gateway                       │
│            (Express.js + REST endpoint)             │
└────────────────────┬────────────────────────────────┘
                     │ enqueue
                     ▼
┌─────────────────────────────────────────────────────┐
│              Redis Queue (BullMQ)                   │
│     Priority lanes · Retry state · Job metadata     │
└────────┬──────────────────────────┬─────────────────┘
         │                          │
         ▼                          ▼
┌────────────────┐        ┌────────────────┐
│   Worker Pod 1 │        │   Worker Pod N │  ← Docker-isolated
│  (job type A)  │  ...   │  (job type X)  │  ← per job type
└────────┬───────┘        └───────┬────────┘
         │                        │
         ▼                        ▼
┌─────────────────────────────────────────────────────┐
│          Exponential Backoff Retry Engine           │
│   Attempt 1 → 2s · Attempt 2 → 4s · ... → DLQ      │
└─────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│       Structured Logging + Job Status Telemetry     │
└─────────────────────────────────────────────────────┘
```

**Measured outcomes:**
- ✅ **10,000+ tasks** processed at **99.9% execution reliability**
- ✅ **85% reduction** in task failures via exponential backoff + dead-letter routing
- ✅ **50+ concurrent workers** containerized and deployed on AWS EC2

**[Live Demo](https://queueforge.duckdns.org) · [Source Code](https://github.com/ManasBhardwaj07/QueueForge)**

---

## Projects

| Project | What it solves | Stack | Key Outcome |
|---|---|---|---|
| **[QueueForge](https://github.com/ManasBhardwaj07/QueueForge)** | Distributed async job processing | Node.js, BullMQ, Redis, Docker, AWS | 10k+ tasks · 99.9% reliability · 85% failure reduction |
| **[CodeForge](https://github.com/ManasBhardwaj07/CodeForge)** | Sandboxed multi-language code execution | Next.js, TypeScript, BullMQ, Docker, PostgreSQL | 5 languages · Docker sandbox · DLQ recovery |
| **[ProjectHub](https://github.com/ManasBhardwaj07/ProjectHub-Frontend)** | Real-time collaboration workspace | MERN, Socket.IO, Redis | Sub-100ms sync · 50+ users · 60% DB read reduction |
| **[ApexDrive](https://github.com/ManasBhardwaj07/ApexDrive)** | Subscription golf rewards platform | React, Node.js, Supabase, Stripe | Stripe webhooks · RLS policies · admin analytics |

---

## Tech Stack

**Languages**  
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)

**Backend & Queue Infrastructure**  
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=node.js&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square)
![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square)
![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white)

**Frontend**  
![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)

**Databases**  
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-005C84?style=flat-square&logo=mysql&logoColor=white)

**DevOps & Cloud**  
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

---

<p align="center">
  <i>Building systems that stay up. Open to backend & full-stack roles at high-impact engineering teams.</i>
</p>
