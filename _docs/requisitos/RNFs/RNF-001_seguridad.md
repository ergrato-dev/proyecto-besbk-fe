<!--
  ¿Qué? Requerimientos no funcionales de seguridad del sistema.
  ¿Para qué? Establecer los estándares de seguridad obligatorios que deben cumplirse
             en toda la implementación.
  ¿Impacto? Sin estos estándares, el sistema sería vulnerable a ataques como
            inyección SQL, exposición de contraseñas o acceso no autorizado.
-->

# RNF-001 — Seguridad

## Datos generales

| Campo        | Detalle                    |
| ------------ | -------------------------- |
| **ID**       | RNF-001                    |
| **Nombre**   | Seguridad del sistema      |
| **Categoría**| No funcional — Seguridad   |

---

## Requisitos

### RNF-001.1 — Hashing de contraseñas

- Todas las contraseñas deben hashearse con **BCrypt** (factor de costo 10 o superior)
  antes de almacenarse.
- Nunca se almacenan ni se transmiten contraseñas en texto plano.
- Se usa `BCryptPasswordEncoder` de Spring Security (no implementaciones propias).

### RNF-001.2 — Tokens JWT

- Los tokens de acceso (`access_token`) deben tener una duración máxima de **15 minutos**.
- Los tokens de refresco (`refresh_token`) deben tener una duración máxima de **7 días**.
- El algoritmo de firmado debe ser **HS256** (HMAC-SHA256) con una clave secreta de
  mínimo 32 caracteres.
- La clave secreta debe estar en variables de entorno y nunca hardcodeada.
- Implementación con la librería **JJWT** (java-jwt 0.12+).

### RNF-001.3 — CORS

- El backend solo debe aceptar solicitudes desde orígenes explícitamente permitidos.
- En desarrollo: `http://localhost:5173` (Vite dev server).
- En producción: solo el dominio del frontend registrado.
- Nunca usar `allowedOrigins("*")` (wildcard).
- Configurado en `SecurityConfig.kt` vía `CorsConfigurationSource`.

### RNF-001.4 — Validación de entradas

- Todos los datos provenientes de clientes deben validarse con **Bean Validation
  (Jakarta)** usando anotaciones `@Valid`, `@NotBlank`, `@Email`, `@Size`, `@Pattern`
  en los DTOs.
- La validación ocurre automáticamente en el controlador con `@Valid` en el parámetro
  del método.
- Nunca confiar en datos del cliente sin validar.

### RNF-001.5 — Prevención de inyección SQL

- Todas las consultas a la base de datos deben realizarse a través de **Spring Data JPA**
  (repositorios tipados y consultas parametrizadas automáticas).
- Prohibido el uso de SQL crudo sin parametrizar (concatenación de strings con datos
  del usuario en queries).
- Las queries `@Query` en repositorios deben usar parámetros nombrados (`:param`),
  nunca concatenación de strings.

### RNF-001.6 — Protección de endpoints

- Los endpoints privados deben estar protegidos por el filtro `JwtAuthFilter`.
- Spring Security rechaza automáticamente solicitudes sin token válido con `401`.
- Los endpoints de autenticación (`/login`, `/register`) son públicos pero con rate
  limiting.

### RNF-001.7 — Rate limiting

- Los siguientes endpoints deben tener rate limiting con **Bucket4j** para prevenir
  ataques de fuerza bruta y abuso:
  - `POST /api/v1/auth/login` — máx. 10 intentos/minuto por IP
  - `POST /api/v1/auth/forgot-password` — máx. 5 solicitudes/hora por IP
  - `POST /api/v1/contact` — máx. 5 mensajes/hora por IP

### RNF-001.8 — Mensajes de error seguros

- Los mensajes de error de autenticación deben ser genéricos — no revelar si el
  correo existe o cuál campo está mal.
- Los errores internos del servidor no deben exponer stack traces ni detalles de
  implementación al cliente.

### RNF-001.9 — Variables de entorno

- Toda información sensible (secrets JWT, credenciales de BD, configuración SMTP)
  debe almacenarse en `.env` y leerse con `@ConfigurationProperties` o `@Value`.
- El archivo `.env` no se versiona en git.
- Siempre debe existir un `.env.example` actualizado.

---

## Relación con OWASP Top 10

| Amenaza OWASP                               | Mitigación implementada                       |
| ------------------------------------------- | --------------------------------------------- |
| A01 Broken Access Control                   | Spring Security + JwtAuthFilter               |
| A02 Cryptographic Failures                  | BCrypt para contraseñas, HS256 para JWT       |
| A03 Injection                               | Spring Data JPA (queries parametrizadas)       |
| A05 Security Misconfiguration               | CORS explícito, variables de entorno          |
| A07 Identification and Authentication Failures | Rate limiting + JWT de corta duración      |

> Ver detalles completos en `_docs/conceptos/owasp-top-10.md`.
