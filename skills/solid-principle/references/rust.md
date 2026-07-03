# Rust — SOLID conventions

Rust has **no classes**. Data is `struct`/`enum`; behavior lives in `impl` blocks and `trait`s; a file is a module. Translate the two structural rules accordingly.

## "One type per file"
Put one primary type per module file (e.g. `user.rs` holds `User` and its `impl`s). But keep tightly-coupled companions **together** — the type, its `impl` blocks, its associated error `enum`, its builder. Scattering those across files hurts readability and helps no one. Re-export the public surface from `mod.rs` / `lib.rs`. If a project or the user wants stricter splitting than this, that's a Step 2 conversation.

## Abstraction = traits
Traits are the boundary mechanism and the way you get Dependency Inversion and Open/Closed:
- Define a `trait` for a behavior seam.
- Depend on it via generics (`fn new<R: Repo>(repo: R)`) for zero-cost static dispatch, or via `Box<dyn Trait>` / `Arc<dyn Trait>` when you need dynamic dispatch or shared ownership.
- Keep traits small and role-focused (Interface Segregation maps directly).

Don't over-abstract: a trait with a single implementation and no need for polymorphism is just indirection. Introduce a trait at a real seam — swapping implementations, or mocking for tests — not by default.

## Dependency injection
No container and no magic — DI is explicit. Pass dependencies in through a constructor function (`Type::new(dep)`) or generic parameters. Choose generics for performance, trait objects when you need heterogeneity or want to store them behind a single type.

## Framework layout
- **Axum:** handlers are async functions; shared services go in application state (`State<AppState>`), often as `Arc<dyn Trait>` fields; cross-cutting concerns as layers/middleware. Organize modules by feature; put swappable services behind traits.
- **Actix-web / Rocket:** similar shape — handler functions plus injected app state/services behind traits.
- **Module layout:** `src/lib.rs` or `src/main.rs` at the root; feature modules as folders with a `mod.rs`; one primary type per file inside.

## How SOLID shows up in Rust
- **SRP** → one focused type + its `impl` per file.
- **DIP / OCP** → depend on traits (generics or trait objects), add new `impl`s to extend.
- **ISP** → small traits, composed with trait bounds (`T: Read + Write`).
- **LSP** → every `impl` must fully honor the trait's contract.

## Detection hints
- Framework: inspect `Cargo.toml` `[dependencies]` — `axum`, `actix-web`, `rocket`, `tokio`.
- Count top-level type definitions per file (rough multi-type check):
  `grep -rc '^\(pub \)\?\(struct\|enum\|trait\) ' src | grep -vE ':(0|1)$'`
  (Expect false positives — a type plus its error enum in one file is fine; use judgment.)
