<!--
  ¿Qué? Historia de usuario para el formulario de contacto.
  ¿Para qué? Describir cómo los visitantes pueden enviar mensajes al equipo a través
             de la aplicación, usando el backend de email.
  ¿Impacto? Permite demostrar el uso de JavaMailSender en un contexto diferente al de
            autenticación, reforzando el aprendizaje de la herramienta.
-->

# HU-011 — Formulario de contacto

## Datos generales

| Campo           | Detalle                                          |
| --------------- | ------------------------------------------------ |
| **ID**          | HU-011                                           |
| **Nombre**      | Formulario de contacto para visitantes           |
| **Prioridad**   | Baja                                             |
| **Iteración**   | 3                                                |
| **Estado**      | Pendiente                                        |
| **RF asociado** | RF-013                                           |

---

## Historia

**Como** visitante del sistema,
**quiero** poder enviar un mensaje de contacto proporcionando mi nombre, correo y
mensaje,
**para** comunicarme con el equipo de desarrollo o soporte.

---

## Criterios de aceptación

### CA-001 — Envío exitoso del formulario

**Dado** que el visitante completa todos los campos del formulario con datos válidos,
**cuando** hace clic en "Enviar mensaje",
**entonces** el sistema:
1. Envía el mensaje a la dirección de correo configurada en el backend.
2. Muestra el mensaje de éxito: *"Tu mensaje ha sido enviado. Te responderemos
   a la brevedad"*.
3. Limpia el formulario después del envío exitoso.

### CA-002 — Validaciones del formulario

**Dado** que el visitante intenta enviar el formulario con campos vacíos o inválidos,
**cuando** hace clic en "Enviar mensaje",
**entonces** el sistema muestra mensajes de error específicos:

| Campo   | Regla de validación                      | Mensaje de error                          |
| ------- | ---------------------------------------- | ----------------------------------------- |
| Nombre  | Requerido, mínimo 2 caracteres           | "El nombre es requerido"                  |
| Correo  | Requerido, formato válido de email       | "Ingresa un correo electrónico válido"    |
| Mensaje | Requerido, mínimo 10 caracteres          | "El mensaje debe tener al menos 10 caracteres" |

### CA-003 — Prevención de spam

**Dado** que el endpoint de contacto está expuesto públicamente,
**cuando** se realizan múltiples solicitudes en corto tiempo desde la misma IP,
**entonces** el rate limiting de Bucket4j bloquea las solicitudes excesivas con un
error `429 Too Many Requests`.

### CA-004 — Acceso público

**Dado** que el formulario está en la sección "Contacto" de la landing page o en
`/contact`,
**entonces** es accesible para cualquier visitante sin necesidad de autenticación.

---

## Notas técnicas

- El envío de correo se realiza mediante `JavaMailSender` de Spring (capturado por
  Mailpit en desarrollo: `http://localhost:8025`).
- El endpoint del formulario de contacto es `POST /api/v1/contact`.
- Rate limiting aplicado con Bucket4j para proteger el endpoint público.
- Implementado en `pages/ContactPage.tsx` (frontend) y un controlador adicional en el
  backend.
