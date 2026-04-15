This extends the previous starter kit with:

* `frontend.instructions.md`
* `mongodb.instructions.md`
* `docs.instructions.md`
* 4 additional prompt files
* richer examples inside the 2 skills
* a reusable `docs/requirements/000-template/`

---

# Suggested v2 structure

```text
.github/
├─ copilot-instructions.md
├─ instructions/
│  ├─ backend.instructions.md
│  ├─ tests.instructions.md
│  ├─ frontend.instructions.md
│  ├─ mongodb.instructions.md
│  └─ docs.instructions.md
├─ agents/
│  ├─ architect.agent.md
│  ├─ backend-spring.agent.md
│  ├─ angular-ui.agent.md
│  ├─ mongo.agent.md
│  └─ reviewer.agent.md
├─ prompts/
│  ├─ analyze-ticket.prompt.md
│  ├─ generate-feature-docs.prompt.md
│  ├─ implement-step.prompt.md
│  ├─ review-pr.prompt.md
│  ├─ validate-feature.prompt.md
│  ├─ backend-refactor.prompt.md
│  ├─ angular-integration.prompt.md
│  └─ mongo-design.prompt.md
└─ skills/
   ├─ spring-boot-standards/
   │  └─ SKILL.md
   └─ mongo-patterns/
      └─ SKILL.md

docs/
└─ requirements/
   └─ 000-template/
      ├─ requerimiento.md
      ├─ analisis-funcional.md
      ├─ analisis-tecnico.md
      ├─ plan-implementacion.md
      ├─ prompt-implementacion.md
      └─ validacion.md
```

---

# 1) `.github/instructions/frontend.instructions.md`

```md
# Frontend Instructions

Apply these instructions when working on Angular frontend code.

## General frontend approach

- Follow the existing Angular project structure and naming conventions.
- Prefer incremental changes over large rewrites.
- Reuse existing modules, shared components, services, interceptors, guards, and utilities when available.
- Keep code readable and maintainable.
- Align frontend contract types with backend DTOs.

## Layering expectations

Prefer this separation of concerns:

- component: page or UI orchestration
- service: HTTP communication and backend interaction
- model/interface: typed contract with backend
- shared/ui: reusable visual pieces
- util/helper: cross-cutting frontend helpers

## Component rules

- Keep components focused.
- Avoid excessive business logic inside components.
- Avoid complex template expressions.
- Move reusable logic into services or helper functions when it improves clarity.
- Keep form initialization, validation, and submission logic understandable.

## Template rules

- Keep templates readable.
- Avoid long inline expressions.
- Prefer explicit bindings and understandable conditional rendering.
- Reuse existing style conventions and UI patterns.

## Service rules

- Services should handle backend communication.
- Avoid hardcoding URLs when environment-based configuration or centralized API configuration exists.
- Keep request and response types explicit.
- Reuse existing error-handling or interceptor patterns.

## TypeScript rules

- Prefer explicit interfaces/types for backend payloads.
- Use descriptive names.
- Avoid unnecessary `any`.
- Keep models aligned with backend contracts.

## Practical expectations

When implementing a frontend requirement, prefer this order:

1. define or update interfaces/models
2. update or create service methods
3. update component logic
4. update template
5. handle validation and user feedback
6. update tests if the project uses them

## Integration expectations

When the frontend depends on backend changes:

- verify request payload shape
- verify response payload shape
- verify loading state behavior
- verify error state behavior
- verify empty-state handling where relevant
```

---

# 2) `.github/instructions/mongodb.instructions.md`

```md
# MongoDB Instructions

Apply these instructions when working on MongoDB persistence logic.

## General rules

- Use Spring Data repositories for simple CRUD.
- Use custom repository implementations with MongoTemplate for non-trivial queries, aggregations, projections, or dynamic filtering.
- Keep Mongo-specific code out of controllers.
- Keep persistence concerns out of the API contract layer.

## Query design

When building queries:

- be explicit about optional filters
- be explicit about null and missing values
- be explicit about empty arrays
- be explicit about sorting and pagination where relevant

Prefer readable query construction over condensed code.

## Aggregation design

For aggregation pipelines:

- keep each stage understandable
- prefer programmatic construction for complex cases
- consider whether `$match` can be applied earlier
- consider null and missing-field behavior
- keep projection logic explicit
- keep grouping logic easy to review

## Updates

For updates:

- keep update intent explicit
- be careful with nested fields and arrays
- avoid accidental overwrites
- prefer maintainable update logic over clever shortcuts

## Custom repository expectations

When logic becomes non-trivial, prefer:

- repository interface
- custom repository interface
- custom repository implementation

This keeps the service layer clean and makes persistence logic easier to evolve.

## Performance awareness

When relevant, consider:

- likely index usage
- unnecessary full collection scans
- large array unwinds
- projection size
- filtering early in pipeline stages

Do not prematurely optimize, but do not ignore obvious performance risks.

## Practical expectations

When implementing Mongo logic, prefer this order:

1. define query intent
2. choose repository vs custom repository
3. implement readable query / aggregation logic
4. consider edge cases
5. verify service usage
6. add tests where applicable
```

---

# 3) `.github/instructions/docs.instructions.md`

```md
# Documentation Instructions

Apply these instructions when generating or updating project documentation.

## General principles

- Write for implementation, not for decoration.
- Prefer concise but precise documentation.
- Make decisions explicit.
- Keep documentation aligned with the current code and intended architecture.
- Avoid vague placeholders when a practical decision can be made.

## Feature documentation

For feature-level documentation, prefer the following files:

- requerimiento.md
- analisis-funcional.md
- analisis-tecnico.md
- plan-implementacion.md
- prompt-implementacion.md
- validacion.md

## Writing style

- Use structured markdown with headings and short sections.
- Prefer bullet lists only when they improve readability.
- Distinguish clearly between facts, assumptions, and open questions.
- Keep implementation plans incremental and testable.

## Technical analysis expectations

When writing technical analysis:

- identify impacted modules
- identify API contract impact
- identify persistence impact
- identify error-handling impact
- identify testing impact
- identify compatibility concerns

## Validation expectations

Validation documentation should help answer:

- does the feature satisfy the requirement?
- are edge cases covered?
- is the API contract aligned?
- are technical risks addressed?
- are tests and docs updated consistently?

## Important rule

Do not generate code when the task is documentation-only.
```

---

# 4) Additional prompt files

## `.github/prompts/validate-feature.prompt.md`

```md
Validate the implementation against the feature documentation.

Use as source of truth:
- docs/requirements/<ID-feature-name>/requerimiento.md
- docs/requirements/<ID-feature-name>/analisis-funcional.md
- docs/requirements/<ID-feature-name>/analisis-tecnico.md
- docs/requirements/<ID-feature-name>/validacion.md

Check for:

1. functional mismatches
2. missing scenarios
3. request/response contract mismatches
4. validation gaps
5. exception handling gaps
6. persistence design issues
7. test gaps
8. documentation drift

Output format:

## Findings
- ...

## Recommended fixes
- ...

Rules:
- do not silently change code
- list issues first
- be concrete and implementation-oriented
```

---

## `.github/prompts/backend-refactor.prompt.md`

```md
Refactor the selected backend code safely.

Goals:
- improve readability
- improve maintainability
- preserve behavior
- align with repository standards
- avoid unrelated changes

Before changing code:
1. summarize current problem
2. explain the refactor target
3. list impacted files
4. mention behavior that must remain unchanged

During refactor:
- preserve API contracts unless explicitly requested otherwise
- preserve business behavior
- avoid broad architectural changes unless requested
- keep layers clear
- improve names and structure where useful

After refactor:
1. summarize what changed
2. list risks to verify
3. suggest any tests that should be added or updated
```

---

## `.github/prompts/angular-integration.prompt.md`

```md
Implement or adjust Angular integration for the selected feature.

Context:
- Angular frontend
- Spring Boot backend
- typed REST contract
- incremental change preferred

Tasks:
1. analyze backend contract impact
2. identify impacted frontend files
3. align interfaces/models
4. update service methods
5. update component logic
6. update templates if needed
7. consider loading, empty, and error states

Rules:
- keep the change small and focused
- follow existing frontend structure
- avoid broad rewrites
- keep types explicit
- reuse existing services and UI patterns when possible

After implementation, summarize:
- files changed
- contract assumptions
- follow-up items if any
```

---

## `.github/prompts/mongo-design.prompt.md`

```md
Design the MongoDB persistence approach for this requirement.

Analyze:
1. the data access need
2. whether simple repository methods are enough
3. whether a custom repository with MongoTemplate is needed
4. whether aggregation is needed
5. edge cases that affect query correctness
6. likely performance concerns

Output:
- recommended repository design
- recommended query / aggregation strategy
- suggested placement in repository layers
- risks or assumptions
- optional example structure of the implementation

Rules:
- prioritize correctness and maintainability
- prefer readable programmatic queries for complex cases
- keep Mongo logic out of controller layer
```

---

# 5) Expanded Spring Boot skill

## `.github/skills/spring-boot-standards/SKILL.md`

````md
---
name: Spring Boot Standards
description: Use this skill when implementing or reviewing Spring Boot backend code, REST endpoints, DTOs, services, exception handling, and tests.
---

# Spring Boot Standards Skill

Use this skill when working on backend code for this repository.

## Goals

- keep backend code consistent
- keep layers well separated
- keep APIs explicit and documented
- keep tests meaningful
- keep changes easy to review

---

## Package and layering guidance

Typical layers:

- controller
- service
- repository
- repository custom implementation
- dto
- entity or document
- exception

### Layer responsibilities

#### Controller
Responsible for:
- endpoint mapping
- request validation entrypoint
- request/response DTO handling
- OpenAPI annotations
- translating service output into HTTP response if needed

Should avoid:
- business logic
- query logic
- persistence decisions

#### Service
Responsible for:
- business rules
- orchestration
- business validation
- calling repositories and collaborators
- deciding domain outcomes

Should avoid:
- direct HTTP concerns
- complex Mongo query construction if it belongs in a repository

#### Repository
Responsible for:
- persistence access

Use:
- standard repository for simple CRUD
- custom repository implementation for complex query logic

---

## DTO guidance

Use DTOs for API contracts.

### Request DTOs
Prefer:
- validation annotations where relevant
- focused fields
- clear names

### Response DTOs
Prefer:
- explicit fields
- stable contract shape
- not leaking persistence concerns

---

## OpenAPI guidance

For public endpoints:
- add clear summary
- add clear description when useful
- document request body
- document response codes
- document validation or error responses when the project already uses that pattern

---

## Exception handling guidance

Prefer:
- domain-specific exceptions when useful
- reusing existing exception hierarchy
- reusing existing `@ControllerAdvice`

Do not:
- create ad hoc error handling inside each controller unless the project explicitly does that

---

## Code style guidance

- constructor injection
- Lombok where appropriate
- avoid star imports
- use clear naming
- prefer readable code over compact cleverness

---

## Testing guidance

### Service tests
Usually test:
- happy path
- branching path
- exception path

### Controller tests
Usually test:
- status codes
- validation
- response payload
- error handling integration

### Mockito
Prefer:
- focused stubbing
- focused verification
- no over-mocking

### Parameterized tests
Use when:
- the same rule is exercised with multiple input/output combinations
- readability improves

---

## Practical implementation checklist

Before implementation:
- understand requirement
- identify impacted layers
- decide DTOs
- decide service responsibilities
- decide repository/custom repository responsibilities

Before finishing:
- endpoint documented
- validation aligned
- exceptions handled
- tests updated
- no unrelated refactor introduced

---

## Example: controller skeleton

```java
@RestController
@RequestMapping("/api/example")
@RequiredArgsConstructor
@Tag(name = "Example")
public class ExampleController {

    private final ExampleService exampleService;

    @Operation(summary = "Get example by id")
    @GetMapping("/{id}")
    public ResponseEntity<ExampleResponseDto> getById(@PathVariable String id) {
        return ResponseEntity.ok(exampleService.getById(id));
    }
}
````

## Example: service skeleton

```java
@Service
@RequiredArgsConstructor
public class ExampleService {

    private final ExampleRepository exampleRepository;

    public ExampleResponseDto getById(String id) {
        ExampleDocument document = exampleRepository.findById(id)
            .orElseThrow(() -> new ExampleNotFoundException(id));

        return ExampleResponseDto.builder()
            .id(document.getId())
            .name(document.getName())
            .build();
    }
}
```

## Example: custom repository split

```java
public interface ExampleRepositoryCustom {
    List<ExampleDocument> search(ExampleSearchCriteria criteria);
}
```

```java
@Repository
@RequiredArgsConstructor
public class ExampleRepositoryCustomImpl implements ExampleRepositoryCustom {

    private final MongoTemplate mongoTemplate;

    @Override
    public List<ExampleDocument> search(ExampleSearchCriteria criteria) {
        Query query = new Query();

        if (criteria.name() != null && !criteria.name().isBlank()) {
            query.addCriteria(Criteria.where("name").regex(criteria.name(), "i"));
        }

        return mongoTemplate.find(query, ExampleDocument.class);
    }
}
```

## Example: test shape

```java
@ExtendWith(MockitoExtension.class)
class ExampleServiceTest {

    @Mock
    private ExampleRepository exampleRepository;

    @InjectMocks
    private ExampleService exampleService;

    @Test
    @DisplayName("""
        given existing id
        when getById is invoked
        then returns mapped response
        """)
    void shouldReturnMappedResponse() {
        // given
        ExampleDocument document = ExampleDocument.builder().id("1").name("Test").build();
        when(exampleRepository.findById("1")).thenReturn(Optional.of(document));

        // when
        ExampleResponseDto result = exampleService.getById("1");

        // then
        assertEquals("1", result.getId());
        assertEquals("Test", result.getName());
    }
}
```

````

---

# 6) Expanded Mongo skill

## `.github/skills/mongo-patterns/SKILL.md`

```md
---
name: Mongo Patterns
description: Use this skill when implementing or reviewing MongoDB persistence logic with Spring Data, MongoTemplate, Criteria, Query, Update, and Aggregation.
---

# Mongo Patterns Skill

Use this skill when the feature requires MongoDB access beyond trivial CRUD.

## Goals

- keep Mongo logic correct
- keep query code readable
- put logic in the right layer
- make complex queries maintainable

---

## When to use standard repository vs custom repository

### Use standard Spring Data repository when:
- CRUD is simple
- derived query methods are enough
- query complexity is low

### Use custom repository + MongoTemplate when:
- query conditions are dynamic
- aggregation is needed
- update logic is custom
- projection logic is custom
- readability would suffer with annotation-based queries

---

## Recommended placement

- simple CRUD: repository interface
- complex query or aggregation: custom repository implementation
- orchestration and business meaning: service
- request handling: controller

Do not put Mongo query construction in controllers.

---

## Query design guidance

When building queries:
- be explicit about optional criteria
- be explicit about null handling
- be explicit about missing fields
- be explicit about empty arrays
- be explicit about sorting and pagination when relevant

Prefer readable, step-by-step query construction over overly condensed code.

---

## Aggregation guidance

When writing an aggregation pipeline:
- keep each stage understandable
- prefer programmatic construction for complex pipelines
- add comments if a stage is not self-evident
- think about how null or missing values affect the result
- think about whether projection, unwind, group, sort, and match order are correct

Good aggregation code should be easy to debug and review.

---

## Update guidance

For updates:
- keep update intent explicit
- avoid accidental partial updates or field loss
- be careful with arrays, nested fields, and absent fields

---

## Performance and maintainability guidance

When relevant, consider:
- whether the query can use an index
- whether the aggregation will scan too much data
- whether filtering can be moved earlier in the pipeline
- whether the projection can reduce payload size

Do not prematurely optimize, but do not ignore obvious performance risks.

---

## Edge cases checklist

Always think about:
- null field
- missing field
- empty array
- duplicated values
- unexpected type
- date/time comparison behavior
- sort behavior with nulls
- partial data

---

## Practical implementation checklist

Before implementation:
- identify if CRUD is simple or not
- decide whether custom repository is needed
- identify dynamic filters
- identify result shape

Before finishing:
- query intent is clear
- edge cases considered
- correct layer used
- service behavior tested
- no Mongo logic leaked into controller

---

## Example: dynamic Query

```java
public List<OrderDocument> search(OrderFilter filter) {
    Query query = new Query();

    if (filter.status() != null) {
        query.addCriteria(Criteria.where("status").is(filter.status()));
    }

    if (filter.customerId() != null) {
        query.addCriteria(Criteria.where("customerId").is(filter.customerId()));
    }

    if (filter.fromDate() != null || filter.toDate() != null) {
        Criteria dateCriteria = Criteria.where("createdAt");

        if (filter.fromDate() != null) {
            dateCriteria = dateCriteria.gte(filter.fromDate());
        }

        if (filter.toDate() != null) {
            dateCriteria = dateCriteria.lte(filter.toDate());
        }

        query.addCriteria(dateCriteria);
    }

    return mongoTemplate.find(query, OrderDocument.class);
}
````

## Example: aggregation skeleton

```java
public List<CategoryTotalDto> totalsByCategory(LocalDate from, LocalDate to) {
    MatchOperation match = Aggregation.match(
        Criteria.where("date").gte(from).lte(to)
    );

    GroupOperation group = Aggregation.group("category")
        .sum("amount").as("total");

    ProjectionOperation project = Aggregation.project()
        .and("_id").as("category")
        .and("total").as("total");

    SortOperation sort = Aggregation.sort(Sort.by(Sort.Direction.DESC, "total"));

    Aggregation aggregation = Aggregation.newAggregation(match, group, project, sort);

    return mongoTemplate.aggregate(aggregation, "expenses", CategoryTotalDto.class)
        .getMappedResults();
}
```

## Example: custom repository structure

```java
public interface ExpenseRepositoryCustom {
    List<ExpenseSummaryDto> summarizeByCategory(ExpenseFilter filter);
}
```

```java
@Repository
@RequiredArgsConstructor
public class ExpenseRepositoryCustomImpl implements ExpenseRepositoryCustom {

    private final MongoTemplate mongoTemplate;

    @Override
    public List<ExpenseSummaryDto> summarizeByCategory(ExpenseFilter filter) {
        // build query or aggregation here
        return List.of();
    }
}
```

## Example review questions

When reviewing Mongo code, ask:

* is this logic simple enough for a repository method?
* should this be a custom repository?
* are null and missing fields handled correctly?
* is the aggregation readable?
* is `$match` applied early enough?
* is the result mapping explicit and safe?

````

---

# 7) `docs/requirements/000-template/`

## `docs/requirements/000-template/requerimiento.md`

```md
# Requerimiento

## Nombre de la funcionalidad
[Completar]

## Objetivo
[Describir qué problema resuelve y para qué se necesita]

## Entrada
[Describir request, parámetros, eventos o datos de entrada]

## Salida esperada
[Describir response, efectos esperados o resultado funcional]

## Reglas funcionales
- [Regla 1]
- [Regla 2]

## Restricciones
- [Restricción 1]
- [Restricción 2]

## Supuestos
- [Supuesto 1]
- [Supuesto 2]
````

---

## `docs/requirements/000-template/analisis-funcional.md`

```md
# Análisis funcional

## Resumen funcional
[Explicar la funcionalidad de forma clara y práctica]

## Escenario principal
1. [Paso 1]
2. [Paso 2]
3. [Paso 3]

## Casos alternativos
- [Caso alternativo 1]
- [Caso alternativo 2]

## Casos de error
- [Error 1]
- [Error 2]

## Edge cases
- [Edge case 1]
- [Edge case 2]

## Impacto funcional
- Backend: [sí/no + detalle]
- Frontend: [sí/no + detalle]
- Persistencia: [sí/no + detalle]
- Contrato API: [sí/no + detalle]
```

---

## `docs/requirements/000-template/analisis-tecnico.md`

```md
# Análisis técnico

## Componentes impactados
- [Módulo / paquete / servicio 1]
- [Módulo / paquete / servicio 2]

## Diseño propuesto
[Explicar la solución técnica elegida]

## Contrato API
- Endpoint: [ruta]
- Método: [GET/POST/etc.]
- Request: [DTO o parámetros]
- Response: [DTO]
- Errores esperados: [lista]

## Persistencia
[Explicar si usa repositorio estándar, custom repository, MongoTemplate, agregaciones, etc.]

## Validaciones
- [Validación 1]
- [Validación 2]

## Manejo de errores
[Explicar excepciones y ControllerAdvice]

## Testing
- Unit tests: [qué cubrir]
- Integration tests: [si aplica]
- Frontend tests: [si aplica]

## Riesgos o consideraciones
- [Riesgo 1]
- [Riesgo 2]
```

---

## `docs/requirements/000-template/plan-implementacion.md`

```md
# Plan de implementación

## Objetivo
[Resumen corto del objetivo técnico]

## Pasos propuestos

1. [Paso pequeño e implementable]
2. [Paso pequeño e implementable]
3. [Paso pequeño e implementable]
4. [Paso pequeño e implementable]

## Orden sugerido
- Primero: [qué conviene hacer primero]
- Luego: [qué sigue]
- Finalmente: [cierre]

## Riesgos por paso
- Paso 1: [riesgo]
- Paso 2: [riesgo]

## Criterio de finalización
- [Criterio 1]
- [Criterio 2]
- [Criterio 3]
```

---

## `docs/requirements/000-template/prompt-implementacion.md`

```md
# Prompt de implementación

Implementar esta funcionalidad tomando como fuente de verdad:

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

## `docs/requirements/000-template/validacion.md`

```md
# Validación

## Checklist funcional
- [ ] cumple el caso principal
- [ ] cubre casos alternativos relevantes
- [ ] cubre errores esperados
- [ ] cubre edge cases identificados

## Checklist técnico
- [ ] contrato API implementado correctamente
- [ ] validaciones aplicadas
- [ ] manejo de errores consistente
- [ ] persistencia implementada en la capa correcta
- [ ] tests agregados o actualizados
- [ ] documentación actualizada

## Verificaciones sugeridas
- [Verificación 1]
- [Verificación 2]
- [Verificación 3]

## Observaciones
[Notas finales]
```

---

# 8) Recommended practical flow for this v2 kit

Use this exact sequence:

### A. Requirement phase

* `architect.agent.md`
* `analyze-ticket.prompt.md`
* `generate-feature-docs.prompt.md`

### B. Backend design / coding

* `backend-spring.agent.md`
* `implement-step.prompt.md`
* `backend-refactor.prompt.md` when needed

### C. Mongo design / coding

* `mongo.agent.md`
* `mongo-design.prompt.md`

### D. Frontend integration

* `angular-ui.agent.md`
* `angular-integration.prompt.md`

### E. Review / validation

* `reviewer.agent.md`
* `validate-feature.prompt.md`
* `review-pr.prompt.md`

---

# 9) What I would add in a v3

The natural next step would be:

* `frontend-ui-patterns` skill
* `testing-junit-mockito` dedicated skill
* `docs/requirements/001-example-feature/` fully filled example
* PR template aligned with this workflow
* prompt variants for:

  * create endpoint
  * refactor Mongo query
  * review OpenAPI contract
  * generate test plan
  * prepare PR description
