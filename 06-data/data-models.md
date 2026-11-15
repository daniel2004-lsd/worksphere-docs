# 06 — Diccionario y Modelo de Datos de la Tríada Esencial

> **Estado:** 🟢 Fase 1 — Definición Técnica  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform  
> **Motor:** PostgreSQL 16 (Database per Service)  
> **Gestión:** Liquibase XML Changelogs + Row-Level Security (RLS)

---

## 🏛️ Topología de Persistencia por Bounded Context

```mermaid
erDiagram
    %% TENANT MANAGEMENT DB
    ORGANIZATIONS {
        uuid id PK
        varchar tenant_id UK
        varchar name
        varchar slug UK
        varchar admin_email
        varchar status
        timestamptz created_at
    }

    MEMBERSHIPS {
        uuid id PK
        varchar tenant_id FK
        uuid organization_id
        varchar user_id
        varchar email
        varchar role
        varchar department
        varchar status
        timestamptz created_at
    }

    %% WORKSPACE CONTEXT DB
    PROJECTS {
        uuid id PK
        varchar tenant_id
        varchar key UK
        varchar name
        text description
        varchar status
        varchar lead_user_id
        timestamptz start_date
        timestamptz target_date
    }

    TASKS {
        uuid id PK
        varchar tenant_id
        uuid project_id
        varchar task_number
        varchar title
        text description
        varchar status
        varchar priority
        varchar assignee_id
        uuid approval_request_id
        timestamptz created_at
    }

    %% WORKFLOW CONTEXT DB
    APPROVAL_REQUESTS {
        uuid id PK
        varchar tenant_id
        varchar process_type
        uuid target_task_id
        varchar requester_id
        varchar required_role
        varchar status
        varchar assigned_approver_id
        text decision_comment
        timestamptz requested_at
        timestamptz resolved_at
    }

    APPROVAL_AUDIT_LOGS {
        uuid id PK
        varchar tenant_id
        uuid approval_request_id
        varchar action
        varchar actor_id
        varchar actor_role
        text comment
        timestamptz timestamp
    }

    ORGANIZATIONS ||--o{ MEMBERSHIPS : "contiene"
    PROJECTS ||--o{ TASKS : "agrupa"
    APPROVAL_REQUESTS ||--o{ APPROVAL_AUDIT_LOGS : "registra historial"
```

> [!IMPORTANT]
> **Sin Llaves Foráneas Inter-Bases de Datos:**  
> `PROJECTS` y `APPROVAL_REQUESTS` residen en diferentes bases de datos físicas (`workspace_db` y `workflow_db`).  
> La relación entre `TASKS.approval_request_id` y `APPROVAL_REQUESTS.id` es estrictamente **lógica** y se coordina mediante eventos en RabbitMQ y validación del `tenant_id`.

---

## 1. Esquema `tenant_db` (Tenant Management Service)

### 1.1. Tabla: `organizations`
Almacena las empresas cliente dadas de alta en la plataforma.
- `id` (UUID, PK, Default `gen_random_uuid()`): Identificador interno.
- `tenant_id` (VARCHAR(64), UNIQUE, NOT NULL): Clave canónica de particionamiento (ej. `ten_techsolutions_15d1e947`).
- `name` (VARCHAR(120), NOT NULL): Razón social o nombre comercial.
- `slug` (VARCHAR(64), UNIQUE, NOT NULL): Identificador URL corporativo normalizado.
- `admin_email` (VARCHAR(120), NOT NULL): Correo del administrador inicial.
- `status` (VARCHAR(30), NOT NULL, Default `'ACTIVE'`): `ACTIVE`, `SUSPENDED`, `DELETED`.
- `created_at`, `updated_at` (TIMESTAMPTZ, NOT NULL).

### 1.2. Tabla: `memberships`
Asocia usuarios con su rol y departamento dentro de la organización.
- `id` (UUID, PK).
- `tenant_id` (VARCHAR(64), NOT NULL, Indexado): Inquilino propietario.
- `organization_id` (UUID, NOT NULL, FK a `organizations.id`).
- `user_id` (VARCHAR(64), NOT NULL): ID del usuario emitido por Keycloak (`sub`).
- `email` (VARCHAR(120), NOT NULL).
- `role` (VARCHAR(40), NOT NULL): `TENANT_ADMIN`, `PROJECT_MANAGER`, `APPROVER`, `ENGINEER`.
- `department` (VARCHAR(60)): Ej. "Engineering", "Operations", "Finance".
- `status` (VARCHAR(30), NOT NULL): `ACTIVE`, `INVITED`, `REVOKED`.

---

## 2. Esquema `workspace_db` (Workspace Service — .NET 9)

### 2.1. Tabla: `projects`
- `id` (UUID, PK).
- `tenant_id` (VARCHAR(64), NOT NULL, Indexado): Aislamiento RLS.
- `key` (VARCHAR(10), NOT NULL): Prefijo corto de proyecto (ej. `WKS`, `ALPHA`).
- `name` (VARCHAR(120), NOT NULL).
- `description` (TEXT).
- `status` (VARCHAR(30), NOT NULL): `PLANNING`, `ACTIVE`, `ON_HOLD`, `COMPLETED`.
- `lead_user_id` (VARCHAR(64), NOT NULL): Responsable del proyecto.
- `start_date`, `target_date` (TIMESTAMPTZ).

### 2.2. Tabla: `tasks`
- `id` (UUID, PK).
- `tenant_id` (VARCHAR(64), NOT NULL, Indexado): Aislamiento RLS.
- `project_id` (UUID, NOT NULL, FK a `projects.id`).
- `task_number` (VARCHAR(20), NOT NULL): Ej. `WKS-104`.
- `title` (VARCHAR(150), NOT NULL).
- `description` (TEXT).
- `status` (VARCHAR(30), NOT NULL): `TODO`, `IN_PROGRESS`, `BLOCKED_BY_APPROVAL`, `READY_FOR_REVIEW`, `DONE`.
- `priority` (VARCHAR(20), NOT NULL): `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.
- `assignee_id` (VARCHAR(64)): Usuario asignado.
- `approval_request_id` (UUID): ID de la solicitud de aprobación externa en `workflow_db`.
- `created_at`, `updated_at` (TIMESTAMPTZ, NOT NULL).

---

## 3. Esquema `workflow_db` (Workflow Service — Spring Boot 3)

### 3.1. Tabla: `approval_requests`
- `id` (UUID, PK).
- `tenant_id` (VARCHAR(64), NOT NULL, Indexado): Aislamiento RLS.
- `process_type` (VARCHAR(50), NOT NULL): `BUDGET_APPROVAL`, `SCOPE_CHANGE`, `RESOURCE_ALLOCATION`, `TECH_VALIDATION`.
- `target_task_id` (UUID, NOT NULL): ID lógico de la tarea afectada en `workspace_db`.
- `requester_id` (VARCHAR(64), NOT NULL): Usuario que solicita la aprobación.
- `required_role` (VARCHAR(40), NOT NULL): Rol autorizado para aprobar (ej. `APPROVER`, `TENANT_ADMIN`).
- `status` (VARCHAR(30), NOT NULL): `PENDING`, `APPROVED`, `REJECTED`, `CANCELLED`.
- `assigned_approver_id` (VARCHAR(64)): Aprobador específico si aplica.
- `decision_comment` (TEXT): Justificación obligatoria de la resolución.
- `requested_at` (TIMESTAMPTZ, NOT NULL).
- `resolved_at` (TIMESTAMPTZ): Momento exacto de la decisión.

### 3.2. Tabla: `approval_audit_logs` (Historial Inmutable)
- `id` (UUID, PK).
- `tenant_id` (VARCHAR(64), NOT NULL).
- `approval_request_id` (UUID, NOT NULL, FK a `approval_requests.id`).
- `action` (VARCHAR(30), NOT NULL): `SUBMITTED`, `ASSIGNED`, `APPROVED`, `REJECTED`.
- `actor_id` (VARCHAR(64), NOT NULL).
- `actor_role` (VARCHAR(40), NOT NULL).
- `comment` (TEXT).
- `timestamp` (TIMESTAMPTZ, NOT NULL, Default `NOW()`).

---

## 4. Políticas Row-Level Security (RLS) Obligatorias

En cada una de las tablas anteriores (`organizations`, `memberships`, `projects`, `tasks`, `approval_requests`, `approval_audit_logs`) el changelog inicial de Liquibase aplica:

```sql
ALTER TABLE <table_name> ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON <table_name>
    FOR ALL
    TO PUBLIC
    USING (
        tenant_id = current_setting('app.current_tenant_id', true)
        OR current_setting('app.current_tenant_id', true) = 'SYSTEM'
    );
```
