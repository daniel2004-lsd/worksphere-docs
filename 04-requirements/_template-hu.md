# Plantilla Oficial de Historia de Usuario

> **ID de Planeación:** `HU-NN`  
> **ID de Ejecución:** `HU-<MICRO>-NNN`  
> **Épica de Producto:** `[EPC-NN: Título]`  
> **Épica de Ejecución:** `[EP-<MICRO>-NN: Título]`  
> **Feature:** `[FEA-NNN: Título]`  
> **Prioridad / Severidad:** `[P0 | P1 | P2 | P3]`  
> **Estatus:** `[Backlog | Refinamiento | Listo | En desarrollo | En revisión | En QA | Hecho]`

---

## 📖 Descripción de la Historia

**Como** [tipo de usuario o arquetipo empresarial],  
**Quiero** [acción o capacidad que se desea ejecutar en el sistema],  
**Para** [beneficio o valor de negocio esperado].

---

## 🎯 Contexto y Justificación
[Breve explicación del problema operativo que resuelve y por qué es necesaria esta funcionalidad en el microservicio correspondiente.]

---

## 🛡️ Reglas de Negocio & Restricciones Técnicas
- **RN-01:** [Regla de negocio, ej. El slug corporativo debe ser único y normalizado].
- **RN-02 (Multi-tenancy):** [Obligatoriedad de aislar los datos mediante `tenant_id` y RLS].
- **RN-03 (Consistencia):** [Si genera efectos secundarios, debe usar Outbox o eventos CloudEvents].

---

## 🧪 Criterios de Aceptación (Gherkin)

### AC-001: [Escenario Principal / Happy Path]
- **Given** [un estado inicial válido del sistema o usuario autenticado],
- **When** [el usuario ejecuta la acción o envía la petición a la API],
- **Then** [el sistema responde con el resultado esperado y confirma el cambio de estado].

### AC-002: [Escenario de Validación / Error Controlado]
- **Given** [un estado o payload inválido / duplicado],
- **When** [se intenta realizar la operación],
- **Then** [el sistema rechaza con código HTTP apropiado y mensaje claro sin exponer detalles internos].

---

## 🔬 Trazabilidad de Pruebas
- **TC-001 (Unit Test):** `[Ruta del test unitario]`
- **TC-002 (Integration / Leakage Test):** `[Ruta del test de integración con base de datos real / RLS]`
