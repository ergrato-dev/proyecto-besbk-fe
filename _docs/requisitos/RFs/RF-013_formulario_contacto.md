<!--
  ¿Qué? Requerimiento funcional para el formulario de contacto.
  ¿Para qué? Especificar el endpoint de backend y el comportamiento del frontend
             para el envío de mensajes de contacto.
  ¿Impacto? Demuestra el uso de JavaMailSender fuera del contexto de auth y aplica
            rate limiting en un endpoint público para prevenir spam.
-->

# RF-013 — Formulario de contacto

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-013                                   |
| **Nombre**      | Formulario de contacto para visitantes   |
| **HU asociada** | HU-011                                   |
| **Endpoint**    | `POST /api/v1/contact`                   |
| **Auth**        | No requerida                             |

---

## Descripción

Permite a cualquier visitante enviar un mensaje de contacto. El backend procesa el
formulario y envía el mensaje al correo de administración del sistema. Se aplica rate
limiting para prevenir spam.

---

## Entrada

### Request Body (JSON)

```json
{
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "message": "Hola, tengo una pregunta sobre el sistema."
}
```

### Validaciones (Bean Validation)

| Campo     | Anotación                           | Regla                               |
| --------- | ----------------------------------- | ----------------------------------- |
| `name`    | `@NotBlank`, `@Size(min=2, max=100)` | Requerido, 2–100 caracteres        |
| `email`   | `@NotBlank`, `@Email`               | Requerido, formato de email válido  |
| `message` | `@NotBlank`, `@Size(min=10, max=1000)` | Requerido, 10–1000 caracteres   |

---

## Proceso

1. El controlador recibe el DTO `ContactRequest` y ejecuta `@Valid`.
2. El servicio usa `EmailService` para enviar el correo con el contenido del formulario.
3. El correo de destino es la dirección configurada en variables de entorno
   (`CONTACT_EMAIL`).
4. Se retorna confirmación al cliente.

---

## Salida

### Respuesta exitosa — `200 OK`

```json
{
  "message": "Tu mensaje ha sido enviado. Te responderemos a la brevedad"
}
```

### Errores posibles

| Código | Situación                          | Mensaje                           |
| ------ | ---------------------------------- | --------------------------------- |
| `400`  | Validaciones de Bean Validation    | Mapa de errores por campo         |
| `429`  | Rate limit excedido (Bucket4j)     | `"Demasiadas solicitudes. Intenta más tarde"` |
| `500`  | Error al enviar el correo          | `"Error al enviar el mensaje. Intenta más tarde"` |

---

## Reglas de negocio

1. Rate limiting con Bucket4j aplicado por IP: máximo 5 mensajes por hora.
2. El mensaje se envía vía `JavaMailSender` (capturado por Mailpit en desarrollo:
   `http://localhost:8025`).
3. El correo de destino se configura en variables de entorno (`CONTACT_EMAIL`), nunca
   hardcodeado.
4. El cuerpo del correo incluye el nombre del remitente, su email y el mensaje.
