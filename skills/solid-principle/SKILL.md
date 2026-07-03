---
name: solid-principle
description: Write, structure, and refactor code so it follows the SOLID principles with a clean, maintainable layout — one class/type per file, and abstraction (interfaces, abstract classes, traits) treated as a first-class citizen. Before writing anything, this skill inspects the existing codebase and detects the framework in use so new code matches the project's conventions, and it asks the user how to proceed when the project's style conflicts with the SOLID rules. Supports PHP, JavaScript/TypeScript, C#/.NET (targeting .NET 9.0 or later), Rust, and Go. Use this skill whenever the user types `/solid-principle`, or asks for SOLID code, clean or hexagonal architecture, dependency injection, decoupled/testable design, separation of concerns, single-responsibility classes, or one-class-per-file structure — and more broadly whenever they want code written in a well-structured, maintainable way in one of these languages, even if they never say the word "SOLID".
---

# SOLID Principle Coding

Write and refactor code so it is decoupled, testable, and easy to maintain, following the SOLID principles — with two structural commitments the user cares about most: **one class/type per file**, and **abstraction as a first-class citizen**. Supported languages: PHP, JavaScript/TypeScript, C#/.NET (targeting .NET 9.0 or later), Rust, and Go.

The thing that makes this skill trustworthy is that it is *codebase-aware*. It looks at what already exists before deciding how to write anything, it follows the conventions of whatever framework the project uses, and when its opinions collide with the project's reality it asks the user rather than guessing. A skill that produces beautiful SOLID code that fights the surrounding project is worse than useless.

## Workflow

Follow these steps in order. Steps 1 and 2 are the whole point — skipping them is how you end up with code that is technically SOLID but wrong for the project.

### Step 1 — Inventory the codebase before deciding anything

Don't write a line until you understand the ground you're standing on. Read what's there:

- **Detect the language and framework** from the manifest and the layout:
  - PHP → `composer.json` (Laravel? Symfony? Slim? plain PHP?)
  - JS/TS → `package.json` (NestJS? Express? Next.js? React? Angular? Vue?)
  - C# / .NET → `*.csproj` / `*.sln` (ASP.NET Core? minimal API or MVC/Web API? confirm `<TargetFramework>` is `net9.0` or later)
  - Rust → `Cargo.toml` (Axum? Actix? Rocket? Tokio? plain?)
  - Go → `go.mod` (Gin? Echo? Fiber? standard library?)
- **Read the existing structure.** How are files and folders organized? Where does domain logic live versus controllers, services, models, interfaces? What are the naming conventions?
- **Note the conventions already in play.** Does the project already do one-class-per-file? How does it handle dependency injection? Does it define abstractions, and where? How is it tested?

By the end you should be able to state plainly: "here is how this project is organized, and here is where new code belongs." If there is no existing code (greenfield), say so, and establish a clean structure from the framework's conventions instead.

Then read the reference for the detected language — it has the concrete conventions and the framework-specific directory layout:

- PHP → `references/php.md`
- JavaScript/TypeScript → `references/typescript.md`
- C# / .NET → `references/csharp.md`
- Rust → `references/rust.md`
- Go → `references/go.md`

### Step 2 — When the existing style conflicts with the rules, ASK

This skill has strong opinions (one class/type per file, abstraction at every real boundary). Real projects don't always share them, and the two structural rules do not map cleanly onto every language (Rust and Go especially — see their references). When the existing code clearly conflicts with the rules — the project keeps several classes in one file, or wires concretes directly with no interfaces, or is written in a language where strict one-type-per-file cuts against the grain — do not silently override it and do not silently conform. Stop and put the choice to the user:

> "This project currently [describe it concretely, e.g. 'groups several classes per file under `src/models/`']. That conflicts with the strict SOLID structure (one class per file). Would you like me to follow the strict SOLID structure, or match the existing project style?"

Then follow their answer for the rest of the task. This keeps them in control and avoids imposing churn they never asked for. If the existing code already agrees with the rules, or there's simply no conflict, just proceed — don't manufacture a question.

The same workflow applies whether the user is asking you to write new code or to refactor existing code toward SOLID; for refactors, this conflict conversation is usually the first thing that comes up.

### Step 3 — Write the code, following SOLID and the framework

Now implement, honoring both the SOLID principles below and the framework conventions from the language reference. Each class/type lives in its own file, placed where the framework expects it.

## The SOLID principles (and why each one matters here)

Explaining the *why* matters more than the acronym — these principles are a means to code that's easy to change, not rules for their own sake.

- **S — Single Responsibility.** A class should have one reason to change. This is the principle underneath one-class-per-file: when each unit is small and focused, responsibilities stay separated and the code is easy to navigate and safe to change. If a class is doing two jobs, split it.
- **O — Open/Closed.** Code should be open to extension but closed to modification. Prefer adding a new implementation behind an existing abstraction over editing working code. Abstractions are the seams that make this possible.
- **L — Liskov Substitution.** Any implementation of an abstraction must be usable wherever that abstraction is expected, with no nasty surprises. If a subtype can't honor the contract, the hierarchy is wrong.
- **I — Interface Segregation.** Keep interfaces small and role-focused. A consumer shouldn't have to depend on methods it never calls. Several small interfaces beat one fat one.
- **D — Dependency Inversion.** High-level code should depend on abstractions, not concrete details. Inject dependencies through their interface/abstract type instead of constructing concretes inside. This is the other reason abstraction is first-class here.

## The two structural commitments

### One class/type per file

Keep each class (PHP, TS, C#) or primary type (Rust `struct`/`enum`, Go `struct`) in its own file, named after it. The payoff isn't ceremony — it's that a reader finds any unit instantly, diffs stay small and focused, and merge conflicts drop.

Important caveat for Rust and Go: they have no classes, and both idiomatically keep a primary type together with the small, tightly-coupled companions that only exist to serve it — a Rust type with its `impl` blocks, error enum, and builder; a Go type with its constructor and unexported helpers. Splitting those apart hurts readability. Follow the language's grain, not a mechanical rule; the language reference draws the line. When strict one-type-per-file would genuinely fight the language, that's a Step 2 conversation.

### Abstraction as a first-class citizen

Depend on abstractions, not concretions. Define an interface / abstract class / trait at each meaningful boundary — the seam between a consumer and a swappable implementation — and have callers depend on the seam. This is what makes code testable (swap a real dependency for a fake) and extensible (add an implementation without editing consumers).

Important caveat: abstraction is a tool, not a goal. An interface with a single implementation and no prospect of another is just indirection. Introduce abstractions at genuine boundaries — where you need to test across a seam, or support more than one implementation — not everywhere. This matters most in Go, whose culture treats speculative single-use interfaces as an anti-pattern and prefers defining an interface on the consumer side only when a real need appears. Each language reference says where the line falls for that language.

## Presenting the result

Produce each class/type as its own file with an **explicit file path** that follows the detected framework's directory conventions (the language reference lists the expected locations). If you're creating files directly, place them there; if you're showing code in chat, label every block with its full path so the structure is unambiguous. Then briefly explain the layout — especially where each abstraction boundary sits — so the user can see the SOLID reasoning, not just the code.
