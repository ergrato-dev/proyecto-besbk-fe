<!--
  ¿Qué? Requerimiento funcional para obtener el perfil del usuario autenticado.
  ¿Para qué? Especificar qué datos del usuario expone la API y cómo se protege
             el acceso.
  ¿Impacto? Define el contrato entre el frontend y el backend para mostrar la
            información del usuario en el dashboard.
-->

# RF-007 — Consulta de perfil del usuario

## Datos generales

| Campo           | Detalle                                    |
| --------------- | ------------------------------------------ |
| **ID**          | RF-007                                     |
| **Nombre**      | Obtención del perfil del usuario autenticado |
| **HU asociada** | HU-003                                     |
| **Endpoint**    | `GET /api/v1/users/me`                     |
| **Auth**        | Requerida (`Authorization: Bearer <token>`) |

---

## Descripción

Permite al usuario autenticado obtener sus datos de perfil. El endpoint usa el token JWT
para identificar al usuario sin necesitar su ID en la URL.

---

## Entrada

### Header obligatorio

```
Authorization: Bearer eyJhbGci...
```

No hay request body.

---

## Proceso

1. `JwtAuthFilter` valida el `access_token` del header `Authorization`.
2. Se extrae el `userId` del token y se carga en el `SecurityContextHolder`.
3. `UserController.getMe()` obtiene el `Authentication` del contexto de seguridad.
4. Se busca el usuario completo en `UserRepository` por ID.
5. Se mapea la entidad `User` al DTO `UserResponse` (sin `hashed_password`).
6. Se retorna el DTO.

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "id": "a1b2c3d4-...",
  "email": "juan@example.com",
  "fullName": "Juan Pérez",
  "isActive": true,
  "isEmailVerified": true,
  "createdAt": "2025-03-15T10:30:00Z"
}
```

### Errores posibles

| Código | Situación                         | Mensaje           |
| ------ | --------------------------------- | ----------------- |
| `401`  | Sin token o token inválido        | `"No autorizado"` |
| `401`  | Token expirado                    | `"Token expirado"` |
| `404`  | Usuario no encontrado (raro)      | `"Usuario no encontrado"` |

---

## Reglas de negocio

1. El campo `hashed_password` **nunca** se incluye en la respuesta.
2. El usuario solo puede acceder a su **propio** perfil — no hay forma de obtener
   el perfil de otro usuario con este endpoint.
3. Si el `access_token` expiró, el frontend debe renovarlo con `POST /api/v1/auth/refresh`
   antes de reintentar.
