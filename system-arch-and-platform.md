# SYSTEM ARCHITECTURE, PLATFORM AND VERIFICATION CONSTRAINTS (Dry Run)

## PLATFORM AND DEPLOYMENT MODEL (MUST)
- The system MUST be implemented as a single Google Cloud Function deployed as a
  single deployment.
- Ingestion and analysis MUST be exposed on the same public HTTPS endpoint.
- The API MUST be public and callable from browsers.
- The Google Cloud Functions framework MUST be used with source-based deployment.
- The entrypoint function MUST be named `Telemetry` and live in the root package.
- The root package MUST NOT be named `main`.
- The repository MUST NOT include a local runner (for example `cmd/main.go`).

## GO TOOLCHAIN AND MODULE FILES (MUST)
- The implementation MUST use Go version 1.25, declared in `go.mod`.
- The generated repository MUST include a `go.mod` containing exactly:
  - `module github.com/drawexact/telemetry`
  - a `go` directive
- The generated `go.mod` MUST NOT include:
  - `require` directives,
  - `replace` directives,
  - `exclude` directives.
- The generated repository MUST NOT include a `go.sum` file.

### Dependency resolution model (MUST)
- Third-party dependencies MAY be used.
- Import statements are the sole source of truth for required modules.
- Dependency resolution and `go.sum` generation are explicit post-generation
  steps performed by the repository consumer.

### Third-party dependency principles (MUST)
- Dependencies SHOULD be preferred when they provide a well-known, widely used
  solution.
- Each dependency MUST have a single, clear responsibility.
- Overlapping or speculative dependencies MUST NOT be included.
- Each dependency MUST be justified in `ASSUMPTIONS_LOG.txt`.

### Generation-time constraints (MUST)
- The generator MAY run Go tooling when it improves correctness.
- The generator MUST NOT resolve or pin module versions during generation.
- The generator MUST NOT write versioned `require` entries or generate `go.sum`.

## CONFIGURATION POLICY (LOCKED)
- The deployed function URL is:
  https://service-injest-event-65030510907.europe-west2.run.app
- The bucket name is `drawexact-telemetry`.
- The system MUST NOT depend on runtime environment variables.

## VERIFICATION AND TESTABILITY CONSTRAINTS (MUST)

These requirements define properties the system MUST have in order to be
verifiable via its public interface.

They do not define general test design practices, which are governed by
`testing.md`.

### Testing strategy
- Tests MUST exercise the public HTTP entrypoint.
- Tests MUST cover:
  - CORS preflight,
  - ingestion success and failure,
  - analysis success and failure.
- Tests MUST use deterministic fakes.
- Tests MUST NOT make live network calls.
- Tests MUST be runnable via `go test ./...`.

### Testability contract
- External systems MUST be accessed only via minimal boundary interfaces.
- Deterministic core logic MUST depend only on those interfaces.
