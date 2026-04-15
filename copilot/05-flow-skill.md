Perfect — here is the **v3 pack**, ready to paste.

This adds:

* `testing-junit-mockito` skill
* `frontend-ui-patterns` skill
* a fully completed sample feature under `docs/requirements/001-sample-feature/`
* a PR template
* 6 specialized prompts

---

# Suggested v3 additions

```text
.github/
├─ PULL_REQUEST_TEMPLATE.md
├─ prompts/
│  ├─ create-endpoint.prompt.md
│  ├─ refactor-mongo-query.prompt.md
│  ├─ review-openapi-contract.prompt.md
│  ├─ generate-test-plan.prompt.md
│  ├─ prepare-pr-description.prompt.md
│  └─ implement-from-feature-docs.prompt.md
└─ skills/
   ├─ testing-junit-mockito/
   │  └─ SKILL.md
   └─ frontend-ui-patterns/
      └─ SKILL.md

docs/
└─ requirements/
   └─ 001-sample-feature/
      ├─ requerimiento.md
      ├─ analisis-funcional.md
      ├─ analisis-tecnico.md
      ├─ plan-implementacion.md
      ├─ prompt-implementacion.md
      └─ validacion.md
```

---

# 1) `.github/skills/testing-junit-mockito/SKILL.md`

````md
---
name: Testing JUnit Mockito
description: Use this skill when creating or reviewing backend tests with JUnit 5 and Mockito.
---

# Testing JUnit Mockito Skill

Use this skill when implementing or reviewing backend tests.

## Goals

- keep tests readable
- keep tests focused on behavior
- reduce brittle mocking
- improve regression safety
- make tests easy to review and maintain

---

## Core principles

- test behavior, not implementation details
- keep one main scenario per test
- use descriptive names
- avoid over-mocking
- verify only what matters
- prefer readability over cleverness

---

## Recommended structure

Prefer:

1. arrange / given
2. act / when
3. assert / then

Use comments when useful.

## Display name style

Prefer multiline display names when they improve readability.

Example:

```java
@DisplayName("""
    given valid request
    when createOrder is invoked
    then returns created response
    """)
````

---

## What to test by layer

### Service tests

Focus on:

* business rules
* branching behavior
* exception behavior
* collaboration with repositories and other services

Typical examples:

* entity exists vs not exists
* valid state vs invalid state
* success vs conflict
* simple orchestration correctness

### Controller tests

Focus on:

* endpoint mapping
* request validation
* response status
* response payload
* error handling integration

Typical examples:

* valid request returns expected status
* invalid payload triggers validation error
* service exception mapped by ControllerAdvice

### Repository-related tests

Only add repository-focused tests when:

* the project already uses them
* custom repository logic is complex
* query correctness is critical

---

## Mockito guidance

Use Mockito to isolate collaborators, not to mock everything.

### Good uses

* repository collaborators
* external clients
* downstream services
* gateway abstractions

### Avoid mocking

* DTOs
* simple value objects
* collections
* trivial domain objects unless strictly necessary

### Verification guidance

Verify only relevant calls.

Good:

* repository save invoked once when create succeeds
* downstream client not invoked on validation failure

Avoid:

* verifying every getter-like interaction
* verifying call order unless behavior depends on it

---

## Parameterized tests

Prefer parameterized tests when:

* the same business rule is evaluated with multiple values
* duplication is reduced
* readability remains good

Good cases:

* invalid enum values
* boundary values
* status transitions
* validation scenarios

Avoid parameterized tests when:

* scenario setup is complex
* each row needs extensive explanation
* readability gets worse

---

## Assertions

Prefer assertions that express intent clearly.

Good:

* assertEquals expected business values
* assertThrows for failure scenarios
* assertThat style if the project already uses AssertJ

Avoid:

* weak assertions that only check not-null unless that is the real requirement

---

## Test data construction

Prefer:

* builders
* factory methods
* helper methods with meaningful defaults

Avoid:

* noisy repeated object construction in every test
* giant setup blocks unrelated to the scenario

---

## Practical review checklist

Before finishing a test, ask:

* what behavior is being protected?
* is the name clear?
* is the setup minimal?
* is the assertion meaningful?
* is the mock usage justified?
* does this test fail for the right reason?

---

## Example: service success test

```java
@ExtendWith(MockitoExtension.class)
class CertificateServiceTest {

    @Mock
    private CertificateRepository repository;

    @InjectMocks
    private CertificateService service;

    @Test
    @DisplayName("""
        given existing certificate
        when getById is invoked
        then returns mapped response
        """)
    void shouldReturnMappedResponse() {
        // given
        CertificateDocument document = CertificateDocument.builder()
            .id("123")
            .status("ACTIVE")
            .build();

        when(repository.findById("123")).thenReturn(Optional.of(document));

        // when
        CertificateResponseDto result = service.getById("123");

        // then
        assertEquals("123", result.getId());
        assertEquals("ACTIVE", result.getStatus());
    }
}
```

## Example: exception test

```java
@Test
@DisplayName("""
    given missing certificate
    when getById is invoked
    then throws not found exception
    """)
void shouldThrowWhenCertificateDoesNotExist() {
    // given
    when(repository.findById("404")).thenReturn(Optional.empty());

    // when / then
    assertThrows(CertificateNotFoundException.class, () -> service.getById("404"));
}
```

## Example: parameterized test

```java
@ParameterizedTest
@ValueSource(strings = {"", " ", "INVALID"})
@DisplayName("""
    given invalid type
    when validating request
    then request is rejected
    """)
void shouldRejectInvalidType(String type) {
    // given
    CertificateRequestDto request = CertificateRequestDto.builder()
        .type(type)
        .build();

    // when / then
    assertThrows(IllegalArgumentException.class, () -> validator.validate(request));
}
```

````

---

# 2) `.github/skills/frontend-ui-patterns/SKILL.md`

```md
---
name: Frontend UI Patterns
description: Use this skill when implementing or reviewing Angular UI code, service integration, typed contracts, forms, tables, and state handling.
---

# Frontend UI Patterns Skill

Use this skill when working on Angular frontend changes.

## Goals

- keep UI changes incremental
- keep components readable
- keep backend integration typed
- keep state handling understandable
- make screens easy to evolve

---

## Core principles

- prefer clear responsibilities
- keep components focused
- keep services responsible for backend communication
- keep templates readable
- keep types explicit
- reuse existing UI patterns

---

## Suggested layering

### Component
Responsible for:
- orchestrating view state
- reacting to user actions
- calling services
- connecting data to template

Should avoid:
- large backend mapping logic
- duplicated HTTP logic
- excessive business rules

### Service
Responsible for:
- backend communication
- typed request/response handling
- reusable calls
- integration with interceptors or shared config

### Model / interface
Responsible for:
- explicit API contract typing
- preserving clarity between request and response shapes

---

## UI state guidance

When useful, think explicitly about:

- loading state
- success state
- empty state
- error state

Do not assume the happy path is enough.

---

## Forms guidance

For forms:
- initialize clearly
- keep validation understandable
- prefer explicit submit flow
- show useful feedback
- do not hide important logic inside templates

---

## Table/list guidance

When displaying backend data:
- define typed row shape
- handle empty results
- keep formatting logic simple
- avoid mixing raw backend payload structure directly into complex templates if mapping would improve readability

---

## Service integration guidance

Prefer:
- typed methods
- clear request object names
- clear response object names
- centralized HTTP behavior when the project already has it

Avoid:
- scattered raw `HttpClient` usage across unrelated components if a service exists
- unnecessary duplication of endpoint paths

---

## Template guidance

Prefer:
- readable conditions
- readable loops
- reusable fragments/components when repetition appears
- simple expressions

Avoid:
- complex inline transformations
- deeply nested logic in HTML

---

## Practical review checklist

Before finishing a frontend change, ask:

- is the backend contract typed?
- is the loading state handled?
- is the empty state handled?
- is the error state handled?
- is the component still readable?
- is the service the right place for HTTP logic?
- is the template easy to understand?

---

## Example: typed service

```ts
export interface CertificateRequest {
  dni: number;
  sexo: string;
  tipo: string;
}

export interface CertificateResponse {
  id: string;
  status: string;
}

@Injectable({ providedIn: 'root' })
export class CertificateService {
  constructor(private readonly http: HttpClient) {}

  createCertificate(request: CertificateRequest): Observable<CertificateResponse> {
    return this.http.post<CertificateResponse>('/api/certificates', request);
  }
}
````

## Example: component usage

```ts
@Component({
  selector: 'app-certificate-form',
  templateUrl: './certificate-form.component.html'
})
export class CertificateFormComponent {
  loading = false;
  errorMessage: string | null = null;
  response: CertificateResponse | null = null;

  constructor(private readonly certificateService: CertificateService) {}

  submit(request: CertificateRequest): void {
    this.loading = true;
    this.errorMessage = null;

    this.certificateService.createCertificate(request).subscribe({
      next: value => {
        this.response = value;
        this.loading = false;
      },
      error: () => {
        this.errorMessage = 'Could not create certificate';
        this.loading = false;
      }
    });
  }
}
```

## Example review questions

When reviewing Angular code, ask:

* should this logic be moved from template to component?
* should this HTTP call be in a service?
* is the contract typed explicitly?
* are loading and error states handled?
* is this change aligned with existing UI patterns?

````

---

# 3) `.github/PULL_REQUEST_TEMPLATE.md`

```md
## Summary

Describe briefly what this PR changes.

## Requirement / Context

- Ticket:
- Feature folder:
- Main goal:

## Changes included

- [ ] Backend
- [ ] Frontend
- [ ] Mongo query / aggregation
- [ ] OpenAPI / API contract
- [ ] Tests
- [ ] Documentation

## Main implementation notes

List the key design decisions taken in this PR.

- Decision 1:
- Decision 2:
- Decision 3:

## Contract impact

Describe any API request/response or behavior changes.

- Endpoint(s):
- Request impact:
- Response impact:
- Error handling impact:
- Backward compatibility notes:

## Testing performed

- [ ] Unit tests
- [ ] Manual testing
- [ ] Existing tests pass
- [ ] Edge cases reviewed

Details:
- Test 1:
- Test 2:
- Test 3:

## Risks / things to review carefully

- Risk 1:
- Risk 2:
- Risk 3:

## Follow-up items

- Follow-up 1:
- Follow-up 2:
````

---

# 4) Specialized prompt files

## `.github/prompts/create-endpoint.prompt.md`

```md
Create or extend a backend endpoint for this feature.

Before coding:
1. summarize the endpoint purpose
2. identify whether the endpoint is new or an extension of an existing one
3. list impacted files
4. state assumptions if needed

Implementation expectations:
- Spring Boot controller
- DTO-based request/response contract
- service layer orchestration
- repository/custom repository for persistence logic
- OpenAPI annotations
- validation where appropriate
- reuse existing ControllerAdvice and exception patterns
- add or update tests if relevant

Constraints:
- keep changes incremental
- do not make unrelated refactors
- preserve backward compatibility unless explicitly requested

After implementation, summarize:
- files changed
- endpoint contract
- remaining risks
- next logical step
```

---

## `.github/prompts/refactor-mongo-query.prompt.md`

```md
Refactor the selected MongoDB query or aggregation safely.

Goals:
- improve readability
- improve maintainability
- preserve behavior
- align with project Mongo patterns

Before changing code:
1. explain the current query intent
2. identify the current problems
3. list impacted files
4. state behavior that must remain unchanged

During refactor:
- prefer programmatic MongoTemplate logic for complex cases
- keep edge cases explicit
- keep match/projection/group/sort stages understandable
- do not move Mongo logic into controller layer

After refactor, provide:
1. summary of improvements
2. edge cases to verify
3. test suggestions if needed
```

---

## `.github/prompts/review-openapi-contract.prompt.md`

```md
Review the OpenAPI / Swagger contract of the selected feature.

Check:
1. endpoint naming clarity
2. method correctness
3. request DTO clarity
4. response DTO clarity
5. validation/documentation alignment
6. status code consistency
7. error response consistency
8. backward compatibility concerns

Output:

## Contract strengths
- ...

## Contract issues
- ...

## Recommended adjustments
- ...

Rules:
- be practical
- prefer consistency with the repository
- focus on API usability and contract safety
```

---

## `.github/prompts/generate-test-plan.prompt.md`

```md
Generate a practical test plan for this feature.

Use as input:
- feature requirement docs
- impacted backend/frontend areas
- API contract
- business rules

Produce:

## Backend unit tests
- ...

## Controller / API tests
- ...

## Mongo persistence behavior to verify
- ...

## Frontend behavior to verify
- ...

## Edge cases
- ...

## Manual verification suggestions
- ...

Rules:
- be concrete
- prefer meaningful scenarios over exhaustive noise
- align with JUnit 5 + Mockito for backend
- include failure scenarios, not only happy path
```

---

## `.github/prompts/prepare-pr-description.prompt.md`

```md
Prepare a pull request description for the current branch.

Include:

1. concise summary
2. requirement context
3. main design decisions
4. impacted areas
5. contract impact
6. tests performed
7. risks / reviewer attention points
8. follow-up items if any

Rules:
- keep it concise but informative
- be suitable for teammates reviewing the PR
- separate facts from follow-up ideas
- do not oversell the change
```

---

## `.github/prompts/implement-from-feature-docs.prompt.md`

```md
Implement the next logical step of the feature using the documentation folder as the source of truth.

Use:
- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md
- plan-implementacion.md
- validacion.md

Before coding:
1. identify the next smallest logical step
2. summarize the goal
3. list impacted files
4. state assumptions if any

Implementation rules:
- keep changes small and reviewable
- follow layered architecture
- reuse existing patterns
- preserve backward compatibility unless explicitly requested
- update tests and docs when appropriate

After coding:
1. summarize what changed
2. identify remaining work
3. identify risks or assumptions still open
```

---

# 5) Fully completed sample feature

## `docs/requirements/001-sample-feature/requerimiento.md`

```md
# Requerimiento

## Nombre de la funcionalidad
Consulta de certificados por DNI y tipo

## Objetivo
Permitir consultar certificados existentes a partir del DNI de una persona y un tipo de certificado, para que un operador pueda recuperar rápidamente el estado y la información básica asociada.

## Entrada
La funcionalidad expone un endpoint REST `GET` que recibe:

- `dni`: número de documento
- `tipo`: tipo de certificado

## Salida esperada
Retorna una lista de certificados coincidentes con la búsqueda, incluyendo:

- id del certificado
- dni
- tipo
- estado
- fecha de creación

## Reglas funcionales
- La búsqueda debe filtrar por DNI exacto.
- La búsqueda debe filtrar por tipo exacto.
- Si no existen resultados, debe retornar una lista vacía.
- No debe fallar por ausencia de resultados.
- Si los parámetros requeridos no están presentes, la request debe considerarse inválida.

## Restricciones
- Debe respetar el estilo REST ya existente en el proyecto.
- Debe usar DTOs para el contrato.
- Debe mantener el código en capas.
- La persistencia usa MongoDB.

## Supuestos
- Existe una colección Mongo de certificados.
- El campo `dni` se almacena como número.
- El campo `tipo` se almacena como string.
- Ya existe un mecanismo global de manejo de errores con `@ControllerAdvice`.
```

---

## `docs/requirements/001-sample-feature/analisis-funcional.md`

```md
# Análisis funcional

## Resumen funcional
Se agrega una consulta para recuperar certificados existentes mediante dos criterios obligatorios: DNI y tipo. La funcionalidad está orientada a búsquedas precisas y devuelve una lista de resultados, incluso si normalmente la cardinalidad esperada fuera baja.

## Escenario principal
1. Un cliente invoca el endpoint con un `dni` válido y un `tipo` válido.
2. El sistema valida la presencia de ambos parámetros.
3. El sistema consulta la base Mongo filtrando por ambos criterios.
4. El sistema retorna una lista con los certificados encontrados.

## Casos alternativos
- Existen múltiples certificados para el mismo DNI y tipo: se retornan todos.
- No existen certificados para el criterio indicado: se retorna una lista vacía.

## Casos de error
- Falta el parámetro `dni`.
- Falta el parámetro `tipo`.
- El tipo de dato recibido no es válido para el parámetro esperado.

## Edge cases
- `tipo` recibido vacío o en blanco.
- `dni` con valor no positivo.
- Registros parcialmente inconsistentes en base.
- Diferencias de mayúsculas/minúsculas en `tipo` si la regla funcional exige exactitud literal.

## Impacto funcional
- Backend: sí, se agrega endpoint, service y acceso a persistencia.
- Frontend: opcional, solo si alguna pantalla consume la nueva búsqueda.
- Persistencia: sí, se agrega consulta por dos criterios.
- Contrato API: sí, nuevo endpoint GET.
```

---

## `docs/requirements/001-sample-feature/analisis-tecnico.md`

```md
# Análisis técnico

## Componentes impactados
- controller de certificados
- service de certificados
- repository de certificados
- dto de respuesta de búsqueda
- documentación OpenAPI
- tests de service y controller

## Diseño propuesto
Se incorpora un endpoint GET `/api/certificados` con parámetros obligatorios `dni` y `tipo`.

La capa controller recibe ambos parámetros y delega en un service.
El service orquesta la búsqueda y mapea documentos a DTOs de respuesta.
La persistencia se resuelve con un custom repository o con un método de repositorio simple según la complejidad actual del proyecto. Si la consulta es directa y estable, puede iniciarse con repositorio estándar. Si ya existe lógica dinámica o futura evolución prevista, conviene custom repository.

## Contrato API
- Endpoint: `/api/certificados`
- Método: `GET`
- Parámetros:
  - `dni`: Long requerido
  - `tipo`: String requerido
- Response:
  - `200 OK` con lista de `CertificadoResumenResponseDto`
- Errores esperados:
  - `400 Bad Request` por parámetros inválidos
  - `500` solo para errores no controlados

## Persistencia
Primera opción:
- método de repositorio: `findByDniAndTipo(Long dni, String tipo)`

Segunda opción si el proyecto privilegia custom repositories:
- custom repository con `MongoTemplate`
- `Query` con `Criteria.where("dni").is(dni)` y `Criteria.where("tipo").is(tipo)`

## Validaciones
- `dni` requerido
- `dni` mayor que cero
- `tipo` requerido
- `tipo` no vacío

## Manejo de errores
- Reutilizar `@ControllerAdvice` existente
- Reutilizar excepciones de validación ya presentes si existen
- No crear manejo ad hoc en controller

## Testing
- Unit tests de service
- Tests de controller para parámetros válidos e inválidos
- Si aplica al proyecto, test de repositorio custom

## Riesgos o consideraciones
- Definir si `tipo` es case-sensitive o normalizado
- Confirmar si el endpoint debe paginar en el futuro
- Verificar si existen índices por `dni` y `tipo`
```

---

## `docs/requirements/001-sample-feature/plan-implementacion.md`

```md
# Plan de implementación

## Objetivo
Agregar una búsqueda REST de certificados por DNI y tipo, manteniendo arquitectura en capas y contrato explícito.

## Pasos propuestos

1. Crear DTO de respuesta para el resumen del certificado.
2. Agregar o extender el controller para exponer el endpoint GET con parámetros requeridos.
3. Agregar el método en service para orquestar la búsqueda y el mapeo.
4. Implementar la búsqueda en repository o custom repository.
5. Agregar anotaciones OpenAPI.
6. Verificar validaciones y manejo de errores.
7. Agregar tests de service y controller.
8. Actualizar documentación si corresponde.

## Orden sugerido
- Primero: contrato y DTO
- Luego: controller + service
- Luego: repository
- Finalmente: tests y documentación

## Riesgos por paso
- Paso 2: riesgo de inconsistencia con endpoints existentes
- Paso 4: riesgo de definir la lógica en una capa inadecuada
- Paso 7: riesgo de cobertura insuficiente de validaciones

## Criterio de finalización
- Endpoint accesible y documentado
- Respuesta correcta para búsquedas válidas
- Lista vacía para ausencia de resultados
- Validaciones y errores alineados con el proyecto
- Tests agregados o actualizados
```

---

## `docs/requirements/001-sample-feature/prompt-implementacion.md`

```md
# Prompt de implementación

Implementar la funcionalidad de consulta de certificados por DNI y tipo tomando como fuente de verdad:

- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md
- plan-implementacion.md

Reglas:

- implementar en pasos pequeños
- respetar arquitectura en capas
- no hacer refactors no relacionados
- reutilizar patrones existentes del proyecto
- si cambia el contrato, actualizar DTOs y OpenAPI
- agregar o actualizar tests cuando corresponda
- reutilizar ControllerAdvice y excepciones existentes si aplica

Antes de cambiar código:
1. resumir el paso a implementar
2. listar archivos impactados
3. explicitar supuestos si hay

Después de cambiar código:
1. resumir cambios
2. indicar riesgos remanentes
3. proponer siguiente paso
```

---

## `docs/requirements/001-sample-feature/validacion.md`

```md
# Validación

## Checklist funcional
- [ ] cumple la búsqueda por DNI y tipo
- [ ] retorna lista vacía sin error cuando no hay resultados
- [ ] valida parámetros requeridos
- [ ] cubre edge cases relevantes

## Checklist técnico
- [ ] endpoint GET implementado correctamente
- [ ] DTO de respuesta explícito
- [ ] service con responsabilidad clara
- [ ] repository o custom repository implementado en la capa correcta
- [ ] OpenAPI actualizado
- [ ] manejo de errores consistente
- [ ] tests agregados o actualizados

## Verificaciones sugeridas
- Invocar endpoint con DNI y tipo válidos con resultados
- Invocar endpoint con DNI y tipo válidos sin resultados
- Invocar endpoint sin tipo
- Invocar endpoint con DNI inválido
- Revisar consistencia del contrato en Swagger

## Observaciones
Confirmar si la comparación por `tipo` debe ser exacta literal o normalizada.
```

---

# 6) How to use this v3 pack

A very strong working flow would be:

### New requirement

* use `analyze-ticket.prompt.md`
* then `generate-feature-docs.prompt.md`

### Backend implementation

* use `implement-from-feature-docs.prompt.md`
* or `create-endpoint.prompt.md`

### Mongo refinement

* use `mongo-design.prompt.md`
* then `refactor-mongo-query.prompt.md` if needed

### Testing

* use `generate-test-plan.prompt.md`
* apply `testing-junit-mockito` skill

### PR preparation

* use `prepare-pr-description.prompt.md`
* then `review-pr.prompt.md`

### Contract review

* use `review-openapi-contract.prompt.md`

---

# 7) My recommendation for the next practical step

The smartest next move is to create a **single bootstrap prompt** that tells Copilot to scaffold this full structure into a repo automatically.

Here is that prompt:

```text
Create the following Copilot support structure in this repository under .github/ and docs/requirements/:

- global instructions
- path-specific instructions
- agents
- prompt files
- skills
- feature templates

Use the repository conventions already present.
Do not modify application code.
Only create or update the support files used to standardize Copilot workflows.

Required files:
- .github/copilot-instructions.md
- .github/instructions/backend.instructions.md
- .github/instructions/tests.instructions.md
- .github/instructions/frontend.instructions.md
- .github/instructions/mongodb.instructions.md
- .github/instructions/docs.instructions.md
- .github/agents/architect.agent.md
- .github/agents/backend-spring.agent.md
- .github/agents/angular-ui.agent.md
- .github/agents/mongo.agent.md
- .github/agents/reviewer.agent.md
- .github/prompts/analyze-ticket.prompt.md
- .github/prompts/generate-feature-docs.prompt.md
- .github/prompts/implement-step.prompt.md
- .github/prompts/review-pr.prompt.md
- .github/prompts/validate-feature.prompt.md
- .github/prompts/backend-refactor.prompt.md
- .github/prompts/angular-integration.prompt.md
- .github/prompts/mongo-design.prompt.md
- .github/prompts/create-endpoint.prompt.md
- .github/prompts/refactor-mongo-query.prompt.md
- .github/prompts/review-openapi-contract.prompt.md
- .github/prompts/generate-test-plan.prompt.md
- .github/prompts/prepare-pr-description.prompt.md
- .github/prompts/implement-from-feature-docs.prompt.md
- .github/skills/spring-boot-standards/SKILL.md
- .github/skills/mongo-patterns/SKILL.md
- .github/skills/testing-junit-mockito/SKILL.md
- .github/skills/frontend-ui-patterns/SKILL.md
- .github/PULL_REQUEST_TEMPLATE.md
- docs/requirements/000-template/*
- docs/requirements/001-sample-feature/*

Content requirements:
- Tailored to Spring Boot + Angular + MongoTemplate + JUnit 5 + Mockito
- Practical and implementation-oriented
- Reusable across multiple tickets
- Focused on docs-first workflow, small implementation slices, validation, and review
```
