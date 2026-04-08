---
description: "Crea un nuevo componente UI reutilizable para el frontend con TypeScript, TailwindCSS, soporte dark/light mode y test. Usar cuando se necesite un componente que se usará en múltiples páginas."
name: "Nuevo componente UI"
argument-hint: "Describe el componente: nombre, qué renderiza, qué props recibe, comportamiento interactivo"
agent: "agent"
---

# Nuevo componente UI reutilizable — NN Auth System (Kotlin Edition)

Crea un componente React reutilizable siguiendo las convenciones del proyecto.

## Convenciones obligatorias

- **Componentes**: funcionales con hooks, `PascalCase` en nombre y archivo
- **Tipos**: `interface` para props (sufijo `Props`), `type` para uniones
- **TailwindCSS**: única fuente de estilos — sin CSS modules, sin `style={{}}` inline
- **Dark mode**: todas las clases de color deben tener variante `dark:`
- **Sin degradados**: `bg-gradient-*` está **PROHIBIDO**
- **Fuente**: sans-serif exclusivamente (`font-sans`, nunca `font-serif`)
- **Transiciones**: `transition-colors duration-200` en elementos interactivos
- **Acento**: siempre `accent-*` — **NUNCA** hardcodear `fuchsia-*` en componentes

## Lo que debes generar

### 1. Componente (`fe/src/components/ui/NombreComponente.tsx`)

Estructura esperada:

```tsx
/**
 * Archivo: NombreComponente.tsx
 * ¿Qué? [descripción del componente]
 * ¿Para qué? [por qué existe este componente]
 * ¿Impacto? [qué pasaría si no existiera o si tiene un bug]
 */

interface NombreComponenteProps {
  /** [descripción en español de la prop] */
  propA: string;
  /** [descripción en español de la prop opcional] */
  propB?: boolean;
  /** Función callback ejecutada al [acción] */
  onAccion?: () => void;
}

export function NombreComponente({
  propA,
  propB = false,
  onAccion,
}: NombreComponenteProps) {
  return (
    <div className="...clases tailwind con dark:...">
      {/* contenido */}
    </div>
  );
}
```

Referencia de componentes existentes:

- [fe/src/components/ui/Button.tsx](../../../fe/src/components/ui/Button.tsx) — botón con variantes y estado loading
- [fe/src/components/ui/InputField.tsx](../../../fe/src/components/ui/InputField.tsx) — input con label, icono, validación y a11y
- [fe/src/components/ui/Alert.tsx](../../../fe/src/components/ui/Alert.tsx) — alerta con variantes semánticas (error/success/info)

### 2. Exportar desde el barrel index (si existe `fe/src/components/ui/index.ts`)

Agregar: `export { NombreComponente } from './NombreComponente';`

### 3. Test (`fe/src/__tests__/components/NombreComponente.test.tsx`)

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { NombreComponente } from '../../components/ui/NombreComponente';

describe('NombreComponente', () => {
  it('renders without errors', () => {
    render(<NombreComponente propA="valor de prueba" />);
    expect(screen.getByText(/valor de prueba/i)).toBeInTheDocument();
  });

  it('calls onAccion when [evento]', async () => {
    const onAccion = vi.fn();
    render(<NombreComponente propA="test" onAccion={onAccion} />);
    await userEvent.click(screen.getByRole('button'));
    expect(onAccion).toHaveBeenCalledOnce();
  });
});
```

## Patrones de color obligatorios

```tsx
// ✅ Botón primario
<button className="
  bg-accent-600 hover:bg-accent-700
  dark:bg-accent-500 dark:hover:bg-accent-600
  text-white font-medium px-4 py-2 rounded-lg
  transition-colors duration-200
  disabled:opacity-50 disabled:cursor-not-allowed
">

// ✅ Link de acento
<a className="text-accent-600 dark:text-accent-400 hover:underline">

// ✅ Focus ring accesible
<input className="focus:outline-none focus:ring-2 focus:ring-accent-500" />

// ✅ Badge del stack
<span className="bg-accent-100 dark:bg-accent-900 text-accent-800 dark:text-accent-200 px-2 py-0.5 rounded text-xs">
  Kotlin
</span>

// ❌ NUNCA — color hardcodeado del stack
<button className="bg-fuchsia-600">  {/* PROHIBIDO */}
```

## Verificar tema correcto

```bash
# Después de crear el componente, verificar que no use fuchsia directamente
grep -n "fuchsia-" fe/src/components/ui/NombreComponente.tsx
# Debe dar 0 resultados — solo accent-* es válido en componentes

cd fe && pnpm test  # Todos los tests deben pasar
cd fe && pnpm lint  # ESLint no debe reportar errores
```

Componente a crear: $arg
