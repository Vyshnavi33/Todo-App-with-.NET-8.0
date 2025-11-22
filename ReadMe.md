# MyWebApp - Minimal Todo API

A small ASP.NET Core minimal API that demonstrates a simple in-memory "Todo" service with basic CRUD endpoints, request validation, and a rewrite rule. This project is intentionally minimal and useful as a learning example or a starting point for a more feature-complete service.

> Program entry: `WebApplication1/Program.cs`

## Features

- Minimal API (single-file Program.cs)
- In-memory task store (no external database)
- Endpoints:
  - GET /todos — list all todos
  - GET /todos/{id} — get a todo by id (returns 404 if not found)
  - POST /todos — create a new todo (with validation via an endpoint filter)
  - DELETE /todos/{id} — delete a todo by id
- Redirect rule: requests to `/tasks/...` are redirected to `/todos/...`
- Basic validation:
  - DueDate cannot be in the past
  - New todos cannot be submitted as already completed

## Requirements

- .NET SDK 6.0 or later (the project uses ASP.NET Core minimal APIs — .NET 6+)
- A terminal / command prompt

Tested with:
- dotnet SDK 6 / 7 (should work on either; use the SDK that matches your project settings)

## Running

1. Clone the repository (or copy the code into a new project).
2. Restore and run:

```bash
dotnet restore
dotnet run --project WebApplication1
```

By default the application will listen on the configured ASP.NET Core URL (see console output). You can also set `ASPNETCORE_URLS` to change the listening address, for example:

```bash
ASPNETCORE_URLS="http://localhost:5000" dotnet run --project WebApplication1
```

## API

Model:
- Todo (record)
  - int Id
  - string Name
  - DateTime DueDate
  - bool IsCompleted

Endpoints:

- GET /todos
  - Response: 200 OK with JSON array of Todo items.

- GET /todos/{id}
  - Response:
    - 200 OK with Todo JSON when found
    - 404 Not Found when not found

- POST /todos
  - Request body: JSON representation of a Todo
  - Validation (endpoint filter):
    - DueDate must not be in the past
    - IsCompleted must be false on creation
  - Response:
    - 201 Created with the submitted Todo
    - 400 Bad Request with validation details if validation fails

- DELETE /todos/{id}
  - Response:
    - 204 No Content (even if id did not exist — the in-memory implementation removes matching items)

Redirect:
- `app.UseRewriter(new RewriteOptions().AddRedirect("tasks/(.*)", "todos/$1"));`
  - Requests to `/tasks/...` will be redirected (302) to `/todos/...`

## Example Requests

List todos:
```bash
curl http://localhost:5000/todos
```

Get a todo:
```bash
curl http://localhost:5000/todos/1
```

Create a todo:
```bash
curl -X POST http://localhost:5000/todos \
  -H "Content-Type: application/json" \
  -d '{"Id":1, "Name":"Buy milk", "DueDate":"2025-12-01T12:00:00Z", "IsCompleted":false}'
```

Delete a todo:
```bash
curl -X DELETE http://localhost:5000/todos/1
```

## Implementation notes & limitations

- Data persistence: The `InMemoryTaskService` stores data in a process-local list. All data is lost when the process restarts. Replace with a database or persistent store for production.
- Concurrency: The in-memory List is not synchronized; concurrent requests that modify the list could cause issues in highly concurrent scenarios. Add proper locking or a thread-safe collection when needed.
- Id handling: The sample expects the caller to supply the `Id` in the POST body. A production-ready API would typically auto-generate unique IDs on the server.
- Validation: Basic validation is implemented using an endpoint filter. Extend this to use model validation attributes or a shared validation library as the app grows.
- Error handling: The minimal app returns basic typed results. Consider adding global exception handling, structured logging, and more detailed responses.

## How the code is organized

All logic is in `Program.cs`:

- Todo record defines the data shape.
- ITaskService defines the service contract.
- InMemoryTaskService is a simple in-memory implementation.
- Minimal API mappings with `MapGet`, `MapPost`, and `MapDelete`.
- An endpoint filter on POST is used to validate incoming Todo objects before creation.
- A URL rewriting rule redirects `/tasks/...` to `/todos/...`.
- Basic console logging middleware prints start/finish logs for each request.

## Suggestions for next steps

- Persist data to a database (e.g., SQLite, SQL Server, PostgreSQL) using EF Core or Dapper.
- Add integration tests for endpoints.
- Improve validation with FluentValidation or data annotations.
- Add logging with Microsoft.Extensions.Logging and structured logs.
- Add OpenAPI/Swagger support (Add `builder.Services.AddEndpointsApiExplorer()` and `builder.Services.AddSwaggerGen()`).

## License

This example is provided as-is for learning purposes. Add a license if you plan to publish or reuse this project.

