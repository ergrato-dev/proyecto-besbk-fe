<!--
  ¿Qué? Requerimiento funcional para el registro de nuevos usuarios.
  ¿Para qué? Especificar con precisión técnica cómo el backend procesa el registro,
             validaciones, hashing y respuesta esperada.
  ¿Impacto? Define el contrato técnico entre frontend y backend para esta operación.
-->

# RF-001 — Registro de usuario

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-001                                   |
| **Nombre**      | Registro de nueva cuenta de usuario      |
| **HU asociada** | HU-001                                   |
| **Endpoint**    | `POST /api/v1/auth/register`             |
| **Auth**        | No requerida                             |

---

## Descripción

El sistema debe permitir registrar nuevos usuarios mediante el envío de nombre completo,
correo electrónico y contraseña. El backend valida los datos, hashea la contraseña y
persiste el usuario en la base de datos.

---

## Entrada

### Request Body (JSON)

```json
{
  "fullName": "Juan Pérez",
  "email": "juan@example.com",
  "password": "MiPassword1"
}
```

### Validaciones (Bean Validation — Jakarta)

| Campo      | Anotación             | Regla                                              |
| ---------- | --------------------- | -------------------------------------------------- |
| `fullName` | `@NotBlank`           | Requerido, no puede estar vacío ni ser solo espacios |
| `fullName` | `@Size(min=2, max=100)` | Entre 2 y 100 caracteres                         |
| `email`    | `@NotBlank`, `@Email` | Requerido y con formato de email válido            |
| `password` | `@NotBlank`           | Requerido                                          |
| `password` | `@Size(min=8)`        | Mínimo 8 caracteres                               |
| `password` | `@Pattern`            | Al menos 1 mayúscula, 1 minúscula y 1 número      |

---

## Proceso

1. El controlador recibe el DTO `RegisterRequest` y ejecuta `@Valid` automáticamente.
2. El servicio verifica que el correo no esté registrado en `users` (via `UserRepository`).
3. La contraseña se hashea con `BCryptPasswordEncoder.encode()` — **nunca** en texto plano.
4. Se crea el registro en la tabla `users` con `is_email_verified = false`.
5. Se genera un `EmailVerificationToken` (UUID, expira en 24h) y se persiste.
6. Se envía el correo de verificación vía `EmailService` usando `JavaMailSender`.
7. Se construye y retorna la respuesta `UserResponse`.

---

## Salida

### Respuesta exitosa — `201 Created`

```json
{
  "id": "a1b2c3d4-...",
  "email": "juan@example.com",
  "fullName": "Juan Pérez",
  "isActive": true,
  "isEmailVerified": false,
  "createdAt": "2025-03-15T10:30:00Z"
}
```

### Errores posibles

| Código | Situación                          | Mensaje                                       |
| ------ | ---------------------------------- | --------------------------------------------- |
| `400`  | Datos de entrada inválidos         | Mapa de errores por campo (Bean Validation)   |
| `409`  | Correo ya registrado               | `"Este correo electrónico ya está registrado"` |
| `500`  | Error interno                      | `"Error interno del servidor"`               |

---

## Reglas de negocio

1. La contraseña **siempre** se hashea con BCrypt antes de almacenar.
2. El correo debe verificarse antes de poder iniciar sesión.
3. El correo de bienvenida/verificación se envía independientemente del entorno
   (capturado por Mailpit en desarrollo).
4. El campo `hashed_password` **nunca** se incluye en ninguna respuesta de la API.
