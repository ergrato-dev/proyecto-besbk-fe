<!--
  ¿Qué? Historia de usuario para las páginas legales del sistema.
  ¿Para qué? Describir el contenido mínimo requerido para las páginas de política de
             privacidad y términos de servicio.
  ¿Impacto? Proyectos reales requieren estas páginas por razones legales; incluirlas
            enseña buenas prácticas profesionales.
-->

# HU-010 — Páginas legales

## Datos generales

| Campo           | Detalle                                              |
| --------------- | ---------------------------------------------------- |
| **ID**          | HU-010                                               |
| **Nombre**      | Páginas de política de privacidad y términos         |
| **Prioridad**   | Baja                                                 |
| **Iteración**   | 3                                                    |
| **Estado**      | Pendiente                                            |
| **RF asociado** | RF-012                                               |

---

## Historia

**Como** visitante del sistema,
**quiero** poder acceder a las páginas de política de privacidad y términos de servicio,
**para** entender cómo se manejan mis datos y cuáles son las condiciones de uso del
sistema.

---

## Criterios de aceptación

### CA-001 — Página de Política de Privacidad

**Dado** que el visitante hace clic en el enlace "Política de privacidad" del footer,
**cuando** carga la página `/privacy`,
**entonces** ve una página con:
- Título claro: "Política de Privacidad".
- Secciones que cubren al menos: datos recopilados, cómo se usan los datos,
  almacenamiento y seguridad, derechos del usuario.
- Fecha de última actualización.
- Botón o enlace "Volver al inicio".

### CA-002 — Página de Términos de Servicio

**Dado** que el visitante hace clic en el enlace "Términos de servicio" del footer,
**cuando** carga la página `/terms`,
**entonces** ve una página con:
- Título claro: "Términos de Servicio".
- Secciones que cubren: aceptación de términos, descripción del servicio, uso
  aceptable, limitación de responsabilidad.
- Fecha de última actualización.
- Botón o enlace "Volver al inicio".

### CA-003 — Acceso público sin autenticación

**Dado** que ambas páginas son de naturaleza pública informativa,
**cuando** cualquier visitante (autenticado o no) accede a ellas,
**entonces** se muestran sin requerir inicio de sesión ni redirección.

### CA-004 — Consistencia visual

**Dado** que el visitante navega a cualquiera de las páginas legales,
**entonces** el diseño es consistente con el resto de la aplicación:
- Misma barra de navegación básica.
- Mismo tema claro/oscuro activo.
- Mismo footer.

---

## Notas técnicas

- Implementadas en `pages/PrivacyPolicyPage.tsx` y `pages/TermsOfServicePage.tsx`.
- El contenido es texto fijo (no proviene de la API).
- El aviso que se muestra en el sistema es para una empresa genérica ficticia "NN"
  con fines exclusivamente educativos.
