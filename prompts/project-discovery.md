# Kotlin + AKS Project — AI-Assisted Development Guide
> A structured reference derived from a consulting session on bootstrapping a Kotlin Gradle API project with GitHub Copilot, deployed to AKS.

---

## Table of Contents
1. [Overall Approach](#overall-approach)
2. [Document Hierarchy](#document-hierarchy)
3. [Phase 1 — Requirements to Structured Spec](#phase-1--requirements-to-structured-spec)
4. [Phase 2 — Project Scaffolding](#phase-2--project-scaffolding)
5. [Phase 3 — Iterative Development with Copilot](#phase-3--iterative-development-with-copilot)
6. [Phase 4 — AKS Deployment Artifacts](#phase-4--aks-deployment-artifacts)
7. [Phase 5 — Quality & Iteration Loop](#phase-5--quality--iteration-loop)
8. [Handling External Dependencies (project-A, project-B)](#handling-external-dependencies-project-a-project-b)
9. [Discovery Documents Inside Dependency Repos](#discovery-documents-inside-dependency-repos)
10. [AGENTS.md — What It Is and Why You Need It](#agentsmd--what-it-is-and-why-you-need-it)
11. [CODEBASE.md — The Technical Glossary](#codebasemd--the-technical-glossary)
12. [AGENTS.md vs copilot-instructions.md](#agentsmd-vs-copilot-instructionsmd)
13. [Full Repository Structure](#full-repository-structure)
14. [Recommended VS Code Setup](#recommended-vs-code-setup)

---

## Overall Approach

Starting from a raw `requirements.txt`, the goal is to produce a production-ready Kotlin/Gradle API deployed to AKS — using GitHub Copilot as an AI pair programmer throughout.

The correct sequence before writing any code:

```
requirements.txt
      ↓
PRD.md                    ← business goals, user stories, acceptance criteria
      ↓
TECHNICAL_SPEC.md         ← derived from PRD (what to build)
      ↓
ARCHITECTURE.md           ← derived from spec + tech decisions (how to build it)
      ↓
AGENTS.md                 ← agent behaviour rules (before any code)
DISCOVERY.md              ← for project-A and project-B repos
      ↓
copilot-instructions.md   ← references all of the above
      ↓
first line of code
```

---

## Document Hierarchy

| Document | Audience | Answers |
|---|---|---|
| `PRD.md` | Stakeholders, PO, QA | Why are we building this? |
| `TECHNICAL_SPEC.md` | Developers, QA | What exactly are we building? |
| `ARCHITECTURE.md` | Developers, DevOps | How is it structured and deployed? |
| `DISCOVERY.md` | Humans + AI — anyone new to a repo | What does this repo do? |
| `CODEBASE.md` | Developers, AI agents | What classes, APIs, modules exist right now? |
| `copilot-instructions.md` | GitHub Copilot specifically | How should Copilot assist in this repo? |
| `AGENTS.md` | All AI agents (Copilot, Cursor, Claude Code) | How should an AI agent autonomously act? |

---

## Phase 1 — Requirements to Structured Spec

### Step 1 — Generate `TECHNICAL_SPEC.md`

Open the `requirements.txt` in VS Code and ask Copilot Chat:

> *"Based on these requirements, generate a technical spec with: functional requirements, API endpoints (REST), domain entities, and non-functional requirements (auth, latency, error handling). Save to `docs/TECHNICAL_SPEC.md`."*

**Contains:**
- Functional requirements (what the system does)
- REST API endpoints (routes, methods, payloads)
- Domain entities (data models)
- Non-functional requirements (auth, latency, error handling)

### Step 2 — Generate `ARCHITECTURE.md`

> *"Based on the technical spec, generate an architecture document covering: service responsibilities, tech stack decisions, and AKS deployment topology. Save to `docs/ARCHITECTURE.md`."*

**Contains:**
- Technology choices and rationale
- Service structure (single service vs modules)
- AKS topology (ingress, pods, ACR, secrets)
- Infrastructure decisions (ConfigMaps, HPA, namespaces)

### Step 3 — Generate `PRD.md`

> *"Based on these requirements, generate a lightweight PRD covering: business goals, user stories, acceptance criteria, and out-of-scope items. Save to `docs/PRD.md`."*

**Why PRD matters in fintech:**
- External stakeholders or clients signing off
- QA/testers need acceptance criteria
- Audit trail — banking projects need documented rationale
- Requirements `.txt` is often ambiguous or incomplete

---

## Phase 2 — Project Scaffolding

Bootstrap the Gradle project using Copilot Agent mode:

> *"Create a Kotlin Spring Boot 3 project with Gradle Kotlin DSL, featuring: REST API, Spring Data JPA, Flyway migrations, OpenAPI/Swagger, health actuator endpoints, and Docker support."*

**Key files Copilot should generate:**
- `build.gradle.kts` — dependency management
- `settings.gradle.kts` — multi-module if needed
- `src/main/resources/application.yml` — with profiles (`dev`, `prod`)
- `Dockerfile` — multi-stage build (builder + slim JRE image)

---

## Phase 3 — Iterative Development with Copilot

### `copilot-instructions.md`

Create `.github/copilot-instructions.md` — Copilot reads this on every session:

```markdown
# Project Context
- Kotlin Spring Boot 3 REST API
- Deployed to AKS (Azure Kubernetes Service)
- Domain: [your domain from requirements]
- Coding conventions: functional style, coroutines where async needed
- Test strategy: JUnit 5 + MockK, integration tests with Testcontainers

## External Dependencies
See docs/integrations/ for full integration specs.
Key deps: project-A (identity), project-B (accounts)

## Agent Behaviour Rules
See AGENTS.md at repo root.
```

### Development Loop Per Feature

1. Describe the requirement in a Copilot Chat prompt
2. Generate entity → repository → service → controller
3. Request unit + integration tests in the same pass
4. Review, adjust, commit

---

## Phase 4 — AKS Deployment Artifacts

Ask Copilot to generate full K8s manifests:

> *"Generate Kubernetes manifests for this Spring Boot app on AKS: Deployment (2 replicas, resource limits), Service (ClusterIP), Ingress (nginx), ConfigMap for app config, HorizontalPodAutoscaler, and a GitHub Actions CI/CD pipeline that builds, pushes to ACR, and deploys to AKS."*

**Generated files:**
- `k8s/deployment.yaml`
- `k8s/service.yaml`
- `k8s/ingress.yaml`
- `k8s/configmap.yaml`
- `k8s/hpa.yaml`
- `.github/workflows/deploy.yml`

---

## Phase 5 — Quality & Iteration Loop

| What | How |
|---|---|
| **OpenAPI spec** | Copilot generates from controllers; validate with Swagger UI |
| **Test coverage** | Ask Copilot for missing test cases per endpoint |
| **Security** | Prompt for Spring Security config (JWT/OAuth2) |
| **Observability** | Ask for Micrometer + Prometheus annotations |

---

## Handling External Dependencies (project-A, project-B)

### Multi-Root Workspace Setup

Add all repos to one `.code-workspace` file so Copilot references files across all roots:

```json
{
  "folders": [
    { "name": "our-project", "path": "./our-new-project" },
    { "name": "project-A",   "path": "./project-A" },
    { "name": "project-B",   "path": "./project-B" }
  ]
}
```

### Priority Files to Analyse Per Dependency Repo

| File Type | What It Tells You |
|---|---|
| `*Controller.kt` / `*Resource.kt` | Exposed endpoints |
| `*DTO.kt` / `*Request.kt` / `*Response.kt` | Data contracts |
| `OpenAPI` / `swagger.yml` | Full API spec if available |
| `application.yml` | Ports, service names, feature flags |
| `*Client.kt` / `*FeignClient.kt` | How they call others |
| `README.md` | Often overlooked but worth checking |

### Copilot Prompt to Generate Integration Docs

> *"Analyse the open files from project-A. Generate a markdown integration reference doc and save it to `docs/integrations/project-a-integration.md` covering: purpose, base URL pattern, all endpoints, request/response schemas, auth mechanism, error format, and key domain concepts."*

Repeat for project-B.

### Integration Docs Location

```
our-new-project/
└── docs/
    └── integrations/
        ├── DEPENDENCIES.md              ← summary of all external deps
        ├── project-a-integration.md     ← written from OUR perspective
        └── project-b-integration.md
```

### Generate `DEPENDENCIES.md`

> *"Based on `docs/integrations/project-a-integration.md` and `docs/integrations/project-b-integration.md`, generate a summary of all external dependencies and save to `docs/integrations/DEPENDENCIES.md`."*

### Relationship Between Integration Files

```
project-A/DISCOVERY.md           ← repo-centric: what is this service?
project-B/DISCOVERY.md
        ↓ summarised into ↓
our-new-project/docs/integrations/project-a-integration.md  ← consumer-centric: how do WE use it?
our-new-project/docs/integrations/project-b-integration.md
        ↓ summarised into ↓
our-new-project/docs/integrations/DEPENDENCIES.md
```

### If Repos Have No Documentation

1. **Check for tests** — integration/contract tests are the best real documentation
2. **Check git log** — commit messages reveal intent
3. **Run the service locally** — hit it with a REST client, let Copilot generate docs from real responses
4. **Ask the team** — come with specific questions, not blank ones

---

## Discovery Documents Inside Dependency Repos

Create a `DISCOVERY.md` at the root of **each** dependency repo.

### Why `DISCOVERY.md` and Not `README.md`?

| File | Purpose |
|---|---|
| `README.md` | Already exists, owned by that team — don't pollute it |
| `DISCOVERY.md` | Your reverse-engineering findings — external perspective, non-intrusive |

### Copilot Prompt

> *"Based on the open files, generate a `DISCOVERY.md` for project-A covering: project purpose, exposed REST endpoints, request/response models, authentication mechanism, key domain entities, known integration quirks, and open questions."*

### `DISCOVERY.md` Template

```markdown
# project-A — Discovery Document

> Generated by: [your name]
> Date: [date]
> Context: Reverse-engineered for integration with our-new-project

## Purpose
What this service does in one paragraph.

## Tech Stack
- Language/Framework
- Database
- Auth mechanism

## Exposed REST Endpoints
| Method | Path | Request | Response | Auth Required |
|--------|------|---------|----------|---------------|
| POST   | /api/customers | CustomerRequest | CustomerResponse | Yes - Bearer |

## Key Domain Entities
- **Customer** — fields, constraints
- **Account** — fields, constraints

## Authentication
How to obtain and pass tokens.

## Error Response Format
```json
{ "code": "...", "message": "..." }
```

## Known Quirks / Gotchas
- Anything non-obvious discovered during analysis

## Open Questions
- [ ] Question for the project-A team
- [ ] Unclear behaviour around X
```

> **Tip:** Commit `DISCOVERY.md` to a separate branch like `docs/discovery` — don't push to main without the owning team's consent.

---

## AGENTS.md — What It Is and Why You Need It

`AGENTS.md` is an **instruction file for AI coding agents** — telling them not just what the project is, but how to behave when autonomously working inside it.

- `copilot-instructions.md` — passive assistant context (inline suggestions, chat)
- `AGENTS.md` — autonomous agent rules (multi-step decisions, running commands, editing multiple files)

### Why It Matters for This Project

| Risk | Without `AGENTS.md` | With `AGENTS.md` |
|---|---|---|
| Agent adds a dependency | Adds it silently | Flags for review |
| Agent touches k8s manifests | Edits prod config freely | Knows to stay out |
| Agent writes a DTO | Random structure | Follows Kotlin data class convention |
| Agent writes a test | Maybe MockK, maybe Mockito | Always MockK + JUnit 5 |
| Agent calls project-A | Guesses the API | Uses your integration doc |

### `AGENTS.md` Template

```markdown
# AGENTS.md

## Project Overview
Kotlin Spring Boot 3 REST API deployed to AKS.
Gradle Kotlin DSL.

## Build Commands
- Build: `./gradlew build`
- Test: `./gradlew test`
- Run locally: `./gradlew bootRun --args='--spring.profiles.active=dev'`
- Lint: `./gradlew ktlintCheck`

## Code Conventions
- Use Kotlin data classes for DTOs
- Coroutines for async operations
- No nullable types without explicit justification
- All endpoints must have OpenAPI annotations

## Testing Rules
- Unit tests: JUnit 5 + MockK
- Integration tests: Testcontainers
- Every new endpoint needs both unit and integration test
- Do not delete existing tests

## File Structure Rules
- Controllers go in `api/controller/`
- Services go in `domain/service/`
- DTOs go in `api/dto/`
- Entities go in `domain/model/`

## What Agents MUST NOT Do
- Do not modify `k8s/` manifests without explicit instruction
- Do not change `application-prod.yml`
- Do not add new dependencies without flagging for review
- Do not commit secrets or hardcoded URLs

## External Dependencies
- project-A: identity service — see docs/integrations/project-a-integration.md
- project-B: accounts service — see docs/integrations/project-b-integration.md

## PR / Commit Rules
- Commit messages: conventional commits format
- One logical change per commit
```

---

## CODEBASE.md — The Technical Glossary

`CODEBASE.md` is a **living technical reference** — the index of everything that exists in the repo.

### What Goes Inside It

```markdown
# CODEBASE.md

## Module Structure
List of all modules and what each one owns.

## Package Structure
com.yourcompany.project
├── api/
│   ├── controller/     — REST controllers
│   ├── dto/            — Request/Response models
│   └── mapper/         — DTO ↔ Domain mappers
├── domain/
│   ├── model/          — Entities / domain objects
│   ├── service/        — Business logic
│   └── repository/     — Data access interfaces
├── infrastructure/
│   ├── config/         — Spring configs, beans
│   ├── client/         — External service clients
│   └── persistence/    — JPA implementations
└── common/
    └── exception/      — Global error handling

## Classes Glossary
| Class | Package | Responsibility |
|-------|---------|---------------|
| PaymentController | api/controller | Handles POST /payments |
| PaymentService | domain/service | Core payment business logic |
| PaymentRepository | domain/repository | DB access for payments |
| PaymentRequest | api/dto | Inbound DTO for payment creation |
| ProjectAClient | infrastructure/client | Feign client for project-A |

## REST API Inventory
| Method | Path | Controller | Description |
|--------|------|-----------|-------------|
| POST | /api/v1/payments | PaymentController | Initiate payment |
| GET  | /api/v1/payments/{id} | PaymentController | Get payment status |

## Test Classes
| Test Class | Type | Covers |
|-----------|------|--------|
| PaymentControllerTest | Unit | Controller layer, MockMvc |
| PaymentServiceTest | Unit | Business logic, MockK |
| PaymentIntegrationTest | Integration | Full flow, Testcontainers |

## Configuration Files
| File | Profile | Purpose |
|------|---------|---------|
| application.yml | all | Base config |
| application-dev.yml | dev | Local dev overrides |
| application-prod.yml | prod | AKS production config |

## Database
- Migrations managed by Flyway
- Scripts location: `src/main/resources/db/migration/`
- Naming convention: `V1__description.sql`, `V2__description.sql`
```

### Keeping It Up to Date

**Option 1 — Periodic Copilot refresh:**
> *"Scan the current codebase and update `CODEBASE.md` with any new classes, endpoints, or test classes added since the last update."*

**Option 2 — Auto-generate from code:**
Use Dokka (Kotlin), SpringDoc, or a custom Gradle task to generate parts of it automatically.

---

## AGENTS.md vs copilot-instructions.md

They are **complementary**, not the same.

| | `AGENTS.md` | `copilot-instructions.md` |
|---|---|---|
| **Audience** | Any AI agent — Copilot, Cursor, Claude Code, Codex | GitHub Copilot **only** |
| **Standard** | Open, emerging industry standard | GitHub/Microsoft proprietary |
| **Scope** | Autonomous agent behaviour — commands, rules, boundaries | Inline suggestions + Copilot Chat behaviour |
| **Focus** | What agents are allowed/forbidden to do | How Copilot should write code for you |
| **Location** | Repo root `AGENTS.md` | `.github/copilot-instructions.md` |

### Simple Analogy

> `AGENTS.md` is like an **employee contract** — rules, boundaries, what you can and cannot do autonomously.
>
> `copilot-instructions.md` is like a **personal briefing** — your coding style, preferences, context for your specific assistant.

### How They Work Together

```
AGENTS.md
"Here are the rules you must follow"
        +
copilot-instructions.md
"Here is the coding style and context"
        ↓
Copilot Agent that writes correct code
AND respects project boundaries
```

`copilot-instructions.md` can reference `AGENTS.md`:
```markdown
# copilot-instructions.md
For agent behaviour rules and boundaries, see AGENTS.md at repo root.
```

---

## Full Repository Structure

```
our-new-project/
├── AGENTS.md                          ← agent behaviour rules
├── CODEBASE.md                        ← living technical index
├── .github/
│   ├── copilot-instructions.md        ← Copilot-specific context
│   └── workflows/
│       └── deploy.yml                 ← CI/CD pipeline
├── docs/
│   ├── PRD.md                         ← business goals, acceptance criteria
│   ├── TECHNICAL_SPEC.md              ← what to build
│   ├── ARCHITECTURE.md                ← how to build and deploy
│   └── integrations/
│       ├── DEPENDENCIES.md            ← summary of all external deps
│       ├── project-a-integration.md   ← consumer-centric integration ref
│       └── project-b-integration.md
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── hpa.yaml
├── src/
│   ├── main/
│   │   ├── kotlin/
│   │   │   └── com/yourcompany/project/
│   │   │       ├── api/
│   │   │       │   ├── controller/
│   │   │       │   ├── dto/
│   │   │       │   └── mapper/
│   │   │       ├── domain/
│   │   │       │   ├── model/
│   │   │       │   ├── service/
│   │   │       │   └── repository/
│   │   │       ├── infrastructure/
│   │   │       │   ├── config/
│   │   │       │   ├── client/
│   │   │       │   └── persistence/
│   │   │       └── common/
│   │   │           └── exception/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/
│   └── test/
│       └── kotlin/
├── build.gradle.kts
├── settings.gradle.kts
└── Dockerfile

project-A/
└── DISCOVERY.md                       ← reverse-engineered reference

project-B/
└── DISCOVERY.md                       ← reverse-engineered reference
```

---

## Recommended VS Code Setup

**Extensions:**
- Kotlin
- Spring Boot Dashboard
- Docker
- Kubernetes
- GitHub Copilot + Copilot Chat

**Copilot Tips:**
- Use **Agent mode** (not just completions) for multi-file generation
- Keep `ARCHITECTURE.md`, `AGENTS.md`, and `CODEBASE.md` open — Copilot includes open files as context
- Use the multi-root `.code-workspace` to give Copilot visibility across all repos

---

*Generated from consulting session — AI-Assisted Development for Kotlin + AKS Projects*
