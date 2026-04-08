<!--
  ¿Qué? Historia de usuario para el cierre de sesión.
  ¿Para qué? Describir cómo el usuario termina su sesión activa de forma segura,
             limpiando los tokens del cliente.
  ¿Impacto? Un logout mal implementado deja tokens activos que pueden ser reutilizados
            por un atacante si intercepta la sesión.
-->

# HU-006 — Cierre de sesión

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | HU-006                                   |
| **Nombre**      | Cierre de sesión del usuario autenticado |
| **Prioridad**   | Alta                                     |
| **Iteración**   | 2                                        |
| **Estado**      | Pendiente                                |
| **RF asociado** | RF-008                                   |

---

## Historia

**Como** usuario autenticado,
**quiero** poder cerrar mi sesión activa,
**para** que mis tokens dejen de ser válidos en este dispositivo y ningún otro usuario
pueda acceder a mi cuenta en la misma sesión.

---

## Criterios de aceptación

### CA-001 — Logout exitoso

**Dado** que el usuario hace clic en "Cerrar sesión" desde cualquier página protegida,
**cuando** se confirma la acción,
**entonces** el sistema:
1. Elimina los tokens del almacenamiento del cliente (memoria, contexto React).
2. Redirige al usuario a la página de inicio de sesión.
3. Muestra el mensaje: *"Has cerrado sesión correctamente"*.

### CA-002 — Protección post-logout

**Dado** que el usuario acaba de cerrar sesión,
**cuando** intenta acceder a una ruta protegida usando el botón "atrás" del navegador,
**entonces** el sistema redirige al login (el frontend detecta que no hay token válido).

### CA-003 — Logout sin conexión al servidor

**Dado** que el frontend implementa JWT stateless (sin revocación en servidor),
**entonces** el logout se implementa exclusivamente borrando el token del lado del
cliente. No es necesario llamar a ningún endpoint cuando el enfoque es stateless puro.

> **Nota pedagógica**: La naturaleza stateless de JWT implica que el token sigue siendo
> técnicamente válido hasta que expire (15 minutos) incluso después del logout. Esto es
> una limitación conocida de JWT que se mitiga con tokens de corta duración.

---

## Notas técnicas

- El logout es una operación **exclusivamente del lado del cliente** en este sistema
  (JWT stateless).
- El `AuthContext.tsx` en React contiene la función `logout()` que limpia el estado.
- Para una implementación más robusta (fuera del alcance de este proyecto), se usaría
  una blocklist de tokens en Redis.
