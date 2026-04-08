---
description: "Realiza una auditoría de seguridad basada en OWASP Top 10 adaptada al stack Spring Boot (Kotlin) + React del proyecto. Usar antes de un PR importante, una release o al agregar funcionalidad de auth."
name: "Revisión de seguridad OWASP Top 10"
argument-hint: "Ruta o módulo a auditar (ej: 'be/src/main/kotlin/com/nn/auth/security/', 'flujo completo de change-password', 'fe/src/api/')"
agent: "agent"
---

# Revisión de Seguridad OWASP Top 10 — NN Auth System (Kotlin Edition)

Realiza una auditoría exhaustiva del código indicado según las categorías OWASP Top 10
adaptadas al stack del proyecto.

## Stack de referencia

- **Backend**: Kotlin + JDK 21 + Spring Boot 3 + Spring Security (BCryptPasswordEncoder) + JJWT
- **Build tool**: Gradle Kotlin DSL (`./gradlew`)
- **Frontend**: React 18 + TypeScript + Axios
- **ORM**: Spring Data JPA + Hibernate (PostgreSQL 17)
- **Rate limiting**: Bucket4j
- **Migraciones**: Flyway SQL

---

## Categorías a auditar

### A01 — Control de Acceso Roto

Verificar:

- ¿Los endpoints protegidos están configurados en `SecurityFilterChain` con `authenticated()`?
- ¿Hay endpoints que deberían requerir auth pero están marcados con `permitAll()`?
- ¿El `JwtAuthFilter` (que extiende `OncePerRequestFilter`) valida el token antes de permitir acceso?
- ¿Los recursos de un usuario no son accesibles por otro? (IDOR — Insecure Direct Object Reference)
- ¿Las rutas del frontend tienen `ProtectedRoute` donde corresponde?
- ¿El método `SecurityFilterChain` deshabilita CSRF correctamente para una API stateless?

### A02 — Fallos Criptográficos

Verificar:

- ¿Las contraseñas se hashean con `BCryptPasswordEncoder` (strength ≥10)?
- ¿Hay algún lugar donde se almacene o loggee una contraseña en texto plano?
- ¿El `JWT_SECRET` tiene mínimo 32 caracteres? ¿Se valida con `@ConfigurationProperties` al arrancar?
- ¿Se usa HMAC-SHA256 (HS256) para firmar los JWT?
- ¿Los tokens de reset/verificación son UUIDs aleatorios (`UUID.randomUUID()`)?
- ¿Se usa HTTPS en producción? ¿Las headers `HSTS`, `nosniff` están configuradas?

### A03 — Inyección (SQL / JPQL / XSS)

Verificar:

- ¿Todos los accesos a BD usan Spring Data JPA Repository o JPQL parametrizado?
- ¿Hay algún uso de `EntityManager.createNativeQuery()` con concatenación de strings?
- ¿Los métodos del repositorio usan `@Query` con parámetros nombrados (`:param`) y nunca concatenación?
- ¿Las respuestas de la API no incluyen HTML sin escapar que pueda causar XSS?
- ¿El frontend React escapa automáticamente el JSX (sí, por defecto)?
- ¿Hay uso de `dangerouslySetInnerHTML`? Si es así, ¿está justificado y sanitizado?

### A04 — Diseño Inseguro

Verificar:

- ¿El flujo de `forgot-password` devuelve siempre la misma respuesta, independientemente de si el email existe?
- ¿Los tokens de reset tienen fecha de expiración (`expires_at`) y se verifica en `resetPassword`?
- ¿Los tokens de verificación de email tienen fecha de expiración?
- ¿Se controla el flujo de rate limiting para prevenir ataques de enumeración de emails?

### A05 — Mala Configuración de Seguridad

Verificar:

- ¿El campo CORS (`allowedOrigins`) usa solo `FRONTEND_URL` del `.env`? ¿Nunca `"*"`?
- ¿Swagger UI se deshabilita en producción? (`@ConditionalOnProperty(name = "app.environment", havingValue = "development")`)
- ¿Los headers de seguridad están configurados en `SecurityConfig`? (`X-Frame-Options`, `X-Content-Type-Options`)
- ¿El `application.yml` versionado no contiene credenciales? ¿Siempre usa `${VARIABLE:default}`?
- ¿El `.env` está en `.gitignore`?

### A06 — Componentes Vulnerables y Desactualizados

Verificar:

```bash
# Auditar dependencias Kotlin/Spring Boot
cd be && ./gradlew dependencyCheckAnalyze

# Auditar dependencias del frontend
cd fe && pnpm audit --audit-level moderate

# Ver árbol de dependencias del backend
./gradlew dependencies --configuration runtimeClasspath | head -100
```

- ¿Todas las dependencias en `build.gradle.kts` tienen versiones exactas (sin `+`, sin rangos)?
- ¿Las dependencias de `package.json` tienen versiones exactas (sin `^`, sin `~`)?
- ¿La versión de Spring Boot es reciente y con soporte activo?

### A07 — Fallos de Identificación y Autenticación

Verificar:

- ¿El access token expira en 15 minutos? (`JWT_ACCESS_TOKEN_EXPIRATION_MINUTES=15`)
- ¿El refresh token expira en 7 días? (`JWT_REFRESH_TOKEN_EXPIRATION_DAYS=7`)
- ¿El frontend guarda tokens en **memoria** (estado React), nunca en `localStorage`?
- ¿El interceptor de Axios maneja automáticamente el refresh cuando obtiene 401?
- ¿La validación de contraseña exige ≥8 chars, 1 mayúscula, 1 minúscula, 1 número?
- ¿El endpoint de `login` está protegido por Bucket4j (rate limiting por IP)?

### A08 — Fallos de Integridad en Software y Datos

Verificar:

- ¿Flyway usa scripts SQL inmutables (nunca se edita un script ya aplicado)?
- ¿Los tokens JWT incluyen `iss` (issuer) y `sub` (subject) correctamente configurados?
- ¿Las dependencias de `build.gradle.kts` usan Gradle Wrapper? (`./gradlew`)
- ¿No hay dependencias instaladas directamente sin pasar por el procesaof de revisión de versiones?

### A09 — Fallos de Registro y Monitoreo de Seguridad

Verificar:

- ¿`AuditLogService` registra los eventos: login exitoso, login fallido, contraseña cambiada?
- ¿Los logs NO incluyen contraseñas ni tokens completos?
- ¿Los logs de auditoría incluyen IP del cliente, timestamp y userId?
- ¿Los errores de autenticación generan logs de nivel WARN o ERROR sin exponer detalles internos?
- ¿Los logs están en formato estructurado JSON (Logback + `logstash-logback-encoder`)?

### A10 — Falsificación de Solicitudes del Lado del Servidor (SSRF)

Verificar:

- ¿El backend realiza peticiones HTTP a URLs externas? (ej: validación de webhooks)
- Si las hay, ¿se valida que la URL sea de un dominio permitido antes de realizar la petición?
- ¿El `JavaMailSender` usa el host del `.env` y no permite configuración dinámica desde el request?

---

## Resumen de hallazgos

Para cada hallazgo, documentar:

```
[A0X] Categoría OWASP
Severidad: CRÍTICA / ALTA / MEDIA / BAJA / INFORMATIVA
Archivo: be/src/main/kotlin/com/nn/auth/...
Línea: N
Descripción: [qué está mal]
Evidencia: [fragmento de código relevante]
Recomendación: [cómo corregirlo con Spring Boot / Kotlin]
```

---

## Checklist final antes de cerrar la auditoría

- [ ] ¿Las contraseñas nunca aparecen en logs, responses ni variables de entorno del SE?
- [ ] ¿Ningún endpoint sensible es accesible sin autenticación?
- [ ] ¿El JWT_SECRET tiene ≥32 caracteres y no está hardcodeado?
- [ ] ¿El CORS restringe explícitamente los orígenes permitidos?
- [ ] ¿Swagger UI está deshabilitado si `ENVIRONMENT != development`?
- [ ] ¿Los tokens expiran correctamente (access: 15min, refresh: 7d)?
- [ ] ¿Rate limiting activo en `/login`, `/register`, `/forgot-password`?
- [ ] ¿No hay `dependencyCheckAnalyze` con CVEs de severidad ALTA sin justificación?

Módulo a auditar: $arg
