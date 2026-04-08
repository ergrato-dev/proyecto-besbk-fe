<!--
  ¿Qué? Historia de usuario para recuperación de contraseña olvidada.
  ¿Para qué? Describir el flujo de dos pasos (solicitud + restablecimiento) que permite
             a un usuario recuperar el acceso si olvidó su contraseña.
  ¿Impacto? Flujo de seguridad crítico: un error en este flujo permite a un atacante
            tomar el control de cuentas ajenas.
-->

# HU-005 — Recuperación de contraseña

## Datos generales

| Campo           | Detalle                                            |
| --------------- | -------------------------------------------------- |
| **ID**          | HU-005                                             |
| **Nombre**      | Recuperación de contraseña olvidada vía email      |
| **Prioridad**   | Alta                                               |
| **Iteración**   | 2                                                  |
| **Estado**      | Pendiente                                          |
| **RFs asociados** | RF-005, RF-006                                   |

---

## Historia

**Como** usuario registrado que olvidó su contraseña,
**quiero** poder solicitar un enlace de recuperación a mi correo y establecer una nueva
contraseña,
**para** recuperar el acceso a mi cuenta sin necesidad de contactar a un administrador.

---

## Criterios de aceptación

### Paso 1 — Solicitud del enlace de recuperación

#### CA-001 — Solicitud exitosa (correo registrado)

**Dado** que el usuario ingresa un correo que existe en el sistema,
**cuando** hace clic en "Enviar enlace de recuperación",
**entonces** el sistema:
1. Genera un token único de restablecimiento (UUID, expira en 1 hora).
2. Envía un correo con el enlace de restablecimiento al correo indicado.
3. Muestra el mensaje genérico: *"Si existe una cuenta con ese correo, recibirás un
   enlace de recuperación en breve"*.

#### CA-002 — Correo no registrado (mensaje genérico)

**Dado** que el usuario ingresa un correo que **no** existe en el sistema,
**cuando** hace clic en "Enviar enlace de recuperación",
**entonces** el sistema muestra **el mismo mensaje** que en CA-001 (respuesta genérica,
sin revelar si el correo está registrado o no).

### Paso 2 — Restablecimiento de la contraseña

#### CA-003 — Restablecimiento exitoso

**Dado** que el usuario accede al enlace del correo con un token válido y no expirado,
**cuando** ingresa una nueva contraseña válida y hace clic en "Restablecer contraseña",
**entonces** el sistema:
1. Valida que el token no esté expirado ni haya sido usado antes.
2. Actualiza la contraseña del usuario (hasheada con BCrypt).
3. Marca el token como usado (para evitar reutilización).
4. Muestra un mensaje de éxito y redirige a la página de login.

#### CA-004 — Token expirado o usado

**Dado** que el usuario accede al enlace con un token expirado o ya utilizado,
**cuando** intenta restablecer su contraseña,
**entonces** el sistema muestra el error: *"El enlace de recuperación ha expirado o ya
fue utilizado. Solicita uno nuevo"*.

---

## Notas técnicas

- Endpoints: `POST /api/v1/auth/forgot-password` y `POST /api/v1/auth/reset-password`.
- El token de restablecimiento se almacena en la tabla `password_reset_tokens`.
- El correo de recuperación se envía vía `EmailService.kt` usando `JavaMailSender`
  (capturado por Mailpit en desarrollo: `http://localhost:8025`).
- El enlace de recuperación tiene el formato:
  `{FRONTEND_URL}/reset-password?token={token}`.
