<!--
  ¿Qué? Documentación del esquema de base de datos del sistema NN Auth.
  ¿Para qué? Servir como referencia técnica para entender la estructura de datos,
             relaciones entre tablas, y cómo Flyway gestiona las migraciones.
  ¿Impacto? Sin esta documentación, sería difícil entender por qué el sistema almacena
             ciertos datos y cómo se relacionan entre sí.
-->

# Esquema de Base de Datos — NN Auth System (Kotlin Edition)

> **Motor**: PostgreSQL 17
> **Herramienta de migraciones**: [Flyway](https://flywaydb.org/) — scripts SQL versionados
>
> **Diferencia con `proyecto-be-fe`**: El proyecto de referencia usa **Alembic** (Python) con
> scripts de migración en Python. Este proyecto usa **Flyway** (JVM) con scripts SQL puro.
> El **esquema resultante es idéntico** — la diferencia está solo en la herramienta.

---

## Diagrama Entidad-Relación (ER)

```
┌─────────────────────────────────────────┐
│                  users                  │
├─────────────────────────────────────────┤
│ id               UUID          PK       │
│ email            VARCHAR(255)  UNIQUE   │
│ full_name        VARCHAR(255)  NOT NULL │
│ hashed_password  VARCHAR(255)  NOT NULL │
│ is_active        BOOLEAN       DEFAULT  │
│ is_email_verified BOOLEAN      DEFAULT  │
│ created_at       TIMESTAMPTZ   NOT NULL │
│ updated_at       TIMESTAMPTZ   NOT NULL │
└──────────────┬──────────────────────────┘
               │ 1
               │
       ┌───────┴───────┐
       │               │
       │ *             │ *
┌──────┴───────────────┴──────────────────────────────┐    ┌─────────────────────────────────────────────────┐
│         password_reset_tokens                       │    │         email_verification_tokens               │
├─────────────────────────────────────────────────────┤    ├─────────────────────────────────────────────────┤
│ id          UUID          PK                        │    │ id          UUID          PK                    │
│ user_id     UUID          FK → users.id (CASCADE)   │    │ user_id     UUID          FK → users.id (CASCADE)│
│ token       VARCHAR(255)  UNIQUE, NOT NULL, INDEXED │    │ token       VARCHAR(255)  UNIQUE, NOT NULL       │
│ expires_at  TIMESTAMPTZ   NOT NULL                  │    │ expires_at  TIMESTAMPTZ   NOT NULL               │
│ used        BOOLEAN       DEFAULT FALSE             │    │ used        BOOLEAN       DEFAULT FALSE          │
│ created_at  TIMESTAMPTZ   DEFAULT NOW()             │    │ created_at  TIMESTAMPTZ   DEFAULT NOW()          │
└─────────────────────────────────────────────────────┘    └─────────────────────────────────────────────────┘
```

---

## Tabla `users`

**Descripción**: Almacena todos los usuarios registrados del sistema.

**Script Flyway**: `V1__create_users_table.sql`

| Columna | Tipo PostgreSQL | Restricciones | Descripción |
|---|---|---|---|
| `id` | `UUID` | PK, `DEFAULT gen_random_uuid()` | Identificador único del usuario |
| `email` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL`, indexado | Email para login y notificaciones |
| `full_name` | `VARCHAR(255)` | `NOT NULL` | Nombre completo del usuario |
| `hashed_password` | `VARCHAR(255)` | `NOT NULL` | Contraseña hasheada con BCrypt |
| `is_active` | `BOOLEAN` | `DEFAULT TRUE`, `NOT NULL` | Si la cuenta está activa |
| `is_email_verified` | `BOOLEAN` | `DEFAULT FALSE`, `NOT NULL` | Si el email fue verificado |
| `locale` | `VARCHAR(10)` | `DEFAULT 'es'`, `NOT NULL` | Idioma preferido del usuario (`es` / `en`) |
| `created_at` | `TIMESTAMPTZ` | `DEFAULT NOW()`, `NOT NULL` | Fecha y hora de registro (UTC) |
| `updated_at` | `TIMESTAMPTZ` | `DEFAULT NOW()`, `NOT NULL` | Fecha y hora de último cambio (UTC) |

### Notas de diseño

- **`id` como UUID**: Se usa `gen_random_uuid()` (función nativa de PostgreSQL 13+) para generar
  identificadores únicos sin necesidad de secuencias. Ventaja: los IDs no son predecibles ni secuenciales,
  lo que dificulta la enumeración de recursos (OWASP A01).

- **`hashed_password`**: Nunca texto plano. `BCryptPasswordEncoder` de Spring Security genera hashes
  en formato `$2a$10$...` de 60 caracteres. `VARCHAR(255)` es amplio por si el factor de costo (rounds) aumenta.

- **`is_email_verified = false` por defecto**: Un usuario recién registrado no puede hacer login hasta
  verificar su email. Esto previene bots y garantiza que el email sea válido.

- **`locale = 'es'` por defecto**: Idioma de la interfaz preferido por el usuario. Solo acepta `'es'`
  (español) o `'en'` (inglés). Se actualiza vía `PATCH /api/v1/users/me/locale` cuando el usuario
  cambia el idioma en el frontend. Ver `RF-014` y `HU-012`.

- **`TIMESTAMPTZ`**: Almacena timestamps con zona horaria (UTC). Importante para sistemas que puedan
  tener usuarios en múltiples zonas horarias.

### Entidad JPA (`User.kt`)

```kotlin
// En Kotlin + Spring Data JPA, esta tabla se mapea con:
@Entity
@Table(name = "users")
data class User(
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    val id: UUID? = null,

    @Column(nullable = false, unique = true)
    val email: String,

    @Column(name = "full_name", nullable = false)
    val fullName: String,

    @Column(name = "hashed_password", nullable = false)
    var hashedPassword: String,

    @Column(name = "is_active", nullable = false)
    var isActive: Boolean = true,

    @Column(name = "is_email_verified", nullable = false)
    var isEmailVerified: Boolean = false,

    @Column(name = "locale", nullable = false, length = 10)
    var locale: String = "es",

    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: OffsetDateTime = OffsetDateTime.now(),

    @Column(name = "updated_at", nullable = false)
    var updatedAt: OffsetDateTime = OffsetDateTime.now()
)
```

> **Comparativa con `proyecto-be-fe`**: En Python, el modelo se define con SQLAlchemy:
> `class User(Base): __tablename__ = "users"`. En Kotlin, se usa `@Entity` de JPA/Hibernate.
> El resultado en la base de datos es **idéntico** (incluyendo la columna `locale`).

---

## Tabla `password_reset_tokens`

**Descripción**: Almacena tokens temporales para el flujo de recuperación de contraseña.
Cada token tiene una vigencia de **1 hora**.

**Script Flyway**: `V2__create_password_reset_tokens_table.sql`

| Columna | Tipo PostgreSQL | Restricciones | Descripción |
|---|---|---|---|
| `id` | `UUID` | PK, `DEFAULT gen_random_uuid()` | Identificador único del token |
| `user_id` | `UUID` | FK → `users.id`, `CASCADE DELETE` | Usuario dueño del token |
| `token` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL`, indexado | Token UUID enviado por email |
| `expires_at` | `TIMESTAMPTZ` | `NOT NULL` | Cuándo expira (creado_en + 1 hora) |
| `used` | `BOOLEAN` | `DEFAULT FALSE`, `NOT NULL` | Si el token ya fue utilizado |
| `created_at` | `TIMESTAMPTZ` | `DEFAULT NOW()`, `NOT NULL` | Fecha de creación |

### Ciclo de vida del token

```
1. Usuario solicita POST /auth/forgot-password
2. Sistema crea registro con:
   - token = UUID aleatorio
   - expires_at = NOW() + 1 hora
   - used = false
3. Se envía email con URL: {FRONTEND_URL}/reset-password?token=<token>
4. Usuario hace clic → POST /auth/reset-password { token, new_password }
5. Sistema verifica:
   - token existe en BD → si no, 400
   - expires_at > NOW() → si no, 400 ("token expirado")
   - used = false → si no, 400 ("token ya utilizado")
6. Sistema:
   - Actualiza contraseña del usuario (hasheada)
   - Marca token como used = true
```

### Notas de diseño

- **`CASCADE DELETE`**: Si un usuario es eliminado, todos sus tokens de reset se eliminan automáticamente.
  Esto evita datos huérfanos en la base de datos.

- **`used = true`**: Un token solo puede usarse una vez. Esto previene que alguien que interceptó
  el email pueda reutilizarlo.

- **Expiración de 1 hora**: Balance entre usabilidad y seguridad. Un enlace de reset válido por
  mucho tiempo es un riesgo de seguridad (OWASP A07).

---

## Tabla `email_verification_tokens`

**Descripción**: Almacena tokens temporales para verificar la dirección de email de nuevos usuarios.
Cada token tiene una vigencia de **24 horas**.

**Script Flyway**: `V3__create_email_verification_tokens_table.sql`

| Columna | Tipo PostgreSQL | Restricciones | Descripción |
|---|---|---|---|
| `id` | `UUID` | PK, `DEFAULT gen_random_uuid()` | Identificador único del token |
| `user_id` | `UUID` | FK → `users.id`, `CASCADE DELETE` | Usuario dueño del token |
| `token` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL`, indexado | Token UUID enviado por email |
| `expires_at` | `TIMESTAMPTZ` | `NOT NULL` | Cuándo expira (creado_en + 24 horas) |
| `used` | `BOOLEAN` | `DEFAULT FALSE`, `NOT NULL` | Si el token ya fue utilizado |
| `created_at` | `TIMESTAMPTZ` | `DEFAULT NOW()`, `NOT NULL` | Fecha de creación |

### Ciclo de vida del token

```
1. Usuario se registra POST /auth/register
2. Sistema crea usuario (is_email_verified = false)
3. Sistema crea token de verificación:
   - token = UUID aleatorio
   - expires_at = NOW() + 24 horas
   - used = false
4. Se envía email con URL: {FRONTEND_URL}/verify-email?token=<token>
5. Usuario hace clic → POST /auth/verify-email { token }
6. Sistema verifica:
   - token existe → si no, 400
   - expires_at > NOW() → si no, 400
   - used = false → si no, 400
7. Sistema:
   - Actualiza usuario: is_email_verified = true
   - Marca token como used = true
```

### ¿Por qué 24 horas y no 1 hora?

La verificación de email es menos crítica en términos de seguridad que el reset de contraseña.
Un usuario podría registrarse y no revisar su email de inmediato. 24 horas es un tiempo razonable
para que encuentre el email y haga clic en el enlace.

---

## Migraciones con Flyway

### ¿Qué es Flyway?

Flyway es una herramienta de migración de bases de datos para el ecosistema JVM que:

1. **Detecta automáticamente** qué scripts SQL ya fueron aplicados (guarda historial en tabla `flyway_schema_history`)
2. **Aplica solo los nuevos** scripts al arrancar la aplicación
3. **Verifica** integridad mediante checksums — si un script ya aplicado es modificado, Flyway falla

> **Comparativa con `proyecto-be-fe`**: Alembic (Python) funciona igual pero los scripts son en Python.
> Con Flyway los scripts son **SQL puro** — más portable y fácil de entender.

### Convención de nombres

```
V{número}__{descripción}.sql
```

| Archivo | Propósito |
|---|---|
| `V1__create_users_table.sql` | Crea la tabla `users` |
| `V2__create_password_reset_tokens_table.sql` | Crea la tabla `password_reset_tokens` |
| `V3__create_email_verification_tokens_table.sql` | Crea la tabla `email_verification_tokens` |
| `V4__add_locale_to_users.sql` | Agrega columna `locale VARCHAR(10) DEFAULT 'es'` a `users` |

> ⚠️ **Regla crítica**: Nunca modificar un archivo `.sql` de Flyway que ya fue aplicado.
> Si necesitas cambiar el esquema, **crea un nuevo script** `V4__...sql`.
> Flyway fallará si detecta que un script ya aplicado fue modificado (checksum mismatch).

### Ubicación en el proyecto

```
be/src/main/resources/
└── db/
    └── migration/
        ├── V1__create_users_table.sql
        ├── V2__create_password_reset_tokens_table.sql
        ├── V3__create_email_verification_tokens_table.sql
        └── V4__add_locale_to_users.sql
```

### Tabla de historial `flyway_schema_history`

Flyway crea automáticamente esta tabla en PostgreSQL para rastrear migraciones:

```sql
-- Esta tabla es gestionada automáticamente por Flyway
-- NO la modifiques manualmente
SELECT * FROM flyway_schema_history;
```

| Columna | Descripción |
|---|---|
| `installed_rank` | Orden de aplicación |
| `version` | Número de versión del script (ej: `1`, `2`, `3`) |
| `description` | Descripción del script |
| `type` | Siempre `SQL` para nuestros scripts |
| `script` | Nombre del archivo |
| `checksum` | Hash del contenido — detecta modificaciones |
| `installed_on` | Cuándo fue aplicado |
| `success` | Si la migración fue exitosa |

### ¿Cómo aplica Flyway las migraciones?

```
Al arrancar Spring Boot:
  1. Flyway conecta a PostgreSQL
  2. Verifica tabla flyway_schema_history (la crea si no existe)
  3. Compara scripts en resources/db/migration/ con los ya aplicados
  4. Aplica scripts pendientes en orden numérico
  5. Registra cada migración en flyway_schema_history
  6. Si todo OK → Spring Boot continúa arrancando
  7. Si hay error → Spring Boot falla y muestra el error SQL

En equivalencia Alembic (Python):
  alembic upgrade head        # equivale a arrancar Spring Boot con Flyway
  alembic current             # equivale a SELECT * FROM flyway_schema_history
  alembic revision --autogenerate  # NO tiene equivalente directo — Flyway no auto-genera
```

### Scripts SQL de migración

#### V1__create_users_table.sql

```sql
-- ¿Qué? Crea la tabla users para almacenar los usuarios del sistema.
-- ¿Para qué? Persistir datos de autenticación y perfil de cada usuario.
-- ¿Impacto? Tabla central del sistema — sin ella, no hay autenticación posible.

CREATE TABLE IF NOT EXISTS users (
    id                 UUID         NOT NULL DEFAULT gen_random_uuid(),
    email              VARCHAR(255) NOT NULL,
    full_name          VARCHAR(255) NOT NULL,
    hashed_password    VARCHAR(255) NOT NULL,
    is_active          BOOLEAN      NOT NULL DEFAULT TRUE,
    is_email_verified  BOOLEAN      NOT NULL DEFAULT FALSE,
    locale             VARCHAR(10)  NOT NULL DEFAULT 'es',
    created_at         TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMPTZ  NOT NULL DEFAULT NOW(),

    CONSTRAINT pk_users PRIMARY KEY (id),
    CONSTRAINT uq_users_email UNIQUE (email)
);

-- Índice explícito en email para búsquedas rápidas durante login
CREATE INDEX IF NOT EXISTS idx_users_email ON users (email);
```

#### V2__create_password_reset_tokens_table.sql

```sql
-- ¿Qué? Crea la tabla para tokens temporales de recuperación de contraseña.
-- ¿Para qué? Permitir el flujo "olvidé mi contraseña" de forma segura.
-- ¿Impacto? Sin esta tabla, no hay mecanismo de recuperación de cuentas.

CREATE TABLE IF NOT EXISTS password_reset_tokens (
    id          UUID         NOT NULL DEFAULT gen_random_uuid(),
    user_id     UUID         NOT NULL,
    token       VARCHAR(255) NOT NULL,
    expires_at  TIMESTAMPTZ  NOT NULL,
    used        BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),

    CONSTRAINT pk_password_reset_tokens PRIMARY KEY (id),
    CONSTRAINT uq_password_reset_tokens_token UNIQUE (token),
    CONSTRAINT fk_password_reset_tokens_user
        FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_password_reset_tokens_token ON password_reset_tokens (token);
```

#### V3__create_email_verification_tokens_table.sql

```sql
-- ¿Qué? Crea la tabla para tokens temporales de verificación de email.
-- ¿Para qué? Confirmar que el usuario es dueño del email con el que se registró.
-- ¿Impacto? Sin verificación, cualquiera podría registrarse con el email de otro.

CREATE TABLE IF NOT EXISTS email_verification_tokens (
    id          UUID         NOT NULL DEFAULT gen_random_uuid(),
    user_id     UUID         NOT NULL,
    token       VARCHAR(255) NOT NULL,
    expires_at  TIMESTAMPTZ  NOT NULL,
    used        BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),

    CONSTRAINT pk_email_verification_tokens PRIMARY KEY (id),
    CONSTRAINT uq_email_verification_tokens_token UNIQUE (token),
    CONSTRAINT fk_email_verification_tokens_user
        FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_email_verification_tokens_token ON email_verification_tokens (token);
```

#### V4__add_locale_to_users.sql

```sql
-- ¿Qué? Agrega la columna locale a la tabla users para soportar internacionalización (i18n).
-- ¿Para qué? Persistir la preferencia de idioma del usuario entre dispositivos y sesiones.
-- ¿Impacto? Sin esta columna, el idioma preferido solo se guarda en el navegador (localStorage)
--           y se pierde al ingresar desde otro dispositivo.

ALTER TABLE users
    ADD COLUMN locale VARCHAR(10) NOT NULL DEFAULT 'es';
```

---

## Convenciones de Nomenclatura

| Concepto | Convención | Ejemplo |
|---|---|---|
| Nombres de tablas | `snake_case`, plural | `users`, `password_reset_tokens` |
| Nombres de columnas | `snake_case` | `created_at`, `hashed_password`, `user_id` |
| Primary Keys | `id` (UUID) | `id UUID NOT NULL DEFAULT gen_random_uuid()` |
| Foreign Keys | `<tabla_singular>_id` | `user_id UUID NOT NULL` |
| Restricciones PK | `pk_<tabla>` | `pk_users`, `pk_password_reset_tokens` |
| Restricciones UNIQUE | `uq_<tabla>_<columna>` | `uq_users_email` |
| Restricciones FK | `fk_<tabla_origen>_<tabla_destino>` | `fk_password_reset_tokens_user` |
| Índices | `idx_<tabla>_<columna>` | `idx_users_email` |
| Timestamps | `created_at`, `updated_at` | En todas las tablas |

---

## Mapeo JPA — Kotlin ↔ PostgreSQL

| Tipo Kotlin | Tipo PostgreSQL | Anotación JPA |
|---|---|---|
| `UUID` | `UUID` | `@GeneratedValue(strategy = GenerationType.UUID)` |
| `String` | `VARCHAR(255)` | `@Column(length = 255)` |
| `Boolean` | `BOOLEAN` | `@Column(columnDefinition = "BOOLEAN DEFAULT TRUE")` |
| `OffsetDateTime` | `TIMESTAMPTZ` | `@Column(columnDefinition = "TIMESTAMPTZ DEFAULT NOW()")` |

---

## Comparativa — Flyway vs Alembic

| Aspecto | Alembic (Python) | Flyway (JVM) |
|---|---|---|
| Lenguaje de scripts | Python | **SQL puro** |
| Auto-generación de migraciones | Sí (`--autogenerate` con SQLAlchemy) | No (scripts manuales) |
| Aplicación | `alembic upgrade head` | Automático al arrancar Spring Boot |
| Historial | Tabla `alembic_version` | Tabla `flyway_schema_history` |
| Reversión | Soporta `downgrade` | Solo `repair` — no hay downgrade nativo |
| Convención de nombres | `<timestamp>_descripcion.py` | `V<n>__descripcion.sql` |
| Integración Spring | N/A | Spring Boot auto-configura Flyway |
