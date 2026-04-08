<!--
  ¿Qué? Documentación completa de todos los endpoints de la API REST del sistema.
  ¿Para qué? Servir como referencia para el frontend y para pruebas manuales en Swagger UI.
             En producción (donde Swagger está deshabilitado), este archivo es la
             única fuente de verdad sobre la API.
  ¿Impacto? Sin esta documentación, el frontend no sabría qué enviar ni qué esperar
             de cada endpoint.
-->

> **Base URL**: `http://localhost:8080`
> **Versionamiento**: Todos los endpoints usan el prefijo `/api/v1/`
> **Formato**: JSON en request y response (`Content-Type: application/json`)
> **Autenticación**: JWT Bearer Token — `Authorization: Bearer <access_token>`
> **Swagger UI** (solo en development): `http://localhost:8080/swagger-ui.html`

> **Diferencia con `proyecto-be-fe`**: El proyecto de referencia usa el **puerto 8000** (FastAPI/Uvicorn).
> Este proyecto usa el **puerto 8080** (Spring Boot/Tomcat embebido), que es el puerto por defecto de Spring Boot.

---

# API Endpoints — NN Auth System (Kotlin Edition)

## Resumen de Endpoints

| Método | Ruta | Descripción | Auth requerida | Rate limit |
|---|---|---|---|---|
| POST | `/api/v1/auth/register` | Registrar nuevo usuario | No | 5/min |
| POST | `/api/v1/auth/login` | Iniciar sesión, obtener tokens JWT | No | 10/min |
| POST | `/api/v1/auth/refresh` | Renovar access token con refresh token | No † | — |
| POST | `/api/v1/auth/change-password` | Cambiar contraseña (usuario autenticado) | Sí | — |
| POST | `/api/v1/auth/forgot-password` | Solicitar email de recuperación | No | 5/min |
| POST | `/api/v1/auth/reset-password` | Restablecer contraseña con token | No † | — |
| POST | `/api/v1/auth/verify-email` | Verificar dirección de email | No † | — |
| GET | `/api/v1/users/me` | Obtener perfil del usuario actual | Sí | — |
| PATCH | `/api/v1/users/me/locale` | Actualizar idioma preferido del usuario | Sí | — |

> † No requiere `Authorization` header, pero sí un token específico en el body
> (refresh token, reset token o verification token).

---

## Códigos de Estado HTTP Comunes

| Código | Significado |
|---|---|
| 200 | OK — Operación exitosa |
| 201 | Created — Recurso creado exitosamente (registro de usuario) |
| 400 | Bad Request — Datos inválidos (email duplicado, lógica de negocio) |
| 401 | Unauthorized — Token inválido, expirado o ausente |
| 403 | Forbidden — Autenticado pero sin permiso (email no verificado) |
| 422 | Unprocessable Entity — Validación Bean Validation fallida |
| 429 | Too Many Requests — Rate limit superado (Bucket4j) |
| 500 | Internal Server Error — Error no controlado del servidor |

> **Diferencia con `proyecto-be-fe`**: Los códigos HTTP son **idénticos**. La diferencia está
> en cómo se generan internamente: Pydantic (Python) → Bean Validation (Kotlin/JVM).

---

## Autenticación — Endpoints Auth

---

### POST /api/v1/auth/register

Registra un nuevo usuario en el sistema y envía un email de verificación.

**Rate limit**: 5 peticiones/minuto por IP (implementado con Bucket4j)

**Request body:**

```json
{
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "password": "MiContrasena1"
}
```

| Campo | Tipo | Requerido | Validaciones (Bean Validation) |
|---|---|---|---|
| email | string | Sí | Formato email válido (`@Email`) |
| full_name | string | Sí | Mínimo 1 caracter (`@NotBlank`) |
| password | string | Sí | ≥8 chars, 1 mayúscula, 1 minúscula, 1 número (`@Pattern`) |

**Respuesta exitosa (201 Created):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "is_active": true,
  "is_email_verified": false,
  "created_at": "2026-03-15T10:30:00+00:00",
  "updated_at": "2026-03-15T10:30:00+00:00"
}
```

> ⚠️ El usuario queda con `is_email_verified: false`. No puede hacer login hasta verificar su email.

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | Email ya registrado en el sistema |
| 400 | Validación fallida (password débil, email con formato inválido, etc.) |
| 429 | Rate limit superado (5/min) |

> **Implementación Spring Boot**: `@Valid` en el parámetro del controller activa Bean Validation automáticamente.
> Si la validación falla, Spring lanza `MethodArgumentNotValidException` → 400.
>
> **Comparativa con `proyecto-be-fe`**: Pydantic valida automáticamente en el endpoint y retorna 422.
> Spring Boot retorna 400 por defecto con Bean Validation — se puede personalizar con `@ControllerAdvice`.

---

### POST /api/v1/auth/login

Autentica al usuario y retorna un par de tokens JWT.

**Rate limit**: 10 peticiones/minuto por IP

**Request body:**

```json
{
  "email": "usuario@ejemplo.com",
  "password": "MiContrasena1"
}
```

| Campo | Tipo | Requerido |
|---|---|---|
| email | string | Sí |
| password | string | Sí |

**Respuesta exitosa (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer"
}
```

| Campo | Tipo | Descripción |
|---|---|---|
| access_token | string | JWT de acceso — duración 15 min |
| refresh_token | string | JWT de renovación — duración 7 días |
| token_type | string | Siempre `"Bearer"` |

**Cómo usar los tokens:**

```bash
# Usar access_token en cada petición protegida:
GET /api/v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Errores posibles:**

| Código | Causa |
|---|---|
| 401 | Email no registrado o contraseña incorrecta |
| 403 | Cuenta creada pero email no verificado aún |
| 429 | Rate limit superado (10/min) |

> ⚠️ **Nota de seguridad**: La API devuelve el mismo `401` tanto si el email no existe
> como si la contraseña es incorrecta. Esto evita la enumeración de usuarios (OWASP A07).

---

### POST /api/v1/auth/refresh

Genera un nuevo par de tokens usando un refresh token válido (**rotación de tokens**).

**Request body:**

```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Respuesta exitosa (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsIproyecto-benR5cCI6IkpXVCJ9...",
  "token_type": "Bearer"
}
```

> **Rotación de tokens**: Se retorna un nuevo par `access_token + refresh_token`.
> El frontend debe reemplazar ambos tokens al recibirlos.

**Cuándo usar este endpoint:**

```
1. El frontend hace GET /api/v1/users/me → 401 (access_token expirado)
2. El frontend llama POST /api/v1/auth/refresh con el refresh_token guardado
3. El servidor retorna nuevos access_token + refresh_token
4. El frontend reintenta GET /api/v1/users/me con el nuevo access_token
```

**Errores posibles:**

| Código | Causa |
|---|---|
| 401 | Refresh token inválido, expirado o malformado |

> **Implementación Spring Boot**: `JwtTokenProvider.validateToken()` verifica la firma HMAC-SHA256
> y la fecha de expiración usando la librería JJWT.
>
> **Comparativa con `proyecto-be-fe`**: Mismo comportamiento. Python usa `python-jose`; Kotlin usa `io.jsonwebtoken` (JJWT).

---

### POST /api/v1/auth/change-password

Cambia la contraseña del usuario autenticado, verificando primero la contraseña actual.

**Auth requerida**: `Authorization: Bearer <access_token>`

**Request body:**

```json
{
  "current_password": "MiContrasenaActual1",
  "new_password": "MiNuevaContrasena1"
}
```

| Campo | Tipo | Validaciones |
|---|---|---|
| current_password | string | Contraseña actual correcta |
| new_password | string | ≥8 chars, 1 mayúscula, 1 minúscula, 1 número |

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Contraseña actualizada exitosamente"
}
```

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | La contraseña actual no es correcta |
| 400 | Nueva contraseña no cumple requisitos de fortaleza |
| 401 | Token ausente, inválido o expirado |

> **Implementación Spring Boot**: `JwtAuthFilter` extrae el usuario del token y lo carga en
> `SecurityContextHolder`. El servicio lo obtiene con `SecurityContextHolder.getContext().authentication`.

---

### POST /api/v1/auth/forgot-password

Solicita el envío de un email con enlace de recuperación de contraseña.

**Rate limit**: 5 peticiones/minuto por IP

**Request body:**

```json
{
  "email": "usuario@ejemplo.com"
}
```

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Si el email está registrado, recibirás un enlace de recuperación"
}
```

> ⚠️ **Nota de seguridad**: La respuesta es **siempre idéntica**, sin importar si el email
> está registrado o no. Esto previene la **enumeración de usuarios** — un atacante no puede
> saber si un email existe consultando la respuesta (OWASP A07).

**El enlace enviado al email tiene este formato:**

```
{FRONTEND_URL}/reset-password?token=<uuid-token>
```

El token expira en **1 hora**.

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | Email con formato inválido |
| 429 | Rate limit superado (5/min) |

---

### POST /api/v1/auth/reset-password

Restablece la contraseña usando el token recibido por email.

**Request body:**

```json
{
  "token": "550e8400-e29b-41d4-a716-446655440000",
  "new_password": "MiNuevaContrasena1"
}
```

| Campo | Tipo | Validaciones |
|---|---|---|
| token | string | Token UUID del email de recuperación |
| new_password | string | ≥8 chars, 1 mayúscula, 1 min., 1 núm. |

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Contraseña restablecida exitosamente"
}
```

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | Token inválido, expirado (>1h) o ya usado |
| 400 | Nueva contraseña no cumple requisitos de fortaleza |

---

### POST /api/v1/auth/verify-email

Verifica la dirección de email del usuario usando el token enviado al registrarse.

**Request body:**

```json
{
  "token": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Respuesta exitosa (200 OK):**

```json
{
  "message": "Email verificado exitosamente. Ya puedes iniciar sesión."
}
```

> Tras la verificación exitosa, el campo `is_email_verified` del usuario pasa a `true`
> y puede hacer login.

**El enlace del email de verificación tiene este formato:**

```
{FRONTEND_URL}/verify-email?token=<uuid-token>
```

El token expira en **24 horas**.

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | Token inválido, expirado (>24h) o ya usado |

---

## Perfil de Usuario — Endpoints Users

---

### GET /api/v1/users/me

Retorna el perfil del usuario autenticado actualmente.

**Auth requerida**: `Authorization: Bearer <access_token>`

**Respuesta exitosa (200 OK):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "is_active": true,
  "is_email_verified": true,
  "locale": "es",
  "created_at": "2026-03-15T10:30:00+00:00",
  "updated_at": "2026-03-15T10:30:00+00:00"
}
```

| Campo | Tipo | Descripción |
|---|---|---|
| id | string (UUID) | Identificador único del usuario |
| email | string | Email del usuario |
| full_name | string | Nombre completo |
| is_active | boolean | Si la cuenta está activa |
| is_email_verified | boolean | Si el email fue verificado |
| locale | string | Idioma preferido del usuario (`es` / `en`) |
| created_at | string (ISO8601) | Fecha de registro |
| updated_at | string (ISO8601) | Fecha de última actualización |

> ⚠️ El campo `hashed_password` **nunca** aparece en ninguna respuesta de la API.
> El DTO `UserResponse` lo excluye explícitamente — la entidad `User` tiene el campo,
> pero el DTO de respuesta no lo incluye.

**Errores posibles:**

| Código | Causa |
|---|---|
| 401 | Token ausente, inválido o expirado |

---

### PATCH /api/v1/users/me/locale

Actualiza la preferencia de idioma del usuario autenticado. Llamado por el frontend
cuando el usuario cambia el idioma en el `LanguageSwitcher`.

**Auth requerida**: `Authorization: Bearer <access_token>`

**Request body:**

```json
{
  "locale": "en"
}
```

| Campo | Tipo | Requerido | Valores válidos |
|---|---|---|---|
| locale | string | Sí | `"es"` o `"en"` exclusivamente |

**Respuesta exitosa (200 OK):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "is_active": true,
  "is_email_verified": true,
  "locale": "en",
  "created_at": "2026-03-15T10:30:00+00:00",
  "updated_at": "2026-03-15T11:45:00+00:00"
}
```

**Errores posibles:**

| Código | Causa |
|---|---|
| 400 | `locale` no es `"es"` ni `"en"`, o el campo falta en el body |
| 401 | Token ausente, inválido o expirado |

---

## Rate Limiting — Detalles Técnicos

El rate limiting está implementado con [Bucket4j](https://github.com/bucket4j/bucket4j),
usando la IP del cliente como identificador.

> **Diferencia con `proyecto-be-fe`**: El proyecto de referencia usa `slowapi` (Python).
> Este proyecto usa `Bucket4j` — mismo concepto (token bucket algorithm), diferente librería.

| Endpoint | Límite | Motivo |
|---|---|---|
| POST /auth/register | 5/min | Prevenir creación masiva de cuentas falsas |
| POST /auth/login | 10/min | Prevenir brute force de contraseñas |
| POST /auth/forgot-password | 5/min | Prevenir spam de emails de recuperación |

**Respuesta cuando se supera el límite (429):**

```json
{
  "error": "Rate limit exceeded",
  "message": "Has superado el límite de intentos. Espera 1 minuto."
}
```

---

## Seguridad de la Documentación

En producción (`APP_ENVIRONMENT=production`), la ruta `/swagger-ui.html` está deshabilitada
— devuelve 404. Esta es una medida de seguridad (OWASP A05) para evitar exponer la
superficie de ataque de la API públicamente.

En desarrollo (`APP_ENVIRONMENT=development`, valor por defecto), Swagger UI está disponible en:

- `http://localhost:8080/swagger-ui.html` — Swagger UI (interactivo)
- `http://localhost:8080/v3/api-docs` — OpenAPI JSON (para herramientas externas)

> **Comparativa con `proyecto-be-fe`**: El proyecto de referencia usa `/docs` y `/redoc` (FastAPI nativo).
> Este proyecto usa `/swagger-ui.html` y `/v3/api-docs` (SpringDoc OpenAPI).

---

## Flujos de Uso — Ejemplos Paso a Paso

### Flujo completo de registro y verificación

```
1. POST /api/v1/auth/register → 201 (is_email_verified: false)
2. (Revisar bandeja de entrada en Mailpit: http://localhost:8025)
3. POST /api/v1/auth/verify-email { token } → 200
4. POST /api/v1/auth/login { email, password } → 200 (access_token, refresh_token)
5. GET /api/v1/users/me con Bearer token → 200 (perfil completo)
```

### Flujo de renovación de sesión

```
1. (access_token expira a los 15 minutos)
2. POST /api/v1/auth/refresh { refresh_token } → 200 (nuevos tokens)
3. (Si refresh_token también expiró por >7 días)
   → 401 → Redirigir al login
```

### Flujo de recuperación de contraseña

```
1. POST /api/v1/auth/forgot-password { email } → 200 (siempre)
2. (El usuario hace clic en el enlace del email)
   Frontend captura: /reset-password?token=<uuid>
3. POST /api/v1/auth/reset-password { token, new_password } → 200
4. POST /api/v1/auth/login { email, new_password } → 200 (nuevos tokens)
```

### Flujo de cambio de contraseña (usuario logueado)

```
1. (Usuario logueado, tiene access_token válido)
2. POST /api/v1/auth/change-password { current_password, new_password }
   Authorization: Bearer <access_token>
   → 200 { message: "Contraseña actualizada exitosamente" }
3. (El usuario puede seguir usando el mismo access_token — no se invalida)
```
