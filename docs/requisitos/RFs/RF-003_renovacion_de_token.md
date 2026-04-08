<!--
  ¿Qué? Requerimiento funcional para la renovación de tokens JWT.
  ¿Para qué? Especificar cómo el sistema emite un nuevo access_token sin que el usuario
             tenga que volver a iniciar sesión.
  ¿Impacto? Permite mantener la sesión activa de forma transparente, balanceando
            seguridad (tokens cortos) y usabilidad (sin re-login frecuente).
-->

# RF-003 — Renovación de token

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-003                                   |
| **Nombre**      | Renovación de access token con refresh   |
| **HU asociada** | HU-002, HU-008                           |
| **Endpoint**    | `POST /api/v1/auth/refresh`              |
| **Auth**        | Token de refresh (no acceso estándar)    |

---

## Descripción

Cuando el `access_token` expira, el frontend puede obtener uno nuevo enviando el
`refresh_token` válido sin necesidad de que el usuario vuelva a introducir sus
credenciales.

---

## Entrada

### Request Body (JSON)

```json
{
  "refreshToken": "eyJhbGci..."
}
```

---

## Proceso

1. El servicio recibe el `refresh_token`.
2. `JwtTokenProvider` verifica la firma y la expiración del token.
3. Se extrae el `userId` (subject) del token.
4. Se verifica que el usuario existe y sigue activo en la base de datos.
5. Se genera un nuevo `access_token` (15 min).
6. Se retorna el nuevo `access_token` (el `refresh_token` no cambia).

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "accessToken": "eyJhbGci...",
  "tokenType": "Bearer"
}
```

### Errores posibles

| Código | Situación                              | Mensaje                              |
| ------ | -------------------------------------- | ------------------------------------ |
| `400`  | Campo `refreshToken` ausente           | Error de validación                  |
| `401`  | Token inválido o malformado            | `"Token inválido"`                   |
| `401`  | Token expirado                         | `"La sesión ha expirado. Inicia sesión nuevamente"` |
| `401`  | Usuario no encontrado o inactivo       | `"Token inválido"`                   |

---

## Reglas de negocio

1. Los mensajes de error para tokens inválidos son genéricos — no revelan la razón
   exacta del rechazo.
2. El `refresh_token` tiene una duración de 7 días.
3. El `refresh_token` no se rota en esta implementación (refresh token rotation está
   fuera del alcance educativo).
4. El interceptor de Axios en el frontend llama automáticamente a este endpoint cuando
   detecta un `401` en cualquier llamada a la API.
