# ERROR PHENOMENA & COVERAGE DOCTRINE 

## PURPOSE (MUST)
- Define a reusable, interface-level pattern for specifying machine-checkable
  failure phenomena via `error_id`.
- Govern generated test coverage of those failure phenomena.
- Enable internal branching behaviour to be externally stimulable and verifiable
  through public-interface tests.

## SCOPE (MUST)
This doctrine applies to all public interfaces defined by this specification pack,
including (but not limited to):
- HTTP endpoints
- CLI commands
- RPC methods

It governs:
- Naming of failure phenomena (`error_id`)
- External stimulation of those phenomena
- Deterministic mapping from stimuli to `error_id`
- Minimum generated test coverage

This doctrine is implementation-agnostic and does not prescribe internal module
structure, function names, or validation libraries.

## DEFINITIONS (MUST)

### Public system interface
The public system interface is defined by `public-interface.md`. All references
to “public interface” in this document refer exclusively to that definition.

### Failure phenomenon
A specific, externally observable failure outcome that can occur when handling a
public-interface request. Each phenomenon is identified by exactly one
`error_id`.

### `error_id`
- `error_id` tokens are part of the external contract.
- Clients and tests MUST rely on `error_id` rather than human-readable messages.
- The complete set of valid `error_id` values is defined across this
  specification pack; this document does not enumerate them.

### Witness condition
A specification-defined condition, expressed purely in terms of the public
interface, that is sufficient to trigger a specific failure phenomenon.

### Interface-boundary dependency injection (DI)
A testing capability in which dependencies at the public interface boundary may
be replaced with controlled substitutes to stimulate specific failure phenomena.

## ERROR IDENTIFICATION RULES (MUST)

### Presence and type
1. Any public-interface response representing a client error (4xx) caused by
   request validation or request plausibility MUST include a response-body field
   named `error_id`.
2. `error_id` MUST be a string.
3. `error_id` MUST NOT be numeric.

### Identity and uniqueness
1. Each distinct failure phenomenon MUST have exactly one `error_id`.
2. Each `error_id` MUST correspond to exactly one failure phenomenon.

## STIMULATION AND SELECTION (MUST)

### Stimulatability
For every `error_id`, the corresponding failure phenomenon MUST be reproducibly
stimulable via:
- request input, and/or
- permitted interface-boundary DI.

### Deterministic selection
- If multiple failure phenomena apply, selection MUST be deterministic.
- When an error occurs, the system MUST emit exactly one error.
- The emitted error MUST be the first failure encountered at runtime.

## WITNESS CONDITIONS (MUST)
For each `error_id`, the specification MUST define at least one witness condition.

## TEST COVERAGE (MUST)

### Coverage completeness
Generated tests MUST include at least one public-interface test per `error_id`.

Coverage requirements are defined in `testing.md` and MUST satisfy this doctrine.

## GENERATOR OBLIGATIONS (MUST)
The generator MUST discover every declared `error_id` in this specification pack
and generate corresponding tests.
