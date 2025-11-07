---
name: dotnet8-coding-standards
description: Use when writing C# code, implementing modern .NET 8 features, or reviewing code for best practices.
---

# .NET 8 Coding Standards

Use this skill when writing C# code, implementing modern .NET 8 features, or reviewing code for best practices.

## Primary Constructors

- **ALWAYS** use primary constructors for dependency injection (C# 12/.NET 8+)
- **ALWAYS** use primary constructors for controllers, handlers, services, and repositories
- **PREFER** primary constructors over traditional constructor patterns
- **ONLY** use traditional constructors when you need complex initialization logic

```csharp
// ✅ Correct - Primary Constructor with logging
public class EntityController(IMediator mediator, ILogger<EntityController> logger)
    : ApiControllerBase
{
    [HttpGet("{id}")]
    public async Task<IActionResult> GetEntity(Guid id, CancellationToken cancellationToken = default)
    {
        logger.LogInformation("Retrieving entity with ID {EntityId}", id);

        var result = await mediator.Send(new GetEntityQuery(id), cancellationToken);

        if (result.IsSuccess)
        {
            logger.LogInformation("Successfully retrieved entity with ID {EntityId}", id);
            return HandleSuccess(result.Data!);
        }

        logger.LogWarning("Failed to retrieve entity with ID {EntityId}", id);
        return CreateProblemDetailsResponse("Failed to retrieve entity", "Entity not found");
    }
}

// ✅ Correct - Primary Constructor with multiple dependencies including logging
public class EntityService(
    IEntityRepository repository,
    IEntityDomainService domainService,
    ILogger<EntityService> logger) : IEntityService
{
    public async Task<ProcessResult<Entity>> CreateEntityAsync(CreateEntityRequest request)
    {
        logger.LogInformation("Creating entity with name {EntityName}", request.Name);

        // Implementation with proper logging
        var result = await domainService.CreateEntityAsync(request.Name, request.Description);

        if (result.IsSuccess)
        {
            logger.LogInformation("Successfully created entity with ID {EntityId}", result.Data?.Id);
        }
        else
        {
            logger.LogWarning("Failed to create entity with name {EntityName}", request.Name);
        }

        return result;
    }
}

// ❌ Incorrect - Traditional Constructor
public class EntityController : ApiControllerBase
{
    private readonly IMediator _mediator;

    public EntityController(IMediator mediator)
    {
        _mediator = mediator;
    }

    // ... rest of implementation
}
```

## Performance Guidelines

### Database Queries

- **ALWAYS** measure query performance with profiling
- **ALWAYS** use `AsNoTracking()` for read-only operations
- **ALWAYS** limit result sets with pagination
- **CONSIDER** caching for frequently accessed reference data

### Memory Management

- **ALWAYS** dispose of resources properly
- **ALWAYS** use `using` statements for disposable objects
- **ALWAYS** avoid memory leaks in long-running operations

## Security Guidelines

### Authentication & Authorization

- **ALWAYS** use JWT Bearer tokens
- **ALWAYS** validate tokens on all endpoints
- **ALWAYS** use role-based authorization where appropriate
- **NEVER** expose sensitive data in logs

### Input Validation

- **ALWAYS** validate all input parameters
- **ALWAYS** sanitize user input
- **ALWAYS** use parameterized queries
- **NEVER** trust client-side validation alone

## Documentation Standards

### API Documentation

- **ALWAYS** document all endpoints with Swagger/OpenAPI
- **ALWAYS** include example requests and responses
- **ALWAYS** document error codes and messages
- **ALWAYS** keep documentation up to date

### Code Documentation

- **ALWAYS** document complex business logic
- **ALWAYS** use XML documentation for public APIs
- **ALWAYS** explain "why" not just "what"
- **NEVER** document obvious code
