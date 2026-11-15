# 00 — Definition of Ready (DoR)

> **Propósito:** Ninguna Historia de Usuario puede pasar al estado **Listo (Ready)** para ser tomada por un desarrollador o agente de IA sin cumplir con cada uno de los siguientes puntos:

1. [ ] **Identificadores Completos:** Posee su ID de planeación (`HU-NN`) y su ID de ejecución (`HU-<MICRO>-NNN`) debidamente vinculados a su épica (`EP-<MICRO>-NN`).
2. [ ] **Descripción Clara:** Sigue el formato estándar (*Como... Quiero... Para...*) con beneficio justificado.
3. [ ] **Criterios de Aceptación Gherkin:** Al menos un escenario principal (`AC-001`) y escenarios de validación/error redactados en formato *Given-When-Then*.
4. [ ] **Aislamiento Multi-Tenant Explícito:** Si la historia persiste datos, especifica la columna `tenant_id` y el impacto en las políticas de Row-Level Security (RLS).
5. [ ] **Contrato o Dependencias Resueltas:** Si expone APIs o eventos, el contrato OpenAPI (`contracts/api/`) o esquema CloudEvents (`contracts/events/`) está redactado o no tiene bloqueantes técnicos.
6. [ ] **Estimación y Alcance Acordado:** El equipo comprende los límites de la historia y no viola las fronteras del sprint o de la arquitectura.
