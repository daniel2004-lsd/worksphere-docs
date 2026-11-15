# 05 — Arquitectura General de WorkSphere

> **Estado:** 🟢 Fase 1 — Definición Técnica  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform

---

## 1. Visión General de la Arquitectura

WorkSphere implementa una **arquitectura distribuida de microservicios orientada a eventos (Event-Driven Architecture)**, diseñada bajo los principios de **Domain-Driven Design (DDD)**, **Clean Architecture**, **Database per Service** y **Multi-Tenancy estricto** con **Row-Level Security (RLS)**.

---

## 2. Topología de Repositorios (Conforme a ADR-004)

El ecosistema descarta el modelo monorepo en favor de una **separación física estricta entre microservicios y bases de datos**:

```
worksphere-ecosystem/
├── worksphere-docs/              # SSOT: Arquitectura, C4, ADRs, Requisitos, Contratos
│
├── Dominio: Tenant Management
│   ├── worksphere-tenant-service/ # API Java / Spring Boot 3
│   └── worksphere-tenant-db/      # Migraciones Liquibase XML + RLS (tenant_db)
│
├── Dominio: Workspace Context
│   ├── worksphere-workspace-service/ # API C# / ASP.NET Core (.NET 9)
│   └── worksphere-workspace-db/      # Migraciones Liquibase XML + RLS (workspace_db)
│
└── Dominio: Workflow Context
    ├── worksphere-workflow-service/  # API Java / Spring Boot 3
    └── worksphere-workflow-db/       # Migraciones Liquibase XML + RLS (workflow_db)
```

---

## 3. Decisiones Arquitectónicas Registradas (ADRs)

Toda decisión relevante en el diseño de WorkSphere se documenta formalmente mediante un **Architecture Decision Record**:

| ID | Título de la Decisión | Estado | Impacto Técnico Clave |
|---|---|:---:|---|
| [`ADR-001`](./decisions/ADR-001-microservices-multitenancy.md) | Arquitectura de Microservicios Distribuidos y Multi-Tenancy | Aceptado | Database-per-Service, comunicación asíncrona, consistencia eventual y división políglota justificada. |
| [`ADR-002`](./decisions/ADR-002-multitenancy-rls-keycloak.md) | Estrategia de Aislamiento Multi-Tenant con Keycloak OIDC y PostgreSQL RLS | Aceptado | Defensa en profundidad: token JWT inmutable con `tenant_id`, interceptor de contexto y RLS en motor de BD. |
| [`ADR-003`](./decisions/ADR-003-async-task-approval-saga.md) | Coordinación de Desbloqueo Operativo Asíncrono mediante RabbitMQ y Saga Coreografiada | Aceptado | Transición `BLOCKED_BY_APPROVAL`, Transactional Outbox y resolución desacoplada sin HTTP inter-servicio. |
| [`ADR-004`](./decisions/ADR-004-multi-repository-architecture.md) | Modelo Multi-Repositorio con Separación de Microservicios y Bases de Datos | **Propuesta** | Separación física de repositorios `*-service` y `*-db`, pipelines de CI/CD independientes y regla Expand/Contract. |

---

## 4. Patrones Fundamentales Aplicados

1. **Database per Service:** Ningún microservicio accede de forma directa a tablas o esquemas ajenos.
2. **Transactional Outbox:** Garantía de cero pérdida de eventos entre el commit local a base de datos y la publicación en el broker.
3. **Idempotencia de Consumidores:** Registro de identificadores de evento (`inbox_processed_events`) para tolerar reentregas de RabbitMQ sin duplicación de efectos.
4. **Row-Level Security (RLS):** Barrera de aislamiento físico en PostgreSQL gobernada en los changelogs de Liquibase.
5. **Estrategia Expand / Contract:** Despliegue de esquemas de datos no destructivo previo a la actualización del código de aplicación.
