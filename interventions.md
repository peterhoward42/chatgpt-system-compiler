# Intervention Specification

## Purpose

This specification defines a small, explicit set of **interventions** that constrain system generation.

Its purpose is to **lock in or lock out specific patterns of behaviour or choice** that have been repeatedly observed during full system regeneration and have been judged, by the human collaborator, to be either reliably desirable or reliably harmful.

This document exists to prevent the generator from re-exploring parts of the design space whose outcomes are already known to be unacceptable, unstable, or unnecessarily fragile, while preserving as much freedom as possible elsewhere.

---

## Rationale

The primary specifications in this pack are intentionally written at a high level of abstraction. They are designed to give the generator wide latitude to synthesise a coherent system and to remain robust under substantive changes of intent across iterations.

However, repeated regeneration of complete systems exposes **recurring emergent patterns** that are not explicitly specified but nevertheless appear consistently. Some of these patterns improve coherence and robustness. Others degrade it.

In addition, experience has shown that certain decisions — particularly around **third-party dependency selection** — are empirically fragile when left to inference alone, even when the high-level intent is clear. In a small number of cases, the cost of repeatedly rediscovering this fragility outweighs the benefit of leaving the choice unconstrained.

This specification captures those lessons explicitly.

---

## Scope

This document is the **only place in the specification pack** where such interventions are declared.

Interventions MAY:

- require or prohibit specific recurring structural patterns,
- require or prohibit specific classes of design choice,
- explicitly select a small number of third-party dependencies where repeated instability has been observed and the choice is considered non-contentious in context.

Interventions MUST:

- be minimal in number,
- be justified by observed behaviour across prior generations,
- and be stated as constraints, not as implementation instructions.

---

## Non-Goals

This specification does **not** exist to:

- restate or refine the core system intent,
- encode general “best practices”,
- prescribe detailed implementations,
- or optimise for stylistic or aesthetic preference.

It is not a substitute for improving the primary specifications.  
It is a deliberate exception mechanism, used only where experience shows that abstraction alone is insufficient.

---

## Authority

Where this specification conflicts with other documents in the pack, **this specification takes precedence**.

Such conflicts are expected to be rare and should be treated as signals that the intervention itself may need to be revisited, reduced, or removed in future iterations.


## Intervention: Dependency Injection and Test Construction Shape

### Application Construction and Dependency Injection

1. The generated system MUST define an `Application` type that is instantiated explicitly by its callers.
2. `Application` MUST have a constructor that accepts exactly one argument, referred to as `dependencies` (or an equivalent name).
3. The `dependencies` argument MUST be a `Dependencies` structure.
4. `Dependencies` MUST contain only system-boundary interfaces required by the system.
5. At minimum, `Dependencies` MUST include:
   - a storage boundary interface, and
   - a clock boundary interface exposing a `timeNow()` operation (or an equivalent).
6. `Application` MUST NOT directly instantiate production implementations of system-boundary dependencies.

---

### Runtime Entry Point Wiring

7. The runtime HTTP entry point handler MUST act as a thin adapter.
8. The entry point handler MUST:
   - construct production implementations of each system-boundary dependency,
   - construct a `Dependencies` value from those implementations,
   - instantiate `Application` with that `Dependencies` value,
   - delegate request handling to the `Application` instance,
   - and return the result.
9. The entry point handler MUST NOT contain application logic beyond adaptation and wiring.

---

### Test Wiring and Immutability

10. Each test case MUST construct its own `Dependencies` value within the lexical scope of the test case.
11. Tests MUST supply fake or test-double implementations for all system-boundary dependencies.
12. Tests MUST NOT use real external resources, including real persistence, network access, or wall-clock time.
13. The `Dependencies` value constructed by a test MUST NOT be mutated after construction.
14. Tests SHOULD instantiate a new `Application` per test case using the test-local `Dependencies` value.

---

### Forbidden Patterns

15. Tests MUST NOT rely on global mutable singletons for system-boundary dependencies.
16. Tests MUST NOT use monkeypatching of module-level dependencies as the primary wiring mechanism.

17. `Application` MUST NOT obtain system-boundary dependencies from ambient global state.


### Intervention: Mandatory third-party packages for storage and validation

1. The generator MUST use the Go package `cloud.google.com/go/storage` as the sole mechanism for interacting with Google Cloud Storage services.
   - The generator MUST NOT introduce alternative Google Cloud Storage clients, REST wrappers, or custom mechanisms for direct interaction with Google Cloud Storage.

2. The generator MUST use the Go package `github.com/go-playground/validator/v10` for validation of externally supplied payloads that are decoded into Go data structures (including structs, maps, or lists).
   - For such payloads, validation MUST be applied after decoding/unmarshalling and before further application-level processing.
   - Validation failures MUST be handled in accordance with the error-handling rules defined elsewhere in the specification pack.
   - The generator MAY also use this package within tests to assert data structure shape or validation rules when doing so improves clarity and readability.

3. This requirement does not mandate use of `validator/v10` for transports or inputs where tag-based validation on decoded Go values is not practical (for example gRPC/protobuf inputs). In such cases, appropriate validation MUST still be performed using a suitable mechanism.
