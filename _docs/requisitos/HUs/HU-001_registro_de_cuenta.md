<!--
  ¿Qué? Historia de usuario para el registro de una nueva cuenta.
  ¿Para qué? Describir el comportamiento esperado desde la perspectiva del usuario final,
             para guiar el diseño del endpoint POST /api/v1/auth/register.
  ¿Impacto? Define el criterio de aceptación que debe cumplir la implementación backend
            y frontend antes de considerar esta funcionalidad completa.
-->

# HU-001 — Registro de cuenta

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | HU-001                                   |
| **Nombre**      | Registro de nueva cuenta de usuario      |
| **Prioridad**   | Alta                                     |
| **Iteración**   | 1                                        |
| **Estado**      | Pendiente                                |
| **RF asociado** | RF-001                                   |

---

## Historia

**Como** visitante no registrado,
**quiero** poder crear una cuenta proporcionando mi nombre completo, correo electrónico y
contraseña,
**para** acceder a las funcionalidades protegidas del sistema NN Auth.

---

## Criterios de aceptación

### CA-001 — Registro exitoso

**Dado** que el visitante completa el formulario con datos válidos (nombre, correo y
contraseña que cumplen los requisitos mínimos),
**cuando** hace clic en "Crear cuenta",
**entonces** el sistema:
1. Crea la cuenta en la base de datos con la contraseña hasheada.
2. Envía un correo de verificación a la dirección proporcionada.
3. Muestra un mensaje de éxito indicando que debe verificar su correo.
4. No inicia sesión automáticamente (la verificación es obligatoria).

### CA-002 — Correo duplicado

**Dado** que el visitante ingresa un correo que ya está registrado en el sistema,
**cuando** intenta registrarse,
**entonces** el sistema muestra un mensaje de error: *"Este correo electrónico ya está
registrado"*.

### CA-003 — Datos inválidos

**Dado** que el visitante intenta registrarse con datos que no cumplen las validaciones,
**cuando** hace clic en "Crear cuenta",
**entonces** el sistema muestra mensajes de error específicos junto a cada campo
incorrecto sin limpiar los valores válidos.

| Caso                               | Mensaje esperado                                        |
| ---------------------------------- | ------------------------------------------------------- |
| Nombre vacío                       | "El nombre completo es requerido"                       |
| Correo con formato inválido        | "Ingresa un correo electrónico válido"                  |
| Contraseña menor a 8 caracteres    | "La contraseña debe tener al menos 8 caracteres"        |
| Contraseña sin mayúscula           | "Debe incluir al menos una letra mayúscula"             |
| Contraseña sin minúscula           | "Debe incluir al menos una letra minúscula"             |
| Contraseña sin número              | "Debe incluir al menos un número"                       |

### CA-004 — Accesibilidad del formulario

**Dado** que el visitante abre la página de registro,
**entonces** todos los campos del formulario tienen:
- Etiqueta `<label>` asociada correctamente.
- Atributo `aria-label` o `aria-describedby` donde aplique.
- Soporte para navegación con teclado (Tab, Enter).
- Atributo `autocomplete` adecuado (`name`, `email`, `new-password`).

---

## Notas técnicas

- El endpoint de registro es `POST /api/v1/auth/register`.
- El backend implementado en Kotlin + Spring Boot válida los datos con Bean Validation
  (`@Valid`, `@NotBlank`, `@Email`).
- La contraseña **nunca** se almacena en texto plano; se hashea con `BCryptPasswordEncoder`
  antes de persistir.
- El correo de verificación se envía a través de `JavaMailSender`, capturado en Mailpit
  durante el desarrollo (`http://localhost:8025`).
- El token de verificación de correo expira en 24 horas.
