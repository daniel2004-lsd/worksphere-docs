# 00 — Gobernanza Técnica, Git Conventions y Calidad

---

## 🌿 Convenciones de Ramas (Git Branching)

Se utiliza una adaptación de **GitHub Flow** orientada a microservicios:

- `main`: Código probado y estable listo para despliegue.
- `feat/<servicio>/<descripcion>`: Nuevas características (ej: `feat/tenant-service/create-org`).
- `fix/<servicio>/<descripcion>`: Corrección de fallos.
- `docs/<descripcion>`: Cambios en documentación de diseño o SDD.
- `chore/<infra>/<descripcion>`: Tareas de infraestructura local, CI/CD o Docker.

---

## 📝 Formato de Commits (Conventional Commits)

```
<tipo>(<alcance>): <descripción corta>

[cuerpo opcional explicando el porqué de la decisión]

[referencia a ticket o requerimiento]
```

**Ejemplos:**
- `feat(tenant): implement outbox relay for OrganizationCreated event`
- `fix(workspace): enforce tenant_id check on task deletion`
- `docs(arch): add ADR-002 multi-tenancy row level security`

---

## 🚦 Definition of Done (DoD)

Para que una historia o tarea se considere **completada**:
1. [ ] El código implementa exactamente el contrato definido en la documentación.
2. [ ] Pasa las pruebas unitarias y de integración del servicio.
3. [ ] Incluye prueba explícita de aislamiento multi-tenant (*tenant isolation check*).
4. [ ] Contrato de API o esquema de eventos actualizado en el repo de docs si hubo modificaciones.
5. [ ] El servicio levanta y se comunica sin errores en el entorno `docker-compose`.
