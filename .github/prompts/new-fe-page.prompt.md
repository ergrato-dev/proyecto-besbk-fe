---
description: "Crea una nueva página React para el frontend con TypeScript, TailwindCSS, manejo de rutas y test. Usar cuando se necesita una pantalla completa nueva (login, register, dashboard, etc.)."
name: "Nueva página React"
argument-hint: "Nombre de la página, ruta URL, si requiere autenticación, qué muestra/hace, qué API endpoints consume"
agent: "agent"
---

# Nueva página React — NN Auth System (Kotlin Edition)

Crea una nueva página siguiendo las convenciones del proyecto.

## Convenciones obligatorias

- **Nombre de archivo**: `NombrePage.tsx` en `fe/src/pages/`
- **JSDoc pedagógico al inicio**: ¿Qué? ¿Para qué? ¿Impacto?
- **Sin degradados**: `bg-gradient-*` está **PROHIBIDO**
- **Dark mode**: todas las clases de color deben tener variante `dark:`
- **Acento**: siempre `accent-*` — **NUNCA** hardcodear `fuchsia-*` en componentes
- **Layout**: usar `AuthLayout` para páginas de autenticación, estructura directa para dashboards
- **Carga**: mostrar estado de loading mientras se espera la API
- **Errores**: mostrar feedback visual al usuario con el componente `Alert`

## Lo que debes generar

### 1. Componente de página (`fe/src/pages/NombrePage.tsx`)

Estructura esperada:

```tsx
/**
 * Archivo: NombrePage.tsx
 * ¿Qué? [descripción de la página]
 * ¿Para qué? [por qué existe esta pantalla en el flujo de auth]
 * ¿Impacto? [qué pasaría si no existiera o si tiene un bug]
 */

import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';
import { Alert } from '../components/ui/Alert';
import { Button } from '../components/ui/Button';

export default function NombrePage() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState(false);
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    setError(null);

    try {
      // llamada a la API...
      setSuccess(true);
    } catch (err) {
      // ¿Qué? Manejo de errores de la API.
      // ¿Para qué? Mostrar feedback apropiado sin exponer detalles técnicos al usuario.
      // ¿Impacto? Sin este manejo, el usuario vería mensajes de error técnicos incomprensibles.
      setError(err instanceof Error ? err.message : 'Error inesperado. Intenta de nuevo.');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <main className="min-h-screen bg-white dark:bg-gray-950 flex items-center justify-center">
      {/* contenido */}
    </main>
  );
}
```

Para páginas de autenticación, usar `AuthLayout`:

```tsx
import { AuthLayout } from '../components/layout/AuthLayout';

export default function NombrePage() {
  return (
    <AuthLayout title="Título de la página">
      {/* formulario de auth */}
    </AuthLayout>
  );
}
```

Referencia de páginas existentes:

- [fe/src/pages/LoginPage.tsx](../../../fe/src/pages/LoginPage.tsx) — login con token handling
- [fe/src/pages/RegisterPage.tsx](../../../fe/src/pages/RegisterPage.tsx) — registro con validación
- [fe/src/pages/DashboardPage.tsx](../../../fe/src/pages/DashboardPage.tsx) — página protegida con perfil

### 2. Registrar ruta en `App.tsx`

Localizar el bloque de rutas en [fe/src/App.tsx](../../../fe/src/App.tsx) y agregar:

**Página pública:**

```tsx
<Route path="/nueva-ruta" element={<NombrePage />} />
```

**Página protegida (requiere autenticación):**

```tsx
<Route
  path="/nueva-ruta"
  element={
    <ProtectedRoute>
      <NombrePage />
    </ProtectedRoute>
  }
/>
```

### 3. Test (`fe/src/__tests__/pages/NombrePage.test.tsx`)

Casos mínimos a cubrir:

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import NombrePage from '../../pages/NombrePage';

// Mock de AuthContext si la página usa useAuth
vi.mock('../../context/AuthContext', () => ({
  useAuth: () => ({
    user: null,
    isAuthenticated: false,
    login: vi.fn(),
  }),
}));

describe('NombrePage', () => {
  it('renders the page without errors', () => {
    render(
      <MemoryRouter>
        <NombrePage />
      </MemoryRouter>
    );
    expect(screen.getByRole('main')).toBeInTheDocument();
  });

  it('shows validation error when form is submitted with invalid data', async () => {
    render(
      <MemoryRouter>
        <NombrePage />
      </MemoryRouter>
    );
    await userEvent.click(screen.getByRole('button', { name: /enviar/i }));
    await waitFor(() => {
      expect(screen.getByRole('alert')).toBeInTheDocument();
    });
  });
});
```

## Verificar

```bash
cd fe

# Ejecutar tests
pnpm test

# Verificar que no haya colores hardcodeados del stack en la nueva página
grep -n "fuchsia-" fe/src/pages/NombrePage.tsx
# Debe dar 0 resultados

# Verificar linting
pnpm lint
```

Página a crear: $arg
