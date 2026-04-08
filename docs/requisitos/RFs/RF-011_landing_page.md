<!--
  ¿Qué? Requerimiento funcional para la landing page pública.
  ¿Para qué? Especificar la estructura, secciones y comportamientos técnicos de la
             página de inicio del sistema.
  ¿Impacto? Es la página de entrada al sistema; define la primera impresión y
            orienta al visitante hacia las acciones principales.
-->

# RF-011 — Landing Page

## Datos generales

| Campo           | Detalle                                  |
| --------------- | ---------------------------------------- |
| **ID**          | RF-011                                   |
| **Nombre**      | Landing page pública del sistema         |
| **HU asociada** | HU-009                                   |
| **Tipo**        | Frontend — sin endpoint de API asociado  |

---

## Descripción

La landing page es la página principal pública del sistema. Debe comunicar el propósito
del sistema, listar sus características clave y ofrecer accesos directos al registro
y login. Es completamente estática (sin llamadas a la API).

---

## Estructura de la página

### 1. Barra de navegación (`Navbar`)

| Elemento                        | Descripción                                              |
| ------------------------------- | -------------------------------------------------------- |
| Logo / nombre                   | "NN Auth" a la izquierda                                 |
| Enlace "Características"        | Scroll suave a la sección de características             |
| Enlace "Contacto"               | Navega a `/contact` o hace scroll a sección de contacto  |
| Botón "Iniciar sesión"          | Navega a `/login`                                        |
| Botón "Crear cuenta"            | Navega a `/register`                                     |
| Interruptor de tema             | Icono `Sun`/`Moon` de `lucide-react`                     |

### 2. Sección Héroe (`Hero`)

- Título principal: ej. "Autenticación segura para tu equipo"
- Subtítulo descriptivo del sistema
- Botón primario: "Comenzar gratis" → `/register`
- Botón secundario: "Iniciar sesión" → `/login`

### 3. Sección de Características

Al menos 4 tarjetas, cada una con:
- Icono de `lucide-react`
- Título
- Descripción breve

Ejemplos de características:
- Autenticación JWT stateless
- Tokens con expiración controlada
- Hashing BCrypt de contraseñas
- Verificación de email

### 4. Footer

| Elemento                 | Contenido                                         |
| ------------------------ | ------------------------------------------------- |
| Logo/nombre              | "NN Auth"                                         |
| Descripción              | Texto corto sobre el proyecto                     |
| Enlace legal 1           | "Política de privacidad" → `/privacy`             |
| Enlace legal 2           | "Términos de servicio" → `/terms`                 |
| Copyright                | "© 2025 NN — SENA. Proyecto educativo."           |

---

## Comportamiento condicional

| Estado del usuario          | Comportamiento                                          |
| --------------------------- | ------------------------------------------------------- |
| No autenticado (visitante)  | Muestra la landing normalmente                          |
| Autenticado                 | Redirige automáticamente a `/dashboard`                 |

---

## Reglas de negocio

1. La landing page no tiene dependencias de la API.
2. Es responsive — debe verse correctamente en móvil, tablet y escritorio.
3. Todos los iconos provienen de `lucide-react`.
4. Sin degradados en colores ni fondos (restricción RD-001).
