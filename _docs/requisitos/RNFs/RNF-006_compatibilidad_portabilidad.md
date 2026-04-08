<!--
  ¿Qué? Requerimientos no funcionales de compatibilidad y portabilidad del sistema.
  ¿Para qué? Establecer en qué entornos, sistemas operativos y navegadores debe
             funcionar el sistema.
  ¿Impacto? Sin estos requisitos, el sistema podría funcionar solo en el equipo del
            desarrollador y fallar en otros entornos.
-->

# RNF-006 — Compatibilidad y Portabilidad

## Datos generales

| Campo        | Detalle                                    |
| ------------ | ------------------------------------------ |
| **ID**       | RNF-006                                    |
| **Nombre**   | Compatibilidad y portabilidad              |
| **Categoría**| No funcional — Compatibilidad/Portabilidad |

---

## Requisitos

### RNF-006.1 — Compatibilidad de navegadores (Frontend)

La aplicación frontend debe funcionar correctamente en las últimas 2 versiones de:

| Navegador        | Versiones mínimas |
| ---------------- | ----------------- |
| Google Chrome    | Últimas 2         |
| Mozilla Firefox  | Últimas 2         |
| Microsoft Edge   | Últimas 2         |
| Safari           | Últimas 2         |

No se requiere compatibilidad con Internet Explorer.

### RNF-006.2 — Resoluciones de pantalla (Frontend)

La interfaz debe ser funcional y sin desbordamientos en:

| Tipo dispositivo | Resolución mínima |
| ---------------- | ----------------- |
| Móvil            | 320 × 568 px      |
| Tablet           | 768 × 1024 px     |
| Escritorio       | 1024 × 768 px     |
| Escritorio Wide  | 1280 × 800 px     |

### RNF-006.3 — Sistema operativo (Desarrollo)

El entorno de desarrollo debe funcionar en:
- Linux (Ubuntu 20.04+, Fedora, etc.)
- macOS (12+)
- Windows 10/11 (con WSL2 o nativo con Docker Desktop)

> **Ventaja JVM**: A diferencia del proyecto de referencia (Python/FastAPI que tiene
> pequeñas diferencias de comportamiento entre SO), la JVM garantiza comportamiento
> idéntico en todos los sistemas operativos. `./gradlew bootRun` funciona igual en
> Linux, macOS y Windows.

### RNF-006.4 — Containerización (Docker)

El sistema completo debe poder levantarse con un único comando:

```bash
docker compose up --build
```

Esto levanta:
- `db`: PostgreSQL 17 en puerto 5432
- `be`: Spring Boot en puerto 8080
- `fe`: React + Nginx en puerto 3000
- `mailpit`: Captura de emails en puerto 8025 (UI) y 1025 (SMTP)

No debe ser necesario instalar dependencias manualmente (excepto Docker).

### RNF-006.5 — Versiones de runtime

| Herramienta              | Versión mínima requerida         |
| ------------------------ | -------------------------------- |
| JDK                      | 21 LTS (OpenJDK o equivalente)   |
| Docker                   | 24+                              |
| Docker Compose           | v2.x (`docker compose`, no `docker-compose`) |
| Node.js (para FE local)  | 20 LTS+                          |
| pnpm                     | 9+                               |

> **Nota**: Para desarrollo backend local, solo se requiere JDK 21. El Gradle Wrapper
> descarga automáticamente la versión correcta de Gradle sin instalación manual.

### RNF-006.6 — Independencia de entorno

El sistema no debe asumir ninguna configuración específica del sistema operativo del
desarrollador:

- Sin rutas absolutas hardcodeadas.
- Sin dependencias de variables de entorno del sistema operativo (solo de `.env`).
- El Gradle Wrapper (`./gradlew`) gestiona la versión de Gradle automáticamente.
- `pnpm-lock.yaml` garantiza instalación reproducible de dependencias Node.js.

---

## Verificación de compatibilidad

```bash
# Verificar versión de JDK
java --version
# Esperado: openjdk 21...

# Verificar Docker
docker --version
docker compose version

# Verificar Node.js
node --version
# Esperado: v20.x.x o superior

# Verificar pnpm
pnpm --version
# Esperado: 9.x.x o superior

# Levantar todo el sistema
docker compose up --build --detach

# Verificar que todo está healthy
docker compose ps
```
