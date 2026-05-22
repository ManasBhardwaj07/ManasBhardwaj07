<h1 align="center">Manas Bhardwaj</h1>

<h3 align="center">
  Backend Engineer · Distributed Systems · Real-Time Infrastructure
</h3>

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/manas-bhardwaj-/)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:manasbhardwaj264@gmail.com)

</div>

---

I build fault-tolerant backend systems — distributed job queues, real-time collaboration engines, and containerized execution pipelines. My stack is Node.js, Spring Boot, Redis, BullMQ, Docker, AWS, and PostgreSQL. I care about the boundary between "working demo" and "production-grade infrastructure."

Currently finishing B.Tech CSE at Lovely Professional University (May 2026).  
Open to **Backend Engineer** and **Full-Stack Engineer** roles where I can build, scale, and take ownership of high-impact infrastructure.

---

## Featured Architecture — QueueForge

> Distributed Asynchronous Job Processing System

**The problem:** Build a job processing engine that handles massive task volume reliably — even under server overload — with no job loss and fast recovery.

**Architecture decisions:**

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
    │   Worker Pod 1 │        │   Worker Pod N │   ← Docker-isolated
    │  (job type A)  │  ...   │  (job type X)  │   ← per job type
    └────────┬───────┘        └───────┬────────┘
             │                        │
             ▼                        ▼
    ┌─────────────────────────────────────────────────────┐
    │          Exponential Backoff Retry Engine           │
    │   Attempt 1 → 2s · Attempt 2 → 4s · ... → DLQ       │
    └─────────────────────────────────────────────────────┘
                         │
                         ▼
    ┌─────────────────────────────────────────────────────┐
    │      Structured Logging + Job Status Telemetry      │
    └─────────────────────────────────────────────────────┘

**Key outcomes:**
- **10,000+ tasks** processed at **99.9% execution reliability**
- **85% reduction** in task failures via exponential backoff + dead-letter routing
- **50+ concurrent workers** containerized with Docker Compose on AWS EC2

**[Live Demo](#) · [Source Code](#)**

---

## Projects

| Project | What it does | Stack | Highlights |
|---|---|---|---|
| **QueueForge** | Distributed async job processing | Node.js, Redis, BullMQ, Docker, AWS | 10k+ tasks · 99.9% reliability · 85% failure reduction |
| **ProjectHub** | Real-time collaboration workspace | MERN, Socket.IO, Redis, AWS | Sub-100ms sync · 50+ users · 60% DB read reduction |
| **CodeForge** | Sandboxed code execution engine | Next.js, TypeScript, BullMQ, Docker, PostgreSQL | Isolated Docker runners · deterministic verdicts |
| **ApexDrive** | Subscription golf rewards platform | React, Node.js, Supabase, Stripe | Stripe webhooks · RLS policies · admin analytics |

---

## Tech Stack

**Languages**
<br>
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?style=flat-square&logo=c%2B%2B&logoColor=white)

**Backend & Infrastructure**
<br>
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=flat-square&logo=node.js&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=spring-boot&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-404D59?style=flat-square)
![BullMQ](https://img.shields.io/badge/BullMQ-EA4335?style=flat-square)
![Redis](https://img.shields.io/badge/Redis-D82C20?style=flat-square&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=flat-square&logo=socketdotio&logoColor=white)

**Databases**
<br>
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-4EA94B?style=flat-square&logo=mongodb&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-005C84?style=flat-square&logo=mysql&logoColor=white)

**Frontend**
<br>
![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)

---

<p align="center">
  <i>Building systems that stay up. Open to backend & full-stack roles at high-impact engineering teams.</i>
</p>
