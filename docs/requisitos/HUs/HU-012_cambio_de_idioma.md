<!--
  ¿Qué? Historia de usuario para el cambio de idioma de la interfaz.
  ¿Para qué? Describir cómo el usuario puede seleccionar su idioma preferido (ES/EN)
             y que la aplicación recuerde esa preferencia entre sesiones.
  ¿Impacto? Hace la aplicación accesible a usuarios hispanohablantes y angloparlantes.
            Refuerza el concepto de i18n (internacionalización) aplicado en producción.
-->

# HU-012 — Cambio de idioma de la interfaz

## Datos generales

| Campo           | Detalle                                               |
| --------------- | ----------------------------------------------------- |
| **ID**          | HU-012                                                |
| **Nombre**      | Selección y cambio de idioma de la interfaz           |
| **Prioridad**   | Baja                                                  |
| **Iteración**   | 4                                                     |
| **Estado**      | Pendiente                                             |
| **RF asociado** | RF-014                                                |

---

## Historia

**Como** usuario de la aplicación,
**quiero** poder cambiar el idioma de la interfaz entre español e inglés,
**para** usar la aplicación en el idioma de mi preferencia.

---

## Criterios de aceptación

### CA-001 — Selector de idioma visible

**Dado** que el usuario está en cualquier página de la aplicación,
**cuando** observa la barra de navegación,
**entonces** encuentra un selector de idioma (`LanguageSwitcher`) que muestra el
idioma actualmente activo (p. ej. `ES` / `EN`).

### CA-002 — Cambio inmediato de idioma

**Dado** que el usuario hace clic en el selector y elige un idioma diferente,
**cuando** se confirma la selección,
**entonces** toda la interfaz (textos de navegación, etiquetas de formularios,
mensajes de error, botones, títulos de página) cambia al idioma seleccionado
sin recargar la página.

### CA-003 — Persistencia en el navegador

**Dado** que el usuario ha seleccionado un idioma,
**cuando** cierra el navegador y vuelve a abrir la aplicación,
**entonces** la preferencia de idioma se mantiene (guardada en `localStorage`
con la clave `i18nextLng`).

### CA-004 — Detección automática en la primera visita

**Dado** que el usuario abre la aplicación por primera vez (sin preferencia guardada),
**cuando** carga la página,
**entonces** el sistema detecta el idioma del navegador del usuario y aplica el
idioma más cercano soportado; si no hay coincidencia, usa español (`es`) como
idioma por defecto.

### CA-005 — Sincronización con el perfil (usuario autenticado)

**Dado** que el usuario está autenticado y cambia el idioma,
**cuando** se realiza el cambio,
**entonces** la preferencia se sincroniza con el backend guardando el valor en la
columna `locale` del usuario (mediante `PATCH /api/v1/users/me/locale`).

### CA-006 — Idiomas soportados

**Dado** que el usuario interactúa con el selector,
**entonces** el selector muestra exactamente dos opciones:
- `ES` — Español (idioma por defecto)
- `EN` — English

---

## Notas técnicas

- Implementación con `i18next`, `react-i18next` e `i18next-browser-languagedetector`.
- Los catálogos de traducción se encuentran en:
  - `fe/src/locales/es/translation.json` — español
  - `fe/src/locales/en/translation.json` — inglés
- La configuración central está en `fe/src/i18n.ts`.
- El componente `LanguageSwitcher` se integra en `Navbar`.
- Para usuarios autenticados, se llama `PATCH /api/v1/users/me/locale` al cambiar el idioma.
- Para usuarios no autenticados, solo se persiste en `localStorage`.
- La columna `locale VARCHAR(10) DEFAULT 'es'` se agrega a la tabla `users` en la
  migración Flyway `V4__add_locale_to_users.sql`.
