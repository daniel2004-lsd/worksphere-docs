# ADR-001: Arquitectura de Microservicios Distribuidos y Multi-Tenancy

- **Estado:** Aceptado
- **Fecha:** 2026-11-15
- **Decisores:** Daniel (Arquitecto de Software & Tech Lead)

---

## 1. Contexto y Problema

WorkSphere es una plataforma SaaS B2B donde distintas organizaciones gestionan equipos, flujos de trabajo, facturación y analítica. Los requerimientos exigen:
- Aislamiento estricto de datos entre empresas.
- Escalado independiente de componentes con cargas dispares (ej. analítica vs. aprobaciones transaccionales).
- Resiliencia: la caída de un servicio secundario (ej. notificaciones) no debe impedir la operación de proyectos.

---

## 2. Decisión

Se decide implementar una **arquitectura de microservicios con Database-per-Service** y **comunicación asíncrona orientada a eventos**:

1. **Aislamiento Multi-Tenancy:** Particionamiento lógico con columna discriminadora `tenant_id` y aplicación de **PostgreSQL Row-Level Security (RLS)** como defensa en profundidad.
2. **Event-Driven:** Publicación confiable mediante el patrón **Transactional Outbox** y RabbitMQ como event broker.
3. **Distribución Políglota Justificada:**
   - **Java / Spring Boot:** Dominios empresariales ricos (Tenant Management, Workflow).
   - **C# / .NET:** Servicios con reglas transaccionales estructuradas (Workspace, Billing).
   - **Go:** Microservicios I/O bound y concurrentes (Files, Notifications).
   - **Python / FastAPI:** Procesamiento analítico y consultas a ClickHouse (Analytics).
   - **Keycloak:** Proveedor de identidad y emisión centralizada de tokens OIDC/OAuth2.

---

## 3. Consecuencias

### Positivas:
- Aislamiento total del fallo: si un servicio colapsa, los demás continúan operando.
- Flexibilidad para escalar servicios según su demanda de CPU/Memoria.
- Seguridad reforzada con RLS en cada base de datos independiente.

### Desafíos & Mitigaciones:
- **Complejidad distribuida:** Se mitiga con trazabilidad distribuida (OpenTelemetry) y estándares SDD estrictos.
- **Consistencia eventual:** Se asume el compromiso; operaciones críticas de negocio se coordinan mediante patrones Saga y reintentos idempotentes.
