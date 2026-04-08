<!--
  ¿Qué? Requerimiento funcional para la protección de rutas en el frontend.
  ¿Para qué? Especificar cómo el componente ProtectedRoute y los interceptores Axios
             garantizan que solo usuarios autenticados accedan a secciones privadas.
  ¿Impacto? Sin este mecanismo, el frontend podría renderizar páginas protegidas
            aunque el token sea inválido.
-->

# RF-009 — Protección de rutas

## Datos generales

| Campo           | Detalle                                     |
| --------------- | ------------------------------------------- |
| **ID**          | RF-009                                      |
| **Nombre**      | Protección de rutas privadas en el frontend |
| **HU asociada** | HU-008                                      |
| **Tipo**        | Frontend — sin endpoint de API asociado     |

---

## Descripción

El frontend debe proteger todas las rutas privadas verificando la presencia y validez
del token JWT antes de renderizar el contenido protegido. Esto se implementa a nivel
de React Router con un componente dedicado.

---

## Implementación

### Componente `ProtectedRoute`

```tsx
// Pseudocódigo del componente ProtectedRoute
function ProtectedRoute({ children }) {
  const { user, isLoading } = useAuth()

  if (isLoading) return <LoadingSpinner />
  if (!user) return <Navigate to="/login" replace />

  return children
}
```

### Configuración en `App.tsx`

```tsx
// Rutas públicas
<Route path="/" element={<LandingPage />} />
<Route path="/login" element={<LoginPage />} />
<Route path="/register" element={<RegisterPage />} />

// Rutas privadas (protegidas)
<Route path="/dashboard" element={
  <ProtectedRoute>
    <DashboardPage />
  </ProtectedRoute>
} />
```

### Interceptor de Axios

El interceptor en `api/axios.ts` maneja la renovación automática de tokens:

1. Si la respuesta es `401`, comprueba si hay `refresh_token`.
2. Si hay `refresh_token`, llama a `POST /api/v1/auth/refresh`.
3. Si la renovación es exitosa, reintenta la solicitud original con el nuevo token.
4. Si la renovación falla, llama a `logout()` y redirige a `/login`.

---

## Rutas del sistema

| Ruta                   | Tipo      | Redirección si no autenticado |
| ---------------------- | --------- | ----------------------------- |
| `/`                    | Pública   | —                             |
| `/login`               | Pública\* | → `/dashboard` (si autenticado) |
| `/register`            | Pública\* | → `/dashboard` (si autenticado) |
| `/forgot-password`     | Pública   | —                             |
| `/reset-password`      | Pública   | —                             |
| `/verify-email`        | Pública   | —                             |
| `/privacy`             | Pública   | —                             |
| `/terms`               | Pública   | —                             |
| `/contact`             | Pública   | —                             |
| `/dashboard`           | Privada   | → `/login`                    |
| `/change-password`     | Privada   | → `/login`                    |

(\*) Si el usuario ya está autenticado, se redirige al dashboard.

---

## Reglas de negocio

1. La verificación de ruta ocurre **antes** de renderizar cualquier componente privado.
2. La redirección preserva la URL de destino para redireccionar después del login
   (comportamiento "redirect after login").
3. El estado de carga inicial se muestra mientras el `AuthContext` verifica el token.
