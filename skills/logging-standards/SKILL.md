---
name: logging-standards
description: Use when implementing logging, structured logging, or debugging issues.
---

# Logging Standards

Use this skill when implementing logging, structured logging, or debugging issues.

## General Logging Standards

- **ALWAYS** use structured logging with Serilog
- **ALWAYS** include correlation IDs
- **ALWAYS** log at appropriate levels (Error, Warning, Information, Debug)
- **NEVER** log sensitive data (passwords, tokens, PII)

## Controller Logging Requirements

- **ALWAYS** inject `ILogger<T>` in primary constructor
- **ALWAYS** log operation start with Information level
- **ALWAYS** log success with relevant data (counts, IDs, etc.)
- **ALWAYS** log failures with Warning level
- **ALWAYS** use structured logging with parameters

```csharp
// ✅ Correct - Controller with proper logging
public class EntityTypesController(IMediator mediator, ILogger<EntityTypesController> logger)
    : ApiControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetEntityTypes(CancellationToken cancellationToken = default)
    {
        logger.LogInformation("Retrieving all entity types");

        var processResult = await mediator.Send(new GetEntityTypesQuery(), cancellationToken);

        if (processResult.IsSuccess)
        {
            logger.LogInformation("Successfully retrieved {Count} entity types",
                processResult.Data?.Count() ?? 0);
            return HandleSuccess(processResult.Data!);
        }

        logger.LogWarning("Failed to retrieve entity types");
        return CreateProblemDetailsResponse("Failed to retrieve entity types", "An error occurred");
    }
}

// ❌ Incorrect - No logging
public class EntityTypesController(IMediator mediator) : ApiControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetEntityTypes(CancellationToken cancellationToken = default)
    {
        var processResult = await mediator.Send(new GetEntityTypesQuery(), cancellationToken);
        return processResult.IsSuccess
            ? HandleSuccess(processResult.Data!)
            : CreateProblemDetailsResponse(...);
    }
}
```

## Handler Logging Requirements

- **ALWAYS** inject `ILogger<T>` in primary constructor
- **ALWAYS** log database operations with Debug level
- **ALWAYS** log success with relevant data (counts, IDs, etc.)
- **ALWAYS** log exceptions with Error level and full exception details
- **ALWAYS** use structured logging with parameters

```csharp
// ✅ Correct - Handler with proper logging
public class GetEntityTypesQueryHandler(
    FieldsDbContext fieldsDbContext,
    ILogger<GetEntityTypesQueryHandler> logger)
    : IRequestHandler<GetEntityTypesQuery, ProcessResult<IEnumerable<EntityType>>>
{
    public async Task<ProcessResult<IEnumerable<EntityType>>> Handle(
        GetEntityTypesQuery request,
        CancellationToken cancellationToken = default)
    {
        try
        {
            logger.LogDebug("Starting to retrieve entity types from database");

            var entityTypes = await fieldsDbContext
                .EntityTypes
                .AsNoTracking()
                .OrderBy(et => et.Name)
                .Select(et => new EntityType { Id = et.EntityTypeId, Name = et.Name })
                .ToListAsync(cancellationToken);

            logger.LogInformation("Successfully retrieved {Count} entity types from database",
                entityTypes.Count);
            return ProcessResult.Success(entityTypes.AsEnumerable());
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Failed to retrieve entity types from database");
            return ProcessResult.Failure<IEnumerable<EntityType>>();
        }
    }
}

// ❌ Incorrect - No logging or poor logging
public class GetEntityTypesQueryHandler(FieldsDbContext fieldsDbContext)
    : IRequestHandler<GetEntityTypesQuery, ProcessResult<IEnumerable<EntityType>>>
{
    public async Task<ProcessResult<IEnumerable<EntityType>>> Handle(
        GetEntityTypesQuery request,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var entityTypes = await fieldsDbContext.EntityTypes.ToListAsync(cancellationToken);
            return ProcessResult.Success(entityTypes.AsEnumerable());
        }
        catch (Exception)
        {
            return ProcessResult.Failure<IEnumerable<EntityType>>();
        }
    }
}
```

## Logging Level Guidelines

- **Debug**: Detailed operation steps, database queries, internal state
- **Information**: Normal operation flow, successful operations with data
- **Warning**: Business logic failures, recoverable errors, validation failures
- **Error**: Technical exceptions, system failures, unrecoverable errors

## Structured Logging Best Practices

- **ALWAYS** use parameterized messages: `"Retrieved {Count} items"` not `"Retrieved " + count + " items"`
- **ALWAYS** include relevant context: IDs, counts, operation names
- **ALWAYS** use consistent property names: `{Count}`, `{Id}`, `{Operation}`
- **NEVER** log sensitive data: passwords, tokens, PII, connection strings
- **ALWAYS** log exceptions with full details in Error level

## Example with Full Pattern

```csharp
// Complete example with proper logging
[HttpGet("{id:guid}")]
public async Task<IActionResult> GetEntity(
    [FromRoute] Guid id,
    CancellationToken cancellationToken = default)
{
    logger.LogInformation("Retrieving entity with ID {EntityId}", id);

    try
    {
        var processResult = await mediator.Send(new GetEntityQuery(id), cancellationToken);

        if (processResult.IsSuccess)
        {
            logger.LogInformation("Successfully retrieved entity with ID {EntityId}", id);
            return HandleSuccess(processResult.Data!);
        }

        logger.LogWarning("Entity with ID {EntityId} not found", id);
        return CreateProblemDetailsResponse(
            "Entity not found",
            $"No entity exists with ID {id}",
            StatusCodes.Status404NotFound);
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "Unexpected error occurred while retrieving entity with ID {EntityId}", id);
        return CreateProblemDetailsResponse(
            "Internal server error",
            "An unexpected error occurred while processing the request",
            StatusCodes.Status500InternalServerError);
    }
}
```
