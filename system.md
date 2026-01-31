# SYSTEM BEHAVIOUR AND PLATFORM SPECIFICATION (Canonical)

## MUST: System overview
The system is a cloud-hosted HTTP API that receives runtime telemetry events from the
DrawExact.click web application, persists them at low cost, and exposes a live analysis
endpoint that computes aggregate user behaviour metrics derived from the stored
events.

## MUST: Platform and deployment model
- The system MUST be implemented as a single Google Cloud Function deployed as a
  single deployment.
- Ingestion and analysis MUST be exposed on the same public HTTPS endpoint.
- The API MUST be public and callable from browsers.
- The Google Cloud Functions framework MUST be used with source-based deployment.
- The entry point function MUST be named `Telemetry` and live in the root package.
- The root package MUST NOT be named `main`.
- The repository MUST NOT include a local runner such as `cmd/main.go`.

## MUST: Go toolchain and module files
- The implementation MUST use Go version 1.25, declared in `go.mod`.
- The generated repository output MUST include a `go.mod` containing exactly:
  - the `module` directive, and
  - the `go` directive.
- The generated `go.mod` MUST NOT include any dependency metadata:
  - it MUST NOT contain any `require` directives (direct or indirect),
  - it MUST NOT contain any `replace` directives, and
  - it MUST NOT contain any `exclude` directives.
- The generated repository output MUST NOT include a `go.sum` file.

### Dependency resolution model
- Third-party dependencies MAY be used.
- Import statements in the generated `.go` source files are the single source of truth for which third-party modules are required.
- Dependency graph resolution (including selecting versions, computing transitive requirements, and generating `go.sum`) is an explicit post-generation step performed by the repo consumer, for example by running:
  - `go get .` (or `go mod tidy`)
  - then `go test ./...`

### Third-party dependency principles
- The implementation SHOULD prefer delegating responsibilities to third-party modules when those modules already provide a well-known solution and are widely used (as a practical proxy for correctness, maintenance, and ecosystem scrutiny).
- This preference is especially relevant for input validation / schema enforcement, Google Cloud service integration (official SDKs), and identifier parsing/validation (ULID, UUID).
- Third-party dependencies MUST remain purposeful and minimal:
  - Each dependency MUST have a clear, named responsibility.
  - The implementation MUST NOT include overlapping dependencies that solve the same responsibility.
  - The implementation MUST NOT add a dependency “just in case”: every third-party module used MUST correspond to an import in the codebase.
- Each third-party dependency MUST be justified in `ASSUMPTIONS_LOG.txt` (one short paragraph): what responsibility it covers, why a library is preferred over bespoke code here, and what the standard-library fallback would have been.

### Generation-time constraints
- The generator MAY run Go tooling as part of generation when doing so improves correctness of the generated code (for example running `go test ./...` in an environment where that is possible).
- The generator MUST NOT attempt to resolve or pin module versions during generation, and therefore MUST NOT write versioned `require` entries into `go.mod` and MUST NOT generate `go.sum`.


## MUST: Configuration policy
- The deployed function URL is
  https://service-injest-event-65030510907.europe-west2.run.app
- The bucket name is `drawexact-telemetry`.
- The system MUST NOT depend on runtime environment variables.

## MUST: HTTP interface and responses
- The function MUST serve on the root path `/`.
- The function MUST accept:
  - `POST /` for ingestion
  - `GET /` for analysis
  - `OPTIONS /` for CORS preflight
- For `POST /`:
  - The request MUST have `Content-Type: application/json`.
  - The request body MUST be a single JSON object matching the Event Payload schema.
  - On success, the function MUST respond with status code **204 No Content** and an
    empty body.
  - If the payload is invalid, the function MUST respond with status code **400 Bad
    Request**.
  - If the `Content-Type` is not supported, the function MUST respond with status code
    **415 Unsupported Media Type**.
  - If persistence fails, the function MUST respond with status code
    **500 Internal Server Error**.
- For `GET /`:
  - On success, the function MUST respond with status code **200 OK** and JSON body
    matching the Analysis Response schema.
  - If analysis fails, the function MUST respond with status code
    **500 Internal Server Error**.

## MUST: logging
- The system MUST output a record of all errors that occur at runtime to STDOUT.
- No other logging to STDOUT should be done.

## MUST: CORS
- The system MUST support browser use from any origin.
- The function MUST set:
  - `Access-Control-Allow-Origin: *`
  - `Access-Control-Allow-Methods: GET, POST, OPTIONS`
  - `Access-Control-Allow-Headers: Content-Type`
- For `OPTIONS /`, the function MUST return **204 No Content**.

## MUST: Event payload schema
The event payload MUST be a JSON object with the following fields:

- `SchemaVersion` (number):
  - MUST be exactly `1`.
- `EventULID` (string):
  - MUST be a valid ULID (26 chars, Crockford base32).
- `ProxyUserID` (string):
  - MUST be a valid UUIDv4 string.
- `TimeUTC` (string):
  - MUST be RFC3339.
  - MUST be in UTC (timezone `Z`).
- `Visit` (number):
  - MUST be integer in range 1–100000.
- `Event` (string):
  - MUST be one of:
    - launched
    - quit
    - sign-in-started
    - sign-in-success
    - recoverable-javascript-error
    - fatal-javascript-error
    - loaded-example
    - created-new-drawing
    - retreived-save-drawing
  - MUST have length 4–40.
- `Parameters` (string):
  - MUST have length <= 80.

## MUST: Validation and plausibility rules
- The payload MUST be validated strictly:
  - Unknown fields MUST cause a 400.
  - Incorrect types MUST cause a 400.
  - Any field violating constraints MUST cause a 400.

## MUST: Persistence model
- Each accepted event MUST be persisted in Google Cloud Storage, as a distinct object.
- The object MUST be stored under the prefix `events/` using this structure:

  `events/y=YYYY/m=MM/d=DD/hour=HH/{EventULID}.ndjson.gz`

  where YYYY, MM, DD, HH are derived from the event's `TimeUTC` in UTC.

- The stored object content MUST be gzip-compressed NDJSON with exactly one line:
  - a single JSON object corresponding to the event payload
  - terminated with a newline (`\n`)

- Object creation MUST be idempotent:
  - If an object with the same name already exists, the ingestion MUST still return
    **204 No Content**.

## MUST: Analysis semantics
- `GET /` MUST compute live metrics by reading all stored events in the bucket under
  `events/`.
- The function MUST skip malformed stored events and continue.
- The Analysis Response JSON MUST have this shape:

```json
{
  "HowManyPeopleHave": {
    "Launched": 0,
    "LoadedAnExample": 0,
    "TriedToSignIn": 0,
    "SucceededSigningIn": 0,
    "CreatedTheirOwnDrawing": 0,
    "RetreivedTheirASavedDrawing": 0
  },
  "TotalRecoverableErrors": 0,
  "TotalFatalErrors": 0
}
```

- The fields MUST be computed as:
  - `HowManyPeopleHave.Launched`: count of distinct `ProxyUserID` that have at least one `launched` event.
  - `HowManyPeopleHave.LoadedAnExample`: distinct `ProxyUserID` with at least one `loaded-example` event.
  - `HowManyPeopleHave.TriedToSignIn`: distinct `ProxyUserID` with at least one `sign-in-started` event.
  - `HowManyPeopleHave.SucceededSigningIn`: distinct `ProxyUserID` with at least one `sign-in-success` event.
  - `HowManyPeopleHave.CreatedTheirOwnDrawing`: distinct `ProxyUserID` with at least one `created-new-drawing` event.
  - `HowManyPeopleHave.RetreivedTheirASavedDrawing`: distinct `ProxyUserID` with at least one `retreived-save-drawing` event.
  - `TotalRecoverableErrors`: total count of events with `Event == recoverable-javascript-error`.
  - `TotalFatalErrors`: total count of events with `Event == fatal-javascript-error`.

## MUST: Testing strategy
- The system MUST include public-interface tests that exercise the HTTP entrypoint:
  - CORS preflight behaviour
  - Ingestion success (204)
  - Ingestion failure (400/415/500)
  - Analysis success (200)
  - Analysis failure (500)
- Tests MUST use deterministic fakes for external capabilities.
- Tests MUST NOT make live network calls.
- Tests MUST be executable via `go test ./...`.

## SHOULD: Simplicity
Simple designs are preferred over clever ones.
- Unit testing of internal components MAY be performed when fixturing is simpler,
  but correctness evidence SHOULD primarily come from public interface tests.

## MUST: Testability and structural contract
- All interactions with external systems MUST be isolated behind minimal capability
  interfaces.
- Interfaces MUST describe required capabilities rather than concrete technologies.
- Deterministic core logic MUST depend only on these interfaces.
