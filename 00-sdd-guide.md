# WorkSphere — SDD Guide & Execution Plan

> **Estado:** 🟡 Inception / Planificación Inicial  
> **Arquitectura:** SaaS B2B Multi-Tenant — Microservicios Distribuidos Orientados a Eventos  
> **Estándar:** Software Design Documentation (SDD) — *Design Before Code*

---

## 🧭 ¿Qué es SDD en WorkSphere?

**Software Design Documentation (SDD)** establece que el diseño de la arquitectura distribuida, los contratos de API y los esquemas de eventos se definen y validan formalmente **antes** de tirar la primera línea de código en los microservicios.

```
Tradicional:   Código rápido -> Bugs distribuidos -> Caos de integración -> Documentación ausente
WorkSphere SDD: Diseño de Dominios -> Contratos & Eventos -> Pruebas de Aislamiento -> Implementación
```

---

## 🧱 Las Fases de Ejecución

```mermaid
flowchart TD
    P0["Fase 0: Cimientos & Gobernanza<br/>00-governance · 01-context · 02-domain · 05-architecture ADRs"]
    G0(["Gate 0: Aprobación de Mapa de Dominios y Multi-Tenancy"])
    
    P1["Fase 1: Definición de Requerimientos y Contratos<br/>04-requirements · 06-data · 07-api · 14-events"]
    G1(["Gate 1: Congelación de Contratos de API y Eventos v1"])
    
    P2["Fase 2: Plataforma Base & Infraestructura Local<br/>Docker Compose · Kong · Keycloak · Tenant Management (Java/Spring)"]
    G2(["Gate 2: Flujo Auth + Multi-tenant Provisioning Funcional"])

    P3["Fase 3: Núcleo de Negocio & Eventos<br/>Workspace (.NET) · RabbitMQ · Outbox Pattern · Workflow"]
    G3(["Gate 3: Consistencia Eventual y Manejo de Proyectos Verificado"])

    P4["Fase 4: Suscripciones, Archivos y Operaciones<br/>Billing (.NET) · Sagas de Pago · File Mgmt (Go) · Analytics (FastAPI) · Observabilidad"]
    G4(["Gate 4: Sistema Completo Listo para Staging / E2E"])

    P0 --> G0 --> P1 --> G1 --> P2 --> G2 --> P3 --> G3 --> P4 --> G4
```

---

## 📋 Lista de Gates y Criterios de Paso

### Gate 0: Fundamentos y Modelo Multi-Tenant
- [ ] Bounded Contexts y límites de los 8 microservicios delimitados.
- [ ] Estrategia de particionamiento lógico de datos y Row-Level Security (RLS) documentada.
- [ ] ADR-001 (Monorepo vs Multirepo), ADR-002 (Estrategia Multi-tenancy), ADR-003 (Event Broker).

### Gate 1: Especificaciones y Contratos
- [ ] Contratos OpenAPI para endpoints críticos de cada servicio.
- [ ] Catálogo de eventos CloudEvents / JSON Schema para RabbitMQ.
- [ ] Definición de esquemas de tablas con columna `tenant_id` y políticas RLS.

### Gate 2: Plataforma Base
- [ ] `docker-compose.yml` base operativo levantando PostgreSQL, RabbitMQ, Keycloak y Kong.
- [ ] Microservicio `tenant-service` registrando organizaciones y emitiendo `OrganizationCreated`.
- [ ] Keycloak configurado con multi-realm o atributos de organización por tenant.

### Gate 3: Orquestación Asíncrona
- [ ] `workspace-service` consumiendo eventos y aislando tareas por organización.
- [ ] Transactional Outbox implementado para evitar eventos fantasma o perdidos.
- [ ] Pruebas automatizadas de no-contaminación cruzada entre inquilinos (*Tenant Leakage Tests*).
