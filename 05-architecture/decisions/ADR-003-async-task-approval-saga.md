# ADR-003: Asynchronous Operational Unblocking via RabbitMQ and Choreography Saga

- **Status:** Accepted
- **Date:** 2026-11-15
- **Decision Makers:** Daniel (`daniel2004-lsd`) — Software Architect & Technical Lead
- **Scope:** `workspace-service`, `workflow-service`, RabbitMQ Messaging

---

## 1. Context and Problem Statement

The core differentiator of WorkSphere's MVP is real-time operational correlation between delivery tasks and business approvals:
1. An engineering task in `workspace-service` requires a formal sign-off (e.g., budget approval) and transitions to `BLOCKED_BY_APPROVAL`.
2. A formal approval request is managed and audited in `workflow-service`.
3. Once decided (`APPROVED` or `REJECTED`), the affected task in `workspace-service` must reliably unblock and resume execution.

Problem to solve: **How to coordinate state transitions across microservices without synchronous HTTP coupling or distributed 2PC/XA transactions?**

---

## 2. Decision

We adopt an **Asynchronous Event-Driven Choreography Saga backed by the Transactional Outbox Pattern**:

```mermaid
sequenceDiagram
    autonumber
    actor Approver as Director / Approver
    participant WF as Workflow Service (Spring Boot)
    participant WF_DB as workflow_db (PostgreSQL)
    participant Relay as Outbox Publisher Worker
    participant RMQ as RabbitMQ (Exchange: worksphere.events)
    participant WS as Workspace Service (.NET 9)
    participant WS_DB as workspace_db (PostgreSQL)

    Approver->>WF: POST /api/v1/approvals/{id}/decision (APPROVED)
    activate WF
    Note over WF,WF_DB: Atomic Local ACID Transaction
    WF->>WF_DB: UPDATE approval_requests SET status = 'APPROVED', decision_comment = ...
    WF->>WF_DB: INSERT INTO approval_audit_logs (...)
    WF->>WF_DB: INSERT INTO outbox_events (event_type='approval.resolved', payload=...)
    WF_DB-->>WF: Commit OK
    WF-->>Approver: 200 OK (Decision Finalized)
    deactivate WF

    Relay->>WF_DB: Poll PENDING outbox events
    Relay->>RMQ: Publish 'worksphere.workflow.approval.resolved'
    RMQ-->>Relay: ACK
    Relay->>WF_DB: Mark PUBLISHED

    RMQ->>WS: Consume 'worksphere.workflow.approval.resolved'
    activate WS
    WS->>WS_DB: Verify if event_id was already processed (Idempotency)
    WS->>WS_DB: UPDATE tasks SET status = 'IN_PROGRESS' WHERE id = target_task_id
    WS->>WS_DB: INSERT INTO inbox_processed_events (event_id)
    WS-->>RMQ: ACK
    deactivate WS
```

### Mandatory Implementation Standards:
1. **Consumer Idempotency:** The consuming service tracks `event_id` in an `inbox_processed_events` table to handle duplicate deliveries gracefully.
2. **Tenant Boundary Validation:** The consumer ensures the event's `tenant_id` matches the task's tenant before mutating state.
3. **Dead Letter Queue (DLQ):** Messages failing 3 retry attempts are routed to `worksphere.events.dlq` for human inspection.
