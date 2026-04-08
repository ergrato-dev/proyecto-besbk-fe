<!--
  ¿Qué? Requerimientos no funcionales de mantenibilidad y calidad del código.
  ¿Para qué? Establecer los estándares de calidad que garantizan que el código sea
             legible, testeable y fácil de mantener.
  ¿Impacto? El código sin estándares de calidad se convierte en deuda técnica que
            dificulta las modificaciones y genera bugs inesperados.
-->

# RNF-005 — Mantenibilidad y Calidad

## Datos generales

| Campo        | Detalle                               |
| ------------ | ------------------------------------- |
| **ID**       | RNF-005                               |
| **Nombre**   | Mantenibilidad y calidad del código   |
| **Categoría**| No funcional — Mantenibilidad         |

---

## Requisitos

### RNF-005.1 — Cobertura de pruebas

- El código del backend debe tener una cobertura mínima del **80%** en servicios y
  controladores, medida con **JUnit 5 + JaCoCo**.
- El frontend debe tener pruebas para los componentes críticos (formularios de auth,
  ProtectedRoute, AuthContext) con **Vitest + Testing Library**.
- No se considera una feature completa hasta que sus pruebas pasen.

```bash
# Backend — ejecutar con cobertura
./gradlew test jacocoTestReport

# Frontend — ejecutar con cobertura
pnpm test:coverage
```

### RNF-005.2 — Tipado estático

- **Backend**: Kotlin exige tipos explícitos en parámetros y retornos de funciones
  públicas. El compilador los verifica en tiempo de compilación — no hay sorpresas
  en runtime por errores de tipos.
- **Frontend**: TypeScript con `"strict": true` en `tsconfig.json`. Sin `any` implícito
  ni explicit salvo casos excepcionales justificados.

### RNF-005.3 — Linting y formateo automático

- **Backend**: `ktlint` como linter y formatter para código Kotlin. Debe ejecutarse
  antes de cada commit.
- **Frontend**: ESLint para linting y Prettier para formateo.

```bash
# Backend
./gradlew ktlintCheck   # Verificar errores de formato
./gradlew ktlintFormat  # Corregir automáticamente

# Frontend
pnpm lint               # Verificar errores ESLint
pnpm format             # Formatear con Prettier
```

El CI/CD debería fallar si `ktlintCheck` o `pnpm lint` reportan errores.

### RNF-005.4 — Arquitectura en capas

El backend debe seguir una arquitectura en capas bien definida:

```
Controllers → Services → Model/Repository → DTO
```

| Capa           | Responsabilidad                                        | Anotación Spring  |
| -------------- | ------------------------------------------------------ | ----------------- |
| `controller/`  | Recibir requests HTTP, delegar al servicio, formar respuesta | `@RestController` |
| `service/`     | Contener la lógica de negocio                          | `@Service`        |
| `repository/`  | Acceso a datos (Spring Data JPA)                       | `@Repository`     |
| `model/`       | Entidades JPA (mapeo objeto-relacional)                | `@Entity`         |
| `dto/`         | Data Transfer Objects (request/response sin lógica)    | `data class`      |
| `security/`    | Filtros, JWT, UserDetails                              | `@Component`      |

> **Comparativa con referencia**: En `proyecto-be-fe` la arquitectura es:
> `routers/` → `services/` → `models/` → `schemas/`.
> El concepto es idéntico; los nombres y las herramientas cambian.

### RNF-005.5 — Comentarios pedagógicos

Cada bloque de código significativo debe incluir comentarios con el formato:

```kotlin
/**
 * ¿Qué? [qué hace este código]
 * ¿Para qué? [por qué existe / cuál es su propósito]
 * ¿Impacto? [qué pasa si se elimina o modifica incorrectamente]
 */
```

### RNF-005.6 — Cabecera de archivo

Cada archivo nuevo debe comenzar con un comentario de cabecera que describa su
propósito:

```kotlin
/**
 * Archivo: JwtTokenProvider.kt
 * Descripción: Generación y verificación de tokens JWT.
 * ¿Para qué? Centralizar la lógica de JWT para ser reusable en toda la capa de seguridad.
 * ¿Impacto? Es la base de la seguridad del sistema — un error compromete toda la auth.
 */
```

### RNF-005.7 — Conventional Commits

Todos los commits deben seguir el formato **Conventional Commits** con:
- Tipo: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Scope: `auth`, `user`, `db`, `api`, `ui`, `config`, `security`, etc.
- Descripción corta en inglés
- Cuerpo con: `What:`, `For:`, `Impact:`

---

## Checklist de calidad antes de commit

- [ ] ¿El código Kotlin tiene tipos explícitos en funciones públicas?
- [ ] ¿El código TypeScript tiene tipos completos (`strict: true`)?
- [ ] ¿Hay comentarios pedagógicos (¿Qué? ¿Para qué? ¿Impacto?)?
- [ ] ¿Los tests pasan? (`./gradlew test` / `pnpm test`)
- [ ] ¿El linter no reporta errores? (`./gradlew ktlintCheck` / `pnpm lint`)
- [ ] ¿El commit sigue Conventional Commits con What/For/Impact?
- [ ] ¿Las variables sensibles están en `.env` y no hardcodeadas?
