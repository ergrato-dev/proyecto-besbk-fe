<!--
  ¿Qué? Requerimientos no funcionales de accesibilidad web.
  ¿Para qué? Establecer los estándares WCAG y ARIA que debe cumplir el frontend
             para ser usable por personas con discapacidades.
  ¿Impacto? Sin accesibilidad, el sistema excluye a usuarios con discapacidades
            visuales, motoras o cognitivas.
-->

# RNF-004 — Accesibilidad

## Datos generales

| Campo        | Detalle                       |
| ------------ | ----------------------------- |
| **ID**       | RNF-004                       |
| **Nombre**   | Accesibilidad web             |
| **Categoría**| No funcional — Accesibilidad  |
| **Estándar** | WCAG 2.1 Nivel AA             |

---

## Requisitos

### RNF-004.1 — Estándar mínimo

El frontend debe cumplir con el estándar **WCAG 2.1 Nivel AA** en todos sus
componentes y páginas.

### RNF-004.2 — Contraste de color

- Texto normal: ratio de contraste mínimo **4.5:1** respecto al fondo.
- Texto grande (≥18pt o ≥14pt bold): ratio mínimo **3:1**.
- Este requisito aplica tanto al tema claro como al oscuro.
- Herramientas de verificación: WebAIM Contrast Checker, axe DevTools.

### RNF-004.3 — Navegación por teclado

- Todos los elementos interactivos (botones, inputs, links) deben ser alcanzables y
  activables usando solo el teclado (Tab, Shift+Tab, Enter, Espacio).
- El foco del teclado debe ser **visible** en todo momento (no se puede quitar el
  outline de foco sin reemplazarlo con un indicador equivalente).
- El orden de tabulación debe ser lógico y seguir el flujo visual de la página.

### RNF-004.4 — Etiquetas de formularios

- Todos los campos de formulario deben tener una etiqueta `<label>` asociada
  correctamente (usando `htmlFor` en React).
- Los campos deben incluir `aria-describedby` cuando tienen mensajes de error o
  instrucciones adicionales.
- Los mensajes de error deben estar vinculados al campo con `aria-invalid="true"` y
  `aria-describedby`.

### RNF-004.5 — Atributos `autocomplete`

Los campos de formulario deben declarar el atributo `autocomplete` adecuado:

| Campo               | Valor `autocomplete`   |
| ------------------- | ---------------------- |
| Nombre completo     | `name`                 |
| Correo electrónico  | `email`                |
| Contraseña nueva    | `new-password`         |
| Contraseña actual   | `current-password`     |

### RNF-004.6 — Texto alternativo e ARIA

- Las imágenes decorativas deben tener `alt=""`.
- Los iconos usados como botones deben tener `aria-label` descriptivo.
- Los componentes de alerta/notificación deben tener `role="alert"` o `aria-live="polite"`.
- Los spinners de carga deben incluir `aria-busy="true"` y texto accesible.

### RNF-004.7 — Semántica HTML

- Usar elementos HTML semánticos correctos: `<main>`, `<nav>`, `<header>`, `<footer>`,
  `<section>`, `<article>`, `<h1>`-`<h6>` (en jerarquía correcta).
- No usar `<div>` o `<span>` para elementos interactivos que deben ser `<button>` o
  `<a>`.
- El documento debe tener un único `<h1>` por página.

### RNF-004.8 — Reducción de movimiento

- Si la aplicación incluye animaciones, deben respetarse las preferencias del sistema
  mediante:
  ```css
  @media (prefers-reduced-motion: reduce) {
    /* desactivar animaciones */
  }
  ```

---

## Herramientas de verificación recomendadas

| Herramienta       | Tipo                | Uso                                  |
| ----------------- | ------------------- | ------------------------------------ |
| axe DevTools      | Extensión Chrome    | Análisis automático de accesibilidad |
| NVDA / VoiceOver  | Lector de pantalla  | Pruebas con tecnología asistiva      |
| WebAIM Checker    | Web                 | Verificación de contraste de color   |
| Lighthouse        | Chrome DevTools     | Auditoría general de accesibilidad   |

> Ver guía completa en `docs/conceptos/accesibilidad-aria-wcag.md`.
