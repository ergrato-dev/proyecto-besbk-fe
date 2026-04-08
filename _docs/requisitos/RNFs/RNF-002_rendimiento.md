<!--
  ¿Qué? Requerimientos no funcionales de rendimiento del sistema.
  ¿Para qué? Establecer los tiempos de respuesta y límites de carga esperados para
             garantizar una buena experiencia de usuario.
  ¿Impacto? Sin métricas de rendimiento definidas, no hay forma de evaluar si el
            sistema es adecuado para su propósito.
-->

# RNF-002 — Rendimiento

## Datos generales

| Campo        | Detalle                      |
| ------------ | ---------------------------- |
| **ID**       | RNF-002                      |
| **Nombre**   | Rendimiento del sistema      |
| **Categoría**| No funcional — Rendimiento   |

---

## Requisitos

### RNF-002.1 — Tiempos de respuesta del backend

Los endpoints deben responder dentro de los siguientes tiempos máximos bajo carga normal
(usuario único, desarrollo local):

| Operación                        | Tiempo máximo esperado |
| -------------------------------- | ---------------------- |
| `POST /api/v1/auth/login`        | < 500 ms               |
| `POST /api/v1/auth/register`     | < 800 ms               |
| `GET /api/v1/users/me`           | < 200 ms               |
| `POST /api/v1/auth/refresh`      | < 300 ms               |
| `POST /api/v1/auth/change-password` | < 500 ms            |
| `POST /api/v1/auth/forgot-password` | < 1000 ms (incluye envío de email) |

> **Nota**: El hashing BCrypt con factor de costo 10 toma ~100-300ms intencionalmente
> (esto es una característica de seguridad, no un bug — hace los ataques de fuerza
> bruta más lentos).

### RNF-002.2 — Modelo de concurrencia

El backend usa **Spring Boot con Tomcat embebido** como servidor de aplicaciones.
Tomcat maneja múltiples hilos concurrentes automáticamente. Para el contexto educativo
de este proyecto (1-5 usuarios simultáneos), el rendimiento es más que suficiente.

> **Comparativa con referencia**: El proyecto `proyecto-be-fe` usa FastAPI + Uvicorn
> (ASGI, asíncrono). Spring Boot usa un modelo de hilos (uno por request por defecto),
> que es sincrónicamente más simple de entender pero técnicamente diferente.

### RNF-002.3 — Conexiones a base de datos

Spring Boot incluye **HikariCP** como pool de conexiones por defecto. HikariCP es
considerado el pool de conexiones más rápido disponible para JVM.

Configuración mínima aceptable:
- Pool mínimo: 2 conexiones
- Pool máximo: 10 conexiones
- Timeout de conexión: 30 segundos

### RNF-002.4 — Tiempo de arranque

El tiempo de arranque del backend en modo desarrollo no debe superar los **30 segundos**
(incluye carga del contexto Spring, Flyway migrations, y pool de conexiones).

### RNF-002.5 — Rendimiento del frontend

- El tiempo de carga inicial de la aplicación React (First Contentful Paint) en
  desarrollo no debe superar los **3 segundos** en una conexión local.
- Vite ofrece HMR (Hot Module Replacement) en menos de 1 segundo durante el desarrollo.

---

## Métricas de referencia (desarrollo local)

Estas métricas son orientativas para un equipo de desarrollo local:

| Métrica                          | Objetivo            |
| -------------------------------- | ------------------- |
| Arranque backend (`./gradlew bootRun`) | < 30 s        |
| Arranque frontend (`pnpm dev`)   | < 5 s               |
| HMR del frontend                 | < 1 s               |
| Build del backend                | < 2 min             |
| Build del frontend (`pnpm build`) | < 30 s             |
| Ejecución de tests backend       | < 2 min             |
| Ejecución de tests frontend      | < 30 s              |
