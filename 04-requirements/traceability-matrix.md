# 04 — Matriz de Trazabilidad de Requerimientos (Traceability Matrix)

> **Estado:** 🟢 Estable (Fase 0)  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform

---

## 🗺️ Mapa de Trazabilidad Bidireccional

Esta matriz asegura que cada requerimiento de negocio se traduzca de forma estricta en una historia de usuario técnica, un requisito funcional, criterios de aceptación verificables y casos de prueba.

| ID Plan | Épica de Producto (`EPC-NN`) | Feature (`FEA-NNN`) | Requisito Funcional | Historia de Ejecución (`HU-<MICRO>-NNN`) | Criterios de Aceptación (`AC-*`) | Casos de Prueba (`TC-*`) | ADR Relacionado |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **HU-01** | `EPC-01` | `FEA-001` | `RF-001` (Alta de Organización) | `HU-TENANT-001` | `AC-001`, `AC-002` | `TC-TENANT-001`, `TC-TENANT-002` | `ADR-001`, `ADR-002` |
| **HU-02** | `EPC-01` | `FEA-002` | `RF-002` (Membresías de Equipo) | `HU-TENANT-002` | `AC-003`, `AC-004` | `TC-TENANT-003` | `ADR-002` |
| **HU-03** | `EPC-01` | `FEA-003` | `RF-003` (Autenticación OIDC JWT) | `HU-IAM-001` | `AC-005`, `AC-006` | `TC-IAM-001` | `ADR-002` |
| **HU-04** | `EPC-02` | `FEA-004` | `RF-004` (Gestión de Proyectos) | `HU-WORKSPACE-001` | `AC-007`, `AC-008` | `TC-WORKSPACE-001` | `ADR-001` |
| **HU-05** | `EPC-02` | `FEA-005` | `RF-005` (Gestión de Tareas) | `HU-WORKSPACE-002` | `AC-009`, `AC-010` | `TC-WORKSPACE-002` | `ADR-001` |
| **HU-06** | `EPC-02` | `FEA-006` | `RF-006` (Bloqueo por Aprobación) | `HU-WORKSPACE-003` | `AC-011`, `AC-012` | `TC-WORKSPACE-003` | `ADR-003` |
| **HU-07** | `EPC-03` | `FEA-007` | `RF-007` (Definición de Workflows) | `HU-WORKFLOW-001` | `AC-013`, `AC-014` | `TC-WORKFLOW-001` | `ADR-003` |
| **HU-08** | `EPC-03` | `FEA-008` | `RF-008` (Emisión de Solicitud) | `HU-WORKFLOW-002` | `AC-015`, `AC-016` | `TC-WORKFLOW-002` | `ADR-003` |
| **HU-09** | `EPC-03` | `FEA-009` | `RF-009` (Resolución de Solicitud) | `HU-WORKFLOW-003` | `AC-017`, `AC-018` | `TC-WORKFLOW-003` | `ADR-003` |
| **HU-10** | `EPC-02` | `FEA-006` | `RF-010` (Desbloqueo Asíncrono) | `HU-WORKSPACE-004` | `AC-019`, `AC-020` | `TC-WORKSPACE-004` | `ADR-003` |
| **HU-11** | `EPC-04` | `FEA-010` | `RF-011` (Resumen de Bloqueos) | `HU-ANALYTICS-001` | `AC-021`, `AC-022` | `TC-ANALYTICS-001` | `ADR-004` |
| **HU-12** | `EPC-04` | `FEA-011` | `RF-012` (Consultas IA con RLS) | `HU-ANALYTICS-002` | `AC-023`, `AC-024` | `TC-ANALYTICS-002` | `ADR-004` |
