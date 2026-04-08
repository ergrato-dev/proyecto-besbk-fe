---
description: "Genera un mensaje de commit Conventional Commits con cuerpo pedagógico (For/Impact) a partir de los cambios realizados. Usar antes de hacer git commit."
name: "Mensaje de commit"
argument-hint: "Describe brevemente los cambios realizados, o usa #changes para que el agente los analice"
agent: "agent"
---

# Generar mensaje de commit — NN Auth System (Kotlin Edition)

Analiza los cambios del workspace y genera un mensaje de commit siguiendo las
convenciones **Conventional Commits** del proyecto con cuerpo pedagógico.

## Formato requerido

```
type(scope): short description in english  ← máx 72 caracteres, sin punto final

For: <razón por la que se necesitaba este cambio>
Impact: <qué habilita o qué afecta este cambio en el sistema>
```

**Regla crítica**: la línea de asunto (primera línea) debe estar en **inglés**.
El cuerpo (`For:` / `Impact:`) puede estar en inglés o español.

## Tipos permitidos

| Tipo       | Cuándo usarlo                                        |
| ---------- | ---------------------------------------------------- |
| `feat`     | Nueva funcionalidad para el usuario                  |
| `fix`      | Corrección de un bug                                 |
| `docs`     | Solo cambios de documentación                        |
| `style`    | Formato, espacios (sin cambio de lógica)             |
| `refactor` | Reestructuración sin cambiar funcionalidad           |
| `test`     | Agregar o corregir tests                             |
| `chore`    | Tareas de mantenimiento, configuración, dependencias |
| `ci`       | Cambios en CI/CD                                     |
| `perf`     | Mejoras de rendimiento                               |

## Scopes del proyecto

| Scope      | Uso                                                       |
| ---------- | --------------------------------------------------------- |
| `auth`     | Autenticación y autorización                              |
| `user`     | Entidad/funcionalidad de usuario                          |
| `db`       | Modelos JPA y migraciones Flyway                          |
| `api`      | Controllers y DTOs                                        |
| `security` | JWT, filtros, BCrypt, Bucket4j                            |
| `ui`       | Componentes y estilos del frontend                        |
| `config`   | application.yml, build.gradle.kts, Docker, configuración  |
| `test`     | Tests unitarios e integración (JUnit 5, MockMvc)          |
| `deps`     | Dependencias (build.gradle.kts, package.json)             |
| `docs`     | Documentación (README, `_docs/`)                          |

## Ejemplos del proyecto

```bash
# Nueva funcionalidad
feat(auth): add email verification on registration

For: Prevent fake accounts and ensure users own the email they register with
Impact: Enables secure onboarding — users must verify before logging in

# Fix
fix(security): handle expired refresh token with 401 instead of 500

For: Prevent confusing 500 errors when users try to refresh after 7 days
Impact: Mejora la UX redirigiendo al login en vez de mostrar página de error

# Migración
feat(db): add V2 Flyway migration for password_reset_tokens table

For: Store password reset tokens linked to users with expiry support
Impact: Habilita el flujo de recuperación de contraseña via email

# Documentación
docs(config): add design-system.md with fuchsia theme reference

For: Document the accent-* token system and fuchsia color palette for Kotlin stack
Impact: Developers can clone and adapt the theme to another stack in 4 steps

# Dependencia con versión exacta (Kotlin/Gradle)
chore(deps): pin jjwt to 0.12.6 in build.gradle.kts

For: Avoid floating version ranges that could introduce CVEs without notice
Impact: Builds become reproducible — same JAR every time regardless of publish date

# Refactor backend Kotlin
refactor(auth): extract token creation logic into JwtTokenProvider

For: AuthService was doing too much — JWT logic belongs in a dedicated class
Impact: Service is now focused on business logic; JwtTokenProvider handles tokens only

# Componente frontend
feat(ui): add Button component with loading and disabled states

For: All pages need a consistent button with visual feedback during API calls
Impact: Prevents double-submit bugs and improves perceived responsiveness
```

## Instrucciones para generar

1. Analiza los archivos modificados con `#changes` o según lo descrito en $arg
2. Determina el tipo y scope más apropiados
3. Escribe la línea de asunto en inglés (máx 72 caracteres)
4. Escribe `For:` explicando la motivación del cambio
5. Escribe `Impact:` describiendo el efecto en el sistema
6. Verifica que no haya punto final en la línea de asunto

Cambios a analizar: $arg
