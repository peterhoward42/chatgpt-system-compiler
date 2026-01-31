# Error Phenomena & Coverage Doctrine

## Purpose
This document defines a reusable, interface-level pattern for specifying machine-checkable failures (`error_id`) and for governing generated test coverage of those failures. The goal is to make internal branching behaviour externally stimulable and verifiable via public-interface tests.

## Scope
This doctrine applies to any public interface defined by this spec pack (e.g. HTTP endpoints, CLI commands, RPC methods). It governs:

- How failure phenomena are named (`error_id`)
- How those phenomena are stimulated (external request and/or permitted interface-boundary dependency injection)
- Deterministic mapping of stimuli to `error_id`
- Minimum test coverage requirements for generated tests

This doctrine is implementation-agnostic and does not prescribe internal module structure, function names, or validation libraries.

---

## Definitions

### Public interface
The externally visible entrypoints to the system (e.g. HTTP routes). “Public interface tests” are tests that exercise these entrypoints.

### Failure phenomenon
A specific, externally observable failure outcome that can occur when handling a public-interface request. A phenomenon is identified by an `error_id`.

### `error_id`
A stable, machine-readable string identifier returned in error responses. It identifies the *failure phenomenon*, not an HTTP status code and not a human-readable message.

### Witness condition
A spec-defined condition, expressed purely in terms of the public interface, that is sufficient to trigger a specific failure phenomenon.

### Interface-boundary dependency injection (DI)
A testing capability in which dependencies at the public interface boundary may be replaced with controlled substitutes to stimulate specific phenomena.

---

## Normative rules

### MUST: `error_id` presence and type
1. Any public-interface response representing a client error (4xx) due to request validation or request plausibility MUST include a response-body field named `error_id`.
2. `error_id` MUST be a string.
3. `error_id` MUST be stable across releases.
4. `error_id` MUST NOT be numeric.

### MUST: Phenomenon identity and uniqueness
1. Each distinct failure phenomenon MUST have exactly one `error_id`.
2. Each `error_id` MUST correspond to exactly one phenomenon.

### MUST: Stimulatability
For every `error_id`, the phenomenon MUST be reproducibly stimulable via request input or permitted interface-boundary DI.

### MUST: Deterministic selection
If multiple failure phenomena apply, selection MUST be deterministic.

### MUST: Witness conditions
For each `error_id`, the spec MUST define at least one witness condition.

### MUST: Test coverage completeness
Generated tests MUST include at least one public-interface test per `error_id`.

---

## Generator obligations

The generator MUST discover every declared `error_id` and generate corresponding tests.

---
