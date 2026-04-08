<!--
  ¿Qué? Historia de usuario para la protección de rutas del frontend.
  ¿Para qué? Describir cómo el sistema impide el acceso a secciones protegidas
             a usuarios no autenticados, sin depender únicamente del backend.
  ¿Impacto? Sin protección de rutas, un usuario podría ver páginas protegidas aunque
            su token sea inválido o haya expirado.
-->

# HU-008 — Protección de rutas

## Datos generales

| Campo           | Detalle                                              |
| --------------- | ---------------------------------------------------- |
| **ID**          | HU-008                                               |
| **Nombre**      | Protección de rutas privadas en el frontend          |
| **Prioridad**   | Alta                                                 |
| **Iteración**   | 2                                                    |
| **Estado**      | Pendiente                                            |
| **RF asociado** | RF-009                                               |

---

## Historia

**Como** sistema de autenticación,
**quiero** que las páginas privadas (dashboard, perfil, cambio de contraseña) solo sean
accesibles para usuarios con sesión activa válida,
**para** evitar que visitantes o usuarios con tokens expirados accedan a información
protegida.

---

## Criterios de aceptación

### CA-001 — Redirección de usuario no autenticado

**Dado** que un visitante no autenticado intenta acceder directamente a una URL
protegida (ej: `/dashboard`, `/change-password`),
**cuando** el React Router evalúa la ruta,
**entonces** el sistema lo redirige automáticamente a `/login` sin mostrar el contenido
protegido ni por un instante.

### CA-002 — Redirección de usuario autenticado hacia login

**Dado** que un usuario ya autenticado intenta acceder a `/login` o `/register`,
**cuando** el React Router evalúa la ruta,
**entonces** el sistema lo redirige automáticamente al `/dashboard` para evitar
confusión.

### CA-003 — Verificación del token

**Dado** que el frontend tiene un token almacenado en memoria,
**cuando** el usuario navega a una ruta protegida,
**entonces** el componente `ProtectedRoute` verifica que el token exista y no haya
expirado antes de permitir el renderizado.

### CA-004 — Manejo de token expirado durante la sesión

**Dado** que el usuario está en una página protegida y su `access_token` expira (15 min),
**cuando** el interceptor de Axios detecta un error `401` al hacer una llamada a la API,
**entonces** el sistema intenta renovar el token con el `refresh_token` automáticamente,
y si eso también falla, cierra la sesión y redirige al login.

---

## Notas técnicas

- Se implementa con un componente `ProtectedRoute.tsx` que envuelve las rutas privadas
  en `App.tsx`.
- El estado de autenticación se obtiene del `AuthContext`.
- El interceptor de Axios en `api/axios.ts` maneja la renovación automática de tokens
  usando `POST /api/v1/auth/refresh`.
