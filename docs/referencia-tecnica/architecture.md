<!--
  ¿Qué? Documento que describe la arquitectura completa del sistema NN Auth System — Kotlin Edition.
  ¿Para qué? Entender cómo se comunican las capas, qué tecnologías usa cada una
             y por qué se tomaron las decisiones de diseño que se tomaron.
  ¿Impacto? Sin este contexto, es difícil entender en qué parte del sistema
             vive cada archivo y cómo encajan los componentes entre sí.
-->

> **Proyecto**: NN Auth System — Kotlin Edition
> **Stack**: Spring Boot 3 (Kotlin / JDK 21) + React (TypeScript) + PostgreSQL 17 + Docker
> **Referencia**: Funcionalidad idéntica a [`proyecto-be-fe`](https://github.com/ergrato-dev/proyecto-be-fe) — diferente stack backend

---

# Arquitectura del Sistema — NN Auth System (Kotlin Edition)

## Vista General del Sistema

El sistema sigue una arquitectura **Cliente–Servidor de 3 capas**, donde cada capa tiene una responsabilidad única y se comunica solo con la capa adyacente:

```
┌──────────────────────────────────────────────────────────────────┐
│  CAPA 3 — CLIENTE (Navegador Web)                                │
│                                                                  │
│  React 18 + TypeScript + TailwindCSS + React Router              │
│  http://localhost:5173  (dev)  /  http://localhost:3000 (Docker) │
│                                                                  │
│  ┌─────────────┐  ┌─────────────────┐  ┌──────────────────────┐ │
│  │   Pages     │  │   Components    │  │  Context / Hooks     │ │
│  │  (vistas)   │  │  (UI + Layout)  │  │  (estado de auth)    │ │
│  └──────┬──────┘  └────────┬────────┘  └──────────────────────┘ │
│         └──────────────────┤                                     │
│                            ▼                                     │
│  ┌───────────────────────────────────┐                           │
│  │   api/auth.ts + api/axios.ts      │  (HTTP + JWT)             │
│  └───────────────────────────────────┘                           │
└────────────────────────────██████████████████────────────────────┘
                              ↕ JSON / HTTPS
┌────────────────────────────██████████████████────────────────────┐
│  CAPA 2 — SERVIDOR (Backend API)                                 │
│                                                                  │
│  Spring Boot 3 + Tomcat embebido (Kotlin / JDK 21)               │
│  http://localhost:8080                                           │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  Controllers │→ │  Services    │→ │  Utils               │   │
│  │  (endpoints) │  │  (lógica)    │  │  (email, audit log)  │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│         │                 │                                      │
│         ▼                 ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  DTOs (Bean Validation) + Entities (JPA) + Repositories  │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │  Spring Security (JwtAuthFilter + SecurityFilterChain)  │     │
│  └─────────────────────────────────────────────────────────┘     │
└────────────────────────────██████████████████────────────────────┘
                              ↕ SQL (PostgreSQL JDBC)
┌────────────────────────────██████████████████────────────────────┐
│  CAPA 1 — DATOS (Base de Datos)                                  │
│                                                                  │
│  PostgreSQL 17 (Docker Container)                                │
│  localhost:5432                                                  │
│                                                                  │
│  ┌────────┐  ┌──────────────────────┐  ┌────────────────────┐   │
│  │ users  │  │ password_reset_      │  │ email_verification │   │
│  │        │  │ tokens               │  │ _tokens            │   │
│  └────────┘  └──────────────────────┘  └────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

> **Diferencia con `proyecto-be-fe`**: La arquitectura de capas es **idéntica**. La diferencia está
> en la implementación de la Capa 2: FastAPI (Python/ASGI) → Spring Boot (Kotlin/Servlet).

---

## Arquitectura del Backend (`be/`)

### Estructura de capas

```
be/src/main/kotlin/com/nn/auth/
│
├── NnAuthApplication.kt     ← Punto de entrada: @SpringBootApplication, inicia Tomcat
├── config/
│   ├── SecurityConfig.kt    ← SecurityFilterChain, CORS, reglas de autorización
│   ├── OpenApiConfig.kt     ← SpringDoc — Swagger UI en /swagger-ui.html
│   └── AppProperties.kt     ← @ConfigurationProperties — lee variables de entorno
│
├── controller/              ← CAPA DE PRESENTACIÓN (HTTP)
│   ├── AuthController.kt    ← POST /register, /login, /refresh, /change-password,
│   │                           /forgot-password, /reset-password, /verify-email
│   └── UserController.kt    ← GET /me
│
├── service/                 ← CAPA DE LÓGICA DE NEGOCIO
│   └── AuthService.kt       ← registerUser, loginUser, refreshToken,
│                               changePassword, requestPasswordReset,
│                               resetPassword, verifyEmail
│
├── repository/              ← CAPA DE ACCESO A DATOS
│   ├── UserRepository.kt    ← JpaRepository<User, UUID>
│   ├── PasswordResetTokenRepository.kt
│   └── EmailVerificationTokenRepository.kt
│
├── model/                   ← ENTIDADES JPA (tablas de la BD)
│   ├── User.kt              ← @Entity — tabla users
│   ├── PasswordResetToken.kt
│   └── EmailVerificationToken.kt
│
├── dto/                     ← DTOs (validación request/response)
│   └── AuthDtos.kt          ← RegisterRequest, LoginRequest, TokenResponse,
│                               ChangePasswordRequest, ForgotPasswordRequest,
│                               ResetPasswordRequest, VerifyEmailRequest,
│                               UserResponse, MessageResponse
│
├── security/                ← SEGURIDAD
│   ├── JwtTokenProvider.kt  ← createAccessToken, createRefreshToken, validateToken
│   ├── JwtAuthFilter.kt     ← OncePerRequestFilter — valida JWT en cada request
│   └── UserDetailsServiceImpl.kt ← Carga usuario desde BD para Spring Security
│
└── util/                    ← UTILIDADES TRANSVERSALES
    ├── EmailService.kt      ← sendVerificationEmail, sendPasswordResetEmail
    └── AuditLogService.kt   ← logLoginSuccess/Failed, logPasswordChanged, etc.
```

> **Comparativa con `proyecto-be-fe`**:
>
> | Python (FastAPI) | Kotlin (Spring Boot) |
> |---|---|
> | `app/main.py` → `FastAPI()` | `NnAuthApplication.kt` → `@SpringBootApplication` |
> | `config.py` → Pydantic Settings | `AppProperties.kt` → `@ConfigurationProperties` |
> | `database.py` → SQLAlchemy Session | `application.yml` → Spring Boot DataSource auto |
> | `routers/auth.py` → `@router.post(...)` | `AuthController.kt` → `@RestController` |
> | `services/auth_service.py` | `AuthService.kt` → `@Service` |
> | `schemas/user.py` → Pydantic model | `dto/AuthDtos.kt` → data class + `@Valid` |
> | `models/user.py` → SQLAlchemy ORM | `model/User.kt` → `@Entity` JPA |
> | `utils/security.py` → `python-jose` | `security/JwtTokenProvider.kt` → JJWT |

---

### Flujo de una petición HTTP

```
1. Cliente envía:   POST /api/v1/auth/login { email, password }
                    ↓
2. Spring Security: JwtAuthFilter.doFilterInternal()
                    → Para este endpoint público, no requiere token
                    → Pasa al siguiente filtro
                    ↓
3. Rate Limiting:   Bucket4j verifica si la IP superó el límite (10/min)
                    → Si sí → 429 Too Many Requests
                    ↓
4. Controller:      AuthController.login() recibe @RequestBody LoginRequest
                    → Bean Validation valida automáticamente (@Valid)
                    → Si error de validación → 400 Bad Request
                    → Llama a authService.loginUser(loginRequest)
                    ↓
5. Service:         AuthService.loginUser():
                    → Busca user por email (userRepository.findByEmail)
                    → Verifica password: passwordEncoder.matches(plain, hashed)
                    → Verifica is_email_verified y is_active
                    → jwtTokenProvider.createAccessToken(user)
                    → jwtTokenProvider.createRefreshToken(user)
                    → auditLogService.logLoginSuccess(user, ip)
                    ↓
6. Response:        Controller retorna ResponseEntity<TokenResponse>
                    Spring Jackson serializa a JSON automáticamente
```

> **Diferencia con `proyecto-be-fe`**: El flujo es conceptualmente idéntico.
> Las diferencias son de implementación:
> - Python usa `Depends()` para inyección; Kotlin usa constructores con `@Autowired`.
> - Python usa Pydantic para validación automática; Kotlin usa `@Valid` + Bean Validation.
> - Python usa `HTTPException`; Kotlin usa `ResponseStatusException` o `@ControllerAdvice`.

---

### Seguridad en el backend

```
┌────────────────────────────────────────────────────────────────┐
│  Capas de seguridad (de afuera hacia adentro)                  │
│                                                                │
│  1. CORS (SecurityConfig)        → Solo FRONTEND_URL           │
│  2. Security Headers             → X-Frame-Options, nosniff   │
│  3. Rate Limiting (Bucket4j)     → Por IP, por endpoint        │
│  4. Bean Validation (@Valid)     → Tipos, formato, fortaleza   │
│  5. JwtAuthFilter                → Valida Bearer token         │
│  6. Business Logic Checks        → is_active, is_email_verified│
│  7. Spring Data JPA              → No raw SQL, no injection    │
│  8. BCryptPasswordEncoder        → Contraseñas nunca en plano  │
│  9. AuditLogService              → Trazabilidad de eventos     │
└────────────────────────────────────────────────────────────────┘
```

> **Diferencia con `proyecto-be-fe`**:
> - Python usa `CORSMiddleware` de Starlette; Kotlin configura `CorsConfigurationSource` en Spring Security.
> - Python usa `slowapi` para rate limiting; Kotlin usa `Bucket4j`.
> - Python usa `Depends(get_current_user)` para proteger endpoints; Kotlin usa `JwtAuthFilter` + `SecurityFilterChain`.

---

## Arquitectura del Frontend (`fe/`)

### Estructura de capas

```
fe/src/
│
├── main.tsx         ← Punto de entrada: renderiza <App /> en el DOM
├── App.tsx          ← Rutas de la aplicación (React Router)
├── index.css        ← Estilos globales + imports de TailwindCSS
│
├── context/         ← ESTADO GLOBAL
│   └── AuthContext.tsx     ← Provider: usuario actual, tokens, loading
│
├── hooks/           ← LÓGICA REUTILIZABLE
│   └── useAuth.ts   ← Acceso al contexto de auth desde cualquier componente
│
├── api/             ← COMUNICACIÓN HTTP
│   ├── auth.ts      ← Funciones: register, login, refresh, changePassword, etc.
│   └── axios.ts     ← Instancia de Axios con interceptores JWT automáticos
│
├── components/      ← COMPONENTES REUTILIZABLES
│   ├── ProtectedRoute.tsx  ← Guarda de rutas autenticadas
│   ├── layout/
│   │   ├── AuthLayout.tsx  ← <main> landmark para páginas de auth
│   │   └── Navbar.tsx      ← Barra de navegación con tema y logout
│   └── ui/
│       ├── Button.tsx      ← Botón con estado loading (aria-busy)
│       ├── InputField.tsx  ← Input con label, ícono, validación y a11y
│       └── Alert.tsx       ← Mensajes de éxito/error/info
│
├── pages/           ← VISTAS (una por ruta)
│   ├── LandingPage.tsx
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   ├── DashboardPage.tsx
│   ├── ChangePasswordPage.tsx
│   ├── ForgotPasswordPage.tsx
│   ├── ResetPasswordPage.tsx
│   ├── TerminosDeUsoPage.tsx
│   ├── PoliticaPrivacidadPage.tsx
│   ├── PoliticaCookiesPage.tsx
│   └── ContactPage.tsx
│
└── types/           ← TIPOS TYPESCRIPT
    └── auth.ts      ← LoginRequest, RegisterRequest, UserResponse, TokenResponse, etc.
```

> **El frontend es idéntico** al del proyecto `proyecto-be-fe`. La única diferencia es
> que la variable de entorno `VITE_API_URL` apunta al puerto `8080` en lugar del `8000`.

---

### Rutas de la aplicación

```
/                → LandingPage (pública)
/login           → LoginPage (pública)
/register        → RegisterPage (pública)
/forgot-password → ForgotPasswordPage (pública)
/reset-password  → ResetPasswordPage (pública, requiere ?token=...)
/verify-email    → Manejado con ?token=... → llama al backend
/dashboard       → DashboardPage (PROTEGIDA — requiere auth)
/change-password → ChangePasswordPage (PROTEGIDA — requiere auth)
/terminos-de-uso → TerminosDeUsoPage (pública)
/privacidad      → PoliticaPrivacidadPage (pública)
/cookies         → PoliticaCookiesPage (pública)
/contacto        → ContactPage (pública)
```

---

### Flujo de autenticación en el frontend

```
Arranque de la app:
1. AuthContext se monta → lee access_token de memoria (no localStorage)
2. Si hay token → verifica con GET /api/v1/users/me
3. Si 200 → usuario autenticado, redirecciona a /dashboard
4. Si 401 → intenta refresh con POST /api/v1/auth/refresh
5. Si refresh falla → usuario va a /login

Login exitoso:
1. LoginPage → auth.login(email, password) → TokenResponse
2. AuthContext guarda tokens en memoria (no localStorage por seguridad)
3. GET /api/v1/users/me → guarda perfil en estado
4. React Router navega a /dashboard

Expiración del access_token (15 min):
1. Axios interceptor detecta 401 en respuesta
2. Automáticamente llama POST /api/v1/auth/refresh
3. Si refresh exitoso → reintenta la petición original
4. Si refresh falla → logout + redirect a /login
```

---

## Flujos de Autenticación de Extremo a Extremo

### Flujo 1 — Registro y Verificación de Email

```
Usuario             Frontend (React)          Backend (Spring Boot)     Email (Mailpit)
   │                      │                          │                       │
   │ Rellena formulario   │                          │                       │
   │ ───────────────────► │                          │                       │
   │                      │ POST /auth/register       │                       │
   │                      │ ────────────────────────► │                       │
   │                      │                          │ 1. @Valid valida DTO   │
   │                      │                          │ 2. Verifica email ∄   │
   │                      │                          │ 3. BCrypt password     │
   │                      │                          │ 4. Crea User          │
   │                      │                          │    (is_verified=false) │
   │                      │                          │ 5. Crea EmailToken     │
   │                      │                          │ 6. emailService.send() │
   │                      │                          │ ──────────────────────►│
   │                      │ ◄────────────────────────│                       │
   │ Ve "Verifica email"  │  201 UserResponse         │ Envía email c/ token   │
   │ ◄────────────────────│                          │                       │
   │                      │                          │                       │
   │ Clic en enlace email │                          │                       │
   │ ───────────────────► │                          │                       │
   │                      │ POST /auth/verify-email  │                       │
   │                      │ ────────────────────────► │                       │
   │                      │                          │ 1. Busca token en BD  │
   │                      │                          │ 2. Verifica no expir. │
   │                      │                          │ 3. Marca used=true    │
   │                      │                          │ 4. isVerified=true    │
   │ "Verificado, login"  │ ◄────────────────────────│                       │
   │ ◄────────────────────│  200 MessageResponse      │                       │
```

### Flujo 2 — Login y Acceso a Dashboard

```
Usuario         Frontend              Backend               PostgreSQL
   │               │                     │                      │
   │ email+pass    │                     │                      │
   │ ────────────► │                     │                      │
   │               │ POST /auth/login    │                      │
   │               │ ──────────────────► │                      │
   │               │                     │ SELECT * FROM users  │
   │               │                     │ WHERE email = ?  ───►│
   │               │                     │ ◄────────────────────│
   │               │                     │ passwordEncoder      │
   │               │                     │ .matches(plain,hash) │
   │               │                     │ jwtTokenProvider     │
   │               │                     │ .createAccessToken() │
   │               │ ◄───────────────────│                      │
   │               │ 200 TokenResponse   │                      │
   │               │ (access+refresh)    │                      │
   │               │                     │                      │
   │               │ GET /users/me        │                      │
   │               │ Authorization: Bearer│                      │
   │               │ ──────────────────► │                      │
   │               │                     │ JwtAuthFilter        │
   │               │                     │ validateToken()      │
   │               │                     │ SELECT user by id ──►│
   │ Dashboard     │ ◄───────────────────│                      │
   │ ◄─────────────│ 200 UserResponse    │                      │
```

### Flujo 3 — Recuperación de Contraseña

```
Usuario         Frontend              Backend               PostgreSQL    Mailpit
   │               │                     │                      │           │
   │ Ingresa email │                     │                      │           │
   │ ────────────► │ POST /auth/forgot-password                 │           │
   │               │ ──────────────────► │                      │           │
   │               │                     │ Busca user por email─►│           │
   │               │                     │ (si no existe, resp. │           │
   │               │                     │ genérica igual)      │           │
   │               │                     │ Crea PasswordReset   │           │
   │               │                     │ Token                │           │
   │               │                     │ ────────────────────────────────► │
   │ "Revisa tu    │ ◄───────────────────│                      │ Envía mail │
   │  email"       │ 200 (siempre igual) │                      │           │
   │ ◄─────────────│                     │                      │           │
   │               │                     │                      │           │
   │ Clic en link  │                     │                      │           │
   │ ────────────► │ POST /auth/reset-password { token, pass }  │           │
   │               │ ──────────────────► │                      │           │
   │               │                     │ Valida token ───────►│           │
   │               │                     │ BCrypt(new_password) │           │
   │               │                     │ UPDATE users pass    │           │
   │               │                     │ UPDATE token used=true│          │
   │ "Contraseña   │ ◄───────────────────│                      │           │
   │  restablecida"│ 200 MessageResponse │                      │           │
   │ ◄─────────────│                     │                      │           │
```

---

## Decisiones Técnicas Clave

### ¿Por qué Spring Boot y no Ktor?

| Criterio | Spring Boot 3 | Ktor |
|---|---|---|
| Madurez | ✅ 20+ años en producción | ⚠️ Relativamente reciente |
| Ecosistema | ✅ Amplísimo (Spring Security, JPA, etc.) | ⚠️ Más limitado |
| Documentación | ✅ Extensísima | ⚠️ Menor cantidad |
| Aprendizaje laboral | ✅ Estándar de la industria JVM | ⚠️ Nicho |
| Autoconfiguración | ✅ Convention over configuration | ❌ Manual |

Spring Boot fue elegido porque es el estándar de la industria en el ecosistema JVM,
tiene el ecosistema más maduro para autenticación (Spring Security), y su modelo
de autoconfiguración acelera el desarrollo pedagógico.

### ¿Por qué JWT stateless y no sesiones en servidor?

| Criterio | JWT Stateless | Sesiones en servidor |
|---|---|---|
| Escalabilidad | ✅ Horizontal trivial | ❌ Requiere sticky sessions o Redis |
| Estado en servidor | ✅ Ninguno | ❌ HttpSession en servidor |
| Revocación | ❌ Requiere blacklist | ✅ Invalidar sesión directamente |
| Apropiado para SPA | ✅ Diseñado para esto | ⚠️ Problemas con CORS |

Para este proyecto educativo, la arquitectura stateless con JWT es la más apropiada.

### ¿Por qué Flyway y no Liquibase?

| Criterio | Flyway | Liquibase |
|---|---|---|
| Simplicidad | ✅ SQL puro, sin XML | ⚠️ XML/YAML/JSON/SQL |
| Curva de aprendizaje | ✅ Muy baja | ❌ Mayor complejidad |
| Equivalente pedagógico | ✅ Comparable a Alembic (SQL directo) | ⚠️ Abstracción extra |

Flyway fue elegido porque los scripts de migración son **SQL puro** — exactamente el mismo
concepto que Alembic en `proyecto-be-fe`, lo que facilita la comparación pedagógica.

### ¿Por qué Kotlin y no Java?

| Criterio | Kotlin | Java |
|---|---|---|
| Nulabilidad | ✅ Nativa en el tipo system | ❌ Requiere @NonNull manual |
| Data classes | ✅ En una línea | ❌ Verboso (getters/setters/equals/hashCode) |
| Concisión | ✅ Menos boilerplate | ❌ Verboso |
| Interoperabilidad JVM | ✅ 100% compatible con Java | ✅ — |
| Preferido por Google (Android) | ✅ Oficial | ⚠️ Segundo lugar |

Kotlin es el lenguaje moderno del ecosistema JVM — conciso, seguro en nulabilidad,
compatible 100% con Java y preferido por Spring Boot para nuevos proyectos.

### ¿Por qué React + Vite y no Next.js?

Este es un proyecto de SPA (Single Page Application) — toda la navegación ocurre
en el cliente. Next.js agrega SSR que no es necesario para una app de auth.
Vite es más rápido en desarrollo y la configuración es más simple para aprendizaje.

---

## Configuración de Entornos

| Variable | Desarrollo | Producción |
|---|---|---|
| `APP_ENVIRONMENT` | `development` | `production` |
| Swagger UI (`/swagger-ui.html`) | ✅ Disponible | ❌ Deshabilitado |
| `SPRING_DATASOURCE_URL` | `localhost:5432` | Servidor de BD en cloud |
| `FRONTEND_URL` | `http://localhost:5173` | `https://tu-dominio.com` |
| `JWT_SECRET` | Clave de desarrollo (≥32 ch) | Clave aleatoria larga |
| `SMTP_HOST` | `localhost` / `mailpit` | SMTP real (SES, SendGrid, etc.) |

> **Diferencia con `proyecto-be-fe`**: El proyecto de referencia usa `ENVIRONMENT=development/production`.
> Este proyecto usa `APP_ENVIRONMENT` con el mismo comportamiento — Swagger/OpenAPI se deshabilita en producción.
