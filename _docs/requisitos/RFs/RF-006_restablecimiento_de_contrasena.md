<!--
  ¿Qué? Requerimiento funcional para el segundo paso de recuperación de contraseña
         (restablecimiento con el token recibido por correo).
  ¿Para qué? Especificar cómo el backend valida el token y actualiza la contraseña.
  ¿Impacto? Un token que pueda reutilizarse o que no expire permitiría a un atacante
            con acceso al correo tomar control de la cuenta indefinidamente.
-->

# RF-006 — Restablecimiento de contraseña

## Datos generales

| Campo           | Detalle                                     |
| --------------- | ------------------------------------------- |
| **ID**          | RF-006                                      |
| **Nombre**      | Restablecimiento de contraseña con token    |
| **HU asociada** | HU-005                                      |
| **Endpoint**    | `POST /api/v1/auth/reset-password`          |
| **Auth**        | No requerida (usa token de restablecimiento) |

---

## Descripción

Con el token recibido por correo, el usuario puede establecer una nueva contraseña. El
sistema verifica que el token sea válido, no haya expirado y no haya sido usado antes.

---

## Entrada

### Request Body (JSON)

```json
{
  "token": "550e8400-e29b-41d4-a716-446655440000",
  "newPassword": "NuevaPassword3"
}
```

### Validaciones

| Campo         | Regla                                              |
| ------------- | -------------------------------------------------- |
| `token`       | Requerido                                          |
| `newPassword` | Requerido, mínimo 8 caracteres, al menos 1 mayúscula, 1 minúscula y 1 número |

---

## Proceso

1. El servicio busca el `PasswordResetToken` en la base de datos por `token`.
2. Se verifica que el token exista.
3. Se verifica que el token no haya expirado (`expires_at > now()`).
4. Se verifica que el token no haya sido usado (`used = false`).
5. Se hashea `newPassword` con `BCryptPasswordEncoder.encode()`.
6. Se actualiza `hashed_password` y `updated_at` del usuario.
7. Se marca el token como `used = true`.
8. Se retorna confirmación.

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "message": "Contraseña restablecida correctamente. Ya puedes iniciar sesión"
}
```

### Errores posibles

| Código | Situación                                            | Mensaje                                   |
| ------ | ---------------------------------------------------- | ----------------------------------------- |
| `400`  | Validaciones fallidas                                | Mapa de errores por campo                 |
| `400`  | Token no encontrado, expirado o ya usado             | `"El enlace de recuperación ha expirado o ya fue utilizado. Solicita uno nuevo"` |

---

## Reglas de negocio

1. Un token solo puede usarse **una vez** (`used = true` después del restablecimiento).
2. El token expira en 1 hora desde su generación.
3. El mensaje de error para token inválido/expirado/usado es el mismo — no se
   especifica la razón exacta del rechazo.
4. Después del restablecimiento exitoso, el usuario debe iniciar sesión manualmente
   con la nueva contraseña.
