<!--
  ¿Qué? Documento que define las restricciones técnicas, organizacionales y de diseño del proyecto.
  ¿Para qué? Establecer los límites y condiciones no negociables bajo las cuales se
             desarrolla el sistema.
  ¿Impacto? Violar una restricción puede comprometer la calidad, seguridad o coherencia
            del proyecto.
-->

# Restricciones del Proyecto — NN Auth System (Kotlin Edition)

---

## 1. Restricciones Tecnológicas

### RT-001 — Stack de backend obligatorio
El backend debe desarrollarse exclusivamente con:
- **Kotlin 1.9+** como lenguaje.
- **JDK 21 LTS** como runtime.
- **Spring Boot 3.x** como framework principal.
- **Spring Data JPA** (sobre Hibernate 6.x) como ORM.
- **Flyway** para migraciones de base de datos versionadas.
- **Bean Validation (Jakarta)** para validación de datos en DTOs.

No se permite el uso de otros frameworks JVM (Quarkus, Micronaut, Ktor, etc.) ni ORMs
alternativos.

### RT-002 — Stack de frontend obligatorio
El frontend debe desarrollarse exclusivamente con:
- **React 18+** como biblioteca de UI.
- **TypeScript 5.0+** como lenguaje (siempre en modo estricto).
- **Vite 6+** como bundler y servidor de desarrollo.
- **TailwindCSS 4+** como framework de estilos.
- **React Router 7+** para enrutamiento del lado del cliente.

No se permite el uso de otros frameworks (Angular, Vue, Svelte, etc.).

### RT-003 — Base de datos obligatoria
La base de datos debe ser **PostgreSQL 17+**, ejecutada en contenedor Docker durante el
desarrollo. No se permiten bases de datos alternativas (MySQL, SQLite en producción,
MongoDB, etc.).

### RT-004 — Método de autenticación
La autenticación debe implementarse exclusivamente mediante **JWT (JSON Web Tokens)** con
enfoque stateless. No se permiten sesiones basadas en cookies de servidor, OAuth de
terceros ni integración con proveedores de identidad externos.

### RT-005 — Algoritmo de hashing
Las contraseñas deben hashearse exclusivamente con **BCrypt** (vía
`BCryptPasswordEncoder` de Spring Security). No se permiten otros algoritmos de hashing
(MD5, SHA-256, Argon2, etc.) salvo aprobación explícita.

---

## 2. Restricciones de Herramientas y Entorno

### RH-001 — Build tool y wrapper Kotlin
El proyecto debe construirse y ejecutarse exclusivamente con **Gradle Kotlin DSL** usando
el **Gradle Wrapper** (`./gradlew`). Queda prohibido usar Gradle instalado globalmente.

```bash
# ✅ CORRECTO — siempre usar el wrapper incluido en el proyecto
./gradlew build
./gradlew bootRun
./gradlew test

# ❌ INCORRECTO — puede diferir en versión
gradle build
```

### RH-002 — Gestor de paquetes Node.js
Las dependencias del frontend deben gestionarse exclusivamente con **pnpm**. Queda
**prohibido** el uso de `npm` o `yarn` para cualquier operación (install, add, run, etc.).

### RH-003 — Linter y formatter Kotlin
Se debe usar exclusivamente **ktlint** como linter y formatter para el código Kotlin. No
se permiten linters alternativos ni formatters alternativos.

### RH-004 — Linter y formatter Frontend
Se deben usar **ESLint** para linting y **Prettier** para formateo en el frontend. No se
permiten herramientas alternativas.

---

## 3. Restricciones de Diseño Visual

### RD-001 — Prohibición de degradados
Queda **estrictamente prohibido** el uso de degradados (`gradient`) en cualquier elemento
de la interfaz. Todos los fondos y colores deben ser sólidos y planos.

### RD-002 — Tipografía sans-serif exclusiva
Solo se permiten fuentes de la familia **sans-serif** (`Inter`, `system-ui`, `sans-serif`).
Queda prohibido el uso de fuentes serif, monospace (fuera de bloques de código) u
ornamentales.

### RD-003 — Alineación de botones de acción
Los botones de acción principal (Guardar, Enviar, Crear cuenta, etc.) deben estar siempre
alineados a la **derecha** del contenedor. Nunca centrados ni alineados a la izquierda.

### RD-004 — Biblioteca de iconos
Se debe usar exclusivamente **lucide-react** como biblioteca de iconos. No se permiten
SVGs inline ni bibliotecas alternativas (@heroicons/react, react-icons, etc.).

---

## 4. Restricciones de Idioma

### RI-001 — Código en inglés
Todo el código fuente debe escribirse en inglés:
- Variables, funciones, clases, métodos, constantes.
- Nombres de archivos y carpetas de código.
- Endpoints y rutas de la API.
- Nombres de tablas y columnas en la base de datos.
- Mensajes de commits y nombres de ramas.

### RI-002 — Documentación en español
Toda la documentación y comentarios deben escribirse en español:
- Comentarios en el código (`//`, `/* */`).
- KDoc de funciones y clases (equivalente a docstrings en Kotlin).
- Archivos de documentación (`.md`).
- Descripciones en archivos de configuración.

---

## 5. Restricciones Organizacionales

### RO-001 — Proyecto educativo
Este es un proyecto educativo del SENA. Cada línea de código y documentación debe tener
enfoque pedagógico. Los comentarios deben explicar el "qué", "para qué" e "impacto" de
cada decisión técnica.

### RO-002 — Conventional Commits
Todos los mensajes de commit deben seguir el formato **Conventional Commits** con cuerpo
que incluya What, For e Impact.

### RO-003 — Versionamiento de API
Todos los endpoints deben estar bajo el prefijo `/api/v1/`. Cambios incompatibles
requerirían una nueva versión (`/api/v2/`).

### RO-004 — No despliegue en producción
El proyecto se desarrolla y ejecuta exclusivamente en entornos de desarrollo local. No
hay requisitos de despliegue en producción, CI/CD ni infraestructura cloud.

---

## 6. Restricciones de Seguridad

### RS-001 — Credenciales en variables de entorno
Toda información sensible (claves secretas, credenciales de BD, configuraciones SMTP)
debe almacenarse en archivos `.env` no versionados en git. Nunca se deben hardcodear
valores sensibles en el código.

### RS-002 — Archivo .env.example obligatorio
Siempre debe existir un `.env.example` actualizado con las variables necesarias y valores
de ejemplo no sensibles.

### RS-003 — No exponer contraseñas
Las contraseñas (hasheadas o en texto plano) nunca deben aparecer en:
- Respuestas de la API.
- Logs del servidor.
- Mensajes de error al cliente.
