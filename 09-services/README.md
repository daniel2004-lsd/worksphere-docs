# Catálogo de Microservicios de WorkSphere

WorkSphere descompone su dominio en **8 microservicios independientes** y componentes transversales de infraestructura.

---

## 📦 Inventario de Microservicios

| Directorio | Microservicio | Tecnología | Base de Datos | Responsabilidad |
|---|---|---|---|---|
| [`identity-access/`](./identity-access/README.md) | **Identity & Access** | Keycloak + Java | PostgreSQL | Autenticación, JWT, roles RBAC y claims multi-tenant |
| [`tenant-management/`](./tenant-management/README.md) | **Tenant Management** | Java (Spring Boot 3) | PostgreSQL | Ciclo de vida de empresas, membresías e invitaciones |
| [`workspace/`](./workspace/README.md) | **Workspace Service** | C# (.NET 9) | PostgreSQL | Gestión de proyectos, listas de tareas y tableros |
| [`workflow/`](./workflow/README.md) | **Workflow Service** | Java (Spring Boot 3) | PostgreSQL | Procesos de aprobación, reglas y cambios de estado |
| [`billing/`](./billing/README.md) | **Billing Service** | C# (.NET 9) | PostgreSQL | Planes SaaS, suscripciones y control de cuotas |
| [`notifications/`](./notifications/README.md) | **Notification Service** | Go 1.22 | Redis / Postgres | Envío de correos, notificaciones push e in-app |
| [`file-management/`](./file-management/README.md) | **File Management** | Go 1.22 | MinIO (S3) | Almacenamiento seguro y pre-signed URLs de adjuntos |
| [`analytics/`](./analytics/README.md) | **Analytics Service** | Python (FastAPI) | ClickHouse | Métricas de uso de inquilinos y reportes BI |

---

## 🧱 Regla de Comunicación

- **Síncrona (REST vía Kong Gateway):** Exclusiva para peticiones de lectura inmediatas o comandos directos del usuario frontend.
- **Asíncrona (RabbitMQ):** Para propagación de efectos secundarios entre servicios (ej: alta de organización -> aprovisionamiento en billing -> envío de email).
