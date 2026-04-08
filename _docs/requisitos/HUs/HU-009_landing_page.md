<!--
  ¿Qué? Historia de usuario para la landing page pública.
  ¿Para qué? Describir el contenido y estructura de la página principal que ven los
             visitantes antes de registrarse o iniciar sesión.
  ¿Impacto? Es la primera impresión del sistema; debe comunicar claramente el propósito
            y las características del producto.
-->

# HU-009 — Landing Page

## Datos generales

| Campo           | Detalle                                          |
| --------------- | ------------------------------------------------ |
| **ID**          | HU-009                                           |
| **Nombre**      | Landing page pública del sistema                 |
| **Prioridad**   | Media                                            |
| **Iteración**   | 3                                                |
| **Estado**      | Pendiente                                        |
| **RF asociado** | RF-011                                           |

---

## Historia

**Como** visitante que llega por primera vez al sistema NN Auth,
**quiero** ver una página de inicio informativa que explique el propósito del sistema y
me ofrezca acceso fácil al registro e inicio de sesión,
**para** entender qué ofrece el sistema antes de crear una cuenta.

---

## Criterios de aceptación

### CA-001 — Sección héroe (Hero)

**Dado** que el visitante accede a la raíz de la aplicación (`/`),
**cuando** carga la página,
**entonces** ve una sección héroe con:
- Título principal claro sobre el propósito del sistema.
- Subtítulo descriptivo.
- Dos botones de llamada a la acción: "Crear cuenta" y "Iniciar sesión".

### CA-002 — Sección de características

**Dado** que el visitante hace scroll hacia abajo,
**cuando** llega a la sección de características,
**entonces** ve al menos 3 tarjetas con las características principales del sistema
(autenticación segura, JWT, hashing de contraseñas, etc.), cada una con:
- Icono representativo (de `lucide-react`).
- Título corto.
- Descripción breve.

### CA-003 — Navbar de la landing

**Dado** que el visitante está en la landing page,
**cuando** observa la barra de navegación,
**entonces** ve:
- El logo/nombre "NN Auth" a la izquierda.
- Enlace a la sección de características.
- Enlace a "Contacto".
- Botones "Iniciar sesión" y "Crear cuenta" a la derecha.
- Botón de cambio de tema (claro/oscuro).

### CA-004 — Footer

**Dado** que el visitante llega al final de la landing page,
**cuando** ve el footer,
**entonces** encuentra:
- Nombre y descripción breve del sistema.
- Enlace a "Política de privacidad".
- Enlace a "Términos de servicio".
- Texto de derechos: "© 2025 NN — SENA. Proyecto educativo."

---

## Notas técnicas

- Esta página es completamente pública, no requiere autenticación.
- Si el usuario ya está autenticado y accede a `/`, se le redirige al `/dashboard`.
- Implementada en `pages/LandingPage.tsx`.
- Sin dependencias de backend — es contenido estático.
