# Go — SOLID conventions

Go has **no classes**. Data is `struct`; behavior is methods on it; `interface`s describe behavior. The unit of organization is the **package** (a directory), which spans multiple files. This changes how the two structural rules apply — read this before writing Go.

## "One type per file" — apply gently
Go idiom favors **package cohesion over per-file splitting**. It is completely normal to keep a `struct`, its constructor (`NewX`), and its unexported helpers in one file, and to place a few closely related small types together. Fragmenting a cohesive package into one-type-per-file files is un-idiomatic and usually hurts readability. So for Go, default to cohesive files grouped by responsibility, not a hard one-type-per-file rule. If the user genuinely wants strict one-type-per-file, surface that this cuts against Go's grain (Step 2) and let them decide.

## Abstraction = interfaces, defined at the consumer
This is the biggest Go-specific point:
- Define interfaces **where they're consumed**, not where types are implemented. "Accept interfaces, return structs."
- Keep them **small** — often a single method (`io.Reader`, `io.Writer`). Interface Segregation is the default, not an aspiration.
- **Do not over-abstract.** A single-implementation interface sitting next to its only concrete type is a recognized Go anti-pattern. Introduce an interface only when a consumer genuinely needs to vary the implementation, or to enable a test double. Concrete types are the norm; abstraction is the exception you reach for at a real seam.

Because of this, "abstraction as a first-class citizen" in Go means *thoughtful, consumer-side, minimal* interfaces — not an interface for every struct.

## Dependency injection
Explicit and boring, on purpose: pass dependencies into constructor functions (`NewService(repo Repository) *Service`). No magic container. If the project already uses `google/wire`, follow it; otherwise wire dependencies by hand in `main`.

## Framework layout
- **Gin / Echo / Fiber:** handlers or route groups per feature; services as structs; interfaces only at boundaries that need them (typically the repository seam). Common layout: `cmd/<app>/main.go`, `internal/` for private packages, domain packages like `internal/user`, `internal/order`.
- **Standard library (`net/http`):** same layout philosophy — `cmd/`, `internal/`, packages by domain.

## How SOLID shows up in Go
- **SRP** → small, focused packages and types; a type does one job.
- **DIP** → accept small consumer-side interfaces as parameters.
- **ISP** → tiny interfaces (frequently one method).
- **OCP / LSP** → add new implementations of an interface; every implementation must satisfy its full behavior.

## Detection hints
- Framework: inspect `go.mod` `require` — `github.com/gin-gonic/gin`, `github.com/labstack/echo`, `github.com/gofiber/fiber`.
- List packages and their files to gauge structure: `find . -name '*.go' -not -path './vendor/*' | sort`.
