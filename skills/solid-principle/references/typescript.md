# JavaScript / TypeScript — SOLID conventions

## One class per file
A common and healthy convention: one primary export per file, filename matching the project's casing (PascalCase or kebab-case — match what's there). TypeScript `interface`s and `type`s are cheap and structural, so lean on them freely for contracts.

Note: React/Vue and much modern TS are **function-first**, not class-first. Don't force classes where the framework is functional (see below) — apply "one unit per file" to the component/hook/module instead.

## Abstraction
- `interface` (or `type`) for contracts — the default boundary in TS.
- `abstract class` only when implementations share real behavior.
- Depend on the interface in constructors/parameters, not the concrete class.

## Dependency injection
- **NestJS:** first-class DI. Mark providers `@Injectable()`, wire them in a module. To depend on an abstraction, use an injection token or `{ provide: FooService, useClass: RealFoo }` and inject the interface/abstract type. Keep controllers thin; logic lives in services.
- **Express / plain Node:** no built-in container. Inject dependencies by passing them into constructors or factory functions; adopt a container (`tsyringe`, `inversify`) only if the project already uses one.
- **Angular:** built-in DI via providers and `InjectionToken`; inject services into components/other services.

## Framework layout
- **NestJS:** feature modules, each with `*.module.ts`, thin `*.controller.ts`, `*.service.ts`, DTOs in `dto/`, entities, and interfaces near their consumers. One class per file throughout.
- **Express / plain Node:** organize by layer (`controllers`, `services`, `repositories`, `domain`) or by feature. Keep route handlers thin and delegate to services.
- **React:** one component per file. Apply SOLID at the module level — extract logic into custom hooks and plain service modules, depend on abstractions for external boundaries (e.g. an `ApiClient` interface with a fetch-based implementation), and keep components focused on rendering. Don't introduce classes just to look "OO".
- **Angular / Vue:** one component/service per file; put cross-cutting logic in injectable services / composables.

## How SOLID shows up in TS
- **SRP** → thin controllers/components; focused services and hooks.
- **DIP** → inject interfaces (NestJS tokens; constructor params elsewhere).
- **ISP** → small interfaces; TS structural typing makes these painless.
- **OCP** → add a new implementation of an interface rather than editing a working one.

## Detection hints
- Framework: inspect `package.json` deps — `@nestjs/core`, `express`, `next`, `react`, `@angular/core`, `vue`.
- Find files exporting more than one class:
  `grep -rc 'export \(default \)\?\(abstract \)\?class ' src | grep -vE ':(0|1)$'`
