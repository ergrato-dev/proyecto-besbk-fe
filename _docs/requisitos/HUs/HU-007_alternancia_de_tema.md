<!--
  ¿Qué? Historia de usuario para la alternancia entre tema claro y oscuro.
  ¿Para qué? Describir cómo el usuario puede personalizar la apariencia visual de la
             aplicación.
  ¿Impacto? Mejora la experiencia visual y la accesibilidad para usuarios con
            preferencias o necesidades visuales específicas.
-->

# HU-007 — Alternancia de tema claro/oscuro

## Datos generales

| Campo           | Detalle                                       |
| --------------- | --------------------------------------------- |
| **ID**          | HU-007                                        |
| **Nombre**      | Alternancia entre tema claro y oscuro         |
| **Prioridad**   | Baja                                          |
| **Iteración**   | 3                                             |
| **Estado**      | Pendiente                                     |
| **RF asociado** | RF-010                                        |

---

## Historia

**Como** usuario de la aplicación,
**quiero** poder alternar entre el tema claro (light) y el tema oscuro (dark),
**para** adaptar la interfaz a mis preferencias visuales o condiciones de luz ambiental.

---

## Criterios de aceptación

### CA-001 — Cambio de tema

**Dado** que el usuario está en cualquier página de la aplicación y encuentra el
interruptor de tema (ícono de sol/luna en la barra de navegación),
**cuando** hace clic en él,
**entonces** la interfaz cambia inmediatamente entre modo claro y oscuro sin recargar
la página.

### CA-002 — Persistencia del tema

**Dado** que el usuario ha seleccionado un tema (claro u oscuro),
**cuando** cierra el navegador y vuelve a abrir la aplicación,
**entonces** el sistema recuerda su preferencia y mantiene el tema seleccionado
(usando `localStorage`).

### CA-003 — Preferencia del sistema

**Dado** que el usuario abre la aplicación por primera vez (sin preferencia guardada),
**cuando** carga la página,
**entonces** el sistema detecta la preferencia del sistema operativo del usuario
(`prefers-color-scheme`) y aplica el tema correspondiente como valor inicial.

### CA-004 — Consistencia visual

**Dado** que el usuario cambia de tema,
**entonces** todos los elementos de la interfaz (fondos, textos, bordes, formularios,
botones, modales, alertas) se actualizan correctamente sin elementos con colores
incoherentes.

---

## Notas técnicas

- Implementación exclusivamente en el frontend con TailwindCSS `dark:` variants.
- La preferencia se persiste en `localStorage` con la clave `nn-auth-theme`.
- Se usa `window.matchMedia('(prefers-color-scheme: dark)')` para detectar la
  preferencia del sistema.
- El estado del tema se gestiona a través de un custom hook `useTheme()`.
