---
description: "Genera una migración Flyway (script SQL) y actualiza el modelo Kotlin para cambios en el esquema de BD: nuevas tablas, columnas, índices o constraints. Usar cuando se modifica una entidad JPA."
name: "Migración Flyway"
argument-hint: "Describe el cambio en la BD: qué tabla/columna se crea, modifica o elimina, y por qué"
agent: "agent"
---

# Migración Flyway — NN Auth System (Kotlin Edition)

Genera la migración Flyway y actualiza el modelo Kotlin para el cambio de esquema indicado.

## Convenciones obligatorias

- **NUNCA** modificar la base de datos manualmente — siempre vía Flyway
- **NUNCA** editar un script de migración ya ejecutado — crear uno nuevo con versión mayor
- Nombres de tablas: `snake_case`, plural (`users`, `password_reset_tokens`)
- Nombres de columnas: `snake_case` (`created_at`, `hashed_password`)
- Toda tabla debe tener `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`
- Tablas que se actualizan deben tener `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`
- Primary keys: `UUID DEFAULT gen_random_uuid()`
- **Gradle Wrapper**: siempre `./gradlew`, nunca `gradle` directamente

## Archivos de referencia

- Modelos JPA: [be/src/main/kotlin/com/nn/auth/model/](../../../be/src/main/kotlin/com/nn/auth/model/)
- Migraciones existentes: [be/src/main/resources/db/migration/](../../../be/src/main/resources/db/migration/)
- Configuración Flyway: [be/src/main/resources/application.yml](../../../be/src/main/resources/application.yml)

## Lo que debes generar

### 1. Actualizar o crear el modelo Kotlin (`be/src/main/kotlin/com/nn/auth/model/`)

Si el cambio implica una nueva entidad, crear `NombreModelo.kt`.
Si modifica una existente, editar el archivo correspondiente.

Usar `data class` con anotaciones JPA — nunca Lombok (ese es Java):

```kotlin
/**
 * Archivo: NombreModelo.kt
 * Descripción: Entidad JPA que representa la tabla nombre_tabla en la BD.
 * ¿Para qué? Mapear las filas de la BD a objetos Kotlin que Spring Data JPA pueda manipular.
 * ¿Impacto? Sin esta entidad, el repositorio no puede acceder a la tabla.
 */
package com.nn.auth.model

import jakarta.persistence.*
import java.time.Instant
import java.util.UUID

@Entity
@Table(name = "nombre_tabla")
data class NombreModelo(

    /**
     * ¿Qué? Identificador único de la entidad.
     * ¿Para qué? Clave primaria UUID — más seguro que un entero autoincremental.
     * ¿Impacto? Sin UUID, los IDs serían predecibles y facilitan enumeración.
     */
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    val id: UUID = UUID.randomUUID(),

    /**
     * ¿Qué? Campo de ejemplo.
     * ¿Para qué? [describir propósito].
     * ¿Impacto? [describir consecuencias si falla].
     */
    @Column(name = "nombre_columna", nullable = false, length = 255)
    val nombreCampo: String,

    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: Instant = Instant.now()
)
```

Notas importantes para Kotlin + JPA:
- Usar `data class` con propiedades `val` donde sean inmutables
- JPA requiere un constructor sin argumentos — Spring Boot lo genera automáticamente con el plugin `kotlin-jpa`
- Usar `@Column(name = "snake_case")` explícito para control total del nombre de columna
- Evitar `@GeneratedValue(strategy = GenerationType.IDENTITY)` — preferir UUID

### 2. Crear el script Flyway en `be/src/main/resources/db/migration/`

El nombre del archivo debe seguir el patrón: `V{n}__{descripcion_snake_case}.sql`

```sql
-- V4__add_nombre_columna_to_tabla.sql
--
-- ¿Qué? Agrega la columna nombre_columna a la tabla nombre_tabla.
-- ¿Para qué? [describir propósito funcional].
-- ¿Impacto? [describir qué habilita esta migración].

ALTER TABLE nombre_tabla
    ADD COLUMN IF NOT EXISTS nombre_columna VARCHAR(255);

-- Si es NOT NULL y sin default, agregar default temporal:
-- ALTER TABLE nombre_tabla ADD COLUMN IF NOT EXISTS nombre_columna VARCHAR(255) NOT NULL DEFAULT '';
-- UPDATE nombre_tabla SET nombre_columna = '';
-- ALTER TABLE nombre_tabla ALTER COLUMN nombre_columna DROP DEFAULT;
```

Para una tabla nueva:

```sql
-- V4__create_nombre_tabla_table.sql
--
-- ¿Qué? Crea la tabla nombre_tabla con UUID PK y foreign key a users.
-- ¿Para qué? Persistir [entidad] vinculada a usuarios.
-- ¿Impacto? Sin esta tabla, el flujo de [feature] no puede persistir datos.

CREATE TABLE IF NOT EXISTS nombre_tabla (
    id          UUID        NOT NULL DEFAULT gen_random_uuid(),
    user_id     UUID        NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token       VARCHAR(255) NOT NULL,
    expires_at  TIMESTAMPTZ NOT NULL,
    used        BOOLEAN     NOT NULL DEFAULT FALSE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT pk_nombre_tabla PRIMARY KEY (id),
    CONSTRAINT uq_nombre_tabla_token UNIQUE (token)
);

CREATE INDEX IF NOT EXISTS idx_nombre_tabla_token ON nombre_tabla(token);
CREATE INDEX IF NOT EXISTS idx_nombre_tabla_user_id ON nombre_tabla(user_id);
```

### 3. Verificar la migración

```bash
# Flyway ejecuta las migraciones automáticamente al arrancar
cd be && ./gradlew bootRun

# Para ejecutar solo las migraciones (sin arrancar la app):
cd be && ./gradlew flywayMigrate

# Para verificar el estado de las migraciones:
cd be && ./gradlew flywayInfo
```

Cambio requerido: $arg
