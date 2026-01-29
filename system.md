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
- A `go.mod` file MUST be generated including the go version and the module name only. It MUST not
  contain anything else.
- External requirements for `go.mod` MUST NOT override this rule unless explicitly
  stated in a future revision.
- A `go.sum` file MUST NOT be generated or committed.
- External requirements for `go.sum` MUST NOT override this rule unless explicitly
  stated in a future revision.

## MUST: Configuration policy
- The deployed function URL is
  https://service-injest-event-65030510907.europe-west2.run.app
- The system MUST NOT use environment variables for non-secret configuration.
- All non-secret configuration values MUST be compile-time constants.
- Changing configuration MUST require redeployment.

## MUST: Telemetry event model
The `EventPayload` schema is defined as follows:

- `SchemaVersion`: integer, MUST equal 1.
- `EventULID`: string, MUST be a valid ULID parsed strictly.
- `ProxyUserID`: string, MUST be a UUID version 4.
- `TimeUTC`: string, MUST parse as an RFC3339 timestamp in UTC.
- `Visit`: integer, MUST be between 1 and 100000 inclusive.
- `Event`: string, length MUST be between 4 and 40 characters and MUST be exactly
  one of: `launched`, `quit`, `sign-in-started`, `sign-in-success`,
  `recoverable-javascript-error`, `fatal-javascript-error`, `loaded-example`,
  `created-new-drawing`, `retreived-save-drawing`.
- `Parameters`: string, opaque, maximum length 80 characters, with no structural
  guarantees.

## MUST: Payload validation policy
- All ingested payloads MUST satisfy the telemetry event model constraints.
- Payload validation MUST enforce reasonable and plausible input.
- The system MUST NOT be considered adversarially robust.

## MUST: HTTP interface
- Permissive CORS MUST be supported with `Access-Control-Allow-Origin: *`.
- OPTIONS preflight requests MUST be handled correctly.

POST `/`:
- Accepts `application/json` containing an `EventPayload`.
- On success, MUST return HTTP 204 with no body.
- Invalid payloads MUST return HTTP 400.
- Unsupported content types MUST return HTTP 415.
- Storage failures MUST return HTTP 500.

GET `/`:
- Performs live aggregate analysis with no parameters.
- On success, MUST return HTTP 200 with a JSON body matching the analysis schema.
- Failures MUST return HTTP 500.

## MUST: Persistence strategy
- Telemetry events MUST be persisted using Google Cloud Storage.
- The storage bucket MUST be named `drawexact-telemetry`.
- Each event MUST be stored as a separate object.
- Objects SHOULD be treated as immutable once created (do not mutate existing objects), but this immutability is an implementation choice and not required to satisfy idempotency.
- Ingestion MUST be idempotent with respect to `EventULID`: re-sending the same event MUST NOT create a second stored record or change analysis outcomes.
- The storage layer SHOULD treat writes for an existing `EventULID` as a no-op (success) where practical.
- Object contents MUST be gzip-compressed.
- When decompressed, each object MUST contain exactly one NDJSON line encoding a
  single `EventPayload`, followed by a newline.

## LOCKED: Storage object naming
Objects MUST be named:

`events/y=YYYY/m=MM/d=DD/hour=HH/EventULID.ndjson.gz`

`EventULID` acts as the event identity key; this naming makes re-ingestion collisions explicit and supports idempotency.

- Date and hour MUST be derived from `EventPayload.TimeUTC`.
- `EventULID` MUST be taken directly from `EventPayload.EventULID`.
- The `events/` prefix is constant.

## MUST: Analysis interface and schema
- The system MUST expose exactly one analysis operation via HTTP GET at `/`.
- Analysis MUST be computed live on each request.
- No parameters MUST be accepted and no caching MUST be performed.

The response MUST contain:
- Object `HowManyPeopleHave` with integer fields: `Launched`,
  `LoadedAnExample`, `TriedToSignIn`, `SucceededSigningIn`,
  `CreatedTheirOwnDrawing`, `RetreivedTheirASavedDrawing`.
- Integer fields `TotalRecoverableErrors` and `TotalFatalErrors`.

## MUST: Analysis semantics
- A distinct person is defined as a distinct `ProxyUserID`.
- Analysis MUST cover the entire stored event history with no time windowing.
- Behaviour metrics MUST count distinct `ProxyUserID` values with at least one
  corresponding event.
- Error totals MUST count total events, not distinct users.
- Malformed stored events MUST be skipped.
- Analysis MUST NOT fail solely due to malformed historical data.

## ASSUME: Analysis memory constraints
It is assumed the full set of stored telemetry objects fits in memory during
analysis.

## MUST: Security and abuse model
- The API is public.
- Validation for reasonableness and plausibility MUST be the only abuse mitigation.
- The system MUST NOT attempt to mitigate denial-of-service attacks beyond relying
  on Google Cloud infrastructure.

## ASSUME: Operational characteristics
- A typical user session produces approximately 20 events.
- Sessions last roughly two hours and occur once per day.
- Initial user population is on the order of 5000 users.
- Occasional dropped events are acceptable.
- Latency is not critical.
- Cost minimisation is the primary operational driver.

## MUST: Logging policy
- All errors MUST be logged to STDOUT.
- No other logging MUST be performed.

## SHOULD: Code organisation and separation of concerns
- Generated code SHOULD separate concerns to maximise readability and ease
  end-to-end testing.
- Separation SHOULD be reflected in package structure, file structure, and
  abstractions.
- Simple designs are preferred over clever ones.
- Unit testing of internal components MAY be performed when fixturing is simpler,
  but correctness evidence SHOULD primarily come from public interface tests.

## MUST: Testability and structural contract
- All interactions with external systems MUST be isolated behind minimal capability
  interfaces.
- Interfaces MUST describe required capabilities rather than concrete technologies.
- Deterministic core logic MUST depend only on these interfaces.
