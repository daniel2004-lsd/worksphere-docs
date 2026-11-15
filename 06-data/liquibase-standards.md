# 06 — Estándares y Convenciones Oficiales de Liquibase

> **Documento:** Database Versioning & Liquibase Governance  
> **Estado:** Decisión Aprobada (Mandatoria)  
> **Ámbito:** Todos los repositorios de bases de datos de WorkSphere (`worksphere-*-db`)  
> **Herramienta Oficial Única:** Liquibase (YAML Master + SQL Modular)  
> **Patrón de Referencia:** Arquitectura de Gobernanza `EcoRuteandoDb`

---

## 1. Principios Rectores

1. **Un Repositorio Exclusivo por Base de Datos (`worksphere-*-db`):**  
   Cada microservicio relacional (`tenant-service`, `workspace-service`, `workflow-service`, etc.) tiene como par obligatorio un repositorio independiente de persistencia que alberga sus migraciones declarativas y políticas de seguridad.
2. **Estructura Modular por Lenguaje SQL (DDL / DML / DCL / TCL / Rollbacks):**  
   Se adopta la arquitectura probada en `EcoRuteandoDb`, dividiendo las mutaciones de base de datos según el estándar formal SQL en carpetas numeradas:
   - `01_ddl/` (Data Definition Language: extensiones, tipos, tablas, restricciones, vistas, funciones, triggers, índices).
   - `02_dml/` (Data Manipulation Language: roles base, datos semilla, inserciones controladas, parches).
   - `03_dcl/` (Data Control Language: permisos de usuarios `GRANT/REVOKE`, políticas de **Row-Level Security**).
   - `04_tcl/` (Transaction Control Language: transacciones explícitas si aplica).
   - `05_rollbacks/` (Estructura espejo de scripts SQL para reversión controlada).
3. **Formato Híbrido YAML Master + Archivos `.sql` Nativos:**  
   - Los changelogs maestros y de módulo se definen en **YAML** (`changelog.yaml`), proporcionando metadatos estructurados (`id`, `author`, `labels`, `comment`).
   - Los cambios DDL y DML se escriben en ficheros `.sql` puros referenciados mediante `sqlFile`, garantizando compatibilidad total con herramientas de PostgreSQL, linters SQL y sintaxis nativa.
4. **Rollback Espejo Obligatorio:**  
   Todo changeSet que aplique un `sqlFile` debe referenciar de forma mandatoria su archivo homólogo de reversión en `05_rollbacks/.../<nombre>.rollback.sql`.
5. **Estrategia Expand / Contract:**  
   Prohibido realizar cambios destructivos (drop de columnas/tablas) en el mismo release en que se introducen nuevas entidades.

---

## 2. Topología Canónica de Carpetas para Repositorios `worksphere-*-db`

Cada repositorio de base de datos (`worksphere-tenant-db`, `worksphere-workspace-db`, `worksphere-workflow-db`) se estructurará exactamente así:

```
worksphere-<micro>-db/
├── changelog/
│   └── db.changelog-master.yaml       # Punto de entrada raíz de Liquibase
│
├── 01_ddl/                             # Mutaciones de Estructura (DDL)
│   ├── 00_extensions/                  # pgcrypto, uuid-ossp, etc.
│   │   └── changelog.yaml
│   ├── 01_schemas/                     # Definición de esquemas de módulo si aplica
│   │   └── changelog.yaml
│   ├── 02_types/                       # ENUMs y tipos personalizados
│   │   └── changelog.yaml
│   ├── 03_tables/                      # Definición de tablas organizadas por dominio
│   │   ├── 001_<dominio>_module/
│   │   │   ├── 001_create_table_foo.sql
│   │   │   └── changelog.yaml
│   │   └── changelog.yaml
│   ├── 04_constraints/                 # Claves foráneas internas y checks complejos
│   │   └── changelog.yaml
│   ├── 08_functions/                   # Funciones almacenadas / triggers
│   │   └── changelog.yaml
│   ├── 09_triggers/                    # Triggers de auditoría o timestamp
│   │   └── changelog.yaml
│   ├── 10_indexes/                     # Índices de rendimiento y búsqueda
│   │   └── changelog.yaml
│   └── changelog.yaml                  # Orquestador general DDL
│
├── 02_dml/                             # Carga y Semillas de Datos (DML)
│   ├── 00_roles/                       # Roles base del sistema
│   ├── 01_inserts/                     # Semillas maestras (planes, catálogos fijos)
│   ├── 05_patches/                     # Migraciones de datos entre versiones
│   └── changelog.yaml                  # Orquestador general DML
│
├── 03_dcl/                             # Seguridad y Control de Acceso (DCL)
│   ├── 01_grants/                      # Asignación de privilegios DML al app_user
│   ├── 02_policies/                    # Políticas de PostgreSQL Row-Level Security (RLS)
│   │   ├── 001_tenant_isolation_policies.sql
│   │   └── changelog.yaml
│   └── changelog.yaml                  # Orquestador general DCL
│
├── 05_rollbacks/                       # Scripts Espejo de Reversión
│   ├── 01_ddl/
│   │   └── 03_tables/
│   │       └── 001_<dominio>_module/
│   │           └── 001_create_table_foo.rollback.sql
│   └── 03_dcl/
│       └── 02_policies/
│           └── 001_tenant_isolation_policies.rollback.sql
│
├── scripts/                            # Herramientas de automatización local
│   ├── run-local-update.sh             # Ejecuta liquibase update contra PostgreSQL local
│   └── run-local-rollback.sh           # Prueba la reversión controlada
│
├── docker-compose.yml                  # Entorno de base de datos aislado para pruebas
├── liquibase.properties.example        # Plantilla de conexión JDBC
├── README.md                           # Documentación técnica del esquema
└── LICENSE                             # Apache 2.0
```

---

## 3. Orquestador Raíz (`changelog/db.changelog-master.yaml`)

El archivo maestro enlaza de forma secuencial los cuatro grandes bloques del ciclo de vida de la base de datos:

```yaml
databaseChangeLog:
  - include:
      file: ../01_ddl/changelog.yaml
      relativeToChangelogFile: true
  - include:
      file: ../02_dml/changelog.yaml
      relativeToChangelogFile: true
  - include:
      file: ../03_dcl/changelog.yaml
      relativeToChangelogFile: true
```

---

## 4. Estándar de ChangeSets con `sqlFile` y Rollback Espejo

Cada mutación se define en su subcarpeta correspondiente con metadatos de trazabilidad:

```yaml
databaseChangeLog:
  - changeSet:
      id: 20261115-001-create-table-organizations
      author: daniel2004-lsd
      labels: "ddl,tables,tenant"
      comment: "Crea la tabla principal de organizaciones para tenant-management"
      changes:
        - sqlFile:
            path: 001_create_table_organizations.sql
            relativeToChangelogFile: true
            stripComments: false
      rollback:
        - sqlFile:
            path: ../../../05_rollbacks/01_ddl/03_tables/001_tenant_module/001_create_table_organizations.rollback.sql
            relativeToChangelogFile: true
            stripComments: false
```

### 4.1. Archivo SQL de Creación (`001_create_table_organizations.sql`):
```sql
CREATE TABLE IF NOT EXISTS organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id VARCHAR(64) NOT NULL UNIQUE,
    name VARCHAR(120) NOT NULL,
    slug VARCHAR(64) NOT NULL UNIQUE,
    admin_email VARCHAR(120) NOT NULL,
    status VARCHAR(30) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

### 4.2. Archivo SQL de Reversión (`001_create_table_organizations.rollback.sql`):
```sql
DROP TABLE IF EXISTS organizations CASCADE;
```

---

## 5. Implementación de Políticas RLS en la Capa `03_dcl`

Siguiendo el estándar, las políticas de aislamiento multi-tenant residen de forma limpia en `03_dcl/02_policies/`:

### `03_dcl/02_policies/001_organizations_rls.sql`:
```sql
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS tenant_isolation_policy ON organizations;
CREATE POLICY tenant_isolation_policy ON organizations
    FOR ALL
    TO PUBLIC
    USING (
        tenant_id = current_setting('app.current_tenant_id', true)
        OR current_setting('app.current_tenant_id', true) = 'SYSTEM'
    );
```

### `05_rollbacks/03_dcl/02_policies/001_organizations_rls.rollback.sql`:
```sql
DROP POLICY IF EXISTS tenant_isolation_policy ON organizations;
ALTER TABLE organizations DISABLE ROW LEVEL SECURITY;
```

---

## 6. Procedimiento de Validación en CI/CD

Antes de aceptar un Pull Request en cualquier repositorio `worksphere-*-db`:
1. **`liquibase validate`:** Comprobación estricta de sintaxis YAML y rutas relativas a ficheros SQL.
2. **Despliegue Limpio (Fresh Test):** Aplicación de todo el `db.changelog-master.yaml` en un PostgreSQL efímero.
3. **Prueba de Rollback Idempotente:** Ejecución de rollback completo y re-aplicación para garantizar reversibilidad.
4. **Verificación de Políticas DCL/RLS:** Intento de consulta sin contexto de tenant para validar rechazo automático.
