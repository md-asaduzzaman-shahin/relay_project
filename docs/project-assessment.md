# Relay — Project Assessment & Roadmap

> This document covers sections A through G as defined in the Master Prompt.
> It is the foundation we agree on **before any code is written**.

---

## A. Project Assessment

### 1. What makes this project technically impressive?

| Signal | Why it impresses |
|---|---|
| **Agentic RAG with sufficiency loop** | Most portfolio RAG projects do single-shot "retrieve → generate." Relay's agent evaluates whether retrieved chunks actually answer the question, and re-queries if not. This is a real engineering problem (loop termination, quality thresholds, latency budget) and demonstrates you understand the difference between a demo and a system. |
| **MCP tool-calling** | Instead of hardcoded function calls, the agent discovers available tools through the Model Context Protocol. This is the current industry-standard pattern (adopted by Anthropic, OpenAI, and Salesforce) and shows you're building with the ecosystem, not around it. |
| **Real Salesforce integration** | Using actual Salesforce Case/Account objects via REST API + OAuth, not a mock schema. This alone separates you from 95% of portfolio projects that mention Salesforce but never touch it. |
| **Human-in-the-loop with confidence routing** | A confidence threshold that decides auto-send vs. human review. This is a production AI pattern (not just calling an LLM and showing the output) and maps directly to how Agentforce Service Agent works. |
| **Observability as a first-class feature** | Logging every run's sources, tokens, cost, latency, and outcome. This is the single biggest differentiator 2026 AI-engineering hiring guides identify — it proves you've *operated* an LLM system, not just prototyped one. |
| **End-to-end system** | Ticket in → grounding → retrieval → reasoning → action → audit trail. Not a script, not a notebook — a system with a frontend, backend, database, queue, and external integrations. |

### 2. What makes it relevant to Salesforce?

| Relay concept | Salesforce equivalent | Why it matters |
|---|---|---|
| Ticket ingestion + classification | Service Cloud Case routing | Same problem: inbound request → categorize → route |
| Customer account grounding | Data 360 / unified customer profile | Same principle: don't answer without context |
| RAG over knowledge base | Einstein Knowledge (Agentforce) | Same architecture: embed articles, retrieve at inference |
| Confidence-based routing | Agentforce Service Agent deflection | Same pattern: auto-resolve high-confidence, escalate low |
| MCP tool server | Agentforce Actions / MuleSoft integrations | Same protocol: agent discovers and calls tools dynamically |
| PII awareness | Einstein Trust Layer | Same concern: what data reaches the LLM? |
| Observability + evals | AI governance / trust metrics | Same need: prove the system works, not just hope |
| Human review console | Agent Console / Omni-Channel | Same workflow: human reviews, approves, or overrides AI |
| OAuth integration | Connected Apps / Salesforce Identity | Same auth pattern used across the entire Salesforce ecosystem |

**Honest framing**: Relay is not a Salesforce product. It is a graduate-built system that demonstrates the same engineering patterns Salesforce uses at production scale. You should say: *"I built my own version of the same architecture to understand it deeply"* — never *"I rebuilt Agentforce."*

### 3. What parts are genuinely necessary?

These are load-bearing components — remove any one and the project stops being a coherent system:

- FastAPI backend with ticket ingestion endpoint
- PostgreSQL database with account/ticket/KB data
- Embedding + vector search (pgvector) for RAG
- LLM integration (Claude) with structured prompts
- Confidence scoring + routing logic (auto-send vs. escalate)
- Human review interface (Next.js)
- Basic observability logging (every run logged to DB)
- Salesforce Dev Org integration (real Case/Account via REST API + OAuth)

### 4. What parts are optional?

| Feature | Value | Verdict |
|---|---|---|
| MCP tool server | High interview value, moderate build effort | **Tier B — build after core works** |
| Agentic (iterative) RAG | High interview value, genuinely hard | **Tier B — build after single-shot RAG works** |
| Redis queue for async ingestion | Moderate value, teaches queues | **Tier B — add once sync path works** |
| Slack escalation notifications | Low-moderate value, easy | **Tier C — nice polish** |
| Chat widget / multi-channel | Low value for effort | **Tier D — cut** |
| Fine-tuned classifier for routing | Low value for effort at this stage | **Tier D — cut** |
| Customer-facing self-serve portal | Low value, large effort | **Tier D — cut** |
| OpenTelemetry tracing | Moderate value but complex setup | **Tier C — structured logs first, OTel if time** |
| CI/CD (GitHub Actions) | Moderate value, demonstrates maturity | **Tier B** |
| Docker | High value, expected for any deployed project | **Tier A** |

### 5. Which parts are likely difficult for a Python-focused graduate?

| Component | Difficulty | Why |
|---|---|---|
| **Next.js / React / TypeScript frontend** | 🔴 High | Completely different paradigm from Python. JSX, component model, hooks, state management, TypeScript types — all new. This will consume the most calendar time of anything in the project. |
| **OAuth 2.0 with Salesforce** | 🔴 High | OAuth is conceptually confusing (multiple grant types, token refresh, redirect flows) and Salesforce's Connected App setup has its own quirks. |
| **Agentic RAG loop** | 🔴 High | Designing the sufficiency check, preventing infinite loops, managing latency budget — this is genuinely hard even for experienced engineers. |
| **MCP tool server** | 🟡 Medium | The protocol itself is straightforward, but designing clean tool schemas, handling partial failures, and testing tool interactions adds real complexity. |
| **Docker + deployment** | 🟡 Medium | Conceptually approachable but lots of "why isn't this working" debugging the first time (networking, volumes, env vars, build caching). |
| **pgvector / embeddings** | 🟡 Medium | The vector math is abstracted away, but understanding chunk size, embedding models, similarity thresholds, and retrieval quality takes iteration. |
| **Redis + async queue** | 🟢 Low-Medium | RQ (Redis Queue) is one of the simplest queue systems. The hard part is understanding *why* you need async, not how to use it. |
| **FastAPI backend** | 🟢 Low | Very Python-friendly, excellent docs, you'll pick this up fast. |
| **PostgreSQL / SQL** | 🟢 Low | You have intermediate SQL. The data model is straightforward. pgvector adds one new concept (vector columns + similarity search). |

### 6. Which parts are likely to consume the most time?

1. **Frontend (Next.js/React/TypeScript)** — ~30% of total project time. Learning curve is steep if you've never built a React app.
2. **OAuth + Salesforce integration** — ~15%. Debugging OAuth flows is notoriously time-consuming.
3. **RAG tuning** — ~15%. Getting retrieval quality right (chunk sizes, prompts, thresholds) is iterative, not one-shot.
4. **Agentic RAG loop** — ~10%. The sufficiency check logic and loop termination require careful design.
5. **Deployment + Docker** — ~10%. First-time containerization always takes longer than expected.

### 7. Which parts provide the highest interview value?

**Tier 1 — Must be able to explain deeply:**
- RAG architecture (why embeddings, how retrieval works, chunk strategy, quality)
- Confidence-based routing (threshold design, auto-send vs. escalate trade-off)
- Salesforce integration (OAuth flow, REST API, why real CRM data matters)
- Observability (what you log, why, how you measure the system)
- System architecture (draw it on a whiteboard, explain every arrow)

**Tier 2 — Strong differentiators:**
- MCP tool-calling (what it is, why the industry adopted it, how your agent uses it)
- Agentic RAG (sufficiency loop, when to stop, trade-offs vs. single-shot)
- Human-in-the-loop design (why, when, how the override feeds back into the system)

**Tier 3 — Good to mention:**
- Async ingestion (why queues, Redis vs. alternatives)
- Docker / deployment (shows you can ship, not just develop locally)
- CI/CD (shows engineering maturity)

### 8. Which parts can be simplified without weakening the project?

| Component | Simplification | Impact |
|---|---|---|
| **Frontend** | Functional review console only — no fancy animations, no complex state management. A table of pending tickets, a detail view showing draft + sources + confidence, approve/edit/reject buttons. | Minimal — interviewers care about the system, not CSS. |
| **Auth** | JWT with hardcoded roles (agent, reviewer, admin) — no registration flow, no social login. | None — RBAC is the important concept, not a signup page. |
| **Async ingestion** | Start with synchronous processing. Add Redis queue only after the sync path works. | None — the queue is about scalability, not correctness. |
| **Multi-channel** | Simulate all ticket sources as JSON POSTs to one endpoint. No real email parsing, no chat widget. | None — the agent logic is identical regardless of source. |
| **Analytics dashboard** | A single page showing auto-send rate, override rate, avg latency, avg cost over time. Not a full BI tool. | None — the data collection matters more than the visualization. |
| **Salesforce data** | 5–10 Accounts, 20–30 Cases in your dev org. You don't need thousands of records to demonstrate the integration. | None. |

### 9. What should I explicitly NOT build?

- ❌ **Real email parsing / IMAP integration** — enormous effort, zero additional interview value over simulated tickets
- ❌ **Chat widget with WebSocket** — complex real-time frontend, not the point of this project
- ❌ **Fine-tuned ML classifier** — training a model for routing is interesting but a separate project; prompt-based classification is sufficient
- ❌ **Customer-facing portal** — your users are support agents, not end customers
- ❌ **Kubernetes** — you're one developer deploying one system; Docker Compose is the right scope
- ❌ **Kafka / RabbitMQ** — Redis + RQ is sufficient; Kafka is for event streams at scale you don't have
- ❌ **Multiple LLM providers / model switching** — use Claude, use it well, explain why
- ❌ **Full Salesforce Apex / Lightning development** — you're integrating *with* Salesforce via API, not building *on* the platform
- ❌ **Mobile app** — the review console is a web app

### 10. What should I know deeply enough to explain in an interview?

#### Must Learn (will be asked directly)
- How RAG works end-to-end (embeddings → indexing → retrieval → generation)
- How your agent decides whether to auto-send or escalate
- How OAuth 2.0 works (specifically the flow you used with Salesforce)
- How MCP works and why it exists
- What observability means and what you measure
- How you evaluated the system's quality (not "it seemed to work")
- Your data model and why you structured it that way
- How you handle failures (API down, bad LLM response, missing data)

#### Should Learn (likely follow-up questions)
- Embedding models: how they work conceptually, dimensions, trade-offs
- Vector similarity: cosine similarity, why it works for semantic search
- Token economics: how much a request costs, what drives cost
- Prompt engineering: how you structured prompts, why
- Async processing: why queues, what happens under load
- Docker: what a container is, how it differs from a VM, your Dockerfile
- JWT: how token-based auth works, what's in the payload, expiry

#### Nice to Know (shows depth if asked)
- pgvector internals: IVFFlat vs. HNSW indexes, approximate vs. exact search
- LLM sampling: temperature, top-p, how they affect output
- Distributed systems basics: CAP theorem, eventual consistency (conceptual only)
- Salesforce platform architecture at a high level
- How Agentforce Service Agent works internally (from public documentation)

---

## B. Final Recommended Scope

### Tier A — Essential (must work for the project to be credible)

| # | Feature | Why essential |
|---|---|---|
| A1 | FastAPI backend with ticket ingestion endpoint | The entry point of the entire system |
| A2 | PostgreSQL database (tickets, accounts, KB articles, audit log) | Persistent state for everything |
| A3 | pgvector for KB embeddings + semantic search | The retrieval half of RAG |
| A4 | Claude LLM integration with structured prompts | The generation half of RAG |
| A5 | Customer context grounding (pull account data before answering) | The key differentiator vs. a generic chatbot |
| A6 | Confidence score + routing (auto-send ≥ threshold, escalate below) | Human-in-the-loop pattern |
| A7 | Next.js review console (ticket list, draft view, approve/edit/reject) | The human interface to the system |
| A8 | Salesforce Dev Org integration (read Cases/Accounts via REST API + OAuth) | The Salesforce relevance signal |
| A9 | Structured observability logging (every run → DB: sources, tokens, cost, latency, outcome) | The "operated an LLM system" signal |
| A10 | Docker Compose for local dev + deployment | Proves you can ship |
| A11 | Synthetic ticket + KB dataset | Realistic test data without PII concerns |
| A12 | Basic evaluation harness (run N tickets, measure auto-send precision) | Real metrics, not fake claims |

### Tier B — Strong Differentiators (build after Tier A works)

| # | Feature | Why valuable |
|---|---|---|
| B1 | MCP tool server (check_order_status, check_plan_tier, create_refund_flag, escalate_to_human) | Demonstrates the industry-standard tool-calling protocol |
| B2 | Agentic RAG with sufficiency check (re-query if retrieval is insufficient) | Demonstrates advanced AI engineering judgment |
| B3 | Redis + RQ async ingestion pipeline | Demonstrates understanding of async processing and queues |
| B4 | CI/CD with GitHub Actions (lint, test, build, deploy) | Demonstrates engineering maturity |
| B5 | Simple analytics view (auto-send rate, override rate, latency/cost over time) | Makes observability visible |

### Tier C — Optional Polish (only if Tier A + B are solid)

| # | Feature | Why optional |
|---|---|---|
| C1 | Slack webhook for escalation notifications | Easy win, shows integration breadth |
| C2 | OpenTelemetry traces (in addition to structured logs) | Shows awareness of production observability standards |
| C3 | PII detection / masking before LLM call | Maps to Einstein Trust Layer, shows security awareness |

### Tier D — Cut Completely

| Feature | Why cut |
|---|---|
| Real email / IMAP ingestion | Huge effort, no additional architectural insight |
| Chat widget with WebSocket | Complex real-time frontend, separate project |
| Fine-tuned routing classifier | ML training is a separate skill; prompt-based is sufficient |
| Customer-facing portal | Wrong user persona for this project |
| Kubernetes | Over-engineered for a solo-dev project |
| Kafka / RabbitMQ | Redis + RQ is sufficient at this scale |
| Multiple LLM providers | Adds complexity without insight |
| Mobile app | Web console is sufficient |

---

## C. Architecture We Will Actually Build

### System Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TICKET SOURCES                               │
│         (simulated: web form POST, email JSON, chat JSON)           │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ HTTP POST (JSON)
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     FastAPI BACKEND                                  │
│                                                                     │
│  ┌──────────────┐   ┌──────────────┐   ┌────────────────────────┐  │
│  │  /api/tickets │   │  /api/review │   │  /api/analytics        │  │
│  │  (ingestion)  │   │  (console)   │   │  (observability data)  │  │
│  └──────┬───────┘   └──────┬───────┘   └────────┬───────────────┘  │
│         │                  │                     │                  │
│         ▼                  ▼                     ▼                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    SERVICE LAYER                             │    │
│  │                                                             │    │
│  │  TicketService │ AgentService │ ReviewService │ AnalyticsS. │    │
│  └────────┬──────────────┬───────────────┬──────────────┬─────┘    │
│           │              │               │              │          │
│           ▼              ▼               ▼              ▼          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    DATA LAYER                               │    │
│  │          PostgreSQL + pgvector (single database)            │    │
│  │                                                             │    │
│  │  tickets │ accounts │ kb_articles │ kb_embeddings │         │    │
│  │  agent_runs │ tool_calls_log │ human_reviews               │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
     ┌──────────────┐ ┌──────────┐ ┌───────────────────┐
     │  Claude API   │ │  MCP     │ │  Salesforce       │
     │  (Anthropic)  │ │  Tool    │ │  Dev Org          │
     │               │ │  Server  │ │  (REST API+OAuth) │
     │  - Reasoning  │ │          │ │                   │
     │  - Drafting   │ │  Tools:  │ │  - Cases          │
     │  - Confidence │ │  - check │ │  - Accounts       │
     │    scoring    │ │    order │ │                   │
     │               │ │  - check │ │                   │
     │               │ │    plan  │ │                   │
     │               │ │  - flag  │ │                   │
     │               │ │    refund│ │                   │
     │               │ │  - esc.  │ │                   │
     └──────────────┘ └──────────┘ └───────────────────┘

              ┌──────────────────────────┐
              │   Redis + RQ (Tier B)    │
              │   Async ticket worker    │
              │   (embed + classify)     │
              └──────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    Next.js FRONTEND                                  │
│                                                                     │
│  ┌────────────────┐  ┌─────────────────┐  ┌──────────────────────┐ │
│  │  Ticket Queue   │  │  Review Detail   │  │  Analytics Dashboard │ │
│  │  (pending list) │  │  (draft+sources  │  │  (auto-send rate,    │ │
│  │                 │  │   +confidence,   │  │   latency, cost,     │ │
│  │                 │  │   approve/edit/  │  │   override rate)     │ │
│  │                 │  │   reject)        │  │                      │ │
│  └────────────────┘  └─────────────────┘  └──────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### The 5 Most Important Data Flows

**Flow 1: Ticket Ingestion → Draft Response (the core loop)**
```
1. Simulated source POSTs ticket JSON to /api/tickets
2. Backend validates + stores ticket in PostgreSQL
3. Backend pulls customer account data from Salesforce (or local cache)
4. Backend embeds the ticket text → vector search against KB embeddings
5. Backend assembles context: ticket + account data + retrieved KB chunks
6. Claude receives context + instruction → generates draft + confidence score
7. If confidence ≥ threshold → auto-send, log as "auto_resolved"
8. If confidence < threshold → store as "pending_review", route to console
9. Entire run logged: sources used, tokens, cost, latency, outcome
```

**Flow 2: Human Review**
```
1. Reviewer opens Next.js console → sees pending tickets
2. Clicks a ticket → sees: original ticket, customer context, AI draft,
   retrieved sources with relevance scores, confidence score
3. Reviewer chooses: Approve (send as-is) / Edit (modify draft) / Reject (manual reply)
4. Decision logged with reviewer ID and timestamp
5. Override data feeds analytics (was the AI right? was the draft useful?)
```

**Flow 3: MCP Tool Call (Tier B)**
```
1. During reasoning, Claude determines it needs data not in the prompt
   (e.g., "what's this customer's current order status?")
2. Agent calls MCP tool server → check_order_status(account_id)
3. MCP server queries Salesforce REST API (or local DB)
4. Result returned to agent → incorporated into reasoning
5. Tool call logged: which tool, inputs, output, latency
```

**Flow 4: Salesforce OAuth + Data Sync**
```
1. On first setup: OAuth 2.0 authorization code flow with Salesforce
2. Backend receives access_token + refresh_token
3. For each ticket: GET /services/data/vXX.0/sobjects/Account/{id}
4. If token expired: use refresh_token to get new access_token
5. Account data cached locally to avoid redundant API calls
```

**Flow 5: Observability + Evaluation**
```
1. Every agent run writes to agent_runs table:
   ticket_id, retrieved_chunks, tokens_used, cost, latency_ms,
   confidence_score, outcome (auto_sent / escalated / human_overridden)
2. Analytics endpoint aggregates: auto-send rate, precision
   (% of auto-sends that were NOT overridden), avg cost, avg latency
3. Evaluation harness: run N synthetic tickets → measure precision, recall
   of auto-send decisions against known-correct answers
```

---

## D. Technology Stack with Justification

| Layer | Technology | Why this? | Why not the alternative? | What problem does it solve? | Complexity cost | What if I remove it? | Interview question that exposes understanding |
|---|---|---|---|---|---|---|---|
| **Backend** | Python + FastAPI | Async-friendly, dominant in AI-engineering roles, excellent docs, you already know Python | Django (heavier, more opinionated than needed), Flask (no async, less modern) | Handles HTTP requests, routing, validation, business logic | Low — very natural for Python developers | No backend. No project. | "Why FastAPI over Flask? What does async buy you?" |
| **Database** | PostgreSQL | Battle-tested relational DB, supports pgvector for embeddings, one DB for everything | MongoDB (no relations, weaker for structured data), SQLite (no concurrent access, no pgvector) | Persistent storage for all structured data | Low — standard SQL, well-documented | No persistence. Everything disappears on restart. | "Why not a separate vector database like Pinecone?" |
| **Vector search** | pgvector (PostgreSQL extension) | Runs inside your existing Postgres — no separate service to manage | Pinecone/Weaviate/Chroma (separate service, separate billing, separate ops) | Semantic similarity search over KB articles and past tickets | Low — one `CREATE EXTENSION`, vector column type, similarity operator | No RAG. The agent can't search the knowledge base. | "How does cosine similarity work? When would pgvector not be enough?" |
| **ORM** | SQLAlchemy + Alembic | Type-safe queries, migration management, works perfectly with FastAPI | Raw SQL (no migration tracking, error-prone), Prisma (TypeScript-oriented) | Maps Python objects to database rows, tracks schema changes | Medium — learning the ORM syntax takes a bit | You write raw SQL everywhere and manage schema changes manually. | "What's a database migration? Why do you need them?" |
| **LLM** | Claude API (Anthropic) | Native tool-use support, strong structured output, MCP ecosystem | OpenAI GPT (also good, but MCP is Anthropic's protocol), local models (too slow, too weak) | Reasoning, drafting responses, confidence scoring | Low — it's an API call | No AI. The project is just a ticketing system. | "How do you handle it when Claude hallucinates? What's your fallback?" |
| **Tool calling** | MCP (Model Context Protocol) | Industry standard adopted by Anthropic + Salesforce; agent discovers tools dynamically | Hardcoded function calls (works but doesn't demonstrate the protocol) | Agent can take actions (check orders, flag refunds) without hardcoded integrations | Medium — need to implement an MCP server | Agent can only read, not act. Tools are hardcoded if they exist at all. | "What is MCP? Why not just call functions directly?" |
| **CRM integration** | Salesforce Developer Edition + REST API + OAuth 2.0 | Free, real Salesforce objects, directly relevant to target employer | Mock data only (loses the entire Salesforce relevance story) | Real Case/Account data, real OAuth flow, real API interaction | Medium-High — OAuth setup and Salesforce admin config | Project loses its primary Salesforce relevance signal. | "Walk me through your OAuth flow. What happens when the token expires?" |
| **Queue** | Redis + RQ | Simplest production-grade queue for Python; teaches async concepts without Kafka complexity | Celery (heavier, more config), Kafka (massively over-engineered for this) | Async ticket processing so the API doesn't block on embedding + LLM calls | Low-Medium — Redis is one `docker run`, RQ is a few decorators | API blocks on every ticket. Fine for demo, bad for explaining scalability. | "Why not just process tickets synchronously? When would Redis+RQ break down?" |
| **Frontend** | Next.js + TypeScript + Tailwind CSS | React-based (most popular framework), TypeScript adds type safety, Tailwind for rapid styling | Plain React (no SSR, manual routing), Vue (smaller ecosystem), vanilla HTML (not credible for 2026) | Human review console, analytics dashboard | High — biggest learning curve in the project | No UI. You're demo-ing via curl commands. | "Why Next.js instead of plain React? What does server-side rendering give you?" |
| **Auth** | JWT tokens with role-based access | Simple, stateless, well-understood; demonstrates RBAC without a full identity provider | Session-based auth (requires server state), Auth0 (external dependency, hides the learning) | Controls who can view/approve/reject tickets | Low-Medium — JWT libraries handle the crypto | Anyone can access everything. No RBAC to discuss. | "What's inside a JWT? How do you prevent token theft?" |
| **Containerization** | Docker + Docker Compose | Standard containerization; one command to run the entire stack | Running services directly (works locally but can't deploy consistently) | Reproducible environment, deployment-ready packaging | Medium — first-time Docker is always frustrating | "Works on my machine" but can't deploy or share reliably. | "What's the difference between a Docker image and a container?" |
| **CI/CD** | GitHub Actions | Free for public repos, integrated with GitHub, YAML-based | Jenkins (self-hosted, complex), GitLab CI (requires GitLab) | Automated linting, testing, building on every push | Low-Medium — YAML config, mostly boilerplate | Manual testing and deployment. Fine for a solo project but misses a maturity signal. | "What runs in your CI pipeline? What would you add for a production system?" |
| **Observability** | Structured JSON logs + custom agent_runs table | Simple, queryable, teaches the concepts without infrastructure overhead | Full ELK stack / Datadog (expensive, complex, overkill) | Tracks every AI decision for evaluation and debugging | Low — you're already writing to the DB | You have no idea if the system is working well or poorly. | "How do you know your system is working? What metrics matter?" |

---

## E. Full Learning / Build Roadmap

> Technologies are introduced **only when the project needs them**, not before. Learning materials should not rely solely on generated explanations; each phase should provide official documentation links, curated free video tutorials/walkthroughs, and if possible a quick "Try It First" practical exercise before implementation. **Keep learning lean and pragmatic**—focus strictly on the necessary basics to get features working rather than deep-diving into complex theory. The primary goal is delivering a functional, deployed MVP efficiently.

### Phase 0 — Environment & Foundations
```
Learn:  Command-line tools, Git workflow, Python project structure,
        virtual environments, environment variables
Build:  Project skeleton, Git repo, .env setup, first commit
```

### Phase 1 — Product Design & Architecture
```
Learn:  System design basics, data flow thinking, API design principles
Build:  Architecture document, data model diagram, API contract sketches
        (no code — design only)
```

### Phase 2 — Backend Fundamentals (FastAPI)
```
Learn:  HTTP basics, REST conventions, request/response lifecycle,
        status codes, validation (Pydantic), async/await basics
Build:  FastAPI app with ticket ingestion endpoint, health check,
        basic error handling. Test with curl/httpie.
```

### Phase 3 — Database & Data Modelling (PostgreSQL)
```
Learn:  Relational modelling, tables, primary/foreign keys, indexes,
        SQL basics (SELECT/INSERT/UPDATE/JOIN), SQLAlchemy ORM,
        Alembic migrations
Build:  Database schema (tickets, accounts, kb_articles),
        seed data, CRUD operations through the API.
        Ticket in → stored in DB → retrieved via API.
```

### Phase 4 — RAG Foundation (Embeddings + Vector Search)
```
Learn:  What embeddings are, how vector similarity works,
        chunking strategies, pgvector extension,
        embedding API (e.g. OpenAI or Voyage)
Build:  Embed KB articles → store vectors in pgvector →
        given a ticket, retrieve top-K relevant articles.
        Test retrieval quality manually.
```

### Phase 5 — AI Agent Core (LLM + Drafting + Confidence)
```
Learn:  Claude API, prompt engineering, structured output,
        system/user message roles, token counting, cost estimation,
        confidence scoring strategies
Build:  Agent receives ticket + context + retrieved KB →
        generates draft response + confidence score.
        Route by confidence: auto-send or pending_review.
        Log every run to agent_runs table.
```

### Phase 6 — Frontend (Next.js Review Console)
```
Learn:  React fundamentals (components, props, state, hooks),
        TypeScript basics, Next.js (pages/app router, API routes),
        Tailwind CSS, fetching data from an API
Build:  Review console: ticket queue page, detail page
        (draft + sources + confidence), approve/edit/reject actions.
```

### Phase 7 — Salesforce Integration (OAuth + REST API)
```
Learn:  OAuth 2.0 concepts, Salesforce Connected Apps,
        Salesforce REST API, SOQL basics,
        token refresh flow, API error handling
Build:  Connect to Salesforce Dev Org, read Account/Case data,
        use real CRM data in the grounding step.
```

### Phase 8 — MCP Tool Calling (Tier B)
```
Learn:  Model Context Protocol concepts, tool schemas,
        how Claude discovers and calls tools,
        error handling for tool calls
Build:  MCP server with 3-4 tools (check_order_status,
        check_plan_tier, create_refund_flag, escalate_to_human).
        Agent uses tools during reasoning.
```

### Phase 9 — Async Processing (Redis + RQ) (Tier B)
```
Learn:  Why async processing, message queues concepts,
        Redis basics, RQ workers, job lifecycle
Build:  Ticket ingestion → Redis queue → worker processes
        (embed, classify, run agent) → result stored in DB.
```

### Phase 10 — Authentication & Authorization
```
Learn:  JWT tokens, RBAC concepts, middleware,
        protected routes, token expiry, refresh
Build:  Login endpoint, JWT middleware on backend,
        role-based access (agent vs. reviewer vs. admin),
        protected frontend routes.
```

### Phase 11 — Agentic RAG (Tier B)
```
Learn:  Sufficiency evaluation, iterative retrieval,
        loop termination strategies, latency budgets
Build:  Agent evaluates retrieval quality → re-queries
        with refined search if insufficient → stops when
        confident or budget exhausted.
```

### Phase 12 — Observability & Evaluation
```
Learn:  What to measure, structured logging, evaluation
        datasets, precision/recall for auto-send decisions,
        cost tracking, latency analysis
Build:  Analytics endpoint, evaluation harness (run N tickets,
        compare against expected outcomes), analytics page
        in frontend.
```

### Phase 13 — Docker & Deployment
```
Learn:  Docker concepts, Dockerfile, Docker Compose,
        environment configuration, health checks,
        secrets management, deployment platforms (Render/Fly.io)
Build:  Dockerize backend + frontend + Postgres + Redis,
        deploy to cloud, verify it works in production.
```

### Phase 14 — CI/CD (Tier B)
```
Learn:  CI/CD concepts, GitHub Actions workflow syntax,
        linting, automated tests, build + deploy pipeline
Build:  GitHub Actions workflow: lint → test → build →
        deploy on push to main.
```

### Phase 15 — Portfolio & Interview Preparation
```
Learn:  How to present technical work, how to explain
        trade-offs, how to handle "what would you change?" questions
Build:  Professional README, architecture diagram,
        case study document, demo script,
        interview knowledge map.
```

---

## F. Estimated Learning Difficulty for Each Technology

| Technology | Your starting level | Difficulty to learn for this project | Estimated time to become functional | Notes |
|---|---|---|---|---|
| **FastAPI** | Beginner (know Python well) | 🟢 Easy | 2–3 days | Very Pythonic, excellent docs. You'll feel at home quickly. |
| **PostgreSQL + SQL** | Intermediate | 🟢 Easy | 1–2 days | You know basics. Add JOINs, indexes, and you're set. |
| **SQLAlchemy + Alembic** | Beginner | 🟡 Medium | 3–4 days | ORM concepts take a bit to click. Migrations are straightforward once you see one. |
| **pgvector** | Beginner | 🟡 Medium | 1–2 days | The extension itself is simple. Understanding embedding quality takes iteration. |
| **Embeddings / RAG** | Beginner | 🟡 Medium | 3–5 days | Conceptually approachable, but tuning retrieval quality is iterative. |
| **Claude API** | Beginner | 🟢 Easy | 1–2 days | It's an API call. The hard part is prompt engineering, not the SDK. |
| **Prompt engineering** | Beginner | 🟡 Medium | Ongoing | Not a one-time learning — you'll iterate throughout the project. |
| **MCP** | Beginner | 🟡 Medium | 3–4 days | Protocol is well-documented. Designing good tool schemas takes thought. |
| **React / Next.js** | Beginner | 🔴 Hard | 7–10 days | Biggest learning curve. Component model, hooks, state, TypeScript — all new. |
| **TypeScript** | Beginner (know JS basics) | 🟡 Medium | 3–4 days | Adds types to JS. Frustrating at first, invaluable once it clicks. |
| **Tailwind CSS** | Beginner | 🟢 Easy | 1–2 days | Utility classes. Fast to learn, fast to build with. |
| **OAuth 2.0** | Beginner | 🔴 Hard | 3–5 days | Conceptually confusing. Salesforce-specific setup adds friction. |
| **Salesforce REST API** | Beginner | 🟡 Medium | 2–3 days | Well-documented. Main challenge is Connected App setup and SOQL. |
| **Redis + RQ** | Beginner | 🟢 Easy-Medium | 2–3 days | RQ is deliberately simple. Understanding *why* is more important than *how*. |
| **JWT auth** | Beginner | 🟡 Medium | 2–3 days | Conceptually clear. Implementation is mostly library calls. |
| **Docker** | Beginner | 🟡 Medium | 3–4 days | First time is frustrating. Once you debug one Dockerfile, it clicks. |
| **Docker Compose** | Beginner | 🟢 Easy-Medium | 1–2 days | YAML config. Straightforward once Docker itself makes sense. |
| **GitHub Actions** | Beginner | 🟢 Easy | 1–2 days | YAML-based, lots of examples. Copy-adapt-understand. |
| **Structured logging** | Beginner | 🟢 Easy | 1 day | Python's `logging` + JSON formatter. Simple. |
| **LLM evaluation** | Beginner | 🟡 Medium | 3–4 days | The concepts matter more than the code. Designing good eval datasets is the hard part. |

### Total estimated learning + build time

| Scope | Estimated time |
|---|---|
| **Tier A only** (working system, no MCP/agentic RAG/async) | 6–8 weeks of focused work |
| **Tier A + B** (MCP, agentic RAG, async, CI/CD, analytics) | 10–14 weeks |
| **Tier A + B + C** (Slack, OTel, PII masking) | 12–16 weeks |

These are rough estimates assuming you're working on this alongside job applications and life, not full-time 8-hour days. Adjust based on your actual pace.

---

## G. Phase 0 — Environment & Foundational Knowledge (Detailed)

### Objective

Set up a professional development environment and project structure that you'll use for the entire project. No product code yet — just the foundation.

### Why this phase matters

Every professional project starts here. A clean environment setup means:
- You won't hit "works on my machine" problems later
- You can share the project with others (interviewers!) who can run it
- You establish good habits (environment variables, gitignore, project structure) from day one
- You avoid the common graduate mistake of having secrets in code, messy file structures, and no reproducibility

If you skip this, you'll pay for it later with debugging time that has nothing to do with your actual project.

### What I need to learn first

#### 1. Python Project Structure (for a real application, not a script)

**What it is**: A conventional way to organize Python files so that imports work correctly, tests are separate from source code, and configuration is manageable.

**Why this project needs it**: Relay will have a backend with multiple modules (routes, services, models, config). Without structure, you'll have circular imports, broken paths, and code that only runs from one specific directory.

**What would happen without it**: Import errors, configuration scattered across files, tests that can't find your code, and an interviewer asking "why is everything in one file?"

**Minimum concepts**:
- `src/` or package directory for source code
- `__init__.py` files to make Python packages
- Separate directories for routes, services, models, schemas
- `pyproject.toml` for project metadata and dependencies
- Virtual environments to isolate dependencies

#### 2. Virtual Environments

**What it is**: An isolated Python environment where packages are installed separately from your system Python.

**Why this project needs it**: You'll install FastAPI, SQLAlchemy, anthropic SDK, and many other packages. Without a virtual environment, these pollute your system Python and can conflict with other projects.

**What would happen without it**: Dependency conflicts, "it worked yesterday" bugs, inability to reproduce your environment on another machine or in Docker.

**Minimum concepts**:
- `python -m venv .venv` creates one
- `source .venv/bin/activate` (Linux/Mac) or `.venv\Scripts\activate` (Windows) activates it
- `pip install -r requirements.txt` installs dependencies
- `pip freeze > requirements.txt` captures current state
- (We'll later move to `pyproject.toml` with `pip install -e .` for cleaner dependency management)

#### 3. Environment Variables and `.env` Files

**What it is**: Configuration values (database URLs, API keys, secrets) stored outside your code, loaded at runtime.

**Why this project needs it**: You'll have a Claude API key, a Salesforce OAuth client secret, a database password, and more. None of these should ever appear in your code or Git history.

**What would happen without it**: You accidentally push your API key to GitHub. Someone finds it. You get a surprise bill. This is not hypothetical — it happens constantly.

**Minimum concepts**:
- `.env` file in your project root (never committed to Git)
- `.env.example` file (committed) showing what variables are needed, without values
- `python-dotenv` library or `pydantic-settings` to load them
- `os.environ` or settings objects to access them in code

#### 4. Git Workflow

**What it is**: Version control with meaningful commits, branches for features, and a clean history.

**Why this project needs it**: Your Git history is visible to interviewers. A history of `"initial commit"` → `"final version"` → `"final final"` signals inexperience. A history of `"feat: add ticket ingestion endpoint"` → `"test: add ticket validation tests"` → `"fix: handle duplicate ticket submission"` signals professionalism.

**Minimum concepts**:
- `git init`, `git add`, `git commit -m "message"`
- Conventional commit messages: `feat:`, `fix:`, `test:`, `docs:`, `refactor:`, `chore:`
- `.gitignore` (never commit `.env`, `__pycache__`, `.venv`, `node_modules`)
- `git branch`, `git checkout -b feature/name`, `git merge`
- Push to GitHub: `git remote add origin`, `git push`

### Mini Learning Exercise

Before touching the project, do this 10-minute exercise:

```
1. Create a temporary directory called "git-practice"
2. Initialize a Git repo
3. Create a Python file that reads an environment variable and prints it
4. Create a .env file with a test value
5. Create a .gitignore that excludes .env
6. Make three commits:
   - "chore: initialize project"
   - "feat: add config reader"
   - "chore: add .gitignore"
7. Verify that .env is NOT tracked by Git (git status should not show it)
8. Delete the directory — this was just practice
```

If this was easy, great — you're ready. If anything was confusing, tell me and we'll clarify before proceeding.

### What I will build

The project skeleton for Relay:

```
relay_project/
├── docs/
│   ├── Master Prompt — ... .md    (already exists)
│   └── project-assessment.md      (this document)
├── backend/
│   ├── src/
│   │   └── relay/
│   │       ├── __init__.py
│   │       ├── main.py            (FastAPI app entry point — empty for now)
│   │       ├── config.py          (settings from environment variables)
│   │       ├── routes/
│   │       │   └── __init__.py
│   │       ├── services/
│   │       │   └── __init__.py
│   │       ├── models/
│   │       │   └── __init__.py
│   │       └── schemas/
│   │           └── __init__.py
│   ├── tests/
│   │   └── __init__.py
│   ├── pyproject.toml
│   ├── requirements.txt
│   └── .env.example
├── frontend/                      (empty for now — created in Phase 6)
├── .gitignore
├── .env.example
├── README.md
└── docker-compose.yml             (empty for now — created in Phase 13)
```

### Implementation Sequence

```
1. Create the GitHub repository (public, with README)
2. Clone it locally
3. Create .gitignore (Python + Node + environment files)
4. Create the backend/ directory structure
5. Set up Python virtual environment
6. Install initial dependencies: fastapi, uvicorn, pydantic-settings, python-dotenv
7. Create pyproject.toml with project metadata
8. Create requirements.txt
9. Create .env.example with placeholder variables
10. Create config.py that loads settings from environment
11. Create main.py with a minimal FastAPI app (just a health check endpoint)
12. Run it locally: uvicorn relay.main:app --reload
13. Test the health check: curl http://localhost:8000/health
14. Create initial README.md with project description
15. Commit: "chore: initialize project structure"
16. Push to GitHub
```

### What I should understand afterward

- [ ] Why Python projects use packages (directories with `__init__.py`)
- [ ] Why virtual environments exist and how to use them
- [ ] Why secrets must never be in code or Git
- [ ] How `.env` and `.env.example` work together
- [ ] What a `.gitignore` does and why it matters
- [ ] How to write meaningful commit messages
- [ ] What FastAPI is and why we chose it
- [ ] How to run a FastAPI app locally
- [ ] What a health check endpoint is and why production systems have one

### How to test it

```bash
# 1. Activate your virtual environment
.venv\Scripts\activate

# 2. Run the server
uvicorn relay.main:app --reload

# 3. Test the health check
curl http://localhost:8000/health
# Expected: {"status": "healthy"}

# 4. Visit the auto-generated API docs
# Open http://localhost:8000/docs in your browser

# 5. Verify .env is not tracked
git status
# .env should NOT appear in the output

# 6. Verify the app reads config from environment
# Set a variable in .env, check it loads in config.py
```

### Common mistakes

| Mistake | Why it happens | How to avoid |
|---|---|---|
| Committing `.env` to Git | Forgot `.gitignore` or added `.env` before creating `.gitignore` | Create `.gitignore` FIRST, before creating `.env` |
| Installing packages globally | Forgot to activate virtual environment | Always check your prompt shows `(.venv)` before `pip install` |
| Import errors | Running from wrong directory, missing `__init__.py` | Always run from project root, ensure every package dir has `__init__.py` |
| Hardcoded config values | Seems easier than environment variables | Never. Not even "temporarily." It becomes permanent. |
| Giant first commit | Want to get the "boring" setup over with | Each logical step gets its own commit. This is the habit we're building. |

### Interview questions

**Beginner**: *"Why do you use environment variables instead of putting configuration directly in your code?"*

> **Model answer**: "Environment variables separate configuration from code. This means the same codebase can run in different environments (development, staging, production) with different settings — different database URLs, different API keys — without changing any code. It also prevents secrets from being committed to version control, which is a serious security risk. In Relay, I use Pydantic Settings to load environment variables with type validation, so the app fails fast at startup if a required variable is missing rather than crashing at runtime when it's first accessed."

**Intermediate**: *"Explain your project structure. Why did you organize it this way?"*

> **Model answer**: "I used a layered architecture: routes handle HTTP concerns (request parsing, response formatting), services contain business logic, models define database schemas, and schemas define API request/response shapes. This separation of concerns means I can change how a ticket is stored in the database without touching the API endpoint code, or change the API response format without touching business logic. It also makes testing easier — I can unit-test a service function without spinning up an HTTP server."

**Difficult**: *"Your project has a `.env.example` but not a `.env` in the repo. What happens if someone clones your repo and tries to run it without creating a `.env`? How does your application handle that?"*

> **Model answer**: "Pydantic Settings will raise a `ValidationError` at startup listing every missing required variable. This is intentional — it's a 'fail fast' pattern. The application tells you exactly what's wrong before it starts serving requests, rather than crashing later when it first tries to use a missing API key. The `.env.example` file documents what variables are needed and serves as a template. In a production deployment, these variables would come from the platform's secrets manager, not a `.env` file."

---

## What Happens Next

This document covers sections A through G. Here is what I recommend:

1. **You review this document.** Flag anything you disagree with, find confusing, or want to change.
2. **We agree on scope.** Particularly: are you comfortable with the Tier A/B/C/D split? Is there anything in Tier D you feel strongly about keeping?
3. **We start Phase 0.** I'll walk you through the implementation sequence above, teaching each concept as we go.

> **Important**: Do not worry about the total estimated time (10–14 weeks for A+B). We will move at your pace. Some phases will go faster than estimated if you pick things up quickly. The roadmap is a guide, not a deadline.

When you're ready, tell me to proceed with Phase 0 and we'll start building.
