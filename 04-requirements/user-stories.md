# 04 — Historias de Usuario de la Tríada Esencial (Capa de Planeación)

> **Estado:** 🟢 Refinado (Fase 0 — Requisitos)  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform  
> **Nomenclatura:** Formato estándar definido en [`00-governance/agile-conventions.md`](../00-governance/agile-conventions.md)

---

## 📋 HU-01: Registrar organización con validación de slug y outbox

> **ID Planeación:** `HU-01` | **ID Ejecución:** `HU-TENANT-001`  
> **Épica de Producto:** `EPC-01: Aislamiento y Gobernanza de Organizaciones Inquilinas`  
> **Épica de Ejecución:** `EP-TENANT-01: tenant-service (gestión de organizaciones y outbox)`  
> **Feature:** `FEA-001: Aprovisionamiento de Organización e Inquilino`  
> **Prioridad:** `P1` | **Estatus:** `Listo`

### 📖 Descripción
**Como** Administrador corporativo de una empresa cliente (ej. *TechSolutions*),  
**Quiero** registrar formalmente mi organización ingresando el nombre comercial, slug corporativo y correo de contacto,  
**Para** obtener un `tenant_id` único, aprovisionar mi espacio de trabajo aislado y activar la membresía de administración.

### 🛡️ Reglas de Negocio
- **RN-01:** El slug corporativo debe ser único en toda la plataforma y cumplir el regex `^[a-z0-9-]+$`. Si ya existe, se rechaza con `409 Conflict`.
- **RN-02:** El registro debe crearse en estado `ACTIVE` y generar un `tenant_id` con prefijo `ten_<slug>_<hash>`.
- **RN-03 (Transactional Outbox):** La inserción de la organización y el evento `worksphere.tenant.organization.created` deben guardarse en la misma transacción atómica en `tenant_db`.

### 🧪 Criterios de Aceptación
- **AC-001 (Happy Path):**  
  **Given** un payload válido con nombre `"TechSolutions Inc"`, slug `"techsolutions"` y correo `"admin@techsolutions.com"`,  
  **When** se invoca `POST /api/v1/organizations`,  
  **Then** el sistema responde con `201 Created`, retorna el `tenant_id` generado y persiste el registro en `organizations` y `outbox_events`.
- **AC-002 (Slug Duplicado):**  
  **Given** una organización existente con slug `"techsolutions"`,  
  **When** se solicita un nuevo registro con el mismo slug,  
  **Then** el sistema responde con `409 Conflict` y un mensaje de error descriptivo sin modificar la base de datos.

---

## 📋 HU-06: Bloquear tarea al requerir aprobación operativa

> **ID Planeación:** `HU-06` | **ID Ejecución:** `HU-WORKSPACE-003`  
> **Épica de Producto:** `EPC-02: Gestión Colaborativa de Proyectos y Tareas Críticas`  
> **Épica de Ejecución:** `EP-WORKSPACE-01: workspace-service (gestión de proyectos y tareas)`  
> **Feature:** `FEA-006: Estado Bloqueado por Aprobación Operativa`  
> **Prioridad:** `P0` | **Estatus:** `Listo`

### 📖 Descripción
**Como** Líder Técnico o Gestor de Proyecto,  
**Quiero** vincular una tarea crítica a un requerimiento de aprobación formal (ej. autorización de presupuesto o visto bueno de arquitectura),  
**Para** que la tarea pase automáticamente al estado `BLOCKED_BY_APPROVAL` y el equipo tenga visibilidad inmediata de que el avance depende de una decisión externa.

### 🛡️ Reglas de Negocio
- **RN-01:** Una tarea en estado `BLOCKED_BY_APPROVAL` no puede ser marcada como `DONE` ni cerrada por ningún colaborador hasta que la aprobación asociada se resuelva favorablemente.
- **RN-02 (Multi-tenancy):** Solo se pueden vincular tareas y procesos de aprobación que pertenezcan al mismo `tenant_id`.
- **RN-03:** Al cambiar a `BLOCKED_BY_APPROVAL`, se debe generar un evento `worksphere.workspace.task.blocked` hacia RabbitMQ.

### 🧪 Criterios de Aceptación
- **AC-001 (Transición a Bloqueado):**  
  **Given** una tarea en progreso (`IN_PROGRESS`) dentro de un proyecto,  
  **When** el gestor solicita una aprobación de tipo `"PRESUPUESTO_EXTERNO"` asociada a la tarea,  
  **Then** el estado de la tarea muta a `BLOCKED_BY_APPROVAL`, se asocia el `approval_request_id` y se actualiza el indicador de salud del proyecto.
- **AC-002 (Intento de Cierre No Autorizado):**  
  **Given** una tarea en estado `BLOCKED_BY_APPROVAL`,  
  **When** un colaborador intenta enviar `PATCH /tasks/{id}` con estado `DONE`,  
  **Then** el sistema rechaza la petición con `422 Unprocessable Entity` indicando que existe una aprobación pendiente.

---

## 📋 HU-09: Resolver aprobación y emitir evento de desbloqueo

> **ID Planeación:** `HU-09` | **ID Ejecución:** `HU-WORKFLOW-003`  
> **Épica de Producto:** `EPC-03: Orquestación de Aprobaciones y Desbloqueo Operativo`  
> **Épica de Ejecución:** `EP-WORKFLOW-01: workflow-service (aprobaciones y estados)`  
> **Feature:** `FEA-009: Decisión Auditable e Historial Inmutable`  
> **Prioridad:** `P0` | **Estatus:** `Listo`

### 📖 Descripción
**Como** Director o Aprobador Autorizado de la organización,  
**Quiero** evaluar la solicitud de aprobación pendiente, ingresar mi decisión (`APROBAR` o `RECHAZAR`) junto con una justificación obligatoria,  
**Para** registrar un historial inmutable y desbloquear automáticamente el hito del proyecto vinculado.

### 🛡️ Reglas de Negocio
- **RN-01:** Solo los usuarios con rol de aprobador asignado en la organización pueden resolver la solicitud.
- **RN-02 (Inmutabilidad):** Una vez resuelta (`APPROVED` o `REJECTED`), la solicitud no puede volver a modificarse bajo ningún término.
- **RN-03 (Desbloqueo Asíncrono):** Al aprobarse, se publica el evento `worksphere.workflow.approval.resolved` con payload `{ tenant_id, task_id, decision: "APPROVED", resolved_by, timestamp }`. El servicio de Workspace consumirá este evento de forma idempotente para cambiar el estado de la tarea a `READY` o `IN_PROGRESS`.

### 🧪 Criterios de Aceptación
- **AC-001 (Aprobación y Desbloqueo Exitoso):**  
  **Given** una solicitud en estado `PENDING` asignada al director,  
  **When** el director envía `POST /api/v1/approvals/{id}/decision` con `{ decision: "APPROVED", comment: "Presupuesto validado para el sprint" }`,  
  **Then** la solicitud pasa a estado `APPROVED`, se guarda el registro de auditoría y se emite el evento para que la tarea pase a `IN_PROGRESS`.
- **AC-002 (Rechazo con Justificación):**  
  **Given** una solicitud en estado `PENDING`,  
  **When** el aprobador envía la decisión `REJECTED` sin incluir el campo `comment`,  
  **Then** el sistema rechaza la operación con `400 Bad Request` indicando que el comentario es obligatorio al rechazar.
