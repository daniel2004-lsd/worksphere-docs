# 00 — Definition of Done (DoD)

> **Propósito:** Ninguna Historia de Usuario puede pasar al estado **Hecho (Done)** ni cerrarse en el tracker sin cumplir con cada uno de los siguientes puntos:

1. [ ] **Implementación Fiel al Contrato:** El código cumple de forma exacta con la especificación y los criterios de aceptación (`AC-*`).
2. [ ] **Pruebas Automatizadas Verificadas:**
   - Pruebas unitarias de lógica de negocio ejecutadas y en verde (`TC-*`).
   - Prueba de aislamiento multi-tenant (*tenant leakage test*) ejecutada contra PostgreSQL real.
3. [ ] **Migraciones con Liquibase:**
   - Todo cambio de esquema cuenta con su changeSet fechado (`YYYYMMDD-HHMM-*`), precondiciones, bloque `<rollback>` probado y RLS activo.
4. [ ] **Cero Violaciones de Frontera:** Ningún servicio consulta directamente tablas de otra base de datos ni evade el patrón Transactional Outbox para eventos.
5. [ ] **Documentación Sincronizada:** La especificación y los contratos de API/eventos en `worksphere-docs` reflejan el comportamiento real del código entregado.
6. [ ] **Code Review Aprobado:** Pull Request revisado y sin advertencias críticas de seguridad o rendimiento.
