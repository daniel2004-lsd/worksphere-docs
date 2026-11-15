# 08 — Diagramas de Arquitectura C4 (Nivel 1 y Nivel 2)

> **Documento:** C4 Model System Architecture  
> **Estado:** 🟢 Estable (Fase 0)  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform

---

## 1. Diagrama C4 Nivel 1: Contexto del Sistema (System Context)

El Diagrama de Contexto muestra a **WorkSphere** en el centro, los diferentes usuarios empresariales que interactúan con él y los sistemas externos con los que se integra o relaciona.

```mermaid
flowchart TD
    subgraph Actores["Usuarios del Sistema"]
        Admin["Administrador Corporativo<br/>(Configuración y gobernanza)"]
        PM["Project / Delivery Manager<br/>(Seguimiento y desbloqueos)"]
        Approver["Director / Aprobador<br/>(Autorización formal)"]
        Engineer["Ingeniero / Colaborador<br/>(Ejecución de tareas)"]
    end

    subgraph WorkSphereSystem["Plataforma WorkSphere (Enterprise Operations Intelligence)"]
        WS["WorkSphere Core Platform<br/>- Aislamiento Multi-Tenant (RLS)<br/>- Proyectos y Tareas críticas<br/>- Flujos de Aprobación inmutables<br/>- IA Explicable sobre hechos autorizados"]
    end

    subgraph ExternalSystems["Ecosistema Externo"]
        GitPlatforms["Plataformas de Código<br/>(GitHub / GitLab / Bitbucket)"]
        SMTP["Servidor de Correo / Notificaciones<br/>(SMTP / Alert Gateway)"]
        LLM["Motor de Inferencia Local<br/>(Ollama / vLLM / RAG)"]
    end

    Admin -->|"Registra organización y miembros"| WS
    PM -->|"Crea proyectos y detecta bloqueos"| WS
    Approver -->|"Resuelve solicitudes con justificación"| WS
    Engineer -->|"Actualiza tareas y solicita aprobaciones"| WS

    GitPlatforms -.->|"Webhooks de commits / PRs"| WS
    WS -.->|"Envía alertas y recordatorios"| SMTP
    WS <-->|"Consultas RAG auditadas"| LLM
```

---

## 2. Diagrama C4 Nivel 2: Contenedores (Container Diagram)

El Diagrama de Contenedores descompone **WorkSphere** en sus aplicaciones web, pasarela de entrada (Gateway), microservicios independientes, bases de datos aisladas y el broker de mensajería asíncrono.

```mermaid
flowchart TD
    subgraph Clientes["Capa de Presentación"]
        SPA["Web Application<br/>(React 18 + Vite + TypeScript)<br/>UI Empresarial & Tableros"]
    end

    subgraph Ingress["Capa de Enrutamiento & Seguridad"]
        Gateway["API Gateway<br/>(Kong Gateway OSS)<br/>Routing, TLS, Rate Limiting"]
        IAM["Identity & Access Management<br/>(Keycloak 24)<br/>OIDC, OAuth2, JWT con tenant_id"]
    end

    subgraph Microservicios["Capa de Microservicios Distribuidos (Database per Service)"]
        TenantSvc["Tenant Service<br/>(Java 21 / Spring Boot 3)<br/>Organizaciones, membresías & Outbox"]
        WorkspaceSvc["Workspace Service<br/>(C# / .NET 9)<br/>Proyectos, tareas & bloqueos"]
        WorkflowSvc["Workflow Service<br/>(Java 21 / Spring Boot 3)<br/>Máquinas de estado de aprobaciones"]
        AnalyticsSvc["Analytics & AI Service<br/>(Python 3.12 / FastAPI)<br/>Métricas & Agente RAG local"]
    end

    subgraph EventBroker["Mensajería Asíncrona Confiable"]
        Broker["RabbitMQ 3.13<br/>Exchange: worksphere.events (Topic)<br/>DLX: worksphere.dlx"]
    end

    subgraph Persistencia["Capa de Datos Aislada (PostgreSQL 16 + RLS)"]
        DB_IAM[("iam_db<br/>(Keycloak)")]
        DB_TENANT[("tenant_db<br/>(Liquibase + RLS)")]
        DB_WORKSPACE[("workspace_db<br/>(Liquibase + RLS)")]
        DB_WORKFLOW[("workflow_db<br/>(Liquibase + RLS)")]
        DB_ANALYTICS[("ClickHouse OLAP<br/>(Telemetría de eventos)")]
    end

    SPA -->|"HTTPS / REST"| Gateway
    SPA -->|"OIDC Login"| IAM

    Gateway -->|"Valida JWT y rutea"| TenantSvc
    Gateway -->|"Valida JWT y rutea"| WorkspaceSvc
    Gateway -->|"Valida JWT y rutea"| WorkflowSvc
    Gateway -->|"Valida JWT y rutea"| AnalyticsSvc

    IAM --> DB_IAM
    TenantSvc --> DB_TENANT
    WorkspaceSvc --> DB_WORKSPACE
    WorkflowSvc --> DB_WORKFLOW
    AnalyticsSvc --> DB_ANALYTICS

    TenantSvc -.->|"Outbox Relay: organization.created"| Broker
    WorkspaceSvc -.->|"Outbox Relay: task.blocked"| Broker
    WorkflowSvc -.->|"Outbox Relay: approval.resolved"| Broker

    Broker -.->|"Consume approval.resolved"| WorkspaceSvc
    Broker -.->|"Stream de eventos"| AnalyticsSvc
```

---

## 3. Justificación de los Límites de Red y Aislamiento

1. **Punto Único de Ingress:** Todo el tráfico externo entra exclusivamente por el **Kong Gateway** (puerto `8000/8005`). Ningún microservicio ni base de datos expone sus puertos directamente hacia clientes de internet.
2. **Database-per-Service Estricto:** Cada microservicio se conecta única y exclusivamente a su propia base de datos mediante credenciales aisladas. No existen *cross-database foreign keys*.
3. **Desacoplamiento Asíncrono de Desbloqueo:** Cuando el `Workflow Service` resuelve una aprobación, no invoca sincrónicamente al `Workspace Service` mediante HTTP (lo cual crearía acoplamiento temporal y riesgo de cascada de fallos). En su lugar, confirma la decisión en su base de datos, escribe en su tabla outbox y emite el evento `approval.resolved` a RabbitMQ; el `Workspace Service` consume el evento y desbloquea la tarea de forma idempotente.
