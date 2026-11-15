# 08 — C4 Architecture Diagrams (Level 1 & Level 2)

> **Document:** C4 Model System Architecture  
> **Status:** 🟢 Baseline  
> **Technical Lead & Architect:** Daniel (`daniel2004-lsd`)  
> **Project:** WorkSphere — Enterprise Operations Intelligence Platform

---

## 1. C4 Level 1: System Context Diagram

The System Context diagram illustrates **WorkSphere** at the center, its enterprise user personas, and external supporting systems.

```mermaid
flowchart TD
    subgraph Users["System Users"]
        Admin["Corporate Administrator<br/>(Organization setup & access governance)"]
        PM["Delivery / Project Manager<br/>(Milestone tracking & bottleneck monitoring)"]
        Approver["Executive Approver<br/>(Formal sign-offs & budget authorizations)"]
        Engineer["Software Engineer / Member<br/>(Task execution & approval requests)"]
    end

    subgraph WorkSphereSystem["WorkSphere Platform (Enterprise Operations Intelligence)"]
        WS["WorkSphere Core System<br/>- Multi-tenant PostgreSQL RLS isolation<br/>- Project delivery and task tracking<br/>- Immutable approval audit logs<br/>- Local explainable AI insights"]
    end

    subgraph ExternalSystems["External Ecosystem"]
        GitPlatforms["Code Repositories<br/>(GitHub / GitLab / Bitbucket)"]
        SMTP["Outbound Notifications<br/>(SMTP Gateway / Alerts)"]
        LLM["Local Inference Engine<br/>(Ollama / vLLM / Local RAG)"]
    end

    Admin -->|"Manages organizations and memberships"| WS
    PM -->|"Tracks projects and unblocks delivery"| WS
    Approver -->|"Resolves approval requests with comments"| WS
    Engineer -->|"Updates tasks and requests approvals"| WS

    GitPlatforms -.->|"Webhook events (commits, PRs)"| WS
    WS -.->|"Dispatches email alerts"| SMTP
    WS <-->|"Permission-aware RAG queries"| LLM
```

---

## 2. C4 Level 2: Container Diagram

The Container diagram decomposes **WorkSphere** into client applications, security ingress, distributed microservices, independent databases, and asynchronous message brokers.

```mermaid
flowchart TD
    subgraph Presentation["Presentation Layer"]
        SPA["Web Application<br/>(React 18 + Vite + TypeScript)<br/>Enterprise UI & Operational Dashboards"]
    end

    subgraph Ingress["Ingress & Security Layer"]
        Gateway["API Gateway<br/>(Kong Gateway OSS)<br/>Routing, TLS, Rate Limiting"]
        IAM["Identity & Access Management<br/>(Keycloak 24)<br/>OIDC, OAuth2, Tenant-enriched JWTs"]
    end

    subgraph Microservices["Distributed Microservices Layer (Database-per-Service)"]
        TenantSvc["Tenant Service<br/>(Java 21 / Spring Boot 3)<br/>Organizations & Outbox Relay"]
        WorkspaceSvc["Workspace Service<br/>(C# / ASP.NET Core .NET 9)<br/>Projects, Tasks & Blockers"]
        WorkflowSvc["Workflow Service<br/>(Java 21 / Spring Boot 3)<br/>Approval State Machine & Audit"]
        AnalyticsSvc["Analytics & AI Service<br/>(Python 3.12 / FastAPI)<br/>Telemetry & Local RAG Agent"]
    end

    subgraph Broker["Asynchronous Message Broker"]
        RabbitMQ["RabbitMQ 3.13<br/>Topic Exchange: worksphere.events<br/>DLX: worksphere.dlx"]
    end

    subgraph Persistence["Isolated Data Layer (PostgreSQL 16 + Liquibase + RLS)"]
        DB_IAM[("iam_db<br/>(Keycloak)")]
        DB_TENANT[("tenant_db<br/>(Liquibase YAML + RLS)")]
        DB_WORKSPACE[("workspace_db<br/>(Liquibase YAML + RLS)")]
        DB_WORKFLOW[("workflow_db<br/>(Liquibase YAML + RLS)")]
        DB_ANALYTICS[("ClickHouse OLAP<br/>(Telemetry Events)")]
    end

    SPA -->|"HTTPS / REST"| Gateway
    SPA -->|"OIDC Login"| IAM

    Gateway -->|"Validates JWT & Routes"| TenantSvc
    Gateway -->|"Validates JWT & Routes"| WorkspaceSvc
    Gateway -->|"Validates JWT & Routes"| WorkflowSvc
    Gateway -->|"Validates JWT & Routes"| AnalyticsSvc

    IAM --> DB_IAM
    TenantSvc --> DB_TENANT
    WorkspaceSvc --> DB_WORKSPACE
    WorkflowSvc --> DB_WORKFLOW
    AnalyticsSvc --> DB_ANALYTICS

    TenantSvc -.->|"Outbox: organization.created"| RabbitMQ
    WorkspaceSvc -.->|"Outbox: task.blocked"| RabbitMQ
    WorkflowSvc -.->|"Outbox: approval.resolved"| RabbitMQ

    RabbitMQ -.->|"Consumes approval.resolved"| WorkspaceSvc
    RabbitMQ -.->|"Event Stream Telemetry"| AnalyticsSvc
```

---

## 3. Network Boundaries & Operational Security Rules

1. **Single Public Ingress:** All external client traffic terminates at **Kong API Gateway**. Microservices, databases, and message brokers are isolated within private Docker bridge / Kubernetes networks.
2. **Database Isolation:** Microservices communicate exclusively over documented contracts; direct cross-database queries are strictly blocked.
3. **Asynchronous Task Unblocking:** `Workflow Service` never calls `Workspace Service` over synchronous HTTP upon approval completion. State changes are communicated via RabbitMQ events backed by the Transactional Outbox pattern.
