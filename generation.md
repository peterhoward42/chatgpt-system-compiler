# GENERATION SPECIFICATION (Canonical)

## MUST: Generation model
ChatGPT **MUST** behave as a compiler.

For each iteration, the entire codebase **MUST** be generated afresh from the
current specification without reliance on previous generated output.

There is no expectation of preserving diffs between iterations.

Global coherence and correctness **MUST** take precedence over local stability.

## MUST: Iteration and collaboration mechanics
The human author iteratively refines the specification.

After each refinement, ChatGPT **MUST** generate a complete, globally consistent
codebase satisfying the specification.

Large changes between iterations are intentional and acceptable.

The generated files should should be returned to the human as a gzipped filestystem and presented as a download link in ChatGPT's UI.

## MUST: Ambiguity resolution policy
ChatGPT **MUST** assume rather than ask whenever possible.

Ambiguity **MUST** be resolved conservatively.

Any assumptions required to complete generation **MUST** be made explicit in code
comments and **SHOULD** be recorded in an assumptions log.

Simplicity is preferred over cleverness.

Avoiding duplication matters, but DRY **MUST NOT** override clarity.

## SHOULD: Code organisation and separation of concerns
Generated code **SHOULD** separate concerns to maximise readability and to make the
public entrypoint(s) easy to test end-to-end using faked external interfaces.

Separation **SHOULD** be reflected in package structure, file structure, and
abstractions where they simplify reasoning.

Simple designs are preferred over clever ones.

The architecture **SHOULD** make it possible (but not necessary) to unit-test
internal components when fixturing is genuinely simpler.

Correctness evidence for externally visible behaviour **SHOULD** primarily come
from tests that drive the system through its public interface(s).

## MUST: Purity-on-import rule
Any package/module intended for unit testing MUST be “pure on import”: importing it cannot fail due to missing credentials/config, and cannot require runtime infrastructure.

Any code whose execution depends on the runtime environment MUST be reachable only from an explicit entrypoint function.

## MUST: Testability and structural contract
All interactions with external systems **MUST** be isolated behind minimal
capability interfaces defined in production code.

Interfaces **MUST** describe required capabilities rather than concrete
technologies.

Deterministic core logic **MUST** depend only on these interfaces.

### MUST: Public-interface-first testing
Tests **MUST** validate behaviour primarily via the system’s public entrypoint(s).

For this system, the public entrypoint is the deployed Cloud Function handler
`InjestEvent` receiving HTTP requests at the root path.

Tests **MUST** exercise production behaviour by constructing HTTP requests and
asserting on HTTP responses and externally observable effects, using fake
implementations of all external capability interfaces.

Tests **MUST** cover the HTTP surface of the entrypoint, including:
- OPTIONS (CORS preflight) behaviour
- POST ingest behaviour (success, invalid payload, unsupported content type,
  storage failure)
- GET analysis behaviour (success, storage failure)

Tests **MUST NOT** import, call, or construct internal or private modules, helpers,
or functions solely for testing convenience.

Tests **MUST NOT** assert on internal implementation details, including private
state, intermediate values, or internal call order.

Assertions **MUST** be expressed only in terms of public outputs and externally
observable effects.

### MUST: Reachability of business-rule branches
All business-rule decisions **MUST** be reachable from the public entrypoint by
varying:
- request inputs, and/or
- the behaviour and returned data of faked capability interfaces.

Business logic **MUST NOT** depend on hidden global state or derived inputs that
cannot be controlled via request inputs or injectable ports.

If a business rule branches on a value X, then X **MUST** come from either the
request or a port method return value, including values computed from port return
values inside the pure core.

Ports **SHOULD** be designed so tests can independently vary each dimension that
affects business rules without adding test-only parameters to the public
interface.

If non-HTTP invocations are part of the externally visible contract, they **MAY**
be treated as additional public entrypoints and **MUST** be tested via their real
invocation shape using faked external interfaces.

### MUST: Fakes, not reimplementation
Tests **MUST NOT** reimplement business logic.

Tests **MAY** construct inputs, supply fake implementations, and assert observable
outputs or effects.

Fakes **MAY** provide inspection through public, behaviour-relevant observations,
but tests **MUST NOT** mirror the code’s internal algorithms.

### MAY: Optional internal unit tests
Internal unit tests **MAY** be added only when they materially simplify fixturing
for hard-to-reach edge cases.

If present, such tests:
- **MUST NOT** be the sole or primary evidence for any externally visible behaviour.
- For each externally visible behaviour, at least one test **MUST** exercise the
  real public entrypoint via HTTP using fake external interfaces.
- **MUST** remain resilient to refactoring by avoiding assertions about internal
  call structure and by focusing on stable contracts.

### MUST: Injectability requirements
Access to time, randomness, configuration, or environment **MUST** be injectable.

Core logic **MUST NOT** call system time or random generators directly.

Any time or randomness used in logic **MUST** be supplied as an argument or via an
injected interface so tests are deterministic.

## MUST: Commenting rules
All exported names **MUST** be commented using concise role-oriented descriptions.

Non-exported names **SHOULD** be commented where helpful for overview reading.

Comments **SHOULD** explain non-obvious choices and subtle effects and **SHOULD**
remain short.

## MUST: third-party SDK method signature correctness rule
For any third-party SDK method calls, the generator must anchor usage to an authoritative minimal pattern and follow the same receiver/type structure. If uncertain, the generator must isolate the dependency behind a small adapter and deal with it according to using the assumptions policy detailed elsewhere in this pack.
