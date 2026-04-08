<!--
  ¿Qué? Historia de usuario para el inicio de sesión.
  ¿Para qué? Describir el comportamiento esperado desde la perspectiva del usuario,
             para guiar el diseño del endpoint POST /api/v1/auth/login.
  ¿Impacto? Define el criterio de aceptación que debe cumplir la implementación antes
            de considerar esta funcionalidad completa.
-->

# HU-002 — Inicio de sesión

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | HU-002                                   |
| **Nombre**      | Inicio de sesión con correo y contraseña |
| **Prioridad**   | Alta                                     |
| **Iteración**   | 1                                        |
| **Estado**      | Pendiente                                |
| **RF asociado** | RF-002                                   |

---

## Historia

**Como** usuario registrado y con correo verificado,
**quiero** poder iniciar sesión ingresando mi correo electrónico y contraseña,
**para** acceder al área protegida del sistema.

---

## Criterios de aceptación

### CA-001 — Login exitoso

**Dado** que el usuario ingresa un correo y contraseña válidos de una cuenta existente
con correo verificado,
**cuando** hace clic en "Iniciar sesión",
**entonces** el sistema:
1. Devuelve un `access_token` (duración: 15 minutos) y un `refresh_token` (duración: 7 días).
2. Redirige al usuario al dashboard principal.
3. El nombre del usuario aparece en la barra de navegación.

### CA-002 — Credenciales incorrectas

**Dado** que el usuario ingresa una combinación de correo/contraseña incorrecta,
**cuando** intenta iniciar sesión,
**entonces** el sistema:
1. Muestra el mensaje genérico: *"Credenciales inválidas"*.
2. **No** especifica si el error es en el correo o en la contraseña (para no filtrar
   información sobre qué cuentas existen).
3. No limpia el campo de correo (pero sí limpia la contraseña).

### CA-003 — Correo no verificado

**Dado** que el usuario tiene una cuenta pero no ha verificado su correo,
**cuando** intenta iniciar sesión con credenciales correctas,
**entonces** el sistema muestra un mensaje indicando que debe verificar su correo antes
de poder iniciar sesión.

### CA-004 — Campos vacíos

**Dado** que el usuario intenta iniciar sesión con campos vacíos,
**cuando** hace clic en "Iniciar sesión",
**entonces** el sistema muestra mensajes de validación en cada campo sin enviar la
petición al servidor.

### CA-005 — Rate limiting

**Dado** que un atacante intenta múltiples combinaciones de contraseña,
**cuando** supera el límite de intentos fallidos (configurado con Bucket4j),
**entonces** el sistema bloquea temporalmente las solicitudes de ese cliente y responde
con un error `429 Too Many Requests`.

---

## Notas técnicas

- El endpoint de login es `POST /api/v1/auth/login`.
- Los tokens JWT se generan en `JwtTokenProvider.kt` usando la librería JJWT.
- En el frontend, el `access_token` se almacena en memoria (no en `localStorage`).
- El `refresh_token` puede almacenarse en una cookie `httpOnly` para mayor seguridad.
- El rate limiting se implementa con **Bucket4j** para prevenir ataques de fuerza bruta.
