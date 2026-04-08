<!--
  ¿Qué? Requerimiento funcional para el primer paso de recuperación de contraseña
         (solicitud del enlace).
  ¿Para qué? Especificar cómo el backend genera y envía el token de restablecimiento
             sin revelar si el correo está registrado.
  ¿Impacto? Un error en este flujo puede filtrar si un correo existe en el sistema
            (enumeración de usuarios).
-->

# RF-005 — Solicitud de recuperación de contraseña

## Datos generales

| Campo           | Detalle                                        |
| --------------- | ---------------------------------------------- |
| **ID**          | RF-005                                         |
| **Nombre**      | Solicitud del enlace de recuperación de contraseña |
| **HU asociada** | HU-005                                         |
| **Endpoint**    | `POST /api/v1/auth/forgot-password`            |
| **Auth**        | No requerida                                   |

---

## Descripción

Cuando un usuario olvida su contraseña, puede solicitar un enlace de recuperación
enviando su correo electrónico. El sistema siempre responde con el mismo mensaje
genérico, independientemente de si el correo existe o no.

---

## Entrada

### Request Body (JSON)

```json
{
  "email": "juan@example.com"
}
```

### Validaciones

| Campo   | Regla                         |
| ------- | ----------------------------- |
| `email` | Requerido, formato de email   |

---

## Proceso

1. El servicio busca el usuario por email.
2. **Si el correo existe:**
   - Se invalidan tokens de restablecimiento previos no usados para ese usuario.
   - Se genera un `PasswordResetToken` (UUID, expira en 1 hora).
   - Se persiste el token en `password_reset_tokens`.
   - Se envía el correo con el enlace vía `EmailService`.
3. **Si el correo no existe:**
   - No se hace nada.
4. En ambos casos, se retorna exactamente el mismo mensaje de éxito.

---

## Salida

### Respuesta exitosa — `200 OK` (siempre, incluso si el correo no existe)

```json
{
  "message": "Si existe una cuenta con ese correo, recibirás un enlace de recuperación en breve"
}
```

### Errores posibles

| Código | Situación               | Mensaje                         |
| ------ | ----------------------- | ------------------------------- |
| `400`  | Email inválido o vacío  | Error de validación             |
| `429`  | Rate limit excedido     | `"Demasiadas solicitudes"`     |

---

## Reglas de negocio

1. La respuesta genérica es **intencional** — previene la enumeración de usuarios,
   un ataque consistente en descubrir qué correos están registrados.
2. El enlace tiene el formato: `{FRONTEND_URL}/reset-password?token={uuid}`.
3. El token expira en 1 hora como máximo.
4. Se aplica rate limiting con Bucket4j para prevenir abuso del endpoint de envío
   masivo de correos.
