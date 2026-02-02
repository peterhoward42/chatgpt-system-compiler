# GENERATION SPECIFICATION (Canonical)

## ROLE AND MODEL (MUST)

### Compiler model
ChatGPT MUST behave as a compiler.

For each iteration:
- The entire codebase MUST be generated afresh from the current specification.
- No reliance on previous generated output is permitted.
- Preservation of diffs between iterations is NOT required.

Global coherence and correctness MUST take precedence over local stability.

## SCOPE (MUST)

This specification defines the behavioural contract of the generator as a whole.

It governs what the generator is responsible for, how it collaborates with the
human author, and which principles control decisions when multiple valid
implementations are possible.


The authority relationship between the Intervention Specification and generator latitude is defined in `compiler-contract.md` and MUST be followed.

Specifically, this document defines:

1. **Human–generator collaboration model**
   - The authority of the specification as the sole input to generation.
   - The iterative workflow in which the human refines the specification and the
     generator regenerates the entire codebase.
   - The absence of continuity or diff-preservation guarantees between
     iterations.

2. **Generation priorities and conflict resolution**
   - Precedence of global coherence and correctness over local stability.
   - Preference for clarity over minimisation of duplication.
   - Preference for simplicity over cleverness.
   - Conservative resolution of ambiguity using assume-rather-than-ask
     principles.

3. **Structural and architectural intent**
   - Separation of concerns within generated code.
   - Explicit boundary definition via interfaces and injectable dependencies.
   - Purity, testability, and public-interface-driven correctness evidence.


## CONCURRENCY AND SYNCHRONISATION DISCIPLINE (MUST)

### Definitions

**Concurrent execution**: multiple logically independent execution streams whose lifetimes overlap in time (for example threads, goroutines, async tasks, or event-loop callbacks).

**Synchronisation primitive**: any mechanism whose primary purpose is to coordinate concurrent execution by controlling mutual exclusion, ordering, waiting/signalling, atomicity, or memory visibility.

This includes (non-exhaustively):
- locks, mutexes, monitors, and read–write locks,
- semaphores, condition variables, barriers, latches, and wait-groups,
- atomic operations (CAS, atomic variables, memory fences),
- thread-safe or concurrent data structures,
- coordination constructs such as channels or queues when used to order or block concurrent execution.

This definition applies to both thread-based and event-loop-based concurrency models.

### Design constraints

1. The generator MUST NOT introduce concurrent execution unless it is required by the specification or justified by a concrete, explicit need.
2. Where a design goal can be satisfied without concurrent execution, the generator MUST prefer a sequential design that avoids temporally overlapping access to mutable state.
3. The generator MUST NOT introduce synchronisation primitives primarily to make a concurrent design safe if a sequential design would satisfy the requirements.
4. If concurrent execution is required, the generator MUST:
   - minimise the scope of temporally overlapping access to mutable state,
   - confine ownership of mutable state where possible,
   - and define tests that would fail if the concurrency behaviour were incorrect.
5. If synchronisation primitives are introduced, the generator MUST either:
   - justify why concurrent execution is required and why a sequential design is insufficient, or
   - record a deviation under `compliance.md` with that justification and test strategy.



## REGULAR EXPRESSION DISCIPLINE (SHOULD)

### Rationale

Regular expressions can be concise, but are easy to mis-specify, hard to debug, and often difficult for maintainers to confidently modify.

The generator SHOULD prefer solutions whose correctness and intent are directly readable from the code.

### Definitions

**Regular expression lookaround**: any use of lookahead or lookbehind constructs (for example `(?=...)`, `(?!...)`, `(?<=...)`, `(?<!...)`).

### Design constraints

1. The generator SHOULD avoid regular expressions when an equally simple solution exists using structured parsing (for example: splitting, trimming, tokenisation, explicit scanning, or standard-library parsers).
2. Regular expressions MAY be used when they are the simplest and clearest approach and are expected to be readable and maintainable by typical developers.
3. If a regular expression is used, it SHOULD be:
   - anchored where appropriate (e.g. `^...$`),
   - limited in scope (matching a single well-defined fragment),
   - and documented with a short comment stating what it matches and what it rejects.
4. The generator MUST NOT use regular expressions with lookarounds.
5. The generator MUST NOT use complex regular expressions to encode non-trivial grammars, protocols, or schema-like validation rules when a structured approach is feasible.
6. For payload validation against a schema or rule-set, the generator SHOULD prefer:
   - explicit structured validation code, and/or
   - a schema-driven validator (for example JSON Schema validation) implemented via a stable standard library or a well-established third-party package.
7. If a complex regular expression is judged necessary, the generator MUST record a deviation under `compliance.md` that justifies the regular expression and specifies tests covering edge cases and known failure modes.


4. **Generator capabilities and tooling boundaries**
   - Which forms of tooling the generator MAY invoke during generation.
   - Which activities are explicitly deferred to the repository consumer.
   - Constraints designed to preserve determinism, portability, and
     reproducibility.

This specification is intentionally broad in scope.

Its purpose is to present a single coherent model of generation behaviour rather
than a collection of narrowly scoped, independent rules.

## TERMINOLOGY AND CONCEPTS (MUST)

### Public interface
The public interface is defined exclusively in `public-interface.md`.

It describes the externally observable contract of the system, including
request/response shapes, status codes, and `error_id` values.

### Public entrypoint
A public entrypoint is the concrete invocation surface through which the public
interface is exercised.

For this system, the public entrypoint is the deployed Cloud Function handler
receiving HTTP requests at the root path.

A public entrypoint is an implementation construct and MUST NOT redefine or
extend the public interface.

### Boundary interface
A boundary interface describes a required interaction with an external system or
side-effecting capability (for example storage, time, randomness, or networking).

Boundary interfaces:
- define required behaviour rather than concrete technologies,
- are supplied to core logic via injection,
- exist to make business rules deterministic and testable.

Boundary interfaces are sometimes referred to as “ports” in architectural
literature.

## TOOLING DURING GENERATION (MUST)

### Permitted tooling
The generator MAY run language tooling when necessary to produce a correct
repository output.

### Dependency resolution constraints (LOCKED)
For this repository, dependency resolution is intentionally deferred:
- The generator MUST NOT run commands that fetch or pin dependencies
  (including `go get`, `go mod tidy`, `go mod download`).
- The generator MUST NOT generate or update `go.sum`.
- The generator MUST output a minimal `go.mod` containing only:
  - `module`
  - `go`

The consumer of the repository is responsible for dependency resolution using
standard Go workflows prior to build or test.

### Testing during generation
The generator MAY run `go test ./...` only if doing so does not require module
fetching.

Any artifacts produced by such tooling that are required for a correct build MUST
be included in the generated output.

## ITERATION AND COLLABORATION (MUST)
- The human author iteratively refines the specification.
- After each refinement, ChatGPT MUST generate a complete and globally consistent
  codebase satisfying the specification.
- Large changes between iterations are intentional and acceptable.

The generated files MUST be returned as a gzipped filesystem via a download link
in the ChatGPT UI.

## AMBIGUITY RESOLUTION POLICY (MUST)
- ChatGPT MUST assume rather than ask whenever possible.
- Ambiguity MUST be resolved conservatively.
- Any assumptions required to complete generation MUST:
  - be made explicit in code comments, and
  - SHOULD be recorded in an assumptions log.

Simplicity is preferred over cleverness.

Avoiding duplication matters, but DRY MUST NOT override clarity.

## CODE ORGANISATION (SHOULD)
Generated code SHOULD:
- separate concerns to maximise readability,
- make public entrypoints easy to test end-to-end using faked external
  interfaces.

Separation SHOULD be reflected in package structure, file structure, and
abstractions where they simplify reasoning.

Simple designs are preferred over clever ones.

The architecture SHOULD make it possible (but not required) to unit-test
internal components when fixturing is genuinely simpler.

Correctness evidence for externally visible behaviour SHOULD primarily come from
tests that drive the system through the public system interface defined in
`public-interface.md`.

## PURITY-ON-IMPORT (MUST)
Any package intended for unit testing MUST be pure on import:
- Importing it MUST NOT fail due to missing credentials or configuration.
- Importing it MUST NOT require runtime infrastructure.

Code that depends on the runtime environment MUST be reachable only from explicit
entrypoint functions.

## TESTABILITY AND STRUCTURAL CONTRACT (MUST)

### External isolation
All interactions with external systems MUST be isolated behind minimal boundary
interfaces defined in production code.

Boundary interfaces MUST describe required capabilities rather than concrete
technologies.

Deterministic core logic MUST depend only on these interfaces.

### Public-interface-first testing (LOCKED)
Tests MUST validate behaviour primarily via the system’s public entrypoint(s).

For this system, the public entrypoint is the deployed Cloud Function handler
receiving HTTP requests at the root path.

Tests MUST:
- construct real HTTP requests,
- assert on HTTP responses and externally observable effects,
- use fake implementations of all external boundary interfaces.

Tests MUST cover:
- OPTIONS (CORS preflight),
- POST ingest (success, invalid payload, unsupported content type, storage
  failure),
- GET analysis (success, storage failure).

Tests MUST NOT:
- import or call internal or private helpers solely for testing convenience,
- assert on internal implementation details.

Assertions MUST be expressed only in terms of public outputs and externally
observable effects.

### Business-rule reachability (MUST)
All business-rule branches MUST be reachable from the public entrypoint by varying:
- request inputs, and/or
- behaviour or returned data of faked boundary interfaces.

Business logic MUST NOT depend on hidden global state or uncontrolled derived
inputs.

If a business rule branches on a value X, X MUST originate from:
- the request, or
- a boundary interface method return value (including values derived inside the
  pure core).

Boundary interfaces SHOULD be designed to allow independent variation of each
business-rule dimension without introducing test-only public parameters.

Non-HTTP invocations that are part of the external contract MAY be treated as
additional public entrypoints and MUST be tested using their real invocation
shape.

### Fakes, not reimplementation (MUST)
Tests MUST NOT reimplement business logic.

Tests MAY:
- construct inputs,
- supply fake implementations,
- assert observable outputs or effects.

Fakes MAY expose inspection hooks relevant to behaviour, but tests MUST NOT mirror
internal algorithms.

### Optional internal unit tests (MAY)
Internal unit tests MAY be added only when they materially simplify fixturing.

If present, they:
- MUST NOT be the sole evidence for externally visible behaviour,
- MUST be supplemented by at least one public-entrypoint test per behaviour,
- MUST avoid assertions on internal call structure.

## INJECTABILITY (MUST)
Access to time, randomness, configuration, or environment MUST be injectable.

Core logic MUST NOT call system time or random generators directly.

Any such values MUST be supplied via arguments or injected interfaces so tests are
deterministic.

## COMMENTING (MUST)
- All exported names MUST have concise, role-oriented comments.
- Non-exported names SHOULD be commented where helpful.
- Comments SHOULD explain non-obvious choices and subtle effects and remain short.

## THIRD-PARTY SDK USAGE (MUST)
For any third-party SDK method calls:
- Usage MUST be anchored to an authoritative minimal pattern.
- Receiver and type structure MUST match the authoritative source.

If uncertainty exists, the dependency MUST be isolated behind a small adapter and
handled according to the assumptions policy defined elsewhere in this
specification pack.
