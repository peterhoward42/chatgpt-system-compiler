# SYSTEM BEHAVIOUR AND PUBLIC CONTRACT (Canonical)

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

Any other HTTP method MUST return:
- `405 Method Not Allowed`,
- appropriate CORS headers,
- JSON body `{ "error_id": "http.method.notallowed" }`.

## POST / INGESTION SEMANTICS (MUST)

### Content-Type handling (MUST)
- Requests MUST include `Content-Type: application/json`.
- Media type matching MUST ignore parameters (for example `charset`).
- Missing Content-Type MUST yield `415`
  (`error_id: http.contenttype.forbidden`).
- Unsupported media types MUST yield `415`
  (`error_id: header.contenttype.notsupported`).

### Request body (MUST)
- The request body MUST be a single JSON object
  (`error_id: json.objectcount.multiple`).
- The body MUST conform to the Event Payload Schema.

### Responses (MUST)
- On success, the system MUST respond with `204 No Content` and an empty body.
- Invalid payloads MUST yield `400 Bad Request`.
- Persistence failures MUST yield `500 Internal Server Error`.

## GET / ANALYSIS SEMANTICS (MUST)
- On success, the system MUST respond with `200 OK` and a JSON body matching the
  Analysis Response Schema.
- Analysis failures MUST yield `500 Internal Server Error`.

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
- `EventULID` (string): MUST be a valid ULID.
- `ProxyUserID` (string): MUST be a valid UUIDv4.
- `TimeUTC` (string):
  - MUST be RFC3339,
  - MUST be UTC with `Z` timezone.
- `Visit` (number): MUST be an integer in the range 1–100000.
- `Event` (string):
  - MUST be one of the defined event names,
  - MUST have length 4–40.
- `Parameters` (string): MUST have length ≤ 80.

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
