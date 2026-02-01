# PUBLIC INTERFACE (Canonical)

## PURPOSE (MUST)
This document defines the system’s public interface.

It exists to provide a single, authoritative definition of what constitutes the
“public interface” or “public system interface” as referenced by other
specifications in this pack.

## SCOPE (MUST)
The definitions in this document apply across the entire specification pack.

Any requirement in any specification that is stated in terms of:
- “the public interface”, or
- “the public system interface”

MUST be interpreted exclusively according to this document.

## AUTHORITY AND EXCLUSIVITY (LOCKED)
- This document is the sole authority for defining the public interface.
- No other document in this specification pack MAY claim to define the public
  interface.
- Any conflicting definition elsewhere MUST be treated as invalid.

## COMPOSITION OF THE PUBLIC INTERFACE (MUST)

The public interface consists of the following classes of externally observable
definitions.

### HTTP interface definitions
All HTTP methods defined in `system.md`, including:
- endpoint URLs,
- HTTP methods,
- supported content types,
- CORS requirements,
- POST request payloads,
- POST request payload schemas,
- POST payload schema validation requirements,
- GET response schemas,
- supported query parameters (if any),
- required HTTP status responses.

This list defines the *classes* of definitions that enter the public interface.
It is not itself the enumeration authority for specific endpoints or schemas.

### Error handling
Every specification in this pack that defines an error condition with an
`error_id` defines part of the public interface.

All such `error_id` values are externally visible contract elements.
