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
- The entry point function MUST be named `InjestEvent` and live in the root package.
- The root package MUST NOT be named `main`.
- The repository MUST NOT include a local runner such as `cmd/main.go`.

## MUST: Go toolchain and module files
- The implementation MUST use Go version 1.25, declared in `go.mod`.
- The generated repository output MUST include a minimal `go.mod` containing only:
  - the `module` directive, and
  - the `go` directive.
- The implementation MAY import third-party modules (for example, Google Cloud Storage client libraries or data-shape validation packages) where this reduces complexity or improves correctness.
- The generated repository output MUST NOT include dependency requirements in `go.mod` (i.e. it MUST NOT contain any `require`, `replace`, or `exclude` directives).
- The generated repository output MUST NOT include a `go.sum` file.
- The generator MUST NOT attempt to build, compile, resolve modules, or run Go tooling (such as `go get` or `go mod tidy`) as part of generation; dependency resolution is an external, user-run step.
- When using third-party modules, the generator MUST avoid hardcoding specific version numbers anywhere in the generated repository output.

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
    - fatal-
