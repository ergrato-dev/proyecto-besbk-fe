<!--
  Archivo: patrones-arquitectonicos.md
  Descripción: Documentación técnica de los patrones arquitectónicos aplicados
               en el proyecto NN Auth System — Kotlin Edition.
  ¿Para qué? Servir como referencia de estudio y consulta para entender por qué
             el sistema está estructurado como lo está.
  ¿Impacto? Comprender los patrones facilita mantener, extender y defender
            decisiones técnicas del proyecto ante evaluaciones o presentaciones.
-->

> **Proyecto:** NN Auth System — Kotlin Edition
> **Stack:** Spring Boot Kotlin + React + PostgreSQL + Docker
> **Tests:** JUnit 5 + MockMvc (backend) · Vitest + Testing Library (frontend)

---

# Patrones Arquitectónicos — NN Auth System

## Resumen ejecutivo

El sistema aplica **10 patrones arquitectónicos y de diseño** de uso profesional.
No son solo teoría: cada patrón resuelve un problema concreto y está presente en
el código del proyecto.

| #   | Patrón                    | Dónde vive                                 | Qué resuelve                                        |
| --- | ------------------------- | ------------------------------------------ | --------------------------------------------------- |
| 1   | Arquitectura en Capas     | `be/src/main/kotlin/com/nn/auth/`          | Separación de responsabilidades en el backend       |
| 2   | DTO — Data Transfer Object| `dto/AuthDtos.kt`                          | Nunca exponer datos internos de BD en respuestas    |
| 3   | Inyección de Dependencias | Constructor injection + `@Service`         | Desacoplar servicios transversales (BD, auth)       |
| 4   | JWT Stateless             | `security/JwtTokenProvider.kt`             | Autenticación sin estado en el servidor             |
| 5   | Context / Provider        | `AuthContext.tsx`                          | Estado de auth global en toda la app React          |
| 6   | Custom Hook               | `useAuth.ts`                               | Encapsular y reutilizar lógica de autenticación     |
| 7   | Interceptor               | `api/axios.ts`                             | Adjuntar token JWT en cada petición automáticamente |
| 8   | SPA + Route Guard         | `ProtectedRoute.tsx`                       | Proteger rutas sin renderizar páginas no autorizadas|
| 9   | Monorepo                  | `be/` + `fe/`                              | Código fuente unificado en un solo repositorio      |
| 10  | REST API                  | `controller/AuthController.kt`             | Interfaz estándar entre frontend y backend          |

---

## Vista general del sistema

El sistema sigue una **arquitectura Cliente–Servidor** de tres capas lógicas:

1. **Frontend (React)** — Interfaz de usuario. Nunca guarda estado en el servidor.
2. **Backend (Spring Boot)** — Lógica de negocio. Expone una API REST bajo `/api/v1/`.
3. **Base de datos (PostgreSQL)** — Persistencia. Solo accedida desde el backend.

La comunicación entre frontend y backend es exclusivamente **HTTP + JSON**. Los
tokens JWT viajan en el header `Authorization: Bearer <token>`. Nunca hay sesiones
en el servidor.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Navegador (React SPA)                                                  │
│  ┌────────────┐  HTTP + JSON   ┌─────────────────────────────────────┐  │
│  │  React App │ ─────────────→ │  Spring Boot (puerto 8080)          │  │
│  │  (Vite)    │ ←───────────── │  /api/v1/auth/** + /api/v1/users/** │  │
│  └────────────┘  JWT Bearer    └───────────────────┬─────────────────┘  │
│   puerto 5173                                      │ JPA/Hibernate       │
│                               ┌────────────────────▼──────────────────┐  │
│                               │  PostgreSQL 17 (puerto 5432)           │  │
│                               │  Tablas: users, password_reset_tokens, │  │
│                               │          email_verification_tokens     │  │
│                               └───────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Patrón 1 — Arquitectura en Capas

### ¿Qué es?

Organizar el código en capas horizontales donde **cada capa solo puede comunicarse
con la capa directamente inferior**.

### ¿Cómo se aplica aquí?

```
HTTP Request
      ↓
┌─────────────────────────────────────────────┐
│  controller/         → Capa HTTP            │  Recibe y devuelve HTTP
├─────────────────────────────────────────────┤
│  service/            → Capa de Negocio      │  Reglas y decisiones
├─────────────────────────────────────────────┤
│  repository/ + dto/  → Capa de Datos        │  JPA + Validación
├─────────────────────────────────────────────┤
│  security/ + util/   → Capa Transversal     │  JWT · email · audit
└─────────────────────────────────────────────┘
      ↓
PostgreSQL (via JPA/Hibernate)
```

### Ejemplo en código

```kotlin
// controller/AuthController.kt — solo recibe el request y delega al service
@RestController
@RequestMapping("/api/v1/auth")
class AuthController(
    private val authService: AuthService
) {
    @PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    fun register(@Valid @RequestBody request: RegisterRequest): UserResponse =
        authService.registerUser(request)  // delega al service; no hay lógica aquí
}

// service/AuthService.kt — contiene la lógica de negocio
@Service
class AuthService(
    private val userRepository: UserRepository,
    private val passwordEncoder: BCryptPasswordEncoder
) {
    fun registerUser(request: RegisterRequest): UserResponse {
        if (userRepository.existsByEmail(request.email))
            throw ResponseStatusException(HttpStatus.CONFLICT, "El email ya está registrado")
        val user = User(
            email = request.email,
            hashedPassword = passwordEncoder.encode(request.password),  // llama a util
            fullName = request.fullName
        )
        return userRepository.save(user).toUserResponse()  // delega al repository
    }
}
```

### Ventaja

Un cambio en la base de datos **no afecta** el controller. Un cambio en el controller
**no afecta** la lógica de negocio. Cada capa es testeable de forma independiente.

> **Comparación con el proyecto de referencia (`proyecto-be-fe`):**
> En FastAPI la capa HTTP se llama `routers/`, la de negocio `services/` y el ORM usa
> SQLAlchemy. En Spring Boot los nombres son `controller/`, `service/` y el ORM es
> JPA/Hibernate — pero el principio de separación de capas es **exactamente el mismo**.

---

## Patrón 2 — DTO (Data Transfer Object)

### ¿Qué es?

Un objeto diseñado exclusivamente para transportar datos entre capas, diferente del
modelo de base de datos.

### ¿Por qué es crítico aquí?

La entidad JPA `User` tiene varias columnas, incluyendo `hashedPassword`. Si
devolviéramos la entidad directamente, **el hash de la contraseña quedaría expuesto
en la respuesta HTTP**. El DTO actúa como filtro.

### Ejemplo en código

```kotlin
// model/User.kt — Entidad JPA (lo que hay en la BD)
@Entity
@Table(name = "users")
data class User(
    @Id
    val id: UUID = UUID.randomUUID(),

    @Column(nullable = false, unique = true)
    val email: String,

    @Column(name = "full_name", nullable = false)
    val fullName: String,

    @Column(name = "hashed_password", nullable = false)
    var hashedPassword: String,     // ← NUNCA debe salir en la respuesta

    @Column(name = "is_active", nullable = false)
    var isActive: Boolean = true,

    @Column(name = "created_at", nullable = false, updatable = false)
    val createdAt: Instant = Instant.now()
)

// dto/AuthDtos.kt — DTO (lo que se devuelve al cliente)
data class UserResponse(
    val id: UUID,
    val email: String,
    val fullName: String,
    // hashedPassword: ← OMITIDO intencionalmente
    val isActive: Boolean,
    val createdAt: Instant
)
```

```kotlin
// controller/UserController.kt — Spring Boot convierte entidad → DTO manualmente
@GetMapping("/me")
fun getMe(authentication: Authentication): UserResponse {
    val user = userRepository.findByEmail(authentication.name)
        ?: throw ResponseStatusException(HttpStatus.NOT_FOUND)
    return UserResponse(                  // conversión explícita: entidad → DTO
        id = user.id,
        email = user.email,
        fullName = user.fullName,
        isActive = user.isActive,
        createdAt = user.createdAt
    )
}
```

### Ventaja

La API puede cambiar su contrato (el DTO) **sin alterar la estructura de la base de
datos**, y viceversa.

> **Comparación con `proyecto-be-fe`:**
> Pydantic `BaseModel` hace la conversión automáticamente con `model_config =
> ConfigDict(from_attributes=True)`. En Kotlin/Spring Boot la conversión es explícita
> (o mediante una función de extensión `fun User.toUserResponse(): UserResponse`).

---

## Patrón 3 — Inyección de Dependencias (DI)

### ¿Qué es?

En lugar de que cada clase cree sus propias dependencias, las **recibe inyectadas
desde afuera**. Spring IoC Container gestiona el ciclo de vida de los objetos.

En Spring Boot con Kotlin, la forma idiomática es mediante el **constructor**:
el contenedor de Spring detecta los parámetros del constructor y los inyecta
automáticamente.

### Ejemplo en código

```kotlin
// ✅ CORRECTO — constructor injection en Kotlin/Spring Boot
@Service
class AuthService(
    // ¿Qué? Spring provee instancias de estas dependencias automáticamente.
    // ¿Para qué? AuthService no necesita saber cómo se crea un UserRepository.
    // ¿Impacto? En tests, se pueden inyectar mocks sin tocar el código de producción.
    private val userRepository: UserRepository,
    private val passwordEncoder: BCryptPasswordEncoder,
    private val jwtTokenProvider: JwtTokenProvider,
    private val emailService: EmailService
) {
    // ...
}

// ❌ INCORRECTO — field injection (Java style, evitar en Kotlin)
@Service
class AuthService {
    @Autowired
    private lateinit var userRepository: UserRepository  // dificulta los tests
}
```

```kotlin
// test/service/AuthServiceTest.kt — override de dependencias con Mockito
@ExtendWith(MockitoExtension::class)
class AuthServiceTest {

    @Mock
    private lateinit var userRepository: UserRepository   // mock en lugar de BD real

    @Mock
    private lateinit var passwordEncoder: BCryptPasswordEncoder

    @InjectMocks
    private lateinit var authService: AuthService         // recibe los mocks

    @Test
    fun `register should throw CONFLICT when email already exists`() {
        whenever(userRepository.existsByEmail("test@email.com")).thenReturn(true)
        assertThrows<ResponseStatusException> {
            authService.registerUser(RegisterRequest("test@email.com", "Pass1234", "Test"))
        }
    }
}
```

### Ventaja

Para los tests, se puede **reemplazar** el `UserRepository` real por un mock sin
tocar el controller ni el service. Spring se encarga de resolver el grafo de
dependencias.

> **Comparación con `proyecto-be-fe`:**
> FastAPI usa `Depends()` como mecanismo declarativo de DI en los parámetros de la
> función. Spring Boot usa su IoC Container que inyecta por constructor. El concepto
> es el mismo — el framework se encarga de la instanciación.

---

## Patrón 4 — JWT Stateless

### ¿Qué es?

El servidor **no guarda sesión**. En cambio, emite un token firmado criptográficamente
que el cliente presenta en cada request. El servidor solo verifica la firma.

### Tokens del sistema

| Token           | Duración       | Propósito                                             |
| --------------- | -------------- | ----------------------------------------------------- |
| `access_token`  | **15 minutos** | Autenticar cada request a endpoints protegidos        |
| `refresh_token` | **7 días**     | Obtener un nuevo `access_token` sin volver a hacer login |

### Ejemplo en código

```kotlin
// security/JwtTokenProvider.kt — creación y verificación del token con JJWT
@Component
class JwtTokenProvider(
    @Value("\${jwt.secret}") private val secret: String,
    @Value("\${jwt.expiration-ms}") private val expirationMs: Long
) {
    // ¿Qué? Genera un Access Token firmado con HS256 para el usuario.
    // ¿Para qué? El cliente presentará este token en el header Authorization.
    // ¿Impacto? Si el secret cambia, todos los tokens anteriores dejan de ser válidos.
    fun createAccessToken(userId: String): String =
        Jwts.builder()
            .subject(userId)
            .issuedAt(Date())
            .expiration(Date(System.currentTimeMillis() + expirationMs))
            .signWith(Keys.hmacShaKeyFor(secret.toByteArray()), Jwts.SIG.HS256)
            .compact()

    // ¿Qué? Verifica la firma y decodifica el payload del token.
    // ¿Para qué? Extraer el userId para saber quién hace el request.
    // ¿Impacto? Si el token fue alterado o expiró, lanza JwtException → 401.
    fun validateAndGetClaims(token: String): Claims =
        Jwts.parser()
            .verifyWith(Keys.hmacShaKeyFor(secret.toByteArray()))
            .build()
            .parseSignedClaims(token)
            .payload
}
```

```kotlin
// security/JwtAuthFilter.kt — filtro que valida el JWT en cada request
@Component
class JwtAuthFilter(
    private val jwtTokenProvider: JwtTokenProvider,
    private val userDetailsService: UserDetailsServiceImpl
) : OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        // Extrae el token del header Authorization: Bearer <token>
        val token = extractToken(request) ?: run {
            filterChain.doFilter(request, response)
            return
        }

        try {
            val claims = jwtTokenProvider.validateAndGetClaims(token)
            val userId = claims.subject
            val userDetails = userDetailsService.loadUserByUsername(userId)
            val auth = UsernamePasswordAuthenticationToken(
                userDetails, null, userDetails.authorities
            )
            SecurityContextHolder.getContext().authentication = auth
        } catch (e: JwtException) {
            // Token inválido — dejar pasar sin autenticar (Spring Security retorna 401)
        }

        filterChain.doFilter(request, response)
    }
}
```

### Ventaja

El backend puede escalar a múltiples instancias sin compartir estado de sesión.
No hay tabla de sesiones. La información del usuario viaja **dentro** del token.

---

## Patrones 5, 6, 7 y 8 — Frontend React

> Los patrones de frontend son **idénticos** en todos los proyectos de la serie
> educativa (FastAPI, Express, Spring Boot Java, Spring Boot Kotlin, etc.).
> El stack de frontend no cambia entre proyectos.

---

## Patrón 5 — Context / Provider

### ¿Qué es?

React usa el patrón **Provider** para compartir estado global sin necesidad de pasar
props manualmente por cada nivel del árbol de componentes.

### Ejemplo en código

```typescript
// context/AuthContext.tsx
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<UserResponse | null>(null);
  const [accessToken, setAccessToken] = useState<string | null>(null);

  const login = async (credentials: LoginRequest) => {
    const response = await authApi.login(credentials);
    setAccessToken(response.access_token);
    // ...
  };

  return (
    <AuthContext.Provider value={{ user, accessToken, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}
```

```tsx
// main.tsx — AuthProvider envuelve toda la app
<AuthProvider>
  <BrowserRouter>
    <App />
  </BrowserRouter>
</AuthProvider>
```

### Ventaja

`DashboardPage`, `Navbar`, `ChangePasswordPage` **todos** acceden al mismo estado
de autenticación sin recibir props.

---

## Patrón 6 — Custom Hook

### ¿Qué es?

Una función de React que encapsula lógica reutilizable y puede usar otros hooks
internamente.

### Ejemplo en código

```typescript
// hooks/useAuth.ts
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth() debe usarse dentro de <AuthProvider>");
  }
  return context;
}
```

```typescript
// pages/DashboardPage.tsx — consumo del hook
export function DashboardPage() {
  const { user, logout } = useAuth();  // una línea, acceso completo al contexto
  return <h1>Bienvenido, {user?.fullName}</h1>;
}
```

### Ventaja

En lugar de escribir `useContext(AuthContext)` con su validación en cada componente,
se centraliza en `useAuth()`. Si el contexto cambia, **solo se modifica el hook**.

---

## Patrón 7 — Interceptor

### ¿Qué es?

Middleware a nivel de cliente HTTP que procesa **todas** las peticiones/respuestas
antes de que lleguen al código de la aplicación.

### Ejemplo en código

```typescript
// api/axios.ts
const api = axios.create({ baseURL: import.meta.env.VITE_API_URL });

// Interceptor de request — adjunta el token automáticamente
api.interceptors.request.use((config) => {
  const token = sessionStorage.getItem("access_token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Interceptor de response — maneja errores de autenticación
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      sessionStorage.clear();
      window.location.href = "/login";
    }
    return Promise.reject(error);
  },
);
```

### Ventaja

Ningún componente ni función de API necesita preocuparse por añadir el header
`Authorization`. Si el token cambia de lugar (por ejemplo, de `sessionStorage` a
una cookie), se modifica **un solo lugar**.

---

## Patrón 8 — SPA + Route Guard

### ¿Qué es?

En una SPA (_Single Page Application_), el enrutamiento ocurre en el cliente
(JavaScript), sin recargar la página. El **Route Guard** protege rutas que requieren
autenticación.

### Ejemplo en código

```typescript
// components/layout/ProtectedRoute.tsx
export function ProtectedRoute() {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) return <LoadingSpinner />;

  // Si no está autenticado, redirige sin mostrar la página
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}
```

```tsx
// App.tsx — configuración de rutas
<Routes>
  {/* Rutas públicas */}
  <Route path="/login" element={<LoginPage />} />
  <Route path="/register" element={<RegisterPage />} />

  {/* Rutas protegidas — requieren autenticación */}
  <Route element={<ProtectedRoute />}>
    <Route path="/dashboard" element={<DashboardPage />} />
    <Route path="/change-password" element={<ChangePasswordPage />} />
  </Route>
</Routes>
```

### Ventaja

Un usuario que visita `/dashboard` sin autenticarse **nunca ve** el HTML de la
página. Es redirigido inmediatamente. No hay necesidad de protección en cada
componente individualmente.

---

## Patrón 9 — Monorepo

### ¿Qué es?

Múltiples proyectos (frontend, backend, infraestructura) conviven en **un solo
repositorio git**.

### Estructura

```
proyecto-besbk-fe/        ← Un solo repositorio git
├── be/                   ← Backend (Kotlin/Spring Boot)
│   ├── src/
│   └── build.gradle.kts
├── fe/                   ← Frontend (React/TypeScript)
│   ├── src/
│   └── package.json
├── docker-compose.yml    ← Infraestructura compartida
└── .github/
    └── copilot-instructions.md
```

### Ventaja

- Un `git clone` obtiene todo el proyecto.
- Los cambios que afectan a backend **y** frontend viajan en el mismo commit.
- La infraestructura (`docker-compose.yml`) es parte del código versionado.

---

## Patrón 10 — REST API

### ¿Qué es?

Interfaz de comunicación basada en recursos HTTP con verbos (`GET`, `POST`, `PATCH`,
`DELETE`) y códigos de estado estándar (`200`, `201`, `400`, `401`, `404`, `422`).

### Endpoints del sistema

| Verbo   | Ruta                              | Código OK | Descripción                             |
| ------- | --------------------------------- | --------- | --------------------------------------- |
| `POST`  | `/api/v1/auth/register`           | `201`     | Registrar nuevo usuario                 |
| `POST`  | `/api/v1/auth/login`              | `200`     | Iniciar sesión, obtener tokens          |
| `POST`  | `/api/v1/auth/refresh`            | `200`     | Renovar access token                    |
| `POST`  | `/api/v1/auth/change-password`    | `200`     | Cambiar contraseña (auth)               |
| `POST`  | `/api/v1/auth/forgot-password`    | `200`     | Solicitar email de recuperación         |
| `POST`  | `/api/v1/auth/reset-password`     | `200`     | Restablecer contraseña con token        |
| `POST`  | `/api/v1/auth/verify-email`       | `200`     | Verificar dirección de email            |
| `GET`   | `/api/v1/users/me`                | `200`     | Obtener perfil del usuario autenticado  |
| `PATCH` | `/api/v1/users/me/locale`         | `200`     | Actualizar preferencia de idioma        |

### Ejemplo de respuesta estándar

```json
// POST /api/v1/auth/login → 200 OK
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}

// POST /api/v1/auth/register → 400 Bad Request (validación fallida - Bean Validation)
{
  "status": 400,
  "error": "Bad Request",
  "message": "El email es requerido"
}
```

### Ventaja

Cualquier cliente (React, Android, iOS, Postman, curl) puede consumir la API porque
habla HTTP estándar. La documentación es automática en `/swagger-ui.html`
(SpringDoc OpenAPI).

> **Comparación con `proyecto-be-fe`:**
> FastAPI genera Swagger en `/docs` automáticamente. Spring Boot usa SpringDoc
> OpenAPI que lo genera en `/swagger-ui.html`. Misma funcionalidad, diferente URL.

---

## Relación entre patrones

```
┌─────────────────────────────────────────────────────────────────────┐
│ Monorepo (#9)                                                        │
│                                                                      │
│  ┌─── REST API (#10) ─────────────────────────────────────────┐     │
│  │                                                             │     │
│  │  Frontend (SPA #8)           Backend (Capas #1)            │     │
│  │  ┌─────────────────────┐     ┌────────────────────────┐    │     │
│  │  │ Provider (#5)       │     │ controller/            │    │     │
│  │  │  Hook (#6)          │←────│ service/  ← DI (#3)    │    │     │
│  │  │  RouteGuard (#8)    │────→│ repository/ ← DTO (#2) │    │     │
│  │  │  Interceptor (#7)   │     │ security/ ← JWT (#4)   │    │     │
│  │  └─────────────────────┘     └──────────┬─────────────┘    │     │
│  │                                         │ JPA/Hibernate     │     │
│  │                              ┌──────────▼─────────────┐    │     │
│  │                              │ PostgreSQL              │    │     │
│  └──────────────────────────────┴────────────────────────┘    │     │
└──────────────────────────────────────────────────────────────┘     │
                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

Cada patrón resuelve un problema específico. Juntos, hacen que el sistema sea:

- **Seguro** — DTO + JWT + BCrypt
- **Mantenible** — Capas + DI + Custom Hook
- **Escalable** — Stateless + REST + Monorepo
- **Testeable** — DI con mocks + JUnit 5 + Vitest
