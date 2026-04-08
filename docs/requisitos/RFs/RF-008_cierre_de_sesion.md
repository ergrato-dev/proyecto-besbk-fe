<!--
  ¿Qué? Requerimiento funcional para el cierre de sesión.
  ¿Para qué? Especificar la naturaleza del logout en un sistema JWT stateless y
             cómo se implementa en el frontend.
  ¿Impacto? Entender que el logout en JWT no invalida el token en el servidor es
            una lección de seguridad importante para los aprendices.
-->

# RF-008 — Cierre de sesión

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-008                                   |
| **Nombre**      | Cierre de sesión del usuario autenticado |
| **HU asociada** | HU-006                                   |
| **Endpoint**    | N/A (operación del lado del cliente)     |
| **Auth**        | No aplica                                |

---

## Descripción

En un sistema JWT stateless, el logout se implementa eliminando los tokens del lado
del cliente. No existe un endpoint de logout en el backend porque el servidor no
mantiene estado de sesión.

---

## Proceso (Frontend)

1. El usuario hace clic en "Cerrar sesión".
2. El `AuthContext` ejecuta la función `logout()`.
3. Se elimina el `access_token` de la memoria del contexto React.
4. Se elimina el `refresh_token` del almacenamiento del cliente.
5. Se limpian los datos del usuario del estado global.
6. React Router redirige a `/login`.
7. Se muestra una notificación de éxito: *"Has cerrado sesión correctamente"*.

---

## Consideraciones de seguridad

### Limitación conocida del JWT stateless

Después del logout del cliente, el `access_token` sigue siendo técnicamente válido
en el servidor hasta que expire (15 minutos). Esta es una limitación intrínseca de JWT.

**Mitigaciones implementadas:**
- Access token de corta duración (15 minutos).
- Refresh token almacenado de forma más segura que en `localStorage`.

**Mitigación no implementada (fuera del alcance):**
- Token blocklist (lista de tokens revocados en Redis).

---

## Reglas de negocio

1. Después del logout exitoso, cualquier intento de acceder a una ruta protegida
   redirige al login.
2. El botón "atrás" del navegador no puede restaurar la sesión cerrada.
3. Al cerrar sesión, se borran todos los tokens del estado del cliente.

---

> **Nota pedagógica**: La diferencia entre sesiones de servidor (stateful) y
> JWT (stateless) es uno de los conceptos más importantes de este proyecto.
> Las sesiones de servidor pueden revocarse instantáneamente; los JWT no,
> a menos que implementes una lista de revocación.
