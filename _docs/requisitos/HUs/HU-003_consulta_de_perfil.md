<!--
  ¿Qué? Historia de usuario para la consulta del perfil autenticado.
  ¿Para qué? Describir el comportamiento esperado al acceder a los datos del usuario
             autenticado mediante GET /api/v1/users/me.
  ¿Impacto? Establece qué información puede ver el usuario de su propia cuenta y
            cómo se protege ese acceso.
-->

# HU-003 — Consulta de perfil

## Datos generales

| Campo           | Detalle                                           |
| --------------- | ------------------------------------------------- |
| **ID**          | HU-003                                            |
| **Nombre**      | Consulta del perfil del usuario autenticado       |
| **Prioridad**   | Media                                             |
| **Iteración**   | 2                                                 |
| **Estado**      | Pendiente                                         |
| **RF asociado** | RF-007                                            |

---

## Historia

**Como** usuario autenticado,
**quiero** poder ver mi información de perfil (nombre, correo y fecha de registro),
**para** conocer los datos que el sistema tiene sobre mi cuenta.

---

## Criterios de aceptación

### CA-001 — Visualización del perfil

**Dado** que el usuario está autenticado y accede a la sección "Mi perfil" o al
dashboard,
**cuando** se carga la página,
**entonces** el sistema muestra:
- Nombre completo del usuario.
- Correo electrónico.
- Fecha de registro (en formato legible, ej: "12 de marzo de 2025").
- Estado de verificación del correo (verificado / no verificado).

### CA-002 — Acceso sin autenticación

**Dado** que un visitante no autenticado intenta acceder directamente a la URL del
dashboard o perfil,
**cuando** carga la página,
**entonces** el sistema lo redirige automáticamente a la página de inicio de sesión.

### CA-003 — Token expirado

**Dado** que el usuario tenía una sesión activa pero su `access_token` ha expirado (15
min),
**cuando** el frontend detecta el error `401 Unauthorized`,
**entonces** el sistema intenta renovar el token automáticamente usando el `refresh_token`
sin que el usuario tenga que volver a iniciar sesión.

### CA-004 — Ausencia de datos sensibles

**Dado** que el sistema muestra el perfil del usuario,
**entonces** en ningún caso se muestra:
- La contraseña (ni hasheada ni en texto plano).
- El `refresh_token`.
- Campos internos del sistema (`hashed_password`, flags internos de BD).

---

## Notas técnicas

- El endpoint del perfil es `GET /api/v1/users/me`.
- El endpoint requiere el header `Authorization: Bearer <access_token>`.
- El filtro `JwtAuthFilter.kt` valida el token y carga el usuario en el `SecurityContext`.
- El `UserController.kt` resuelve el usuario desde `SecurityContextHolder`.
- La respuesta es un DTO `UserResponse` que excluye explícitamente `hashed_password`.
