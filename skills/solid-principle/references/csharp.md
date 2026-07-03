# C# / .NET — SOLID conventions

Targets **.NET 9.0 or later** and modern C#. First confirm the project's `<TargetFramework>` is `net9.0`+ (check the `.csproj`); if it's older, flag it — some idioms below (primary constructors, collection expressions) need recent language versions. Lean on the modern feature set: **file-scoped namespaces**, **primary constructors** for injection, `record` types for DTOs and value objects, nullable reference types enabled, and the **minimal hosting model** (`Program.cs` with `WebApplication`).

## One class per file
Strongly idiomatic and often enforced (StyleCop `SA1402`, "a file may only contain a single type"; many teams turn it on). One **top-level** type per file, filename equal to the type name, namespace mirroring the folder via a file-scoped namespace (`namespace MyApp.Orders;`). Nested types and a `record` living beside the type it belongs to are fine — the rule is about top-level types. If a project already agrees with this (most do), no Step 2 question is needed.

## Abstraction
- `interface` for contracts — the default boundary. Convention is the `I` prefix (`IOrderService`, `IOrderRepository`).
- `abstract class` when implementations share real behavior or state.
- Depend on the interface in constructors, never the concrete type. .NET's DI container makes this the path of least resistance.

## Dependency injection
First-class via `Microsoft.Extensions.DependencyInjection`. Register interface → implementation in `Program.cs` and inject the interface:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
```

Use **constructor injection**, and prefer a **primary constructor** for brevity in .NET 9:

```csharp
public sealed class OrderService(IOrderRepository repository) : IOrderService
{
    public Task<IReadOnlyList<Order>> GetForUserAsync(int userId) =>
        repository.GetForUserAsync(userId);
}
```

Pick lifetimes deliberately: `AddScoped` (per request — the usual choice for web), `AddSingleton`, `AddTransient`.

## Framework layout (ASP.NET Core)
- **Minimal APIs (common in .NET 9):** define endpoints in `Program.cs` or, better, group them with `MapGroup` into endpoint classes/extension methods (e.g. `Endpoints/OrderEndpoints.cs`). Keep handlers **thin** — resolve a service from DI and delegate. Use `record` request/response DTOs and typed results (`Results<Ok<T>, NotFound>`).
- **Controller-based (MVC / Web API):** thin controllers in `Controllers/`, logic in `Services/`, data access behind an interface in `Repositories/` (or an EF Core `DbContext`). Interfaces commonly in an `Abstractions/` or `Interfaces/` folder, or beside their consumers. DTOs as `record`s.
- **Layered / Clean architecture (popular in .NET):** separate projects — `Domain`, `Application`, `Infrastructure`, `Api`/`Web` — with dependencies pointing inward (the Dependency Inversion boundary at the project level). Match this if the solution already uses it.
- **EF Core note:** a repository interface is a clean seam for testing, but some .NET teams deliberately use `DbContext` directly and consider a thin repository redundant. Read what the project does and don't impose a repository layer where they've chosen EF-direct — if in doubt, that's a Step 2 question.

## How SOLID shows up in C#
- **SRP** → thin controllers/endpoints; each service does one job.
- **DIP** → register interfaces in the container, inject them via (primary) constructors.
- **ISP** → small `I`-prefixed interfaces rather than one broad service contract.
- **OCP** → add a new implementation and register it, instead of editing a working service.
- **LSP** → each implementation fully honors its interface's contract.

## Detection hints
- Framework / target: inspect the `.csproj` — `<TargetFramework>net9.0</TargetFramework>` (or `net9.0-*`), and the SDK (`Microsoft.NET.Sdk.Web` or a `Microsoft.AspNetCore.App` framework reference) for ASP.NET Core.
- Analyzers already enforcing one-type-per-file? Check `.editorconfig` / `stylecop.json` for `SA1402`.
- Find files with more than one top-level type (rough — nested types and records add noise, so use judgment):
  `grep -rcE '^\s*(public |internal |sealed |static |abstract |partial )*(class|interface|record|struct|enum) ' --include=*.cs . | grep -vE ':(0|1)$'`
