<!--
  ¿Qué? Requerimiento funcional para las páginas legales del sistema.
  ¿Para qué? Especificar el contenido mínimo requerido en las páginas de política
             de privacidad y términos de servicio.
  ¿Impacto? Son páginas obligatorias en sistemas reales que manejan datos personales.
            Incluirlas enseña buenas prácticas profesionales a los aprendices.
-->

# RF-012 — Páginas legales

## Datos generales

| Campo           | Detalle                                           |
| --------------- | ------------------------------------------------- |
| **ID**          | RF-012                                            |
| **Nombre**      | Páginas de política de privacidad y términos      |
| **HU asociada** | HU-010                                            |
| **Tipo**        | Frontend — sin endpoint de API asociado           |

---

## Descripción

El sistema debe incluir dos páginas de contenido legal estático: Política de Privacidad
y Términos de Servicio. Ambas son accesibles públicamente y referenciadas desde el footer.

---

## Página 1 — Política de Privacidad (`/privacy`)

### Secciones obligatorias

| Sección                           | Contenido mínimo                                            |
| --------------------------------- | ----------------------------------------------------------- |
| 1. Introducción                   | Quién recopila los datos y propósito del documento          |
| 2. Datos que recopilamos          | Email, nombre completo, fecha de registro                   |
| 3. Cómo usamos los datos          | Autenticación, comunicaciones del sistema                   |
| 4. Almacenamiento y seguridad     | Base de datos PostgreSQL, contraseñas hasheadas con BCrypt  |
| 5. Derechos del usuario           | Acceso, rectificación, eliminación de datos                 |
| 6. Cookies                        | Solo se usan para el refresh_token (httpOnly)               |
| 7. Contacto                       | Enlace al formulario de contacto                            |

### Metadatos

- Título: "Política de Privacidad — NN Auth"
- Fecha de última actualización visible en la página

---

## Página 2 — Términos de Servicio (`/terms`)

### Secciones obligatorias

| Sección                          | Contenido mínimo                                             |
| -------------------------------- | ------------------------------------------------------------ |
| 1. Aceptación de términos        | Al usar el sistema, el usuario acepta las condiciones        |
| 2. Descripción del servicio      | Sistema de autenticación educativo / sin garantías de producción |
| 3. Uso aceptable                 | No usar para actividades ilegales, no realizar ingeniería inversa |
| 4. Cuentas de usuario            | Responsabilidad del usuario sobre su cuenta y contraseña    |
| 5. Limitación de responsabilidad | Proyecto educativo SENA, sin garantías de disponibilidad     |
| 6. Cambios en los términos       | El sistema puede actualizar los términos con notificación    |
| 7. Contacto                      | Enlace al formulario de contacto                            |

### Metadatos

- Título: "Términos de Servicio — NN Auth"
- Fecha de última actualización visible en la página

---

## Requisitos técnicos comunes

- Ambas páginas son completamente **estáticas** (sin API).
- Accesibles sin autenticación.
- Incluyen la barra de navegación y el footer estándar.
- Respetan el tema claro/oscuro activo.
- Botón "Volver al inicio" que navega a `/`.

---

## Reglas de negocio

1. El contenido es para la empresa ficticia "NN" con fines exclusivamente educativos.
2. El disclaimer "Proyecto educativo — SENA" debe estar presente en ambas páginas.
3. Las páginas no requieren inicio de sesión para ser leídas.
