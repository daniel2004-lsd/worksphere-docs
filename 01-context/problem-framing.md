# 01 — Planteamiento del Problema (Problem Framing)

> **Documento:** Problem Statement & Justification  
> **Estado:** Fase 0 — Definición de Negocio  
> **Líder Técnico & Arquitecto:** Daniel (`daniel2004-lsd`)  
> **Proyecto:** WorkSphere — Enterprise Operations Intelligence Platform

---

## 1. El Contexto Operativo Actual

En las organizaciones intensivas en conocimiento y servicios profesionales (fábricas de software, consultoras tecnológicas y agencias de ingeniería), la operación diaria descansa sobre herramientas altamente especializadas pero totalmente aisladas:

- **Seguimiento de tareas técnicas:** Jira, Linear, Trello, GitHub Issues.
- **Comunicación y coordinación inmediata:** Slack, Microsoft Teams, correo electrónico, reuniones virtuales.
- **Gestión de personas y estructura:** Sistemas de RRHH, hojas de cálculo, organigramas estáticos.
- **Repositorios y código:** GitHub, GitLab, Bitbucket.

Cada una de estas herramientas cumple su función puntual de forma adecuada; sin embargo, **ninguna de ellas comprende la operación de la empresa como un sistema integral**.

---

## 2. El Dolor Central: Ceguera de Contexto (Context Blindness)

El problema de raíz no es la falta de herramientas, sino la **fragmentación y pérdida de causalidad operativa**.

```
[Gestores de Tareas]          [Canales de Chat / Email]         [Estructura Organizacional]
         │                                │                                  │
  "Ticket en progreso"            "Aprobación informal"            "Roles y asignaciones"
         │                                │                                  │
         └────────────────────────────────┼──────────────────────────────────┘
                                          ▼
                         FRAGMENTACIÓN Y PÉRDIDA DE CAUSALIDAD
         - No se sabe si una tarea está detenida por código o por una aprobación.
         - Se desconoce la sobrecarga real de las personas que aprueban.
         - Las decisiones se toman tarde cuando el proyecto ya está retrasado.
```

### Síntomas Específicos Identificados:

1. **Bloqueos Invisibles por Aprobaciones Informales:**  
   Gran parte de los retrasos en proyectos de software no se deben a dificultades técnicas de programación, sino a **aprobaciones administrativas o de negocio estancadas**: autorizaciones de presupuesto, validaciones de arquitectura, accesos a entornos de clientes o visto bueno de cambios de alcance. Al gestionarse por chat o correo, no existe trazabilidad, SLA ni visibilidad de quién tiene el balón detenido.

2. **Desbalance de Carga y Sobreasignación No Detectada:**  
   Un líder técnico o gerente puede tener asignadas tareas en un gestor tradicional que aparentan 20 horas de trabajo semanal. No obstante, esa misma persona recibe decenas de solicitudes de revisión y aprobación en paralelo. Las herramientas actuales no correlacionan la carga de procesos con la carga de tareas de desarrollo, generando cuellos de botella y *burnout*.

3. **Ceguera Ejecutiva en Tiempo Real:**  
   Para que la dirección de operaciones (COO / VP) conozca la salud real de los proyectos, debe convocar reuniones síncronas de estatus (*status meetings*) o exigir reportes manuales a los directores de proyecto, los cuales reflejan información desfasada o filtrada subjetivamente.

4. **Pérdida de Soberanía y Dependencia de Proveedores (*Vendor Lock-in*):**  
   Las soluciones de gestión empresarial tradicionales imponen infraestructuras de nube cerrada, licencias privativas por usuario con costos crecientes y almacenamiento de datos sensibles (propiedad intelectual, clientes, nóminas) fuera del control y la red de la organización.

---

## 3. La Justificación de WorkSphere: Capa de Inteligencia e Integración

WorkSphere **no busca reemplazar de forma destructiva las herramientas existentes**. Su propósito es constituirse como la **Capa de Inteligencia Operativa y Gobernanza (System of Intelligence)** que:

1. **Formaliza y relaciona los Procesos de Aprobación** directamente vinculados a proyectos y tareas.
2. **Integra la Estructura Organizacional** para saber quién tiene autoridad para aprobar, qué equipo está impactado y cómo se distribuye la capacidad operativa.
3. **Ofrece Soberanía Total:** Arquitectura *Open Source First* y *Self-Hosted First*, ejecutable en infraestructura propia sin fugas de datos hacia terceros.
4. **Inteligencia Artificial Local y Verificable:** Asistencia basada exclusivamente en hechos auditables y autorizados mediante permisos de usuario (Role-Based Access Control + Row-Level Security), eliminando alucinaciones y garantizando la privacidad corporativa.
