---
description: "Crea un endpoint Spring Boot (Kotlin) completo: Controller, Service, DTO y test MockMvc. Usar cuando se necesite agregar una nueva ruta a la API."
name: "Nuevo endpoint Kotlin/Spring Boot"
argument-hint: "Describe el endpoint: método HTTP, ruta, qué hace, qué datos recibe y devuelve"
agent: "agent"
---

# Nuevo endpoint Kotlin/Spring Boot — NN Auth System (Kotlin Edition)

Crea un endpoint Spring Boot completo en Kotlin siguiendo las convenciones del proyecto.

## Convenciones obligatorias

- **Idioma del código**: inglés (nombres de clases, funciones, variables, rutas, columnas)
- **Idioma de comentarios y KDoc**: español
- **Comentarios pedagógicos**: cada bloque significativo responde ¿Qué? ¿Para qué? ¿Impacto?
- **Tipos explícitos**: obligatorios en todos los parámetros y retornos de funciones públicas
- **Cabecera de archivo**: incluir en todo archivo nuevo (ver copilot-instructions.md §3.4)
- **Gradle Wrapper**: siempre `./gradlew`, nunca `gradle` directamente
- **Data classes**: usar `data class` para DTOs y modelos — nunca Lombok (eso es Java)
- **Inyección de dependencias**: constructor injection, nunca `@Autowired` en campo

## Lo que debes generar

### 1. DTO de request con Bean Validation

Crear en `be/src/main/kotlin/com/nn/auth/dto/` o en `AuthDtos.kt` si ya existe:

```kotlin
/**
 * Archivo: AuthDtos.kt (agregar al existente, o crear NombreRequest.kt)
 * ¿Qué? DTO que encapsula los datos del request para [descripción].
 * ¿Para qué? Validar y transportar los datos del cliente al service sin exponer el modelo.
 * ¿Impacto? Sin este DTO, el modelo JPA quedaría expuesto a modificaciones directas.
 */
data class NombreRequest(

    @field:NotBlank(message = "El campo email es requerido")
    @field:Email(message = "Formato de email inválido")
    val email: String,

    @field:NotBlank(message = "La contraseña es requerida")
    @field:Size(min = 8, message = "La contraseña debe tener al menos 8 caracteres")
    @field:Pattern(regexp = ".*[A-Z].*", message = "Debe contener al menos una mayúscula")
    @field:Pattern(regexp = ".*[a-z].*", message = "Debe contener al menos una minúscula")
    @field:Pattern(regexp = ".*\\d.*", message = "Debe contener al menos un número")
    val password: String
)
```

> **Importante en Kotlin**: usar `@field:Anotación` (no `@Anotación`) para que Bean Validation
> aplique sobre el campo backing en lugar de la property Kotlin. Sin `@field:`, las validaciones
> no se ejecutan.

### 2. DTO de respuesta

```kotlin
/**
 * ¿Qué? DTO de respuesta que expone solo los campos seguros al cliente.
 * ¿Para qué? Evitar filtrar campos sensibles (hashedPassword, etc.) en la respuesta HTTP.
 * ¿Impacto? Si se devolviera el modelo JPA directamente, se expondrían datos internos.
 */
data class NombreResponse(
    val id: UUID,
    val email: String,
    val createdAt: Instant
) {
    companion object {
        /**
         * ¿Qué? Función de fábrica que convierte un modelo JPA a DTO de respuesta.
         * ¿Para qué? Centralizar la conversión para no repetir código en el servicio.
         * ¿Impacto? Si se olvidara excluir un campo, datos sensibles se expondrían en la API.
         */
        fun from(entity: NombreModelo): NombreResponse =
            NombreResponse(
                id = entity.id,
                email = entity.email,
                createdAt = entity.createdAt
            )
    }
}
```

### 3. Lógica de negocio en el Service

Agregar el método en `be/src/main/kotlin/com/nn/auth/service/AuthService.kt`:

```kotlin
/**
 * ¿Qué? [Descripción de la función].
 * ¿Para qué? [Por qué existe esta operación de negocio].
 * ¿Impacto? [Qué pasa si hay un error aquí].
 *
 * @param request DTO validado con los datos del cliente.
 * @return [descripción del retorno].
 * @throws ResponseStatusException 400 si [condición], 404 si [condición], 409 si [condición].
 */
fun nombreFuncion(request: NombreRequest): NombreResponse {
    // ¿Qué? Verificar precondición de negocio.
    // ¿Para qué? Evitar procesar datos inválidos.
    // ¿Impacto? Sin esta verificación, [consecuencia].
    val entity = nombreRepository.findByEmail(request.email)
        ?: throw ResponseStatusException(
            HttpStatus.NOT_FOUND,
            "Recurso no encontrado"
        )

    // Lógica de negocio...

    return NombreResponse.from(entity)
}
```

### 4. Endpoint en el Controller

Agregar en `be/src/main/kotlin/com/nn/auth/controller/AuthController.kt`
o en `UserController.kt` según corresponda:

```kotlin
/**
 * ¿Qué? Endpoint POST /api/v1/auth/nombre que [descripción breve].
 * ¿Para qué? [Por qué existe este endpoint en la API].
 * ¿Impacto? [Qué parte del flujo de auth habilita].
 *
 * @param request DTO anotado con @Valid — Bean Validation ejecuta las restricciones automáticamente.
 * @return ResponseEntity<NombreResponse> con status 200/201 y el resultado.
 */
@PostMapping("/nombre")
@ResponseStatus(HttpStatus.CREATED) // o HttpStatus.OK según corresponda
fun nombre(@Valid @RequestBody request: NombreRequest): NombreResponse {
    // ¿Qué? El controller solo recibe el request, valida con @Valid y delega al service.
    // ¿Para qué? Separación de responsabilidades — toda la lógica va en el Service.
    // ¿Impacto? Si se pone lógica aquí, se viola el principio Single Responsibility.
    return authService.nombreFuncion(request)
}
```

### 5. Test MockMvc

Crear o agregar en `be/src/test/kotlin/com/nn/auth/controller/AuthControllerTest.kt`:

```kotlin
/**
 * ¿Qué? Test de integración que verifica el endpoint POST /api/v1/auth/nombre.
 * ¿Para qué? Garantizar que el endpoint responde correctamente a inputs válidos e inválidos.
 * ¿Impacto? Sin este test, un bug en el endpoint podría llegar a producción sin detectarse.
 */
@Test
fun `POST nombre - should return 201 with valid data`() {
    val request = NombreRequest(
        email = "test@example.com",
        password = "ValidPass1"
    )

    mockMvc.perform(
        post("/api/v1/auth/nombre")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request))
    )
        .andExpect(status().isCreated)
        .andExpect(jsonPath("$.email").value("test@example.com"))
}

@Test
fun `POST nombre - should return 400 with invalid email`() {
    val request = mapOf("email" to "not-an-email", "password" to "ValidPass1")

    mockMvc.perform(
        post("/api/v1/auth/nombre")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request))
    )
        .andExpect(status().isBadRequest)
}
```

### 6. Verificar

```bash
# Compilar y ejecutar tests
cd be && ./gradlew test

# Ejecutar solo el test específico
./gradlew test --tests "com.nn.auth.controller.AuthControllerTest.*nombre*"

# Iniciar la app para probar en Swagger UI
./gradlew bootRun
# → http://localhost:8080/swagger-ui.html
```

Endpoint a crear: $arg
