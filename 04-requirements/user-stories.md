# 04 — Essential Triad User Stories (Planning Layer)

> **Status:** 🟢 Refined (Phase 0 — Requirements)  
> **Technical Lead & Architect:** Daniel (`daniel2004-lsd`)  
> **Project:** WorkSphere — Enterprise Operations Intelligence Platform  
> **Standard:** Formatted per [`00-governance/agile-conventions.md`](../00-governance/agile-conventions.md)

---

## 📋 HU-01: Register organization with slug validation and outbox

> **Plan ID:** `HU-01` | **Execution ID:** `HU-TENANT-001`  
> **Product Epic:** `EPC-01: Multi-Tenant Organization Isolation & Governance`  
> **Execution Epic:** `EP-TENANT-01: tenant-service (organization lifecycle & outbox)`  
> **Feature:** `FEA-001: Tenant Organization Provisioning`  
> **Priority:** `P1` | **Status:** `Ready`

### 📖 User Story
**As a** Corporate Administrator of a customer company (e.g., *TechSolutions*),  
**I want to** formally register my organization by providing the business name, slug, and administrative email,  
**So that** I obtain a unique `tenant_id`, provision an isolated data workspace, and activate administrative membership.

### 🛡️ Business Rules
- **BR-01:** The corporate slug must be globally unique across the platform, matching regex `^[a-z0-9-]+$`. Duplicates return `409 Conflict`.
- **BR-02:** Organizations are created with status `ACTIVE` and assigned a deterministic `tenant_id` prefixed with `ten_<slug>_<hash>`.
- **BR-03 (Transactional Outbox):** The organization entity and the `worksphere.tenant.organization.created` event must be committed atomically in `tenant_db`.

### 🧪 Acceptance Criteria
- **AC-001 (Happy Path):**  
  **Given** a valid payload with name `"TechSolutions Inc"`, slug `"techsolutions"`, and email `"admin@techsolutions.com"`,  
  **When** `POST /api/v1/organizations` is invoked,  
  **Then** the service returns `201 Created` with the generated `tenant_id` and persists both the organization and the outbox event.
- **AC-002 (Duplicate Slug):**  
  **Given** an existing organization with slug `"techsolutions"`,  
  **When** a registration request arrives with the same slug,  
  **Then** the service returns `409 Conflict` with a descriptive error message without database mutation.

---

## 📋 HU-06: Block task when requiring operational approval

> **Plan ID:** `HU-06` | **Execution ID:** `HU-WORKSPACE-003`  
> **Product Epic:** `EPC-02: Collaborative Project Delivery & Task Tracking`  
> **Execution Epic:** `EP-WORKSPACE-01: workspace-service (project & task lifecycle)`  
> **Feature:** `FEA-006: Blocked by Operational Approval State`  
> **Priority:** `P0` | **Status:** `Ready`

### 📖 User Story
**As a** Technical Lead or Project Manager,  
**I want to** bind a critical task to a formal approval requirement (e.g., cloud budget or architecture sign-off),  
**So that** the task automatically transitions to `BLOCKED_BY_APPROVAL`, providing instant visibility that work is suspended pending external decision.

### 🛡️ Business Rules
- **BR-01:** A task in status `BLOCKED_BY_APPROVAL` cannot be marked as `DONE` or completed by any team member until the bound approval is resolved.
- **BR-02 (Multi-tenancy):** Tasks and approval processes can only be linked if they share the exact same `tenant_id`.
- **BR-03:** On transitioning to `BLOCKED_BY_APPROVAL`, the service publishes `worksphere.workspace.task.blocked` to RabbitMQ.

### 🧪 Acceptance Criteria
- **AC-001 (Transition to Blocked):**  
  **Given** an active task in progress (`IN_PROGRESS`),  
  **When** the manager initiates an approval request of type `"BUDGET_APPROVAL"`,  
  **Then** task status changes to `BLOCKED_BY_APPROVAL`, the `approval_request_id` is linked, and project health metrics update.
- **AC-002 (Unauthorized Closure Rejection):**  
  **Given** a task in status `BLOCKED_BY_APPROVAL`,  
  **When** a user attempts to update status to `DONE`,  
  **Then** the request is rejected with `422 Unprocessable Entity` stating a pending approval is required.

---

## 📋 HU-09: Resolve approval request and publish unblocking event

> **Plan ID:** `HU-09` | **Execution ID:** `HU-WORKFLOW-003`  
> **Product Epic:** `EPC-03: Operational Approval Workflows & Automated Unblocking`  
> **Execution Epic:** `EP-WORKFLOW-01: workflow-service (approval state machine)`  
> **Feature:** `FEA-009: Auditable Decision & Immutable History`  
> **Priority:** `P0` | **Status:** `Ready`

### 📖 User Story
**As an** Authorized Approver or Executive Director,  
**I want to** evaluate a pending approval request, submit my decision (`APPROVED` or `REJECTED`) with a mandatory comment,  
**So that** an immutable audit trail is recorded and the linked task is automatically unblocked.

### 🛡️ Business Rules
- **BR-01:** Only users possessing the authorized role in the organization can submit approval decisions.
- **BR-02 (Immutability):** Once resolved (`APPROVED` or `REJECTED`), a request is permanently locked against further updates.
- **BR-03 (Asynchronous Unblock):** Upon approval, the service emits `worksphere.workflow.approval.resolved` containing `{ tenant_id, target_task_id, decision: "APPROVED", resolved_by, timestamp }`. `workspace-service` consumes this event to transition the task back to active work.

### 🧪 Acceptance Criteria
- **AC-001 (Successful Approval & Event Emission):**  
  **Given** a pending request assigned to an authorized approver,  
  **When** `POST /api/v1/approvals/{id}/decision` is sent with `{ decision: "APPROVED", comment: "Budget verified and approved." }`,  
  **Then** the request state changes to `APPROVED`, an immutable audit log is created, and the unblocking event is published.
- **AC-002 (Rejection Comment Validation):**  
  **Given** a pending approval request,  
  **When** a rejection decision is submitted without a `comment`,  
  **Then** the request is rejected with `400 Bad Request` requiring a justification.
