# 06 — Estándares y Convenciones Oficiales de Liquibase

> **Documento:** Database Versioning & Liquibase Governance  
> **Estado:** Decisión Aprobada (Mandatoria)  
> **Ámbito:** Todos los microservicios relacionales de WorkSphere (PostgreSQL)  
> **Herramienta Oficial Única:** Liquibase (Flyway queda formalmente descartado)

---

## 1. Principios Rectores

1. **Database-per-Service con Changelogs Independientes:** Cada microservicio (`tenant-service`, `workspace-service`, `workflow-service`, etc.) es dueño exclusivo de su catálogo y carpeta de changelogs.
2. **Formato Estándar XML / YAML:** Se prioriza XML o YAML para changelogs debido a la portabilidad entre motores, soporte nativo de precondiciones y capacidad de validación estricta de esquemas XSD.
3. **Atomicidad por ChangeSet:** Cada `changeSet` realiza exactamente una mutación lógica (creación de una tabla, adición de columnas, creación de índices o habilitación de políticas RLS).
4. **Rollback Obligatorio y Verificado:** Todo changeSet que altere el esquema debe incluir su bloque `<rollback>`. Ningún PR de base de datos se aprueba si su rollback no ha sido probado y documentado.
5. **Estrategia Expand / Contract:** Nunca se borra ni se renombra una columna en un solo paso destructivo; se introduce la columna nueva (*Expand*), se sincroniza el código, y en la versión siguiente se retira la antigua (*Contract*).

---

## 2. Estructura de Directorios por Servicio

En cada microservicio que interactúe con base de datos:

```
<service-name>/src/main/resources/db/changelog/
├── db.changelog-master.xml           # Archivo maestro que incluye los releases en orden
├── changes/                          # ChangeSets individuales fechados
│   ├── 20261115-1000-init-schema.xml
│   ├── 20261115-1030-enable-rls.xml
│   └── 20261115-1100-add-outbox-table.xml
└── post-migrate/                     # Vistas, procedimientos o seeds controlados
    └── 001-dev-seeds.xml
```

---

## 3. Convención de Nomenclatura e Identificación

- **ID del ChangeSet:** Formato numérico temporal `YYYYMMDD-HHMM-<descripcion-corta>`  
  Ejemplo: `id="20261115-1000-create-organizations-table"`
- **Author:** Identificador de GitHub del autor o rol (ej. `author="daniel2004-lsd"` o `author="worksphere-core"`).

---

## 4. Plantilla de ChangeSet con Precondiciones y RLS

Ejemplo canónico para una tabla de negocio multi-tenant con Row-Level Security:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:pro="http://www.liquibase.org/xml/ns/pro"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">

    <changeSet id="20261115-1000-create-organizations-table" author="daniel2004-lsd">
        <!-- Precondición: La tabla no debe existir previamente -->
        <preConditions onFail="MARK_RAN">
            <not>
                <tableExists tableName="organizations"/>
            </not>
        </preConditions>

        <createTable tableName="organizations">
            <column name="id" type="UUID" defaultValueComputed="gen_random_uuid()">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="tenant_id" type="VARCHAR(64)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="name" type="VARCHAR(120)">
                <constraints nullable="false"/>
            </column>
            <column name="slug" type="VARCHAR(64)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="admin_email" type="VARCHAR(120)">
                <constraints nullable="false"/>
            </column>
            <column name="status" type="VARCHAR(30)" defaultValue="ACTIVE">
                <constraints nullable="false"/>
            </column>
            <column name="created_at" type="TIMESTAMP WITH TIME ZONE" defaultValueComputed="NOW()">
                <constraints nullable="false"/>
            </column>
            <column name="updated_at" type="TIMESTAMP WITH TIME ZONE" defaultValueComputed="NOW()">
                <constraints nullable="false"/>
            </column>
        </createTable>

        <rollback>
            <dropTable tableName="organizations"/>
        </rollback>
    </changeSet>

    <changeSet id="20261115-1015-enable-rls-organizations" author="daniel2004-lsd">
        <comment>Activar Row-Level Security y política de aislamiento multi-tenant</comment>
        <sql>
            ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
            CREATE POLICY tenant_isolation_policy ON organizations
                FOR ALL
                TO PUBLIC
                USING (
                    tenant_id = current_setting('app.current_tenant_id', true)
                    OR current_setting('app.current_tenant_id', true) = 'SYSTEM'
                );
        </sql>
        <rollback>
            <sql>
                DROP POLICY IF EXISTS tenant_isolation_policy ON organizations;
                ALTER TABLE organizations DISABLE ROW LEVEL SECURITY;
            </sql>
        </rollback>
    </changeSet>

</databaseChangeLog>
```

---

## 5. Integración con CI/CD y Verificación

En el pipeline de integración continua de cada servicio:
1. `liquibase validate`: Comprueba la sintaxis de los XML y que no existan IDs duplicados.
2. `liquibase updateTestingRollback`: Aplica los cambios en una base de datos efímera de test, ejecuta el rollback completo para verificar que es funcional y vuelve a aplicar el update para dejar la BD lista.
