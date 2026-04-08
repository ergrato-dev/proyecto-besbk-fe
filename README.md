# 🔐 NN Auth System — Kotlin Edition

> Proyecto educativo — SENA | Marzo 2026

Sistema de autenticación completo para una empresa genérica "NN", con **idéntica funcionalidad**
al proyecto de referencia [`proyecto-be-fe`](https://github.com/ergrato-dev/proyecto-be-fe),
pero implementado con un stack tecnológico diferente:
**Kotlin + Spring Boot** en el backend y **React** en el frontend.

---

## 📋 Tabla de Contenidos

- [🛠️ Stack Tecnológico](#️-stack-tecnológico)
- [✅ Prerrequisitos](#-prerrequisitos)
- [🚀 Instalación y Setup](#-instalación-y-setup)
  - [Con Docker (recomendado)](#con-docker-recomendado)
  - [Sin Docker](#sin-docker)
- [▶️ Ejecución](#️-ejecución)
  - [Con Docker](#con-docker)
  - [Sin Docker (3 terminales)](#sin-docker-3-terminales)
- [🧪 Testing](#-testing)
- [📁 Estructura del Proyecto](#-estructura-del-proyecto)
- [📏 Convenciones](#-convenciones)
- [📚 Documentación Adicional](#-documentación-adicional)
- [🎓 Propósito Educativo](#-propósito-educativo)
- [⚠️ Exención de Responsabilidades](#️-exención-de-responsabilidades)
- [📄 Licencia](#-licencia)

---

## 🛠️ Stack Tecnológico

| Capa | Tecnología |
|---|---|
| **Backend** | Kotlin 1.9+, Spring Boot 3.x, Spring Security, Spring Data JPA, JWT |
| **Frontend** | React 18+, Vite, TypeScript, TailwindCSS 4+ |
| **Base de datos** | PostgreSQL 17+ |
| **Migraciones** | Flyway |
| **Email (dev)** | Mailpit — captura SMTP local, UI en puerto 8025 |
| **Testing BE** | JUnit 5, Spring Boot Test, MockMvc |
| **Testing FE** | Vitest + Testing Library |
| **Linting BE** | ktlint (Kotlin) |
| **Linting FE** | ESLint + Prettier (TypeScript) |
| **Build** | Gradle (Kotlin DSL) |
| **Runtime** | JDK 21 (LTS) |

> **Diferencias con el proyecto de referencia `proyecto-be-fe`:**
> El proyecto de referencia usa Python 3.12 + FastAPI + SQLAlchemy + Alembic.
> Este proyecto reemplaza ese stack por Kotlin + Spring Boot 3 + Spring Data JPA + Flyway,
> manteniendo **exactamente la misma funcionalidad, API, esquema de BD y comportamiento**.

---

## ✅ Prerrequisitos

Antes de comenzar, asegúrate de tener instalado:

| Herramienta | Versión mínima | Verificar |
|---|---|---|
| JDK | 21 LTS+ | `java -version` |
| Node.js | 20 LTS+ | `node --version` |
| pnpm | 9+ | `pnpm --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |
| Git | 2.40+ | `git --version` |

> ⚠️ **Importante:** Usar `pnpm` como gestor de paquetes de Node.js. **Nunca** usar `npm` ni `yarn`.

> ⚠️ **JDK 21 obligatorio** — Spring Boot 3.x requiere como mínimo JDK 17; este proyecto usa JDK 21 (LTS actual).
> Para instalar: [https://adoptium.net/](https://adoptium.net/) (Temurin 21).

<details>
<summary>🖥️ Usuarios de Windows — leer antes de continuar</summary>

Todos los comandos de este proyecto usan sintaxis Bash (`source`, `export`, `/`, etc.).
Usa siempre **Git Bash** como terminal — viene incluido al instalar [Git para Windows](https://git-scm.com/download/win).
No uses CMD ni PowerShell — los comandos no funcionarán igual.

</details>

### Instalar pnpm (si no lo tienes)

```bash
# Opción recomendada — vía corepack (incluido con Node.js 16+)
corepack enable
corepack prepare pnpm@latest --activate

# Alternativa — instalación independiente
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

---

## 🚀 Instalación y Setup

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd proyecto-besbk-fe
```

---

### Con Docker (recomendado)

Docker Compose levanta **todos los servicios** (PostgreSQL, backend Spring Boot, frontend React/Nginx, Mailpit) con un solo comando.

```bash
# Construir imágenes y levantar todos los servicios
docker compose up --build

# O en segundo plano
docker compose up --build -d

# Verificar que están corriendo
docker compose ps
# Deberías ver: nn_auth_db, nn_auth_be, nn_auth_fe, nn_auth_mailpit con estado "healthy"/"running"
```

Tras el inicio, los servicios estarán disponibles en:

| Servicio | URL |
|---|---|
| Frontend (React) | http://localhost:3000 |
| Backend API | http://localhost:8080 |
| Swagger UI (solo dev) | http://localhost:8080/swagger-ui.html |
| Mailpit (emails dev) | http://localhost:8025 |

---

### Sin Docker

Para correr cada pieza por separado (útil para desarrollo activo).

#### 2. Levantar solo la base de datos y Mailpit

```bash
# Levanta SOLO PostgreSQL y Mailpit en contenedores Docker
docker compose up -d db mailpit

# Verificar que están corriendo
docker compose ps
# Deberías ver nn_auth_db y nn_auth_mailpit con estado "healthy"
```

#### 3. Configurar el Backend

```bash
cd be

# Copiar y configurar variables de entorno
cp .env.example .env
# Editar .env con tus valores si es necesario

# Ejecutar las migraciones de base de datos (Flyway las aplica automáticamente
# al arrancar la aplicación — no requiere comando separado)

# Compilar el proyecto (descarga dependencias Gradle)
./gradlew build -x test
```

#### 4. Configurar el Frontend

```bash
cd fe

# Instalar dependencias con pnpm (¡NUNCA con npm!)
pnpm install

# Copiar y configurar variables de entorno
cp .env.example .env
```

---

## ▶️ Ejecución

### Con Docker

```bash
# Levanta todos los servicios (db + be + fe + mailpit)
docker compose up

# Ver logs de un servicio específico
docker compose logs -f be
docker compose logs -f fe

# Detener todos los servicios
docker compose down

# Detener y borrar volúmenes (¡borra datos de BD!)
docker compose down -v
```

### Sin Docker (3 terminales)

```bash
# Terminal 1 — Base de datos y Mailpit (si no están corriendo)
docker compose up -d db mailpit

# Terminal 2 — Backend (Spring Boot)
cd be
./gradlew bootRun
# → API disponible en http://localhost:8080
# → Swagger UI en http://localhost:8080/swagger-ui.html (solo si ENVIRONMENT=development)

# Terminal 3 — Frontend (React + Vite)
cd fe && pnpm dev
# → App disponible en http://localhost:5173
```

> 📧 **Mailpit** — bandeja de entrada de emails de desarrollo: `http://localhost:8025`
> Aquí se capturan los emails de verificación de cuenta y recuperación de contraseña.

---

## 🧪 Testing

### Backend

```bash
cd be

# Ejecutar todos los tests
./gradlew test

# Ejecutar con reporte de cobertura (JaCoCo)
./gradlew test jacocoTestReport

# Ejecutar un test específico
./gradlew test --tests "com.nn.auth.controller.AuthControllerTest"

# Ver reporte HTML de cobertura
open build/reports/jacoco/test/html/index.html
```

### Frontend

```bash
cd fe

# Ejecutar todos los tests
pnpm test

# Ejecutar en modo watch
pnpm test:watch

# Ejecutar con cobertura
pnpm test:coverage
```

### Linting

```bash
# Backend — ktlint
cd be && ./gradlew ktlintCheck     # Verificar errores
cd be && ./gradlew ktlintFormat    # Formatear código

# Frontend — ESLint + Prettier
cd fe && pnpm lint                 # Verificar errores
cd fe && pnpm format               # Formatear código
```

---

## 📁 Estructura del Proyecto

```
proyecto-besbk-fe/
├── .github/copilot-instructions.md     # Reglas y convenciones del proyecto
├── .gitignore                          # Archivos ignorados por git
├── docker-compose.yml                  # Todos los servicios: PostgreSQL, BE, FE, Mailpit
├── README.md                           # ← Este archivo
├── _docs/                              # Documentación técnica
│   ├── referencia-tecnica/
│   │   ├── architecture.md             # Arquitectura general, flujos y decisiones técnicas
│   │   ├── api-endpoints.md            # Todos los endpoints con parámetros y respuestas
│   │   └── database-schema.md         # Esquema ER, tablas, columnas y migraciones
│   └── conceptos/
│       ├── owasp-top-10.md            # Implementación del OWASP Top 10 2021
│       └── accesibilidad-aria-wcag.md # Estándares ARIA/WCAG 2.1 AA aplicados
│
├── be/                                 # Backend — Kotlin + Spring Boot
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/com/nn/auth/
│   │   │   │   ├── NnAuthApplication.kt       # Punto de entrada Spring Boot
│   │   │   │   ├── config/                    # Configuración: Security, CORS, Swagger
│   │   │   │   ├── controller/                # Endpoints (AuthController, UserController)
│   │   │   │   ├── service/                   # Lógica de negocio (AuthService)
│   │   │   │   ├── repository/                # Spring Data JPA repositories
│   │   │   │   ├── model/                     # Entidades JPA (User, PasswordResetToken, etc.)
│   │   │   │   ├── dto/                       # DTOs request/response
│   │   │   │   ├── security/                  # JWT filter, UserDetailsService
│   │   │   │   └── util/                      # Utilidades (email, audit log)
│   │   │   └── resources/
│   │   │       ├── application.yml            # Configuración Spring Boot
│   │   │       └── db/migration/             # Scripts SQL de Flyway (V1__, V2__...)
│   │   └── test/
│   │       └── kotlin/com/nn/auth/           # Tests con JUnit 5 + MockMvc
│   ├── build.gradle.kts                       # Dependencias y configuración Gradle
│   ├── .env.example                           # Plantilla de variables de entorno
│   └── Dockerfile                             # Imagen Docker del backend
│
└── fe/                                 # Frontend — React + Vite + TypeScript
    ├── src/
    │   ├── api/                        # Clientes HTTP (auth.ts, axios.ts)
    │   ├── components/                 # Componentes reutilizables
    │   ├── pages/                      # Páginas/vistas (Landing, Login, Register, Dashboard…)
    │   ├── hooks/                      # Custom hooks (useAuth)
    │   ├── context/                    # Context providers (AuthContext)
    │   └── types/                      # Tipos TypeScript (auth.ts)
    ├── package.json                    # Dependencias (pnpm)
    ├── .env.example                    # Plantilla de variables de entorno
    ├── vite.config.ts                  # Configuración de Vite
    └── Dockerfile                      # Imagen Docker del frontend (Nginx)
```

---

## 📏 Convenciones

| Aspecto | Convención |
|---|---|
| Nomenclatura técnica | Inglés (variables, funciones, clases, endpoints) |
| Comentarios/docs | Español (con ¿Qué? ¿Para qué? ¿Impacto?) |
| Commits | Conventional Commits en inglés + What/For/Impact |
| Kotlin | ktlint + coroutines donde aplique |
| TypeScript | strict mode + ESLint + Prettier |
| Gestor de paquetes | Gradle Kotlin DSL (BE), pnpm (FE) |
| Testing | Código generado = código probado |

Para las reglas completas, ver [.github/copilot-instructions.md](.github/copilot-instructions.md).

---

## 📚 Documentación Adicional

| Archivo | Contenido |
|---|---|
| [_docs/referencia-tecnica/architecture.md](_docs/referencia-tecnica/architecture.md) | Arquitectura general, flujos y decisiones técnicas |
| [_docs/referencia-tecnica/api-endpoints.md](_docs/referencia-tecnica/api-endpoints.md) | Todos los endpoints con parámetros, respuestas y errores |
| [_docs/referencia-tecnica/database-schema.md](_docs/referencia-tecnica/database-schema.md) | Esquema ER, tablas, columnas y migraciones Flyway |
| [_docs/conceptos/owasp-top-10.md](_docs/conceptos/owasp-top-10.md) | Implementación del OWASP Top 10 2021 con Spring Security |
| [_docs/conceptos/accesibilidad-aria-wcag.md](_docs/conceptos/accesibilidad-aria-wcag.md) | Estándares ARIA/WCAG 2.1 AA aplicados en el frontend |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Reglas y convenciones del proyecto |

---

## 🎓 Propósito Educativo

Este proyecto está diseñado para aprender haciendo. Cada archivo, función y componente incluye
comentarios pedagógicos que explican:

- **¿Qué?** — Qué hace este código
- **¿Para qué?** — Por qué existe y cuál es su propósito
- **¿Impacto?** — Qué pasa si no existiera o si se implementa mal

El proyecto es funcionalmente idéntico a [`proyecto-be-fe`](https://github.com/ergrato-dev/proyecto-be-fe),
lo que permite **comparar directamente** las decisiones de diseño entre:

| Aspecto | Proyecto de referencia | Este proyecto |
|---|---|---|
| Lenguaje BE | Python 3.12 | Kotlin 1.9 |
| Framework BE | FastAPI | Spring Boot 3 |
| ORM | SQLAlchemy 2.0 | Spring Data JPA (Hibernate) |
| Migraciones | Alembic | Flyway |
| Validación | Pydantic | Bean Validation (Jakarta) |
| Tests BE | pytest + httpx | JUnit 5 + MockMvc |
| Servidor BE | Uvicorn (ASGI) | Embedded Tomcat (Servlet) |

> "La calidad no es una opción, es una obligación."

---

## ⚠️ Exención de Responsabilidades

Este proyecto es de naturaleza exclusivamente educativa, desarrollado como ejercicio formativo en el marco del **SENA**.

- **No apto para producción** — El sistema no ha sido auditado ni endurecido para entornos productivos reales. No debe usarse para proteger datos sensibles de usuarios reales sin una revisión de seguridad profesional previa.
- **Credenciales de ejemplo** — Las contraseñas, claves secretas y cadenas de conexión presentes en `.env.example` y en la documentación son únicamente ilustrativas. Nunca usarlas en producción.
- **Sin garantía de disponibilidad** — El proyecto puede contener bugs o comportamientos no documentados propios de un entorno de aprendizaje.
- **Responsabilidad del aprendiz** — Cada aprendiz es responsable de comprender el código que ejecuta en su equipo y de no reutilizarlo sin entenderlo completamente.

> Este material se provee "tal cual", sin garantías explícitas ni implícitas de ningún tipo.

---

## 📄 Licencia

Proyecto educativo — SENA. Uso exclusivamente académico.
