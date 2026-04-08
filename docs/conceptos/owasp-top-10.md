<!--
  ¿Qué? Análisis de las 10 vulnerabilidades más críticas de seguridad web (OWASP Top 10)
         aplicado específicamente al sistema NN Auth en Kotlin + Spring Boot.
  ¿Para qué? Demostrar cómo cada riesgo de seguridad está mitigado (o debe serlo) en este proyecto,
             con ejemplos concretos del stack Kotlin/Spring.
  ¿Impacto? Un sistema de autenticación que no sigue estas prácticas puede comprometer
             los datos de todos sus usuarios.
-->

> **Referencia oficial**: [OWASP Top 10 (2021)](https://owasp.org/Top10/)
>
> **Diferencia con `proyecto-be-fe`**: Las prácticas de seguridad son **conceptualmente idénticas**.
> La diferencia está en las **herramientas usadas** para implementarlas.
> Python/FastAPI → Kotlin/Spring Boot.

---

# OWASP Top 10 — NN Auth System (Kotlin Edition)

El OWASP Top 10 es una lista de las 10 vulnerabilidades de seguridad más críticas en aplicaciones web, publicada por el [Open Web Application Security Project](https://owasp.org).

Este documento analiza cómo cada vulnerabilidad **se mitiga en este proyecto** con ejemplos de código en Kotlin/Spring Boot.

---

## A01 — Broken Access Control (Control de Acceso Roto)

**Riesgo**: Usuarios que acceden a recursos o acciones que no deberían poder realizar.
Ejemplo: Un usuario normal que puede ver el perfil de otro usuario, o que puede
acceder a datos de admin.

### ¿Cómo lo mitigamos?

**Spring Security** gestiona el control de acceso mediante el `SecurityFilterChain`:

```kotlin
// SecurityConfig.kt
@Bean
fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
    http
        .authorizeHttpRequests { auth ->
            // ¿Qué? Define qué endpoints son públicos y cuáles requieren autenticación.
            // ¿Para qué? Evitar que usuarios no autenticados accedan a datos protegidos.
            // ¿Impacto? Sin esto, cualquiera podría ver /api/v1/users/me sin autenticarse.
            auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()  // Todo lo demás requiere auth
        }
    return http.build()
}
```

**Los UUIDs como identificadores** también protegen contra la enumeración:
- ❌ `/api/v1/users/1` → fácil de explorar: prueba 1, 2, 3...
- ✅ `/api/v1/users/550e8400-e29b-41d4-a716-446655440000` → imposible de adivinar

**El endpoint `/me`** garantiza que cada usuario solo vea sus propios datos:
```kotlin
// UserController.kt — el usuario solo puede ver SU perfil, no el de otros
@GetMapping("/me")
fun getCurrentUser(authentication: Authentication): UserResponse {
    val email = authentication.name  // Extraído del JWT — no del request body
    return userService.findByEmail(email)
}
```

---

## A02 — Cryptographic Failures (Fallos Criptográficos)

**Riesgo**: Datos sensibles expuestos por uso de cifrado débil, ausente o incorrecto.
Ejemplos: Contraseñas en texto plano, tokens predecibles, HTTP en lugar de HTTPS.

### ¿Cómo lo mitigamos?

#### 1. BCrypt para contraseñas

```kotlin
// AuthService.kt
@Service
class AuthService(
    private val passwordEncoder: BCryptPasswordEncoder
) {
    // ¿Qué? Hashea la contraseña antes de almacenarla.
    // ¿Para qué? Si la BD es comprometida, las contraseñas NUNCA se exponen en texto plano.
    // ¿Impacto? BCrypt usa salt automático y es resistente a ataques de fuerza bruta.
    fun register(request: RegisterRequest): UserResponse {
        val hashedPassword = passwordEncoder.encode(request.password)
        // ... guardar usuario con hashedPassword
    }
}
```

**¿Por qué BCrypt?**
- Genera un **salt aleatorio** automáticamente — dos hashes del mismo password siempre son diferentes
- El factor de costo configurado (por defecto: 10 rounds) lo hace **computacionalmente caro** de romper
- Es el estándar de la industria para hashing de contraseñas

#### 2. JWT con HS256 y clave segura

```kotlin
// JwtTokenProvider.kt
// La clave secreta mínimo 32 caracteres, almacenada en .env — NUNCA hardcodeada
@Value("\${app.jwt.secret}")
private lateinit var jwtSecret: String
```

#### 3. HTTPS en producción

Configurar en `application.yml` de producción o via proxy inverso (Nginx).

> **Comparativa con `proyecto-be-fe`**: Python usa `passlib[bcrypt]` con `CryptContext`.
> Kotlin usa `BCryptPasswordEncoder` de Spring Security. Mismo algoritmo BCrypt, diferente librería.

---

## A03 — Injection (Inyección)

**Riesgo**: Datos no confiables enviados a un intérprete (SQL, Shell, LDAP).
El más clásico: SQL Injection, donde un atacante puede robar o destruir la BD.

Ejemplo de ataque:
```
email: admin@ejemplo.com' OR '1'='1
```

### ¿Cómo lo mitigamos?

**Spring Data JPA** previene SQL Injection automáticamente mediante **consultas parametrizadas**.
Nunca construimos SQL concatenando strings:

```kotlin
// UserRepository.kt
@Repository
interface UserRepository : JpaRepository<User, UUID> {
    // ¿Qué? Spring Data genera automáticamente SQL parametrizado para este método.
    // ¿Para qué? Los parámetros nunca se insertan directamente en el SQL — se escapan.
    // ¿Impacto? Imposible hacer SQL Injection a través de este método.
    fun findByEmail(email: String): Optional<User>

    // Equivale al SQL parametrizado:
    // SELECT * FROM users WHERE email = ? (parámetro vinculado, no concatenado)
}
```

```kotlin
// ❌ NUNCA hacer esto:
entityManager.createNativeQuery("SELECT * FROM users WHERE email = '$email'")

// ✅ Spring Data JPA siempre usa esto internamente:
entityManager.createQuery("SELECT u FROM User u WHERE u.email = :email")
    .setParameter("email", email)
```

**Bean Validation** también filtra entradas antes de procesarlas:
```kotlin
data class LoginRequest(
    @field:Email(message = "Formato de email inválido")
    @field:NotBlank
    val email: String,

    @field:NotBlank
    val password: String
)
```

---

## A04 — Insecure Design (Diseño Inseguro)

**Riesgo**: Problemas arquitécturales donde la seguridad no fue considerada desde el inicio.

### Decisiones de diseño seguro en este proyecto

#### 1. Enumeración de usuarios — prevenida por diseño

El endpoint de login devuelve el **mismo error** si el email no existe o si la contraseña es incorrecta:

```kotlin
// AuthService.kt
fun login(request: LoginRequest): TokenResponse {
    // ¿Qué? Buscar usuario y verificar contraseña, retornando el mismo error genérico.
    // ¿Para qué? Un atacante no puede saber si un email está registrado o no.
    // ¿Impacto? Sin esto, podría enumerar todos los emails registrados.
    val user = userRepository.findByEmail(request.email)
        .orElseThrow { UnauthorizedException("Credenciales inválidas") }

    if (!passwordEncoder.matches(request.password, user.hashedPassword)) {
        throw UnauthorizedException("Credenciales inválidas")  // Mismo mensaje
    }
    // ...
}
```

También aplicado en `forgot-password` — siempre devuelve el mismo mensaje.

#### 2. Un solo uso para tokens temporales

Los tokens de reset y verificación tienen `used = false` por defecto y se marcan
como `used = true` al usarse. No pueden reutilizarse.

#### 3. Verificación de email obligatoria antes de login

Un usuario que no verificó su email recibe `403 Forbidden`, no `401 Unauthorized`.
Esto distingue "no autenticado" de "no autorizado por cuenta inactiva".

---

## A05 — Security Misconfiguration (Configuración de Seguridad Incorrecta)

**Riesgo**: Configuraciones predeterminadas inseguras, Swagger expuesto en producción,
cabeceras HTTP faltantes, CORS abierto.

### ¿Cómo lo mitigamos?

#### 1. Spring Security headers por defecto

Spring Security agrega automáticamente cabeceras de seguridad HTTP:
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0 (moderno — delega en CSP)
Strict-Transport-Security: max-age=31536000 (en HTTPS)
```

#### 2. CORS configurado explícitamente

```kotlin
// SecurityConfig.kt
@Bean
fun corsConfigurationSource(): CorsConfigurationSource {
    val config = CorsConfiguration()
    // ¿Qué? Permite solo el origen del frontend, no cualquier origen.
    // ¿Para qué? Prevenir ataques CSRF y accesos cross-origin no autorizados.
    // ¿Impacto? Sin esto, cualquier web podría hacer requests al backend del usuario.
    config.allowedOrigins = listOf(frontendUrl)  // Variable de entorno — no hardcodeada
    config.allowedMethods = listOf("GET", "POST", "PUT", "DELETE", "OPTIONS")
    config.allowedHeaders = listOf("Authorization", "Content-Type")
    // ...
}
```

#### 3. Swagger deshabilitado en producción

```yaml
# application.yml
springdoc:
  swagger-ui:
    enabled: ${SWAGGER_ENABLED:true}  # false en producción via .env
```

#### 4. Variables de entorno para toda configuración sensible

```yaml
# application.yml — NUNCA credenciales hardcodeadas
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
app:
  jwt:
    secret: ${JWT_SECRET}
```

---

## A06 — Vulnerable and Outdated Components (Componentes Vulnerables y Desactualizados)

**Riesgo**: Usar librerías con vulnerabilidades de seguridad conocidas (CVEs).

### ¿Cómo lo mitigamos?

#### 1. Versiones LTS y actualizadas

- JDK 21 LTS (Long-Term Support) — soporte hasta 2031
- Spring Boot 3.x — recibe parches de seguridad regulares
- Gradle Kotlin DSL — gestión centralizada de dependencias

#### 2. Monitoreo de vulnerabilidades

```bash
# Verificar vulnerabilidades conocidas en dependencias
./gradlew dependencyCheckAnalyze
```

Se puede integrar con [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)
para automatizar esto en CI/CD.

#### 3. Gradle Wrapper — versiones reproducibles

```bash
# El wrapper garantiza que todos usen la misma versión de Gradle
./gradlew --version   # Siempre la versión del proyecto, no la instalada globalmente
```

> **Comparativa con `proyecto-be-fe`**: Python usa `pip` con `requirements.txt`. El equivalente
> de Gradle en Python sería `pip-audit` para auditar vulnerabilidades.

---

## A07 — Identification and Authentication Failures (Fallos de Identificación y Autenticación)

**Riesgo**: Autenticación débil — tokens predecibles, contraseñas simples permitidas,
sin límite de intentos.

### ¿Cómo lo mitigamos?

#### 1. Validación de fortaleza de contraseña — Bean Validation

```kotlin
// AuthDtos.kt
data class RegisterRequest(
    @field:Email
    @field:NotBlank
    val email: String,

    @field:NotBlank
    val fullName: String,

    // ¿Qué? Valida fortaleza mínima de contraseña con regex.
    // ¿Para qué? Prevenir contraseñas triviales como "123456" o "password".
    // ¿Impacto? Contraseñas débiles son el principal vector de compromiso de cuentas.
    @field:Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).{8,}$",
        message = "La contraseña debe tener al menos 8 caracteres, una mayúscula, una minúscula y un número"
    )
    val password: String
)
```

#### 2. Rate limiting con Bucket4j — anti brute force

```kotlin
// RateLimitFilter.kt o interceptor
// ¿Qué? Limita intentos de login a 10 por minuto por IP.
// ¿Para qué? Hacer inviable un ataque de fuerza bruta de contraseñas.
// ¿Impacto? Sin rate limiting, un atacante puede probar millones de contraseñas.
val bucket = Bucket.builder()
    .addLimit(Bandwidth.classic(10, Refill.greedy(10, Duration.ofMinutes(1))))
    .build()
```

#### 3. JWT de corta duración

- **Access Token**: 15 minutos — si es robado, el daño es limitado
- **Refresh Token**: 7 días — permite sesiones largas sin comprometer seguridad

#### 4. Rotación de tokens al renovar

Al hacer `POST /auth/refresh`, se generan **nuevos** access y refresh tokens.
El refresh token anterior queda sin validez implícitamente (stateless).

---

## A08 — Software and Data Integrity Failures (Fallos de Integridad de Software y Datos)

**Riesgo**: Código o datos que se asumen íntegros sin verificación.
Ejemplos: Dependencias sin checksum, pipelines CI/CD sin verificación.

### ¿Cómo lo mitigamos?

#### 1. Gradle Wrapper con checksum

El archivo `gradle/wrapper/gradle-wrapper.properties` incluye el hash SHA-256 del Gradle Wrapper,
lo que garantiza que el wrapper no ha sido manipulado:

```properties
distributionSha256Sum=<hash-sha256-del-gradle>
```

#### 2. Flyway checksum en migraciones

Flyway almacena un checksum de cada script SQL aplicado. Si alguien modifica un script
ya ejecutado, Flyway falla al arrancar:

```
ERROR: Validate failed: Migration V1__create_users_table.sql has changed
since it was applied! Please revert the changes or repair the schema history.
```

Esto garantiza la **integridad del esquema de BD** a lo largo del ciclo de vida del proyecto.

#### 3. Refresh tokens stateless — sin manipulación del lado cliente

Los tokens JWT están firmados con `HMAC-SHA256`. Si alguien modifica el payload del token,
la firma no coincidirá y `JwtTokenProvider.validateToken()` rechazará el token.

---

## A09 — Security Logging and Monitoring Failures (Fallos de Registro y Monitoreo)

**Riesgo**: No registrar eventos de seguridad críticos, lo que impide detectar ataques.

### ¿Cómo lo mitigamos?

#### `AuditLogService.kt` — Registro de eventos de seguridad

```kotlin
// AuditLogService.kt
@Service
class AuditLogService(private val log: Logger = LoggerFactory.getLogger(AuditLogService::class.java)) {

    // ¿Qué? Registra eventos de seguridad en el log de la aplicación.
    // ¿Para qué? Detectar intentos de acceso no autorizado, patrones de ataque.
    // ¿Impacto? Sin logs, un ataque de brute force pasaría completamente desapercibido.

    fun logSuccessfulLogin(email: String, ipAddress: String) {
        log.info("[SECURITY] LOGIN_SUCCESS | email={} | ip={}", email, ipAddress)
    }

    fun logFailedLogin(email: String, ipAddress: String) {
        log.warn("[SECURITY] LOGIN_FAILED | email={} | ip={}", email, ipAddress)
    }

    fun logPasswordReset(email: String) {
        log.info("[SECURITY] PASSWORD_RESET | email={}", email)
    }

    fun logPasswordChange(userId: UUID) {
        log.info("[SECURITY] PASSWORD_CHANGE | userId={}", userId)
    }

    fun logRateLimitExceeded(endpoint: String, ipAddress: String) {
        log.warn("[SECURITY] RATE_LIMIT_EXCEEDED | endpoint={} | ip={}", endpoint, ipAddress)
    }
}
```

> ⚠️ **Nunca loggear contraseñas ni tokens JWT completos.** Solo loggear metadatos
> (email, IP, endpoint, timestamp).

**Eventos que se deben registrar:**
- Logins exitosos y fallidos (con IP)
- Cambios de contraseña
- Resets de contraseña
- Rate limits superados
- Intento de usar tokens ya expirados o usados

---

## A10 — Server-Side Request Forgery — SSRF

**Riesgo**: El servidor hace requests a URLs controladas por el atacante,
permitiéndole acceder a recursos internos (metadata de cloud, servicios internos).

### Estado en este proyecto

Este sistema **no tiene endpoints que acepten URLs del usuario** para hacer requests,
por lo que el riesgo de SSRF es mínimo.

Las URLs que el sistema maneja son:
1. `FRONTEND_URL` — configurada en `.env`, no controlada por el usuario
2. `SMTP_HOST` — configurada en `.env`, no controlada por el usuario

### Buenas prácticas aplicadas

#### 1. URLs de configuración solo desde variables de entorno

```yaml
# application.yml — Las URLs vienen del .env del servidor, no del request
app:
  frontend-url: ${FRONTEND_URL:http://localhost:5173}
  smtp:
    host: ${SMTP_HOST:localhost}
    port: ${SMTP_PORT:1025}
```

#### 2. Si en el futuro se aceptan URLs externas...

Si se agrega funcionalidad que requiera hacer requests a URLs externas
(como imágenes de perfil o webhooks), se debe:

```kotlin
// Validar que la URL sea de un dominio permitido (allowlist)
private val allowedHosts = setOf("storage.example.com", "cdn.example.com")

fun validateUrl(url: String): Boolean {
    val parsedUrl = URL(url)
    return parsedUrl.host in allowedHosts
}
```

---

## Resumen — Tabla de Mitigaciones

| OWASP | Riesgo | Herramienta (Kotlin) | Herramienta (Python/referencia) |
|---|---|---|---|
| A01 — Access Control | Acceso no autorizado | `Spring Security` — `SecurityFilterChain` | FastAPI `Depends` + middleware |
| A02 — Crypto Failures | Contraseñas expuestas | `BCryptPasswordEncoder` (Spring Security) | `passlib[bcrypt]` + `CryptContext` |
| A03 — Injection | SQL Injection | `Spring Data JPA` — queries parametrizados | `SQLAlchemy ORM` — queries parametrizados |
| A04 — Insecure Design | Enumeración de usuarios | Mensajes de error genéricos en `AuthService` | Mensajes de error genéricos en `auth_service.py` |
| A05 — Misconfiguration | CORS abierto, Swagger expuesto | `CorsConfigurationSource` + Swagger condicional | `CORSMiddleware` + Swagger condicional |
| A06 — Outdated Deps | Vulnerabilidades conocidas | JDK 21 LTS + Gradle Dependency Check | Python 3.12 + `pip audit` |
| A07 — Auth Failures | Contraseñas débiles, brute force | `@Pattern` (Bean Validation) + `Bucket4j` | Pydantic validators + `slowapi` |
| A08 — Integrity | Scripts modificados | Flyway checksums + Gradle Wrapper SHA256 | Alembic checksums |
| A09 — Logging | Sin auditoría de seguridad | `AuditLogService.kt` + `Logger` de SLF4J | Python `logging` + auditoría manual |
| A10 — SSRF | Requests a URLs maliciosas | URLs solo desde `.env` (no del request) | URLs solo desde `.env` (no del request) |

---

## Recursos de Aprendizaje

- [OWASP Top 10 Oficial (2021)](https://owasp.org/Top10/)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/index.html)
- [JJWT Reference](https://github.com/jwtk/jjwt)
- [Bucket4j Documentation](https://bucket4j.com/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
