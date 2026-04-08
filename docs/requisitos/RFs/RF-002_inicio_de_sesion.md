<!--
  ¿Qué? Requerimiento funcional para el inicio de sesión.
  ¿Para qué? Especificar cómo el backend autentica credenciales y genera tokens JWT.
  ¿Impacto? Es el núcleo del sistema de autenticación; un error aquí compromete
            la seguridad de todas las cuentas.
-->

# RF-002 — Inicio de sesión

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-002                                   |
| **Nombre**      | Inicio de sesión con correo y contraseña |
| **HU asociada** | HU-002                                   |
| **Endpoint**    | `POST /api/v1/auth/login`                |
| **Auth**        | No requerida                             |

---

## Descripción

El sistema autentica a un usuario verificando su correo y contraseña, confirma que la
cuenta esté activa y con correo verificado, y genera un par de tokens JWT
(access + refresh).

---

## Entrada

### Request Body (JSON)

```json
{
  "email": "juan@example.com",
  "password": "MiPassword1"
}
```

### Validaciones

| Campo      | Regla                          |
| ---------- | ------------------------------ |
| `email`    | Requerido, formato de email    |
| `password` | Requerido                      |

---

## Proceso

1. El servicio busca el usuario por email en `UserRepository`.
2. Se verifica la contraseña con `BCryptPasswordEncoder.matches(rawPassword, hashedPassword)`.
3. Se verifica que `is_active = true`.
4. Se verifica que `is_email_verified = true`.
5. Se genera el `access_token` (JWT, 15 min) con `JwtTokenProvider`.
6. Se genera el `refresh_token` (JWT, 7 días) con `JwtTokenProvider`.
7. Se registra el evento de login en `AuditLogService`.
8. Se retorna `TokenResponse` con ambos tokens.

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "accessToken": "eyJhbGci...",
  "refreshToken": "eyJhbGci...",
  "tokenType": "Bearer"
}
```

### Errores posibles

| Código | Situación                             | Mensaje                   |
| ------ | ------------------------------------- | ------------------------- |
| `400`  | Campos inválidos o vacíos             | Mapa de errores por campo |
| `401`  | Credenciales incorrectas              | `"Credenciales inválidas"` |
| `401`  | Correo no verificado                  | `"Debes verificar tu correo electrónico antes de iniciar sesión"` |
| `401`  | Cuenta inactiva                       | `"Tu cuenta está desactivada"` |
| `429`  | Rate limit excedido (Bucket4j)        | `"Demasiados intentos. Intenta más tarde"` |

---

## Reglas de negocio

1. El mensaje de error para credenciales incorrectas es genérico — nunca debe
   revelar si el correo existe o cuál campo está mal.
2. BCrypt compara automáticamente el hash almacenado sin necesidad de desencriptar.
3. El `access_token` tiene duración de 15 minutos y el `refresh_token` de 7 días.
4. Rate limiting aplicado con Bucket4j para prevenir ataques de fuerza bruta.
