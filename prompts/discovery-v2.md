# Kotlin + AKS — AI-Assisted Development: Complete Guide
> End-to-end reference for bootstrapping a Kotlin Gradle API project from raw requirements to production on AKS, using GitHub Copilot as your AI pair programmer.
>
> Audience: Software Engineer / Architect in Fintech & Banking

---

## Table of Contents

1. [The Master Sequence](#1-the-master-sequence)
2. [Document Hierarchy — What Each File Is For](#2-document-hierarchy--what-each-file-is-for)
3. [Step 1 — PRD.md](#3-step-1--prdmd)
4. [Step 2 — TECHNICAL_SPEC.md](#4-step-2--technical_specmd)
5. [Step 3 — ARCHITECTURE.md](#5-step-3--architecturemd)
6. [Step 4 — Understand External Dependencies](#6-step-4--understand-external-dependencies)
7. [Step 5 — DISCOVERY.md (inside dependency repos)](#7-step-5--discoverymd-inside-dependency-repos)
8. [Step 6 — Integration Docs (inside your repo)](#8-step-6--integration-docs-inside-your-repo)
9. [Step 7 — AGENTS.md](#9-step-7--agentsmd)
10. [Step 8 — CODEBASE.md](#10-step-8--codebasemd)
11. [Step 9 — copilot-instructions.md](#11-step-9--copilot-instructionsmd)
12. [Step 10 — TASKS.md](#12-step-10--tasksmd)
13. [Step 11 — Project Scaffolding](#13-step-11--project-scaffolding)
14. [Step 12 — Iterative Development Loop](#14-step-12--iterative-development-loop)
15. [Step 13 — AKS Deployment Artifacts](#15-step-13--aks-deployment-artifacts)
16. [Step 14 — Quality & Hardening](#16-step-14--quality--hardening)
17. [AGENTS.md vs copilot-instructions.md — Key Differences](#17-agentsmd-vs-copilot-instructionsmd--key-differences)
18. [Full Repository Structure](#18-full-repository-structure)
19. [VS Code Setup](#19-vs-code-setup)

---

## 1. The Master Sequence

Everything in order, from raw requirements to production:

```
requirements.txt                  ← starting point — plain text from stakeholders
        ↓
[DOCUMENTATION PHASE]
        ↓
docs/PRD.md                       ← WHY: business goals, user stories, acceptance criteria
        ↓
docs/TECHNICAL_SPEC.md            ← WHAT: endpoints, entities, NFRs
        ↓
docs/ARCHITECTURE.md              ← HOW: tech stack, AKS topology, infra decisions
        ↓
[DEPENDENCY DISCOVERY PHASE]
        ↓
project-A/DISCOVERY.md            ← reverse-engineer dependency repos
project-B/DISCOVERY.md
        ↓
docs/integrations/
  project-a-integration.md        ← how OUR project consumes project-A
  project-b-integration.md        ← how OUR project consumes project-B
  DEPENDENCIES.md                 ← single-glance summary of all external deps
        ↓
[AI AGENT SETUP PHASE]
        ↓
AGENTS.md                         ← rules for ALL AI agents (Copilot, Cursor, Claude Code)
CODEBASE.md                       ← living index of classes, APIs, modules, tests
.github/copilot-instructions.md   ← Copilot-specific coding style and context
        ↓
[IMPLEMENTATION PHASE]
        ↓
TASKS.md                          ← full implementation task list as checkboxes
        ↓
Project scaffolding               ← Gradle, Spring Boot, Dockerfile
        ↓
Iterative feature development     ← entity → service → controller → tests (per task)
        ↓
AKS deployment artifacts          ← k8s manifests, GitHub Actions CI/CD
        ↓
Quality & hardening               ← security, observability, load testing
        ↓
PRODUCTION
```

---

## 2. Document Hierarchy — What Each File Is For

| Document | Lives In | Audience | Answers |
|---|---|---|---|
| `PRD.md` | `docs/` | Stakeholders, PO, QA | Why are we building this? |
| `TECHNICAL_SPEC.md` | `docs/` | Developers, QA | What exactly are we building? |
| `ARCHITECTURE.md` | `docs/` | Developers, DevOps | How is it structured and deployed? |
| `DISCOVERY.md` | Dependency repo root | Anyone new to that repo | What does this service do? |
| `project-a-integration.md` | `docs/integrations/` | Our developers | How do we consume project-A? |
| `DEPENDENCIES.md` | `docs/integrations/` | Our developers | What are all our external deps? |
| `AGENTS.md` | Our repo root | All AI agents | What are agents allowed/forbidden to do? |
| `CODEBASE.md` | Our repo root | Developers, AI agents | What classes, APIs, modules exist now? |
| `copilot-instructions.md` | `.github/` | GitHub Copilot only | How should Copilot write code here? |
| `TASKS.md` | Our repo root | Developer + AI agent | What needs to be built, in what order? |

---

## 3. Step 1 — PRD.md

### What it is
The business case. Written before any technical decisions. Critical in fintech for stakeholder sign-off and audit trail.

### Copilot prompt
> *"Based on the open requirements.txt, generate a lightweight PRD covering: business goals, user stories with acceptance criteria, assumptions, constraints, and out-of-scope items. Save to `docs/PRD.md`."*

### Contains
- Business goals and success metrics
- User stories: `As a [user], I want to [action] so that [outcome]`
- Acceptance criteria per story
- Out-of-scope — explicit list of what is NOT being built
- Assumptions and constraints

### Why it matters in fintech/banking
- Stakeholders and clients need sign-off before development starts
- QA teams derive test cases from acceptance criteria
- Provides an audit trail for regulatory or compliance review
- Prevents scope creep mid-development

---

## 4. Step 2 — TECHNICAL_SPEC.md

### What it is
The technical translation of the PRD. Answers: what does the system do in technical terms?

### Copilot prompt
> *"Based on `docs/PRD.md`, generate a technical specification covering: functional requirements, REST API endpoints (method, path, request/response schemas), domain entities with fields, and non-functional requirements (auth, latency targets, error handling strategy). Save to `docs/TECHNICAL_SPEC.md`."*

### Contains
- Functional requirements list
- REST API endpoint inventory

| Method | Path | Request Body | Response | Auth |
|---|---|---|---|---|
| POST | /api/v1/payments | PaymentRequest | PaymentResponse | Bearer |
| GET | /api/v1/payments/{id} | — | PaymentResponse | Bearer |

- Domain entities with field definitions
- Non-functional requirements: auth mechanism, SLAs, error format, pagination

---

## 5. Step 3 — ARCHITECTURE.md

### What it is
The technical blueprint. Answers: how will the system be built and deployed?

### Copilot prompt
> *"Based on `docs/TECHNICAL_SPEC.md`, generate an architecture document covering: tech stack decisions with rationale, service/module structure, AKS deployment topology (ingress, pods, services, ACR, secrets), infrastructure decisions (ConfigMaps, HPA, namespaces), and external service integrations. Save to `docs/ARCHITECTURE.md`."*

### Contains
- Tech stack: Kotlin, Spring Boot 3, Gradle Kotlin DSL, Flyway, JPA
- Module/package structure
- AKS topology diagram (described in markdown)
- Infrastructure: ConfigMaps, Secrets, HPA thresholds
- External integrations: project-A, project-B endpoints and auth patterns

---

## 6. Step 4 — Understand External Dependencies

Before writing any integration code, reverse-engineer project-A and project-B.

### Set Up a Multi-Root Workspace

Create a `.code-workspace` file so Copilot can see all repos simultaneously:

```json
{
  "folders": [
    { "name": "our-project", "path": "./our-new-project" },
    { "name": "project-A",   "path": "./project-A" },
    { "name": "project-B",   "path": "./project-B" }
  ]
}
```

### Priority Files to Open in Each Dependency Repo

| File Type | What It Reveals |
|---|---|
| `*Controller.kt` / `*Resource.kt` | All exposed endpoints |
| `*DTO.kt` / `*Request.kt` / `*Response.kt` | Data contracts |
| `swagger.yml` / `openapi.yaml` | Full API spec if available |
| `application.yml` | Ports, service names, feature flags |
| `*Client.kt` / `*FeignClient.kt` | How they call other services |
| `README.md` | Often overlooked — start here |
| Test classes | Best real documentation of expected behaviour |

### If No Documentation Exists — Escalation Order
1. Read the test classes — integration tests document real behaviour
2. Check `git log` — commit messages reveal intent
3. Run the service locally, call it with a REST client
4. Ask the team — but come with specific questions, not blank ones

---

## 7. Step 5 — DISCOVERY.md (inside dependency repos)

### What it is
A reverse-engineering document that lives **inside project-A and project-B repos**. Written from the perspective of someone exploring that repo for the first time.

### Where it lives
```
project-A/
└── DISCOVERY.md    ← at the root of the dependency repo

project-B/
└── DISCOVERY.md
```

### Why not README.md?
`README.md` is owned by that team — don't pollute it. `DISCOVERY.md` is clearly an external discovery artefact and non-intrusive.

### Copilot prompt (run with project-A files open)
> *"Based on the open files from project-A, generate a `DISCOVERY.md` at the project root covering: project purpose, all exposed REST endpoints, request/response models, authentication mechanism, key domain entities, known quirks or gotchas, and open questions. Save to `DISCOVERY.md`."*

### DISCOVERY.md Template

```markdown
# project-A — Discovery Document

> Generated by: [your name]
> Date: [date]
> Context: Reverse-engineered for integration with our-new-project

## Purpose
What this service does in one paragraph.

## Tech Stack
- Language / Framework:
- Database:
- Auth mechanism:

## Exposed REST Endpoints
| Method | Path | Request | Response | Auth Required |
|--------|------|---------|----------|---------------|
| POST   | /api/customers | CustomerRequest | CustomerResponse | Yes — Bearer |

## Key Domain Entities
- **Customer** — fields and constraints
- **Account** — fields and constraints

## Authentication
How to obtain and pass tokens/credentials.

## Error Response Format
```json
{ "code": "ERR_001", "message": "Descriptive error" }
```

## Known Quirks / Gotchas
- Non-obvious behaviour found during analysis

## Open Questions
- [ ] Unclear behaviour around X — ask the project-A team
- [ ] Confirm pagination behaviour on /api/customers
```

> **Important:** Commit `DISCOVERY.md` to a branch like `docs/discovery` — get the owning team's consent before merging to main. If you have no write access, keep it locally or in your own repo.

---

## 8. Step 6 — Integration Docs (inside your repo)

### What they are
Consumer-centric documents that live in **your repo**. While `DISCOVERY.md` answers "what does project-A do?", these answer "how do **we** use project-A?".

### File locations
```
our-new-project/
└── docs/
    └── integrations/
        ├── DEPENDENCIES.md              ← summary of all external deps
        ├── project-a-integration.md     ← our consumption of project-A
        └── project-b-integration.md     ← our consumption of project-B
```

### Copilot prompt
> *"Based on the open files from project-A and `project-A/DISCOVERY.md`, generate a consumer-focused integration reference and save to `docs/integrations/project-a-integration.md`. Cover: base URL config, endpoints we will call, request/response schemas, auth setup, error handling strategy, and any known limitations."*

### Generate DEPENDENCIES.md
> *"Based on `docs/integrations/project-a-integration.md` and `docs/integrations/project-b-integration.md`, generate a single-page summary of all external service dependencies. Save to `docs/integrations/DEPENDENCIES.md`."*

### The full information flow
```
project-A source code
        ↓ analysed into ↓
project-A/DISCOVERY.md          ← repo-centric (what is it?)
        ↓ consumed into ↓
docs/integrations/project-a-integration.md   ← consumer-centric (how do we use it?)
        ↓ summarised into ↓
docs/integrations/DEPENDENCIES.md            ← one-glance overview of all deps
```

---

## 9. Step 7 — AGENTS.md

### What it is
An instruction file for **all AI coding agents** — not just Copilot. Defines what agents are allowed to do autonomously, what they must never touch, and how they should behave in this specific repo.

Think of it as an **employee contract for AI agents**.

### Where it lives
```
our-new-project/
└── AGENTS.md    ← always at repo root
```

### Why it is essential for this project

| Risk | Without AGENTS.md | With AGENTS.md |
|---|---|---|
| Agent modifies k8s manifests | Edits prod config silently | Knows to stay out |
| Agent adds a dependency | Adds without asking | Flags for your review |
| Agent writes a DTO | Random structure | Follows Kotlin data class convention |
| Agent writes a test | Random test framework | Always MockK + JUnit 5 |
| Agent calls project-A | Guesses the API shape | Uses your integration docs |

### AGENTS.md Template

```markdown
# AGENTS.md

## Project Overview
Kotlin Spring Boot 3 REST API deployed to AKS.
Gradle Kotlin DSL. Fintech domain.
See docs/ARCHITECTURE.md for full context.

## Build Commands
- Build:      `./gradlew build`
- Test:       `./gradlew test`
- Run (dev):  `./gradlew bootRun --args='--spring.profiles.active=dev'`
- Lint:       `./gradlew ktlintCheck`

## Code Conventions
- Use Kotlin data classes for all DTOs
- Prefer coroutines over CompletableFuture for async
- No nullable types without explicit justification
- All REST endpoints must have OpenAPI/Swagger annotations
- Follow conventional commits: feat:, fix:, chore:, docs:

## Testing Rules
- Unit tests: JUnit 5 + MockK (never Mockito)
- Integration tests: Testcontainers
- Every new endpoint requires both unit and integration test
- Never delete existing tests

## Package Conventions
- Controllers  → `api/controller/`
- DTOs         → `api/dto/`
- Mappers      → `api/mapper/`
- Services     → `domain/service/`
- Entities     → `domain/model/`
- Repositories → `domain/repository/`
- Feign clients → `infrastructure/client/`
- Config beans  → `infrastructure/config/`

## What Agents MUST NOT Do
- Do NOT modify any file in `k8s/` without explicit instruction
- Do NOT modify `application-prod.yml`
- Do NOT add new Gradle dependencies without flagging for review
- Do NOT hardcode URLs, secrets, or credentials anywhere
- Do NOT push directly to main branch

## External Dependencies
- project-A: see docs/integrations/project-a-integration.md
- project-B: see docs/integrations/project-b-integration.md
- Always use documented contracts — do not guess API shapes

## Task Execution
- Work from TASKS.md — complete one task at a time
- Mark tasks complete [x] after finishing
- Update CODEBASE.md after every 3–5 tasks
```

---

## 10. Step 8 — CODEBASE.md

### What it is
A **living technical index** of everything that exists in the repo. The glossary/dictionary of the codebase — classes, APIs, modules, test classes, config files.

Not an architecture doc (no decisions). Not a spec (no requirements). Just: *what is here right now?*

### Where it lives
```
our-new-project/
└── CODEBASE.md    ← repo root, alongside AGENTS.md
```

### CODEBASE.md Template

```markdown
# CODEBASE.md
> Last updated: [date]

## Module Structure
| Module | Responsibility |
|--------|---------------|
| payment-api | REST layer, DTOs, OpenAPI spec |
| payment-domain | Business logic, entities, repositories |
| payment-infra | External clients, DB config, Spring beans |

## Package Structure
com.yourcompany.project
├── api/
│   ├── controller/     — REST controllers
│   ├── dto/            — Request/Response models
│   └── mapper/         — DTO ↔ Domain mappers
├── domain/
│   ├── model/          — JPA entities / domain objects
│   ├── service/        — Business logic
│   └── repository/     — Spring Data interfaces
├── infrastructure/
│   ├── config/         — Spring configuration beans
│   ├── client/         — Feign clients for external services
│   └── persistence/    — JPA repository implementations
└── common/
    └── exception/      — Global exception handler, error DTOs

## Classes Glossary
| Class | Package | Responsibility |
|-------|---------|---------------|
| PaymentController | api/controller | POST /payments, GET /payments/{id} |
| PaymentService | domain/service | Core payment business logic |
| PaymentRepository | domain/repository | DB access for Payment entity |
| PaymentRequest | api/dto | Inbound DTO — create payment |
| PaymentResponse | api/dto | Outbound DTO — payment details |
| ProjectAClient | infrastructure/client | Feign client for project-A identity service |
| ProjectBClient | infrastructure/client | Feign client for project-B accounts service |
| GlobalExceptionHandler | common/exception | Maps exceptions to HTTP error responses |

## REST API Inventory
| Method | Path | Controller | Auth | Description |
|--------|------|-----------|------|-------------|
| POST | /api/v1/payments | PaymentController | Bearer | Initiate payment |
| GET | /api/v1/payments/{id} | PaymentController | Bearer | Get payment by ID |

## Test Classes
| Test Class | Type | Framework | Covers |
|-----------|------|-----------|--------|
| PaymentControllerTest | Unit | MockMvc + MockK | Controller layer |
| PaymentServiceTest | Unit | JUnit 5 + MockK | Business logic |
| PaymentIntegrationTest | Integration | Testcontainers | Full flow with DB |
| ProjectAClientTest | Integration | WireMock | External client |

## Configuration Files
| File | Profile | Purpose |
|------|---------|---------|
| application.yml | all | Shared base config |
| application-dev.yml | dev | Local dev overrides |
| application-prod.yml | prod | AKS production values |

## Database Migrations (Flyway)
| File | Description |
|------|-------------|
| V1__create_payment_table.sql | Initial payment schema |

## Key Dependencies
| Library | Purpose |
|---------|---------|
| spring-boot-starter-web | REST API |
| spring-boot-starter-data-jpa | Database access |
| flyway-core | DB migrations |
| springdoc-openapi | Swagger UI + OpenAPI spec |
| spring-cloud-starter-openfeign | External HTTP clients |
| resilience4j | Circuit breaker for external calls |
| mockk | Kotlin-idiomatic mocking |
| testcontainers | Integration tests with real DB |
```

### Keeping CODEBASE.md current

Ask Copilot periodically:
> *"Scan the current codebase and update `CODEBASE.md` — add any new classes, endpoints, or test classes created since the last update."*

Or automate with Dokka (Kotlin KDoc) or a custom Gradle task.

---

## 11. Step 9 — copilot-instructions.md

### What it is
A **GitHub Copilot-specific** context file. Copilot reads it automatically on every session. Focuses on coding style, preferences, and project context — not on agent rules (those go in `AGENTS.md`).

### Where it lives
```
our-new-project/
└── .github/
    └── copilot-instructions.md
```

### Template

```markdown
# copilot-instructions.md

## Project
Kotlin Spring Boot 3 REST API — fintech domain — deployed to AKS.
Gradle Kotlin DSL. See docs/ARCHITECTURE.md for full context.

## Coding Style
- Kotlin data classes for all DTOs
- Coroutines for async (not CompletableFuture)
- Null safety: avoid nullable types without justification
- Functional style preferred over imperative where idiomatic

## Testing
- Unit tests: JUnit 5 + MockK
- Integration tests: Testcontainers
- Every endpoint needs both unit and integration test

## OpenAPI
- All controllers must have @Operation, @ApiResponse annotations
- Springdoc is used — not springfox

## External Dependencies
- project-A (identity service): docs/integrations/project-a-integration.md
- project-B (accounts service): docs/integrations/project-b-integration.md
- Use documented contracts — do not invent API shapes

## Agent Rules
See AGENTS.md at repo root for what agents are and are not allowed to do.

## Codebase Reference
See CODEBASE.md for current classes, endpoints, and test inventory.
```

---

## 12. Step 10 — TASKS.md

### What it is
A **checkbox-driven implementation plan** generated from your spec and architecture docs. The single source of truth for what needs to be built, in what order. Both you and Copilot Agent work from it.

### Copilot prompt to generate it
With `docs/TECHNICAL_SPEC.md`, `docs/ARCHITECTURE.md`, and `AGENTS.md` open:

> *"Based on the open spec and architecture documents, generate a complete `TASKS.md` with all implementation work broken into phases, each task as a checkbox, ordered by dependency. Include task IDs."*

### TASKS.md Template

```markdown
# TASKS.md — Implementation Task List

## Phase 1 — Project Setup
- [ ] TASK-001: Initialise Kotlin Spring Boot 3 project with Gradle Kotlin DSL
- [ ] TASK-002: Configure application.yml with dev and prod profiles
- [ ] TASK-003: Set up Dockerfile with multi-stage build
- [ ] TASK-004: Configure ktlint and code style rules
- [ ] TASK-005: Set up Flyway migration baseline

## Phase 2 — Domain Layer
- [ ] TASK-006: Create Payment entity with JPA mapping
- [ ] TASK-007: Create PaymentRepository interface
- [ ] TASK-008: Write V1__create_payment_table.sql migration
- [ ] TASK-009: Create PaymentService with core business logic
- [ ] TASK-010: Unit test PaymentService with MockK

## Phase 3 — API Layer
- [ ] TASK-011: Create PaymentController — POST /api/v1/payments
- [ ] TASK-012: Create PaymentController — GET /api/v1/payments/{id}
- [ ] TASK-013: Create PaymentRequest and PaymentResponse DTOs
- [ ] TASK-014: Add OpenAPI annotations to all endpoints
- [ ] TASK-015: Create GlobalExceptionHandler

## Phase 4 — External Integrations
- [ ] TASK-016: Create ProjectAClient (Feign) — identity service
- [ ] TASK-017: Create ProjectBClient (Feign) — accounts service
- [ ] TASK-018: Configure Resilience4j circuit breakers
- [ ] TASK-019: WireMock integration tests for external clients

## Phase 5 — Security
- [ ] TASK-020: Configure Spring Security with JWT validation
- [ ] TASK-021: Apply role-based access to all endpoints
- [ ] TASK-022: Security unit tests

## Phase 6 — Observability
- [ ] TASK-023: Add Micrometer + Prometheus metrics
- [ ] TASK-024: Configure structured JSON logging (Logback)
- [ ] TASK-025: Verify Actuator health endpoints

## Phase 7 — AKS Deployment
- [ ] TASK-026: Generate k8s/deployment.yaml
- [ ] TASK-027: Generate k8s/service.yaml and k8s/ingress.yaml
- [ ] TASK-028: Generate k8s/configmap.yaml and k8s/hpa.yaml
- [ ] TASK-029: Create GitHub Actions CI/CD pipeline (build → ACR → AKS)

## Phase 8 — QA & Hardening
- [ ] TASK-030: Full integration tests with Testcontainers
- [ ] TASK-031: Load test key endpoints
- [ ] TASK-032: Final CODEBASE.md update and review
```

### Executing Tasks with Copilot

Pick a task and prompt:
> *"Complete TASK-009 from TASKS.md. Refer to TECHNICAL_SPEC.md for business rules and AGENTS.md for conventions."*

After completion, mark it done:
```markdown
- [x] TASK-009: Create PaymentService with core business logic
```

### Bridging to JIRA
> *"Convert TASKS.md into JIRA story descriptions with acceptance criteria, one story per task."*

---

## 13. Step 11 — Project Scaffolding

With all documentation in place, generate the project skeleton.

### Copilot Agent prompt
> *"Create a Kotlin Spring Boot 3 project with Gradle Kotlin DSL featuring: REST API, Spring Data JPA, Flyway migrations, OpenAPI/Swagger via Springdoc, Actuator health endpoints, Feign clients, Resilience4j, and a multi-stage Dockerfile. Follow the package structure in AGENTS.md."*

### Key files generated

| File | Purpose |
|---|---|
| `build.gradle.kts` | All dependencies, Kotlin DSL |
| `settings.gradle.kts` | Module definitions |
| `application.yml` | Base configuration |
| `application-dev.yml` | Local dev overrides |
| `application-prod.yml` | AKS production config |
| `Dockerfile` | Multi-stage: builder + slim JRE |

---

## 14. Step 12 — Iterative Development Loop

Repeat this cycle for every feature in TASKS.md:

```
Pick next unchecked task from TASKS.md
        ↓
Prompt Copilot with task + reference to spec and AGENTS.md
        ↓
Copilot generates: entity → repository → service → controller → DTOs
        ↓
Ask Copilot for unit tests (MockK) + integration tests (Testcontainers)
        ↓
Review, adjust, commit (conventional commit format)
        ↓
Mark task [x] in TASKS.md
        ↓
Every 3–5 tasks: ask Copilot to update CODEBASE.md
```

---

## 15. Step 13 — AKS Deployment Artifacts

### Copilot prompt
> *"Generate complete Kubernetes manifests for this Kotlin Spring Boot app on AKS: Deployment (2 replicas, resource limits/requests), Service (ClusterIP), Ingress (nginx), ConfigMap for non-secret config, HorizontalPodAutoscaler, and a GitHub Actions pipeline that builds the Docker image, pushes to ACR, and deploys to AKS using kubectl."*

### Files generated

```
k8s/
├── deployment.yaml      ← pod spec, image, resource limits, probes
├── service.yaml         ← ClusterIP service
├── ingress.yaml         ← nginx ingress with TLS
├── configmap.yaml       ← non-secret application config
└── hpa.yaml             ← auto-scaling rules

.github/workflows/
└── deploy.yml           ← build → push ACR → deploy AKS
```

> **AGENTS.md reminder:** Agents must not modify these files without explicit instruction. Protect them.

---

## 16. Step 14 — Quality & Hardening

| Area | What to Do | Copilot Prompt |
|---|---|---|
| **OpenAPI** | Validate all endpoints documented | *"Review controllers and flag any endpoints missing OpenAPI annotations"* |
| **Test coverage** | Identify gaps | *"List endpoints that have no integration test and generate them"* |
| **Security** | JWT + role-based access | *"Generate Spring Security config for JWT validation and role-based endpoint protection"* |
| **Observability** | Metrics + structured logs | *"Add Micrometer counters to PaymentService and configure Logback for JSON output"* |
| **Resilience** | Circuit breakers on external calls | *"Add Resilience4j circuit breaker config for ProjectAClient and ProjectBClient"* |
| **Load testing** | Confirm NFRs are met | Use k6 or Gatling; prompt Copilot to generate test scripts |

---

## 17. AGENTS.md vs copilot-instructions.md — Key Differences

These are **complementary**, not interchangeable.

| | `AGENTS.md` | `copilot-instructions.md` |
|---|---|---|
| **Audience** | All AI agents — Copilot, Cursor, Claude Code, Codex | GitHub Copilot only |
| **Standard** | Open, agent-agnostic, emerging industry standard | GitHub/Microsoft proprietary |
| **Scope** | Autonomous behaviour — commands, boundaries, forbidden actions | Inline suggestions + Chat coding style |
| **Analogy** | Employee contract — what you can and cannot do | Personal briefing — how to write code here |
| **Location** | Repo root `AGENTS.md` | `.github/copilot-instructions.md` |

### How they work together

```
AGENTS.md                          copilot-instructions.md
"Rules you must follow"     +      "How to write code here"
                ↓
    Copilot Agent that writes
    correct, safe, consistent code
```

`copilot-instructions.md` should reference `AGENTS.md`:
```markdown
## Agent Rules
See AGENTS.md at repo root for what agents are and are not allowed to do autonomously.
```

---

## 18. Full Repository Structure

```
workspace/
├── our-new-project/
│   ├── AGENTS.md                              ← AI agent rules and boundaries
│   ├── CODEBASE.md                            ← living index of classes, APIs, tests
│   ├── TASKS.md                               ← implementation checklist
│   ├── .github/
│   │   ├── copilot-instructions.md            ← Copilot-specific context
│   │   └── workflows/
│   │       └── deploy.yml                     ← CI/CD: build → ACR → AKS
│   ├── docs/
│   │   ├── PRD.md                             ← business goals, acceptance criteria
│   │   ├── TECHNICAL_SPEC.md                  ← what to build
│   │   ├── ARCHITECTURE.md                    ← how to build and deploy
│   │   └── integrations/
│   │       ├── DEPENDENCIES.md                ← all external deps at a glance
│   │       ├── project-a-integration.md       ← how we consume project-A
│   │       └── project-b-integration.md       ← how we consume project-B
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   ├── configmap.yaml
│   │   └── hpa.yaml
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/com/yourcompany/project/
│   │   │   │   ├── api/
│   │   │   │   │   ├── controller/
│   │   │   │   │   ├── dto/
│   │   │   │   │   └── mapper/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/
│   │   │   │   │   ├── service/
│   │   │   │   │   └── repository/
│   │   │   │   ├── infrastructure/
│   │   │   │   │   ├── config/
│   │   │   │   │   ├── client/
│   │   │   │   │   └── persistence/
│   │   │   │   └── common/
│   │   │   │       └── exception/
│   │   │   └── resources/
│   │   │       ├── application.yml
│   │   │       ├── application-dev.yml
│   │   │       ├── application-prod.yml
│   │   │       └── db/migration/
│   │   │           └── V1__create_payment_table.sql
│   │   └── test/kotlin/
│   ├── build.gradle.kts
│   ├── settings.gradle.kts
│   └── Dockerfile
│
├── project-A/
│   └── DISCOVERY.md                           ← reverse-engineered, repo-centric
│
└── project-B/
    └── DISCOVERY.md                           ← reverse-engineered, repo-centric
```

---

## 19. VS Code Setup

### Extensions
- **Kotlin** — language support
- **Spring Boot Dashboard** — run/debug Spring apps
- **Docker** — Dockerfile editing and container management
- **Kubernetes** — k8s manifest editing and cluster management
- **GitHub Copilot + Copilot Chat** — AI pair programmer

### Copilot Tips
- Use **Agent mode** (not just inline completions) for multi-file generation
- Keep `AGENTS.md`, `CODEBASE.md`, and the relevant spec open — Copilot uses open files as context
- Use the multi-root `.code-workspace` file to give Copilot visibility across all repos simultaneously
- Reference `TASKS.md` explicitly in every agent-mode prompt

### The Three Documents to Always Have Open
```
AGENTS.md           ← agent knows what it can and cannot do
CODEBASE.md         ← agent knows what already exists
TASKS.md            ← agent knows what to do next
```

---

*Complete guide — AI-Assisted Development for Kotlin + AKS Projects*
*Fintech / Banking context — GitHub Copilot + VS Code*
