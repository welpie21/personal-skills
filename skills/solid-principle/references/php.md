# PHP — SOLID conventions

## One class per file
Idiomatic in PHP and effectively enforced by PSR-4 autoloading: one `class` / `interface` / `trait` / `enum` per file, the filename equal to the type name, and the namespace mirroring the directory path. This is the norm — you rarely need the Step 2 conflict conversation for PHP unless a legacy project bundles types.

## Abstraction
- `interface` for contracts (the default choice for a boundary).
- `abstract class` when implementations genuinely share behavior/state.
- Type-hint constructors and methods against the **interface**, never the concrete class. That single habit delivers most of Dependency Inversion.

## Dependency injection
Constructor injection is the standard. Let the framework's container resolve dependencies:
- **Laravel:** bind interface → implementation in a service provider (`app/Providers/*ServiceProvider.php`) via `$this->app->bind(FooInterface::class, Foo::class)`, then type-hint `FooInterface` and let the container autowire it.
- **Symfony:** autowiring resolves type-hints automatically; bind an interface to a concrete in `config/services.yaml` when there's more than one implementation.

## Framework layout
- **Laravel:** keep controllers in `app/Http/Controllers` **thin** — push real logic into single-purpose Services (`app/Services`), Actions (`app/Actions`), or Repositories (`app/Repositories`). Models in `app/Models`. Interfaces commonly in `app/Contracts`. Validation in Form Requests. Always defer to folders the project already uses.
- **Symfony:** Services in `src/Service`, thin Controllers in `src/Controller`, Entities in `src/Entity`, Repositories in `src/Repository`. Interfaces in `src/Contract` or beside their implementations.
- **Slim / plain PHP:** organize by layer (`Domain`, `Application`, `Infrastructure`) or by feature; still one class per file under PSR-4.

## How SOLID shows up in PHP
- **SRP** → thin controllers; each service does one thing.
- **DIP** → type-hint interfaces, bind them in the container.
- **ISP** → small, role-based contracts rather than one god-interface.
- **OCP** → add a new implementation of a contract instead of editing an existing service.

## Detection hints
- Framework: inspect `composer.json` `require` — `laravel/framework` vs `symfony/*` vs `slim/slim`.
- Find files with more than one type (one-class-per-file violations):
  `grep -rcE '^\s*(final |abstract )?(class|interface|trait|enum) ' app src | grep -vE ':(0|1)$'`
