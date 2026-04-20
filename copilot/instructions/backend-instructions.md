# Backend-specific instructions

Apply these rules to backend code.

## Architecture

- Follow controller -> service -> repository boundaries.
- Keep controllers focused on HTTP concerns.
- Keep services focused on business logic and orchestration.
- Keep repositories focused on persistence.
- Avoid placing mapping logic inline across many layers; centralize where practical.

## Persistence

When a table gains new foreign keys or relations, analyze:
- entity mappings
- repository interfaces
- custom repository implementations
- native SQL / JdbcTemplate / NamedParameterJdbcTemplate queries
- projections
- seed/test fixtures
- migration scripts

## Repositories
For repository creation follow this rules
- for basic operations prefer JPARepository with derived queries when no too comples (no more than two fields)
- if need to create a custom repository use a "fragment" pattern add the interface and the implementation and make the interface extends JPA repository


## DTOs and API

- Do not expose entities directly from controllers use a DTO for Request other for Response.
- Keep Bruno request definitions in the `bruno/` folder aligned with endpoint behavior.
- Any new endpoint should include a Bruno request example.
- Any endpoint update that changes method, path, params, headers, body, or sample payloads should update the related Bruno request.
- Keep DTO changes intentional and documented.
- If a new relation can create breaking response-shape changes, flag it explicitly.
- If compatibility matters, prefer additive changes.
- When adding or updating any public API controller endpoint, always also:
  - create or update the corresponding Bruno request
  - update required Bruno environment variables if new values are needed
  - keep the Bruno request aligned with the current request/response contract
  - mention the impacted Bruno files in the change summary
- A controller/API change is not complete unless the Bruno collection is updated accordingly.
 
## Error handling
- Always handle the exception through the @ControllerAdvice
- Specific logic exception should be in a specific logic package
- Generic custom exception like DataAccessFailureException should be in upper level shared package.

## Migrations

- Prefer safe rollout order.
- Consider whether code must run temporarily against both old and new schemas.
- Call out backfill requirements explicitly.
- Recommend indexes for new foreign keys where appropriate.

## Performance and risk

- Check for N+1 risks when introducing new relations.
- Check whether joins or fetch changes can affect large queries.
- Flag any change that can alter transaction scope or lock behavior.
