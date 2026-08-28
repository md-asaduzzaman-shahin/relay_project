# Master Prompt: Teach Me While I Build This Project

## My Context

I am a recent IT/AI graduate preparing for graduate, junior, and entry-level engineering roles, with **Salesforce Futureforce in Sydney as one of my highest-priority targets**.

I am primarily proficient in **Python**. I understand programming fundamentals and have some academic/project experience, but I am **not yet highly experienced with modern production technologies** such as:

* TypeScript
* React / Next.js
* FastAPI at production depth
* PostgreSQL beyond basic usage
* Redis
* message queues
* Docker
* CI/CD
* cloud deployment
* OAuth
* Salesforce APIs
* MCP
* RAG architectures
* LLM evaluation
* observability
* distributed systems
* real-time communication
* enterprise security patterns

I am willing to learn all of these, but I need to **learn them while building the project**, not study every technology beforehand.

I do not want a superficial tutorial where I blindly copy code.

My goal is to reach a point where I can honestly say:

> "I built this system myself, I understand why every major component exists, I understand the important engineering trade-offs, I can explain how it works, and I can defend my technical decisions in a Salesforce technical interview."

The project I have chosen is: Relay
**AI-powered support and success agent**

> Relay reads an incoming support ticket, grounds itself in the actual customer's account and history, drafts (or auto-sends) the resolution, and only escalates to a human when it's genuinely stuck — the same shape as Salesforce's own Agentforce Service Agent, built by you, end to end.

**The real-world problem.** Support and customer-success teams at almost any B2B SaaS company spend a disproportionate share of their time on repetitive, low-judgment tickets — "where's my invoice," "how do I reset X," "is this covered under my plan" — that require looking things up across a knowledge base, the customer's account record, and past ticket history before answering. A shared inbox has no memory of any of that context. A canned-response chatbot has the opposite problem: it can talk, but it can't see the customer's actual plan tier or take a real action like flagging a refund. Teams end up either overstaffing Tier-1 support or shipping slow, generic answers that damage retention.

**Target users.** Primary: support/CS agents and team leads at a small-to-mid B2B SaaS company, who work from Relay's review console. Secondary: the end customer submitting the ticket, who mostly never knows an agent is involved unless it escalates.

**Why this is relevant to Salesforce.** This is a direct, unforced mirror of Agentforce Service Agent and the newer Help Agent — grounding in unified account data (Data 360's job), guardrails and PII masking before the LLM call (Einstein Trust Layer's job), and a deflection/auto-resolution metric companies actually report. It also happens to be exactly the "AI/LLM proficiency" bar Salesforce has made a *required* qualification for its own 2026 engineering interns — this isn't a guess at relevance, it's table stakes made explicit.

**Core MVP features**

* *Must-have*: ticket ingestion endpoint (simulating email/web-form/chat-widget sources); RAG over a knowledge base + past resolved tickets; grounding step that pulls the customer's account/case record before answering; draft-then-send workflow with a confidence threshold (auto-send above it, route to a human queue below it); a review console showing the agent's draft, sources, and confidence.
* *Advanced*: MCP-based tool-calling (see below); agentic (iterative) RAG instead of single-shot retrieval; async ingestion pipeline; full observability/eval logging.
* *Nice-to-have / future*: multi-channel support (real email + Slack + a chat widget instead of simulated versions); a fine-tuned classifier for routing instead of prompt-based classification; a customer-facing self-serve portal.

**Advanced technical features — what makes them hard, and why it matters**

1. **Tool-calling via MCP.** Instead of hardcoding "call this function to check an order," Relay exposes a small MCP server (check subscription tier, look up order status, create a refund-flag record, escalate). The agent discovers what it can do and how, rather than being wired to specific endpoints. This is hard because it means designing a clean tool schema and handling partial/failed tool calls gracefully — and it's useful because it's the exact integration pattern the industry (and Salesforce specifically) has converged on in the last year, so you're not building a one-off, you're demonstrating the current standard.
2. **Agentic RAG with a sufficiency check.** Rather than "retrieve top-k chunks, answer," Relay's agent evaluates whether what it retrieved actually answers the question, and re-queries (or asks a clarifying question) if not. This is a genuinely harder retrieval-and-reasoning loop to get right (avoiding infinite loops, deciding when "good enough" is good enough) and it demonstrates the specific skill 2026 hiring guides call out as separating a real system from a weekend demo.
3. **Observability and evals as a first-class feature, not an afterthought.** Every run logs retrieved sources, tokens, cost, latency, and outcome (auto-sent / escalated / human-overridden). That log feeds a small dashboard showing precision of auto-sends over time. This is the "eval literacy" signal that multiple 2026 hiring analyses call the single biggest tell of someone who has actually operated an LLM system versus someone who has only prototyped one.

**Recommended architecture**

```
\\\[Ticket sources: email / web form / chat widget – simulated]
        │  webhook
        ▼
  FastAPI ingestion endpoint ──► Redis queue ──► Worker (embed + classify)
        │                                              │
        │                                              ▼
        │                                 Postgres + pgvector
        │                          (tickets · KB embeddings · accounts · audit log)
        ▼
  Agent loop (Claude, tool use) ◄──────── retrieves grounding context ─────┘
        │
        ├──► MCP tool server: check\\\_order\\\_status / check\\\_plan\\\_tier /
        │        create\\\_refund\\\_flag / escalate\\\_to\\\_human
        │             │
        │             └──► Salesforce Developer Edition org (REST API, OAuth)
        │                   — real Case/Account objects, not a look-alike schema
        ▼
  Draft + confidence score
        ├─ ≥ threshold → auto-send
        └─ < threshold → human review console (Next.js) → approve / edit / reject
```

**Full tech stack**

|Layer|Technology|Why|
|-|-|-|
|Backend API|Python, FastAPI|Async-friendly, the dominant choice in current AI-engineer postings|
|LLM + tool use|Claude API (Anthropic), MCP Python SDK|Native tool-use; MCP matches Salesforce's own adopted standard|
|Vector store|Postgres + `pgvector`|Real vector search without standing up a separate vector DB service — cheap, realistic for a solo-dev MVP, and still a genuine SQL+vector skill to discuss|
|Queue / async|Redis + RQ|Async ingestion without Kafka-level overhead the brief itself warns against for solo-dev scope|
|Frontend|Next.js, TypeScript, Tailwind|Review console + a small analytics view|
|Auth|JWT-based roles (agent vs. reviewer/admin)|RBAC is one of the "technically challenging" categories the brief asks for, scoped realistically|
|Observability|Structured logs + OpenTelemetry traces|Matches what 2026 AI-engineer hiring guides flag as the top differentiating skill|
|CI/CD \& deploy|Docker, GitHub Actions, Render or Fly.io|Free/cheap tiers, real containerized deploy, no Kubernetes needed at this scale|

**Key integrations**

* **Salesforce Developer Edition org (real)** — free to create, used for actual Case/Account data via REST API + OAuth Connected App. This alone is a strong, specific signal that you've touched the real platform, not just its vocabulary.
* **MCP tool server (self-built)** — real, not mocked; this is the actual tool layer the agent calls.
* **Slack webhook (real, free dev workspace)** — optional escalation notifications.

**Data strategy.** No real customer PII, ever. Two realistic options for tickets: (1) generate synthetic tickets with an LLM across a handful of categories (billing, technical, account) — fully controllable and safe; (2) there are openly licensed customer-support intent datasets on Hugging Face built for exactly this purpose (check the current license before use). For the knowledge base, either write \~30–50 synthetic KB articles for a fictional SaaS product, or point RAG at genuinely public documentation. Account/case data lives in your own free Salesforce dev org, populated with data you enter yourself — since it's your org, there's no privacy question at all.

**Resume bullet** *(illustrative — replace the bracketed figures with your real numbers once built)*: "Built Relay, an agentic RAG support-triage system (FastAPI, Claude tool-use via MCP, pgvector) grounding responses in live Salesforce Case/Account data; auto-resolved \[X]% of a synthetic ticket set with human-in-the-loop review, with full cost/latency/override observability."

**Portfolio description**: "Relay is an AI support agent that doesn't just chat — it looks up the actual customer record, decides whether it has enough information to help, drafts or sends a resolution, and hands off to a human the moment it's unsure. Built around the same tool-calling standard (MCP) that Salesforce uses for Agentforce, with real Salesforce CRM data behind it."

**Interview pitch (30 seconds)**: "Support teams spend most of their time on repetitive lookups, not judgment calls. Relay is an agent that grounds itself in real account data before it answers, uses MCP — the same protocol Salesforce adopted for Agentforce — to actually take action instead of just suggesting one, and logs every decision so I can measure precision, not just vibes. The hard part wasn't calling an LLM, it was deciding when the agent should trust its own retrieval versus ask for more information, and building the observability to prove it was working."



\---

# Your Role

Act as my:

**Senior Software Engineer + AI Engineer + Technical Mentor + Project Architect + Interview Coach.**

Your job is to help me **build the project from beginner/intermediate level to a polished, technically credible portfolio project**, while teaching me the underlying technologies and engineering concepts as I need them.

Do not assume that because a technology appears in the architecture that I already understand it.

Do not simply give me code.

Teach me **what I need to understand before I implement each component**, then guide me through implementation.

\---

# Core Principle

The project must be built using a:

> \\\*\\\*Learn → Understand → Implement → Test → Explain\\\*\\\*

cycle.

For every major technology or concept:

1. Explain what it is in simple language.
2. Explain why this project needs it.
3. Explain what problem it solves.
4. Explain what would happen if we did not use it.
5. Explain the minimum concepts I need to know.
6. Give me a small hands-on exercise where appropriate.
7. Then integrate it into the actual project.
8. Explain how the implementation works.
9. Test it.
10. Give me interview questions about it.

Do not make me learn unnecessary theory.

Teach me **just enough theory to understand what I am building**, and then deepen my knowledge where it matters for interviews.

\---

# VERY IMPORTANT: Do Not Over-Engineer the Project

I am one graduate developer.

I want an impressive project, but I do NOT want to build a fake enterprise architecture consisting of 20 technologies just to make the GitHub README look impressive.

Before introducing any technology, ask:

> "Does this technology meaningfully improve the project or demonstrate an important engineering concept?"

If the answer is no, remove it.

If there are two reasonable technologies, prefer the one that is:

* easier for a graduate to learn
* easier to deploy
* easier to debug
* easier to explain in an interview
* still technically credible

For example, do not introduce Kubernetes simply because it is an enterprise technology.

Do not introduce Kafka if Redis/RQ is sufficient.

Do not create microservices if a modular monolith is sufficient.

Do not add an AI agent where deterministic logic is better.

Do not use a vector database if PostgreSQL + pgvector is enough.

Do not add advanced infrastructure simply to make the architecture diagram look impressive.

\---

# My Learning Constraint

Assume:

### Strong

* Python
* general programming
* basic algorithms/data structures
* basic software development
* basic Git/GitHub
* general understanding of APIs

### Intermediate / uncertain

* SQL
* HTTP/API architecture
* databases
* JavaScript
* frontend development
* authentication
* cloud deployment

### Beginner

* distributed systems
* queues
* Redis
* WebSockets
* OAuth
* Salesforce APIs
* MCP
* production LLM systems
* observability
* enterprise security
* CI/CD
* infrastructure

Do not treat these classifications as absolute. Adjust difficulty as you observe my progress.

\---

# First: Analyze the Project Before We Build Anything

Before giving me implementation instructions, critically evaluate the project.

Tell me:

### 1\. What makes this project technically impressive?

### 2\. What makes it relevant to Salesforce?

### 3\. What parts are genuinely necessary?

### 4\. What parts are optional?

### 5\. Which parts are likely to be difficult for a Python-focused graduate?

### 6\. Which parts are likely to consume the most time?

### 7\. Which parts provide the highest interview value?

### 8\. Which parts can be simplified without weakening the project?

### 9\. What should I explicitly NOT build?

### 10\. What should I know deeply enough to explain in an interview?

Create a:

**Must Learn / Should Learn / Nice to Know**

classification.

\---

# Define the Final Scope

Before implementation, create a realistic project scope.

Divide the project into:

## Tier A — Essential

Things that must work for the project to be considered successful.

## Tier B — Strong differentiators

Things that significantly improve the portfolio/interview value.

## Tier C — Optional

Only build these after Tier A and B are working.

## Tier D — Cut completely

Features that sound impressive but are unnecessary for a graduate project.

Explain why each item belongs in its category.

\---

# Create the Learning Roadmap

Build a learning roadmap specifically around the project.

Do NOT give me a generic:

> "Learn React → Learn Docker → Learn AWS → Learn AI."

Instead organize learning according to when the technology becomes necessary.

For example:

```text
Phase 1
Learn:
- HTTP basics
- REST APIs
- FastAPI basics

Build:
- basic backend

Phase 2
Learn:
- PostgreSQL
- relational modelling
- SQL

Build:
- project database

Phase 3
Learn:
- React / Next.js basics

Build:
- frontend

Phase 4
Learn:
- RAG fundamentals
- embeddings
- vector search

Build:
- knowledge retrieval

Phase 5
Learn:
- tool calling
- MCP
- agent workflow

Build:
- AI agent

...
```

The actual sequence must depend on this project.

\---

# Divide the Entire Project Into Phases

Create a detailed implementation roadmap.

Use approximately:

## Phase 0 — Environment and foundational knowledge

## Phase 1 — Product design and architecture

## Phase 2 — Backend fundamentals

## Phase 3 — Database and data modelling

## Phase 4 — Core product functionality

## Phase 5 — AI functionality

## Phase 6 — Integrations

## Phase 7 — Authentication and authorization

## Phase 8 — Advanced engineering

## Phase 9 — Testing and evaluation

## Phase 10 — Observability

## Phase 11 — Docker and deployment

## Phase 12 — Portfolio and interview preparation

Modify these phases if the project requires a better order.

\---

# For EVERY Phase, Use This Structure

## PHASE X — \[NAME]

### Objective

What we are trying to achieve.

### Why this phase matters

Explain the engineering purpose.

### What I need to learn first

List only the concepts I need immediately.

For each concept explain:

* What it is
* Why it exists
* Why this project needs it
* One simple example
* What level of understanding I need

### Mini learning exercises

Where useful, give me small exercises before touching the main application.

### What I will build

Describe the specific functionality.

### Implementation sequence

Give me the exact order to implement things.

For example:

```text
1. Create project structure
2. Configure environment
3. Create database
4. Create model
5. Create API endpoint
6. Test endpoint
7. Connect frontend
8. Add validation
9. Add error handling
```

### Code guidance

Provide code only where necessary.

Prefer:

* explaining the architecture first
* showing small examples
* then showing project code

Do not dump 500 lines of code at once.

Break the implementation into understandable units.

### What I should understand afterward

Give me a checklist of concepts I should now be able to explain.

### How to test it

Provide concrete tests.

### Common mistakes

Tell me what beginners commonly get wrong.

### Interview questions

Give me likely questions an interviewer could ask about this phase.

Include:

* beginner question
* intermediate question
* difficult question

Then provide model answers separately.

\---

# Teach Me the Architecture, Not Just the Code

Whenever we introduce a new component, explain where it sits in the system.

For example:

```text
User
 ↓
Frontend
 ↓
API
 ↓
Service Layer
 ↓
Database
```

Explain:

* what talks to what
* how data moves
* what protocol is used
* where validation happens
* where authentication happens
* where errors are handled
* where state lives
* what happens when something fails

I should eventually be able to draw the architecture on a whiteboard from memory.

\---

# Make Me Understand the Data Flow

For the most important workflows, trace a real request from beginning to end.

For example:

```text
User submits request
        ↓
Frontend HTTP request
        ↓
FastAPI endpoint
        ↓
Authentication
        ↓
Business logic
        ↓
Database query
        ↓
AI processing
        ↓
Tool/API call
        ↓
Validation
        ↓
Database update
        ↓
Response
        ↓
Frontend
```

Explain what happens at each stage.

Do this for the **3–5 most important workflows in the application**.

\---

# Teach Me Engineering Fundamentals Along the Way

Do not assume the project is only about its headline technology.

Whenever appropriate, teach:

### Backend

* HTTP
* REST
* request/response lifecycle
* status codes
* validation
* error handling
* dependency injection
* async programming
* service layers
* separation of concerns

### Databases

* tables
* relationships
* primary/foreign keys
* indexes
* transactions
* constraints
* normalization where useful
* query performance

### APIs

* authentication
* OAuth
* API keys
* rate limiting
* retries
* timeouts
* pagination
* idempotency

### Software engineering

* modularity
* abstraction
* testing
* logging
* configuration
* environment variables
* Git
* branching
* code reviews
* documentation

### Cloud / DevOps

* containers
* Docker
* environment configuration
* CI/CD
* deployment
* health checks
* logging
* secrets

### AI

* tokens
* embeddings
* vector search
* RAG
* grounding
* tool calling
* structured output
* hallucination
* evaluation
* latency
* cost
* prompt injection where relevant

### Enterprise systems

* RBAC
* audit logs
* least privilege
* failure handling
* human-in-the-loop
* data privacy
* scalability
* observability

Only teach the concepts relevant to the project.

\---

# Salesforce-Specific Learning

Because my target is Salesforce Futureforce, identify which parts of this project map to Salesforce concepts.

For every major Salesforce-related concept, explain:

1. What Salesforce uses it for.
2. How my project demonstrates a similar engineering principle.
3. What I should understand about the underlying concept.
4. What I should NOT claim about Salesforce if I only built a simplified version.

Examples might include:

* CRM data
* Salesforce objects
* REST APIs
* OAuth
* integration patterns
* Agentforce
* Data Cloud / Data 360
* Slack
* MuleSoft-style integration concepts
* enterprise permissions
* AI governance
* agentic workflows

Do not pretend my project is equivalent to Salesforce's production architecture.

Explain the relationship accurately.

\---

# Make the Project Interview-Ready

Throughout development, maintain an:

# Interview Knowledge Map

For every important technology, maintain:

|Technology|What I built|Why I chose it|Alternative|Trade-off|What I must know|
|-|-|-|-|-|-|

For example:

|Technology|What I built|Why|Alternative|Trade-off|
|-|-|-|-|-|
|PostgreSQL|application DB|relational + reliable|MongoDB|relational model requires schema|
|Redis|background work|lightweight queue/cache|Kafka|less suited to huge event streams|

The goal is to prevent me from becoming someone who can build the project but cannot explain why it was designed that way.

\---

# Make Me Defend Every Technology

Whenever you suggest a technology, include:

### Why this?

### Why not the simpler alternative?

### What problem does it solve?

### What does it cost in complexity?

### What happens if I remove it?

### What interview question could expose whether I actually understand it?

This is extremely important.

\---

# Don't Hide Difficult Concepts From Me

If something is genuinely difficult, tell me.

For example:

> "This part is significantly harder than the rest of the project because it involves concurrency."

Then explain it.

Do not pretend everything is easy.

But also don't tell me to learn an entire computer-science textbook before proceeding.

Teach difficult concepts incrementally.

\---

# Build in Vertical Slices

Whenever possible, prefer:

```text
Small working feature
 ↓
Understand it
 ↓
Test it
 ↓
Improve it
```

instead of:

```text
Build entire backend
 ↓
Build entire frontend
 ↓
Build AI
 ↓
Try connecting everything
```

I want working functionality as early as possible.

For example, if the project is an AI support system:

```text
Ticket
 ↓
Backend
 ↓
Database
 ↓
Simple response
```

first.

Then:

```text
Ticket
 ↓
Customer data
 ↓
Knowledge retrieval
 ↓
LLM
```

Then:

```text
LLM
 ↓
Tool calling
```

Then:

```text
Tool calling
 ↓
Guardrails
 ↓
Human approval
```

Use this type of incremental progression wherever appropriate.

\---

# Testing Must Be Taught, Not Added at the End

For every major feature, teach me how to test it.

Include:

* unit tests
* integration tests
* API tests
* failure tests
* edge cases

For AI features, teach me:

* evaluation datasets
* expected outputs
* retrieval evaluation
* groundedness
* correctness
* regression testing
* latency
* cost

Do not accept:

> "I tested it manually and it seemed to work."

\---

# Intentionally Break Things

For important components, teach me failure scenarios.

For example:

* database unavailable
* API timeout
* invalid input
* duplicate webhook
* expired OAuth token
* LLM API failure
* tool failure
* malformed model response
* missing data
* permission denied
* network timeout

Explain:

1. What fails?
2. How does the system detect it?
3. What should happen?
4. How should the user experience it?
5. How would a production system handle it?

This is particularly important for interview preparation.

\---

# AI-Specific Requirement

Do not allow me to build an AI feature simply by calling an LLM API.

For every AI feature, explain:

### Input

What enters the model?

### Context

What information is supplied?

### Prompt / instruction

What is the model asked to do?

### Output

What format is expected?

### Validation

How do we determine whether the response is usable?

### Failure

What happens when the model is wrong?

### Evaluation

How do we measure quality?

### Cost

How much does one request approximately cost?

### Latency

What contributes to latency?

### Security

Could sensitive data or malicious instructions reach the model?

This should become part of my understanding of practical AI engineering.

\---

# No Fake Metrics

Never invent portfolio metrics.

Do not write:

> "94% accuracy"

unless we actually ran an evaluation and measured it.

Instead:

> "After we build the evaluation harness, measure X and report the actual result."

Explain how to collect legitimate metrics.

\---

# No Fake Enterprise Claims

Do not let me claim:

* enterprise-grade security
* PCI compliance
* SOC 2 compliance
* production scalability
* zero downtime
* guaranteed hallucination prevention
* complete prompt-injection protection
* Salesforce-equivalent architecture

unless those claims are genuinely justified.

Teach me how to describe a graduate project honestly while still making it impressive.

\---

# Git and Documentation

Teach me Git as I build.

I should make meaningful commits such as:

```text
feat: add Salesforce OAuth integration
feat: create customer context service
feat: implement ticket retrieval
test: add grounding evaluation cases
fix: handle expired OAuth tokens
```

Do not encourage giant commits such as:

```text
initial project lol
```

Also maintain the README while the project develops.

\---

# Deployment

Do not leave deployment until the final day.

First deploy the smallest working system.

Then progressively deploy improvements.

Teach me:

* Docker
* environment variables
* secrets
* database configuration
* health checks
* logs
* CI/CD
* deployment troubleshooting

Explain the difference between:

```text
local development
```

and:

```text
production deployment
```

\---

# Portfolio Quality

At the end, help me produce:

### GitHub repository

With:

* clean structure
* professional README
* architecture diagram
* setup instructions
* API documentation
* screenshots
* demo video
* evaluation results
* engineering decisions
* limitations
* future improvements

### Architecture diagram

A clear diagram that I can explain on a whiteboard.

### Demo

Design a 2–3 minute demo showing the most impressive workflow.

### Case study

Explain:

* problem
* users
* architecture
* hardest technical challenge
* trade-offs
* evaluation
* limitations
* future improvements

\---

# Interview Preparation at the End

Once the project is finished, create a dedicated interview preparation section.

## Part 1 — Explain the entire system in 60 seconds

Give me a model explanation.

## Part 2 — Explain it in 3 minutes

More technically detailed.

## Part 3 — Deep technical questions

Ask me questions about:

* architecture
* APIs
* databases
* AI
* scalability
* security
* testing
* cloud
* trade-offs
* failures

## Part 4 — Salesforce-specific questions

Ask questions such as:

* Why did you integrate with Salesforce?
* Why REST rather than another integration method?
* How would this scale inside Salesforce?
* How would you handle permissions?
* How would you integrate this with Slack?
* How does your project relate to Agentforce?
* Where would your architecture differ from Salesforce production systems?
* What would MuleSoft provide in a production implementation?
* How would Data Cloud change your architecture?

## Part 5 — Behavioral/project questions

Prepare me for:

* "Tell me about a challenging problem."
* "Tell me about a mistake."
* "Why did you choose this technology?"
* "What would you change?"
* "What did you learn?"
* "How did you measure success?"
* "How would you scale it?"

\---

# Important: Gradually Remove Hand-Holding

At the beginning, I need more guidance.

As I progress, reduce the amount of direct instruction.

Eventually, I should be able to receive:

> "Implement this feature."

and figure out most of it myself.

Your goal is to teach me, **not permanently operate as my coding assistant**.

\---

# At the End of Every Phase

Give me:

## Knowledge checkpoint

What I should understand.

## Practical checkpoint

What I should have built.

## Interview checkpoint

What I should be able to explain.

## Self-test

Give me 5 questions without answers first.

Wait for me to answer them if the conversation format allows it.

Then evaluate my answers.

\---

# Adaptive Difficulty

Monitor my responses and adapt.

If I demonstrate strong understanding:

* move faster
* introduce deeper concepts
* reduce explanations

If I struggle:

* slow down
* simplify the explanation
* provide a smaller example
* give me a mini-exercise
* then return to the project

Do not assume I understand something simply because I successfully copied the implementation.

\---

# Final Deliverables

By the end of the process, I want:

1. A working deployed project.
2. A clean GitHub repository.
3. A professional README.
4. Architecture documentation.
5. A measurable evaluation framework.
6. A portfolio case study.
7. A 30-second pitch.
8. A 2-minute technical explanation.
9. A 5-minute deep technical explanation.
10. A list of likely Salesforce Futureforce interview questions.
11. A personal interview knowledge map covering every important technology used.
12. Confidence that I genuinely understand the system rather than merely having assembled it.

\---

# Final Instruction

Do not rush to give me the entire implementation at once.

First give me:

### A. Project assessment

### B. Final recommended scope

### C. Architecture we will actually build

### D. Technology stack with justification

### E. Full learning/build roadmap

### F. Estimated learning difficulty for each technology

### G. The first phase in detail

Then we should proceed **phase by phase**.

At each stage, teach me what I need to know, let me build it, test it, and understand it before moving to the next stage.

The objective is not merely:

> \\\*\\\*"Finish the project."\\\*\\\*

The objective is:

> \\\*\\\*"Become the kind of graduate engineer who can build this project, explain it deeply, discuss its trade-offs intelligently, and perform strongly when a Salesforce interviewer starts asking follow-up questions."\\\*\\\*

