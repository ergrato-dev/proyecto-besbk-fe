<!--
  ¿Qué? Requerimiento funcional para el cambio de contraseña.
  ¿Para qué? Especificar el proceso de verificación de la contraseña actual y
             actualización segura a la nueva.
  ¿Impacto? Operación de seguridad crítica: solo el usuario legítimo debe poder
            cambiar su propia contraseña.
-->

# RF-004 — Cambio de contraseña

## Datos generales

| Campo           | Detalle                                   |
| --------------- | ----------------------------------------- |
| **ID**          | RF-004                                    |
| **Nombre**      | Cambio de contraseña del usuario activo   |
| **HU asociada** | HU-004                                    |
| **Endpoint**    | `POST /api/v1/auth/change-password`       |
| **Auth**        | Requerida (`Authorization: Bearer <token>`) |

---

## Descripción

Permite a un usuario autenticado cambiar su contraseña verificando primero la contraseña
actual. La nueva contraseña se almacena hasheada.

---

## Entrada

### Request Body (JSON)

```json
{
  "currentPassword": "MiPassword1",
  "newPassword": "NuevoPassword2"
}
```

### Validaciones

| Campo             | Regla                                              |
| ----------------- | -------------------------------------------------- |
| `currentPassword` | Requerido                                          |
| `newPassword`     | Requerido, mínimo 8 caracteres, al menos 1 mayúscula, 1 minúscula y 1 número |

---

## Proceso

1. `JwtAuthFilter` intercepta la solicitud y valida el `access_token` del header.
2. El servicio extrae el usuario del `SecurityContextHolder`.
3. Se verifica que `currentPassword` coincide con `hashed_password` usando
   `BCryptPasswordEncoder.matches()`.
4. Se verifica que `newPassword` no sea igual a `currentPassword`.
5. Se hashea `newPassword` con `BCryptPasswordEncoder.encode()`.
6. Se actualiza `hashed_password` y `updated_at` en la base de datos.
7. Se registra el evento en `AuditLogService`.
8. Se retorna confirmación.

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "message": "Contraseña actualizada correctamente"
}
```

### Errores posibles

| Código | Situación                                  | Mensaje                                  |
| ------ | ------------------------------------------ | ---------------------------------------- |
| `400`  | Validaciones de Bean Validation fallidas   | Mapa de errores por campo                |
| `400`  | `currentPassword` incorrecta              | `"La contraseña actual es incorrecta"`   |
| `400`  | `newPassword` igual a `currentPassword`   | `"La nueva contraseña no puede ser igual a la actual"` |
| `401`  | Sin token o token inválido                 | `"No autorizado"`                        |

---

## Reglas de negocio

1. La sesión activa no se invalida después de cambiar la contraseña (por diseño del
   JWT stateless). En producción se recomienda invalidar todos los tokens activos.
2. La verificación de la contraseña actual **siempre** usa `BCryptPasswordEncoder.matches()`,
   nunca comparación directa de strings.
