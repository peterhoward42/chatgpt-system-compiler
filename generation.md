# GENERATION SPECIFICATION (generation.md)

## GENERATION MODEL (GENERATION PLANE, MUST)

ChatGPT must behave as a compiler. For each iteration, the entire codebase must be
generated afresh from the current specification without reliance on previous
generated output. There is no expectation of preserving diffs between iterations.
Global coherence and correctness take precedence over local stability.

## ITERATION AND COLLABORATION MECHANICS (GENERATION PLANE, MUST)

The human author iteratively refines the specification. After each refinement,
ChatGPT must generate a complete, globally consistent codebase satisfying the
specification. Large changes between iterations are intentional and acceptable.

## AMBIGUITY RESOLUTION POLICY (GENERATION PLANE, MUST)

ChatGPT must assume rather than ask whenever possible. Ambiguity must be resolved
conservatively. Any assumptions required to complete generation must be made
explicit in code comments and should be recorded in an assumptions log. Simplicity
is preferred over cleverness. Avoiding duplication matters, but DRY does not
override clarity.

## CODE ORGANISATION AND SEPARATION OF CONCERNS (GENERATION PLANE, SHOULD)

Generated code should separate concerns to maximise readability and to make the
public entrypoint(s) easy to test end-to-end using faked external interfaces.
Separation should be reflected in package structure, file structure, and
abstractions where they simplify reasoning. Simple designs are preferred over
clever ones.

The architecture should make it possible (but not necessary) to unit-test internal
components when fixturing is genuinely simpler; however, correctness evidence for
externally visible behaviour must primarily come from tests that drive the system
through its public interface(s).

## TESTABILITY AND STRUCTURAL CONTRACT (GENERATION PLANE, MUST)

All interactions with external systems must be isolated behind minimal capability
interfaces defined in production code. Interfaces must describe required
capabilities rather than concrete technologies. Deterministic core logic must
depend only on these interfaces.

### PUBLIC-INTERFACE-FIRST TESTING (MANDATORY)

Tests must validate behaviour primarily via the system’s public entrypoint(s).
For this system, the public entrypoint is the deployed Cloud Function handler
InjestEvent receiving HTTP requests at the root path.

Accordingly:

Tests MUST exercise production behaviour by constructing HTTP requests and
asserting on HTTP responses and externally observable effects, using fake
implementations of all external capability interfaces.

Tests MUST cover the HTTP surface of the entrypoint, including:
- OPTIONS (CORS preflight) behaviour
- POST ingest behaviour (success, invalid payload, unsupported content type,
  storage failure)
- GET analysis behaviour (success, storage failure)

Tests MUST NOT import, call, or construct internal/private modules, helpers, or
functions solely for testing convenience. (If a package or symbol is not part of
the explicitly defined public interface, tests must not depend on it.)

Tests MUST NOT assert on internal implementation details (private state,
intermediate values, internal call order). Assertions must be expressed in terms
of public outputs and externally observable effects only.

### REACHABILITY OF BUSINESS-RULE BRANCHES (MANDATORY)

All business-rule decisions (branching logic) MUST be reachable from the public
entrypoint by varying:
(a) request inputs and/or
(b) the behaviour and returned data of faked capability interfaces (ports).

Business logic MUST NOT depend on hidden global state or derived inputs that
cannot be controlled via request inputs or injectable ports. If a business rule
branches on a value X, then X MUST come from either the request or a port method
return value (including values computed from port return values inside the pure
core).

Ports SHOULD be designed so tests can independently vary each dimension that
affects business rules (e.g. user attributes, feature flags, prior state) without
adding test-only parameters to the public interface.

(For other projects: if queue subscribers, scheduled triggers, or other non-HTTP
invocations are part of the externally visible contract, they MAY be treated as
additional public entrypoints and MUST be tested via their real invocation shape
using faked external interfaces, following the same principles above.)

### FAKES, NOT REIMPLEMENTATION

Tests must not reimplement business logic. Tests may construct inputs, supply fake
implementations, and assert observable outputs or effects. Fakes may provide
inspection through public, behaviour-relevant observations (e.g. “objects written”
or “data stored”), but tests must not mirror the code’s internal algorithms.

### OPTIONAL INTERNAL UNIT TESTS (PERMITTED, BUT NOT PRIMARY)

Internal unit tests MAY be added only when they materially simplify fixturing for
hard-to-reach edge cases. If present, such tests:
- MUST NOT be the sole or primary evidence for any externally visible behaviour.
  For each externally visible behaviour, at least one test MUST exercise the real
  public entrypoint via HTTP using fake external interfaces.
- MUST remain resilient to refactoring by avoiding assertions about internal call
  structure and by focusing on stable contracts.

### INJECTABILITY REQUIREMENTS

Access to time, randomness, configuration, or environment must be injectable.
Core logic must not call system time or random generators directly. Configuration
must follow the system-plane configuration policy (compile-time constants for
non-secrets), but any time/randomness used in logic must be supplied as an
argument or via an injected interface so tests are deterministic.

## COMMENTING RULES (GENERATION PLANE, MUST)

All exported names must be commented using concise role-oriented descriptions.
Non-exported names should be commented where helpful for overview reading.
Comments should explain non-obvious choices and subtle effects and should remain
short.
