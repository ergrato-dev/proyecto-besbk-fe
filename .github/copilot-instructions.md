<!--
  ¿Qué? Archivo de instrucciones para GitHub Copilot y colaboradores del proyecto.
  ¿Para qué? Define TODAS las reglas, convenciones, tecnologías y estándares que se
  deben seguir en cada archivo, commit, test y decisión técnica del proyecto.
  ¿Impacto? Garantiza consistencia, calidad y enfoque pedagógico en todo el código generado.
  Este archivo es la "ley" del proyecto — todo lo que se haga debe alinearse con estas reglas.
-->

---

# 🎓 Instrucciones del Proyecto — NN Auth System (Kotlin Edition)

## 1. Identidad del Proyecto

| Campo               | Valor                                                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Nombre**          | NN Auth System — Kotlin Edition                                                                                         |
| **Tipo**            | Proyecto educativo — SENA                                                                                               |
| **Propósito**       | Sistema de autenticación completo (registro, login, cambio y recuperación de contraseña) para una empresa genérica "NN" |
| **Enfoque**         | Aprendizaje guiado: cada línea de código y documentación debe enseñar                                                   |
| **Referencia**      | Idéntica funcionalidad a [`proyecto-be-fe`](https://github.com/ergrato-dev/proyecto-be-fe) — diferente stack            |
| **Fecha de inicio** | Marzo 2026                                                                                                              |

---

## 2. Stack Tecnológico

### 2.1 Backend (`be/`)

| Tecnología                    | Versión | Propósito                                                 |
| ----------------------------- | ------- | --------------------------------------------------------- |
| Kotlin                        | 1.9+    | Lenguaje principal del backend                            |
| JDK                           | 21 LTS  | Runtime de Kotlin/JVM                                     |
| Spring Boot                   | 3.x     | Framework principal — auto-configuración, embedded Tomcat |
| Spring Security               | 6.x     | Autenticación, autorización y seguridad HTTP              |
| Spring Data JPA               | 3.x     | ORM y repositorios sobre Hibernate                        |
| Hibernate                     | 6.x     | Implementación JPA — mapeo objeto-relacional              |
| Flyway                        | 9+      | Migraciones de base de datos versionadas                  |
| JJWT (java-jwt)               | 0.12+   | Creación y verificación de tokens JWT                     |
| BCrypt (Spring Security)      | —       | Hashing seguro de contraseñas                             |
| SpringDoc OpenAPI             | 2.x     | Documentación Swagger auto-generada                       |
| JavaMailSender (Spring)       | —       | Envío de emails                                           |
| Bean Validation (Jakarta)     | 3.x     | Validación de DTOs (request/response)                     |
| Bucket4j                      | 8+      | Rate limiting (equivalente a slowapi en Python)           |
| JUnit 5                       | 5.x     | Framework de testing                                      |
| MockMvc                       | —       | Tests de integración de endpoints HTTP                    |
| JaCoCo                        | —       | Cobertura de tests                                        |
| ktlint                        | 1.x     | Linter + formatter para Kotlin                            |
| Gradle (Kotlin DSL)           | 8+      | Herramienta de build                                      |
| psql driver (PostgreSQL JDBC) | 42+     | Driver de PostgreSQL para JVM                             |

### 2.2 Frontend (`fe/`)

| Tecnología      | Versión | Propósito                                    |
| --------------- | ------- | -------------------------------------------- |
| Node.js         | 20 LTS+ | Runtime de JavaScript                        |
| React           | 18+     | Biblioteca para interfaces de usuario        |
| Vite            | 6+      | Bundler y dev server ultrarrápido            |
| TypeScript      | 5.0+    | Superset tipado de JavaScript                |
| TailwindCSS     | 4+      | Framework CSS utility-first                  |
| React Router    | 7+      | Enrutamiento del lado del cliente            |
| Axios           | latest  | Cliente HTTP para comunicación con la API    |
| Vitest                           | latest  | Framework de testing compatible con Vite     |
| Testing Library                  | latest  | Utilidades de testing para componentes React |
| ESLint                           | latest  | Linter para TypeScript/React                 |
| Prettier                         | latest  | Formateador de código                        |
| i18next                          | latest  | Motor de internacionalización (i18n)         |
| react-i18next                    | latest  | Integración de i18next con React (hooks/HOC) |
| i18next-browser-languagedetector | latest  | Detecta idioma del navegador automáticamente |

> ⚠️ El frontend es **idéntico** al del proyecto de referencia `proyecto-be-fe`.
> No hay cambios en el stack de frontend entre ambos proyectos.

### 2.3 Base de Datos

| Tecnología     | Versión | Propósito                                       |
| -------------- | ------- | ----------------------------------------------- |
| PostgreSQL     | 17+     | Base de datos relacional principal              |
| Docker Compose | latest  | Orquestación de contenedores (BD en desarrollo) |

### 2.4 Autenticación

| Concepto      | Implementación                                                |
| ------------- | ------------------------------------------------------------- |
| Método        | JWT (JSON Web Tokens) — stateless                             |
| Access Token  | Duración: 15 minutos                                          |
| Refresh Token | Duración: 7 días                                              |
| Hashing       | BCryptPasswordEncoder (Spring Security)                       |
| Flujos        | Registro, Login, Cambio de contraseña, Recuperación por email |

---

## 3. Reglas de Lenguaje — OBLIGATORIAS

### 3.1 Nomenclatura técnica → INGLÉS

Todo lo que sea código debe estar en **inglés**:

- Variables, funciones, clases, métodos
- Nombres de archivos y carpetas de código
- Endpoints y rutas de la API
- Nombres de tablas y columnas en la base de datos
- Nombres de componentes React
- Mensajes de commits
- Ramas de git

```kotlin
// ✅ CORRECTO
fun getUserByEmail(email: String): User? { ... }

// ❌ INCORRECTO
fun obtenerUsuarioPorEmail(correo: String): Usuario? { ... }
```

### 3.2 Comentarios y documentación → ESPAÑOL

Todo lo que sea documentación o comentarios debe estar en **español**:

- Comentarios en el código (`//`, `/* */`)
- KDoc de funciones y clases
- Archivos de documentación (`.md`)
- README.md

### 3.3 Regla del comentario pedagógico — ¿QUÉ? ¿PARA QUÉ? ¿IMPACTO?

**Cada comentario significativo debe responder tres preguntas:**

```kotlin
/**
 * ¿Qué? Hashea la contraseña del usuario usando BCrypt.
 * ¿Para qué? Almacenar contraseñas de forma segura, nunca en texto plano.
 * ¿Impacto? Si se omite el hashing, las contraseñas quedan expuestas ante una filtración de la BD.
 */
fun hashPassword(rawPassword: String): String =
    passwordEncoder.encode(rawPassword)
```

```typescript
/**
 * ¿Qué? Hook personalizado que provee el estado de autenticación y sus acciones.
 * ¿Para qué? Centralizar la lógica de auth para que cualquier componente pueda consumirla.
 * ¿Impacto? Sin este hook, cada componente tendría que reimplementar la lógica de auth,
 *           causando duplicación de código y posibles inconsistencias.
 */
export function useAuth(): AuthContextType { ... }
```

### 3.4 Cabecera de archivo obligatoria

Cada archivo nuevo debe incluir un **comentario de cabecera** al inicio:

```kotlin
/**
 * Archivo: JwtTokenProvider.kt
 * Descripción: Utilidades de seguridad — generación y verificación de tokens JWT.
 * ¿Para qué? Proveer funciones reutilizables de seguridad que se usan en todo el sistema de auth.
 * ¿Impacto? Es la base de la seguridad del sistema. Un error aquí compromete toda la autenticación.
 */
```

```typescript
/**
 * Archivo: AuthContext.tsx
 * Descripción: Contexto de React que gestiona el estado de autenticación global.
 * ¿Para qué? Proveer a toda la aplicación acceso al usuario autenticado, tokens y acciones de auth.
 * ¿Impacto? Sin este contexto, no habría forma de saber si el usuario está logueado
 *           ni de proteger rutas que requieren autenticación.
 */
```

---

## 4. Reglas de Entorno y Herramientas — OBLIGATORIAS

### 4.1 Kotlin/JVM — SIEMPRE usar Gradle Wrapper

```bash
# ✅ CORRECTO — Usar siempre el wrapper incluido en el proyecto
./gradlew build
./gradlew test
./gradlew bootRun

# ❌ INCORRECTO — No usar Gradle instalado globalmente (puede diferir en versión)
gradle build
```

**¿Por qué?** El wrapper (`gradlew`) usa la versión de Gradle definida en `gradle/wrapper/gradle-wrapper.properties`,
garantizando que todos los desarrolladores usen **exactamente** la misma versión.

### 4.2 Node.js — SIEMPRE usar `pnpm`

```bash
# ✅ CORRECTO
pnpm install
pnpm add axios
pnpm add -D vitest
pnpm dev
pnpm test
pnpm build

# ❌ INCORRECTO — NUNCA usar npm
npm install        # ← PROHIBIDO
npm run dev        # ← PROHIBIDO

# ❌ INCORRECTO — NUNCA usar yarn
yarn install       # ← PROHIBIDO
```

### 4.3 Variables de entorno

- **NUNCA** hardcodear credenciales, URLs de base de datos, secrets, o configuración sensible.
- Usar archivos `.env` (no versionados en git).
- Proveer **siempre** un `.env.example` con las variables necesarias y valores de ejemplo.
- Las variables se leen en Spring Boot mediante `@Value` o `@ConfigurationProperties`.

```bash
# be/.env.example
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/nn_auth_db
SPRING_DATASOURCE_USERNAME=nn_user
SPRING_DATASOURCE_PASSWORD=nn_password
JWT_SECRET=your-super-secret-key-change-in-production-min-32-chars
JWT_EXPIRATION_MS=900000
JWT_REFRESH_EXPIRATION_MS=604800000
FRONTEND_URL=http://localhost:5173
SMTP_HOST=localhost
SMTP_PORT=1025
APP_ENVIRONMENT=development
```

---

## 5. Estructura del Proyecto

```
proyecto-besbk-fe/                     # Raíz del monorepo
├── .github/
│   ├── copilot-instructions.md        # ← ESTE ARCHIVO — reglas del proyecto
│   └── prompts/                       # Prompts de agente para tareas recurrentes
│       ├── commit-message.prompt.md   # Generar mensajes Conventional Commits
│       ├── db-migration.prompt.md     # Crear migraciones Flyway + modelos Kotlin
│       ├── new-endpoint.prompt.md     # Crear endpoint Kotlin completo (DTO + Service + Controller + test)
│       ├── new-fe-component.prompt.md # Crear componente UI React reutilizable
│       ├── new-fe-page.prompt.md      # Crear página React con ruta, layout y test
│       └── security-review.prompt.md  # Auditoría OWASP Top 10 para Spring Boot + Kotlin
├── .gitignore                         # Archivos ignorados por git
├── docker-compose.yml                 # Servicios: PostgreSQL, BE, FE, Mailpit
├── README.md                          # Documentación principal del proyecto
│
├── _docs/                             # 📚 Documentación del proyecto
│   ├── referencia-tecnica/
│   │   ├── architecture.md            # Arquitectura general y diagramas
│   │   ├── api-endpoints.md           # Documentación de todos los endpoints
│   │   ├── database-schema.md         # Esquema de base de datos y ER diagram
│   │   └── design-system.md           # Sistema de color fuchsia, tokens accent-*, logo SVG
│   └── conceptos/
│       ├── owasp-top-10.md           # Implementación OWASP Top 10
│       └── accesibilidad-aria-wcag.md # Accesibilidad ARIA/WCAG
│
├── be/                                # 🟠 Backend — Kotlin + Spring Boot
│   ├── .env                           # Variables de entorno (NO versionado)
│   ├── .env.example                   # Plantilla de variables de entorno
│   ├── build.gradle.kts               # Dependencias y configuración Gradle
│   ├── settings.gradle.kts            # Nombre y configuración del proyecto
│   ├── gradlew / gradlew.bat          # Gradle Wrapper
│   ├── Dockerfile                     # Imagen Docker del backend
│   └── src/
│       ├── main/
│       │   ├── kotlin/com/nn/auth/
│       │   │   ├── NnAuthApplication.kt      # Punto de entrada — @SpringBootApplication
│       │   │   ├── config/
│       │   │   │   ├── SecurityConfig.kt     # Spring Security — CORS, filtros, reglas
│       │   │   │   ├── OpenApiConfig.kt      # Swagger/OpenAPI — documentación auto
│       │   │   │   └── AppProperties.kt      # @ConfigurationProperties — .env a POJO
│       │   │   ├── controller/
│       │   │   │   ├── AuthController.kt     # POST /register, /login, /refresh, etc.
│       │   │   │   └── UserController.kt     # GET /me
│       │   │   ├── service/
│       │   │   │   └── AuthService.kt        # Lógica de negocio de autenticación
│       │   │   ├── repository/
│       │   │   │   ├── UserRepository.kt     # Spring Data JPA — tabla users
│       │   │   │   ├── PasswordResetTokenRepository.kt
│       │   │   │   └── EmailVerificationTokenRepository.kt
│       │   │   ├── model/
│       │   │   │   ├── User.kt               # Entidad JPA — tabla users
│       │   │   │   ├── PasswordResetToken.kt
│       │   │   │   └── EmailVerificationToken.kt
│       │   │   ├── dto/
│       │   │   │   └── AuthDtos.kt           # RegisterRequest, LoginRequest, TokenResponse, etc.
│       │   │   ├── security/
│       │   │   │   ├── JwtTokenProvider.kt   # Generación y verificación de JWT
│       │   │   │   ├── JwtAuthFilter.kt      # OncePerRequestFilter — valida JWT en cada request
│       │   │   │   └── UserDetailsServiceImpl.kt # Carga usuario desde BD para Spring Security
│       │   │   └── util/
│       │   │       ├── EmailService.kt       # Envío de emails (verificación + reset)
│       │   │       └── AuditLogService.kt    # Registro de eventos de seguridad
│       │   └── resources/
│       │       ├── application.yml           # Configuración principal de Spring Boot
│       │       ├── application-dev.yml       # Overrides para desarrollo
│       │       └── db/migration/            # Scripts Flyway
│       │           ├── V1__create_users_table.sql
│       │           ├── V2__create_password_reset_tokens_table.sql
│       │           └── V3__create_email_verification_tokens_table.sql
│       └── test/
│           └── kotlin/com/nn/auth/
│               ├── controller/
│               │   └── AuthControllerTest.kt  # Tests de integración con MockMvc
│               └── service/
│                   └── AuthServiceTest.kt     # Tests unitarios del servicio
│
└── fe/                                # ⚛️ Frontend — React + Vite + TypeScript
    ├── .env                           # Variables de entorno (NO versionado)
    ├── .env.example                   # Plantilla de variables de entorno
    ├── index.html                     # HTML base de Vite
    ├── package.json                   # Dependencias y scripts
    ├── pnpm-lock.yaml                 # Lockfile de pnpm
    ├── vite.config.ts                 # Configuración de Vite
    ├── tsconfig.json                  # Configuración de TypeScript
    ├── eslint.config.js               # Configuración de ESLint
    ├── Dockerfile                     # Imagen Docker del frontend (multi-stage + Nginx)
    └── src/
        ├── main.tsx                   # Punto de entrada — renderiza App en el DOM
        ├── App.tsx                    # Componente raíz — define rutas
        ├── index.css                  # Estilos globales + imports de Tailwind
        ├── i18n.ts                    # Configuración de i18next (idiomas, fallback, detector)
        ├── api/
        │   ├── auth.ts                # Funciones para cada endpoint de auth
        │   └── axios.ts               # Instancia Axios con interceptores JWT
        ├── components/
        │   ├── ui/                    # Componentes UI genéricos (Button, Input, Alert, LanguageSwitcher)
        │   └── layout/               # Layout, Navbar, Footer, ProtectedRoute
        ├── locales/
        │   ├── es/
        │   │   └── translation.json   # Catálogo de traducciones en español
        │   └── en/
        │       └── translation.json   # Catálogo de traducciones en inglés
        ├── pages/                     # Páginas/vistas (una por ruta)
        ├── hooks/
        │   └── useAuth.ts
        ├── context/
        │   └── AuthContext.tsx
        └── types/
            └── auth.ts                # LoginRequest, RegisterRequest, UserResponse, etc.
```

---

## 6. Convenciones de Código

### 6.1 Kotlin (Backend)

| Aspecto             | Convención                                                                |
| ------------------- | ------------------------------------------------------------------------- |
| Estilo              | Kotlin Coding Conventions + ktlint                                        |
| Naming variables    | `camelCase`                                                               |
| Naming clases       | `PascalCase`                                                              |
| Naming constantes   | `UPPER_SNAKE_CASE` (companion object)                                     |
| Tipos explícitos    | Obligatorios en parámetros y retornos de funciones públicas               |
| KDoc                | Formato estándar, en español                                              |
| Imports             | Separados por grupos (stdlib → spring → local)                            |
| Nulabilidad         | Preferir tipos no-nulos; usar `?` solo cuando sea semánticamente correcto |
| Data classes        | Para entidades y DTOs — no usar POJOs con getters/setters                 |
| Extension functions | Para reutilizar lógica sin herencia innecesaria                           |

```kotlin
// ✅ Ejemplo de función bien documentada y tipada
/**
 * ¿Qué? Busca un usuario en la BD por su dirección de email.
 * ¿Para qué? Verificar existencia durante login y registro.
 * ¿Impacto? Si retorna null sin verificar, el sistema podría fallar silenciosamente.
 *
 * @param email Dirección de email a buscar.
 * @return El usuario encontrado, o null si no existe.
 */
fun findByEmail(email: String): User? =
    userRepository.findByEmail(email)
```

### 6.1.1 Patrones Kotlin obligatorios (diferencias clave vs Spring Boot Java)

#### DTOs — usar `data class`, NO Lombok ni Java `record`

```kotlin
// ✅ CORRECTO — data class Kotlin con Bean Validation
data class RegisterRequest(

    // ¿Qué? @field: dirige la anotación al campo backing, no a la property Kotlin.
    // ¿Para qué? Bean Validation solo funciona si las anotaciones apuntan al campo JVM.
    // ¿Impacto? Sin @field:, las validaciones de @NotBlank/@Email se ignoran silenciosamente.
    @field:NotBlank(message = "El email es requerido")
    @field:Email(message = "Formato de email inválido")
    val email: String,

    @field:NotBlank(message = "La contraseña es requerida")
    @field:Size(min = 8, message = "Mínimo 8 caracteres")
    @field:Pattern(regexp = ".*[A-Z].*", message = "Al menos una mayúscula")
    @field:Pattern(regexp = ".*[a-z].*", message = "Al menos una minúscula")
    @field:Pattern(regexp = ".*\\d.*", message = "Al menos un número")
    val password: String
)

// ❌ INCORRECTO en Kotlin — Java record (esto no es Kotlin)
public record RegisterRequest(@NotBlank String email) {}

// ❌ INCORRECTO en Kotlin — Lombok (esto no aplica a Kotlin)
@Data @Builder
public class RegisterRequest { @NotBlank private String email; }
```

#### Inyección de dependencias — constructor, NO `@Autowired` en campo

```kotlin
// ✅ CORRECTO — constructor injection (Kotlin idiomático)
@Service
class AuthService(
    private val userRepository: UserRepository,
    private val passwordEncoder: BCryptPasswordEncoder,
    private val jwtTokenProvider: JwtTokenProvider,
    private val emailService: EmailService
) {
    // Spring inyecta automáticamente por constructor
}

// ❌ INCORRECTO — @Autowired en campo (Java style, evitar en Kotlin)
@Service
class AuthService {
    @Autowired
    private lateinit var userRepository: UserRepository
}
```

#### Entidades JPA — `data class` con `@Entity`

```kotlin
// ✅ CORRECTO — entidad JPA en Kotlin
@Entity
@Table(name = "users")
data class User(

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    val id: UUID = UUID.randomUUID(),

    @Column(name = "email", nullable = false, unique = true, length = 255)
    val email: String,

    @Column(name = "hashed_password", nullable = false, length = 255)
    var hashedPassword: String,   // var porque se actualiza al cambiar contraseña

    @Column(name = "full_name", nullable = false, length = 255)
    val fullName: String,

    @Column(name = "is_active", nullable = false)
    var isActive: Boolean = true,

    @Column(name = "is_email_verified", nullable = false)
    var isEmailVerified: Boolean = false,

    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: Instant = Instant.now(),

    @Column(name = "updated_at", nullable = false)
    var updatedAt: Instant = Instant.now()
)
```

> **Nota sobre `data class` y JPA**: Spring Boot con el plugin `kotlin-jpa` genera automáticamente
> el constructor sin argumentos que JPA requiere. Asegurarse de incluir el plugin en `build.gradle.kts`:
>
> ```kotlin
> plugins {
>     kotlin("plugin.jpa") version kotlinVersion
> }
> ```

#### Manejo de errores — `ResponseStatusException`

```kotlin
// ✅ CORRECTO en Kotlin/Spring Boot
throw ResponseStatusException(
    HttpStatus.CONFLICT,
    "Este correo electrónico ya está registrado"
)

// ✅ También válido — @ControllerAdvice con clases de excepción propias
throw EmailAlreadyExistsException("Este correo electrónico ya está registrado")
```

### 6.2 TypeScript/React (Frontend)

| Aspecto             | Convención                                                            |
| ------------------- | --------------------------------------------------------------------- |
| Estilo              | ESLint + Prettier                                                     |
| Naming variables    | `camelCase`                                                           |
| Naming componentes  | `PascalCase`                                                          |
| Naming archivos     | `PascalCase` para componentes, `camelCase` para utilidades            |
| Naming tipos        | `PascalCase` con sufijo descriptivo (`UserResponse`, `LoginRequest`)  |
| Componentes         | **Funcionales** con hooks — nunca clases                              |
| Interfaces vs Types | Preferir `interface` para objetos, `type` para uniones/intersecciones |
| CSS                 | TailwindCSS utility classes — evitar CSS custom                       |
| Strict mode         | `"strict": true` en tsconfig.json                                     |

### 6.3 SQL / Base de Datos

| Aspecto             | Convención                                                                     |
| ------------------- | ------------------------------------------------------------------------------ |
| Nombres de tablas   | `snake_case`, plural (`users`, `password_reset_tokens`)                        |
| Nombres de columnas | `snake_case` (`created_at`, `hashed_password`)                                 |
| Primary Keys        | `id` (UUID)                                                                    |
| Foreign Keys        | `<tabla_singular>_id` (ej: `user_id`)                                          |
| Timestamps          | `created_at`, `updated_at` en toda tabla                                       |
| Migraciones         | Siempre vía Flyway — scripts en `resources/db/migration/` con prefijo `V<n>__` |

> **Diferencia respecto a `proyecto-be-fe`**: El proyecto de referencia usa Alembic para migraciones.
> Este proyecto usa **Flyway** — mismo concepto, diferente herramienta. Los scripts son SQL puro.

---

### 6.4 Internacionalización (i18n) — Frontend

> **Conceptos clave para los aprendices:**
>
> - **i18n** (internacionalización): preparar el código para admitir múltiples idiomas sin cambios de código
> - **l10n** (localización): adaptar el contenido al idioma y región específicos
> - **locale**: identificador de idioma/región (ej: `es`, `en`, `es-CO`, `en-US`)

| Aspecto                    | Convención                                                              |
| -------------------------- | ----------------------------------------------------------------------- |
| Librería                   | `react-i18next` + `i18next` + `i18next-browser-languagedetector`        |
| Idiomas soportados         | `es` (español — por defecto) y `en` (inglés)                            |
| Archivos de traducción     | `src/locales/{locale}/translation.json`                                 |
| Namespace                  | Un único namespace `translation` (simplicidad pedagógica)               |
| Claves de traducción       | **inglés**, `camelCase`, agrupadas por sección/página                   |
| Valores de traducción      | En el idioma correspondiente al archivo                                 |
| Textos con variables       | Usar interpolación `{{variable}}` (ej: `"welcome": "Hola, {{name}}"`)  |
| Almacenamiento preferencia | `localStorage` (clave `i18nextLng`) + columna `locale` en BD            |
| Detección automática       | `navigator.language` → fallback a `es`                                  |

```typescript
// ✅ CORRECTO — Clave en inglés, sintaxis de hook
import { useTranslation } from "react-i18next";

function LoginPage() {
  const { t } = useTranslation();

  return <h1>{t("auth.login.title")}</h1>;
  // Renderiza: "Iniciar sesión" (es) | "Sign in" (en)
}

// ✅ CORRECTO — Interpolación de variables
function DashboardPage() {
  const { t } = useTranslation();
  const { user } = useAuth();

  return <h1>{t("dashboard.welcome", { name: user?.fullName })}</h1>;
  // Renderiza: "Bienvenido, Carlos" | "Welcome, Carlos"
}

// ❌ INCORRECTO — Texto hardcoded en componentes
function LoginPage() {
  return <h1>Iniciar sesión</h1>; // ← No se puede traducir
}
```

```json
// ✅ CORRECTO — Estructura de un archivo de traducción (es/translation.json)
{
  "auth": {
    "login": {
      "title": "Iniciar sesión",
      "submit": "Iniciar sesión"
    }
  },
  "dashboard": {
    "welcome": "Bienvenido, {{name}}"
  }
}
```

**Convenciones de estructura de claves:**

```
auth.login.*          → textos de la página de login
auth.register.*       → textos del formulario de registro
auth.changePassword.* → textos de la página cambio de contraseña
dashboard.*           → textos del dashboard
nav.*                 → textos de la navbar (brand, logout, etc.)
common.*              → textos reutilizables (loading, cancel, etc.)
language.*            → textos del selector de idioma
```

---

## 7. Conventional Commits — OBLIGATORIO

### 7.1 Formato

```
type(scope): short description in english

What: Detailed description of what was done
For: Why this change is needed
Impact: What effect this has on the system
```

### 7.2 Tipos permitidos

| Tipo       | Uso                                                  |
| ---------- | ---------------------------------------------------- |
| `feat`     | Nueva funcionalidad                                  |
| `fix`      | Corrección de bug                                    |
| `docs`     | Solo documentación                                   |
| `style`    | Formato, espacios, puntos y comas (no afecta lógica) |
| `refactor` | Reestructuración sin cambiar funcionalidad           |
| `test`     | Agregar o corregir tests                             |
| `chore`    | Tareas de mantenimiento, configuración, dependencias |
| `ci`       | Cambios en CI/CD                                     |
| `perf`     | Mejoras de rendimiento                               |

### 7.3 Scopes sugeridos

- `auth` — Autenticación y autorización
- `user` — Modelo/funcionalidad de usuario
- `db` — Base de datos y migraciones
- `api` — Endpoints y controladores
- `ui` — Componentes y estilos del frontend
- `i18n` — Internacionalización, catálogos de traducción, LanguageSwitcher
- `config` — Configuración y entorno
- `test` — Tests
- `deps` — Dependencias
- `security` — Spring Security, JWT, rate limiting

### 7.4 Ejemplos

```bash
# ✅ Ejemplo de commit de feature backend
git commit -m "feat(auth): add user registration endpoint

What: Creates POST /api/v1/auth/register with Bean Validation,
      BCrypt password hashing, and duplicate email check
For: Allow new users to create accounts in the NN Auth System
Impact: Enables the user onboarding flow; stores hashed passwords using
        BCryptPasswordEncoder in the users table"

# ✅ Ejemplo de migración
git commit -m "feat(db): add V1 Flyway migration for users table

What: Adds V1__create_users_table.sql with UUID PK, email UNIQUE index,
      BCrypt hashed_password, is_active and is_email_verified flags
For: Define the schema for the users table managed by Flyway
Impact: Applied automatically on startup — no manual SQL needed"
```

---

## 8. Calidad — NO es Opcional, es OBLIGACIÓN

### 8.1 Principio fundamental

> **Código que se genera, código que se prueba.**

Cada función, endpoint, componente o utilidad que se cree **debe** tener su test correspondiente.
No se considera "terminada" una feature hasta que sus tests pasen.

### 8.2 Testing — Backend

| Herramienta       | Propósito                                                                   |
| ----------------- | --------------------------------------------------------------------------- |
| `JUnit 5`         | Framework principal de testing                                              |
| `MockMvc`         | Tests de integración de endpoints HTTP                                      |
| `@SpringBootTest` | Contexto completo de Spring para tests de integración                       |
| `@WebMvcTest`     | Contexto parcial para tests de controladores                                |
| `JaCoCo`          | Cobertura de código                                                         |
| `H2` (en memoria) | Base de datos en memoria para tests (o Testcontainers para PostgreSQL real) |

```bash
# Ejecutar todos los tests del backend
cd be && ./gradlew test

# Ejecutar con cobertura
./gradlew test jacocoTestReport

# Ejecutar un test específico
./gradlew test --tests "com.nn.auth.controller.AuthControllerTest.registerUser_shouldReturn201"
```

**Cobertura mínima esperada**: 80% en servicios y controladores.

### 8.3 Testing — Frontend

```bash
# Ejecutar todos los tests del frontend
cd fe && pnpm test

# Ejecutar en modo watch
pnpm test:watch

# Ejecutar con cobertura
pnpm test:coverage
```

### 8.4 Linting y Formateo

```bash
# Backend — ktlint
cd be && ./gradlew ktlintCheck      # Verificar errores
cd be && ./gradlew ktlintFormat     # Formatear código

# Frontend — ESLint + Prettier
cd fe && pnpm lint                  # Verificar errores
cd fe && pnpm format                # Formatear código
```

### 8.5 Checklist antes de commit

- [ ] ¿El código Kotlin tiene tipos explícitos en funciones públicas?
- [ ] ¿El código TypeScript tiene tipos completos (`strict: true`)?
- [ ] ¿Hay comentarios pedagógicos (¿Qué? ¿Para qué? ¿Impacto?)?
- [ ] ¿Los tests pasan? (`./gradlew test` / `pnpm test`)
- [ ] ¿El linter no reporta errores? (`./gradlew ktlintCheck` / `pnpm lint`)
- [ ] ¿El commit sigue Conventional Commits con What/For/Impact?
- [ ] ¿Las variables sensibles están en `.env` y no hardcodeadas?
- [ ] ¿El `.env.example` se actualizó si se agregaron nuevas variables?

---

## 9. Seguridad — Mejores Prácticas

### 9.1 Contraseñas

- **SIEMPRE** hashear con `BCryptPasswordEncoder` antes de almacenar
- **NUNCA** almacenar contraseñas en texto plano
- **NUNCA** loggear contraseñas ni incluirlas en responses
- Validar fortaleza mínima: ≥8 caracteres, al menos 1 mayúscula, 1 minúscula, 1 número

### 9.2 JWT (Tokens)

- Access Token: corta duración (15 min) — se envía en header `Authorization: Bearer <token>`
- Refresh Token: larga duración (7 días) — se usa solo para obtener nuevos access tokens
- Secret key: mínimo 32 caracteres, aleatoria, en variable de entorno
- Algoritmo: HS256 (HMAC-SHA256)
- **NUNCA** almacenar tokens en `localStorage` en producción (usar memoria o httpOnly cookies)

### 9.3 CORS

- Configurar orígenes permitidos explícitamente en `SecurityConfig.kt`
- En desarrollo: permitir `http://localhost:5173` (Vite dev server)
- En producción: nunca usar `allowedOrigins("*")`

### 9.4 API

- Versionamiento: `/api/v1/...`
- Rate limiting en endpoints sensibles con Bucket4j (prevenir brute force)
- Validación de inputs con Bean Validation — nunca confiar en datos del cliente
- Mensajes de error genéricos en auth (no revelar si el email existe)
- Swagger UI deshabilitado en producción (`APP_ENVIRONMENT=production`)

### 9.5 Base de datos

- Usar siempre Spring Data JPA / Hibernate (nunca raw SQL con concatenación)
- Queries parametrizadas — Spring Data las genera automáticamente
- Credenciales en variables de entorno

---

## 10. Estructura de la API

### 10.1 Prefijo base

Todos los endpoints van bajo `/api/v1/`

### 10.2 Endpoints de autenticación (`/api/v1/auth/`)

| Método | Ruta               | Descripción                           | Auth requerida |
| ------ | ------------------ | ------------------------------------- | -------------- |
| POST   | `/register`        | Registrar nuevo usuario               | No             |
| POST   | `/login`           | Iniciar sesión, obtener tokens        | No             |
| POST   | `/refresh`         | Renovar access token con refresh      | No (\*)        |
| POST   | `/change-password` | Cambiar contraseña (usuario logueado) | Sí             |
| POST   | `/forgot-password` | Solicitar email de recuperación       | No             |
| POST   | `/reset-password`  | Restablecer contraseña con token      | No (\*)        |
| POST   | `/verify-email`    | Verificar dirección de email          | No (\*)        |

(\*) Requiere un token válido (refresh, reset o verification), pero no el access token estándar.

### 10.3 Endpoints de usuario (`/api/v1/users/`)

| Método | Ruta  | Descripción                       | Auth requerida |
| ------ | ----- | --------------------------------- | -------------- |
| GET    | `/me` | Obtener perfil del usuario actual | Sí             |

---

## 11. Esquema de Base de Datos

### 11.1 Tabla `users`

| Columna             | Tipo         | Restricciones                 |
| ------------------- | ------------ | ----------------------------- |
| `id`                | UUID         | PK, default gen_random_uuid()               |
| `email`             | VARCHAR(255) | UNIQUE, NOT NULL, INDEXED                   |
| `full_name`         | VARCHAR(255) | NOT NULL                                    |
| `hashed_password`   | VARCHAR(255) | NOT NULL                                    |
| `is_active`         | BOOLEAN      | DEFAULT TRUE                                |
| `is_email_verified` | BOOLEAN      | DEFAULT FALSE                               |
| `locale`            | VARCHAR(10)  | DEFAULT 'es', NOT NULL                      |
| `created_at`        | TIMESTAMPTZ  | DEFAULT NOW(), NOT NULL                     |
| `updated_at`        | TIMESTAMPTZ  | DEFAULT NOW(), NOT NULL                     |

### 11.2 Tabla `password_reset_tokens`

| Columna      | Tipo         | Restricciones                 |
| ------------ | ------------ | ----------------------------- |
| `id`         | UUID         | PK, default gen_random_uuid() |
| `user_id`    | UUID         | FK → users.id, CASCADE DELETE |
| `token`      | VARCHAR(255) | UNIQUE, NOT NULL, INDEXED     |
| `expires_at` | TIMESTAMPTZ  | NOT NULL                      |
| `used`       | BOOLEAN      | DEFAULT FALSE                 |
| `created_at` | TIMESTAMPTZ  | DEFAULT NOW(), NOT NULL       |

### 11.3 Tabla `email_verification_tokens`

| Columna      | Tipo         | Restricciones                 |
| ------------ | ------------ | ----------------------------- |
| `id`         | UUID         | PK, default gen_random_uuid() |
| `user_id`    | UUID         | FK → users.id, CASCADE DELETE |
| `token`      | VARCHAR(255) | UNIQUE, NOT NULL, INDEXED     |
| `expires_at` | TIMESTAMPTZ  | NOT NULL                      |
| `used`       | BOOLEAN      | DEFAULT FALSE                 |
| `created_at` | TIMESTAMPTZ  | DEFAULT NOW(), NOT NULL       |

> **Nota**: El esquema de BD es **casi idéntico** al del proyecto de referencia `proyecto-be-fe`.
> Diferencias: (1) herramienta de migración: Alembic (Python) → Flyway (JVM); (2) la columna `locale`
> en `users` usa `VARCHAR(10)` con `DEFAULT 'es'` en ambos proyectos.

---

## 12. Flujos de Autenticación

### 12.1 Registro

```
Cliente → POST /api/v1/auth/register { email, full_name, password }
  → Validar datos (Bean Validation — @Valid)
  → Verificar email no duplicado
  → Hashear password (BCryptPasswordEncoder)
  → Crear usuario en BD (is_email_verified = false)
  → Crear EmailVerificationToken
  → Enviar email de verificación
  → Retornar usuario creado (UserResponse — sin password)
```

### 12.2 Login

```
Cliente → POST /api/v1/auth/login { email, password }
  → Buscar usuario por email (UserRepository)
  → Verificar password contra hash (BCryptPasswordEncoder.matches)
  → Verificar is_email_verified = true
  → Verificar is_active = true
  → Generar access_token (15 min) + refresh_token (7 días) via JwtTokenProvider
  → Retornar TokenResponse { access_token, refresh_token, token_type: "Bearer" }
```

### 12.3 Cambio de contraseña (usuario autenticado)

```
Cliente → POST /api/v1/auth/change-password { current_password, new_password }
  → (Requiere Authorization: Bearer <access_token>)
  → JwtAuthFilter verifica el token y carga el usuario en SecurityContext
  → Verificar current_password contra hash
  → Hashear new_password
  → Actualizar en BD + actualizar updated_at
  → Retornar confirmación
```

### 12.4 Recuperación de contraseña (forgot + reset)

```
Paso 1: Solicitar recuperación
Cliente → POST /api/v1/auth/forgot-password { email }
  → Buscar usuario por email
  → Crear PasswordResetToken (UUID + expires_at = now + 1h)
  → Enviar email con enlace: {FRONTEND_URL}/reset-password?token={token}
  → Retornar mensaje genérico (no revelar si el email existe)

Paso 2: Restablecer contraseña
Cliente → POST /api/v1/auth/reset-password { token, new_password }
  → Buscar token en BD
  → Verificar que no haya expirado ni sido usado
  → Hashear new_password
  → Actualizar password del usuario
  → Marcar token como used = true
  → Retornar confirmación
```

---

## 13. Configuración de Docker Compose

Servicios del proyecto:

```yaml
# Servicios en docker-compose.yml
services:
  db: # PostgreSQL 17 — puerto 5432
  be: # Spring Boot — puerto 8080
  fe: # React + Nginx — puerto 3000
  mailpit: # Captura SMTP — puerto 8025 (UI) + 1025 (SMTP)
```

> **Diferencia con `proyecto-be-fe`**: El backend de referencia corre en el puerto `8000` (FastAPI/Uvicorn).
> Este proyecto corre en el puerto `8080` (Spring Boot/Tomcat embebido), que es el puerto por defecto de Spring Boot.

---

## 14. Comparativa de Implementación — `proyecto-be-fe` vs Este Proyecto

Esta tabla es clave para entender las diferencias entre los dos proyectos:

| Concepto                   | `proyecto-be-fe` (Python)                 | Este proyecto (Kotlin)                             |
| -------------------------- | ----------------------------------------- | -------------------------------------------------- |
| Punto de entrada           | `app/main.py` — `FastAPI()`               | `NnAuthApplication.kt` — `@SpringBootApplication`  |
| Configuración              | `config.py` — Pydantic Settings           | `AppProperties.kt` — `@ConfigurationProperties`    |
| Conexión a BD              | `database.py` — SQLAlchemy Engine+Session | `application.yml` — Spring Boot DataSource auto    |
| Modelos                    | `models/user.py` — SQLAlchemy ORM         | `model/User.kt` — `@Entity` JPA                    |
| Schemas (request/response) | `schemas/user.py` — Pydantic BaseModel    | `dto/AuthDtos.kt` — data class + `@Valid`          |
| Endpoints                  | `routers/auth.py` — `@router.post(...)`   | `controller/AuthController.kt` — `@RestController` |
| Lógica de negocio          | `services/auth_service.py`                | `service/AuthService.kt` — `@Service`              |
| Hashing                    | `passlib[bcrypt]` — `CryptContext`        | `BCryptPasswordEncoder` (Spring Security)          |
| JWT                        | `python-jose`                             | `JJWT` (java-jwt)                                  |
| Validación de entrada      | Pydantic — automática en el endpoint      | Bean Validation (`@Valid`, `@NotBlank`, etc.)      |
| Rate limiting              | `slowapi` — `@limiter.limit("10/min")`    | `Bucket4j` — filter o interceptor                  |
| Inyección de dependencias  | FastAPI `Depends()`                       | Spring IoC — `@Autowired` / constructor injection  |
| Migraciones                | Alembic — `alembic upgrade head`          | Flyway — automático al arrancar                    |
| Seguridad HTTP             | Middleware manual en `main.py`            | Spring Security `SecurityFilterChain`              |
| CORS                       | `CORSMiddleware` en FastAPI               | `CorsConfigurationSource` en Spring Security       |
| Tests                      | pytest + httpx `AsyncClient`              | JUnit 5 + MockMvc                                  |
| Cobertura                  | pytest-cov                                | JaCoCo                                             |
| Servidor                   | Uvicorn (ASGI)                            | Embedded Tomcat (Servlet)                          |
| Docs auto                  | Swagger en `/docs` (FastAPI nativo)       | SpringDoc OpenAPI en `/swagger-ui.html`            |

---

## 15. Diseño y UX/UI — OBLIGATORIO

### 15.1 Reglas visuales generales

| Aspecto        | Regla                                                              |
| -------------- | ------------------------------------------------------------------ |
| Temas          | Dark mode y Light mode con toggle — clase `.dark` en `<html>`      |
| Tipografía     | Fuentes sans-serif exclusivamente (Inter, system-ui)               |
| Colores        | Sólidos y planos — **SIN degradados** (`gradient`) en ningún lugar |
| Estilo visual  | Moderno, limpio, minimalista con excelente UX/UI                   |
| Botones acción | Siempre alineados a la derecha (`justify-end`, `text-right`)       |
| Spacing        | Escala consistente de Tailwind (p-4, gap-6, space-y-4)             |
| Responsividad  | Mobile-first — formularios de auth visibles desde 320px            |
| Accesibilidad  | Labels en inputs, `aria-*` básicos, contraste suficiente (WCAG AA) |

### 15.2 Sistema de Temas Diferenciales por Stack — OBLIGATORIO

Este proyecto pertenece a una **serie educativa** donde cada variante del sistema
tiene un color de acento único que identifica su stack visualmente.
Los componentes React **NUNCA** usan colores hardcodeados — siempre el token `accent-*`.

#### Identidad visual — Spring Boot Kotlin → **fuchsia**

| Stack                    | Proyecto                | Acento Tailwind | Hex           |
| ------------------------ | ----------------------- | --------------- | ------------- |
| FastAPI (Python)         | `proyecto-be-fe`        | `emerald`       | `#059669`     |
| Express.js (Node)        | `proyecto-beex-fe`      | `blue`          | `#2563eb`     |
| Next.js (fullstack)      | `proyecto-be-fe-next`   | `violet`        | `#7c3aed`     |
| Spring Boot (Java)       | `proyecto-besb-fe`      | `amber`         | `#d97706`     |
| **Spring Boot (Kotlin)** | **`proyecto-besbk-fe`** | **`fuchsia`**   | **`#c026d3`** |
| Go REST API              | `proyecto-bego-fe`      | `cyan`          | `#0891b2`     |

#### Reglas del token `accent-*`

```tsx
// ✅ CORRECTO — usar siempre el token accent-*
<button className="bg-accent-600 hover:bg-accent-700 text-white">
  Iniciar sesión
</button>

// ❌ NUNCA hardcodear el color del stack en los componentes
<button className="bg-fuchsia-600 hover:bg-fuchsia-700 text-white">
  Iniciar sesión
</button>
```

#### Configuración en `fe/src/index.css`

```css
/* Spring Boot Kotlin → fuchsia */
@theme inline {
  --color-accent-50:  var(--color-fuchsia-50);
  --color-accent-100: var(--color-fuchsia-100);
  --color-accent-200: var(--color-fuchsia-200);
  --color-accent-300: var(--color-fuchsia-300);
  --color-accent-400: var(--color-fuchsia-400);
  --color-accent-500: var(--color-fuchsia-500);
  --color-accent-600: var(--color-fuchsia-600); /* botones primarios */
  --color-accent-700: var(--color-fuchsia-700); /* hover */
  --color-accent-800: var(--color-fuchsia-800);
  --color-accent-900: var(--color-fuchsia-900);
  --color-accent-950: var(--color-fuchsia-950);
}
```

#### Colores del logo SVG — Fuchsia

| Elemento             | Atributo SVG          | Color       |
| -------------------- | --------------------- | ----------- |
| Borde del badge      | `stroke="#c026d3"`    | fuchsia-600 |
| Trazos de la letra   | `stroke="#e879f9"`    | fuchsia-400 |

> Ver `_docs/referencia-tecnica/design-system.md` para el sistema completo de tokens,
> instrucciones de clonación y verificación visual.

---

## 16. Reglas para Copilot / IA — Al Generar Código

1. **Dividir respuestas largas** — Si la implementación es extensa, dividirla en pasos incrementales.
2. **Código generado = código probado** — Siempre incluir o sugerir tests para lo que se genere.
3. **Comentarios pedagógicos** — Cada bloque significativo debe tener comentarios con ¿Qué? ¿Para qué? ¿Impacto?
4. **Tipos explícitos obligatorios** — Nunca omitir tipado en Kotlin ni en TypeScript.
5. **Gradle Wrapper** — Siempre usar `./gradlew`, nunca `gradle` directamente.
6. **Variables de entorno** — Toda configuración sensible va en `.env`, nunca hardcodeada.
7. **Conventional Commits** — Sugerir mensajes de commit con formato correcto.
8. **Seguridad primero** — Nunca almacenar passwords en texto plano, nunca exponer secrets.
9. **Legibilidad sobre cleverness** — El código debe ser entendible para un aprendiz.
10. **Comparar con referencia** — Cuando sea relevante, señalar cómo se haría en `proyecto-be-fe` para reforzar el aprendizaje.

---

## 17. Plan de Trabajo — Fases

> Cada fase es independiente y verificable. No avanzar a la siguiente sin completar y probar la actual.

### Fase 0 — Fundamentos y Configuración Base

- [ ] Crear `.github/copilot-instructions.md` (este archivo)
- [ ] Crear `.gitignore` raíz
- [ ] Crear `docker-compose.yml` con todos los servicios
- [ ] Crear `README.md` con descripción, stack, prerrequisitos y setup

### Fase 1 — Backend Setup

- [ ] Inicializar proyecto Spring Boot con Gradle Kotlin DSL en `be/`
- [ ] Configurar `build.gradle.kts` con todas las dependencias
- [ ] Crear `application.yml` con configuración base
- [ ] Crear `AppProperties.kt` — `@ConfigurationProperties` para leer `.env`
- [ ] Crear `.env.example` y `.env`
- [ ] ✅ Verificar: `./gradlew bootRun` → Swagger UI en `/swagger-ui.html`

### Fase 2 — Modelo de Datos y Migraciones

- [ ] Crear entidades JPA: `User.kt`, `PasswordResetToken.kt`, `EmailVerificationToken.kt`
- [ ] Crear repositorios: `UserRepository.kt`, etc.
- [ ] Crear scripts Flyway: `V1__`, `V2__`, `V3__`
- [ ] ✅ Verificar: tablas creadas en PostgreSQL al arrancar

### Fase 3 — Seguridad y JWT

- [ ] Crear `JwtTokenProvider.kt` — generación y verificación de JWT
- [ ] Crear `JwtAuthFilter.kt` — filtro HTTP que valida el token en cada request
- [ ] Crear `UserDetailsServiceImpl.kt` — carga usuario desde BD
- [ ] Crear `SecurityConfig.kt` — configura Spring Security y CORS
- [ ] ✅ Verificar: endpoint sin token → 401; con token válido → 200

### Fase 4 — Autenticación Backend

- [ ] Crear DTOs en `AuthDtos.kt` con Bean Validation
- [ ] Crear `AuthService.kt` — toda la lógica de negocio
- [ ] Crear `AuthController.kt` — todos los endpoints
- [ ] Crear `UserController.kt` — GET /me
- [ ] Crear `EmailService.kt` — envío de emails
- [ ] Crear `AuditLogService.kt` — logging de eventos de seguridad
- [ ] Implementar rate limiting con Bucket4j en endpoints sensibles
- [ ] ✅ Verificar: probar todos los endpoints en Swagger UI

### Fase 5 — Tests Backend

- [ ] Crear `AuthControllerTest.kt` — tests de integración con MockMvc
- [ ] Crear `AuthServiceTest.kt` — tests unitarios
- [ ] ✅ Verificar: `./gradlew test` → todos los tests pasan (cobertura ≥80%)

### Fase 6 — Frontend Setup

- [ ] Inicializar proyecto Vite con React + TypeScript en `fe/`
- [ ] Instalar dependencias con `pnpm`
- [ ] Configurar TailwindCSS
- [ ] Configurar TypeScript strict mode
- [ ] Crear `.env.example`
- [ ] ✅ Verificar: `pnpm dev` → app base visible en `http://localhost:5173`

### Fase 7 — Frontend Auth

- [ ] Crear tipos TypeScript (`types/auth.ts`)
- [ ] Crear cliente HTTP (`api/auth.ts`) apuntando al backend Kotlin (puerto 8080)
- [ ] Crear `AuthContext.tsx` + Provider
- [ ] Crear hook `useAuth`
- [ ] Crear componentes UI (InputField, Button, Alert)
- [ ] Crear ProtectedRoute
- [ ] Crear páginas: Login, Register, Dashboard, ChangePassword, ForgotPassword, ResetPassword
- [ ] Crear LandingPage, páginas legales y formulario de contacto
- [ ] ✅ Verificar: flujo completo funciona contra la API Kotlin

### Fase 8 — Tests Frontend

- [ ] Configurar Vitest + Testing Library
- [ ] Crear tests para componentes y flujos de auth
- [ ] ✅ Verificar: `pnpm test` → todos los tests pasan

### Fase 9 — Docker y Dockerfiles

- [ ] Crear `be/Dockerfile` (multi-stage: build con Gradle → run con JRE 21)
- [ ] Crear `fe/Dockerfile` (multi-stage: build con Node → serve con Nginx)
- [ ] Verificar `docker-compose.yml` completo
- [ ] ✅ Verificar: `docker compose up --build` levanta todo correctamente

### Fase 10 — Internacionalización (i18n)

- [ ] Crear `HU-012` — Historia de usuario: cambio de idioma de la interfaz
- [ ] Crear `RF-014` — Requisito funcional: i18n ES/EN
- [ ] Backend: agregar columna `locale VARCHAR(10) DEFAULT 'es'` en tabla `users` (Flyway `V4__`)
- [ ] Backend: actualizar `User.kt` con campo `locale`
- [ ] Backend: agregar `locale` en `UserResponse` DTO + nuevo `UpdateLocaleRequest`
- [ ] Backend: agregar endpoint `PATCH /api/v1/users/me/locale`
- [ ] Backend: agregar método `updateUserLocale` en `AuthService.kt`
- [ ] Backend: agregar test de integración para el endpoint de locale
- [ ] Frontend: instalar `react-i18next@X.Y.Z`, `i18next@X.Y.Z`, `i18next-browser-languagedetector@X.Y.Z`
- [ ] Frontend: crear `src/locales/es/translation.json` — catálogo español
- [ ] Frontend: crear `src/locales/en/translation.json` — catálogo inglés
- [ ] Frontend: crear `src/i18n.ts` — configuración de i18next
- [ ] Frontend: importar i18n en `main.tsx`
- [ ] Frontend: crear componente `LanguageSwitcher`
- [ ] Frontend: integrar `LanguageSwitcher` en `Navbar`
- [ ] Frontend: adaptar todas las páginas al hook `useTranslation()`
- [ ] Frontend: sincronizar preferencia de idioma con backend (usuario autenticado)
- [ ] Frontend: agregar mock de i18n en `setup.ts` para tests
- [ ] Frontend: crear test para `LanguageSwitcher`
- [ ] ✅ Verificar: la app cambia de idioma completamente al seleccionar ES/EN

### Fase 11 — Documentación Final

- [ ] Revisar y completar `_docs/referencia-tecnica/architecture.md`
- [ ] Revisar y completar `_docs/referencia-tecnica/api-endpoints.md`
- [ ] Revisar y completar `_docs/referencia-tecnica/database-schema.md`
- [ ] Actualizar `README.md` con instrucciones finales verificadas
- [ ] ✅ Verificar: documentación completa y coherente

---

## 18. Verificación Final del Sistema

```bash
# 1. Levantar todos los servicios
docker compose up -d

# 2. Verificar que todos están healthy
docker compose ps

# 3. Ejecutar tests backend
cd be && ./gradlew test

# 4. Ejecutar tests frontend
cd fe && pnpm test

# 5. Flujo manual completo (en http://localhost:3000):
#    Registro → Verificar email (ver en Mailpit:8025) → Login
#    → Ver perfil → Cambiar contraseña → Logout
#    → Forgot password → Reset password → Login con nueva contraseña
#    → Cambiar idioma (ES↔EN) → Verificar que toda la interfaz cambia
```

---

> **Recuerda**: La calidad no es una opción, es una obligación.
> Cada línea de código es una oportunidad de aprender y enseñar.
