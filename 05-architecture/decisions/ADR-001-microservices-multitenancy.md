# ADR-001: Distributed Microservices Architecture and Multi-Tenancy

- **Status:** Accepted
- **Date:** 2026-11-15
- **Decision Makers:** Daniel (`daniel2004-lsd`) — Software Architect & Technical Lead
- **Scope:** Universal across all WorkSphere backend services

---

## 1. Context and Problem Statement

WorkSphere is an enterprise B2B SaaS platform designed to support multiple organizations managing projects, workflows, and operations. The platform requires:
- Strict logical data isolation between customer companies.
- Independent scalability across distinct workloads (e.g., analytical telemetry vs. transactional state machines).
- High fault tolerance: failure in non-critical components (e.g., notifications) must never disrupt core delivery operations.

---

## 2. Decision

We adopt a **distributed microservices architecture with Database-per-Service and Event-Driven asynchronous communication**:

1. **Logical Multi-Tenancy:** Single shared physical database cluster per microservice, using a discriminator `tenant_id` column protected by **PostgreSQL Row-Level Security (RLS)** as defense-in-depth.
2. **Event-Driven Resilience:** Asynchronous communication decoupled by **RabbitMQ** with mandatory **Transactional Outbox** implementation to eliminate dual-write hazards.
3. **Justified Polyglot Distribution:**
   - **Java / Spring Boot 3:** Complex enterprise domains with rich lifecycle state machines (`tenant-service`, `workflow-service`).
   - **C# / ASP.NET Core (.NET 9):** High-throughput transactional data operations with strict contract validation (`workspace-service`, `billing-service`).
   - **Go 1.22:** Concurrent, I/O-intensive network proxies and media pipelines (`notification-service`, `file-service`).
   - **Python 3.12 / FastAPI:** Analytical processing, aggregation, and local LLM/RAG pipelines (`analytics-service`).
   - **Keycloak:** Off-the-shelf, open-source identity provider managing OpenID Connect (OIDC), OAuth2, and RBAC token issuance.

---

## 3. Consequences

### Positive:
- Total fault isolation: failure of one microservice does not bring down the entire platform.
- Granular vertical scaling based on CPU and memory demands of specific domains.
- Data sovereignty and defense-in-depth enforcement across all operational layers.

### Operational Challenges & Mitigations:
- **Distributed Complexity:** Mitigated by strict Software Design Documentation (SDD), formal OpenAPI/CloudEvents contracts, and OpenTelemetry tracing.
- **Eventual Consistency:** Acknowledged by design; cross-service workflows are coordinated via Choreography Sagas and idempotent consumers.
