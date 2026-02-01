# SYSTEr BEHAVIOUR AND PUBLIC CONTRACT (Canonical)

## PURPOSE (MUST)
This document defines the externally observable behaviour and public contract of
the system.

It specifies what the system does, how it is invoked, and the semantics that MUST
be preserved by any compliant implementation.

## SYSTEM OVERVIEW (MUST)
The system is a cloud-hosted HTTP API that:
- receives runtime telemetry events from the DrawExact.click web application,
- persists those events at low cost, and
- exposes a live analysis endpoint that computes aggregate user-behaviour metrics
  derived from the stored events.

## HTTP INTERFACE (MUST)

- The system MUST serve HTTP requests on path `/`.

- The system MUST support:
  - `POST /` for ingestion,
  - `GET /` for analysis,
  - `OPTIONS /` for CORS preflight.

- For any other method it MUST return MethodNotAllowed-405. Error-ID: `http.method.disallowed`.


## POST / INGESTION SEMANTICS (MUST)

### Content-Type handling (MUST)

- Requests MUST include `Content-Type: application/json`.
	- If content type is missing: it MUST return UnsupportedMediaType-415. Error-ID:
	  `request.content-type.unsupported`.


### Request body (MUST)
- The request body MUST be a single JSON object
	- If zero or multiple: it MUST return Bad request-400. Error-ID: `payload.objects.not-one`.

- The body MUST conform to the Event Payload Schema.
	- If it does not: Bad request-400, with Error-ID as specified in EventPayloadSchema.

### Responses (MUST)
- On success, the system MUST respond with `204 No Content` and an empty body.
- Invalid payloads MUST yield `400 Bad Request`.
- Persistence failures MUST yield `500 Internal Server Error`. Error-ID:
  `event.storage-write.failed`.

## GET / ANALYSIS SEMANTICS (MUST)
- On success, the system MUST respond with `200 OK` and a JSON body matching the
  Analysis Response Schema.
- Analysis failures MUST yield `500 Internal Server Error`.  Error-ID:
  `get.analysis.failed`.

## LOGGING POLICY (MUST)
- All runtime errors MUST be written to STDOUT.
- No other logging to STDOUT is permitted.

## CORS POLICY (MUST)
- The system MUST support browser use from any origin.
- Responses MUST include:
  - `Access-Control-Allow-Origin: *`
  - `Access-Control-Allow-Methods: GET, POST, OPTIONS`
  - `Access-Control-Allow-Headers: Content-Type`
- `OPTIONS /` MUST return `204 No Content`.

## EVENT PAYLOAD SCHEMA (MUST)
The ingestion payload MUST be a JSON object with the following fields:

- `SchemaVersion` (number): MUST equal `1`.
	- For this validation failure use ErrorID: event.schema-version.unsupported
- `EventULID` (string): MUST be a valid ULID.
	- For this validation failure use ErrorID: event.schema-event-ulid.malformed
- `ProxyUserID` (string): MUST be a valid UUIDv4.
	- For this validation failure use ErrorID: event.schema-event-proxyuserid.malformed
- `TimeUTC` (string):
  - MUST be RFC3339,
	- For this validation failure use ErrorID: event.schema-event-timeutc.notrfc3339
  - MUST be UTC with `Z` timezone.
	- For this validation failure use ErrorID: event.schema-event-timeutc.notZoneZ
- `Visit` (number): MUST be an integer in the range 1–100000.
	- For this validation failure use ErrorID: event.schema-event-visit.outofbounds
- `Event` (string):
	- If not a string, uuse ErrorID: event.schema-event-event.notstring
  - MUST be one of the defined event names,
	- For this validation failure use ErrorID: event.schema-event-event.notrecognised
  - MUST have length 4–40.
	- For this validation failure use ErrorID: event.schema-event-event.illegallength
- `Parameters` (string): MUST have length ≤ 80.
	- For this validation failure use ErrorID: event.schema-event-parameters.illegallength

## VALIDATION RULES (MUST)
- Unknown fields MUST cause `400 Bad Request`.
- Missing fields MUST cause `400 Bad Request`.
- Type violations MUST cause `400 Bad Request`.
- Constraint violations MUST cause `400 Bad Request`.
- Validation failures MUST include an `error_id` and follow `errors.md`
  semantics.

## PERSISTENCE MODEL (MUST)
- Each accepted event MUST be stored as a distinct object in Google Cloud Storage.
- Objects MUST be stored under:
  `events/y=YYYY/m=MM/d=DD/hour=HH/{EventULID}.ndjson.gz`
- Content MUST be gzip-compressed NDJSON with exactly one line and a trailing
  newline.
- Object creation MUST be idempotent; existing objects MUST still yield `204 No
  Content`.

## ANALYSIS RESPONSE SCHEMA (MUST)
The analysis response JSON MUST match the following shape:

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

Metrics MUST be computed as counts of distinct `ProxyUserID` or total events as
defined in the original specification.

Malformed stored events MUST be skipped.
