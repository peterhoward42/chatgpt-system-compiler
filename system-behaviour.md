# SYSTEM BEHAVIOUR AND PUBLIC CONTRACT

## PURPOSE (MUST)
This document defines the externally observable behaviour and public
contract of the system.

It specifies what the system does, how it is invoked, and the semantics
that MUST be preserved by any compliant implementation.

## SYSTEM OVERVIEW (MUST)
The system is a cloud-hosted HTTP API that:

- receives runtime telemetry events from the DrawExact.click web
  application,
- persists those events at low cost, and
- exposes a live analysis endpoint that computes aggregate
  user-behaviour metrics derived from the stored events.

## HTTP INTERFACE (MUST)

### General
- The system MUST serve HTTP requests on path `/`.

- The system MUST support exactly the following methods:
  - `POST /` for ingestion,
  - `GET /` for analysis,
  - `OPTIONS /` for CORS preflight.

- Any other HTTP method MUST return:
  - status: `405 Method Not Allowed`
  - error_id: `http.method.disallowed`.

## POST / — INGESTION SEMANTICS (MUST)

### Content-Type Handling (MUST)
- Requests MUST include header `Content-Type: application/json`.
- If the content type is missing or not supported, the system MUST
  return:
  - status: `415 Unsupported Media Type`
  - error_id: `request.content-type.unsupported`.

### Request Body Shape (MUST)
- The request body MUST contain exactly one JSON object.
- If zero objects or multiple objects are supplied, the system MUST
  return:
  - status: `400 Bad Request`
  - error_id: `payload.objects.not-one`.

### Request Body Validation (MUST)
- The JSON object MUST conform to the Event Payload Schema.
- If schema validation fails, the system MUST return:
  - status: `400 Bad Request`
  - error_id: as specified by the violated schema rule.

### Success and Failure Responses (MUST)
- On successful ingestion, the system MUST return:
  - status: `204 No Content`
  - empty response body.

- Persistence failures MUST return:
  - status: `500 Internal Server Error`
  - error_id: `event.storage-write.failed`.

## GET / — ANALYSIS SEMANTICS (MUST)
- On success, the system MUST return:
  - status: `200 OK`
  - a JSON body conforming to the Analysis Response Schema.

- Analysis failures MUST return:
  - status: `500 Internal Server Error`
  - error_id: `get.analysis.failed`.

## CORS POLICY (MUST)
- The system MUST support browser use from any origin.

- All responses MUST include the following headers:
  - `Access-Control-Allow-Origin: *`
  - `Access-Control-Allow-Methods: GET, POST, OPTIONS`
  - `Access-Control-Allow-Headers: Content-Type`

- `OPTIONS /` MUST return:
  - status: `204 No Content`.

## LOGGING POLICY (MUST)
- All runtime errors MUST be written to STDOUT.
- No other logging to STDOUT is permitted.

## EVENT PAYLOAD SCHEMA (MUST)
The ingestion payload MUST be a JSON object with exactly the following
fields and no others.



### Fields

- `SchemaVersion` (number)
  - MUST equal `1`.
  - On violation, error_id:
    `event.schema-version.unsupported`.

- `EventULID` (string)
  - MUST be a valid ULID.
  - On violation, error_id:
    `event.schema-event-ulid.malformed`.

- `ProxyUserID` (string)
  - MUST be a valid UUIDv4.
  - On violation, error_id:
    `event.schema-event-proxyuserid.malformed`.

- `TimeUTC` (string)
  - MUST be RFC3339.
    - On violation, error_id:
      `event.schema-event-timeutc.notrfc3339`.
  - MUST be UTC with `Z` timezone.
    - On violation, error_id:
      `event.schema-event-timeutc.notZoneZ`.

- `Visit` (number)
  - MUST be an integer in the range 1–100000.
  - On violation, error_id:
    `event.schema-event-visit.outofbounds`.

- `Event` (string)
  - MUST be a string.
    - On violation, error_id:
      `event.schema-event-event.notstring`.
  - MUST have length 4–40.
    - On violation, error_id:
      `event.schema-event-event.illegallength`.
  - MUST be one of the names defined in Allowed Event Names.
    - On violation, error_id:
      `event.schema-event-eventname.notrecognised`.

- `Parameters` (string)
  - MUST have length ≤ 80.
  - On violation, error_id:
    `event.schema-event-parameters.illegallength`.

  
### Allowed Event Names (MUST)

Event-Names MUST be one of:

launched
quit
sign-in-started
sign-in-success
recoverable-javascript-error
fatal-javascript-error
loaded-example
created-new-drawing
retreived-save-drawing

## VALIDATION RULES (MUST)
- Unknown fields MUST cause `400 Bad Request`.
- Missing fields MUST cause `400 Bad Request`.
- Type violations MUST cause `400 Bad Request`.
- Constraint violations MUST cause `400 Bad Request`.
- All validation failures MUST include an `error_id` and MUST follow
  `errors.md` semantics.

## PERSISTENCE MODEL (MUST)
- Each accepted event MUST be stored as a distinct object in Google
  Cloud Storage.

- Objects MUST be stored under the path:
  ```
  events/y=YYYY/m=MM/d=DD/hour=HH/{EventULID}.ndjson.gz
  ```

- Stored content MUST be:
  - gzip-compressed,
  - NDJSON,
  - exactly one line,
  - terminated by a trailing newline.

- Object creation MUST be idempotent.
  - If an object already exists, the system MUST still return
    `204 No Content`.

## ANALYSIS RESPONSE SCHEMA (MUST)
The analysis response MUST be a JSON object with the following shape:

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

## RESPONSE CALCULATION (MUST)

The response json's fields MUST be computed as:
- HowManyPeopleHave.Launched: count of distinct ProxyUserID that have at least one launched event.
- HowManyPeopleHave.LoadedAnExample: distinct ProxyUserID with at least one loaded-example event.
- HowManyPeopleHave.TriedToSignIn: distinct ProxyUserID with at least one sign-in-started event.
- HowManyPeopleHave.SucceededSigningIn: distinct ProxyUserID with at least one sign-in-success event.
- HowManyPeopleHave.CreatedTheirOwnDrawing: distinct ProxyUserID with at least one created-new-drawing event.
- HowManyPeopleHave.RetreivedTheirASavedDrawing: distinct ProxyUserID with at least one retreived-save-drawing event.
- TotalRecoverableErrors: total count of events with Event == recoverable-javascript-error.
- TotalFatalErrors: total count of events with Event == fatal-javascript-error.


- Malformed stored events MUST be skipped.
