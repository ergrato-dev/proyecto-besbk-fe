<!--
  ¿Qué? Requerimientos no funcionales de usabilidad y UX/UI.
  ¿Para qué? Establecer los estándares de experiencia de usuario que debe cumplir
             el frontend para ser intuitivo y agradable de usar.
  ¿Impacto? Una interfaz confusa o inconsistente genera frustración y errores en los
            usuarios, aunque el backend funcione perfectamente.
-->

# RNF-003 — Usabilidad y UX/UI

## Datos generales

| Campo        | Detalle                         |
| ------------ | ------------------------------- |
| **ID**       | RNF-003                         |
| **Nombre**   | Usabilidad y experiencia de usuario |
| **Categoría**| No funcional — Usabilidad       |

---

## Requisitos

### RNF-003.1 — Coherencia visual

- Todos los formularios, botones y mensajes deben seguir un diseño consistente en toda
  la aplicación.
- Se usa **TailwindCSS** como único sistema de estilos — sin CSS personalizado excepto
  en `index.css` para configuración global.
- Todos los iconos provienen exclusivamente de **lucide-react**.
- Sin degradados en ningún elemento (restricción de diseño RD-001).

### RNF-003.2 — Retroalimentación inmediata

- Los errores de validación del formulario deben mostrarse de forma visible junto al
  campo correspondiente, sin esperar la respuesta del servidor cuando sea posible.
- Los estados de carga (`loading`) deben indicarse visualmente (spinner o texto
  "Procesando...") para evitar que el usuario haga clic múltiple.
- Los mensajes de éxito o error deben desaparecer automáticamente después de 5 segundos
  o al ejecutar la siguiente acción.

### RNF-003.3 — Responsividad

La interfaz debe ser funcional y visualmente correcta en los siguientes breakpoints:

| Breakpoint       | Ancho mínimo | Dispositivo típico          |
| ---------------- | ------------ | --------------------------- |
| Mobile           | 320px        | Teléfono pequeño            |
| Mobile L         | 375px        | Teléfono estándar           |
| Tablet           | 768px        | Tablet                      |
| Desktop          | 1024px       | Laptop / pantalla pequeña   |
| Desktop L        | 1280px       | Monitor estándar            |

### RNF-003.4 — Mensajes de error legibles

- Los mensajes de error deben estar en **español** y ser comprensibles para un usuario
  no técnico.
- Nunca mostrar mensajes técnicos como "Internal Server Error" o stack traces al
  usuario final.
- Los mensajes de validación deben indicar exactamente qué está mal y cómo corregirlo.

### RNF-003.5 — Posición de acciones primarias

Siguiendo la restricción de diseño RD-003, los botones de acción principal
(Guardar, Enviar, Crear cuenta, Iniciar sesión, etc.) deben estar siempre alineados a
la **derecha** del contenedor del formulario.

### RNF-003.6 — Tema claro y oscuro

La aplicación debe soportar ambos temas sin elementos con contraste insuficiente.
Ratio de contraste mínimo: 4.5:1 para texto normal (WCAG AA).

### RNF-003.7 — Estados del botón de envío

Los botones de envío de formularios deben tener tres estados visuales claramente
diferenciados:
- **Normal**: estilo activo, cursor pointer.
- **Loading**: spinner + texto "Procesando...", deshabilitado.
- **Disabled**: opacidad reducida, cursor not-allowed (cuando el formulario tiene
  errores de validación).

---

## Anti-patrones prohibidos

Los siguientes patrones de UX están explícitamente **prohibidos**:

| Anti-patrón                            | Razón                                             |
| -------------------------------------- | ------------------------------------------------- |
| Limpiar todos los campos al haber error | El usuario pierde los datos válidos ya ingresados |
| Mostrar errores técnicos al usuario    | Confunde y no ayuda a resolver el problema        |
| Botones sin estado de carga            | El usuario puede hacer clic múltiple              |
| Redirecciones sin feedback             | El usuario no sabe si la acción tuvo efecto       |
