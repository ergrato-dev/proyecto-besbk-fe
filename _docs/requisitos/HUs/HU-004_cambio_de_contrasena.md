<!--
  ¿Qué? Historia de usuario para el cambio de contraseña.
  ¿Para qué? Describir el flujo que permite a un usuario autenticado cambiar su
             contraseña actual por una nueva, vía POST /api/v1/auth/change-password.
  ¿Impacto? Es una función de seguridad crítica: protege contra accesos no autorizados
            si la contraseña actual fue comprometida.
-->

# HU-004 — Cambio de contraseña

## Datos generales

| Campo           | Detalle                                           |
| --------------- | ------------------------------------------------- |
| **ID**          | HU-004                                            |
| **Nombre**      | Cambio de contraseña del usuario autenticado      |
| **Prioridad**   | Alta                                              |
| **Iteración**   | 2                                                 |
| **Estado**      | Pendiente                                         |
| **RF asociado** | RF-004                                            |

---

## Historia

**Como** usuario autenticado,
**quiero** poder cambiar mi contraseña ingresando mi contraseña actual y la nueva,
**para** mantener la seguridad de mi cuenta.

---

## Criterios de aceptación

### CA-001 — Cambio exitoso

**Dado** que el usuario está autenticado y proporciona su contraseña actual correcta y
una nueva contraseña válida,
**cuando** hace clic en "Cambiar contraseña",
**entonces** el sistema:
1. Actualiza la contraseña en la base de datos (almacenada con BCrypt).
2. Muestra un mensaje de éxito: *"Tu contraseña ha sido actualizada correctamente"*.
3. No cierra la sesión activa del usuario.

### CA-002 — Contraseña actual incorrecta

**Dado** que el usuario proporciona una contraseña actual que no coincide con la
almacenada,
**cuando** intenta el cambio,
**entonces** el sistema muestra el error: *"La contraseña actual es incorrecta"*.

### CA-003 — Nueva contraseña igual a la actual

**Dado** que el usuario proporciona la misma contraseña tanto en el campo "Actual" como
en "Nueva",
**cuando** intenta el cambio,
**entonces** el sistema muestra el error: *"La nueva contraseña no puede ser igual a la
actual"*.

### CA-004 — Validaciones de la nueva contraseña

**Dado** que el usuario introduce una nueva contraseña que no cumple los requisitos,
**cuando** intenta el cambio,
**entonces** el sistema muestra el mensaje de error correspondiente antes de enviar la
petición:

| Caso                               | Mensaje esperado                                 |
| ---------------------------------- | ------------------------------------------------ |
| Menos de 8 caracteres              | "La contraseña debe tener al menos 8 caracteres" |
| Sin letra mayúscula                | "Debe incluir al menos una letra mayúscula"      |
| Sin letra minúscula                | "Debe incluir al menos una letra minúscula"      |
| Sin número                         | "Debe incluir al menos un número"                |

### CA-005 — Acceso sin autenticación

**Dado** que un visitante no autenticado intenta acceder directamente a la URL de cambio
de contraseña,
**cuando** intenta cargar la página,
**entonces** es redirigido automáticamente a la página de inicio de sesión.

---

## Notas técnicas

- El endpoint es `POST /api/v1/auth/change-password`.
- Requiere el header `Authorization: Bearer <access_token>` (ruta protegida por
  Spring Security).
- El `AuthService.kt` verifica la contraseña actual con `BCryptPasswordEncoder.matches()`
  y almacena la nueva usando `BCryptPasswordEncoder.encode()`.
