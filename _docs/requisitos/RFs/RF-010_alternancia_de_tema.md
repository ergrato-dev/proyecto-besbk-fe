<!--
  ¿Qué? Requerimiento funcional para la alternancia de tema claro/oscuro.
  ¿Para qué? Especificar técnicamente cómo se implementa y persiste la preferencia
             de tema en el frontend.
  ¿Impacto? Define el comportamiento esperado del sistema de temas y su persistencia
            entre sesiones del navegador.
-->

# RF-010 — Alternancia de tema claro/oscuro

## Datos generales

| Campo           | Detalle                                    |
| --------------- | ------------------------------------------ |
| **ID**          | RF-010                                     |
| **Nombre**      | Alternancia entre tema claro y oscuro      |
| **HU asociada** | HU-007                                     |
| **Tipo**        | Frontend — sin endpoint de API asociado    |

---

## Descripción

La aplicación debe soportar dos modos visuales (claro y oscuro) que el usuario puede
alternar en cualquier momento. La preferencia se persiste en `localStorage` y se
detecta automáticamente en la primera visita.

---

## Implementación técnica

### Hook `useTheme`

```tsx
// Pseudocódigo del hook
function useTheme() {
  // 1. Leer preferencia guardada en localStorage
  // 2. Si no hay preferencia, usar prefers-color-scheme del SO
  // 3. Aplicar la clase 'dark' en el elemento <html>
  // 4. Guardar cambios en localStorage
  return { theme, toggleTheme }
}
```

### TailwindCSS — Configuración

```js
// tailwind.config.js
export default {
  darkMode: 'class',  // Activar modo oscuro basado en clase CSS
  // ...
}
```

### Aplicación de la clase

El hook aplica/quita la clase `dark` en el elemento raíz:

```typescript
document.documentElement.classList.toggle('dark', isDark)
```

---

## Persistencia

| Clave localStorage | Valores posibles |
| ------------------ | ---------------- |
| `nn-auth-theme`    | `"light"`, `"dark"` |

---

## Detección automática inicial

```typescript
const systemPreference = window.matchMedia('(prefers-color-scheme: dark)').matches
  ? 'dark'
  : 'light'
```

---

## Reglas de negocio

1. El icono del interruptor muestra el estado opuesto al actual (si es modo oscuro, muestra sol).
2. El cambio de tema es inmediato — sin animaciones de transición (para simplificar).
3. La preferencia del sistema operativo se usa **solo** en la primera visita.
4. El tema persiste independientemente de si el usuario está autenticado o no.
5. Todos los componentes usan variantes `dark:` de Tailwind — sin CSS personalizado.
